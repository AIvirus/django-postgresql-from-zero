# SQL Practice for Your Learning Journal

Run these in **Neon → SQL Editor** (Lesson 16). Start with `SELECT` (read-only) queries.

## Your tables

| Table | Comes from |
|---|---|
| `auth_user` | Django's built-in users |
| `auth_group`, `auth_user_groups` | Groups and who is in them |
| `accounts_profile` | `Profile` model (One-to-One with user) |
| `journal_category` | `Category` model |
| `journal_article` | `Article` model (`author_id` = One-to-Many) |
| `journal_article_categories` | Linking table for Article ↔ Category (Many-to-Many) |
| `django_migrations` | Django's diary of applied migrations |
| `django_session` | Logged-in sessions |

## Reading

```sql
-- All articles, newest first
SELECT id, title, author_id, is_published FROM journal_article ORDER BY created_at DESC;

-- How many published articles?
SELECT COUNT(*) FROM journal_article WHERE is_published = TRUE;

-- Search titles (case-insensitive)
SELECT id, title FROM journal_article WHERE title ILIKE '%django%';

-- The 3 newest articles
SELECT title, created_at FROM journal_article ORDER BY created_at DESC LIMIT 3;
```

## Joining linked tables

```sql
-- Article + author name (One-to-Many)
SELECT a.title, u.username
FROM journal_article AS a
JOIN auth_user AS u ON a.author_id = u.id;

-- Article + its categories (Many-to-Many through the linking table)
SELECT a.title, c.name AS category
FROM journal_article_categories AS ac
JOIN journal_article  AS a ON ac.article_id  = a.id
JOIN journal_category AS c ON ac.category_id = c.id;

-- User + profile (One-to-One). LEFT JOIN keeps users with no profile.
SELECT u.username, p.display_name
FROM auth_user AS u
LEFT JOIN accounts_profile AS p ON p.user_id = u.id;

-- Articles per author
SELECT u.username, COUNT(a.id) AS articles
FROM auth_user AS u
LEFT JOIN journal_article AS a ON a.author_id = u.id
GROUP BY u.username
ORDER BY articles DESC;
```

## ORM ↔ SQL

| Django ORM | SQL |
|---|---|
| `Article.objects.all()` | `SELECT * FROM journal_article` |
| `.filter(is_published=True)` | `WHERE is_published = TRUE` |
| `.filter(title__icontains="x")` | `WHERE title ILIKE '%x%'` |
| `.order_by("-created_at")` | `ORDER BY created_at DESC` |
| `[:5]` | `LIMIT 5` |
| `.count()` | `SELECT COUNT(*)` |
| `.select_related("author")` | `JOIN auth_user ...` |

To see the SQL of any QuerySet: `print(Article.objects.filter(is_published=True).query)` in `python manage.py shell`.

## Changing data

`UPDATE` and `DELETE` change data. **Always** write the `WHERE` first, and only practise on test rows.

```sql
UPDATE journal_article SET title = 'Updated title' WHERE id = 1;
DELETE FROM journal_article WHERE id = 1;
```

Without `WHERE`, **every row** is changed or deleted. In a Django project, use the website, Admin or ORM for normal changes.

Never build SQL by gluing user input into a string — that allows **SQL injection**. The ORM protects you automatically.

More practice without Neon: `notebooks/01_Database_and_SQL_Basics.ipynb`.
