# Lesson 14 — Profiles and Roles (Who Is Allowed to Do What?)

**Time:** 30 minutes  |  **You need:** Lesson 13 finished

---

## Goal

By the end of this lesson:

- every user has a **profile page** they can edit (`/users/profile/`),
- there is a secret **"Editors only"** page (`/users/editor-area/`) that only members of the **`ContentEditors`** group (and superusers) can open,
- you can explain **authentication vs authorization**, **ownership** and **roles**.

---

## Understand

### Authentication vs Authorization

| | Question | Example |
|---|---|---|
| **Authentication** | **Who** are you? | Logging in with username + password (Lesson 12) |
| **Authorization** | What are you **allowed** to do? | Only the author can edit an article (Lesson 13) · Only editors can open the editor area (today) |

Being logged in does **not** mean you can do everything.

### Two common kinds of authorization

1. **Ownership** — "It's **mine**, so I can change it." (You did this: `author=request.user`.)
2. **Roles** — "I'm a **teacher**, so I can enter the staff room." Django uses **Groups** for roles: a Group is a named list of users, e.g. `ContentEditors`.

### Least privilege

Give each person **only** the access they need. Don't make everyone a superuser — just like not every student gets the master key to the school.

---

## Do it now

### Step 1 — Profile form

Create **`accounts/forms.py`**:

```python
from django import forms

from .models import Profile


class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ["display_name", "bio"]
        widgets = {"bio": forms.Textarea(attrs={"rows": 4})}

    def clean_display_name(self):
        return self.cleaned_data["display_name"].strip()
```

### Step 2 — Profile and editor views

Open **`accounts/views.py`**.

**a)** Replace the imports at the top with:

```python
from django.contrib.auth import login
from django.contrib.auth.decorators import login_required, user_passes_test
from django.contrib.auth.forms import UserCreationForm
from django.shortcuts import redirect, render

from .forms import ProfileForm
from .models import Profile
```

**b)** Keep `register` as it is. **Add** at the bottom:

```python
@login_required
def profile_view(request):
    profile, _ = Profile.objects.get_or_create(user=request.user)
    return render(request, "accounts/profile.html", {"profile": profile})


@login_required
def profile_edit(request):
    profile, _ = Profile.objects.get_or_create(user=request.user)
    if request.method == "POST":
        form = ProfileForm(request.POST, instance=profile)
        if form.is_valid():
            form.save()
            return redirect("accounts:profile")
    else:
        form = ProfileForm(instance=profile)
    return render(request, "accounts/profile_form.html", {"form": form})


def is_content_editor(user):
    return user.is_authenticated and (
        user.is_superuser or user.groups.filter(name="ContentEditors").exists()
    )


@user_passes_test(is_content_editor)
def editor_area(request):
    return render(request, "accounts/editor_area.html")
```

### Step 3 — URLs

Make `urlpatterns` in **`accounts/urls.py`**:

```python
urlpatterns = [
    path("register/", views.register, name="register"),
    path("profile/", views.profile_view, name="profile"),
    path("profile/edit/", views.profile_edit, name="profile_edit"),
    path("editor-area/", views.editor_area, name="editor_area"),
]
```

### Step 4 — Templates

Create the folder **`accounts/templates/accounts/`**:

```bash
mkdir accounts/templates/accounts
```

**`accounts/templates/accounts/profile.html`**

```html
{% extends "journal/base.html" %}
{% block title %}My profile{% endblock %}
{% block content %}
<h1>My profile</h1>
<p><strong>Username:</strong> {{ request.user.username }}</p>
<p><strong>Display name:</strong> {{ profile.display_name|default:"Not added" }}</p>
<p><strong>About:</strong> {{ profile.bio|default:"No biography added" }}</p>
<a class="button" href="{% url 'accounts:profile_edit' %}">Edit profile</a>
{% endblock %}
```

**`accounts/templates/accounts/profile_form.html`**

```html
{% extends "journal/base.html" %}
{% block title %}Edit profile{% endblock %}
{% block content %}
<h1>Edit profile</h1>
<form method="post" class="card form-card">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="button" type="submit">Save profile</button>
</form>
{% endblock %}
```

**`accounts/templates/accounts/editor_area.html`**

```html
{% extends "journal/base.html" %}
{% block title %}Editor area{% endblock %}
{% block content %}
<h1>Content editor area</h1>
<p>You can see this page because your account belongs to the ContentEditors group or is a superuser.</p>
{% endblock %}
```

