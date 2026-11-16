# Lesson 13 — Forms: Create, Edit and Delete Articles

**Time:** 45 minutes  |  **You need:** Lesson 12 finished, and **two** normal users (e.g. `student1`, `student2`)

---

## Goal

By the end of this lesson logged-in users can:

- **write** a new article (**C**reate),
- see **My articles**, including drafts (**R**ead),
- **edit** their article (**U**pdate),
- **delete** their article after a confirmation (**D**elete),

…and they **cannot** touch anybody else's articles — even if they type the URL by hand.

---

## Understand

### What is a ModelForm?

A **form** collects data from a user. A **ModelForm** builds the form **from your model automatically**: you say "use the Article model, these fields" and Django makes the inputs, checks the types, and can save it.

### Form vs ModelForm

| | `forms.Form` | `forms.ModelForm` |
|---|---|---|
| Fields come from | You write each field yourself | Your **model** (automatically) |
| Can `save()` to the database | No | Yes |
| Good for | Contact form, search box, login | Create/edit a database row (our articles, profiles) |

You've already used both: `UserCreationForm` (Lesson 12) is a ModelForm for the User model; Django's login form is a plain Form.

### GET forms vs POST forms

- **GET** forms put the values in the address (`/articles/?q=django`). Use them for **searching and filtering** — nothing changes, and the link can be shared. (Your search form in Lesson 11.)
- **POST** forms send the values in the request body. Use them for anything that **creates, changes or deletes** data. They always need `{% csrf_token %}`.

### Validation = checking the data on the server

The browser can be tricked, so the **server** checks everything. You can add your own rules with a method named `clean_<fieldname>`:

```python
def clean_title(self):
    title = self.cleaned_data["title"].strip()
    if len(title) < 3:
        raise forms.ValidationError("The title must contain at least 3 characters.")
    return title
```

### Two important security rules

1. **The server decides the author.** The form has **no** author field. The view sets `article.author = request.user`. Otherwise a user could pretend to be someone else.
2. **Ownership check:** when editing/deleting, look up the article **by id AND author**:
   ```python
   article = get_object_or_404(Article, pk=pk, author=request.user)
   ```
   If it's not yours → **404**. Hiding the Edit button is **not** security — someone can type `/articles/1/edit/` by hand.

### `@login_required`

Put `@login_required` above a view → people who aren't logged in are sent to the login page first.

### Delete needs POST

Never delete on a GET (a simple link). Browsers, link previews and search robots open links automatically! So: GET shows **"Are you sure?"**, and only the **POST** (clicking the button) deletes.

---

## Do it now

### Step 1 — Create the form

Create a new file **`journal/forms.py`**:

```python
from django import forms

from .models import Article


class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ["title", "subtitle", "body", "categories", "is_published"]
        widgets = {
            "body": forms.Textarea(attrs={"rows": 8}),
            "categories": forms.CheckboxSelectMultiple(),
        }

    def clean_title(self):
        title = self.cleaned_data["title"].strip()
        if len(title) < 3:
            raise forms.ValidationError("The title must contain at least 3 characters.")
        return title

    def clean_body(self):
        body = self.cleaned_data["body"].strip()
        if len(body) < 10:
            raise forms.ValidationError("Please write at least 10 characters in the article.")
        return body
```

> Look at `fields`: there is **no `author`**. On purpose (security rule 1).

### Step 2 — Add four new views

Open **`journal/views.py`**.

**a)** Replace the import lines at the top with:

```python
from django.contrib.auth.decorators import login_required
from django.db.models import Q
from django.shortcuts import get_object_or_404, redirect, render

from .forms import ArticleForm
from .models import Article, Category
```

**b)** Keep `home`, `article_list` and `article_detail` as they are. **Add** these four functions at the **bottom** of the file:

```python
@login_required
def my_articles(request):
    articles = Article.objects.filter(author=request.user).prefetch_related("categories")
    return render(request, "journal/my_articles.html", {"articles": articles})


@login_required
def article_create(request):
    if request.method == "POST":
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)   # build the article, but don't save yet
            article.author = request.user       # the SERVER decides the author
            article.save()                      # now save it
            form.save_m2m()                     # then save the categories (many-to-many)
            return redirect("journal:my_articles")
    else:
        form = ArticleForm()
    return render(request, "journal/article_form.html", {
        "form": form,
        "page_heading": "Create an article",
    })


@login_required
def article_update(request, pk):
    article = get_object_or_404(Article, pk=pk, author=request.user)   # ownership check
    if request.method == "POST":
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            form.save()
            return redirect("journal:my_articles")
    else:
        form = ArticleForm(instance=article)
    return render(request, "journal/article_form.html", {
        "form": form,
        "page_heading": "Edit an article",
    })


@login_required
def article_delete(request, pk):
    article = get_object_or_404(Article, pk=pk, author=request.user)   # ownership check
    if request.method == "POST":
        article.delete()
        return redirect("journal:my_articles")
    return render(request, "journal/article_confirm_delete.html", {"article": article})
```

