# Lesson 06 — Create a PostgreSQL Database and Connect Django

**Time:** 40 minutes  |  **You need:** Lesson 05 finished, an email address, internet

This is the **most important setup lesson** of the day. Go slowly and do every step.

---

## Goal

By the end of this lesson:

- you have a **free PostgreSQL database** on the internet (Neon),
- you have a **secret key** generated on your own computer,
- both are stored safely in a private **`.env`** file,
- Django is **connected** to PostgreSQL and has created its first tables.

---

## Understand

### What is PostgreSQL? What is Neon?

- **PostgreSQL** (say "post-gress") is a database program used by many big companies. It stores data in tables.
- **Neon** is a website that runs PostgreSQL **for you** on the internet, for free. You don't need to install a database on your laptop.

```text
Your laptop                                  The internet
┌──────────────────────┐   connection     ┌─────────────────────┐
│ Django (my_journal)  │ ───────────────▶ │ Neon: PostgreSQL DB │
└──────────────────────┘   (encrypted)    └─────────────────────┘
```

### What does Django need to connect?

Like logging into a website, Django needs: **where** the database is (host), **which** database (name), **who** is logging in (user) and the **password**. All of that fits in **one line**, called the **connection string** or **`DATABASE_URL`**:

```text
postgresql://neondb_owner:AbC123secret@ep-cool-name-123.ap-southeast-1.aws.neon.tech/neondb?sslmode=require
            └────┬─────┘ └────┬─────┘ └──────────────────┬─────────────────────┘ └──┬─┘ └──────┬──────┘
               user        password                      host                       name   use encryption
```

(This example is fake. Yours will be different.)

PostgreSQL usually uses **port 5432** (the "door number" on the database server). If it isn't written in the string, 5432 is used.

### Server, database, role, table — four different things

| Word | Meaning | In Neon |
|---|---|---|
| **Database server** | The running PostgreSQL program | Your Neon project |
| **Database** | A named collection of tables inside the server | `neondb` |
| **Role** (database user) | An account that is allowed to connect, with a password and permissions | `neondb_owner` |
| **Table** | Rows and columns of one kind of data | `auth_user`, `journal_article`… |

One server can hold many databases, and each database many tables.

### Why PostgreSQL and not SQLite?

Django starts with **SQLite** — a tiny database stored in one file (`db.sqlite3`). It's great for quick experiments (the notebooks use it). **PostgreSQL** is a real database server built for many users at once, and it's what companies use in production. We use PostgreSQL from now on so that your laptop and the live website (Lesson 17) use the **same** kind of database.

### What is a SECRET_KEY?

Django uses a **secret key** to sign things like login cookies, so nobody can fake them. It is like the **stamp of your school** — if someone copies it, they can make fake documents. So:

- every project gets its **own** random secret key,
- it must **never** be shared or uploaded to GitHub.

### What is `.env`? Why not just write passwords in `settings.py`?

`settings.py` is **code**. Code gets shared — with classmates, on GitHub, in screenshots. Passwords must **never** be inside code.

So we put secrets in a separate private file called **`.env`**, and `settings.py` **reads** them from there.

```text
.env  (PRIVATE — never share)          settings.py  (code — safe to share)
DATABASE_URL=postgresql://...    ───▶   DATABASE_URL = os.environ.get("DATABASE_URL")
DJANGO_SECRET_KEY=k3j$9...       ───▶   SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY")
```

The values in `.env` are called **environment variables**: settings given to a program from **outside** its code. This lets the **same code** run in different places with different settings:

| Environment | What it is | Its settings |
|---|---|---|
| **Local / development** | Your laptop | `DEBUG=True`, your practice database |
| **Testing** | Where automated tests run | A temporary test database (Lesson 15) |
| **Staging** | A private copy of the live site, for a final check | Its own database and secrets |
| **Production** | The real live website used by real people | `DEBUG=False`, its own database and secrets (Lesson 17) |

Only the `.env` values change — the Python code stays the same.

### The four settings in our `.env`

| Setting | What it controls |
|---|---|
| `DATABASE_URL` | Which database to connect to |
| `DJANGO_SECRET_KEY` | The secret used to sign cookies |
| `DJANGO_DEBUG` | `True` = show detailed yellow error pages (only safe on your laptop). On a live site it must be `False`, or strangers could see your code and settings in error pages |
| `DJANGO_ALLOWED_HOSTS` | Which web addresses this site may be served on (`127.0.0.1` and `localhost` on your laptop; your real domain when live) |

---

## Do it now

### Part A — Create your free Neon database

