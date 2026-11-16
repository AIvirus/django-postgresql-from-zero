# Lesson 02 — Set Up Your Computer

**Time:** 20 minutes  |  **You need:** Python, VS Code, internet

---

## Goal

By the end of this lesson:

- Python works in your terminal,
- you have a **virtual environment** (a private box for this course's packages),
- Django and the other packages are installed.

---

## Understand

### What is a terminal?

A **terminal** is a window where you type commands instead of clicking. VS Code has one built in.

### What is `pip`?

`pip` is Python's **app store**. It downloads **packages** — code written by other people that we can reuse. **Django is a package.**

### What is a virtual environment (`.venv`)?

Imagine each school subject has its own bag. Your maths bag has maths books; your science bag has science books. They never get mixed up.

A **virtual environment** is a separate "bag" of Python packages for **one** project. The packages for this course stay inside the `.venv` folder and never mix with other projects on your computer.

### What is `requirements.txt`?

It is a **shopping list** of packages. Instead of installing six packages one by one, you give pip the whole list:

```bash
pip install -r requirements.txt
```

The `requirements.txt` for this course is in the **main course folder** (`Day-02-Django`). That is why you must run that command **from the `Day-02-Django` folder**.

---

## Do it now

### Step 1 — Install the tools (skip what you already have)

1. **Python** — https://www.python.org/downloads/ → install **3.12** (3.11 or 3.13 also work).
   - **Windows:** on the first installer screen, tick **"Add python.exe to PATH"**. This is very important.
2. **VS Code** — https://code.visualstudio.com/
3. In VS Code, open the **Extensions** panel (the four-squares icon on the left), search **Python**, and install the one by **Microsoft**.

### Step 2 — Open the course folder in VS Code

1. In VS Code: **File → Open Folder…**
2. Select the **`Day-02-Django`** folder (the one that contains `00_START_HERE.md` and `requirements.txt`).
3. Click **Select Folder**. If VS Code asks "Do you trust the authors?", click **Yes**.

### Step 3 — Open a terminal

In VS Code: **Terminal → New Terminal**. A panel opens at the bottom.

The terminal line should end with `Day-02-Django>`. That means you are in the right folder.

### Step 4 — Check Python

Type this and press **Enter**:

```bash
python --version
```

You should see something like `Python 3.12.8`.

> Didn't work? On **Windows** try `py --version`. On **macOS/Linux** try `python3 --version`.
> If one of those works, use that word (`py` or `python3`) instead of `python` in the next step only.

### Step 5 — Create the virtual environment

```bash
python -m venv .venv
```

Wait a few seconds. A new folder called **`.venv`** appears in the Explorer on the left.

> Already have a `.venv` folder from an earlier try? That's fine, skip this step.

### Step 6 — Turn the virtual environment ON (activate it)

Pick the command for **your** computer:

**Windows — PowerShell** (the VS Code default on Windows):
```powershell
.venv\Scripts\Activate.ps1
```

**Windows — Command Prompt:**
```cmd
.venv\Scripts\activate.bat
```

**macOS / Linux:**
```bash
source .venv/bin/activate
```

You should now see **`(.venv)`** at the start of the terminal line, like this:

```text
(.venv) PS D:\...\Day-02-Django>
```

> **Every time you open a new terminal, you must activate `.venv` again.** If you don't see `(.venv)`, run the activate command again.

### Step 7 — Install the packages

Make sure the line starts with `(.venv)` **and** ends with `Day-02-Django>`. Then:

```bash
python -m pip install --upgrade pip
```

```bash
pip install -r requirements.txt
```

This downloads Django and friends. It can take 1–2 minutes.

At the end you should see a line starting with `Successfully installed ...`.

### Step 8 — Check Django is installed

```bash
python -m django --version
```

You should see a version number such as `5.2.8`.

---

## Check

All three must be true:

- [ ] The terminal line starts with **`(.venv)`**.
- [ ] `python -m django --version` prints **5.2.something**.
- [ ] There is a **`.venv`** folder inside `Day-02-Django`.

---

## What just happened?

- `python -m venv .venv` created a private Python "bag" called `.venv`.
- Activating it told the terminal: "from now on, use the Python **inside** `.venv`".
- `pip install -r requirements.txt` read the shopping list and installed six packages **into** `.venv`:

| Package | What it does for us |
|---|---|
| `Django` | The web framework — builds our website |
| `psycopg` | Lets Python talk to PostgreSQL |
| `dj-database-url` | Reads the database address from one line of text |
| `python-dotenv` | Reads our private `.env` file (passwords, secret keys) |
| `whitenoise` | Sends CSS/images when the site is online (Lesson 17) |
| `gunicorn` | Runs Django on an internet server (Lesson 17) |

Open `requirements.txt` in VS Code and read it — it is just this list.

---

## Try it yourself

1. Close the terminal (click the bin (trash) icon on the terminal panel).
2. Open a **new** terminal (**Terminal → New Terminal**).
3. Is `(.venv)` there? (No — it's a new terminal!)
4. Activate it again (Step 6). Run `python -m django --version` again.

This "new terminal → activate" habit is the #1 thing beginners forget. Practise it now.

---

## If it does not work

| Problem | Fix |
|---|---|
| `python` is not recognized | Python is not installed or not added to PATH. Re-run the installer, tick **Add python.exe to PATH**, then **close and reopen VS Code**. Or try `py`. |
| PowerShell says **"running scripts is disabled on this system"** | Easiest fix: click the small **down-arrow** next to the **+** in the terminal panel → choose **Command Prompt** → use `.venv\Scripts\activate.bat`. (Or run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once, type `Y`, then try again.) |
| `ERROR: Could not open requirements file: requirements.txt` | You are in the wrong folder. The terminal must end with `Day-02-Django>`. Use `cd` to go there, or close the terminal and open a new one after opening the right folder (Step 2). |
| `No module named django` | `.venv` is not activated. Look for `(.venv)`. Activate, then try again. |
| Install fails with red text | Check your internet. Read the **first** red line — it usually names the problem. Make sure Python is 3.11–3.13. |

---

**Next:** [03_YOUR_FIRST_DJANGO_PROJECT.md](03_YOUR_FIRST_DJANGO_PROJECT.md)