### Step 3 — Add the URLs

Make `urlpatterns` in **`journal/urls.py`** look like this:

```python
urlpatterns = [
    path("", views.home, name="home"),
    path("articles/", views.article_list, name="article_list"),
    path("articles/mine/", views.my_articles, name="my_articles"),
    path("articles/new/", views.article_create, name="article_create"),
    path("articles/<int:pk>/", views.article_detail, name="article_detail"),
    path("articles/<int:pk>/edit/", views.article_update, name="article_update"),
    path("articles/<int:pk>/delete/", views.article_delete, name="article_delete"),
]
```

### Step 4 — Create three templates

**`journal/templates/journal/my_articles.html`**

```html
{% extends "journal/base.html" %}
{% block title %}My articles{% endblock %}
{% block content %}
<h1>My articles</h1>
<p><a class="button" href="{% url 'journal:article_create' %}">Write an article</a></p>
{% for article in articles %}
    <article class="card">
        <h2>{{ article.title }}</h2>
        <p>Status: {% if article.is_published %}Published{% else %}Draft{% endif %}</p>
        <a href="{% url 'journal:article_update' article.pk %}">Edit</a>
        · <a href="{% url 'journal:article_delete' article.pk %}">Delete</a>
        {% if article.is_published %} · <a href="{{ article.get_absolute_url }}">View public page</a>{% endif %}
    </article>
{% empty %}
    <p>You have not written any articles yet.</p>
{% endfor %}
{% endblock %}
```

**`journal/templates/journal/article_form.html`** (used for **both** create and edit)

```html
{% extends "journal/base.html" %}
{% block title %}{{ page_heading }}{% endblock %}
{% block content %}
<h1>{{ page_heading }}</h1>
<p>Choose whether this article should be published. Drafts are visible only in your own article list and Admin.</p>
<form method="post" class="card form-card">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="button" type="submit">Save article</button>
    <a href="{% url 'journal:my_articles' %}">Cancel</a>
</form>
{% endblock %}
```

**`journal/templates/journal/article_confirm_delete.html`**

```html
{% extends "journal/base.html" %}
{% block title %}Delete article{% endblock %}
{% block content %}
<h1>Delete this article?</h1>
<p>You are about to delete <strong>{{ article.title }}</strong>.</p>
<form method="post">
    {% csrf_token %}
    <button class="button danger" type="submit">Confirm delete</button>
    <a href="{% url 'journal:my_articles' %}">Cancel</a>
</form>
{% endblock %}
```

### Step 5 — Add the new links to the header

In **`base.html`**, inside `{% if user.is_authenticated %}`, add two links **before** the `Hi, ...` span:

```html
        {% if user.is_authenticated %}
            <a href="{% url 'journal:my_articles' %}">My articles</a>
            <a href="{% url 'journal:article_create' %}">Write</a>
            <span class="muted">Hi, {{ user.username }}</span>
            ...the logout form stays here...
```

### Step 6 — Test like a professional tester

Start the server. Log in as **`student1`**.

