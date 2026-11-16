# Lesson 12 — Register, Log In, Log Out

**Time:** 35 minutes  |  **You need:** Lesson 11 finished

---

## Goal

By the end of this lesson visitors can:

- **register** a new account at `/users/register/`,
- **log in** at `/accounts/login/`,
- **log out** with a button in the header,
- and the header changes depending on whether you're logged in.

You will also **see** the scrambled (hashed) password in PostgreSQL and the **session cookie** in your browser.

---

## Understand

### Authentication = "Who are you?"

**Authentication** means checking someone's identity — usually username + password. (Next lessons: **authorization** = "What are you **allowed** to do?")

Django already has login, logout and password tools built in. We just **connect** them. We never write our own password system — that's how security bugs happen.

### Passwords are never stored as text

Django stores a **hash** — a one-way scramble:

```text
You type:           MySecret123
Database stores:    pbkdf2_sha256$1000000$Xy7...$k8Jh2...   ← can't be turned back into MySecret123
```

When you log in, Django scrambles what you typed **the same way** and compares the two scrambles. Even the website owner can't read your password.

### How does the site "remember" you? Sessions and cookies

HTTP has no memory — every request is new. So after you log in:

1. Django creates a **session** (a row in the `django_session` table) saying "this visitor is user #3".
2. Django gives your browser a **cookie** called `sessionid` — like a **cloakroom ticket**.
3. The browser shows that ticket on every next request. Django looks it up → "Ah, user #3!"

Logging out **tears up** the ticket.

### Why is "Log out" a button inside a form?

Logging out **changes** something on the server, so it must be a **POST** (Lesson 01). A plain link would be a GET. Django requires POST for logout.

### `{% csrf_token %}` — the anti-trick token

Every form that sends **POST** must contain `{% csrf_token %}`. It adds a secret one-time code to the form, so **another website can't secretly submit forms on your behalf** while you're logged in. (This attack is called **CSRF** — Cross-Site Request Forgery.) If you forget it, Django blocks the form with **403 Forbidden**.

---

## Do it now

### Step 1 — Connect Django's built-in login/logout URLs and our accounts URLs

Open **`config/urls.py`**. Make it:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("django.contrib.auth.urls")),
    path("users/", include("accounts.urls")),
    path("", include("journal.urls")),
]
```

> `django.contrib.auth.urls` gives you, for free: `/accounts/login/` (named `login`), `/accounts/logout/` (named `logout`) and password-change pages.

### Step 2 — Tell Django where to go after login/logout

At the **bottom** of **`config/settings.py`**, add:

```python
LOGIN_URL = "login"
LOGIN_REDIRECT_URL = "journal:article_list"
LOGOUT_REDIRECT_URL = "journal:home"
```

| Setting | Meaning |
|---|---|
| `LOGIN_URL` | Where to send people who must log in first |
| `LOGIN_REDIRECT_URL` | Where to go after a successful login |
| `LOGOUT_REDIRECT_URL` | Where to go after logout |

### Step 3 — Write the register view

Open **`accounts/views.py`**. Replace everything with:

```python
from django.contrib.auth import login
from django.contrib.auth.forms import UserCreationForm
from django.shortcuts import redirect, render

from .models import Profile


def register(request):
    if request.user.is_authenticated:
        return redirect("journal:article_list")

    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            Profile.objects.get_or_create(user=user)
            login(request, user)
            return redirect("journal:article_list")
    else:
        form = UserCreationForm()

    return render(request, "registration/register.html", {"form": form})
```

### Step 4 — Create `accounts/urls.py`

Create a new file **`accounts/urls.py`**:

```python
from django.urls import path

from . import views

app_name = "accounts"

urlpatterns = [
    path("register/", views.register, name="register"),
]
```

### Step 5 — Create the login and register templates

Create the folders **`accounts/templates/registration/`** (from inside `my_journal`):

```bash
mkdir accounts/templates/registration
```

Create **`accounts/templates/registration/login.html`**:

```html
{% extends "journal/base.html" %}
{% block title %}Log in{% endblock %}
{% block content %}
<h1>Log in</h1>
<form method="post" class="card form-card">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="hidden" name="next" value="{{ next }}">
    <button class="button" type="submit">Log in</button>
</form>
{% endblock %}
```

Create **`accounts/templates/registration/register.html`**:

```html
{% extends "journal/base.html" %}
{% block title %}Register{% endblock %}
{% block content %}
<h1>Create an account</h1>
<form method="post" class="card form-card">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="button" type="submit">Create account</button>
</form>
{% endblock %}
```

> Django's login view looks for a template called **`registration/login.html`** by default — that's why we use that folder name.

### Step 6 — Update the header

In **`journal/templates/journal/base.html`**, replace the `<nav>...</nav>` part with:

```html
    <nav>
        <a href="{% url 'journal:article_list' %}">Articles</a>
        {% if user.is_authenticated %}
            <span class="muted">Hi, {{ user.username }}</span>
            <form class="inline-form" method="post" action="{% url 'logout' %}">
                {% csrf_token %}
                <button type="submit" class="link-button">Log out</button>
            </form>
        {% else %}
            <a href="{% url 'login' %}">Log in</a>
            <a href="{% url 'accounts:register' %}">Register</a>
        {% endif %}
    </nav>
