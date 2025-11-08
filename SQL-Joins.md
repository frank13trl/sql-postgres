# SQL Joins

## SQL Joins
SQL Joins are used to combine rows from two or more tables, based on a related column between them. They are a fundamental part of relational database management, allowing you to retrieve data that is spread across multiple tables. There are several types of joins, each serving a specific purpose in combining data from related tables.

### Joins and Foreign Keys

A foreign key is a key used to link two tables together. It is a field (or collection of fields) in one table that refers to the PRIMARY KEY in another table. The table containing the foreign key is called the child table, and the table containing the candidate key is called the referenced or parent table.

Joins use these foreign key relationships to combine data. The join condition typically matches the foreign key in one table with the primary key in another table. This ensures that only related data is combined.

## INNER JOIN

An `INNER JOIN` returns only the rows that have matching values in both tables. It's the most common type of join and is often the default join type if you don't specify otherwise.

**Example (PostgreSQL):**

Let's say we have two tables: `customers` and `orders`.

`customers` table:
| customer_id | customer_name |
|-------------|---------------|
| 1           | Alice         |
| 2           | Bob           |
| 3           | Charlie       |

`orders` table:
| order_id | customer_id | order_date |
|----------|-------------|------------|
| 101      | 1           | 2023-01-01 |
| 102      | 3           | 2023-01-02 |
| 103      | 1           | 2023-01-03 |
| 104      | 4           | 2023-01-04 |

To get a list of all orders along with the customer's name for those orders, we can use an `INNER JOIN`:

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date
FROM
    customers c
INNER JOIN
    orders o ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | order_date |
|---------------|----------|------------|
| Alice         | 101      | 2023-01-01 |
| Charlie       | 102      | 2023-01-02 |
| Alice         | 103      | 2023-01-03 |

## LEFT JOIN (or LEFT OUTER JOIN)

A `LEFT JOIN` returns all rows from the left table (the first table in the `FROM` clause) and the matching rows from the right table. If there is no match from the right table, `NULL` values are returned for the right table's columns.

**Example (PostgreSQL):**

Using the same `customers` and `orders` tables:

To get a list of all customers and any orders they might have, including customers who have not placed any orders:

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date
FROM
    customers c
LEFT JOIN
    orders o ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | order_date |
|---------------|----------|------------|
| Alice         | 101      | 2023-01-01 |
| Alice         | 103      | 2023-01-03 |
| Bob           | NULL     | NULL       |
| Charlie       | 102      | 2023-01-02 |

## RIGHT JOIN (or RIGHT OUTER JOIN)

A `RIGHT JOIN` returns all rows from the right table (the second table in the `FROM` clause) and the matching rows from the left table. If there is no match from the left table, `NULL` values are returned for the left table's columns.

**Example (PostgreSQL):**

Using the same `customers` and `orders` tables:

To get a list of all orders and the customer who placed them, including orders that might not have a matching customer (e.g., due to data inconsistency):

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date
FROM
    customers c
RIGHT JOIN
    orders o ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | order_date |
|---------------|----------|------------|
| Alice         | 101      | 2023-01-01 |
| Charlie       | 102      | 2023-01-02 |
| Alice         | 103      | 2023-01-03 |
| NULL          | 104      | 2023-01-04 |

## FULL OUTER JOIN

A `FULL OUTER JOIN` returns all rows when there is a match in either the left or right table. It is a combination of `LEFT JOIN` and `RIGHT JOIN`.

**Example (PostgreSQL):**

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date
FROM
    customers c
FULL OUTER JOIN
    orders o ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | order_date |
|---------------|----------|------------|
| Alice         | 101      | 2023-01-01 |
| Alice         | 103      | 2023-01-03 |
| Bob           | NULL     | NULL       |
| Charlie       | 102      | 2023-01-02 |
| NULL          | 104      | 2023-01-04 |

## CROSS JOIN

A `CROSS JOIN` returns the Cartesian product of the two tables. This means it returns every possible combination of rows from both tables.

**Example (PostgreSQL):**

```sql
SELECT
    c.customer_name,
    o.order_id
FROM
    customers c
CROSS JOIN
    orders o;
```

**Result:**

| customer_name | order_id |
|---------------|----------|
| Alice         | 101      |
| Alice         | 102      |
| Alice         | 103      |
| Alice         | 104      |
| Bob           | 101      |
| Bob           | 102      |
| Bob           | 103      |
| Bob           | 104      |
| Charlie       | 101      |
| Charlie       | 102      |
| Charlie       | 103      |
| Charlie       | 104      |

## SELF JOIN

A `SELF JOIN` is a regular join, but the table is joined with itself. This is useful for querying hierarchical data or comparing rows within the same table.

**Example (PostgreSQL):**

Let's say we have an `employees` table where each employee has a `manager_id` that refers to another employee's `employee_id`.

