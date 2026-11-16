# Lesson 09 — Relationships Between Tables

**Time:** 30 minutes  |  **You need:** Lesson 08 finished (you have a superuser and some articles)

---

## Goal

By the end of this lesson you can explain and **show** the three kinds of relationships:

| Relationship | Our example | Django field |
|---|---|---|
| **One-to-One** | one user ↔ one profile | `OneToOneField` |
| **One-to-Many** | one user → many articles | `ForeignKey` |
| **Many-to-Many** | many articles ↔ many categories | `ManyToManyField` |

You will also create a second app, **`accounts`**, with a **Profile** model.

---

## Understand

### Why link tables at all?

Imagine every article stored the author's name, email and phone number. If the author changes their email, you must fix **100 articles**. Miss one → wrong data.

Better: store the author **once** in the users table, and each article just stores the author's **id**. That id is called a **foreign key**.

### 1. One-to-Many (ForeignKey) — "one class teacher, many students"

```text
   auth_user                     journal_article
┌────┬──────────┐            ┌────┬────────────────────┬───────────┐
│ id │ username │            │ id │ title              │ author_id │
├────┼──────────┤            ├────┼────────────────────┼───────────┤
│ 1  │ admin    │◀───────────│ 1  │ My first article   │ 1         │
│    │          │◀───────────│ 2  │ A secret draft     │ 1         │
└────┴──────────┘            └────┴────────────────────┴───────────┘
  ONE user               ◀──   MANY articles point to that user
```

The `ForeignKey` lives on the **"many"** side (Article). You already wrote it in Lesson 07: `author = models.ForeignKey(...)`.

### 2. Many-to-Many — "students and clubs"

One student can join many clubs; one club has many students. An article can have many categories; a category can have many articles.

Neither table can hold this alone, so Django makes a **third, linking table**:

```text
journal_article          journal_article_categories          journal_category
┌────┬──────────┐        ┌────────────┬─────────────┐        ┌────┬───────────┐
│ 1  │ My first │◀───────│ article_id │ category_id │───────▶│ 1  │ Django    │
│ 2  │ A secret │        ├────────────┼─────────────┤        │ 2  │ Databases │
└────┴──────────┘        │ 1          │ 1 (Django)  │        │ 3  │ Python    │
                         │ 1          │ 3 (Python)  │        └────┴───────────┘
                         └────────────┴─────────────┘
```

You already wrote it too: `categories = models.ManyToManyField(Category, ...)`.

### 3. One-to-One — "one student, one ID card"

Each user has exactly **one** profile, and each profile belongs to exactly **one** user. Django's `User` table handles login (username, password). Extra info (display name, bio) goes in a separate **Profile** table, linked one-to-one.

### `on_delete` — what happens when the "one" side is deleted?

| Option | Meaning | Real example |
|---|---|---|
| `CASCADE` | Delete the linked rows **too** | User deleted → their profile is deleted too |
| `PROTECT` | **Refuse** to delete while linked rows exist | Can't delete a user who still has articles |
| `SET_NULL` | Keep the row but empty the link (field needs `null=True`) | Keep the article, author becomes "unknown" |

Choosing one is a **business decision**: "What should happen to the data?"

---

## Do it now

> Stop the server (**Ctrl + C**) while you run commands.

### Step 1 — Create the `accounts` app

```bash
python manage.py startapp accounts
```

### Step 2 — Add it to `INSTALLED_APPS`

In **`config/settings.py`**:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "accounts",                     # ADD THIS LINE
    "journal",
]
```

### Step 3 — Write the Profile model

Open **`accounts/models.py`**. Replace everything with:

```python
from django.conf import settings
from django.db import models


class Profile(models.Model):
    """One-to-One: each profile belongs to exactly one user."""

    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="profile",
    )
    display_name = models.CharField(max_length=80, blank=True)
    bio = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.display_name or self.user.get_username()
```

> `settings.AUTH_USER_MODEL` means "Django's User model". Using this name (instead of importing `User` directly) is the professional way.

### Step 4 — Show Profile in Admin

Open **`accounts/admin.py`**. Replace everything with:

```python
from django.contrib import admin

from .models import Profile


@admin.register(Profile)
class ProfileAdmin(admin.ModelAdmin):
    list_display = ("display_name", "user", "created_at")
    search_fields = ("display_name", "user__username")
