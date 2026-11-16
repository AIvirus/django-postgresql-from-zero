# Jupyter Notebooks — Optional Extra Practice

These notebooks are **extra practice**. They are **not** required for the main lessons.

| Notebook | Best time to use it | What you practise |
|---|---|---|
| `01_Database_and_SQL_Basics.ipynb` | After Lesson 07 (or with Lesson 16) | Tables, keys, `SELECT`, `WHERE`, `JOIN`, `UPDATE`, `DELETE` |
| `02_Django_ORM_Practice.ipynb` | After Lesson 10 | `create`, `filter`, `get`, `order_by`, `save`, `delete` |

## How to open them

1. In VS Code, install the **Jupyter** extension (by Microsoft) from the Extensions panel.
2. In a terminal with `.venv` active, from the `Day-02-Django` folder, install the notebook kernel once:
   ```bash
   pip install ipykernel
   ```
3. Open a `.ipynb` file in VS Code.
4. Top-right, click **Select Kernel** → **Python Environments** → choose the one that shows **`.venv`**.
5. Click **Run All**, or run the cells one by one from the top with **Shift + Enter**.

## Which database do they use?

- They use **SQLite**, a tiny database stored in a file inside this folder (e.g. `day2_sql_practice.db`).
- They do **not** connect to Neon and **cannot** change your `my_journal` data. Experiment freely!
- Your real website (`my_journal`) uses **PostgreSQL** on Neon — Lesson 06.

Never put real passwords or connection strings inside a notebook.
