# Deployment Checklist (Lesson 17)

Tick every box before you share your live link.

**Code**
- [ ] All tests pass locally: `python manage.py test`.
- [ ] `requirements.txt` is next to `manage.py`.
- [ ] WhiteNoise is in `MIDDLEWARE`; `STATIC_ROOT` and `STORAGES` are set.
- [ ] `python manage.py check` shows no issues.

**Secrets**
- [ ] `.env` is listed in `.gitignore`.
- [ ] `git status` does **not** show `.env`.
- [ ] GitHub does **not** contain `.env`, passwords or connection strings.

**Render environment variables**
- [ ] `DATABASE_URL` → the **live** Neon database (not your laptop one).
- [ ] `DJANGO_SECRET_KEY` → a **new** key, different from your laptop.
- [ ] `DJANGO_DEBUG` → `False`.
- [ ] `DJANGO_ALLOWED_HOSTS` → `.onrender.com` or your exact host (no `https://`).
- [ ] `DJANGO_CSRF_TRUSTED_ORIGINS` → `https://*.onrender.com` or your exact `https://` address.

**Render commands**
- [ ] Build: `pip install -r requirements.txt && python manage.py collectstatic --no-input && python manage.py migrate`
- [ ] Start: `gunicorn config.wsgi:application`

**After deploying**
- [ ] Home page loads **with** CSS.
- [ ] Register, log in, log out work.
- [ ] Create an article; refresh; it's still there.
- [ ] Another user cannot edit it (404).
- [ ] Superuser created securely (Lesson 17, Part F) and `/admin/` works.
- [ ] A missing page shows a plain "Not Found" (no yellow debug page).

**Remember:** a free demo is not a production system. Real production also needs backups, monitoring, logging, secret rotation and a security review.
