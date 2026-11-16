# Lesson 03 — Your First Django Project

**Time:** 15 minutes  |  **You need:** Lesson 02 finished (`(.venv)` is active)

---

## Goal

By the end of this lesson you will:

- create your own Django project called **`my_journal`**,
- start the Django server,
- open your website in the browser and see the rocket page.

---

## Understand

### What is a framework?

Building a website from zero is like building a house by first making your own bricks. A **framework** gives you ready-made bricks, doors and wiring. You still design the house, but you don't start from nothing.

**Django** is a Python framework for building websites. It already knows how to: handle URLs, show HTML pages, talk to databases, log users in, protect forms, and much more.

What Django gives you ready-made:

| Tool | Lesson where you use it |
|---|---|
| URL routing (which address runs which code) | 04 |
| HTML templates | 05 |
| Models, ORM and migrations (database work in Python) | 07, 10 |
| Admin control panel | 08 |
| Forms and validation | 13 |
| Login, logout, password hashing, sessions | 12 |
| Groups and permissions | 14 |
| Automated tests | 15 |
| Security protections (CSRF, safe templates, SQL-injection protection) | 05, 10, 12 |

Django is popular for content websites, company internal tools, portals and backend services. Instagram and Mozilla have used it.

What Django does **not** do for you: it doesn't replace knowing Python, HTML/CSS, SQL, or good database design, and it doesn't make your app secure automatically — **you** still have to write the right permission checks (Lessons 13 and 14).

### Project vs app

- A **project** = the **whole website** (like a whole school).
- An **app** = **one feature area** inside it (like one classroom). You will create apps in the next lesson.

Our project's settings folder will be called **`config`** (because it holds the configuration).

---

## Do it now

> Check first: your terminal line starts with **`(.venv)`** and ends with **`Day-02-Django>`**. If not, go back to Lesson 02, Step 6.

### Step 1 — Create a folder for your project and go into it

```bash
mkdir my_journal
```

```bash
cd my_journal
```

The terminal line now ends with **`my_journal>`**.

### Step 2 — Ask Django to create the project

Type this **exactly** — including the **dot `.` at the end**:

```bash
django-admin startproject config .
```

> The dot means "create it **here**, in this folder". Without the dot you get an extra folder inside a folder, which is confusing.

### Step 3 — Look at what was created

In the VS Code Explorer, open `my_journal`. You should see:

```text
my_journal/
├── manage.py          ← the "remote control" for your project
└── config/
    ├── __init__.py    ← tells Python "this folder is a package" (empty, leave it)
    ├── settings.py    ← ALL the settings of your website
    ├── urls.py        ← the list of web addresses (URLs) your site has
    ├── asgi.py        ← used by some web servers (leave it)
    └── wsgi.py        ← used by web servers when the site goes online (leave it)
```

### Step 4 — Start the server

```bash
python manage.py runserver
```

You should see something like this:

```text
Watching for file changes with StatReloader
...
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations ...
Django version 5.2.8, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

> The red/yellow message **"You have 18 unapplied migration(s)"** is **normal**. Ignore it for now. We fix it in Lesson 06 when we connect the real database.

### Step 5 — Open your website

Open your browser and go to:

**http://127.0.0.1:8000/**

### Step 6 — Stop the server

Click inside the terminal and press **Ctrl + C**. The server stops.

---

## Check

- [ ] You saw a page with a **rocket** and the words **"The install worked successfully! Congratulations!"**
- [ ] After **Ctrl + C**, refreshing the browser shows "This site can't be reached" (because the server is stopped).

---

## What just happened?

- `django-admin startproject config .` created the basic files every Django website needs.
- `python manage.py runserver` started a **small web server on your own computer**.
- `127.0.0.1` means **"this computer"** (you may also see it called `localhost`). `8000` is the **port** — like a door number on your computer.
- Your browser sent a **GET** request to that server, and Django answered with the rocket page — exactly like the Wikipedia example in Lesson 01, but this time **you** are the server!
- A file called `db.sqlite3` may have appeared. It is a tiny practice database Django makes by default. We will **delete it** in Lesson 06 and use **PostgreSQL** instead.

### `manage.py` = your remote control

You will use it all day. Always run it **from the `my_journal` folder** (where `manage.py` is):

| Command | What it does |
|---|---|
| `python manage.py runserver` | Start the website on your computer |
| `python manage.py startapp name` | Create a new app |
| `python manage.py makemigrations` | Prepare database changes |
| `python manage.py migrate` | Apply database changes |
| `python manage.py createsuperuser` | Create an admin account |
| `python manage.py shell` | Python shell connected to your project |
| `python manage.py test` | Run automated tests |

---

## Try it yourself

1. Start the server again on a **different port**: `python manage.py runserver 8080`
2. Open **http://127.0.0.1:8080/** — does the rocket appear?
3. Stop it with **Ctrl + C**.
4. Open `config/settings.py` and find the line `INSTALLED_APPS`. How many apps are already listed? (Answer: 6 — Django's built-in apps.)

---

## If it does not work

| Problem | Fix |
|---|---|
| `django-admin` is not recognized | `.venv` is not active. Activate it (Lesson 02, Step 6) and try again. You can also use `python -m django startproject config .` |
| `can't open file 'manage.py'` | You are not inside `my_journal`. Run `cd my_journal`. |
| You forgot the dot and now have `my_journal/config/config/` | Delete the `config` folder and `manage.py` inside `my_journal`, then run Step 2 again **with the dot**. |
| `Error: That port is already in use` | Another server is still running. Find the other terminal and press **Ctrl + C**, or use `python manage.py runserver 8001`. |

### Every time you come back to work

Open a new terminal in VS Code, then:

```powershell
.venv\Scripts\Activate.ps1
```
```bash
cd my_journal
```

(macOS/Linux: `source .venv/bin/activate` then `cd my_journal`.)

---

**Next:** [04_YOUR_FIRST_APP_AND_PAGE.md](04_YOUR_FIRST_APP_AND_PAGE.md)
