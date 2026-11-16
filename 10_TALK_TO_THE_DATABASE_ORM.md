# Lesson 10 — Talk to the Database with Python (the ORM)

**Time:** 30 minutes  |  **You need:** Lesson 09 finished (you have categories and articles)

---

## Goal

By the end of this lesson you can, **with Python code**:

- **read** rows (all, filtered, one),
- **create** a row,
- **update** a row,
- **delete** a row,
- follow relationships (`article.author`, `user.articles.all()`),
- see the **SQL** Django writes for you.

These four actions — **C**reate, **R**ead, **U**pdate, **D**elete — are called **CRUD**. Almost every app in the world is CRUD.

---

## Understand

### What is an ORM?

PostgreSQL speaks **SQL**. You speak **Python**. The **ORM** (Object-Relational Mapper) is the **translator** in the middle.

| You write (Python) | Django sends (SQL) |
|---|---|
| `Article.objects.all()` | `SELECT * FROM journal_article` |
| `Article.objects.filter(is_published=True)` | `SELECT ... WHERE is_published = true` |
| `Article.objects.create(title="Hi", ...)` | `INSERT INTO journal_article ...` |
| `article.save()` | `UPDATE journal_article SET ...` |
| `article.delete()` | `DELETE FROM journal_article WHERE id = ...` |

### What is a QuerySet?

`Article.objects.filter(...)` gives you a **QuerySet** — like a **list of rows** from the database. You can filter it more, sort it, count it, or loop over it.

### `get()` vs `filter()`

| | `get(...)` | `filter(...)` |
|---|---|---|
| Returns | **exactly one** object | a QuerySet (0, 1 or many) |
| If nothing matches | error `DoesNotExist` | empty QuerySet `<QuerySet []>` |
| Use it when | you want one specific row (by id) | you want a list |

### Lookups — the double underscore `__`

| Write | Means |
|---|---|
| `title="Hi"` | title is exactly "Hi" |
| `title__icontains="django"` | title contains "django" (any case) |
| `id__gt=2` | id **g**reater **t**han 2 (`gte`, `lt`, `lte` also exist) |
| `author__username="admin"` | follow the link to the author, then check the username |

---

## Do it now

### Step 1 — Open the Django shell

Stop the server or use a second terminal (activated, inside `my_journal`):

```bash
python manage.py shell
```

You see `>>>`. This is Python, but **connected to your project and database**.

> Type each line below **one at a time** and press **Enter**. Look at the result before typing the next one. Your ids and dates may be different — that's OK.

### Step 2 — Import your models

```python
from journal.models import Article, Category
from accounts.models import Profile
from django.contrib.auth import get_user_model
User = get_user_model()
```

### Step 3 — READ

```python
Article.objects.all()
```
Something like: `<QuerySet [<Article: My first article>, <Article: A secret draft>]>`
(It shows titles because of your `__str__` method!)

```python
Article.objects.count()
```
```python
Article.objects.filter(is_published=True)
```
```python
Article.objects.filter(title__icontains="first")
```
```python
Category.objects.all()
```
`<QuerySet [<Category: Databases>, <Category: Django>, <Category: Python>]>` — sorted A→Z by your `Meta.ordering`.

```python
article = Article.objects.get(title="My first article")
article.title
article.is_published
article.created_at
```

### Step 4 — Follow relationships

```python
article.author
article.author.username
article.categories.all()
```

Now go **backwards** (thanks to `related_name`):

```python
admin_user = User.objects.get(username="admin")
admin_user.articles.all()
admin_user.articles.count()
django_cat = Category.objects.get(name="Django")
django_cat.articles.all()
```

### Step 5 — CREATE

```python
new = Article.objects.create(author=admin_user, title="Made in the shell", body="I created this article with Python code.", is_published=True)
new.id
```
You get a new id number.

Add a category to it (Many-to-Many):

```python
python_cat = Category.objects.get(name="Python")
new.categories.add(python_cat)
new.categories.all()
python_cat.articles.all()
```

Create a profile for your admin user (One-to-One):

```python
Profile.objects.get_or_create(user=admin_user)
admin_user.profile
```
> `get_or_create` = "get it if it exists, otherwise create it". It returns a pair: `(profile, True)` if it was just created, `(profile, False)` if it already existed.

Now open **http://127.0.0.1:8000/admin/** (start the server in another terminal if needed) → **Articles**. "Made in the shell" is there! Python code and Admin use the **same** database.

### Step 6 — UPDATE

```python
new.title = "Made in the shell (updated)"
new.save()
Article.objects.get(id=new.id).title
```
`'Made in the shell (updated)'`

