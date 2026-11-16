# Professional Habits — How Real Software Teams Work

These habits don't need advanced skills — just care and consistency. You already practised all of them today.

## 1. Keep each file to its job
| File | Job |
|---|---|
| `models.py` | What data is stored |
| `views.py` | What happens on each request |
| `forms.py` | Checking user input |
| templates | How it looks |
| `admin.py` | Staff tools |
| `tests.py` | Proving it works |

## 2. Protect data
- Validate on the **server** (forms), not only in the browser.
- Check **ownership** and **roles** in the view (`author=request.user`, groups).
- Use database rules for important things (`unique=True`, `on_delete=PROTECT`).
- Use `POST` + `{% csrf_token %}` for anything that changes data.

## 3. Protect secrets
- Secrets live in `.env` (laptop) or the host's environment settings (live) — **never** in code.
- `.env` is in `.gitignore`. Check `git status` before every commit.
- Different secrets and databases for laptop and live.
- Leaked a secret? Replace it immediately.
- No passwords in screenshots or chats.

## 4. Change the database carefully
- Change model → `makemigrations` → **read** the migration → `migrate`.
- Commit migration files together with the model change.
- Make sure you're migrating the **right** database.
- Never delete migrations or the database to "fix" an error.

## 5. Write code others can read
- Clear names (`article_list`, not `al`).
- Small functions that do one thing.
- Don't copy-paste the same logic everywhere.
- A short comment when something is unusual — explain **why**.

## 6. "Done" means really done
Before saying a feature is finished, check:
- [ ] Valid input works.
- [ ] Invalid input shows a helpful error and saves nothing.
- [ ] A missing record gives 404.
- [ ] Users can only reach what they're allowed to.
- [ ] Data is really saved in the database.
- [ ] There's a test for the important rule.
- [ ] No secrets in the code.
- [ ] A teammate could understand it.

## 7. Debug calmly
1. Read the **last line** of the error.
2. Find the file and line number.
3. Change **one** thing.
4. Run again.
5. Write down what fixed it — you'll see it again!