`employees` table:
| employee_id | employee_name | manager_id |
|-------------|---------------|------------|
| 1           | Alice         | NULL       |
| 2           | Bob           | 1          |
| 3           | Charlie       | 1          |

To get a list of employees and their managers:

```sql
SELECT
    e.employee_name AS employee_name,
    m.employee_name AS manager_name
FROM
    employees e
LEFT JOIN
    employees m ON e.manager_id = m.employee_id;
```

**Result:**

| employee_name | manager_name |
|---------------|--------------|
| Alice         | NULL         |
| Bob           | Alice        |
| Charlie       | Alice        |

**Note on Foreign Keys and SELF JOINs:**
While a `SELF JOIN` operates on a single table, the concept of a foreign key is still crucial. Often, a `SELF JOIN` is used when a table contains a **self-referencing foreign key**. This is a column (like `manager_id` in our example) that refers to the primary key (`employee_id`) within the *same* table. This mechanism establishes relationships (e.g., hierarchical structures) within the data of a single table, which the `SELF JOIN` then leverages.

## Set Operators

Set operators are used to combine the result sets of two or more `SELECT` statements. For all set operators, the `SELECT` statements must have the same number of columns, and the columns must have compatible data types.

## UNION

The `UNION` operator combines the result sets and **removes duplicate rows**.

**Example (PostgreSQL):**

Let's say we have two tables, `employees` and `contractors`, and we want a combined list of all unique names.

`employees` table:
| id | name    |
|----|---------|
| 1  | Alice   |
| 2  | Bob     |

`contractors` table:
| id | name      |
|----|-----------|
| 10 | Charlie   |
| 11 | Alice     |

```sql
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

**Result:**

| name    |
|---------|
| Alice   |
| Bob     |
| Charlie |

## UNION ALL

The `UNION ALL` operator combines the result sets but **includes all duplicate rows**. This makes `UNION ALL` generally faster than `UNION` because it doesn't need to check for duplicates.

**Example (PostgreSQL):**

Using the same `employees` and `contractors` tables:

```sql
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

**Result:**

| name    |
|---------|
| Alice   |
| Bob     |
| Charlie |
| Alice   |

## INTERSECT

The `INTERSECT` operator returns only the rows that are present in **both** result sets.

**Example (PostgreSQL):**

Using the same `employees` and `contractors` tables, let's find the names that appear in both tables.

```sql
SELECT name FROM employees
INTERSECT
SELECT name FROM contractors;
```

**Result:**

| name    |
|---------|
| Alice   |

## EXCEPT

The `EXCEPT` operator returns all rows that are present in the **first** result set but **not** in the second result set.

*(Note: Some databases, like Oracle, use the keyword `MINUS` instead of `EXCEPT`.)*

**Example (PostgreSQL):**

Using the same `employees` and `contractors` tables, let's find the names that are in the `employees` table but not in the `contractors` table.

```sql
SELECT name FROM employees
EXCEPT
SELECT name FROM contractors;
```

**Result:**

| name    |
|---------|
| Bob     |

### INTERSECT vs. INNER JOIN: Key Differences

While both `INTERSECT` and `INNER JOIN` can be used to find matching data, they operate in fundamentally different ways and serve different purposes.

| Feature | `INTERSECT` (Set Operator) | `INNER JOIN` (Join Operator) |
| :--- | :--- | :--- |
| **Purpose** | Compares the **entire rows** of two result sets and returns only the rows that are identical in both. | Combines rows from two tables into a **single new row** based on a matching condition in one or more columns. |
| **Result Columns** | The number and names of columns are determined by the **first `SELECT` statement**. | The result can include columns from **both tables**. The structure is a new, wider row. |
| **Duplicate Handling** | **Removes duplicate rows** from the final result set. It returns only unique rows that are common to both queries. | **Does not remove duplicates**. If there are multiple matches on the join condition, it returns the Cartesian product of those matching rows. |
| **Column Requirements** | The `SELECT` statements must have the **same number of columns** with compatible data types. | The columns in the join condition must have compatible data types, but the tables can have a different number and type of other columns. |
| **`NULL` Handling** | Treats `NULL` values as **equal**. A row with a `NULL` can be matched with another row with a `NULL` in the same position. | Does **not** treat `NULL` values as equal. A `NULL` in a join key will never match another `NULL`. |

---

### Practical Example

Let's use two simple tables to illustrate the difference.

`Table_A`
| id | name  |
|----|-------|
| 1  | Alice |
| 2  | Bob   |
| 3  | Alice |

`Table_B`
| id | name  |
|----|-------|
| 1  | Alice |
| 4  | Carol |
| 5  | Alice |

#### Using `INTERSECT`

`INTERSECT` looks for **identical full rows**.

```sql
SELECT id, name FROM Table_A
INTERSECT
SELECT id, name FROM Table_B;
```

