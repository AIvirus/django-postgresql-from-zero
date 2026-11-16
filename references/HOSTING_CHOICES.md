# Hosting Choices — Where Can a Django Site Live? (Optional Reading)

Lesson 17 uses **GitHub + Render + Neon**. This page explains why, and what other options exist.

## What a host does

A **host** runs your application on a computer connected to the internet. Your **database** is a separate service.

| Job | Service used in this course |
|---|---|
| Store code | GitHub |
| Run Django | Render (free web service) |
| Store data | Neon (free PostgreSQL) |

## Comparison (free plans change — always check the official pages)

| Platform | Free option | Main limitation | Use in this course |
|---|---|---|---|
| **Render** | Free Python web service | Sleeps after ~15 min idle (slow first visit); its disk is temporary; Render's own free PostgreSQL expires after 30 days | Main guided path (with Neon for the database) |
| **Vercel** | Hobby plan with Python functions | Runs Django as short-lived "functions", not an always-on server; time and size limits | Optional comparison only |
| **Koyeb** | One small free service in some regions | Small resources; scales to zero; no permanent disk | Optional comparison |
| **Railway** | Trial credit | Becomes paid after credits | Not free long-term |
| **PythonAnywhere** | Limited free account | Restrictions on outgoing connections and features | Not the main path |

## Why Render + Neon?

- Render runs Django the **normal** way: one process that stays running and answers requests — the same model you'll learn properly on Day 6.
- Neon keeps your data **separate** from Render, so it isn't lost when Render's temporary disk is reset.

## Optional: Vercel (only after Render works)

1. Push your project to GitHub (Lesson 17, Part D).
2. In Vercel: **Add New Project** → import the repository → root directory = the folder with `manage.py`.
3. Follow Vercel's **current** official Django guide (it changes over time).
4. Add the same environment variables as on Render (`DATABASE_URL` of a **separate** Neon database, a **new** `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=False`, `DJANGO_ALLOWED_HOSTS`, `DJANGO_CSRF_TRUSTED_ORIGINS`).
5. Vercel does not run `migrate` for you. Run it once from your laptop against the live database (same trick as Lesson 17, Part F).

## Never store important files on a free host's disk

Free hosts reset their disk. Uploaded images/files need proper storage (e.g. object storage like S3) — a later-day topic.

## Official pages

- Render free services: https://render.com/docs/free
- Render + Django: https://render.com/docs/deploy-django
- Neon pricing: https://neon.com/pricing
- Neon docs: https://neon.com/docs
- Vercel Python runtime: https://vercel.com/docs/functions/runtimes/python
- Koyeb free instances: https://www.koyeb.com/docs/reference/instances
- Railway pricing: https://railway.com/pricing
