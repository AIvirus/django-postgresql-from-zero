# Glossary — Simple Meanings of Every New Word

Words are in A→Z order. The number in brackets is the lesson where you meet it first.

| Word | Simple meaning |
|---|---|
| **Admin (Django Admin)** | A ready-made control panel website for staff to manage data. (08) |
| **App** | One feature area inside a Django project, e.g. `journal`, `accounts`. (04) |
| **Authentication** | Checking **who** someone is (log in). (12) |
| **Authorization** | Checking what someone is **allowed** to do. (14) |
| **Backend** | The part of a website that runs on the server: rules, data, security. (01) |
| **Browser / Client** | The program that asks for web pages (Chrome, Edge…). (01) |
| **CASCADE** | "If the main row is deleted, delete the linked rows too." (09) |
| **Column** | One kind of information in a table, e.g. `title`. (01) |
| **Connection string / `DATABASE_URL`** | One line with everything needed to connect to a database: user, password, host, name. (06) |
| **Context** | The dictionary a view sends to a template, e.g. `{"student_name": "Asha"}`. (05) |
| **Cookie** | A small value the browser stores and sends back to the website each time. (12) |
| **CRUD** | Create, Read, Update, Delete — the four basic data actions. (10) |
| **CSRF / `{% csrf_token %}`** | An attack where another site secretly submits your forms; the token blocks it. (12) |
| **CSS** | The file that controls how a page looks: colours, spacing, fonts. (05) |
| **Database** | An organised, permanent store of data. (01) |
| **DEBUG** | A setting that shows detailed error pages. `True` on your laptop, **`False`** online. (06, 17) |
| **Deploy** | Put your website on a computer on the internet. (17) |
| **Environment variable** | A setting given to a program from outside its code (e.g. from `.env`). (06) |
| **`.env`** | A private file holding secrets like passwords. Never shared. (06) |
| **Field** | One attribute of a model, which becomes a column, e.g. `title = CharField(...)`. (07) |
| **Filter (template)** | A small formatter in templates, e.g. `{{ date|date:"d M Y" }}`. (11) |
| **Foreign key / `ForeignKey`** | A column that stores the id of a row in another table. One-to-Many. (09) |
| **Form / ModelForm** | Collects and checks user input. A ModelForm is built from a model. (13) |
| **Framework** | A ready-made toolkit and structure for building software. Django is one. (03) |
| **Frontend** | What the user sees in the browser: HTML, CSS, JavaScript. (01) |
| **GET** | A request to **see** something. Must not change data. (01) |
| **Group** | A named set of users, used as a role, e.g. `ContentEditors`. (14) |
| **gunicorn** | A real web server program that runs Django online. (17) |
| **Hash (password hash)** | A one-way scramble of a password. The real password is never stored. (12) |
| **HTML** | The structure of a web page: headings, paragraphs, links, forms. (05) |
| **HTTP** | The rules browsers and servers use to send requests and responses. (01) |
| **JavaScript** | Code that runs inside the browser, e.g. reacting to a button click. (05) |
| **JOIN** | SQL that combines rows from linked tables. (16) |
| **Lazy (QuerySet)** | Django waits until results are really needed before asking the database. (10) |
| **Many-to-Many** | Many rows on each side can be linked, e.g. articles ↔ categories. Uses a linking table. (09) |
| **Migration** | A file of instructions to create or change tables. (07) |
| **Model** | A Python class that describes a kind of data; becomes a table. (07) |
| **MVT** | Model–View–Template: Django's way of organising code. (04) |
| **N+1 problem** | Running 1 query for a list, then 1 more for **each** item. Fixed with `select_related` / `prefetch_related`. (11) |
| **Namespace (`app_name`)** | A family name for an app's URLs, e.g. `journal:home`. (04) |
| **Neon** | A website that runs PostgreSQL databases for you, with a free plan. (06) |
| **On delete** | What happens to linked rows when a row is deleted (CASCADE, PROTECT, SET_NULL). (09) |
| **One-to-Many** | One row links to many rows, e.g. one user → many articles. (09) |
| **One-to-One** | Exactly one row on each side, e.g. one user ↔ one profile. (09) |
| **ORM** | The translator between Python objects and SQL. (10) |
| **Ownership** | A record belongs to a user; only they may change it. (13) |
| **pip** | Python's package installer. (02) |
| **`pk` / primary key / `id`** | The unique number of each row. (07) |
| **Port** | A "door number" on a computer, e.g. `8000`. (03) |
| **POST** | A request that **sends** data to save or change something. (01) |
| **PostgreSQL** | A professional, free database system. (06) |
| **PROTECT** | "Don't allow deleting a row while other rows link to it." (09) |
| **Query string** | The part after `?` in a URL, e.g. `?q=django`. Read with `request.GET`. (11) |
| **QuerySet** | A list-like set of rows from the database. (10) |
| **Redirect (302)** | The server tells the browser "go to this other page". (12) |
| **`related_name`** | The name used to go backwards along a link, e.g. `user.articles`. (09) |
| **Request / Response** | What the browser asks for / what the server sends back. (01) |
| **Row** | One record in a table, e.g. one article. (01) |
| **`SECRET_KEY`** | A random secret Django uses to sign cookies. Never shared. (06) |
| **Server** | The computer/program that answers requests. (01) |
| **Session** | The server's memory that "this browser is logged in as user X". (12) |
| **Settings (`settings.py`)** | The configuration file of a Django project. (03) |
| **Shell** | An interactive Python prompt (`>>>`). `manage.py shell` connects it to your project. (10) |
| **SQL** | The language databases speak. (16) |
| **Static files** | CSS, JavaScript and images. (05) |
| **Status code** | A number saying how a request went: 200 OK, 302 redirect, 403 forbidden, 404 not found, 500 error. (01) |
| **Superuser** | An account that can do everything in Admin. (08) |
| **Table** | Rows and columns of one kind of data, like a spreadsheet sheet. (01) |
| **Template** | An HTML file with gaps (`{{ }}`, `{% %}`) that Django fills in. (05) |
| **Template inheritance** | A page `{% extends %}` a base layout and fills its blocks. (05) |
| **Test** | Code that checks your code automatically. (15) |
| **URL pattern** | Connects an address to a view: `path("articles/", views.article_list)`. (04) |
| **Validation** | Checking that data follows the rules, on the server. (13) |
| **View** | A Python function that receives a request and returns a response. (04) |
| **Virtual environment (`.venv`)** | A private "bag" of packages for one project. (02) |
| **WhiteNoise** | A package that serves CSS/JS/images for a live Django site. (17) |
