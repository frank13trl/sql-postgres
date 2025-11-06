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
