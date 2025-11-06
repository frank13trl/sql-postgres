# Introduction to SQL

Structured Query Language (SQL) is a standard language for managing and manipulating relational databases. It is used to perform tasks such as retrieving data, updating data, and managing database schemas.

## Types of SQL Commands

SQL commands are broadly categorized into several types based on their functionality.

### 1. Data Definition Language (DDL)

DDL commands are used to define and manage the database structure or schema. These commands are responsible for creating, altering, and deleting database objects.

*   **`CREATE`**: Used to create new database objects like tables, views, indexes, and databases.
    ```sql
    CREATE TABLE employees (
        id INT PRIMARY KEY,
        name VARCHAR(100)
    );
    ```

*   **`ALTER`**: Used to modify the structure of an existing database object, such as adding or dropping columns in a table.
    ```sql
    ALTER TABLE employees
    ADD COLUMN email VARCHAR(100);
    ```

*   **`DROP`**: Used to delete existing database objects.
    ```sql
    DROP TABLE employees;
    ```

*   **`TRUNCATE`**: Used to remove all records from a table, but the table structure remains. It is faster than `DELETE` for emptying a table.
    ```sql
    TRUNCATE TABLE employees;
    ```

### 2. Data Manipulation Language (DML)

DML commands are used for managing data within the database objects.

*   **`INSERT`**: Used to add new rows of data into a table.
    ```sql
    INSERT INTO employees (id, name) VALUES (1, 'John Doe');
    ```

*   **`UPDATE`**: Used to modify existing records in a table.
    ```sql
    UPDATE employees
    SET name = 'Jane Doe'
    WHERE id = 1;
    ```

*   **`DELETE`**: Used to remove existing records from a table.
    ```sql
    DELETE FROM employees WHERE id = 1;
    ```

### 3. Data Query Language (DQL)

DQL is used to retrieve data from the database. The primary command for DQL is `SELECT`.

*   **`SELECT`**: Used to query the database and retrieve data that matches criteria that you specify.
    ```sql
    SELECT name, email FROM employees WHERE id = 1;
    ```

### 4. Data Control Language (DCL)

DCL commands are used to manage user access and permissions to the database.

*   **`GRANT`**: Used to give a user access privileges to the database.
    ```sql
    GRANT SELECT, UPDATE ON employees TO user_jane;
    ```

*   **`REVOKE`**: Used to take back permissions from a user.
    ```sql
    REVOKE SELECT, UPDATE ON employees FROM user_jane;
    ```

### 5. Transaction Control Language (TCL)

TCL commands are used to manage transactions in the database. Transactions are sequences of operations performed as a single logical unit of work.

*   **`COMMIT`**: Saves all the transactions to the database since the last `COMMIT` or `ROLLBACK`.
    ```sql
    COMMIT;
    ```

*   **`ROLLBACK`**: Undoes transactions that have not already been saved to the database.
    ```sql
    ROLLBACK;
    ```

*   **`SAVEPOINT`**: Sets a point within a transaction to which you can later roll back.
    ```sql
    SAVEPOINT my_savepoint;
    ```
