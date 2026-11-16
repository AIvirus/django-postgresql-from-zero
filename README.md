# django-postgresql-from-zero
A step-by-step, self-learning Django course for complete beginners. Build a Learning Journal website with PostgreSQL (Neon): models, admin, login, CRUD forms, tests, SQL, and optional deployment to Render. Every lesson says what to read, what to run, and what you should see.

## Start here

Open **[00_START_HERE.md](00_START_HERE.md)** and follow the lessons in order (00 to 18). The whole course takes about one day.

| Folder | What is inside |
|---|---|
| `00_...` to `18_...` | The lessons. Read, run, check, then go to the next one. |
| `practice/` | Workbook exercises and answers |
| `references/` | Cheat sheet, common errors, glossary, SQL practice, deployment notes |
| `notebooks/` | Optional Jupyter practice for SQL and the Django ORM |
| `django_lab/` | The finished project (answer key). Use it to compare, not to copy. |

## Quick setup

```bash
python -m venv .venv
.venv\Scripts\Activate.ps1        # Windows PowerShell
source .venv/bin/activate         # macOS / Linux
pip install -r requirements.txt
```

Your database password and secret key go in a `.env` file, which is never uploaded (see `.gitignore`).
