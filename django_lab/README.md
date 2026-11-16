# `django_lab` — The Finished Website (Answer Key)

This folder is the **finished** Learning Journal website — exactly what you build yourself in Lessons 03–15 inside `my_journal/`.

Use it to:

- **compare** — if a file in your `my_journal` doesn't work, open the same file here (e.g. `my_journal/journal/views.py` ↔ `django_lab/journal/views.py`);
- **preview** — run it to see what you are going to build.

> Learning tip: don't copy this whole folder. You learn by typing the code in the lessons.

---

## Run the finished website (about 15 minutes)

You need: **Lesson 02 done** (`.venv` created, packages installed) and a **Neon database** (Lesson 06, Part A).

### 1. Open a terminal in VS Code and activate `.venv`

The terminal must be in the **`Day-02-Django`** folder first:

```powershell
.venv\Scripts\Activate.ps1
```
(macOS/Linux: `source .venv/bin/activate`)

### 2. Make sure the packages are installed

Still in `Day-02-Django` (where `requirements.txt` is):

```bash
pip install -r requirements.txt
```

### 3. Go into this folder

```bash
cd django_lab
```

The terminal line now ends with `django_lab>`. From now on run every command here.

### 4. Create your private `.env` file

Copy `.env.example` to a new file named `.env` **in this folder** (next to `manage.py`):

**Windows:**
```powershell
copy .env.example .env
```
**macOS/Linux:**
```bash
cp .env.example .env
```

Generate a secret key:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Open `.env` and replace:

- `DATABASE_URL=` → your Neon connection string (Lesson 06, Part A)
- `DJANGO_SECRET_KEY=` → the key you just generated

> Tip: use a **separate** Neon project (or Neon branch) for `django_lab`, so it doesn't mix with your own `my_journal` tables.

### 5. Check, create tables, create an admin

```bash
python manage.py check
```
```bash
python manage.py makemigrations accounts journal
```
```bash
python manage.py migrate
```
```bash
python manage.py createsuperuser
```

> Migration files are not included on purpose — `makemigrations` creates them from the models.

### 6. Run it

```bash
python manage.py runserver
```

Open http://127.0.0.1:8000/

### 7. Try it

| Page | Address |
|---|---|
| Home | http://127.0.0.1:8000/ |
| Articles + search | http://127.0.0.1:8000/articles/ |
| Register | http://127.0.0.1:8000/users/register/ |
| Log in | http://127.0.0.1:8000/accounts/login/ |
| My articles | http://127.0.0.1:8000/articles/mine/ |
| Write an article | http://127.0.0.1:8000/articles/new/ |
| Profile | http://127.0.0.1:8000/users/profile/ |
| Editors only | http://127.0.0.1:8000/users/editor-area/ (needs the `ContentEditors` group) |
| Admin | http://127.0.0.1:8000/admin/ |

### 8. Run the tests

```bash
python manage.py test
```

`Ran 12 tests ... OK`

---

## Which lesson built which file?

| File | Lesson |
|---|---|
| `config/settings.py` — `.env`, `DATABASE_URL`, `SECRET_KEY` | 06 |
| `config/settings.py` — `LOGIN_URL`, redirects | 12 |
| `config/settings.py` — WhiteNoise, `STATIC_ROOT`, secure cookies | 17 |
| `config/urls.py` | 04, 12 |
| `journal/models.py` — `Category`, `Article` | 07 (+ `get_absolute_url` in 11) |
| `journal/admin.py` | 08 |
| `accounts/models.py`, `accounts/admin.py` — `Profile` | 09 |
| `journal/views.py` — `home`, `article_list`, `article_detail` | 11 |
| `journal/views.py` — `my_articles`, `article_create`, `article_update`, `article_delete` | 13 |
| `journal/forms.py` | 13 |
| `journal/urls.py` | 04, 11, 13 |
| `journal/templates/journal/*.html` | 05, 11, 13 |
| `journal/static/journal/*` | 05 |
| `accounts/views.py` — `register` | 12 |
| `accounts/views.py` — profile + editor area, `accounts/forms.py` | 14 |
| `accounts/urls.py` | 12, 14 |
| `accounts/templates/registration/*.html` | 12 |
| `accounts/templates/accounts/*.html` | 14 |
| `journal/tests.py`, `accounts/tests.py` | 15 |
| `build.sh`, `render.yaml` | 17 |

---

## Problems?

See [../references/COMMON_ERRORS.md](../references/COMMON_ERRORS.md). The most common ones:

- `Could not open requirements file` → you are in the wrong folder. Both `Day-02-Django` and `django_lab` contain a `requirements.txt`; the terminal must end with one of those folder names.
- `DATABASE_URL is missing` → `.env` must be in **`django_lab`**, next to `manage.py`.
- `No module named ...` → `.venv` is not activated.

## Safety

Never commit `.env`. Never put real credentials in source code. This free setup is for learning and demos, not for real production use.
