# Answers — Day 2 Practice Workbook

Try the questions in [WORKBOOK.md](WORKBOOK.md) first. Your wording can be different — what matters is the meaning.

1. The browser sends a request to a server and displays the response.
2. Client asks; server receives and answers.
3. Frontend is the user-facing part; backend handles server-side rules and data.
4. HTTP is a protocol for exchanging requests and responses.
5. GET reads; POST submits; PUT replaces; PATCH changes part; DELETE requests removal.
6. 200 success; 201 created; 302 redirect; 400 bad request; 401 authentication missing; 403 access denied; 404 not found; 500 server error.
7. Django is a Python web framework.
8. Project is the whole site configuration; app is one feature area.
9. Model describes data; View handles request logic; Template creates the page.
10. A Python function that receives a request and returns a response.
11. A database that stores information in connected tables.
12. Database contains tables; table contains rows; row is a record; column is a field.
13. A unique identifier for a row.
14. A reference to a record in another table.
15. Structured Query Language, used to work with relational databases.
16. An open-source relational database system.
17. Server is the database service; database is the named collection of tables.
18. A database account with permissions.
19. Host, port, database name, username, password and connection/security options.
20. A connection string containing database connection information.
21. Source code may be shared; secrets must remain private.
22. A group of database operations treated as one unit.
23. A data structure that can help find records faster.
24. A database-enforced rule.
25. A Python class describing stored data.
26. Short text, long text, True/False, date/time, and exact decimal values.
27. `blank` affects validation; `null` affects whether the database stores SQL NULL.
28. Prevents duplicate values.
29. A saved instruction for changing database structure.
30. `makemigrations` creates migration files; `migrate` applies them to the database.
31. Other environments need the same database structure changes.
32. You may change the wrong database's schema.
33. One user has one profile; OneToOneField.
34. One user can have many articles; each article has one author; ForeignKey.
35. An article can have many categories and a category can have many articles; ManyToManyField.
36. It stores pairs connecting records from both tables.
37. What happens to dependent records if a referenced record is deleted.
38. CASCADE deletes dependent rows; PROTECT blocks deletion; SET_NULL clears the reference if null is allowed.
39. Object-Relational Mapper.
40. An object representing a collection of database results.
41. `get()` expects one record; `filter()` may return zero or many.
42. It can load ForeignKey/OneToOne related data with a SQL join.
43. It can load Many-to-Many or reverse-related data efficiently.
44. One query loads a list and extra queries load related data for each item.
45. Create, Read, Update, Delete.
46. GET should not change data and may be repeated by browsers or crawlers.
47. A Django class defining input fields and validation.
48. A Form connected to a model.
49. Checking input on the server before accepting it.
50. Cross-Site Request Forgery; token helps Django verify internal unsafe form submissions.
51. Checking who the user is.
52. Checking what the user may do.
53. Server-side state that helps remember a logged-in browser.
54. Small data stored by the browser and sent back to the site.
55. So the original password is not stored as readable text.
56. A user can send a direct request; the server must check permission.
57. A named collection of users and permissions.
58. Give only the permissions needed for the job.
59. `python manage.py migrate` (or `check`) runs without errors and the tables appear in Neon → Tables.
60. e.g. `auth_user`, `django_migrations`, `django_session`, `journal_article`, `journal_category`, `journal_article_categories`, `accounts_profile`.
61–65. Follow Lessons 08, 12 and 13. Check each result on the website and in Admin.
66. You get a **404**. Caused by `get_object_or_404(Article, pk=pk, author=request.user)` in `article_update` — it only finds articles you own.
67. Without the group you are redirected to login; with it you see "Content editor area". (Lesson 14)
68. Add `reading_time = models.PositiveIntegerField(default=1)`, then `makemigrations` and `migrate`. (Lesson 15)
69. 12 tests (13 if you added the reading-time test). All should say OK.
70. e.g. `SELECT a.title, u.username FROM journal_article a JOIN auth_user u ON a.author_id = u.id;`
71. To avoid accidental cross-environment changes and use separate credentials.
72. Public repositories can expose credentials.
73. Free plans have limits and may not provide uptime, backup or support guarantees.
74. They protect data rules even when multiple parts of an app write to the database.
75. It proves the tested cases behaved as expected, not that the whole application is bug-free.
