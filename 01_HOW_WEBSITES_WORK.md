# Lesson 01 — How Websites Work

**Time:** 20 minutes  |  **You need:** a web browser (Chrome or Edge is easiest)

---

## Goal

By the end of this lesson you can explain, in your own words:

- what happens when you open a website,
- what a **request** and a **response** are,
- what **GET**, **POST**, **200** and **404** mean,
- why a website needs a **database**,
- where **Django** fits in.

No coding yet. But you **will** look at real requests in your browser.

---

## Understand

### 1. The restaurant story

Think of a restaurant:

| Restaurant | Website |
|---|---|
| You (the customer) | **Browser** (Chrome, Edge, Firefox…) — also called the **client** |
| You give your order to the waiter | The browser sends a **request** |
| The kitchen | The **server** (a computer running our **Django** code) |
| The kitchen's store room | The **database** (where all the data is kept) |
| The waiter brings your food | The server sends back a **response** (usually a web page) |

So: **Browser → asks → Server (Django) → looks in → Database → Server builds the page → Browser shows it.**

### 2. Frontend and backend

- **Frontend** = what you **see** in the browser: buttons, text, colours. Made with **HTML** (structure), **CSS** (looks) and **JavaScript** (small actions in the page).
- **Backend** = what happens **on the server**, which you cannot see: checking passwords, saving articles, finding search results. **Django is a backend tool, written in Python.**

Important rule: **never trust the browser for important decisions.** Anyone can change what their own browser sends. The **backend** must check "Is this person logged in?" and "Is this person allowed to edit this article?"

### 3. A request has a *method*

The method says **what kind of thing** the browser wants:

| Method | Simple meaning | Example |
|---|---|---|
| **GET** | "Please **show** me something" | Open the article list |
| **POST** | "Please **take** this data" (save / change something) | Submit a new article |
| PUT | "**Replace** this whole thing" | Replace all fields of an article |
| PATCH | "**Change part** of this thing" | Change only the title |
| DELETE | "**Remove** this thing" | Delete an article |

Today's website uses only **GET** and **POST** (normal HTML forms can only send these two). You will use PUT, PATCH and DELETE on Day 3 when you build APIs.

A method name does not make anything safe by itself. The **server** must still check that the request is allowed.

A request has four parts, and so does a response:

| Request | Response |
|---|---|
| **Method** (GET, POST…) | **Status code** (200, 404…) |
| **Path** (`/articles/`) | **Headers** (extra information) |
| **Headers** (extra information, e.g. cookies) | **Body** (the HTML page, or other data) |
| **Body** (optional data, e.g. a filled form) | |

### 4. A response has a *status code*

The status code is a number that says **how it went**:

| Code | Meaning | Like in the restaurant… |
|---|---|---|
| **200** | OK — here it is | Here is your food |
| 201 | Created — a new thing was made | "Your table is booked" |
| 204 | Done, nothing to send back | "Noted." |
| **302** | Go somewhere else (redirect) | "Please sit at table 5 instead" |
| 400 | Bad request — what you sent doesn't make sense | "That isn't on the menu form" |
| 401 | Not logged in — tell us who you are first | "Show your membership card" |
| **403** | Forbidden — we know who you are, but you are not allowed | "Staff only!" |
| **404** | Not found | "We don't have that dish" |
| **500** | Server error — something broke on the server | The kitchen caught fire |

Easy way to remember: **2xx** = success, **3xx** = go elsewhere, **4xx** = the visitor's request has a problem, **5xx** = the server has a problem. The **bold** ones are the codes you will see today.

### 5. Why do we need a database?

If you keep data in a normal Python list, it **disappears** when the program stops. A **database** keeps the data safely, even after the computer restarts.

A database stores data in **tables**, like a spreadsheet:

| id | title | author_id | is_published |
|---|---|---|---|
| 1 | Django for beginners | 4 | true |
| 2 | Understanding databases | 4 | false |

- One **row** = one article.
- One **column** = one kind of information (title, author…).
- **id** = a unique number for each row (like a roll number in class).

Today we use a database called **PostgreSQL**, which professional companies use.

### 6. The full journey (remember this!)

When someone opens `/articles/` on our website:

```text
Browser ──GET /articles/──▶ Django URL list ──▶ View (Python function)
                                                   │
                                                   ▼
                                        Database (PostgreSQL)
                                                   │
Browser ◀──HTML page── Template (HTML) ◀── View gets the articles
```

Short version to remember:

> **Browser → URL → View → Database → View → Template → Browser**

You will build every box of this picture yourself in the next lessons.

---

## Do it now — see real requests with your own eyes

### Step 1 — Open the browser's Developer Tools

1. Open **Chrome** or **Edge**.
2. Go to `https://www.wikipedia.org`
3. Press **F12** (on a Mac: **Cmd + Option + I**). A panel opens.
4. Click the **Network** tab in that panel.
5. Press **F5** to reload the page.

### Step 2 — Look at the first request

1. In the list, click the **very first row** (it is the page itself).
2. Look at the **Headers** section. Find:
   - **Request Method** → you should see `GET`
   - **Status Code** → you should see `200`

### Step 3 — Make a 404 on purpose

1. In the address bar, type `https://www.wikipedia.org/this-page-does-not-exist-123` and press Enter.
2. Click the first row in the Network tab again.
3. Look at **Status Code**.

---

## Check

- In Step 2 you saw **Request Method: GET** and **Status Code: 200**.
- In Step 3 you saw **Status Code: 404**.

If yes — you have just watched HTTP requests and responses happen for real.

---

## What just happened?

- Your browser (the **client**) sent a **GET request** to Wikipedia's **server**.
- The server answered with a **response**: status **200** and an HTML page.
- When you asked for a page that does not exist, the server answered **404**.

Every Django page you build today works in exactly this way. Later, when your own page shows a 404 or a 302, you will know what it means.

---

## Try it yourself

Answer in your notebook (short answers are fine):

1. Which one is the client: the browser or the server?
2. You fill in a "Create article" form and click **Save**. Is that a GET or a POST?
3. What does status code **404** mean?
4. Why should the **server** (not the browser) check if a user is allowed to delete an article?
5. Why can't we just keep all articles in a Python list?

<details>
<summary>Click to see the answers</summary>

1. The browser is the client.
2. POST — it sends data that changes something on the server.
3. Not found.
4. Because anyone can change what their own browser sends, so only the server can be trusted.
5. A Python list disappears when the program stops. A database keeps data safely.
</details>

---

## If it does not work

- **F12 does nothing:** right-click anywhere on the page → **Inspect**, then click **Network**.
- **The Network list is empty:** keep the panel open and press **F5** again.
- **Cannot find "Status Code":** click the first row, then look under **Headers → General**.

---

**Next:** [02_SET_UP_YOUR_COMPUTER.md](02_SET_UP_YOUR_COMPUTER.md)
