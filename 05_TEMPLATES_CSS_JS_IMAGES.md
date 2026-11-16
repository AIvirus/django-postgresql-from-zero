# Lesson 05 — Templates, CSS, JavaScript and Images

**Time:** 30 minutes  |  **You need:** Lesson 04 finished

---

## Goal

By the end of this lesson your home page will be a **real-looking web page** with:

- an HTML **template** (instead of plain text),
- a shared **base layout** (header + footer) that every page reuses,
- a **CSS** file (colours and layout),
- a **JavaScript** file (a button that does something),
- an **image**,
- a value sent **from Python into the HTML**.

---

## Understand

### What is a template?

A **template** is an HTML file with **gaps** that Django fills in.

```html
<h1>Hello, {{ student_name }}!</h1>
```

The view sends `{"student_name": "Asha"}` → Django fills the gap → the browser gets `<h1>Hello, Asha!</h1>`.

The dictionary the view sends is called the **context**.

### Two kinds of template code

| Looks like | Name | What it does | Example |
|---|---|---|---|
| `{{ ... }}` | **variable** | **Shows** a value | `{{ student_name }}` |
| `{% ... %}` | **tag** | **Does** something (if, loop, load…) | `{% if %}`, `{% for %}`, `{% url %}` |

### Template inheritance (= don't repeat yourself)

Every page needs the same header and footer. Instead of copying them into every page:

- **`base.html`** has the header, footer and an **empty box** called `content`.
- Each page says `{% extends "journal/base.html" %}` and fills the box.

```text
base.html                       home.html
┌─────────── header ─────────┐   {% extends "journal/base.html" %}
│ {% block content %}        │ ◀─ {% block content %}
│     (empty box)            │       <h1>My page</h1>
│ {% endblock %}             │    {% endblock %}
└─────────── footer ─────────┘
```

Change the header once in `base.html` → every page changes.

### Static files

**Static files** are files that don't change per user: **CSS**, **JavaScript**, **images**. In a template we load them with:

```html
{% load static %}
<link rel="stylesheet" href="{% static 'journal/style.css' %}">
```

---

## Do it now

> Server running? Leave it running — it reloads by itself. Use a **second terminal** only if you need to type commands (remember to activate `.venv` and `cd my_journal` there).

### Step 1 — Create the folders

Inside `my_journal/journal/`, create these folders (right-click `journal` → **New Folder**):

```text
journal/
├── templates/
│   └── journal/        ← yes, "journal" again inside "templates"
└── static/
    └── journal/        ← and "journal" again inside "static"
```

> **Why `journal` twice?** Later we will have more apps. If two apps both have `home.html`, Django could pick the wrong one. Putting it in `templates/journal/home.html` gives it a unique name: `journal/home.html`.

You can also create all four folders from the terminal (from inside `my_journal`):

```bash
mkdir journal/templates/journal
```
```bash
mkdir journal/static/journal
```

(On Windows PowerShell this creates the parent folders too. On macOS/Linux add `-p`: `mkdir -p journal/templates/journal`.)

### Step 2 — Create the base layout

Create **`journal/templates/journal/base.html`**:

```html
{% load static %}
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Django Learning Journal{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'journal/style.css' %}">
</head>
<body>
<header class="site-header">
    <a class="brand" href="{% url 'journal:home' %}">Learning Journal</a>
    <nav>
        <!-- We will add more links here in later lessons -->
    </nav>
</header>
<main class="container">
    {% block content %}{% endblock %}
</main>
<footer class="site-footer">Django Day 2 practical learning application</footer>
</body>
</html>
```

> `{% url 'journal:home' %}` builds the link to the URL named `home` in the `journal` app (you named it in Lesson 04). If you change the address later, the link still works.

### Step 3 — Create the home page template

Create **`journal/templates/journal/home.html`**:

```html
{% extends "journal/base.html" %}
{% load static %}

{% block title %}Home | Learning Journal{% endblock %}

{% block content %}
<section class="hero">
    <p class="eyebrow">DJANGO + POSTGRESQL PRACTICAL</p>
    <img class="learning-image" src="{% static 'journal/learning-mark.svg' %}" alt="Notebook illustration">
    <h1>Hello, {{ student_name }}! Let's learn how a real Django website works.</h1>
    <p>Today you will build articles, categories, user accounts, forms and a PostgreSQL database.</p>
    <p data-demo-message>Press the button to see a small JavaScript interaction.</p>
    <button type="button" data-demo-button>Try JavaScript</button>
</section>
<script src="{% static 'journal/app.js' %}" defer></script>
{% endblock %}
```

### Step 4 — Create the CSS file

Create **`journal/static/journal/style.css`** and paste:

```css
* { box-sizing: border-box; }
body { margin: 0; font-family: Arial, sans-serif; color: #1f2937; background: #f4f6f9; line-height: 1.6; }
a { color: #1557a0; }
.site-header { display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center; gap: 1rem; padding: 1rem 5%; background: #fff; border-bottom: 1px solid #dce2ea; }
.site-header nav { display: flex; flex-wrap: wrap; align-items: center; gap: 1rem; }
.brand { font-weight: 700; font-size: 1.1rem; text-decoration: none; color: #17243a; }
.container { max-width: 960px; margin: 2rem auto; padding: 0 1rem; min-height: 68vh; }
.hero, .card { background: #fff; border: 1px solid #e0e5ec; border-radius: 12px; padding: 1.5rem; margin-bottom: 1rem; }
.hero { padding: 2.5rem; }
.eyebrow { letter-spacing: .12em; font-size: .8rem; font-weight: bold; color: #53657d; }
.learning-image { width: 120px; height: auto; }
.button, button { display: inline-block; border: 0; border-radius: 7px; padding: .65rem 1rem; background: #174f91; color: white; text-decoration: none; cursor: pointer; font: inherit; }
.danger { background: #a52a2a; }
.link-button { background: transparent; color: #1557a0; padding: 0; }
.inline-form { display: inline; margin: 0; }
input, textarea, select { display: block; width: 100%; max-width: 100%; padding: .6rem; border: 1px solid #aeb8c5; border-radius: 6px; margin: .25rem 0 .8rem; }
label { font-weight: bold; }
.tag { display: inline-block; padding: .15rem .5rem; margin: .1rem; border-radius: 1rem; background: #e7eef8; font-size: .85rem; }
.muted, small { color: #667085; }
.subtitle { font-size: 1.15rem; }
.errorlist { color: #a52a2a; }
.site-footer { padding: 1.5rem; text-align: center; color: #667085; }
.filter-form { background: white; padding: 1rem; border-radius: 10px; margin-bottom: 1rem; }
```

