# Commands Cheat Sheet

## Every time you open a new terminal

| Computer | Activate `.venv` (from the `Day-02-Django` folder) |
|---|---|
| Windows PowerShell | `.venv\Scripts\Activate.ps1` |
| Windows Command Prompt | `.venv\Scripts\activate.bat` |
| macOS / Linux | `source .venv/bin/activate` |

Then go into your project: `cd my_journal`

You're ready when the line starts with `(.venv)` and ends with `my_journal>`.

## One-time setup (Lesson 02)

| Command | Run from | What it does |
|---|---|---|
| `python -m venv .venv` | `Day-02-Django` | Create the virtual environment |
| `pip install -r requirements.txt` | `Day-02-Django` | Install Django and friends |
| `python -m django --version` | anywhere (with `.venv` on) | Check Django is installed |

## Django commands (run from the folder that has `manage.py`)

| Command | What it does | Lesson |
|---|---|---|
| `django-admin startproject config .` | Create a new project in this folder (note the dot!) | 03 |
| `python manage.py runserver` | Start the website at http://127.0.0.1:8000/ (stop: **Ctrl + C**) | 03 |
| `python manage.py startapp NAME` | Create a new app (then add it to `INSTALLED_APPS`) | 04 |
| `python manage.py check` | Check the project for setup mistakes | 06 |
| `python manage.py makemigrations` | Write migration files from model changes | 07 |
| `python manage.py migrate` | Apply migrations to the database | 06, 07 |
| `python manage.py showmigrations` | Show which migrations are applied `[X]` | 07 |
| `python manage.py sqlmigrate APP 0001` | Show the SQL a migration will run | 07 |
| `python manage.py createsuperuser` | Create an Admin account | 08 |
| `python manage.py changepassword USERNAME` | Change a user's password | 08 |
| `python manage.py shell` | Python shell connected to your project (`exit()` to leave) | 10 |
| `python manage.py dbshell` | Open a SQL prompt to your database (only works if the PostgreSQL `psql` tool is installed; otherwise use the Neon SQL Editor) | 16 |
| `python manage.py test` | Run all automated tests | 15 |
| `python manage.py test journal` | Run only one app's tests | 15 |
| `python manage.py collectstatic` | Copy all static files into `staticfiles/` (for deployment) | 17 |
| `python manage.py check --deploy` | Extra security checks for a live site | 17 |

## Generate a secret key (Lesson 06)

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

## Remember

- **Changed a model?** → `makemigrations` → `migrate`.
- **Created a new template/static folder?** → restart `runserver`.
- **New terminal?** → activate `.venv` → `cd my_journal`.
- `runserver` is for your laptop only, never for a public website.
