# SQL Functions

This file will contain various SQL function definitions and examples.

## Aggregate Functions

Aggregate functions perform a calculation on a set of values and return a single value.

### COUNT()

The `COUNT()` function returns the number of rows that matches a specified criterion.

**Example:**

```sql
-- Count the number of customers
SELECT COUNT(customer_id)
FROM customers;
```

**Example with COUNT(*):**

```sql
-- Count all rows in the customers table
SELECT COUNT(*)
FROM customers;
```

**Example with WHERE:**

```sql
-- Count customers from a specific city
SELECT COUNT(customer_id)
FROM customers
WHERE city = 'New York';
```

### SUM()

The `SUM()` function returns the total sum of a numeric column.

**Example:**

```sql
-- Get the total amount of all orders
SELECT SUM(amount)
FROM orders;
```

**Example with WHERE:**

```sql
-- Get the total amount of orders for a specific customer
SELECT SUM(amount)
FROM orders
WHERE customer_id = 101;
```

### AVG()

The `AVG()` function returns the average value of a numeric column.

**Example:**

```sql
-- Calculate the average order amount
SELECT AVG(amount)
FROM orders;
```

**Example with WHERE:**

```sql
-- Calculate the average order amount for a specific product
SELECT AVG(amount)
FROM order_items
WHERE product_id = 50;
```

### MIN()

The `MIN()` function returns the minimum value of the selected column.

**Example:**

```sql
-- Find the minimum order amount
SELECT MIN(amount)
FROM orders;
```

**Example with WHERE:**

```sql
-- Find the minimum order amount for a specific customer
SELECT MIN(amount)
FROM orders
WHERE customer_id = 101;
```

### MAX()

The `MAX()` function returns the maximum value of the selected column.

**Example:**

```sql
-- Find the maximum order amount
SELECT MAX(amount)
FROM orders;
```

**Example with WHERE:**

```sql
-- Find the maximum order amount for a specific customer
SELECT MAX(amount)
FROM orders
WHERE customer_id = 101;
```

## String Functions

String functions are used to manipulate string data.

### LENGTH()

The `LENGTH()` function returns the length of a string (number of characters).

**Example:**

```sql
SELECT LENGTH('Hello World'); -- Returns 11
```

```sql
-- Get the length of customer names
SELECT customer_name, LENGTH(customer_name) AS name_length
FROM customers;
```

### LOWER()

The `LOWER()` function converts a string to lowercase.

**Example:**

```sql
SELECT LOWER('HELLO World'); -- Returns 'hello world'
```

```sql
-- Convert customer names to lowercase
SELECT LOWER(customer_name) AS lower_name
FROM customers;
```

### UPPER()

The `UPPER()` function converts a string to uppercase.

**Example:**

```sql
SELECT UPPER('hello World'); -- Returns 'HELLO WORLD'
```

```sql
-- Convert customer names to uppercase
SELECT UPPER(customer_name) AS upper_name
FROM customers;
```

### CONCAT()

The `CONCAT()` function concatenates two or more strings.

**Example:**

```sql
SELECT CONCAT('Hello', ' ', 'World'); -- Returns 'Hello World'
```