> Changing `new.title` alone does **nothing** to the database. Only `.save()` writes it.

### Step 7 — Sort and limit

```python
Article.objects.order_by("title")
Article.objects.order_by("-created_at")[:2]
```
`-` = reverse (newest first). `[:2]` = only the first 2.

### Step 8 — See the SQL

```python
print(Article.objects.filter(is_published=True).query)
```
You see a real `SELECT ... FROM "journal_article" WHERE "journal_article"."is_published" ...` — the SQL Django made for you.

### Step 9 — DELETE

```python
new.delete()
Article.objects.filter(title__icontains="shell")
```
`(2, {...})` (the article **plus** its 1 link row in `journal_article_categories`), then `<QuerySet []>` — it's gone.

### Step 10 — See an error on purpose

```python
Article.objects.get(title="This does not exist")
```
`journal.models.Article.DoesNotExist: Article matching query does not exist.`
That's why web pages use `get_object_or_404()` instead — it turns this into a friendly **404** page (Lesson 11).

### Step 11 — Leave the shell

```python
exit()
```

---

## Check

- [ ] You created, updated and deleted an article from the shell.
- [ ] `admin_user.articles.all()` showed that user's articles.
- [ ] You saw the SQL with `print(... .query)`.
- [ ] You saw the `DoesNotExist` error and know why it happens.

---

## What just happened?

- `Article.objects` is the **manager** — the doorway to the table.
- `.all()`, `.filter()`, `.order_by()` build a **QuerySet**. Django **waits** until you actually need the results (e.g. print them) before talking to the database — this is called being **lazy**.
- `.create()` → `INSERT`, `.save()` → `UPDATE` (or `INSERT` if new), `.delete()` → `DELETE`.
- You never wrote SQL, but SQL ran on PostgreSQL. And because the ORM sends your values **separately** from the SQL text, it is protected against **SQL injection** attacks.

In the next lessons, the **same** ORM code goes inside **views**, so it runs when someone opens a web page.

### Two more things worth knowing

**1. Updating many rows at once**

```python
# Example only. Don't type this now: it would publish your draft, which you need in Lesson 11.
Article.objects.filter(author__username="admin").update(is_published=True)
```

This changes all matching rows in **one** SQL `UPDATE` — fast. But it does **not** call each object's `.save()`, so fields like `updated_at` (`auto_now`) are **not** updated. Use `.save()` for one object, `.update()` for bulk changes where that doesn't matter.

**2. Transactions — "all or nothing"**

Some jobs need several database changes that must **all** succeed together. Example: moving money — take 100 from Asha **and** give 100 to Rahul. If the computer crashes in the middle, money would disappear!

A **transaction** groups changes so that either **all** of them are saved, or **none** are (the database **rolls back**):

```python
from django.db import transaction

with transaction.atomic():
    asha.balance -= 100
    asha.save()
    rahul.balance += 100
    rahul.save()
# If any line inside raises an error, BOTH saves are undone.
```

You don't need this for today's website (each page saves one thing), but you'll need it in bigger projects like AI WarRoom (Day 5).

---

## Try it yourself

In the shell:

1. Create a category called `Web` with the description `HTML, CSS and JS`.
2. Find all articles that are **not** published. *(Hint: `is_published=False`)*
3. Find all articles whose author's username is `admin`. *(Hint: `author__username`)*
4. Count how many articles are in the `Django` category.
5. Delete the `Web` category.

<details>
<summary>Answers</summary>

```python
web = Category.objects.create(name="Web", description="HTML, CSS and JS")
Article.objects.filter(is_published=False)
Article.objects.filter(author__username="admin")
Category.objects.get(name="Django").articles.count()
web.delete()
```
</details>

**Want more practice?** Open `notebooks/02_Django_ORM_Practice.ipynb` (see `notebooks/README.md`).

---

## If it does not work

| Problem | Fix |
|---|---|
| `NameError: name 'Article' is not defined` | Run the imports in Step 2 again (they're lost when you exit the shell). |
| `DoesNotExist` for `"My first article"` | Your title is different. Run `Article.objects.all()` and copy a real title. |
| `IntegrityError: duplicate key ... name` | That category name already exists (`unique=True`). |
| `ProtectedError` when deleting a user | The user still has articles (`PROTECT`). Delete or reassign the articles first. |
| Typing got messy | Press **Ctrl + C** to cancel the line, or `exit()` and start the shell again. |

---

**Next:** [11_SHOW_ARTICLES_ON_YOUR_WEBSITE.md](11_SHOW_ARTICLES_ON_YOUR_WEBSITE.md)