```

> `user` is available in **every** template automatically. `user.is_authenticated` is `True` when someone is logged in.

### Step 7 — Try it all

**Restart** the server (**Ctrl + C**, then `python manage.py runserver`) — needed because you made new template folders.

1. If you are logged into Admin, click **Log out** in the header first.
2. Click **Register**. Create a user `student1` with a password.
   - Try a weak password like `12345` first → Django shows the rules it broke. That's server-side validation.
   - Then use a strong one.
3. After registering you land on **Articles**, and the header says **Hi, student1** + **Log out**.
4. Click **Log out** → you're back on the home page, header shows **Log in / Register**.
5. Click **Log in** → log in as `student1` → you land on **Articles**.

### Step 8 — Look at the hashed password

Neon → **SQL Editor**:

```sql
SELECT id, username, password, is_superuser FROM auth_user;
```

The `password` column shows `pbkdf2_sha256$...` — **not** your real password.

### Step 9 — Look at your session cookie

While logged in: **F12 → Application** tab (Edge/Chrome) → **Cookies** → `http://127.0.0.1:8000` → you'll see **`sessionid`** and **`csrftoken`**.

Then in Neon:

```sql
SELECT session_key, expire_date FROM django_session;
```

One of the `session_key` values matches your `sessionid` cookie! Log out and run it again — that session row is **gone**.

---

## Check

- [ ] You registered a new user and were logged in automatically.
- [ ] The header changes between **Log in / Register** and **Hi, name / Log out**.
- [ ] Log out and log in both work.
- [ ] `auth_user.password` shows a hash, not a real password.
- [ ] You found the `sessionid` cookie.

---

## What just happened?

The **register** view is the pattern you will use for **every** form:

```text
GET  /users/register/  →  show an EMPTY form
POST /users/register/  →  fill the form with request.POST
                          ├─ valid?   → save → log in → REDIRECT to /articles/
                          └─ invalid? → show the SAME form again, with error messages
```

| Code | Meaning |
|---|---|
| `UserCreationForm` | Django's ready-made sign-up form (username + password twice + password rules) |
| `form.is_valid()` | Run all the checks (passwords match? strong enough? username taken?) |
| `form.save()` | Create the user — with the password **hashed** |
| `Profile.objects.get_or_create(user=user)` | Give every new user a profile (One-to-One!) |
| `login(request, user)` | Create the session + cookie |
| `redirect(...)` | Send the browser to another page (status **302**) |

Why redirect after a successful POST? If you pressed **F5** on a page made by a POST, the browser would **send the form again**. Redirecting first prevents that. This is called **Post → Redirect → Get**.

---

## Try it yourself

1. **CSRF test:** remove `{% csrf_token %}` from `register.html`, save, and try to register → **403 Forbidden — CSRF verification failed**. Put it back!
2. Try to register `student1` **again** → *"A user with that username already exists."*
3. In Admin → **Users**, find `student1`. Is **Staff status** ticked? (No — normal users can't open Admin. Try http://127.0.0.1:8000/admin/ as `student1`.)
4. In Admin → **Profiles** — was a profile created automatically for `student1`?
5. Register a **second** user `student2`. You'll need two users in the next lesson.

---

## If it does not work

| Problem | Fix |
|---|---|
| `TemplateDoesNotExist: registration/login.html` | File must be `accounts/templates/registration/login.html`. `"accounts"` must be in `INSTALLED_APPS`. **Restart** the server. |
| `NoReverseMatch: 'accounts' is not a registered namespace` | `app_name = "accounts"` missing in `accounts/urls.py`, or `path("users/", include("accounts.urls"))` missing in `config/urls.py`. |
| `NoReverseMatch: Reverse for 'logout' not found` | `path("accounts/", include("django.contrib.auth.urls"))` is missing. |
| Log out gives **405 Method Not Allowed** | Your logout is a link (GET). It must be the `<form method="post">` from Step 6. |
| **403 CSRF verification failed** | Add `{% csrf_token %}` inside the `<form>`. |
| After login you go to `/accounts/profile/` (404) | `LOGIN_REDIRECT_URL` is missing from `settings.py`. |

Compare with the answers: `django_lab/config/urls.py`, `django_lab/accounts/views.py` (the `register` function), `django_lab/accounts/templates/registration/`.

---

**Next:** [13_FORMS_CREATE_EDIT_DELETE.md](13_FORMS_CREATE_EDIT_DELETE.md)