**Result:**

| id | name  |
|----|-------|
| 1  | Alice |

**Why?** The only row that is exactly the same (matching on both `id` and `name`) in both tables is `(1, 'Alice')`.

#### Using `INNER JOIN`

`INNER JOIN` combines rows based on a **join condition**. Let's join on the `name` column.

```sql
SELECT
    A.id AS a_id,
    A.name,
    B.id AS b_id
FROM
    Table_A A
INNER JOIN
    Table_B B ON A.name = B.name;
```

**Result:**

| a_id | name  | b_id |
|------|-------|------|
| 1    | Alice | 1    |
| 1    | Alice | 5    |
| 3    | Alice | 1    |
| 3    | Alice | 5    |

**Why?** The join condition is `A.name = B.name`.
*   `Table_A` has two rows where `name = 'Alice'` (ids 1 and 3).
*   `Table_B` has two rows where `name = 'Alice'` (ids 1 and 5).
*   The `INNER JOIN` creates a Cartesian product of these matches (2 rows from A * 2 rows from B = 4 result rows).

### Summary: When to Use Which?

*   Use **`INTERSECT`** when you want to find the **common, identical records** between two similar result sets. Think of it as finding the "overlap" of two lists.
*   Use **`INNER JOIN`** when you want to **enrich data** by combining columns from two different tables based on a related key. This is the standard and far more common way to bring related data together.

<br></br>

# SQL Views

A `VIEW` in SQL is a virtual table based on the result-set of an SQL statement. A view contains rows and columns, just like a real table. The fields in a view are fields from one or more real tables in the database.

Views do not store data themselves; instead, they are dynamic and reflect the data in the underlying tables at the time the view is queried. They are useful for:

*   **Simplifying complex queries:** You can encapsulate complex `JOIN` operations, aggregations, or subqueries into a single view, making subsequent queries simpler.
*   **Enhancing security:** You can restrict access to certain rows or columns by creating a view that only exposes a subset of the data from a table, without granting direct access to the base table.
*   **Data abstraction:** Views can present data in a different structure than the underlying tables, providing a consistent interface even if the base table schema changes (as long as the view definition is updated).

**Syntax (PostgreSQL):**

To create a view, you use the `CREATE/REPLACE VIEW` statement:

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

**Example (PostgreSQL):**

Let's create a view that combines customer names with their order details, similar to an `INNER JOIN` example, but now stored as a reusable view.

First, ensure you have the `customers` and `orders` tables as defined in the `INNER JOIN` section.

```sql
CREATE VIEW customer_order_details AS
SELECT
    c.customer_name,
    o.order_id,
    o.order_date
FROM
    customers c
INNER JOIN
    orders o ON c.customer_id = o.customer_id;
```

Once the view `customer_order_details` is created, you can query it just like a regular table:

```sql
SELECT *
FROM customer_order_details
WHERE customer_name = 'Alice';
```

**Result (from querying the view):**

| customer_name | order_id | order_date |
|---------------|----------|------------|
| Alice         | 101      | 2023-01-01 |
| Alice         | 103      | 2023-01-03 |

You can also update or delete data through views, but there are restrictions. Generally, views are used for querying and simplifying data retrieval.

## Dropping Views

To delete a view, you use the `DROP VIEW` statement. The syntax has important differences between PostgreSQL and MySQL regarding dependencies.

### PostgreSQL

```sql
DROP VIEW [ IF EXISTS ] view_name [, ...] [ CASCADE | RESTRICT ];
```

*   **`IF EXISTS`**: Prevents an error if the view doesn't exist.
*   **`CASCADE`**: Automatically drops objects that depend on the view (like other views).
*   **`RESTRICT`**: (Default) Prevents dropping the view if any other objects depend on it.

**Example:**
```sql
-- Drops the view if nothing depends on it
DROP VIEW customer_order_details;

-- Drops the view and any objects that depend on it
DROP VIEW IF EXISTS customer_order_details CASCADE;
```

### MySQL

```sql
DROP VIEW [ IF EXISTS ] view_name [, view_name] ...;
```

MySQL does not support the `CASCADE` or `RESTRICT` options. You must manually drop any dependent objects before dropping the view.

<br></br>

# SQL Indexes

An SQL index is a special lookup table that the database search engine can use to speed up data retrieval. Think of it like the index in the back of a book; it allows the database to find rows with specific column values much more quickly than scanning the entire table.

While indexes speed up `SELECT` queries and `WHERE` clauses, they can slow down data modification (`INSERT`, `UPDATE`, `DELETE`) because the index also needs to be updated.

> **Note on Automatic Indexes:**
> Databases like PostgreSQL and MySQL **automatically create an index** when you define a `PRIMARY KEY` or a `UNIQUE` constraint. This is necessary to enforce the uniqueness of the data.
> 
> For all other columns (like foreign keys or columns frequently used in `WHERE`, `JOIN`, `ORDER BY` clauses), you must **manually create indexes** using the `CREATE INDEX` command to improve query performance.