```sql
-- Concatenate first and last names
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

## Date/Time Functions

Date and time functions are used to manipulate date and time values.

### NOW() / CURRENT_TIMESTAMP

Returns the current date and time.

**Example:**

```sql
SELECT NOW();
SELECT CURRENT_TIMESTAMP;
```

### CURRENT_DATE / CURDATE()

Returns the current date. `CURDATE()` is common in MySQL, while `CURRENT_DATE` is standard SQL.

**Example:**

```sql
SELECT CURRENT_DATE;
-- SELECT CURDATE(); -- For MySQL
```

### EXTRACT() / YEAR() / MONTH()

`EXTRACT()` extracts a part of a date/time value (e.g., year, month, day, hour). `YEAR()` and `MONTH()` are common in some SQL dialects (like MySQL) as shorthand for extracting the year or month.

**Example with EXTRACT:**

```sql
SELECT EXTRACT(YEAR FROM CURRENT_DATE); -- Returns the current year
SELECT EXTRACT(MONTH FROM order_date) AS order_month
FROM orders;
```

**Example with YEAR() / MONTH() (MySQL-like):**

```sql
-- SELECT YEAR(order_date) AS order_year FROM orders; -- For MySQL
-- SELECT MONTH(order_date) AS order_month FROM orders; -- For MySQL
```

### AGE()

Calculates the age (difference) between two timestamps or a timestamp and the current time.

**Example:**

```sql
SELECT AGE('2025-11-01', '2020-01-01'); -- Returns interval '5 years 10 months 0 days'
SELECT AGE(birth_date) AS age_of_person
FROM persons;
```

### DATEDIFF()

Calculates the difference between two dates. The exact syntax and units (e.g., days, months, years) can vary between SQL dialects.

**Example (MySQL-like syntax for days difference):**

```sql
-- SELECT DATEDIFF('2025-11-01', '2025-10-20'); -- Returns 12 (days)
-- SELECT DATEDIFF(end_date, start_date) AS days_difference FROM projects;
```

**Example (PostgreSQL equivalent for days difference):**

```sql
SELECT '2025-11-01'::date - '2025-10-20'::date; -- Returns 12 (days)
SELECT end_date - start_date AS days_difference FROM projects;
```

## Mathematical Functions

Mathematical functions perform calculations on numeric data.

### ROUND()

The `ROUND()` function rounds a number to a specified number of decimal places.

**Example:**

```sql
SELECT ROUND(123.456, 2); -- Returns 123.46
SELECT ROUND(price) AS rounded_price FROM products;
```

### CEIL() / CEILING()

The `CEIL()` or `CEILING()` function rounds a number up to the nearest integer.

**Example:**

```sql
SELECT CEIL(123.45); -- Returns 124
SELECT CEILING(price) AS ceiling_price FROM products;
```

### FLOOR()

The `FLOOR()` function rounds a number down to the nearest integer.

**Example:**

```sql
SELECT FLOOR(123.45); -- Returns 123
SELECT FLOOR(price) AS floor_price FROM products;
```

### ABS()

The `ABS()` function returns the absolute (non-negative) value of a number.

**Example:**

```sql
SELECT ABS(-123.45); -- Returns 123.45
SELECT ABS(price_difference) FROM product_price_history;
```

## Conditional Functions

Conditional functions allow you to introduce if-else logic into your SQL queries.

### CASE

The `CASE` statement goes through conditions and returns a value when the first condition is met. So, once a condition is true, it will stop reading and return the result. If no conditions are true, it returns the value in the `ELSE` clause. If there is no `ELSE` part and no conditions are true, it returns NULL.

**Example:**

```sql
SELECT
    product_name,
    price,
    CASE
        WHEN price > 100 THEN 'Expensive'
        WHEN price > 50 THEN 'Moderate'
        ELSE 'Cheap'
    END AS price_category
FROM
    products;
```

**Output:**

```
 product_name | price | price_category
--------------+-------+----------------
 Product A    |   120 | Expensive
 Product B    |    80 | Moderate
 Product C    |    40 | Cheap
 Product D    |   120 | Expensive
```

### COALESCE()

The `COALESCE()` function returns the first non-null expression among its arguments. It's often used to provide a default value for columns that might contain `NULL`s.

**Example:**

Assume a `products` table with `product_name`, `description`, and `short_description` columns, where `description` might be `NULL`.

```sql
SELECT
    product_name,
    COALESCE(description, short_description, 'No description available') AS display_description
FROM
    products;
```

**Output (assuming sample data):**

```
| product_name | display_description        |
|--------------|----------------------------|
| Product X    | Full description here      |
| Product Y    | Short desc Y               |
| Product Z    | No description available   |
```

<br></br>

# Window Functions (Advanced)

Window functions perform calculations across a set of table rows that are somehow related to the current row. This is comparable to the type of calculation that can be done with an aggregate function. However, window functions do not cause rows to become grouped into a single output row like non-window aggregate calls do. Instead, the rows retain their separate identities.

### PARTITION BY

The `PARTITION BY` clause is a subclause of the `OVER` clause. It divides the rows into partitions to which the window function is applied. If `PARTITION BY` is not specified, the entire result set is treated as a single partition.

### ROW_NUMBER()

`ROW_NUMBER()` assigns a unique integer to each row within a partition of a result set.

**Example without PARTITION BY:**

```sql
SELECT
    product_name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) as row_num
FROM
    products;
```

**Output:**

```
 product_name | price | row_num
--------------+-------+---------
 Product A    |   120 |       1
 Product D    |   120 |       2
 Product B    |    80 |       3
 Product C    |    40 |       4
```

**Example with PARTITION BY:**

```sql
SELECT
    product_name,
    category,
    price,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as row_num
FROM
    products;
```

**Output:**

```
 product_name |  category  | price | row_num
--------------+------------+-------+---------
 Product A    | Category 1 |   120 |       1
 Product D    | Category 1 |   120 |       2
 Product E    | Category 1 |   100 |       3
 Product B    | Category 2 |    80 |       1
 Product C    | Category 2 |    40 |       2
