# Lesson 15 — Automated Tests (Let Python Check Your Website)

**Time:** 30 minutes  |  **You need:** Lesson 14 finished

---

## Goal

By the end of this lesson you will:

- write tests that **visit your pages automatically** and check the results,
- run them with **one command**,
- see a test **fail** on purpose and read the error,
- add a new model field **and** a test for it (the full professional cycle).

---

## Understand

### What is a test?

A **test** is a small program that checks your program. Instead of clicking through 20 pages by hand after every change, you type one command and Python checks them all in seconds.

```python
def test_home_page_returns_success(self):
    response = self.client.get(reverse("journal:home"))   # visit the home page
    self.assertEqual(response.status_code, 200)            # it must answer 200 OK
```

### The 3 A's — every test has the same shape

| Step | Meaning | Example |
|---|---|---|
| **Arrange** | Prepare the data | Create a user and an article |
| **Act** | Do the action | Visit `/articles/` |
| **Assert** | Check the result | The article title is on the page |

### Where does test data go?

Django creates a **separate, temporary test database** (named `test_neondb`), runs the tests, then **deletes it**. Your real articles are **never touched**.

`self.client` is a **pretend browser** that can `get` and `post` pages and even `login`.

---

## Do it now

### Step 1 — Write your first 3 tests

Open **`journal/tests.py`**. Replace everything with:

```python
from django.contrib.auth import get_user_model
from django.test import TestCase
from django.urls import reverse

from .models import Article

User = get_user_model()


class JournalTests(TestCase):
    def setUp(self):
        # ARRANGE: this runs before EVERY test
        self.author = User.objects.create_user(username="author", password="Strong-password-12345")
        self.article = Article.objects.create(
            author=self.author,
            title="A useful Django article",
            body="This is a sufficiently long article body.",
            is_published=True,
        )

    def test_home_page_returns_success(self):
        response = self.client.get(reverse("journal:home"))     # ACT
        self.assertEqual(response.status_code, 200)              # ASSERT

    def test_published_article_is_listed(self):
        response = self.client.get(reverse("journal:article_list"))
        self.assertContains(response, self.article.title)

    def test_logged_out_user_cannot_create_article(self):
        response = self.client.get(reverse("journal:article_create"))
        self.assertEqual(response.status_code, 302)               # redirected...
        self.assertIn("/accounts/login/", response.url)           # ...to the login page
```

### Step 2 — Run them

```bash
python manage.py test
```

Expected (it takes 10–30 seconds because it creates a database on Neon):

```text
Found 3 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...
----------------------------------------------------------------------
Ran 3 tests in 2.345s

OK
Destroying test database for alias 'default'...
```

Each **`.`** is one passing test. **OK** = all passed.

### Step 3 — Break something on purpose (to see a failure)

In **`journal/views.py`**, in `article_list`, change `is_published=True` to `is_published=False`. Save. Run the tests again:

```bash
python manage.py test
```

Now you see an **`F`** and:

```text
FAIL: test_published_article_is_listed (journal.tests.JournalTests...)
...
AssertionError: False is not true : Couldn't find 'A useful Django article' in the following response
```

**How to read it:** look at the **FAIL** line (which test) and the **last line** (why). The test caught the bug before any real user did!

**Change it back to `is_published=True`** and run the tests again → **OK**.

### Step 4 — Add the full test suite

Now replace **`journal/tests.py`** with the complete version, which also tests drafts, editing, ownership, categories and search:

```python
from django.contrib.auth import get_user_model
from django.test import TestCase
from django.urls import reverse

from .models import Article, Category

User = get_user_model()


class JournalTests(TestCase):
    def setUp(self):
        self.author = User.objects.create_user(username="author", password="Strong-password-12345")
        self.other = User.objects.create_user(username="other", password="Other-strong-password-12345")
        self.article = Article.objects.create(
            author=self.author,
            title="A useful Django article",
            body="This is a sufficiently long article body.",
            is_published=True,
        )

    def test_home_page_returns_success(self):
        response = self.client.get(reverse("journal:home"))
        self.assertEqual(response.status_code, 200)

    def test_published_article_is_listed(self):
        response = self.client.get(reverse("journal:article_list"))
        self.assertContains(response, self.article.title)

    def test_draft_article_is_not_public(self):
        draft = Article.objects.create(
            author=self.author,
            title="Private draft",
            body="This draft should not be visible publicly.",
            is_published=False,
        )
        response = self.client.get(reverse("journal:article_list"))
        self.assertNotContains(response, draft.title)

    def test_logged_out_user_cannot_create_article(self):
        response = self.client.get(reverse("journal:article_create"))
        self.assertEqual(response.status_code, 302)
        self.assertIn("/accounts/login/", response.url)

    def test_author_can_update_own_article(self):
        self.client.login(username="author", password="Strong-password-12345")
        response = self.client.post(
            reverse("journal:article_update", args=[self.article.pk]),
            {
                "title": "Updated Django title",
                "subtitle": "",
                "body": "This updated body is long enough.",
                "is_published": "on",
            },
        )
        self.assertEqual(response.status_code, 302)
        self.article.refresh_from_db()
        self.assertEqual(self.article.title, "Updated Django title")

    def test_user_cannot_update_another_users_article(self):
        self.client.login(username="other", password="Other-strong-password-12345")
        response = self.client.get(reverse("journal:article_update", args=[self.article.pk]))
        self.assertEqual(response.status_code, 404)

    def test_many_to_many_categories(self):
        first = Category.objects.create(name="Django")
        second = Category.objects.create(name="Database")
        self.article.categories.add(first, second)
        self.assertEqual(self.article.categories.count(), 2)

    def test_search_filters_articles(self):
        response = self.client.get(reverse("journal:article_list"), {"q": "useful"})
        self.assertContains(response, self.article.title)
```

