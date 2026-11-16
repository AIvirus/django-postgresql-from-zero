# Lesson 16 — Look Inside PostgreSQL with SQL

**Time:** 20 minutes  |  **You need:** Lesson 15 finished, some users/articles/categories in your database

---

## Goal

By the end of this lesson you can read your own website's data **directly with SQL** in the Neon SQL Editor, and you'll see how each ORM command you used matches an SQL command.

---

## Understand

**SQL** (Structured Query Language) is the language databases speak. Django's ORM writes SQL **for** you, but knowing basic SQL helps you:

- check what's really saved,
- understand what the ORM is doing,
- find bugs and slow pages.

| ORM (Python) | SQL |
|---|---|
| `Article.objects.all()` | `SELECT * FROM journal_article;` |
| `.filter(is_published=True)` | `WHERE is_published = TRUE` |
| `.order_by("-created_at")` | `ORDER BY created_at DESC` |
| `[:5]` | `LIMIT 5` |
| `.count()` | `SELECT COUNT(*)` |
| `select_related("author")` | `JOIN auth_user ON ...` |

Table names = `app_model`: `journal_article`, `journal_category`, `journal_article_categories`, `accounts_profile`, `auth_user`.

---

## Do it now

Open **Neon → your project → SQL Editor**. Paste each query, click **Run**, and look at the result.

### 1. All articles

```sql
SELECT id, title, author_id, is_published
FROM journal_article
ORDER BY created_at DESC;
```

### 2. Count them

```sql
SELECT COUNT(*) FROM journal_article;
```

### 3. Only published (= `filter(is_published=True)`)

```sql
SELECT id, title
FROM journal_article
WHERE is_published = TRUE;
```

### 4. Search (= `title__icontains="django"`)

```sql
SELECT id, title
FROM journal_article
WHERE title ILIKE '%django%';
```

`ILIKE` = "like, ignoring capital letters". `%` = "anything here".

### 5. JOIN — show the author's **name** instead of a number (One-to-Many)

```sql
SELECT a.title, u.username
FROM journal_article AS a
JOIN auth_user AS u ON a.author_id = u.id;
```

This is what `select_related("author")` does.

### 6. Many-to-Many — which article has which category?

```sql
SELECT a.title, c.name AS category
FROM journal_article_categories AS ac
JOIN journal_article  AS a ON ac.article_id  = a.id
JOIN journal_category AS c ON ac.category_id = c.id
ORDER BY a.title;
```

Two JOINs through the hidden linking table.

### 7. One-to-One — users and their profiles

```sql
SELECT u.username, p.display_name
FROM auth_user AS u
LEFT JOIN accounts_profile AS p ON p.user_id = u.id;
```

`LEFT JOIN` keeps users **even if** they have no profile (their `display_name` shows as empty/NULL).

### 8. How many articles per author? (GROUP BY)

```sql
SELECT u.username, COUNT(a.id) AS articles
FROM auth_user AS u
LEFT JOIN journal_article AS a ON a.author_id = u.id
GROUP BY u.username
ORDER BY articles DESC;
```

### 9. Who is in which group?

```sql
SELECT u.username, g.name AS group_name
FROM auth_user_groups AS ug
JOIN auth_user  AS u ON ug.user_id  = u.id
JOIN auth_group AS g ON ug.group_id = g.id;
```

---

## Check

- [ ] Query 5 shows article titles **with usernames**.
- [ ] Query 6 shows the categories of each article.
- [ ] Query 8 shows how many articles each user wrote.

---

## What just happened?

You read the **same** data your website shows — but straight from PostgreSQL. Every page you built runs queries like these; Django writes them for you.

### Read before you write

`UPDATE` and `DELETE` **change** data. Without a `WHERE`, they change **every row**!

```sql
-- changes ONE article
UPDATE journal_article SET title = 'Fixed title' WHERE id = 1;

-- DANGER: changes EVERY article's title!
-- UPDATE journal_article SET title = 'Fixed title';
```

In a Django project, change data through the website, Admin or the ORM — not with raw SQL. Only practise `UPDATE`/`DELETE` on test rows.

### Data is valuable — keep it safe

A database isn't "just a file". In a real company, losing it can mean losing the business. Real projects need:

- **Backups**, and regular tests that the backups can really be **restored**.
- **Access control** — only the people and programs that need the database can connect.
- **Least-privilege roles** — the website's database role can read and write data, but not, for example, drop every table.
- **Encrypted connections** (`sslmode=require`, which you already use).
- **Reviewing migrations** before applying them to the live database.

Neon's free plan keeps some history so you can restore a recent point in time — see Neon's docs on "restore". But don't keep anything irreplaceable in a free learning database.

---

## Try it yourself

Write the SQL yourself (answers below):

1. Show all categories sorted A→Z.
2. Show only **draft** articles.
3. Show the 3 newest articles.
4. Show every article in the **Django** category.

<details>
<summary>Answers</summary>

```sql
SELECT name FROM journal_category ORDER BY name;

SELECT title FROM journal_article WHERE is_published = FALSE;

SELECT title, created_at FROM journal_article ORDER BY created_at DESC LIMIT 3;

SELECT a.title
FROM journal_article_categories AS ac
JOIN journal_article  AS a ON ac.article_id  = a.id
JOIN journal_category AS c ON ac.category_id = c.id
WHERE c.name = 'Django';
```
</details>

**Want more SQL practice without touching Neon?** Open `notebooks/01_Database_and_SQL_Basics.ipynb`, and see `references/SQL_PRACTICE.md`.

---

## If it does not work

| Error | Fix |
|---|---|
| `relation "journal_article" does not exist` | Wrong database/branch selected at the top of the SQL Editor (choose `neondb`, branch `main`), or migrations weren't applied. |
| `column "..." does not exist` | Check the spelling. In Neon → **Tables**, click the table to see real column names. |
| Text values give an error | Use **single quotes** for text: `'Django'`, not `"Django"`. |

---

**Next:** [17_DEPLOY_YOUR_WEBSITE_OPTIONAL.md](17_DEPLOY_YOUR_WEBSITE_OPTIONAL.md)
