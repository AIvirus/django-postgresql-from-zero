# Common Errors and How to Fix Them

**How to read an error:** start with the **last line** of the red text. It names the problem. Then look a few lines up for the **file name and line number** that caused it. Fix **one** thing, then try again.

**Never** "fix" an error by deleting your database, your migrations folder, or turning off security (CSRF, SSL).

---

## Setup errors

| Error | Cause | Fix |
|---|---|---|
| `python` is not recognized | Python not installed or not on PATH | Reinstall Python and tick **Add python.exe to PATH**; reopen VS Code. Or use `py` (Windows) / `python3` (Mac). |
| `running scripts is disabled on this system` (PowerShell) | Windows blocks the activate script | Use a **Command Prompt** terminal and `.venv\Scripts\activate.bat`, or run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once. |
| `Could not open requirements file: requirements.txt` | Wrong folder | Run it from **`Day-02-Django`** (the folder that contains `requirements.txt`). |
| `No module named django` / `dj_database_url` / `dotenv` | `.venv` not active, or packages not installed | Look for `(.venv)`. Activate it. Then `pip install -r requirements.txt` from `Day-02-Django`. |
| `django-admin` is not recognized | `.venv` not active | Activate it, or use `python -m django startproject config .` |
| `can't open file '...manage.py'` | Wrong folder | `cd my_journal` (the folder that contains `manage.py`). |
| `That port is already in use` | Another server is running | Stop it with **Ctrl + C** in the other terminal, or `runserver 8001`. |

## `.env` and database connection errors

| Error | Cause | Fix |
|---|---|---|
| `DATABASE_URL is missing` / `DJANGO_SECRET_KEY is missing` | `.env` not found or misspelled | `.env` must be **next to `manage.py`**, named exactly `.env` (not `.env.txt`), and the names must be spelled exactly. |
| `password authentication failed` | Wrong password in the connection string | In Neon → **Connect** → **Show password** → copy again. Or reset the role password and copy the new string. |
| `could not translate host name` | Wrong host or no internet | Copy the **whole** connection string again, no spaces. Check your internet. |
| `connection timed out` | Database asleep, or network blocks port 5432 | Try again. Try a different network (phone hotspot). |
| `invalid connection option "channel_binding"` / channel binding error | Older database driver | Remove `&channel_binding=require` from the end of `DATABASE_URL` (keep `?sslmode=require`). |
| SSL error | SSL part missing | The URL must end with `?sslmode=require`. Don't remove SSL. |
| `relation "..." does not exist` / `no such table` | Migrations not applied to **this** database | `python manage.py makemigrations` then `python manage.py migrate`. |

## Model and migration errors

| Error | Fix |
|---|---|
| `No changes detected` | Save `models.py`. Check the app is in `INSTALLED_APPS`. Try `makemigrations APPNAME`. |
| Django asks *"Please select a fix: 1) Provide a one-off default"* | You added a required field to a table that has rows. Type `2`, add `default=...` to the field, try again. |
| `fields.E120: CharFields must define a 'max_length'` | Add `max_length=...` to the `CharField`. |
| `IntegrityError: duplicate key value violates unique constraint` | You tried to save a value that must be unique (e.g. a category name, or a second profile for the same user). |
| `ProtectedError` / "would require deleting protected related objects" | `on_delete=PROTECT` is working. Delete the linked rows first. |

## URL, view and template errors

| Error | Fix |
|---|---|
| `Page not found (404)` | The URL isn't in `urls.py`, the record doesn't exist, it isn't yours, or it's a draft. The yellow page lists every URL Django knows. |
| `NoReverseMatch` | The URL **name** in `{% url %}` / `redirect()` / `reverse()` is wrong, the namespace (`app_name`) is missing, or a required argument (like `article.pk`) is missing. |
| `'xyz' is not a registered namespace` | Add `app_name = "xyz"` to that app's `urls.py`, and `include()` it in `config/urls.py`. |
| `TemplateDoesNotExist` | Wrong folder. Templates go in `APP/templates/APP/name.html`. Restart the server after creating folders. |
| `Invalid block tag 'static'` | Add `{% load static %}` at the top of the template. |
| `The view ... didn't return an HttpResponse object` | Your view is missing `return render(...)` / `return redirect(...)`. |
| `NameError: name 'X' is not defined` | Missing import at the top of the file. |
| `IndentationError` | Python needs 4 spaces of indentation inside functions/classes. |
| Page has no CSS | Static file in the wrong folder (`APP/static/APP/style.css`), or browser cache — press **Ctrl + F5**. |

## Forms and login errors

| Error | Fix |
|---|---|
| `403 Forbidden — CSRF verification failed` | Add `{% csrf_token %}` inside every `<form method="post">`. Never turn off CSRF. |
| `405 Method Not Allowed` on logout | Logout must be a `<form method="post">` button, not a link. |
| `null value in column "author_id"` | Set `article.author = request.user` before `article.save()`. |
| `needs to have a value for field "id" before this many-to-many relationship can be used` | Call `form.save_m2m()` **after** `article.save()`. |
| After login you get `/accounts/profile/` 404 | Add `LOGIN_REDIRECT_URL` to `settings.py`. |

## Test errors

| Error | Fix |
|---|---|
| `database "test_neondb" already exists` → `Type 'yes'` | A previous test run was interrupted. Type `yes`. |
| `permission denied to create database` | Use the default `neondb_owner` connection string. |
| `Ran 0 tests` | Test methods must start with `test_`; file must be `tests.py`. |

## Deployment errors (Lesson 17)

| Error | Fix |
|---|---|
| **Bad Request (400)** | `DJANGO_ALLOWED_HOSTS` must contain your host **without** `https://` (e.g. `.onrender.com`). |
| CSRF error on the live site | Set `DJANGO_CSRF_TRUSTED_ORIGINS=https://*.onrender.com`. |
| No CSS on the live site | WhiteNoise middleware missing, or `collectstatic` missing in the build command. |
| `Could not open requirements file` on Render | `requirements.txt` must be in the repository's top folder, next to `manage.py`. |
| First visit very slow | Free service was sleeping. Wait up to a minute. |