```

### RANK()

`RANK()` assigns a rank to each row within a partition of a result set. The rank of a row is one plus the number of ranks that come before the row in question.

**Example without PARTITION BY:**

```sql
SELECT
    product_name,
    price,
    RANK() OVER (ORDER BY price DESC) as rank
FROM
    products;
```

**Output:**

```
 product_name | price | rank
--------------+-------+------
 Product A    |   120 |    1
 Product D    |   120 |    1
 Product B    |    80 |    3
 Product C    |    40 |    4
```

**Example with PARTITION BY:**

```sql
SELECT
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as rank
FROM
    products;
```

**Output:**

```
 product_name |  category  | price | rank
--------------+------------+-------+------
 Product A    | Category 1 |   120 |    1
 Product D    | Category 1 |   120 |    1
 Product E    | Category 1 |   100 |    3
 Product B    | Category 2 |    80 |    1
 Product C    | Category 2 |    40 |    2
```

### DENSE_RANK()

`DENSE_RANK()` assigns a rank to each row within a partition of a result set, without any gaps in the ranking. The rank of a row is one plus the number of distinct ranks that come before the row in question.

**Example without PARTITION BY:**

```sql
SELECT
    product_name,
    price,
    DENSE_RANK() OVER (ORDER BY price DESC) as dense_rank
FROM
    products;
```

**Output:**

```
 product_name | price | dense_rank
--------------+-------+------------
 Product A    |   120 |          1
 Product D    |   120 |          1
 Product B    |    80 |          2
 Product C    |    40 |          3
```

**Example with PARTITION BY:**

```sql
SELECT
    product_name,
    category,
    price,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) as dense_rank
FROM
    products;
```

**Output:**

```
 product_name |  category  | price | dense_rank
--------------+------------+-------+------------
 Product A    | Category 1 |   120 |          1
 Product D    | Category 1 |   120 |          1
 Product E    | Category 1 |   100 |          2
 Product B    | Category 2 |    80 |          1
 Product C    | Category 2 |    40 |          2
```

### LEAD() and LAG()

`LEAD()` provides access to a row at a given physical offset that follows the current row. `LAG()` provides access to a row at a given physical offset that comes before the current row.

**Example without PARTITION BY:**

```sql
SELECT
    product_name,
    price,
    LEAD(price, 1) OVER (ORDER BY price) as next_price,
    LAG(price, 1) OVER (ORDER BY price) as prev_price
FROM
    products;
```

**Output:**

```
 product_name | price | next_price | prev_price
--------------+-------+------------+------------
 Product C    |    40 |         80 |       NULL
 Product B    |    80 |        120 |         40
 Product A    |   120 |        120 |         80
 Product D    |   120 |       NULL |        120
```

**Example with PARTITION BY:**

```sql
SELECT
    product_name,
    category,
    price,
    LEAD(price, 1) OVER (PARTITION BY category ORDER BY price) as next_price,
    LAG(price, 1) OVER (PARTITION BY category ORDER BY price) as prev_price
FROM
    products;
```

**Output:**

```
 product_name |  category  | price | next_price | prev_price
--------------+------------+-------+------------+------------
 Product E    | Category 1 |   100 |        120 |       NULL
 Product A    | Category 1 |   120 |        120 |        100
 Product D    | Category 1 |   120 |       NULL |        120
 Product C    | Category 2 |    40 |         80 |       NULL
 Product B    | Category 2 |    80 |       NULL |         40
```

### NTILE()

`NTILE(n)` is a window function that distributes the rows in an ordered partition into a specified number of ranked groups (`n`). For each row, `NTILE()` returns the group number (from 1 to `n`) to which the row belongs.

**Example without PARTITION BY:**

This example divides the products into two groups based on price.

```sql
SELECT
    product_name,
    price,
    NTILE(2) OVER (ORDER BY price) as price_tile
FROM
    products;
```

**Output:**

```
 product_name | price | price_tile
--------------+-------+------------
 Product C    |    40 |          1
 Product B    |    80 |          1
 Product E    |   100 |          1
 Product A    |   120 |          2
 Product D    |   120 |          2
```

**Example with PARTITION BY:**

This example divides the products within each category into two groups.

```sql
SELECT
    product_name,
    category,
    price,
    NTILE(2) OVER (PARTITION BY category ORDER BY price) as price_tile
FROM
    products;
```

**Output:**

```
 product_name |  category  | price | price_tile
