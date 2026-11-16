# Lesson 04 — Your First App and Your First Page

**Time:** 25 minutes  |  **You need:** Lesson 03 finished

---

## Goal

By the end of this lesson your website will show **your own message** at http://127.0.0.1:8000/ instead of the rocket.

You will learn the three things every Django page needs:

1. a **view** (a Python function that makes the answer),
2. a **URL** (the address that calls the view),
3. connecting the app's URLs to the project.

---

## Understand

### What is a view?

A **view** is a normal Python function. Django gives it the **request** (what the browser asked for), and the view must **return a response** (what to send back).

```python
def home(request):                  # Django gives us the request
    return HttpResponse("Hello!")   # we send back a response
```

### What is a URL pattern?

A **URL pattern** connects an address to a view. Like a school reception desk: "You want *Room 5*? Go this way."

```python
path("", views.home, name="home")
#    │        │            └── a nickname for this URL (we use it later)
#    │        └── which view to call
#    └── the address ("" means the home page: http://127.0.0.1:8000/)
```

### Two URL files

- `config/urls.py` = the **main reception** of the whole website.
- `journal/urls.py` = the **reception of one app**.

The main reception sends visitors to the app's reception using `include()`.

---

## Do it now

> Terminal check: `(.venv)` at the start, `my_journal>` at the end. If the server is running, stop it with **Ctrl + C**.

### Step 1 — Create the `journal` app

```bash
python manage.py startapp journal
```

A new `journal` folder appears with files like `views.py`, `models.py`, `admin.py`.

### Step 2 — Tell the project about the new app

Open **`config/settings.py`**. Find `INSTALLED_APPS` and add `"journal",` at the end:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "journal",                      # ADD THIS LINE
]
```

> Your file may use single quotes `'...'` — that's fine, both work in Python. Don't forget the **comma**.

Save the file (**Ctrl + S**).

### Step 3 — Write your first view

Open **`journal/views.py`**. Delete everything in it and type:

```python
from django.http import HttpResponse


def home(request):
    return HttpResponse("Hello! This page was made by MY Django view.")
```

Save.

### Step 4 — Create the app's URL file

Inside the `journal` folder, create a **new file** called **`urls.py`**
(right-click the `journal` folder → **New File** → type `urls.py`).

Type this into it:

```python
from django.urls import path

from . import views

app_name = "journal"

urlpatterns = [
    path("", views.home, name="home"),
]
```

Save.

### Step 5 — Connect the app's URLs to the main URL file

Open **`config/urls.py`**. Delete everything and type:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("journal.urls")),
]
```

Save.

### Step 6 — Run it!

```bash
python manage.py runserver
```

Open **http://127.0.0.1:8000/**

---

## Check

- [ ] The browser shows: **Hello! This page was made by MY Django view.**
- [ ] The rocket page is gone.

You just made your first Django page!

---

## What just happened?

Follow the request step by step:

```text
1. Browser:  GET http://127.0.0.1:8000/
2. config/urls.py:   path "" → include("journal.urls")  → "go to the journal app"
3. journal/urls.py:  path "" → views.home               → "call the home function"
4. journal/views.py: home(request) returns HttpResponse("Hello! ...")
5. Browser shows the text.
```

- `startapp journal` made a new app (a feature area for our articles).
- Adding `"journal"` to `INSTALLED_APPS` told Django the app exists.
- `app_name = "journal"` gives the app's URLs a **family name**. The home page's full nickname is now **`journal:home`**. We will use this in templates later.

### MVT — the Django pattern

Django code is organised in three parts. Today you made the **V**:

| Letter | Name | Job | File |
|---|---|---|---|
| **M** | Model | Describes the **data** (Lesson 07) | `models.py` |
| **V** | View | Decides **what to do** with a request | `views.py` |
| **T** | Template | The **HTML page** the user sees (Lesson 05) | `templates/` |

> You may hear about **MVC** (Model–View–Controller) in other frameworks. It's the same idea with different names: Django's **URLs + view** together do the job other frameworks call the "controller". Don't worry about the names — remember the jobs: **data**, **logic**, **page**.

**Why split a project into apps?** If everything is in one giant file, it becomes hard to find, test and change. Each app owns one feature: today `journal` (articles) and, from Lesson 09, `accounts` (profiles). A bigger project might have `courses`, `payments`, `reports`…

### A view can do more than return text

| A view can… | You'll do it in |
|---|---|
| Return plain text (`HttpResponse`) | this lesson |
| Show an HTML template (`render`) | Lesson 05 |
| Read from the database | Lesson 11 |
| Read what the user typed (`request.GET`, `request.POST`) | Lessons 11, 13 |
| Save to the database | Lesson 13 |
| Send the browser elsewhere (`redirect`) | Lesson 12 |
| Return a 404 page (`get_object_or_404`) | Lesson 11 |

A view **must always return a response**. If it returns nothing, Django shows an error: *"The view didn't return an HttpResponse object."* The kind of view you write today (a normal Python function) is called a **Function-Based View** (FBV).

---

## Try it yourself

Make an **About** page at http://127.0.0.1:8000/about/

1. In `journal/views.py`, add a second function:
   ```python
   def about(request):
       return HttpResponse("This website is my Day 2 Django project.")
   ```
2. In `journal/urls.py`, add a line inside `urlpatterns`:
   ```python
   path("about/", views.about, name="about"),
   ```
3. Save both files. (The server reloads by itself — no need to restart.)
4. Open http://127.0.0.1:8000/about/

**Bonus — see a 404 page:** open http://127.0.0.1:8000/banana/. Django shows a yellow **"Page not found (404)"** page that lists all the URLs your site knows. That list is very helpful when you are stuck. (You only see this helpful page while `DEBUG` is on, which is only for your own computer.)

**Bonus 2:** Open **F12 → Network** like in Lesson 01, reload your home page, and check: method **GET**, status **200**.

---

## If it does not work

| Error message | What it means | Fix |
|---|---|---|
| `ModuleNotFoundError: No module named 'journal.urls'` | The file `journal/urls.py` doesn't exist or has a different name | Create it in Step 4. Check the spelling. |
| `AttributeError: module 'journal.views' has no attribute 'home'` | The function name in `views.py` doesn't match `urls.py` | Make sure it's `def home(request):` |
| `NameError: name 'include' is not defined` | Missing import | First line of `config/urls.py` must import `include`: `from django.urls import include, path` |
| The page still shows the rocket | Files not saved, or `config/urls.py` not changed | Save all files (**File → Save All**). |
| `IndentationError` | Spaces are wrong | The `return` line must be indented with **4 spaces** under `def`. |

Still stuck? Compare with the answer files: `django_lab/journal/urls.py` and `django_lab/config/urls.py` (the answer files have more lines because they are finished — focus on the lines you just wrote).

---

**Next:** [05_TEMPLATES_CSS_JS_IMAGES.md](05_TEMPLATES_CSS_JS_IMAGES.md)
