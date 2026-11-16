# Lesson 11 — Show Articles on Your Website (Read)

**Time:** 40 minutes  |  **You need:** Lesson 10 finished, at least one **published** article (make one in Admin if needed)

---

## Goal

By the end of this lesson your website will have:

- a **home page** showing the 5 latest published articles,
- an **Articles** page (`/articles/`) with **search** and a **category filter**,
- a **detail** page for each article (`/articles/1/`),
- a proper **404** when an article doesn't exist.

This is the **R** in CRUD — **Read** — on real web pages.

---

## Understand

### Views + ORM + templates, together

Now we join everything you learned:

```text
GET /articles/  →  urls.py  →  views.article_list
                                   │  Article.objects.filter(is_published=True)   (ORM, Lesson 10)
                                   ▼
                          render("journal/article_list.html", {"articles": ...})
                                   │  {% for article in articles %}               (template, Lesson 05)
                                   ▼
                          HTML page in the browser
```

### URLs with a number inside

```python
path("articles/<int:pk>/", views.article_detail, name="article_detail")
```

`<int:pk>` means "**a number goes here**, call it `pk`". For `/articles/7/`, Django calls `article_detail(request, pk=7)`. (`pk` = **primary key** = the id.)

### Reading the search box: `request.GET`

When you search, the browser opens `/articles/?q=django&category=2`. The part after `?` is the **query string**. In the view:

```python
request.GET.get("q", "")         # → "django"   ("" if missing)
request.GET.get("category")      # → "2"        (None if missing)
```

### `get_object_or_404`

```python
article = get_object_or_404(Article, pk=pk, is_published=True)
```
= "Get this article. If it doesn't exist (or isn't published), show a **404 Not Found** page." No crash, no secrets leaked.

---

## Do it now

### Step 1 — Teach the Article its own address

Open **`journal/models.py`**.

At the top, add this import (under the other imports):

```python
from django.urls import reverse
```

At the bottom of the `Article` class (after `__str__`, same indentation), add:

```python
    def get_absolute_url(self):
        return reverse("journal:article_detail", kwargs={"pk": self.pk})
```

> This is a **method**, not a field. It doesn't change the table, so **no migration is needed**. (Try `python manage.py makemigrations` — it says `No changes detected`.)

### Step 2 — Write the views

Open **`journal/views.py`**. Replace everything with:

```python
from django.db.models import Q
from django.shortcuts import get_object_or_404, render

from .models import Article, Category


def home(request):
    latest_articles = Article.objects.filter(is_published=True).select_related("author")[:5]
    return render(request, "journal/home.html", {"latest_articles": latest_articles})


def article_list(request):
    articles = (
        Article.objects.filter(is_published=True)
        .select_related("author")
        .prefetch_related("categories")
    )

    query = request.GET.get("q", "").strip()
    if query:
        articles = articles.filter(
            Q(title__icontains=query)
            | Q(subtitle__icontains=query)
            | Q(body__icontains=query)
        )

    category_id = request.GET.get("category")
    if category_id:
        articles = articles.filter(categories__id=category_id)

    return render(request, "journal/article_list.html", {
        "articles": articles.distinct(),
        "categories": Category.objects.all(),
        "query": query,
        "selected_category": category_id,
    })


def article_detail(request, pk):
    article = get_object_or_404(
        Article.objects.select_related("author").prefetch_related("categories"),
        pk=pk,
        is_published=True,
    )
    return render(request, "journal/article_detail.html", {"article": article})
```

### Step 3 — Add the URLs

Open **`journal/urls.py`**. Make `urlpatterns` look like this:

```python
urlpatterns = [
    path("", views.home, name="home"),
    path("articles/", views.article_list, name="article_list"),
    path("articles/<int:pk>/", views.article_detail, name="article_detail"),
]
```

### Step 4 — Article list template

Create **`journal/templates/journal/article_list.html`**:

```html
{% extends "journal/base.html" %}
{% block title %}Articles{% endblock %}
{% block content %}
<h1>Published articles</h1>

<form method="get" class="filter-form">
    <label for="q">Search articles</label>
    <input id="q" type="search" name="q" value="{{ query }}" placeholder="Search by title or text">

    <label for="category">Category</label>
    <select id="category" name="category">
        <option value="">All categories</option>
        {% for category in categories %}
            <option value="{{ category.id }}" {% if selected_category == category.id|stringformat:"s" %}selected{% endif %}>{{ category.name }}</option>
        {% endfor %}
    </select>

    <button class="button" type="submit">Search</button>
</form>

{% for article in articles %}
    <article class="card">
        <h2><a href="{{ article.get_absolute_url }}">{{ article.title }}</a></h2>
        {% if article.subtitle %}<p>{{ article.subtitle }}</p>{% endif %}
        <p>{{ article.body|truncatewords:35 }}</p>
        <small>By {{ article.author.username }} · {{ article.created_at|date:"d M Y" }}</small>
        <p>{% for category in article.categories.all %}<span class="tag">{{ category.name }}</span>{% endfor %}</p>
    </article>
{% empty %}
    <p>No articles match your search.</p>
{% endfor %}
{% endblock %}
```

### Step 5 — Article detail template

Create **`journal/templates/journal/article_detail.html`**:

```html
{% extends "journal/base.html" %}
{% block title %}{{ article.title }}{% endblock %}
{% block content %}
<article class="card">
    <h1>{{ article.title }}</h1>
    {% if article.subtitle %}<p class="subtitle">{{ article.subtitle }}</p>{% endif %}
    <p class="muted">By {{ article.author.username }} · {{ article.created_at|date:"d M Y, H:i" }}</p>
    <div>{{ article.body|linebreaks }}</div>
    <p>{% for category in article.categories.all %}<span class="tag">{{ category.name }}</span>{% endfor %}</p>
</article>
<a href="{% url 'journal:article_list' %}">Back to articles</a>
{% endblock %}
```

