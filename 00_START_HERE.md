# START HERE — Day 2: Build a Website with Django and a Real Database

Welcome! Today **you will build a real website yourself**, step by step.

The website is called **Learning Journal**. People can:

- register and log in,
- write articles,
- tag articles with categories,
- search articles,
- edit or delete **only their own** articles,
- and the data is saved in a real **PostgreSQL** database on the internet.

You do **not** need to know Django before you start. You only need basic Python (variables, functions, `if`, lists, dictionaries).

---

## How every lesson works

Every lesson has the same parts, in the same order. Do them in this order:

| Part | What you do |
|---|---|
| **Goal** | Read what you will be able to do at the end. |
| **Understand** | Read a short, simple explanation. |
| **Do it now** | Type the commands and code **exactly** as shown. |
| **Check** | Compare your screen with what you *should* see. |
| **What just happened?** | Read what your code actually did. |
| **Try it yourself** | A small challenge, so you really learn it. |
| **If it does not work** | The most common mistakes and how to fix them. |

> **Golden rule:** Do not move to the next lesson until the **Check** of the current lesson works.

---

## Your learning path (do these in order)

| # | Lesson | What you build / do | Time |
|---|---|---|---|
| 01 | [How websites work](01_HOW_WEBSITES_WORK.md) | See real requests in your browser | 20 min |
| 02 | [Set up your computer](02_SET_UP_YOUR_COMPUTER.md) | Python + virtual environment + install packages | 20 min |
| 03 | [Your first Django project](03_YOUR_FIRST_DJANGO_PROJECT.md) | Start Django and see the rocket page | 15 min |
| 04 | [Your first app and page](04_YOUR_FIRST_APP_AND_PAGE.md) | Make your own page say "Hello" | 25 min |
| 05 | [Templates, CSS, JavaScript, images](05_TEMPLATES_CSS_JS_IMAGES.md) | Make a real-looking home page | 30 min |
| 06 | [Create a PostgreSQL database and connect it](06_CREATE_POSTGRESQL_DATABASE_AND_CONNECT.md) | Free Neon database + secret key + `.env` | 40 min |
| 07 | [Models and migrations](07_MODELS_AND_MIGRATIONS.md) | Create the Article and Category tables | 30 min |
| 08 | [Django Admin](08_DJANGO_ADMIN.md) | Add data using a ready-made admin website | 20 min |
| 09 | [Relationships between tables](09_RELATIONSHIPS_BETWEEN_TABLES.md) | One-to-One, One-to-Many, Many-to-Many | 30 min |
| 10 | [Talk to the database with Python (ORM)](10_TALK_TO_THE_DATABASE_ORM.md) | Create, find, change, delete rows from the shell | 30 min |
| 11 | [Show articles on your website](11_SHOW_ARTICLES_ON_YOUR_WEBSITE.md) | Article list, search, and detail pages | 40 min |
| 12 | [Register, log in, log out](12_REGISTER_LOGIN_LOGOUT.md) | User accounts | 35 min |
| 13 | [Forms: create, edit, delete](13_FORMS_CREATE_EDIT_DELETE.md) | Users write and manage their own articles | 45 min |
| 14 | [Profiles and roles](14_PROFILES_AND_ROLES.md) | Profile page + "editors only" page | 30 min |
| 15 | [Automated tests](15_AUTOMATED_TESTS.md) | Let Python check your website for you | 30 min |
| 16 | [Look inside PostgreSQL with SQL](16_LOOK_INSIDE_POSTGRESQL_WITH_SQL.md) | Read your tables with SQL | 20 min |
| 17 | [Put your website on the internet (optional)](17_DEPLOY_YOUR_WEBSITE_OPTIONAL.md) | Deploy to Render + Neon | 45 min |
| 18 | [Final checklist and what's next](18_FINAL_CHECKLIST_AND_WHATS_NEXT.md) | Check everything you learned | 15 min |

Total: about 8–9 hours. It is fine to split it over two days.

---

## What is in this folder?

```text
Day-02-Django/
├── 00_START_HERE.md          ← you are here
├── 01_ ... 18_ lessons       ← read and do these in order
├── requirements.txt          ← list of Python packages to install (Lesson 02)
├── my_journal/               ← YOU will create this folder in Lesson 03 (your own project)
├── django_lab/               ← the FINISHED website (answer key — use it when you are stuck)
├── practice/                 ← extra questions + answers for revision
├── references/               ← cheat sheet, common errors, glossary, SQL practice
└── notebooks/                ← optional extra practice (SQL and ORM in Jupyter)
```

### `my_journal/` vs `django_lab/` — important!

- **`my_journal/`** is **your** project. You build it yourself, lesson by lesson.
- **`django_lab/`** is the **finished answer key**. When something in your project does not work, open the same file in `django_lab/` and compare.
  - Example: your `my_journal/journal/views.py` ↔ the answer `django_lab/journal/views.py`.
- Do **not** just copy the whole `django_lab` folder. You learn by typing and running.

> Want to see the finished website first, before you start? Follow [django_lab/README.md](django_lab/README.md). It takes about 15 minutes after you finish Lesson 06 (you need a database first).

---

## What you need before Lesson 01

- A laptop with **Windows, macOS, or Linux**.
- **Python 3.11, 3.12 or 3.13** (Lesson 02 shows you how to check).
- **VS Code** — free editor: https://code.visualstudio.com/
- **Internet connection.**
- An **email address** (to create a free Neon database account in Lesson 06).
- *(Optional, Lesson 17 only)* a GitHub account and a Render account.

---

## When you are stuck

1. Read the **If it does not work** part at the bottom of the lesson.
2. Read [references/COMMON_ERRORS.md](references/COMMON_ERRORS.md).
3. Compare your file with the same file inside `django_lab/`.
4. Read the **last line** of the red error message first — it usually tells you exactly what is wrong.
5. Do not know a word? Look in [references/GLOSSARY.md](references/GLOSSARY.md).

Never fix an error by deleting your database or your migrations folder. That usually makes things worse.

Ready? Open **[01_HOW_WEBSITES_WORK.md](01_HOW_WEBSITES_WORK.md)**
