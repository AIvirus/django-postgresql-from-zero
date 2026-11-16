# Practice Workbook — Day 2

Use this **after** the lessons, for revision. Answer in your own words first, then check [ANSWERS.md](ANSWERS.md).
The lesson number in brackets tells you where to look if you are stuck.

## Section A — Web and Django (Lessons 01, 03, 04)
1. What happens when you type a website address into a browser?
2. Explain client and server using your own example.
3. Explain frontend and backend.
4. What is HTTP?
5. Explain GET, POST, PUT, PATCH and DELETE.
6. What do 200, 201, 302, 400, 401, 403, 404 and 500 mean?
7. What is Django?
8. What is the difference between a project and an app?
9. Explain Model, View and Template.
10. What is a Function-Based View?

## Section B — Databases (Lessons 01, 06, 07, 16)
11. What is a relational database?
12. Explain database, table, row and column.
13. What is a primary key?
14. What is a foreign key?
15. What is SQL?
16. What is PostgreSQL?
17. What is the difference between a database server and a database?
18. What is a database role?
19. What information is needed to connect to PostgreSQL?
20. What does `DATABASE_URL` contain?
21. Why should a database password not be written in Python source code?
22. What is a database transaction?
23. What is an index?
24. What is a constraint?

## Section C — Models and migrations (Lesson 07)
25. What is a model?
26. Explain CharField, TextField, BooleanField, DateTimeField and DecimalField.
27. What is the difference between `blank` and `null`?
28. What does `unique=True` do?
29. What is a migration?
30. What is the difference between `makemigrations` and `migrate`?
31. Why should migration files be committed to Git?
32. What can happen if you run migrations against the wrong database?

## Section D — Relationships (Lesson 09)
33. Explain One-to-One using User and Profile.
34. Explain One-to-Many using User and Article.
35. Explain Many-to-Many using Article and Category.
36. Why does Django create an intermediate table for Many-to-Many?
37. What does `on_delete` control?
38. Explain `CASCADE`, `PROTECT`, and `SET_NULL`.

## Section E — ORM, pages and forms (Lessons 10, 11, 13)
39. What does ORM stand for?
40. What is a QuerySet?
41. What is the difference between `get()` and `filter()`?
42. What does `select_related()` help with?
43. What does `prefetch_related()` help with?
44. Explain the N+1 query problem.
45. Expand CRUD and explain each operation.
46. Why should delete actions use POST?
47. What is a Django Form?
48. What is a ModelForm?
49. What is server-side validation?
50. What is CSRF and why is the token used?

## Section F — Accounts and security (Lessons 12, 14)
51. What is authentication?
52. What is authorization?
53. What is a session?
54. What is a cookie?
55. Why should passwords be hashed?
56. Why is hiding an Edit button not enough?
57. What is a Django Group?
58. Explain least privilege.

## Section G — Practical checks (do these on your own `my_journal` project)
59. Show that Django is connected to your Neon database. Which command proves it? (06)
60. Run migrations and name 5 tables that were created. (06, 07)
61. Create a superuser and open Admin. (08)
62. Register two normal users from the website. (12)
63. Create an article for each user. (13)
64. Create two categories and attach both to one article. (08, 13)
65. Edit your own article. (13)
66. Try to edit the other user's article. What happens, and which line of code causes it? (13)
67. Create a `ContentEditors` group and test `/users/editor-area/` with and without it. (14)
68. Add a `reading_time` field to Article and apply a migration. (15)
69. Run all automated tests. How many pass? (15)
70. Run a SQL `SELECT` with a `JOIN` in the Neon SQL Editor. (16)

## Section H — Professional thinking (Lessons 15, 17)
71. Why should local and deployment databases be separate?
72. Why should secrets not be pushed to GitHub?
73. Why is a free deployment not automatically production-ready?
74. Why should database constraints be used for important data rules?
75. What does a passing automated test actually prove?