```

### Step 5 — Migrate

```bash
python manage.py makemigrations accounts
```
```bash
python manage.py migrate
```

Expected: `+ Create model Profile` and then `Applying accounts.0001_initial... OK`.

Neon → **Tables** now also shows **`accounts_profile`** with a **`user_id`** column.

### Step 6 — See all three relationships in action (in Admin)

Start the server (`python manage.py runserver`) and open http://127.0.0.1:8000/admin/

**One-to-One:**
1. **Users → Add user** → username `bob`, a password → **Save**.
2. **Profiles → Add** → User: `bob`, Display name: `Bob the Builder` → **Save**.
3. Try adding **another** profile for `bob`. *"Profile with this User already exists."* — one user, **one** profile!

**One-to-Many:**

4. **Articles → Add** → Author: `bob`, Title: `Bob's article`, any body, published → **Save**.
   Now `admin` has 2 articles and `bob` has 1. One user → many articles.

**Many-to-Many:**

5. Open `My first article` — it has **Django** and **Python**. Open **Categories → Django** — it is used by many articles. Many ↔ many.
6. Neon → **SQL Editor**:
   ```sql
   SELECT * FROM journal_article_categories;
   ```
   You see the linking rows (`article_id`, `category_id`) — the hidden third table!

**`on_delete` — PROTECT vs CASCADE:**

7. **Users → bob → Delete**. Django refuses: *"Deleting the user 'bob' would require deleting the following protected related objects: Article: Bob's article"*. That's **`PROTECT`** on `Article.author`.
8. Go to **Articles**, delete **Bob's article**.
9. Now **Users → bob → Delete** again. This time the confirmation page lists **Profile: Bob the Builder** as something that will **also** be deleted. That's **`CASCADE`** on `Profile.user`. Click **Yes, I'm sure**.
10. **Profiles** — Bob's profile is gone too.

---

## Check

- [ ] `accounts_profile` table exists in Neon.
- [ ] You saw the "already exists" error for a second profile (One-to-One).
- [ ] You saw PROTECT stop the deletion of `bob` while he had an article.
- [ ] You saw CASCADE delete `bob`'s profile with him.
- [ ] You ran `SELECT * FROM journal_article_categories;` and saw linking rows.

---

## What just happened?

| You wrote | Django created in PostgreSQL |
|---|---|
| `Article.author = ForeignKey(User)` | column `journal_article.author_id` |
| `Article.categories = ManyToManyField(Category)` | a whole new table `journal_article_categories` |
| `Profile.user = OneToOneField(User)` | column `accounts_profile.user_id` with a **unique** rule |

`related_name` lets you walk the link **backwards**:

| Forwards | Backwards (thanks to `related_name`) |
|---|---|
| `article.author` → the user | `user.articles.all()` → all their articles |
| `article.categories.all()` → its categories | `category.articles.all()` → articles in it |
| `profile.user` → the user | `user.profile` → their profile |

You'll use these in Python in the next lesson.

---

## Try it yourself

Design on paper (no code needed): a **school library**.

- Tables: `Student`, `Book`, `Author`, `Loan`.
- Which relationship is **Book ↔ Author**? (A book can have several authors, an author writes several books.)
- Which relationship is **Student → Loan**? (A student can borrow many times.)
- If a `Book` is deleted, what should happen to old `Loan` records — CASCADE, PROTECT or SET_NULL? Why?

<details>
<summary>Possible answers</summary>

- Book ↔ Author: **Many-to-Many**.
- Student → Loan: **One-to-Many** (`Loan.student = ForeignKey(Student)`).
- Probably **PROTECT** (don't lose borrowing history) — but it's a business decision; explain your reason.
</details>

---

## If it does not work

| Problem | Fix |
|---|---|
| `No changes detected` for accounts | `"accounts"` not added to `INSTALLED_APPS`, or `models.py` not saved. |
| Profiles don't appear in Admin | `accounts/admin.py` not saved. Restart the server. |
| `IntegrityError ... duplicate key ... user_id` | You tried to give one user two profiles — that's the One-to-One rule working. |

Compare with the answers: `django_lab/accounts/models.py` and `django_lab/accounts/admin.py`.

---

**Next:** [10_TALK_TO_THE_DATABASE_ORM.md](10_TALK_TO_THE_DATABASE_ORM.md)