| # | Do this | Expected |
|---|---|---|
| 1 | Click **Write** | An empty form with checkboxes for categories |
| 2 | Title `a`, body `short` → **Save** | Form comes back with **red errors** (your `clean_title` / `clean_body`!) — nothing saved |
| 3 | Title `Student One's Article`, a long body, tick 2 categories, tick **Is published** → **Save** | Redirected to **My articles**; status **Published** |
| 4 | Write another, **don't** publish | Shows **Draft** in My articles, but **not** on `/articles/` |
| 5 | Click **Edit** on the first article, change the title, save | New title in My articles |
| 6 | Hover over **Edit** and note the id in the address, e.g. `/articles/5/edit/` | Remember that id |
| 7 | **Log out**, log in as **`student2`**. Type `http://127.0.0.1:8000/articles/5/edit/` (student1's id) | **404 Page not found** |
| 8 | As student2, try `/articles/5/delete/` | **404** again |
| 9 | **Log out** completely. Open `http://127.0.0.1:8000/articles/new/` | Sent to the **login page**, with `?next=/articles/new/` in the address |
| 10 | Log in from that page | You land back on **/articles/new/** — that's what `next` does |
| 11 | As student1, **Delete** the draft → **Confirm delete** | Gone from My articles |

---

## Check

- [ ] Invalid data shows error messages and saves nothing.
- [ ] Valid data creates an article whose author is the logged-in user.
- [ ] Edit and delete work for **your own** articles.
- [ ] Another user gets **404** on your edit/delete URLs.
- [ ] Logged-out visitors are sent to the login page.

---

## What just happened?

### The form pattern (same as register in Lesson 12)

```text
GET  → show the form (empty, or filled with the article for editing)
POST → form = ArticleForm(request.POST)
       is_valid()? ── yes → save → redirect (Post-Redirect-Get)
                   └─ no  → show the form again WITH error messages
```

| Code | Meaning |
|---|---|
| `ArticleForm(request.POST)` | Fill the form with what the user sent |
| `ArticleForm(instance=article)` | Fill the form with an **existing** article (for editing) |
| `form.save(commit=False)` | Build the Article object **without** saving, so we can add the author first |
| `form.save_m2m()` | Save the categories. Many-to-many links need the article to have an id first, so they're saved after `article.save()` |
| `get_object_or_404(Article, pk=pk, author=request.user)` | SQL: `WHERE id = 5 AND author_id = <me>` → not mine = not found |
| `@login_required` | Not logged in → redirect to `LOGIN_URL` with `?next=...` |

### CRUD → HTTP → URL map

| Action | Method | URL |
|---|---|---|
| Create | GET form, POST to save | `/articles/new/` |
| Read (mine) | GET | `/articles/mine/` |
| Update | GET form, POST to save | `/articles/<id>/edit/` |
| Delete | GET "are you sure?", POST to delete | `/articles/<id>/delete/` |

### Good error handling — what your app now does

| Situation | What the user gets |
|---|---|
| Invalid form data | The same form again, with clear red messages next to the fields |
| Record doesn't exist | 404 page |
| Record belongs to someone else | 404 page (we don't even say it exists) |
| Not logged in | Redirect to the login page |
| Missing CSRF token | 403 page |

And with `DEBUG=False` on a live site, unexpected errors show a plain "Server Error (500)" page — never your code or secrets.

### CRUD is more than four buttons

A **professional** CRUD feature also needs: validation, permission checks, safe handling of missing records, correct relationships, and **tests** (Lesson 15). You now have all of these except the tests.

> Order of URLs matters! `articles/mine/` and `articles/new/` come **before** `articles/<int:pk>/`. (Here it would still work because `<int:...>` only matches numbers — but it's a good habit.)

---

## Try it yourself

1. Add a rule: the title must not be `"test"` (any case). *(Hint: in `clean_title`, `if title.lower() == "test": raise ...`)*
2. Show the article's categories on **My articles** using a `{% for %}` loop and `class="tag"`.
3. Why is it wrong to put `author` in the form's `fields`? Write one sentence.
4. Open **F12 → Network**, submit a valid article, and find the **POST** request with status **302**, followed by a **GET** to `/articles/mine/` with **200**. That's Post-Redirect-Get, live!

---

## If it does not work

| Error | Fix |
|---|---|
| `ModuleNotFoundError: No module named 'journal.forms'` | Create `journal/forms.py` (Step 1). |
| `NameError: name 'login_required' is not defined` | Missing import at the top of `views.py` (Step 2a). |
| `NoReverseMatch: Reverse for 'article_update' with arguments ...` | URL name typo, or you forgot `article.pk` in `{% url 'journal:article_update' article.pk %}`. |
| `ValueError: ... needs to have a value for field "id" before this many-to-many relationship can be used` | You called `form.save_m2m()` **before** `article.save()`. Swap them. |
| `IntegrityError: null value in column "author_id"` | You forgot `article.author = request.user` before `article.save()`. |
| Categories don't save | You forgot `form.save_m2m()` after `commit=False`. |
| 403 CSRF on save | `{% csrf_token %}` missing in the form template. |

Compare with the answers: `django_lab/journal/forms.py`, `django_lab/journal/views.py`, `django_lab/journal/urls.py`.

---

**Next:** [14_PROFILES_AND_ROLES.md](14_PROFILES_AND_ROLES.md)