### Step 5 — Final header

Replace the whole `<nav>...</nav>` in **`base.html`** with the **final** version:

```html
    <nav>
        <a href="{% url 'journal:article_list' %}">Articles</a>
        {% if user.is_authenticated %}
            <a href="{% url 'journal:my_articles' %}">My articles</a>
            <a href="{% url 'journal:article_create' %}">Write</a>
            <a href="{% url 'accounts:profile' %}">Profile</a>
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

### Step 6 — Test the profile

**Restart** the server (new template folder). Log in as `student1`.

1. Click **Profile** → shows *Not added*.
2. **Edit profile** → Display name `Student One`, a short bio → **Save profile**.
3. You're back on **Profile** with your new info.

### Step 7 — Test the role (group)

1. Still as `student1`, open **http://127.0.0.1:8000/users/editor-area/** → you are sent to the **login page**. Not allowed.
2. **Log out.** Log in to **http://127.0.0.1:8000/admin/** as your **superuser**.
3. **Groups → Add group** → Name: **`ContentEditors`** (exactly, capital C and E) → **Save**.
4. **Users → student1** → scroll to **Groups** → move `ContentEditors` to **Chosen groups** → **Save**.
5. **Log out**, log in on the website as `student1`, open **/users/editor-area/** → **"Content editor area"**!
6. In Admin (as superuser), remove `student1` from the group. Log in as `student1` again and try the page → blocked again.
7. Your **superuser** can always open it (because of `user.is_superuser` in `is_content_editor`).

---

## Check

- [ ] Profile page shows and saves your display name and bio.
- [ ] `student1` **without** the group can't open `/users/editor-area/`.
- [ ] `student1` **with** the `ContentEditors` group can open it.
- [ ] The header shows **Articles · My articles · Write · Profile · Hi, name · Log out** when logged in.

---

## What just happened?

| Code | Meaning |
|---|---|
| `profile, _ = Profile.objects.get_or_create(user=request.user)` | Get my profile, or make one if missing. `_` means "I don't need the second value" (the True/False "was it created?") |
| `ProfileForm(request.POST, instance=profile)` | Update **my** profile — always looked up by `request.user`, so you can never edit someone else's |
| `user.groups.filter(name="ContentEditors").exists()` | "Is this user in that group?" → True / False |
| `@user_passes_test(is_content_editor)` | Run my check function first. `False` → send to the login page |

Three levels of protection you now have:

```text
@login_required                   → must be logged in             (authentication)
get_object_or_404(..., author=me) → must OWN the record            (ownership)
@user_passes_test(is_editor)      → must have the ROLE             (role-based access)
```

All three run **on the server**. That is what makes them real security.

> In bigger projects you can also give **permissions** to groups (e.g. "can delete article") in Admin, and check them with `user.has_perm("journal.delete_article")`. Same idea, more detail.

---

## Try it yourself

1. Show the display name instead of the username in the header: `Hi, {{ user.profile.display_name|default:user.username }}`. (One-to-One backwards: `user.profile`!)
2. Add a link **Editor area** in the header that only shows for editors. *(Hint: you can't call `is_content_editor` in a template, but `{% if user.is_superuser %}` works — try it for superusers.)*
3. Make the editor area show a list of **all draft articles** from every author. (Edit the `editor_area` view: pass `Article.objects.filter(is_published=False)` and loop over it. Import `Article` from `journal.models`.)

---

## If it does not work

| Problem | Fix |
|---|---|
| `TemplateDoesNotExist: accounts/profile.html` | File must be `accounts/templates/accounts/profile.html`. Restart the server. |
| Editor area still blocked after adding the group | Group name must be **exactly** `ContentEditors`. Log out and in again. |
| `RelatedObjectDoesNotExist: User has no profile` (in the header try-it) | That user has no profile yet — open the Profile page once (it creates one), or use `get_or_create`. |
| `NameError: name 'user_passes_test' is not defined` | Check the imports in Step 2a. |

Compare with the answers: `django_lab/accounts/` (all files) and `django_lab/journal/templates/journal/base.html`.

**Your website is now feature-complete — it should behave exactly like `django_lab`.**

---

**Next:** [15_AUTOMATED_TESTS.md](15_AUTOMATED_TESTS.md)
