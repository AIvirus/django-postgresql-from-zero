# Lesson 18 — Final Checklist and What's Next

**Time:** 15 minutes  |  **You need:** Lessons 01–16 finished (17 optional)

---

## Goal

Check that you really understand everything from today, and see how it connects to the next days.

---

## Part 1 — "I can DO it" checklist

Tick each one **on your own project** (`my_journal`):

**Setup**
- [ ] I can open a new terminal, activate `.venv`, and `cd my_journal`.
- [ ] I can start (`runserver`) and stop (**Ctrl + C**) the server.

**Database**
- [ ] My `.env` has `DATABASE_URL` and `DJANGO_SECRET_KEY`, and it is **not** in Git.
- [ ] I can generate a new secret key with Django's command.
- [ ] I can change a model, run `makemigrations` and `migrate`, and see the change in Neon.

**Website**
- [ ] Home, article list, search, category filter and detail pages work.
- [ ] A missing or draft article shows **404**.
- [ ] Register, log in and log out work.
- [ ] A logged-in user can create, edit and delete **their own** articles.
- [ ] Another user gets **404** on my edit/delete URLs.
- [ ] Profile page works.
- [ ] Only `ContentEditors` members (and superusers) can open the editor area.

**Quality**
- [ ] `python manage.py test` → all tests **OK**.
- [ ] I can read a red error message from the **last line** upwards.
- [ ] I can query my tables in the Neon SQL Editor.

---

## Part 2 — "I can EXPLAIN it" checklist

Explain each one **out loud** (or to a friend) in one or two sentences. If you can't, re-read the lesson in brackets.

1. What happens from typing a URL to seeing the page? *(01, 04)*
2. GET vs POST — and why delete must be POST. *(01, 13)*
3. Project vs app. *(03, 04)*
4. Model, View, Template — what each one does. *(04, 05, 07)*
5. Why `.env` exists and why it's never uploaded. *(06)*
6. `makemigrations` vs `migrate`. *(07)*
7. One-to-One, One-to-Many, Many-to-Many — with an example each. *(09)*
8. `CASCADE` vs `PROTECT`. *(09)*
9. What the ORM is, and `get()` vs `filter()`. *(10)*
10. What `select_related` fixes (the N+1 problem). *(11)*
11. How a password is stored. What a session cookie is. *(12)*
12. What `{% csrf_token %}` protects against. *(12)*
13. Authentication vs authorization. *(14)*
14. Why hiding the Edit button is **not** security. *(13, 14)*
15. What a test proves — and what it does not prove. *(15)*
16. Why `DEBUG` must be `False` on a live website. *(17)*

Want more practice questions? → [practice/WORKBOOK.md](practice/WORKBOOK.md) (answers in [practice/ANSWERS.md](practice/ANSWERS.md) — try first, then check!)

---

## Part 3 — Bonus challenges (pick any)

Each one uses only what you learned today.

1. **Comments:** add a `Comment` model (`article` ForeignKey, `author` ForeignKey, `text`, `created_at`). Show comments on the detail page and let logged-in users add one.
2. **Category page:** `/categories/<id>/` showing all published articles in that category.
3. **Author page:** `/authors/<username>/` showing one author's published articles. *(Hint: `filter(author__username=username)`)*
4. **Pagination:** show 5 articles per page on `/articles/` using Django's `Paginator`.
5. **Likes:** a `ManyToManyField` from `Article` to users called `liked_by`, and a **Like** button (POST!).

For each challenge: model → migration → view → URL → template → **test**.

---

## Part 4 — How today connects to the next days

| Day | What you'll do | What you'll reuse from today |
|---|---|---|
| **Day 3** | Django REST Framework, FastAPI, Git/GitHub, Docker | Your models, ORM and auth — DRF turns them into an **API** (JSON instead of HTML) |
| **Day 4** | **Blog Discovery & Recommendation** project | Article + Author (One-to-Many) + Category (Many-to-Many), CRUD, search, Admin, templates |
| **Day 5** | **AI WarRoom** project | Models with relationships (business → scenarios → simulation runs), form validation, permissions, PostgreSQL |
| **Day 6** | Linux, servers, CI/CD, monitoring | Environment variables, `DEBUG=False`, gunicorn, static files — today's Lesson 17, done properly |

If you understand **a model, a ForeignKey, a migration, a QuerySet, a form and a permission check**, you are ready for the projects.

---

## Keep these handy

| File | Use it when… |
|---|---|
| [references/COMMANDS_CHEAT_SHEET.md](references/COMMANDS_CHEAT_SHEET.md) | You forgot a command |
| [references/COMMON_ERRORS.md](references/COMMON_ERRORS.md) | Something is red and you don't know why |
| [references/GLOSSARY.md](references/GLOSSARY.md) | You don't know a word |
| [references/SQL_PRACTICE.md](references/SQL_PRACTICE.md) | You want to practise SQL |
| [references/PROFESSIONAL_HABITS.md](references/PROFESSIONAL_HABITS.md) | You want to work like a real software team |
| [django_lab/](django_lab/README.md) | You want to compare your code with the finished version |

**Congratulations — you built a real, database-backed website with user accounts, security rules and tests!**