1. Open **https://neon.com/** and click **Sign up**. Sign up with Google, GitHub or email. Finish any email verification.
2. Neon asks you to **create a project**. Fill in:
   - **Project name:** `day2-django`
   - **Postgres version:** leave the default.
   - **Region:** choose the one closest to you (for India: **Asia Pacific (Singapore)**).
3. Click **Create project**. Wait a few seconds.
4. On the project dashboard, click the **Connect** button.
5. In the Connect window:
   - **Branch:** `main` (default) · **Database:** `neondb` (default) · **Role:** `neondb_owner` (default)
   - Find the **Connection pooling** switch and turn it **OFF**. (We want the "direct" connection. It works best with Django's `migrate` command.)
   - Make sure the format shown is the **connection string** (it starts with `postgresql://`).
   - Click **Show password**, then click **Copy snippet** / the copy icon.
6. Paste it somewhere temporary (e.g. Notepad) for a minute. It looks like:
   ```text
   postgresql://neondb_owner:npg_XXXXXXXX@ep-xxxx-xxxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require
   ```

> This line contains your **password**. Never post it in a chat, screenshot or GitHub.
> Neon's buttons may move a little over time — look for **Connect** and **connection string**.

### Part B — Generate your secret key

In the VS Code terminal (with `(.venv)` active, inside `my_journal`), run:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

It prints one long random line, for example (yours will be different!):

```text
x8m!p2@v#k9q(3r_t5z-w7e)u1y=b4n^c6j&h0s*f%l+d$g!a2o
```

Copy **your** line. Do **not** use the example above.

> **Why not just type random letters?** People are bad at being random. Django's tool makes a truly random key of 50 characters.
>
> *Other way (also fine):* `python -c "import secrets; print(secrets.token_urlsafe(50))"`

### Part C — Create the private `.env` file

1. In the VS Code Explorer, right-click the **`my_journal`** folder (the one with `manage.py`) → **New File**.
2. Name it exactly **`.env`** (dot env, nothing else — **not** `.env.txt`).
3. Paste this, then replace the two values with **your** connection string and **your** secret key:

```text
DATABASE_URL=postgresql://PASTE-YOUR-NEON-CONNECTION-STRING-HERE
DJANGO_SECRET_KEY=PASTE-YOUR-GENERATED-SECRET-KEY-HERE
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
```

Rules for `.env`:
- **No spaces** around `=`.
- **No quotes** around values.
- One setting per line.

Save. Your folder now looks like:

```text
my_journal/
├── .env          ← NEW (private!)
├── manage.py
├── config/
└── journal/
```

### Part D — Create `.gitignore` (protects `.env` from being uploaded)

Create another new file in `my_journal` called **`.gitignore`** with:

```text
.env
.venv/
__pycache__/
*.pyc
db.sqlite3
staticfiles/
```

Git (Day 3) will now **ignore** these files, so your `.env` never goes to GitHub.

### Part E — Create `.env.example` (a safe template for others)

Create one more file in `my_journal` called **`.env.example`**:

```text
# Copy this file to .env and put your own values in it.
DATABASE_URL=postgresql://USER:PASSWORD@HOST/DBNAME?sslmode=require
DJANGO_SECRET_KEY=generate-your-own-secret-key
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
```

This file has **no real secrets**. It shows a teammate **which** settings they need.

### Part F — Make `settings.py` read the `.env` file

Open **`config/settings.py`**.

**Change 1.** Near the top, find this part (your secret key text will be different):

```python
from pathlib import Path

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/5.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'django-insecure-.....'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []
```

**Delete all of it** and put this in its place:

```python
import os
from pathlib import Path

import dj_database_url
from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent

# Read the private .env file that sits next to manage.py
load_dotenv(BASE_DIR / ".env")

SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY")
if not SECRET_KEY:
    raise RuntimeError("DJANGO_SECRET_KEY is missing. Put it in your .env file.")

DEBUG = os.environ.get("DJANGO_DEBUG", "False").lower() == "true"

ALLOWED_HOSTS = [
    host.strip()
    for host in os.environ.get("DJANGO_ALLOWED_HOSTS", "127.0.0.1,localhost").split(",")
    if host.strip()
]
```

**Change 2.** Further down, find the database part:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

**Delete it** and put this in its place:

```python
DATABASE_URL = os.environ.get("DATABASE_URL")
if not DATABASE_URL:
    raise RuntimeError("DATABASE_URL is missing. Put your Neon connection string in .env.")

DATABASES = {
    "default": dj_database_url.parse(DATABASE_URL, conn_max_age=0, ssl_require=True)
}
```

**Change 3 (optional).** Find `TIME_ZONE = 'UTC'` and change it to your time zone, e.g. `TIME_ZONE = "Asia/Kolkata"`.

Save the file.

### Part G — Delete the old practice database

If there is a file called **`db.sqlite3`** in `my_journal`, delete it (right-click → **Delete**). We use PostgreSQL now.

### Part H — Test the connection

Stop the server if it is running (**Ctrl + C**). Then:

```bash
python manage.py check
```

Expected: `System check identified no issues (0 silenced).`

Now create Django's built-in tables **inside your Neon database**:

```bash
python manage.py migrate
```

Expected (it takes a few seconds, because it talks to the internet):

```text
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  ...
  Applying sessions.0001_initial... OK
```

### Part I — See your tables in Neon

1. Go back to the Neon dashboard → your project.
2. Click **Tables** in the left menu (database `neondb`, schema `public`).
3. You should see about **10 new tables**, like `auth_user`, `auth_group`, `django_migrations`, `django_session`.

### Part J — Start the server again

```bash
python manage.py runserver
```

Open http://127.0.0.1:8000/ — your home page still works, and the **"unapplied migrations"** warning is **gone**.

---

## Check

- [ ] `python manage.py check` says **no issues**.
- [ ] `python manage.py migrate` printed lines ending in **OK**.
- [ ] Neon → **Tables** shows `auth_user`, `django_migrations`, etc.
- [ ] `runserver` no longer warns about unapplied migrations.
- [ ] Your `.env` file is **not** inside `config/` — it is next to `manage.py`.

---

## What just happened?

```text
1. load_dotenv() opened my_journal/.env and loaded its lines as "environment variables".
2. os.environ.get("DJANGO_SECRET_KEY") read your secret key.
3. os.environ.get("DATABASE_URL") read your connection string.
4. dj_database_url.parse(...) turned that one line into the settings Django needs
   (host, user, password, database name, encryption).
5. psycopg (installed in Lesson 02) carried the commands to Neon over the internet.
6. migrate created Django's built-in tables (users, groups, sessions...) in PostgreSQL.
```

- **`check`** = "Is my project set up correctly?" (doesn't change anything)
- **`migrate`** = "Build the tables in the database." (changes the database)
- `ssl_require=True` makes sure the connection is **encrypted**, so nobody on the Wi-Fi can read your data.
- If the `.env` value is missing, our `RuntimeError` gives a **clear message** instead of a confusing crash.

### Security rules (learn these by heart)

1. **Never** put passwords or secret keys inside `.py` files.
2. **Never** upload `.env` to GitHub (that's why we made `.gitignore`).
3. **Never** show `.env` in screenshots or chats.
4. Use a **different** secret key and database for your laptop and for the internet version (Lesson 17).
5. If a secret leaks: make a **new** one (Neon: **Roles → Reset password**; secret key: run the command again) and update `.env`.

---

## Try it yourself

1. **Break it on purpose:** in `.env`, rename `DATABASE_URL` to `DATABASE_URLX`. Save. Run `python manage.py check`. Read the error — it's **our** message from settings! Fix it back. Run `check` again.
2. In Neon, open **SQL Editor** and run:
   ```sql
   SELECT app, name FROM django_migrations;
   ```
   This is Django's **diary** of which migrations it already applied.
3. Answer: Why do we commit `.env.example` but never `.env`?

---

## If it does not work

| Error | Fix |
|---|---|
| `DATABASE_URL is missing` / `DJANGO_SECRET_KEY is missing` | `.env` is in the wrong folder, has the wrong name (`.env.txt`), or the variable name is misspelled. It must be **next to `manage.py`**. In VS Code Explorer it must show as `.env` exactly. |
| `ModuleNotFoundError: No module named 'dj_database_url'` (or `dotenv`) | `.venv` is not active, or Lesson 02 Step 7 was skipped. Activate, then from `Day-02-Django` run `pip install -r requirements.txt`. |
| `password authentication failed` | Copy the connection string again (click **Show password** first). Or reset the password in Neon (**Roles**) and copy the new string. |
| `could not translate host name` | Check your internet. Make sure you copied the **whole** string, no spaces. |
| `connection ... timeout` | Your Neon database may be "sleeping" — run the command again. Some school/office Wi-Fi blocks port 5432 — try a phone hotspot. |
| `channel_binding` error | Remove `&channel_binding=require` from the end of `DATABASE_URL` (keep `?sslmode=require`). |
| `RuntimeError` mentions settings when running `migrate` | Read the message carefully — it tells you which `.env` value is missing. |

Compare with the answer: `django_lab/config/settings.py` and `django_lab/.env.example`.

---

**Next:** [07_MODELS_AND_MIGRATIONS.md](07_MODELS_AND_MIGRATIONS.md)