### Step 6 — Update the home page template

Replace **`journal/templates/journal/home.html`** with:

```html
{% extends "journal/base.html" %}
{% load static %}
{% block title %}Home | Learning Journal{% endblock %}
{% block content %}
<section class="hero">
    <p class="eyebrow">DJANGO + POSTGRESQL PRACTICAL</p>
    <img class="learning-image" src="{% static 'journal/learning-mark.svg' %}" alt="Notebook illustration">
    <h1>Learn how a real Django website works.</h1>
    <p>Explore articles, categories, user accounts, forms, database relationships, and PostgreSQL.</p>
    <p data-demo-message>Press the button to see a small JavaScript interaction.</p>
    <button type="button" data-demo-button>Try JavaScript</button>
    <p><a class="button" href="{% url 'journal:article_list' %}">Explore articles</a></p>
</section>

<h2>Latest published articles</h2>
{% for article in latest_articles %}
    <article class="card">
        <h3><a href="{{ article.get_absolute_url }}">{{ article.title }}</a></h3>
        <p>{{ article.subtitle }}</p>
        <small>By {{ article.author.username }}</small>
    </article>
{% empty %}
    <p>No published articles yet. Add one in Admin and tick "Is published".</p>
{% endfor %}
<script src="{% static 'journal/app.js' %}" defer></script>
{% endblock %}
```

### Step 7 — Add an "Articles" link to the header

In **`journal/templates/journal/base.html`**, replace the `<nav>...</nav>` part with:

```html
    <nav>
        <a href="{% url 'journal:article_list' %}">Articles</a>
    </nav>
```

### Step 8 — Try every page

Start the server (`python manage.py runserver`) and open each link:

| Open | You should see |
|---|---|
| http://127.0.0.1:8000/ | Your published articles under **Latest published articles** |
| http://127.0.0.1:8000/articles/ | A search form + a card for each **published** article |
| Click an article title | Its detail page, with category tags |
| Search for `django` | Only matching articles |
| Choose a category + **Search** | Only articles in that category. Look at the address bar: `?q=&category=1` |
| http://127.0.0.1:8000/articles/9999/ | **Page not found (404)** |
| The id of your **draft** article, e.g. `/articles/2/` | **404** — drafts are not public! |

---

## Check

- [ ] Home shows the latest **published** articles.
- [ ] `/articles/` lists published articles; the draft is **not** there.
- [ ] Search and category filter work.
- [ ] Clicking a title opens `/articles/<id>/`.
- [ ] A missing or draft article gives **404**.

---

## What just happened?

| Code | What it does |
|---|---|
| `filter(is_published=True)` | Only public articles — drafts stay hidden |
| `Q(a) \| Q(b) \| Q(c)` | **OR**: title **or** subtitle **or** body contains the search word |
| `filter(categories__id=category_id)` | Follow the many-to-many link and match the category id |
| `.distinct()` | Remove duplicates that can appear when filtering through a many-to-many link |
| `[:5]` | Only the first 5 → SQL `LIMIT 5` |
| `get_object_or_404(...)` | One article, or a 404 page |
| `{{ article.get_absolute_url }}` | Builds `/articles/7/` from the URL **name** — if you change the address in `urls.py`, all links still work |
| `{% for %} ... {% empty %}` | Loop; the `empty` part shows when there are no results |
| `|truncatewords:35`, `|date:"d M Y"`, `|linebreaks` | **Filters** — small helpers that format a value |

### Why `select_related` and `prefetch_related`?

The list shows each article's **author**. Without help, Django would run **1** query for the articles, then **1 more for each article's author** — 50 articles = 51 queries! This is called the **N+1 problem**.

- `select_related("author")` → gets articles **and** authors in **one** query (a SQL `JOIN`). Use it for ForeignKey / OneToOne.
- `prefetch_related("categories")` → gets all the categories in **one extra** query. Use it for Many-to-Many.

The page looks the same — but it's much faster with many articles.

---

## Try it yourself

1. In `article_list.html`, show how many results there are: `<p>{{ articles|length }} article(s) found.</p>`
2. Change `[:5]` in `home` to `[:2]`. How many articles show on the home page now? Change it back.
3. Make a page `/categories/` that lists all categories (view + URL + template). Use what you learned!
4. In Admin, publish your draft. Is it now on `/articles/`? Can you open its detail page?

---

## If it does not work

| Error | Fix |
|---|---|
| `NoReverseMatch: Reverse for 'article_detail' not found` | URL name in `journal/urls.py` must be exactly `article_detail`, and `app_name = "journal"` must be there. |
| `TemplateDoesNotExist: journal/article_list.html` | File must be in `journal/templates/journal/`. Check spelling. |
| `NameError: name 'reverse' is not defined` | Add `from django.urls import reverse` at the top of `models.py`. |
| `NameError: name 'Q' is not defined` | Add `from django.db.models import Q` at the top of `views.py`. |
| The list is empty | No **published** articles. In Admin, tick **Is published** on an article. |
| `TypeError: article_detail() got an unexpected keyword argument 'pk'` | The view's parameter must be named `pk` to match `<int:pk>`. |

Compare with the answers: `django_lab/journal/views.py` (top 3 functions), `django_lab/journal/urls.py`, and the templates in `django_lab/journal/templates/journal/`.

---

**Next:** [12_REGISTER_LOGIN_LOGOUT.md](12_REGISTER_LOGIN_LOGOUT.md)
