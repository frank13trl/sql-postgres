# SQL Stored Procedures and Functions

This document provides a comprehensive guide to understanding and using stored procedures and functions in SQL.

## Introduction

Stored procedures and functions are reusable blocks of code that are stored in the database. They allow you to encapsulate complex logic, improve performance, and enforce security. By using them, you can write a piece of SQL code once and execute it multiple times with different parameters.

### Key Benefits:
*   **Modularity:** Break down complex problems into smaller, manageable, and reusable units of code.
*   **Performance:** Stored procedures are compiled and stored in the database, which can lead to faster execution times compared to ad-hoc queries.
*   **Reduced Network Traffic:** Instead of sending long SQL statements over the network, you can simply call the stored procedure or function.
*   **Security:** You can grant users permission to execute a stored procedure without giving them direct access to the underlying tables.
*   **Consistency:** Ensures that business logic is implemented consistently across different applications.

### Key Differences:

| Feature              | Stored Procedure                                  | Function                                           |
| -------------------- | ------------------------------------------------- | -------------------------------------------------- |
| **Return Value**     | Can return zero, one, or multiple values (using `OUT` parameters). | Must return a single value.                      |
| **Invocation**       | Can be called independently using `CALL` or `EXECUTE`. | Can be called from within a SQL statement (e.g., `SELECT`). |
| **Side Effects**     | Can have side effects (e.g., `INSERT`, `UPDATE`, `DELETE`). | Primarily used for computations and should not have side effects. |
| **Transaction Mgmt** | Can manage transactions (`COMMIT`, `ROLLBACK`).   | Cannot manage transactions.                        |

## Changing the Delimiter