And replace **`accounts/tests.py`** with:

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.models import Group
from django.test import TestCase
from django.urls import reverse

from .models import Profile

User = get_user_model()


class AccountTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username="learner", password="Strong-password-12345")

    def test_profile_is_one_to_one(self):
        profile = Profile.objects.create(user=self.user, display_name="Learner")
        self.assertEqual(self.user.profile, profile)

    def test_registration_creates_account_and_profile(self):
        response = self.client.post(reverse("accounts:register"), {
            "username": "newlearner",
            "password1": "Another-strong-password-12345",
            "password2": "Another-strong-password-12345",
        })
        self.assertEqual(response.status_code, 302)
        self.assertTrue(User.objects.filter(username="newlearner").exists())
        self.assertTrue(Profile.objects.filter(user__username="newlearner").exists())

    def test_normal_user_cannot_open_editor_area(self):
        self.client.login(username="learner", password="Strong-password-12345")
        response = self.client.get(reverse("accounts:editor_area"))
        self.assertEqual(response.status_code, 302)

    def test_content_editor_group_can_open_editor_area(self):
        group, _ = Group.objects.get_or_create(name="ContentEditors")
        self.user.groups.add(group)
        self.client.login(username="learner", password="Strong-password-12345")
        response = self.client.get(reverse("accounts:editor_area"))
        self.assertEqual(response.status_code, 200)
```

Run:

```bash
python manage.py test
```

Expected: **`Ran 12 tests ... OK`**

> Tip: to run only one app's tests: `python manage.py test journal`

---

## Check

- [ ] `python manage.py test` → **Ran 12 tests … OK**.
- [ ] You saw an **F** (failure) in Step 3, understood it, and fixed it.

---

## What just happened?

| Code | Meaning |
|---|---|
| `class JournalTests(TestCase)` | A group of tests. Each runs inside a transaction that is undone afterwards — tests never affect each other |
| `setUp(self)` | Runs **before each** test (Arrange) |
| Methods starting with **`test_`** | Django finds and runs them automatically |
| `self.client.get(url)` / `.post(url, data)` | The pretend browser |
| `self.client.login(...)` | Log the pretend browser in |
| `reverse("journal:home")` | Turn a URL **name** into an address (`/`) — same as `{% url %}` in templates |
| `assertEqual(a, b)` | Check `a == b` |
| `assertContains(response, text)` | The page HTML contains `text` (and status is 200) |
| `refresh_from_db()` | Reload the object from the database to see the saved change |

The **ownership** and **role** tests are the most valuable ones: they prove your security rules work, and they'll shout if someone breaks them later.

> A passing test proves **only what it checks**. It doesn't prove there are zero bugs. Write tests for the things that would hurt most if they broke.

### How to read an error (a traceback)

When something crashes, Python prints a **traceback** — the path the program took to the error:

```text
Traceback (most recent call last):
  File ".../django/core/handlers/exception.py", line 55, in inner      ← Django's own code (skip)
  ...
  File "D:\...\my_journal\journal\views.py", line 12, in article_list ← YOUR file and line
    articles = Article.objects.filter(is_publshed=True)
django.core.exceptions.FieldError: Cannot resolve keyword 'is_publshed' into field.  ← START HERE
```

1. Read the **last line** first — it says **what** went wrong (here: a typo, `is_publshed`).
2. Go up until you find a line in **your own** file (`my_journal\...`) — that's **where**.
3. Open that file at that line and look at the code around it.
4. Fix **one** thing, then run again.
5. Still stuck? Copy the **last line** into a search engine, or check `references/COMMON_ERRORS.md`.

Don't change five files at once hoping something works — you won't know what fixed it (or what you broke).

---

## Try it yourself — the full professional cycle

Add a **reading time** to articles:

1. **Model** — in `journal/models.py`, inside `Article`, under `body`:
   ```python
   reading_time = models.PositiveIntegerField(default=1, help_text="Minutes to read")
   ```
2. **Migrate:**
   ```bash
   python manage.py makemigrations journal
   ```
   ```bash
   python manage.py migrate
   ```
3. **Form** — add `"reading_time"` to `fields` in `journal/forms.py`.
4. **Admin** — add `"reading_time"` to `list_display` in `journal/admin.py`.
5. **Template** — in `article_detail.html`, show `<p>{{ article.reading_time }} min read</p>`.
6. **Test** — add to `JournalTests`:
   ```python
   def test_reading_time_defaults_to_one(self):
       self.assertEqual(self.article.reading_time, 1)
   ```
7. **Run** `python manage.py test` → **Ran 13 tests … OK**.
8. Fix the **old** test `test_author_can_update_own_article` if it fails — the form now has a new field. *(Hint: add `"reading_time": 3` to the posted data.)*

That's how real developers ship a feature: **model → migration → form → admin → template → test**.

---

## If it does not work

| Problem | Fix |
|---|---|
| `Got an error creating the test database: database "test_neondb" already exists` → `Type 'yes'...` | A previous test run was interrupted. Type **`yes`** and press Enter. |
| `permission denied to create database` | Use the **`neondb_owner`** role's connection string (the default) in `.env`. |
| Tests are slow | Normal — the test database is on the internet. Use `python manage.py test --keepdb` to reuse it between runs. |
| `NoReverseMatch` in a test | A URL name in the test doesn't match `urls.py`. |
| `Ran 0 tests` | Test methods must start with `test_`, and the file must be named `tests.py`. |

Compare with the answers: `django_lab/journal/tests.py` and `django_lab/accounts/tests.py`.

---

**Next:** [16_LOOK_INSIDE_POSTGRESQL_WITH_SQL.md](16_LOOK_INSIDE_POSTGRESQL_WITH_SQL.md)
