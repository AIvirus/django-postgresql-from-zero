# Lesson 07 — Models and Migrations (Python Classes → Database Tables)

**Time:** 30 minutes  |  **You need:** Lesson 06 finished (Django connected to Neon)

---

## Goal

By the end of this lesson you will:

- describe an **Article** and a **Category** in Python (a **model**),
- turn that description into **real tables** in PostgreSQL (a **migration**),
- see the new tables in Neon,
- see the actual **SQL** Django wrote for you.

---

## Understand

### What is a model?

A **model** is a Python **class** that describes **one kind of thing** you want to save.

- The class = the **table**.
- Each attribute (`title = ...`) = a **column**.
- Each object you save = one **row**.

```python
class Article(models.Model):          # → table "journal_article"
    title = models.CharField(max_length=160)   # → column "title" (short text)
    body = models.TextField()                  # → column "body" (long text)
```

You never write `CREATE TABLE` yourself. Django does it.

### What is a migration?

Think of building a house:

| House | Django |
|---|---|
| The **drawing** of the house | The **model** (`models.py`) |
| The **building instructions** ("add a room here") | The **migration file** |
| The **actual house** | The **table** in PostgreSQL |

Two commands — always in this order:

| Command | Meaning |
|---|---|
| `python manage.py makemigrations` | "Look at my models and **write** the building instructions." (does NOT touch the database) |
| `python manage.py migrate` | "**Follow** the instructions and build/change the tables in the database." |

### The common field types

| Field | Stores | Example |
|---|---|---|
| `CharField(max_length=...)` | Short text (needs a max length) | Title, name |
| `TextField()` | Long text | Article body |
| `BooleanField(default=False)` | True / False | Is it published? |
| `DateTimeField(auto_now_add=True)` | Date + time, set **once** when created | Created at |
| `DateTimeField(auto_now=True)` | Date + time, updated **every save** | Updated at |
| `IntegerField`, `PositiveIntegerField` | Whole numbers | Reading time |
| `DecimalField` | Exact decimals | Price (never use float for money!) |
| `EmailField`, `URLField` | Text that must look like an email / link | Contact email |
| `ForeignKey`, `ManyToManyField`, `OneToOneField` | Links to other tables | Lesson 09 explains these |

Useful options:

| Option | Meaning |
|---|---|
| `blank=True` | The field may be left **empty** in forms |
| `null=True` | The database may store **NULL** ("no value at all") in this column |
| `default=...` | Value used if nothing is given |
| `unique=True` | No two rows can have the same value |
| `db_index=True` | Ask the database to build an **index** on this column (see below) |

**`blank` vs `null` — they are different:**

- `blank` is about **forms**: "may the user leave this empty?"
- `null` is about the **database**: "may this column contain NULL?"
- For **text** fields, use only `blank=True`. An empty text is stored as `""` (empty string), so you don't need NULL. (That's what we do for `subtitle`.)
- For **non-text** fields that may be missing (a number, a date, a ForeignKey), use `null=True, blank=True` together.

### Constraints — rules the database itself enforces

A **constraint** is a rule that PostgreSQL checks on every save — even if the data comes from Admin, the shell, another app or an API, not just your form.

| Constraint | You create it with | It stops… |
|---|---|---|
| Not empty | any field without `null=True` | saving a row with a missing value |
| Unique | `unique=True` | two categories both called "Django" |
| Foreign key | `ForeignKey(...)` | an article pointing to a user who doesn't exist |
| Check | `models.CheckConstraint(...)` in `Meta` | e.g. a negative price |

Rule of thumb: **important rules belong in the database**, not only in the browser or the form.

### Indexes — the index at the back of a textbook

An **index** helps the database find rows fast without reading every row, like the index at the back of a book. Django automatically creates indexes for `id`, `unique` fields and ForeignKeys. You can add more with `db_index=True`. But indexes take space and slow down saving a little, so add them only for columns you really search or sort on a lot.

---

## Do it now

> Stop the server (**Ctrl + C**) for this lesson — or use a second terminal.

### Step 1 — Write the models

Open **`journal/models.py`**. Replace everything with:

```python
from django.conf import settings
from django.db import models


class Category(models.Model):
    name = models.CharField(max_length=60, unique=True)
    description = models.CharField(max_length=160, blank=True)

    class Meta:
        ordering = ["name"]
        verbose_name_plural = "categories"

    def __str__(self):
        return self.name


class Article(models.Model):
    """One-to-Many: many articles can belong to one author.
    Many-to-Many: one article can have many categories.
    """

    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.PROTECT,
        related_name="articles",
    )
    title = models.CharField(max_length=160)
    subtitle = models.CharField(max_length=200, blank=True)
    body = models.TextField()
    categories = models.ManyToManyField(
        Category,
        blank=True,
        related_name="articles",
    )
    is_published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["-created_at"]

    def __str__(self):
        return self.title
```

Save.

### Step 2 — Read your model line by line