When writing stored procedures or functions in a command-line client (like MySQL's `mysql` client or Oracle's `SQL*Plus`), you need to redefine the delimiter. The delimiter is the character or string of characters that signals the end of a SQL statement (by default, it's the semicolon `;`).

Stored procedures and functions can contain multiple SQL statements, each ending with a semicolon. If you don't change the delimiter, the client will interpret the first semicolon it encounters as the end of your `CREATE PROCEDURE` or `CREATE FUNCTION` statement, leading to a syntax error.

By changing the delimiter to something else (like `$$` or `//`), you can write the entire procedure or function, including all its internal semicolons, and then use your new delimiter to signal the end of the entire block.

**Example (MySQL):**

```sql
-- Change the delimiter to $$
DELIMITER $$

CREATE PROCEDURE GetAllProducts()
BEGIN
    SELECT * FROM products;
END$$

-- Change the delimiter back to ;
DELIMITER ;
```

*   **Note:** The `DELIMITER` command is a client-side command and is not part of the SQL standard. It is primarily used in MySQL clients. In other clients, like PostgreSQL's `psql`, this is not necessary as `psql` is smart enough to handle multiline statements within a `CREATE FUNCTION` or `CREATE PROCEDURE` block without needing to change the delimiter.

## Creating a Stored Procedure

The `CREATE PROCEDURE` statement is used to create a stored procedure. The exact syntax can vary slightly between different database systems.

**Basic Syntax (MySQL):**

```sql
CREATE PROCEDURE procedure_name(parameter1 datatype, parameter2 datatype, ...)
BEGIN
    -- SQL statements
END;
```

**Basic Syntax (PostgreSQL):**

```sql
CREATE OR REPLACE PROCEDURE procedure_name(parameter1 datatype, parameter2 datatype, ...)
LANGUAGE plpgsql
AS $$
BEGIN
    -- SQL statements
END;
$$;
```

### Parameters

Stored procedures can accept parameters. There are three types of parameters:

*   `IN`: The default mode. The calling program passes the argument, and the procedure can use it. The procedure cannot modify the original value.
*   `OUT`: The procedure can change the value and pass it back to the calling program.
*   `INOUT`: A combination of `IN` and `OUT`. The calling program passes a value, the procedure can modify it, and the new value is available to the calling program.

### Variables in Procedures

Using variables in stored procedures is fundamental for storing and manipulating values within the procedure's logic. The syntax for declaring and using variables differs between database systems.

**MySQL:**

In MySQL, you use the `DECLARE` keyword to define a variable and its data type, and then `SET` to assign a value to it.

```sql
DECLARE my_variable INT;
SET my_variable = 10;
-- Or directly assign from a query:
SELECT COUNT(*) INTO my_variable FROM employees WHERE department = 'Sales';
```

**PostgreSQL:**

In PostgreSQL, you declare a variable with its name and data type in the `DECLARE` section of the procedure/function, and then use the `:=` operator for assignment within the `BEGIN...END` block.

```sql
DECLARE
    my_variable INT;
BEGIN
    my_variable := 10;
    -- Or directly assign from a query:
    SELECT COUNT(*) INTO my_variable FROM employees WHERE department = 'Sales';
    -- ...
END;
```

### Example: Create a procedure to get employees by department

This example creates a procedure that accepts a department name as an input parameter and returns all employees in that department.

**MySQL Example:**

```sql
DELIMITER $$

CREATE PROCEDURE GetEmployeesByDepartment(IN department_name VARCHAR(100))
BEGIN
    SELECT * FROM employees WHERE department = department_name;
END$$

DELIMITER ;

-- To call the procedure:
CALL GetEmployeesByDepartment('Sales');
```

**PostgreSQL Example:**

In PostgreSQL, to return a result set from a procedure-like structure, a function that returns a table is more idiomatic.

```sql
CREATE OR REPLACE FUNCTION GetEmployeesByDepartment(department_name VARCHAR(100))
RETURNS TABLE(employee_id INT, first_name VARCHAR, last_name VARCHAR, department VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT e.employee_id, e.first_name, e.last_name, e.department
    FROM employees e
    WHERE e.department = department_name;
END;
$$;

-- To call the function:
SELECT * FROM GetEmployeesByDepartment('Sales');
```

### Simple Procedure Example: Adding Two Numbers

Here are examples of procedures (or functions in PostgreSQL, which is more common for returning a single value) to add two numbers:

**MySQL Example (using an `OUT` parameter to return the sum):**

```sql
DELIMITER $$

CREATE PROCEDURE AddTwoNumbers(IN num1 INT, IN num2 INT, OUT sum_result INT)
BEGIN
    SET sum_result = num1 + num2;
END$$

DELIMITER ;

-- How to call it and get the result:
SET @result = 0;
CALL AddTwoNumbers(5, 7, @result);
SELECT @result; -- This will output 12
```

**PostgreSQL Example (using a function to return the sum):**

```sql
CREATE OR REPLACE FUNCTION AddTwoNumbers(num1 INT, num2 INT)
RETURNS INT
LANGUAGE plpgsql
AS $$
DECLARE
    sum_result INT;
BEGIN
    sum_result := num1 + num2;
    RETURN sum_result;
END;
$$;

-- How to call it:
SELECT AddTwoNumbers(5, 7); -- This will output 12
```


*   **Invocation in PostgreSQL:** While PostgreSQL 11+ introduced the `CALL` statement for procedures (which do not return values directly), functions that return values (including table sets) are typically invoked within a `SELECT` statement, as shown above. For procedures that perform actions without returning a value, you would use `CALL procedure_name(arguments);`.

*   **Note on `RETURNS TABLE`:** Yes, the `RETURNS TABLE` clause is necessary in this PostgreSQL example. It defines the structure of the table that the function will return. Without it, PostgreSQL would not know the column names and data types of the result set. This is a key difference from how some other databases, like MySQL, handle result sets from procedures.

## Viewing Stored Procedures

Once you have created stored procedures, you will need to be able to view them. The commands for this vary between database systems.

### MySQL

*   **List all stored procedures:**
    You can get a list of all stored procedures in the current database with the following command:
    ```sql
    SHOW PROCEDURE STATUS WHERE Db = 'your_database_name';
    ```

*   **View the code of a stored procedure:**
    To see the `CREATE PROCEDURE` statement for an existing procedure, you can use:
    ```sql
    SHOW CREATE PROCEDURE procedure_name;
    ```

### PostgreSQL

*   **List all functions and procedures:**
    In PostgreSQL, you can use the `\df` meta-command in `psql` to list all functions and procedures.
    ```sql
    \df
    ```

*   **View the code of a function or procedure:**
    To see the source code of a specific function or procedure, you can use the `\df+` meta-command followed by the function name.
    ```sql
    \df+ function_name
    ```
    Alternatively, you can query the `pg_proc` system catalog:
    ```sql
    SELECT prosrc FROM pg_proc WHERE proname = 'function_name';
    ```

## Dropping Stored Procedures

To remove a stored procedure from the database, you use the `DROP PROCEDURE` statement. It's good practice to use `IF EXISTS` to avoid errors if the procedure does not exist.

### MySQL

```sql
DROP PROCEDURE [IF EXISTS] procedure_name;
```

**Example:**

```sql
DROP PROCEDURE IF EXISTS GetEmployeesByDepartment;
```

### PostgreSQL

In PostgreSQL, you use `DROP PROCEDURE` for procedures and `DROP FUNCTION` for functions. You also need to specify the parameter types if there are multiple procedures/functions with the same name (overloading).

*   **Dropping a Procedure:**
    ```sql
    DROP PROCEDURE [IF EXISTS] procedure_name([argmode] [argname] argtype [, ...]);
    ```

*   **Dropping a Function:**
    ```sql
    DROP FUNCTION [IF EXISTS] function_name([argmode] [argname] argtype [, ...]);
    ```

**Example (PostgreSQL Procedure):**

```sql
DROP PROCEDURE IF EXISTS GetAllEmployees;
```

**Example (PostgreSQL Function with parameters):**

```sql
DROP FUNCTION IF EXISTS GetEmployeesByDepartment(VARCHAR(100));
```
<br></br>

# SQL Triggers

## Introduction to SQL Triggers

A trigger is a special type of stored procedure that automatically runs when a specific event occurs in the database. These events are typically `INSERT`, `UPDATE`, or `DELETE` operations on a specific table. Triggers are used to maintain data integrity, enforce complex business rules, and automate tasks.

### Key Components of a Trigger:

*   **Event:** The DML (Data Manipulation Language) operation that causes the trigger to fire (e.g., `INSERT`, `UPDATE`, `DELETE`).
*   **Timing:** When the trigger should fire. This can be `BEFORE` or `AFTER` the event.
*   **Table:** The table on which the trigger is defined.
*   **Trigger Body:** The block of code that is executed when the trigger fires.

### When to Use Triggers:

*   **Auditing:** Logging changes to a table.
*   **Data Validation:** Enforcing complex validation rules that cannot be handled by constraints.
*   **Maintaining Data Integrity:** Keeping related data in sync across multiple tables.
*   **Automating Tasks:** Performing actions based on changes to a table, such as sending notifications or updating summary tables.

## Creating Triggers

The `CREATE TRIGGER` statement is used to create a trigger. The syntax differs between MySQL and PostgreSQL.

### MySQL

**Basic Syntax:**

```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE} ON table_name
FOR EACH ROW
BEGIN
    -- Trigger body
END;
```

**Example: Log new employee insertions**

This trigger logs the `employee_id`, `employee_email`, and the current date into an `employee_audit` table every time a new employee is added. (This assumes the `employee_audit` table has an `employee_email` column).

```sql
DELIMITER $$

CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, employee_email, action, action_date)
    VALUES (NEW.employee_id, NEW.employee_email, 'INSERT', NOW());
END$$

DELIMITER ;
```

*   **`NEW` and `OLD` Keywords:** In MySQL, you can use special keywords to access the columns of the rows affected by a trigger:
    *   `NEW`: Refers to the new row of data. It is available in `INSERT` and `UPDATE` triggers. For example, `NEW.column_name` gives you the value of `column_name` for the row being inserted or the new value for the row being updated.
    *   `OLD`: Refers to the old row of data. It is available in `UPDATE` and `DELETE` triggers. For example, `OLD.column_name` gives you the value of `column_name` for the row being deleted or the old value for the row being updated.

### PostgreSQL

**Basic Syntax:**

```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE} ON table_name
[FOR EACH ROW | FOR EACH STATEMENT]
EXECUTE FUNCTION trigger_function_name();
```

In PostgreSQL, creating a trigger involves two steps:

1.  **Create a trigger function:** This is a special function that contains the code that will be executed when the trigger fires.
2.  **Create the trigger:** This binds the trigger function to a table and specifies the event and timing.

*   **Note on the two-step process:** Yes, this is a fundamental design choice in PostgreSQL. It separates the trigger's logic (the function) from its execution context (the trigger itself). This allows for greater flexibility, as the same trigger function can be reused by multiple triggers on different tables.

**1. Create the trigger function:**

```sql
CREATE OR REPLACE FUNCTION log_employee_insert()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO employee_audit (employee_id, employee_email, action, action_date)
    VALUES (NEW.employee_id, NEW.employee_email, 'INSERT', NOW());
    RETURN NEW;
END;
$$;
```

**2. Create the trigger:**

```sql
CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
EXECUTE FUNCTION log_employee_insert();
```

*   **`NEW` and `OLD` Records:** In PostgreSQL, `NEW` and `OLD` are special record variables that contain the data for the row being affected by the trigger:
    *   `NEW`: A record variable containing the new database row for `INSERT` and `UPDATE` operations in row-level triggers.
    *   `OLD`: A record variable containing the old database row for `UPDATE` and `DELETE` operations in row-level triggers.
    *   In a `BEFORE` trigger, you can modify the values in the `NEW` record before they are written to the table. For example, `NEW.column_name := some_value;`.

## Viewing Triggers

To inspect existing triggers in your database, the commands vary by system.

### MySQL

*   **List all triggers:**
    ```sql
    SHOW TRIGGERS;
    ```

*   **View the code of a trigger:**
    ```sql
    SHOW CREATE TRIGGER trigger_name;
    ```

### PostgreSQL

*   **List all triggers for a table:**
    You can use the `\d` meta-command in `psql` to describe a table, which will also list its triggers.
    ```sql
    \d table_name
    ```

*   **View the code of a trigger function:**
    Since PostgreSQL triggers execute functions, you view the function's code:
    ```sql
    \df+ trigger_function_name
    ```
    Or query `pg_proc`:
    ```sql
    SELECT prosrc FROM pg_proc WHERE proname = 'trigger_function_name';
    ```

## Dropping Triggers

To remove a trigger from the database, you use the `DROP TRIGGER` statement.

### MySQL

```sql
DROP TRIGGER [IF EXISTS] trigger_name;
```

**Example:**

```sql
DROP TRIGGER IF EXISTS after_employee_insert;
```

### PostgreSQL

In PostgreSQL, you need to specify the table on which the trigger is defined.

```sql
DROP TRIGGER [IF EXISTS] trigger_name ON table_name;
```

**Example:**

```sql
DROP TRIGGER IF EXISTS after_employee_insert ON employees;
```

*   **Note:** When you drop a trigger, the associated trigger function (in PostgreSQL) is not automatically dropped. If the function is no longer needed, you should drop it manually using `DROP FUNCTION`.