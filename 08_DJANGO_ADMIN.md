# Lesson 08 — Django Admin: A Free Control Panel for Your Data

**Time:** 20 minutes  |  **You need:** Lesson 07 finished (tables exist in Neon)

---

## Goal

By the end of this lesson you will:

- create a **superuser** (the boss account),
- log into Django's built-in **Admin** website,
- add **categories** and **articles** with clicks,
- see those rows appear in **PostgreSQL**.

---

## Understand

### What is Django Admin?

Django gives you a **ready-made control panel** for your data, for free. Like the staff room of a school: only staff can enter, and they can add, edit and delete records.

- It is for **staff only**, not for normal visitors.
- It shows a model **only if you register it** in `admin.py`.

### What is a superuser?

A **superuser** is an account that is allowed to do **everything** in Admin. You make the first one from the terminal.

---

## Do it now

### Step 1 — Create your superuser

```bash
python manage.py createsuperuser
```

Answer the questions:

```text
Username: admin
Email address: (you can press Enter to skip)
Password: ********
Password (again): ********
Superuser created successfully.
```

> **The password is invisible while you type.** That's normal! Just type it and press Enter.
> If Django says the password is "too common" or "too short", choose a stronger one (it's good practice).
> Remember this username and password — you'll need them all day.

### Step 2 — Register your models in Admin

Open **`journal/admin.py`**. Replace everything with:

```python
from django.contrib import admin

from .models import Article, Category


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ("name", "description")
    search_fields = ("name",)


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ("title", "author", "is_published", "created_at")
    list_filter = ("is_published", "categories", "created_at")
    search_fields = ("title", "subtitle", "body", "author__username")
    filter_horizontal = ("categories",)
    readonly_fields = ("created_at", "updated_at")
```

Save.

### Step 3 — Open Admin

```bash
python manage.py runserver
```

Open **http://127.0.0.1:8000/admin/** and log in with your superuser.

You see sections **AUTHENTICATION AND AUTHORIZATION** (Groups, Users) and **JOURNAL** (Articles, Categories).

### Step 4 — Add categories

1. Click **Categories → + Add**.
2. Name: `Django` → **Save and add another**.
3. Name: `Databases` → **Save and add another**.
4. Name: `Python` → **Save**.

You see 3 categories, sorted A→Z (that's your `ordering = ["name"]`!).

5. Try adding `Django` **again**. Django says *"Category with this Name already exists."* That's your `unique=True` protecting the data.

### Step 5 — Add articles

1. Click **Articles → + Add**.
2. Fill in:
   - **Author:** choose `admin`
   - **Title:** `My first article`
   - **Body:** `Django makes building websites much easier.`
   - **Categories:** move `Django` and `Python` to the right side (click them, then the arrow)
   - tick **Is published**
3. **Save**.
4. Add a second article: title `A secret draft`, any body, **do not** tick *Is published*. Save.

### Step 6 — Play with the list

On the Articles list page:
- Use the **search box** — search `secret`.
- Use the **FILTER** on the right — click *Is published: No*.
- Notice the columns: Title, Author, Is published, Created at — that's your `list_display`.

### Step 7 — Find your rows in PostgreSQL

Go to Neon → **SQL Editor** and run:

```sql
SELECT id, title, author_id, is_published FROM journal_article;
```

Your two articles are there, saved in the real database!

---

## Check

- [ ] You can log into http://127.0.0.1:8000/admin/
- [ ] You have **3 categories** and **2 articles** (1 published, 1 draft).
- [ ] The SQL query in Neon shows your articles.

---

## What just happened?

- `createsuperuser` added a row to the **`auth_user`** table with `is_superuser = true`. The password was **not** saved as plain text — it was turned into a scrambled **hash** (you'll look at it in Lesson 12).
- `@admin.register(Article)` told Admin: "show this model".
- The `ModelAdmin` options decide **how** it looks:

| Option | Does |
|---|---|
| `list_display` | Columns in the list |
| `search_fields` | What the search box searches (`author__username` = "the username of the author" — the double underscore `__` means *follow the link*) |
| `list_filter` | Filters on the right |
| `filter_horizontal` | The nice two-box picker for many-to-many |
| `readonly_fields` | Shown but not editable |

- When you clicked **Save**, Admin ran an `INSERT` into `journal_article` for you.

**Admin is for trusted staff only.** Normal users will use the pages **you** build (Lessons 11–14). Never make every user a superuser.

---

## Try it yourself

1. Add `"is_published"` to `list_editable` so you can tick it right in the list:
   ```python
   list_editable = ("is_published",)
   ```
   (Put it in `ArticleAdmin` under `list_display`.) Refresh the Admin list. Publish your draft from the list.
2. Edit an article and look at **Created at** and **Updated at**. Save it again — which one changes? (Answer: only *Updated at*, because of `auto_now`.)
3. Go to **Users** → click `admin`. Scroll to **Password**: it says `algorithm: pbkdf2_sha256 ...` — no readable password stored anywhere!

---

## If it does not work

| Problem | Fix |
|---|---|
| Admin page has **no styling** | Make sure `"django.contrib.staticfiles"` is in `INSTALLED_APPS` and `DEBUG=True` in `.env`. Restart the server. |
| "Please enter the correct username and password" | Create another superuser with `createsuperuser`, or reset: `python manage.py changepassword admin` |
| **Journal** section doesn't appear | `admin.py` not saved, or a typo in the import. Check the terminal for a red error. |
| `ProgrammingError: relation "journal_category" does not exist` | You skipped `migrate` in Lesson 07. Run `python manage.py migrate`. |

Compare with the answer: `django_lab/journal/admin.py`.

---

**Next:** [09_RELATIONSHIPS_BETWEEN_TABLES.md](09_RELATIONSHIPS_BETWEEN_TABLES.md)