| Line | Meaning in plain English |
|---|---|
| `class Category(models.Model)` | "Make a table for categories." |
| `name = CharField(max_length=60, unique=True)` | Each category has a name, max 60 letters, **no duplicates** (only one "Django" category). |
| `description = CharField(..., blank=True)` | Optional short description. |
| `class Meta: ordering = ["name"]` | When we list categories, sort them A→Z. |
| `verbose_name_plural = "categories"` | Otherwise Django would write "Categorys" |
| `def __str__` | How the object is shown as text (e.g. in Admin). Shows the name instead of "Category object (1)". |
| `author = ForeignKey(settings.AUTH_USER_MODEL, ...)` | Each article belongs to **one user** (Django's built-in User table). |
| `on_delete=models.PROTECT` | Don't allow deleting a user who still has articles. |
| `related_name="articles"` | Lets us later write `some_user.articles.all()` to get a user's articles. |
| `categories = ManyToManyField(Category, blank=True)` | An article can have **many** categories (or none). |
| `is_published = BooleanField(default=False)` | New articles start as **drafts**. |
| `created_at = DateTimeField(auto_now_add=True)` | Filled in automatically when the article is created. |
| `updated_at = DateTimeField(auto_now=True)` | Updated automatically every time it is saved. |
| `ordering = ["-created_at"]` | Newest first (the minus `-` means "reverse"). |

> Notice: we didn't write an `id` field. **Django adds `id` automatically** to every model — the unique number for each row (the **primary key**).

### Step 3 — Make the migration (write the instructions)

```bash
python manage.py makemigrations journal
```

Expected:

```text
Migrations for 'journal':
  journal\migrations\0001_initial.py
    + Create model Category
    + Create model Article
```

### Step 4 — Look at the migration file

Open **`journal/migrations/0001_initial.py`**. You will see `migrations.CreateModel(name="Category", ...)` and `migrations.CreateModel(name="Article", ...)` with all your fields. **This is the instruction list.** Don't edit it — just read it.

### Step 5 — See the real SQL (optional but cool)

```bash
python manage.py sqlmigrate journal 0001
```

This prints the **SQL** that Django will send to PostgreSQL — things like `CREATE TABLE "journal_article" (...)`. You didn't have to write any of it!

### Step 6 — Apply the migration (build the tables)

```bash
python manage.py migrate
```

Expected:

```text
Running migrations:
  Applying journal.0001_initial... OK
```

### Step 7 — See the new tables in Neon

Neon dashboard → **Tables**. Refresh. You should now see **three** new tables:

| Table | What it is |
|---|---|
| `journal_category` | Your categories |
| `journal_article` | Your articles (look: it has an `author_id` column!) |
| `journal_article_categories` | A "linking" table — which article has which category (Lesson 09) |

Click `journal_article` to see its columns.

---

## Check

- [ ] `journal/migrations/0001_initial.py` exists.
- [ ] `migrate` printed **`Applying journal.0001_initial... OK`**.
- [ ] Neon shows `journal_category`, `journal_article`, `journal_article_categories`.
- [ ] Run `python manage.py showmigrations journal` → you see `[X] 0001_initial` (the **X** means "applied").

---

## What just happened?

```text
models.py  ──makemigrations──▶  0001_initial.py  ──migrate──▶  tables in PostgreSQL (Neon)
(drawing)                       (instructions)                  (the real house)
```

- Table names are **`appname_modelname`** in lowercase: `journal` + `article` → `journal_article`.
- The `author` field became a column called **`author_id`** — it stores the **id** of a user.
- `django_migrations` (Django's diary) got a new row saying `journal 0001_initial` is done, so Django never applies it twice.

### Golden rules of migrations

1. **Changed a model? → `makemigrations` → `migrate`.** Every time.
2. **Never delete migration files** to "fix" an error. They are part of your code.
3. When you share code, share migration files too — teammates need them to build the same tables. Commit the model change **and** its migration together.
4. Always **read** a new migration file before running `migrate` — check it does what you expect.
5. Check **which database** your `.env` points to before `migrate`. Running it on the wrong database changes the wrong database.

**What if two people change models at the same time?** Both make a migration with the same number (e.g. two `0002_...` files). Django notices and asks you to run `python manage.py makemigrations --merge`, which creates a small migration that joins them. Only do this after you understand both changes.

---

## Try it yourself

Practise the change-a-model cycle:

1. Add one field to `Article` in `models.py` (under `body`):
   ```python
   reading_minutes = models.PositiveIntegerField(default=1)
   ```
2. Run `python manage.py makemigrations journal` → you get `0002_article_reading_minutes.py`.
3. Run `python manage.py migrate`.
4. Check in Neon → `journal_article` now has a `reading_minutes` column.
5. Now **remove** that field again from `models.py`, and run `makemigrations` + `migrate` again. Django makes `0003_remove_...` and removes the column.

> Why `default=1`? The table might already have rows. A new column needs **some** value for old rows. Without a default, Django will ask you a question in the terminal.

---

## If it does not work

| Problem | Fix |
|---|---|
| `No changes detected` | File not saved, or `"journal"` not in `INSTALLED_APPS`. Use `makemigrations journal` (with the app name). |
| `SystemCheckError ... fields.E...` | A typo in a field. Read the message: it names the field. E.g. `CharField` **needs** `max_length`. |
| Django asks *"Please select a fix: 1) Provide a one-off default..."* | You added a required field without a `default`. Type `2` to quit, add `default=...` to the field, try again. |
| Tables don't appear in Neon | Click refresh. Make sure you're looking at database **neondb**, schema **public**, branch **main**. |
| `relation "journal_article" does not exist` (later) | You forgot `migrate`. Run it. |

Compare with the answer: `django_lab/journal/models.py` (the answer also has a `get_absolute_url` method — you will add it in Lesson 11).

---

**Next:** [08_DJANGO_ADMIN.md](08_DJANGO_ADMIN.md)