--------------+------------+-------+------------
 Product E    | Category 1 |   100 |          1
 Product A    | Category 1 |   120 |          1
 Product D    | Category 1 |   120 |          2
 Product C    | Category 2 |    40 |          1
 Product B    | Category 2 |    80 |          2
 ```
 
 ### FIRST_VALUE() and LAST_VALUE()
 
 `FIRST_VALUE()` returns the value of an expression from the first row of the window frame.
 `LAST_VALUE()` returns the value of an expression from the last row of the window frame.
 
 A critical concept for these functions is the **window frame**. By default, the frame for an `ORDER BY` clause is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This means `FIRST_VALUE()` works as expected (it sees the first row of the partition), but `LAST_VALUE()` will only see up to the *current* row, which is often not the desired behavior.
 
 To make `LAST_VALUE()` consider all rows in the partition, you must explicitly define the window frame as `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.
 
 **Example:**
 
 This example finds the cheapest and most expensive product within each category.
 
 ```sql
 SELECT
     category,
     product_name,
     price,
     FIRST_VALUE(product_name) OVER (PARTITION BY category ORDER BY price) as cheapest_in_cat,
     LAST_VALUE(product_name) OVER (
         PARTITION BY category ORDER BY price
         ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
     ) as most_expensive_in_cat
 FROM
     products;
 ```
 
 **Output:**
 
 ```
   category  | product_name | price | cheapest_in_cat | most_expensive_in_cat
 ------------+--------------+-------+-----------------+-----------------------
  Category 1 | Product E    |   100 | Product E       | Product D
  Category 1 | Product A    |   120 | Product E       | Product D
  Category 1 | Product D    |   120 | Product E       | Product D
  Category 2 | Product C    |    40 | Product C       | Product B
  Category 2 | Product B    |    80 | Product C       | Product B
 ```

## Common Table Expressions (CTE)

A Common Table Expression (CTE) is a temporary named result set that you can reference within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement. CTEs are defined using the `WITH` clause and help improve the readability and structure of complex queries.

### CTE vs. Views

| Feature | Common Table Expression (CTE) | View |
| :--- | :--- | :--- |
| **Lifecycle** | Exists only for the duration of a single query. | A persistent database object that remains until dropped. |
| **Storage** | Not stored in the database schema. Defined at the start of a query. | Stored in the database as a query definition. |
| **Usage** | Ideal for simplifying a specific, complex query. | Ideal for providing a reusable, secure, or simplified interface to tables for many different queries. |
| **Recursion** | Can be recursive, which is essential for hierarchical data queries. | Standard views cannot be recursive. |

### Non-Recursive CTE

This is the most common type of CTE. It acts like a temporary view that exists only for the duration of a single query.

**Benefits:**
- **Readability:** Breaks down complex queries into simple, logical building blocks.
- **Maintainability:** Makes it easier to understand and modify complex logic.
- **Reusability:** A CTE can be referenced multiple times within the main query.

**Example:**

This query first defines a CTE named `RegionalCustomers` to select customers from a specific region, and then the main query selects from that CTE.

```sql
WITH RegionalCustomers AS (
    SELECT customer_id, customer_name
    FROM customers
    WHERE region = 'North America'
)

-- RegionalCustomer is now CTE
SELECT *
FROM RegionalCustomers;

-- OR any query
SELECT COUNT(*) FROM RegionalCustomers;
```

**Example with Multiple CTEs:**

You can define multiple CTEs within a single `WITH` clause, separating them with commas. Subsequent CTEs can reference previously defined CTEs within the same `WITH` clause.

This example uses two CTEs: `ExpensiveProducts` and `Category1Products`, and then joins them to find products that are both expensive and belong to 'Category 1'.

```sql
WITH ExpensiveProducts AS (
    SELECT product_name, category, price
    FROM products
    WHERE price > 100
),
Category1Products AS (
    SELECT product_name, category, price
    FROM products
    WHERE category = 'Category 1'
)
SELECT
    ep.product_name,
    ep.category,
    ep.price
FROM
    ExpensiveProducts ep
JOIN
    Category1Products c1p ON ep.product_name = c1p.product_name;
```

**Output:**

```
 product_name |  category  | price
--------------+------------+-------
 Product A    | Category 1 |   120
 Product D    | Category 1 |   120
```

### Recursive CTE (Optional)

A recursive CTE is one that references itself. It consists of an "anchor" member (the initial query) and a "recursive" member that references the CTE's own name, combined with a `UNION ALL`. A termination condition in the recursive member's `WHERE` clause is crucial to prevent infinite loops.

**Use Cases:**
- Querying hierarchical data (e.g., organizational charts, bill of materials).
- Generating series or sequences.

**Example:**

This recursive CTE generates a sequence of numbers from 1 to 5.