(Styling is not today's main topic, so it's OK to paste this one.)

### Step 5 — Create the JavaScript file

Create **`journal/static/journal/app.js`**:

```javascript
document.addEventListener("DOMContentLoaded", () => {
    const button = document.querySelector("[data-demo-button]");
    const message = document.querySelector("[data-demo-message]");
    if (button && message) {
        button.addEventListener("click", () => {
            message.textContent = "Great! Your JavaScript file is connected and responding.";
        });
    }
});
```

### Step 6 — Create the image

Create **`journal/static/journal/learning-mark.svg`** (an SVG image is just text, so you can type it):

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="160" height="100" viewBox="0 0 160 100">
  <rect width="160" height="100" rx="14" fill="#e7eef8"/>
  <path d="M35 22h90v60H35z" fill="#fff" stroke="#174f91" stroke-width="3"/>
  <path d="M49 39h62M49 52h52M49 65h57" stroke="#174f91" stroke-width="4" stroke-linecap="round"/>
</svg>
```

Your folders should now look like this:

```text
journal/
├── static/journal/
│   ├── app.js
│   ├── learning-mark.svg
│   └── style.css
└── templates/journal/
    ├── base.html
    └── home.html
```

### Step 7 — Make the view use the template

Open **`journal/views.py`**. Replace everything with:

```python
from django.shortcuts import render


def home(request):
    context = {"student_name": "Asha"}
    return render(request, "journal/home.html", context)
```

> Did you make the **about** page in Lesson 04? You can delete it now, or keep it — but if you keep it, keep its `HttpResponse` import too.

### Step 8 — Look at it

If the server is not running: `python manage.py runserver`.
Open **http://127.0.0.1:8000/** (press **Ctrl + F5** to fully reload).

---

## Check

- [ ] White header with **"Learning Journal"** at the top, and a footer at the bottom.
- [ ] The heading says **"Hello, Asha! Let's learn…"**
- [ ] A small blue **notebook image** is visible.
- [ ] Clicking **Try JavaScript** changes the text to **"Great! Your JavaScript file is connected…"**

---

## What just happened?

```text
Browser: GET /
  → views.home builds context {"student_name": "Asha"}
  → render() opens journal/home.html
      → home.html says "extends base.html" → Django puts home's content into base's box
      → {{ student_name }} is replaced with "Asha"
      → {% static ... %} becomes /static/journal/style.css etc.
  → Browser receives HTML, then asks for style.css, app.js, learning-mark.svg (3 more GET requests!)
```

Open **F12 → Network** and reload: you will see the page request **plus** separate requests for `style.css`, `app.js` and `learning-mark.svg`.

- **Python decides WHAT to show** (the view + context).
- **The template decides HOW it looks** (HTML).
- **CSS** = looks, **JavaScript** = actions in the browser. The JavaScript button does **not** touch the database — it only runs inside the browser.

**Safety bonus:** Django automatically makes `{{ }}` values safe. If someone types `<script>` as their name, Django shows it as text instead of running it. This protects against an attack called **XSS**.

---

## Try it yourself

1. Change `"Asha"` in the view to **your own name**. Refresh.
2. In `style.css`, change `background: #f4f6f9;` (in `body`) to `background: lightyellow;`. Refresh with **Ctrl + F5**. Change it back.
3. Add a second value to the context: `"topic": "Templates"`, and show it in `home.html` with `<p>Today's topic: {{ topic }}</p>`.
4. Try an `if` tag in `home.html`:
   ```html
   {% if student_name == "Asha" %}
       <p>You are using the example name. Change it to yours!</p>
   {% endif %}
   ```

---

## If it does not work

| Problem | Fix |
|---|---|
| `TemplateDoesNotExist at / journal/home.html` | The file is in the wrong folder. It must be **`journal/templates/journal/home.html`**. Check `"journal"` is in `INSTALLED_APPS`. Restart the server (**Ctrl + C**, then `runserver`) after creating new folders. |
| Page shows but has **no colours** | Wrong path for CSS. Must be `journal/static/journal/style.css`. Press **Ctrl + F5**. Restart the server. |
| `TemplateSyntaxError: Invalid block tag 'static'` | You forgot `{% load static %}` at the top of the template. |
| `NoReverseMatch: 'journal' is not a registered namespace` | `app_name = "journal"` is missing in `journal/urls.py`. |
| Button does nothing | Check the file name `app.js` and the `<script>` line. Press **Ctrl + F5**. |

Compare with the answer: `django_lab/journal/templates/journal/base.html` (the finished version has more links — you'll add them later).

---

**Next:** [06_CREATE_POSTGRESQL_DATABASE_AND_CONNECT.md](06_CREATE_POSTGRESQL_DATABASE_AND_CONNECT.md)
