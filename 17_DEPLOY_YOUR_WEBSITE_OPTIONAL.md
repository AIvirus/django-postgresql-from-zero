# Lesson 17 — Put Your Website on the Internet (Optional)

**Time:** 45 minutes  |  **You need:** Lessons 01–15 finished, a **GitHub** account, a **Render** account, **Git** installed

> This lesson is **optional**. Git and GitHub are taught properly on Day 3, and servers on Day 6. If you finish it today — great! If not, come back after Day 3.

---

## Goal

By the end of this lesson, **anyone** in the world can open your Learning Journal at an address like `https://my-journal-xyz.onrender.com`.

---

## Understand

### Your laptop vs "deployed"

So far your website only runs **on your laptop** (`127.0.0.1` = this computer). **Deploying** means running it on a computer **on the internet** that is always on.

```text
Visitor's browser
      │  https://my-journal-xyz.onrender.com
      ▼
Render  (runs your Django code with gunicorn)
      │  encrypted connection (DATABASE_URL)
      ▼
Neon    (a SECOND, separate PostgreSQL database for the live site)
```

- **GitHub** stores your code.
- **Render** takes the code from GitHub and runs it.
- **Neon** stores the live data.

### What changes on the internet?

| | Your laptop | Live website |
|---|---|---|
| `DEBUG` | `True` (detailed yellow error pages) | **`False`** (detailed errors would show secrets to strangers!) |
| `SECRET_KEY` | your local key | a **new, different** key |
| Database | `day2-django` Neon project | a **separate** Neon project |
| `ALLOWED_HOSTS` | `127.0.0.1,localhost` | your Render address |
| Who serves CSS/JS | `runserver` | **WhiteNoise** |
| Who runs Django | `runserver` (for development only) | **gunicorn** (a real web server) |
| Where settings come from | `.env` file | Render's **Environment** page (never upload `.env`!) |
| Address | `http://` (fine on your own computer) | **`https://`** — encrypted, so passwords and cookies can't be read on the way. Render gives you HTTPS for free |

### Free plans have limits

- Render's free service **sleeps** after ~15 minutes without visitors. The first visit after that takes up to a minute. That's normal.
- Free plans are for **learning and demos**, not for real businesses. Limits can change — check https://render.com/docs/free and https://neon.com/pricing.

---

## Do it now

### Part A — Make the settings ready for the internet

Open **`my_journal/config/settings.py`**.

**1.** In `MIDDLEWARE`, add WhiteNoise **as the second line**, right after `SecurityMiddleware`:

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",        # ADD THIS
    "django.contrib.sessions.middleware.SessionMiddleware",
    # ... the rest stays the same
]
```

**2.** Find `STATIC_URL = 'static/'`. Under it, add:

```python
STATIC_ROOT = BASE_DIR / "staticfiles"
STORAGES = {
    "default": {"BACKEND": "django.core.files.storage.FileSystemStorage"},
    "staticfiles": {"BACKEND": "whitenoise.storage.CompressedStaticFilesStorage"},
}
```

**3.** At the **very bottom** of the file, add:

```python
CSRF_TRUSTED_ORIGINS = [
    origin.strip()
    for origin in os.environ.get("DJANGO_CSRF_TRUSTED_ORIGINS", "").split(",")
    if origin.strip()
]

if not DEBUG:
    SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
```

Save. Check nothing broke:

```bash
python manage.py check
```

```bash
python manage.py collectstatic --no-input
```

`... static files copied to '...\my_journal\staticfiles'`. (That folder is in `.gitignore`, so it won't be uploaded — Render makes its own.)

### Part B — Put `requirements.txt` inside your project

Render needs the shopping list **inside** `my_journal`. From inside `my_journal`:

**Windows:**
```powershell
copy ..\requirements.txt .
```
**macOS / Linux:**
```bash
cp ../requirements.txt .
```

### Part C — Create a separate database for the live site

1. Neon dashboard → **New Project** → name `day2-django-live` → same region → **Create**.
2. **Connect** → Connection pooling **OFF** → **Show password** → copy the connection string.
3. Keep it in Notepad for now (do **not** put it in any file in your project).

> Why separate? So your classroom experiments never mess up the live website's data, and a leaked laptop password can't touch the live data.

### Part D — Upload your code to GitHub

1. Install Git if needed: https://git-scm.com/downloads. Close and reopen VS Code.
2. On https://github.com, click **New repository** → name `my-journal` → **Private** is fine → **don't** add a README → **Create repository**.
3. In the VS Code terminal, **inside `my_journal`**:

```bash
git init
```

If Git has never been used on this computer, tell it who you are (once):

```bash
git config --global user.name "Your Name"
```
```bash
git config --global user.email "you@example.com"
```

Now check what will be uploaded:

```bash
git add .
```
```bash
git status
```

**STOP AND CHECK:** in the list, you must **NOT** see `.env`, `.venv` or `staticfiles`. You **should** see `.env.example`, `manage.py`, `requirements.txt`, `config/`, `journal/`, `accounts/`.
If `.env` is in the list → your `.gitignore` is wrong (Lesson 06, Part D). Fix it, then run `git rm --cached .env` and `git add .` again.

```bash
git commit -m "My Learning Journal"
```
```bash
git branch -M main
```

Copy the two lines GitHub shows under **"…or push an existing repository"**. They look like this (with **your** username):

```bash
git remote add origin https://github.com/YOUR-USERNAME/my-journal.git
```
```bash
git push -u origin main
```

A browser window may ask you to sign in to GitHub — do it. Refresh the GitHub page: your code is there (and `.env` is **not**).

### Part E — Create the website on Render

1. Go to https://render.com → sign up / log in **with GitHub**.
2. **New → Web Service** → choose your `my-journal` repository (click **Configure GitHub** if you don't see it, and give access to that repo).
3. Fill in:

| Field | Value |
|---|---|
| **Name** | `my-journal-yourname` (this becomes your web address) |
| **Language / Runtime** | `Python 3` |
| **Branch** | `main` |
| **Root Directory** | *(leave empty)* |
| **Build Command** | `pip install -r requirements.txt && python manage.py collectstatic --no-input && python manage.py migrate` |
| **Start Command** | `gunicorn config.wsgi:application` |
| **Instance Type** | **Free** |

4. Scroll to **Environment Variables** → **Add** each of these:

| Key | Value |
|---|---|
| `DATABASE_URL` | the **live** Neon connection string from Part C |
| `DJANGO_SECRET_KEY` | a **brand-new** key — run the Lesson 06 command again and paste the new output |
| `DJANGO_DEBUG` | `False` |
| `DJANGO_ALLOWED_HOSTS` | `.onrender.com` *(the dot at the start means "any address ending in .onrender.com")* |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | `https://*.onrender.com` |