```sql
-- The RECURSIVE keyword is required in PostgreSQL, but optional in some other dialects like SQL Server.
WITH RECURSIVE NumberSequence AS (
    -- Anchor member: the starting point of the recursion
    SELECT 1 AS n

    UNION ALL

    -- Recursive member: references the CTE itself
    SELECT n + 1
    FROM NumberSequence
    WHERE n < 5 -- Termination condition
)
-- Main query to select from the CTE
SELECT n
FROM NumberSequence;
```

**Output:**

```
 n
---
 1
 2
 3
 4
 5
```

## Data Transformation: Pivoting and Unpivoting

Pivoting and unpivoting are powerful SQL techniques for transforming the structure of your data, often used for reporting and analysis.

### Pivoting

**Pivoting** transforms data from a row-level format to a columnar format. It rotates unique values from one column into multiple new columns. Think of it as turning a "tall" table into a "wide" table.

**Example:**

Imagine you have a "tall" table of product sales by month:

**Before Pivoting:**
| product | month | sales |
|---------|-------|-------|
| Laptop  | Jan   | 1000  |
| Laptop  | Feb   | 1500  |
| Mouse   | Jan   | 200   |
| Mouse   | Feb   | 300   |

You want to pivot this data so that each month becomes its own column.

**After Pivoting:**
| product | Jan_Sales | Feb_Sales |
|---------|-----------|-----------|
| Laptop  | 1000      | 1500      |
| Mouse   | 200       | 300       |

**How it's done (Standard SQL Technique):**
The most common and portable way to pivot data, which works in virtually all SQL databases, is to use an aggregate function (like `SUM` or `MAX`) with a `CASE` statement.

```sql
SELECT
    product,
    SUM(CASE WHEN month = 'Jan' THEN sales ELSE 0 END) AS Jan_Sales,
    SUM(CASE WHEN month = 'Feb' THEN sales ELSE 0 END) AS Feb_Sales
FROM
    sales_data
GROUP BY
    product;
```
*   **Database-Specific `PIVOT` Operator:** Some databases (like SQL Server, Oracle, DB2) have a dedicated `PIVOT` operator that can simplify this syntax. However, the `CASE` statement method is more universal.
*   **PostgreSQL Specific (`FILTER` clause):** PostgreSQL offers a `FILTER` clause for aggregate functions, which can sometimes make the pivoting syntax cleaner than `CASE`:
    ```sql
    SELECT
        product,
        SUM(sales) FILTER (WHERE month = 'Jan') AS Jan_Sales,
        SUM(sales) FILTER (WHERE month = 'Feb') AS Feb_Sales
    FROM
        sales_data
    GROUP BY
        product;
    ```

### Unpivoting

**Unpivoting** is the reverse operation. It transforms data from a columnar format back into a row-level format. It rotates columns into rows. Think of it as turning a "wide" table back into a "tall" table.

**Example:**

Using the "wide" table from before:

**Before Unpivoting:**
| product | Jan_Sales | Feb_Sales |
|---------|-----------|-----------|
| Laptop  | 1000      | 1500  |
| Mouse   | 200       | 300   |

You want to unpivot this data to have a single column for the month and a single column for sales.

**After Unpivoting:**
| product | month | sales |
|---------|-------|-------|
| Laptop  | Jan   | 1000  |
| Laptop  | Feb   | 1500  |
| Mouse   | Jan   | 200   |
| Mouse   | Feb   | 300   |

**How it's done (Standard SQL Techniques):**

*   **`UNION ALL` (Universal):** This is the most common and portable way to unpivot. You use `UNION ALL` to combine multiple `SELECT` statements, each selecting one of the columns you want to turn into a row and assigning it to a common "value" column, while also creating a "key" column to identify the original column name.

    ```sql
    SELECT product, 'Jan' AS month, Jan_Sales AS sales FROM pivoted_sales
    UNION ALL
    SELECT product, 'Feb' AS month, Feb_Sales AS sales FROM pivoted_sales;
    ```

*   **`CROSS JOIN LATERAL` with `VALUES` (PostgreSQL, Oracle, some others):** This is a more advanced and often more flexible method, especially when dealing with many columns or dynamic unpivoting.

    ```sql
    SELECT p.product, v.month, v.sales
    FROM pivoted_sales p
    CROSS JOIN LATERAL (
        VALUES
        ('Jan', p.Jan_Sales),
        ('Feb', p.Feb_Sales)
    ) AS v (month, sales);
    ```
*   **Database-Specific `UNPIVOT` Operator:** Some databases (like SQL Server, Oracle, DB2) provide an `UNPIVOT` operator for this purpose.