## Creating an Index

The basic syntax to create an index is similar in PostgreSQL and MySQL. You specify a name for the index, the table it applies to, and the column(s) to be indexed.

**Syntax:**
```sql
CREATE INDEX index_name
ON table_name (column_name1, column_name2, ...);
```

**Example (PostgreSQL & MySQL):**

Let's say we frequently search for customers by their name. We can create an index on the `customer_name` column in the `customers` table to speed this up.

```sql
CREATE INDEX idx_customers_name
ON customers (customer_name);
```

Now, queries that filter by `customer_name` will be significantly faster.

```sql
-- This query will now use the index
SELECT *
FROM customers
WHERE customer_name = 'Alice';
```

## Showing Indexes

It's often useful to see what indexes already exist on your tables. The method for doing this differs between PostgreSQL and MySQL.

**PostgreSQL:**

PostgreSQL does not have a direct `SHOW INDEXES` command. You typically query the `pg_indexes` system catalog view or use `psql` meta-commands.

*   **Querying `pg_indexes`:**
    ```sql
    SELECT indexname, tablename, indexdef
    FROM pg_indexes
    WHERE schemaname = 'public' -- or your specific schema
    ORDER BY tablename, indexname;
    ```
    To filter for a specific table:
    ```sql
    SELECT indexname, indexdef
    FROM pg_indexes
    WHERE tablename = 'customers';
    ```

*   **Using `psql` meta-commands (from the `psql` command-line client):**
    *   To list all indexes in the current database: `\di`
    *   To describe a table and its indexes: `\d table_name` (e.g., `\d customers`)

**MySQL:**

MySQL provides a `SHOW INDEX` statement.

*   **For a specific table:**
    ```sql
    SHOW INDEX FROM table_name;
    ```
    (e.g., `SHOW INDEX FROM customers;`)

*   **Querying `INFORMATION_SCHEMA.STATISTICS`:**
    ```sql
    SELECT DISTINCT TABLE_NAME, INDEX_NAME
    FROM INFORMATION_SCHEMA.STATISTICS
    WHERE TABLE_SCHEMA = 'your_database_name';
    ```

## Dropping an Index

To remove an index, you use the `DROP INDEX` statement.

**Syntax (PostgreSQL):**
```sql
DROP INDEX [ IF EXISTS ] index_name;
```

**Syntax (MySQL):**
```sql
DROP INDEX index_name ON table_name;
```

**Example:**
```sql
-- PostgreSQL
DROP INDEX IF EXISTS idx_customers_name;

-- MySQL
DROP INDEX idx_customers_name ON customers;
```
 <br></br>

# Subqueries (Inner Queries or Nested Queries)

A subquery is a `SELECT` statement that is nested inside another SQL statement (like `SELECT`, `INSERT`, `UPDATE`, or `DELETE`). The result of the inner query is used by the outer query. Subqueries are always enclosed in parentheses `()`.

## Subqueries in the `WHERE` Clause

This is the most common use case, allowing you to filter results based on the output of another query.

**Example:** Find all customers who have placed an order.

```sql
SELECT customer_name
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
```
In this example, the inner query `(SELECT DISTINCT customer_id FROM orders)` runs first, creating a list of all unique customer IDs from the `orders` table. The outer query then retrieves the names of customers whose `customer_id` is in that list.

### Subqueries in the `FROM` Clause

When a subquery is used in the `FROM` clause, its result set is treated as a temporary table, often called a "derived table" or "inline view". This derived table must be given an alias.

**Example:** Find the total number of orders per customer.

```sql
SELECT
    c.customer_name,
    order_counts.total_orders
FROM
    customers c
JOIN
    (SELECT customer_id, COUNT(order_id) AS total_orders
     FROM orders
     GROUP BY customer_id) AS order_counts
ON
    c.customer_id = order_counts.customer_id;
```
Here, the subquery calculates the total orders for each `customer_id` and creates a temporary table named `order_counts`. The outer query then joins the `customers` table with this derived table to get the customer names.

## Subqueries in the `SELECT` Clause

A subquery in the `SELECT` clause is used to retrieve a single value (one row, one column) and is known as a scalar subquery. It is executed for each row returned by the outer query.

**Example:** Show each customer's name and the date of their most recent order.

```sql
SELECT
    c.customer_name,
    (SELECT MAX(o.order_date)
     FROM orders o
     WHERE o.customer_id = c.customer_id) AS latest_order_date
FROM
    customers c;
```
This is a **correlated subquery** because the inner query depends on the `customer_id` from the outer query (`c.customer_id`). For each customer processed by the outer query, the inner query runs to find the latest order date for that specific customer.