5. Click **Deploy Web Service**. Watch the **Logs**. It takes 2–5 minutes. You'll see packages installing, `collectstatic`, then `Applying ... OK` (migrations on the live database!), then **"Your service is live"**.

### Part F — Create an admin account for the live site

Render's free plan has no terminal, so we run `createsuperuser` **from your laptop**, pointed at the **live** database **just for one command**. (A value set in the terminal wins over the `.env` file.)

**Windows PowerShell** (inside `my_journal`, `.venv` active) — paste your **live** URL between the quotes:

```powershell
$env:DATABASE_URL="postgresql://...your-LIVE-connection-string..."
```
```powershell
python manage.py createsuperuser
```

Then **close this terminal** (bin icon on the terminal panel) so you don't accidentally use the live database later. Open a fresh one for normal work.

> **macOS/Linux:** `DATABASE_URL="postgresql://...live..." python manage.py createsuperuser`
> **Command Prompt:** `set DATABASE_URL=postgresql://...live...` then `python manage.py createsuperuser`, then close the window.

### Part G — Test your live website

Open the address at the top of your Render page, e.g. `https://my-journal-yourname.onrender.com`:

- [ ] Home page with styling (CSS works = WhiteNoise works).
- [ ] Register a user, log in, write and publish an article.
- [ ] Refresh — the article is still there (saved in the **live** Neon database).
- [ ] `/admin/` works with the superuser from Part F.
- [ ] A missing page like `/articles/99999/` shows a **plain** "Not Found" page — **not** the yellow debug page (because `DEBUG=False`).
- [ ] Send the link to a friend!

### Part H — Updating the live site later

Change code → test locally → then:

```bash
git add .
```
```bash
git commit -m "Describe what you changed"
```
```bash
git push
```

Render sees the push and **redeploys automatically** (including any new migrations).

---

## Check

- [ ] Your site opens on a public `https://...onrender.com` address.
- [ ] `.env` is **not** on GitHub.
- [ ] The live site uses a **different** database and secret key from your laptop.
- [ ] `DEBUG` is `False` on Render.

---

## What just happened?

| Step | Why |
|---|---|
| WhiteNoise + `collectstatic` | With `DEBUG=False`, Django stops serving CSS/JS itself. `collectstatic` gathers them into `staticfiles/` and WhiteNoise serves them |
| `gunicorn config.wsgi:application` | `runserver` is only for development. gunicorn is a real server that uses `config/wsgi.py` |
| `migrate` in the build command | Creates/updates tables in the **live** database on every deploy |
| `SECURE_PROXY_SSL_HEADER` + secure cookies | Render talks HTTPS to visitors; this tells Django the connection is secure and makes cookies HTTPS-only |
| Environment Variables on Render | The live site's `.env` — kept secret by Render, never in your code |

### Real production needs more

A real company website also needs backups, monitoring, logging, a custom domain, secret rotation, file storage for uploads and more. You'll meet these on Day 6. See also [references/HOSTING_CHOICES.md](references/HOSTING_CHOICES.md) and [references/DEPLOYMENT_CHECKLIST.md](references/DEPLOYMENT_CHECKLIST.md).

---

## If it does not work

Always look at Render → your service → **Logs** first. Read the **last red lines**.

| Problem | Fix |
|---|---|
| Build fails: `Could not open requirements file` | `requirements.txt` is not in the repo's top folder. Do Part B, then `git add .`, `git commit`, `git push`. |
| Build fails: `DJANGO_SECRET_KEY is missing` / `DATABASE_URL is missing` | Add the environment variable in Render → **Environment**, then **Manual Deploy → Deploy latest commit**. |
| **Bad Request (400)** | `DJANGO_ALLOWED_HOSTS` is wrong. Use `.onrender.com` (or your exact host, **without** `https://`). |
| **403 CSRF verification failed** when logging in | Check `DJANGO_CSRF_TRUSTED_ORIGINS` = `https://*.onrender.com`. |
| Page has no styling | WhiteNoise line missing in `MIDDLEWARE`, or `collectstatic` missing from the Build Command. |
| `ModuleNotFoundError: No module named 'config'` | Your repo has an extra folder level. Set **Root Directory** to the folder that contains `manage.py`. |
| `git push` asks for a password and fails | GitHub doesn't accept account passwords in the terminal. Let the browser sign-in window appear, or install **GitHub Desktop**. Day 3 covers this. |
| Site is slow on first visit | Free service was sleeping. Wait up to a minute. |

Compare with the answers: `django_lab/config/settings.py`, `django_lab/build.sh`, `django_lab/render.yaml`.

---

**Next:** [18_FINAL_CHECKLIST_AND_WHATS_NEXT.md](18_FINAL_CHECKLIST_AND_WHATS_NEXT.md)
