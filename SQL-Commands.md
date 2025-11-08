# SQL Learning Commands

This document will serve as a reference for SQL commands as I learn them.

## PostgreSQL Service Management (Arch Linux)

These commands are used to manage the PostgreSQL database server on Arch Linux.

*   **Check PostgreSQL service status:**
    ```bash
    sudo systemctl status postgresql
    ```

*   **Initialize PostgreSQL database cluster (if needed):**
    ```bash
    sudo -u postgres initdb --locale en_US.UTF-8 -D /var/lib/postgres/data
    ```
    *Note: This command should only be run once to set up the initial database directory.*

*   **Start PostgreSQL service:**
    ```bash
    sudo systemctl start postgresql
    ```

*   **Enable PostgreSQL service to start on boot:**
    ```bash
    sudo systemctl enable postgresql
    ```

## Connecting to PostgreSQL Database

These commands are used to connect to a running PostgreSQL database.

*   **Connect using `psql` (general):**
    ```bash
    psql -U <username> -d <database_name> -h <host> -p <port>
    ```
    *Replace `<username>`, `<database_name>`, `<host>`, and `<port>` with your specific details.*

*   **Connecting without specifying a database:**
    When you don't specify a database with `-d`, `psql` attempts to connect to a database with the same name as the user. If that database doesn't exist, it may result in an error.

*   **Connect to the default `postgres` database:**
    ```bash
    psql -U <your_username> -d postgres
    ```

*   **Connect as `postgres` user (local):**
    ```bash
    sudo -u postgres psql -d postgres
    ```

*   **Connect with specified user and database (local):**
    ```bash
    psql -U myuser -d mydb
    ```

## Creating a Database

*   **Create a new database:**
    ```sql
    CREATE DATABASE <database_name>;
    ```
    *To simplify login, you can create a database with the same name as your user.*

## Common PostgreSQL Data Types

For a complete list, see the [official documentation](https://www.postgresql.org/docs/current/datatype.html).

### Numeric Types
*   `INTEGER` or `INT`: 4-byte integer.
*   `SMALLINT`: 2-byte integer.
*   `BIGINT`: 8-byte integer.
*   `NUMERIC(precision, scale)` or `DECIMAL(precision, scale)`: Exact decimal numbers.
*   `REAL` or `FLOAT4`: 4-byte floating-point number.
*   `DOUBLE PRECISION` or `FLOAT8`: 8-byte floating-point number.
*   `SERIAL`: Auto-incrementing 4-byte integer.
*   `BIGSERIAL`: Auto-incrementing 8-byte integer.

### Character Types
*   `VARCHAR(n)`: Variable-length string.
*   `CHAR(n)`: Fixed-length string.
*   `TEXT`: Variable-length string with no limit.

### Date/Time Types
*   `DATE`: Date (year, month, day).
*   `TIME`: Time of day.
*   `TIMESTAMP`: Date and time.
*   `TIMESTAMPTZ`: Date and time with time zone.
*   `INTERVAL`: Period of time.

### Boolean Type
*   `BOOLEAN`: `true` or `false`.

### Other Useful Types
*   `UUID`: Universally unique identifier.
*   `JSON`: JSON data.
*   `JSONB`: JSON data (binary format).
*   `ARRAY`: Array of any data type.
*   **`ENUM` (Enumerated Type):** A user-defined type that comprises a static, ordered set of values.
    *   **Create an ENUM type:**
        ```sql
        CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
        ```
    *   **Use an ENUM type in a table:**
        ```sql
        CREATE TABLE persons (
            person_id SERIAL PRIMARY KEY,
            name VARCHAR(100),
            current_mood mood
        );
        ```
    *   **Insert data with an ENUM type:**
        ```sql
        INSERT INTO persons (name, current_mood) VALUES ('Alice', 'happy');
        INSERT INTO persons (name, current_mood) VALUES ('Bob', 'sad');
        ```

## Table Management

### Table Constraints

Constraints are rules enforced on data columns in a table. They are used to limit the type of data that can go into a table, ensuring the accuracy and reliability of the data.

#### `PRIMARY KEY`

A **Primary Key** is a special relational database table column (or combination of columns) designated to uniquely identify all table records. It serves several critical purposes:

1.  **Unique Identification:** Each value in the primary key column(s) must be unique across all rows in the table. This ensures that every record can be uniquely identified.
2.  **Non-Nullability:** A primary key column cannot contain `NULL` values. Every record must have a primary key value.
3.  **Data Integrity:** It enforces entity integrity, ensuring that each row in the table is distinct and can be referenced reliably.
4.  **Relationships:** Primary keys are often referenced by foreign keys in other tables to establish relationships between them.

**Characteristics of a good Primary Key:**
*   **Unique:** No two rows can have the same primary key value.
*   **Non-null:** Cannot contain `NULL` values.
*   **Stable:** The value of a primary key should rarely, if ever, change.
*   **Simple:** Often a single column, but can be a composite of multiple columns.
*   **Relevant:** While not strictly necessary, it's often a column that naturally identifies the entity (e.g., `student_id`, `product_id`).

**Example (from previous `students` table):**

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY, -- student_id is the primary key
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```

#### Composite Primary Keys

A table can only have **one primary key**, but that single primary key can be composed of **multiple columns**. This is called a **composite primary key**.

A composite primary key ensures that the combination of values in those specified columns is unique for each row in the table. No single column in the composite key needs to be unique on its own, but their combination must be.

**Example:** Consider a table `enrollments` that tracks which student is enrolled in which course. The combination of `student_id` and `course_id` would uniquely identify a single enrollment record.

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id), -- Composite Primary Key
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

In this example:
*   `PRIMARY KEY (student_id, course_id)` defines a composite primary key using both `student_id` and `course_id`.
*   This means that a student (identified by `student_id`) can only be enrolled in a specific course (identified by `course_id`) once.
*   Both `student_id` and `course_id` are also foreign keys, linking back to their respective `students` and `courses` tables (assuming those tables exist).

#### `FOREIGN KEY`

A **foreign key** is a key used to link two tables together. It is a field (or collection of fields) in one table that refers to the `PRIMARY KEY` in another table.

**Example:** Create a `courses` table with a foreign key that references a `students` table.

```sql
-- First, create the parent table (if it doesn't exist)
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE
);

-- Then, create the child table with the foreign key
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    student_id INT,
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);
```

#### `ON DELETE` Operations

The `ON DELETE` clause is used with foreign keys to define what happens to the rows in the child table when the corresponding row in the parent table is deleted. This is crucial for maintaining referential integrity.

*   **`ON DELETE RESTRICT` (or `ON DELETE NO ACTION`)**: This is the default behavior in most SQL databases. It prevents you from deleting a row from the parent table if there are any corresponding rows in the child table. You must first delete the rows in the child table.

*   **`ON DELETE CASCADE`**: When a row in the parent table is deleted, all corresponding rows in the child table are automatically deleted as well. This is useful for cleaning up related data.

    **Example:**
    ```sql
    CREATE TABLE orders (
        order_id SERIAL PRIMARY KEY,
        customer_id INT,
        order_date DATE,
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
    );
    -- If a customer is deleted, all their orders will be deleted too.
    ```

*   **`ON DELETE SET NULL`**: When a row in the parent table is deleted, the foreign key column(s) in the corresponding rows of the child table are set to `NULL`. This is only possible if the foreign key column in the child table allows `NULL` values.

    **Example:**
    ```sql
    CREATE TABLE products (
        product_id SERIAL PRIMARY KEY,
        product_name VARCHAR(100),
        supplier_id INT,
        FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id) ON DELETE SET NULL
    );
    -- If a supplier is deleted, the supplier_id for their products will be set to NULL.
    ```

*   **`ON DELETE SET DEFAULT`**: When a row in the parent table is deleted, the foreign key column(s) in the corresponding rows of the child table are set to their default value. This requires that a default value is defined for the foreign key column.


#### `NOT NULL`

*   **Purpose:** Ensures that a column cannot have a `NULL` value. This means that a value must always be provided for this column when a new row is inserted or an existing row is updated.
*   **Example:**
    ```sql
    CREATE TABLE users (
        user_id SERIAL PRIMARY KEY,
        username VARCHAR(50) NOT NULL, -- username cannot be NULL
        email VARCHAR(100)
    );
    ```

#### `UNIQUE`

*   **Purpose:** Ensures that all values in a column (or a combination of columns) are different. While a `PRIMARY KEY` automatically has a `UNIQUE` constraint, you can have multiple `UNIQUE` constraints on a table.
*   **Characteristics:** A `UNIQUE` column can contain `NULL` values, but only one `NULL` value (as `NULL` is not considered equal to `NULL`).
*   **Example:**
    ```sql
    CREATE TABLE employees (
        employee_id SERIAL PRIMARY KEY,
        employee_email VARCHAR(100) UNIQUE, -- employee_email must be unique
        phone_number VARCHAR(20) UNIQUE -- phone_number must also be unique
    );
    ```

#### `CHECK`

*   **Purpose:** Ensures that all values in a column satisfy a specific condition. This allows you to define custom rules for your data.
*   **Example:**
    ```sql
    CREATE TABLE products_with_check (
        product_id SERIAL PRIMARY KEY,
        product_name VARCHAR(100),
        price NUMERIC(10, 2) CHECK (price > 0), -- price must be greater than 0
        stock_quantity INT CHECK (stock_quantity >= 0) -- stock_quantity must be non-negative
    );
    ```

#### `DEFAULT`

*   **Purpose:** Provides a default value for a column when no value is explicitly specified during an `INSERT` operation.
*   **Example:**
    ```sql
    CREATE TABLE tasks (
        task_id SERIAL PRIMARY KEY,
        task_description TEXT NOT NULL,
        status VARCHAR(20) DEFAULT 'pending', -- default status is 'pending'
        created_at TIMESTAMP DEFAULT NOW() -- default created_at is the current timestamp
    );
    ```
    When inserting into `tasks` without specifying `status` or `created_at`:
    ```sql
    INSERT INTO tasks (task_description) VALUES ('Buy groceries');
    -- status will be 'pending', created_at will be the current timestamp
    ```

### Auto-Incrementing Columns (`SERIAL` and `IDENTITY`)

An auto-incrementing column is a column (typically a primary key) whose value is automatically generated and incremented by the database system whenever a new row is inserted into the table. This ensures that each new record gets a unique identifier without manual intervention.

#### 1. `SERIAL` and `BIGSERIAL` (PostgreSQL-specific)

*   **How it works:** `SERIAL` is a convenience pseudo-type that automatically creates a sequence object and sets the column's default value to `nextval()` from that sequence. It also adds a `NOT NULL` constraint.
    *   `SERIAL` is equivalent to `INT` (4-byte integer) + sequence.
    *   `BIGSERIAL` is equivalent to `BIGINT` (8-byte integer) + sequence, for larger ranges.
*   **Usage:**
    ```sql
    CREATE TABLE products_serial (
        product_id SERIAL PRIMARY KEY, -- Automatically increments
        product_name VARCHAR(100)
    );
    ```

#### 2. `IDENTITY` Columns (SQL Standard, PostgreSQL 10+)

*   **How it works:** `IDENTITY` columns are the SQL standard way to create auto-incrementing columns. They are more explicit and offer better control than `SERIAL`. You can specify `ALWAYS` or `BY DEFAULT`.
    *   `GENERATED ALWAYS AS IDENTITY`: The column value is *always* generated by the sequence. You cannot explicitly insert a value into this column.
    *   `GENERATED BY DEFAULT AS IDENTITY`: The column value is generated by default, but you *can* explicitly provide a value during `INSERT` if you wish.
*   **Usage:**
    ```sql
    CREATE TABLE products_identity (
        product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Always generated
        product_name VARCHAR(100)
    );

    CREATE TABLE orders_identity (
        order_id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY, -- Generated by default, but can be overridden
        order_description TEXT
    );
    ```

#### 3. `AUTO_INCREMENT` (MySQL-specific)

*   **How it works:** In MySQL, you use the `AUTO_INCREMENT` keyword with an integer primary key column to make it auto-incrementing.
*   **Usage:**
    ```sql
    CREATE TABLE products_mysql (
        product_id INT PRIMARY KEY AUTO_INCREMENT, -- Automatically increments in MySQL
        product_name VARCHAR(100)
    );
    ```

### Altering Auto-Increment Start Value

Modifying the starting value of an auto-incrementing column (sequence) is a common task, especially when migrating data or resetting test environments. The method varies significantly between database systems.

#### PostgreSQL (`SERIAL` / `IDENTITY` columns)

In PostgreSQL, `SERIAL` and `BIGSERIAL` columns are backed by sequences. `IDENTITY` columns also use sequences. To alter the starting value, you need to modify the associated sequence.

1.  **Find the sequence name:** The sequence associated with a `SERIAL` or `IDENTITY` column is typically named `<table_name>_<column_name>_seq`.
    *   You can find the sequence name by describing the table:
        ```sql
        \d <table_name>
        ```
        Look for the `Default` column for your `SERIAL` or `IDENTITY` column; it will show `nextval('<sequence_name>'::regclass)`. The `<sequence_name>` is what you need.

2.  **Alter the sequence:** Use `ALTER SEQUENCE` to set the next value.

    **Example:** To set the next value for `product_id` in `products_serial` to 100:
    ```sql
    -- Assuming the sequence name is products_serial_product_id_seq
    ALTER SEQUENCE products_serial_product_id_seq RESTART WITH 100;
    ```
    *   **Note:** `RESTART WITH N` sets the *next* value that the sequence will generate. If you want the next inserted row to have ID `N`, you use `RESTART WITH N`.

#### Other Databases (Brief Mention)

*   **MySQL**: Uses `ALTER TABLE` directly.
    **Example:** To set the next `id` for the `products` table to 100 in MySQL:
    ```sql
    ALTER TABLE products AUTO_INCREMENT = 100;
    ```
    *   **Note:** This sets the *next* value that the auto-increment column will generate. If you want the next inserted row to have ID `N`, you use `AUTO_INCREMENT = N`.

*   **SQL Server**: Uses `DBCC CHECKIDENT`.
    **Example:** To set the next `id` for the `products` table to 100 in SQL Server:
    ```sql
    DBCC CHECKIDENT ('products', RESEED, 99); -- Reseeds to 99, so the next value will be 100
    ```

### Creating a Table

**Example:** Create a table with various common data types.

```sql
-- First, ensure the ENUM type 'mood' exists if you want to use it
-- CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

CREATE TABLE comprehensive_data_types (
    -- Primary Key
    id SERIAL PRIMARY KEY,

    -- Numeric Types
    small_num SMALLINT,
    regular_num INT,
    large_num BIGINT,
    exact_decimal NUMERIC(15, 5), -- 15 total digits, 5 after decimal
    approx_float REAL,
    double_precision_float DOUBLE PRECISION,
    money_value MONEY, -- Currency type

    -- Character Types
    fixed_char CHAR(10),
    variable_char VARCHAR(255),
    long_text TEXT,

    -- Date/Time Types
    just_date DATE,
    just_time TIME,
    datetime_no_tz TIMESTAMP WITHOUT TIME ZONE,
    datetime_with_tz TIMESTAMP WITH TIME ZONE,
    duration INTERVAL,

    -- Boolean Type
    is_active BOOLEAN,

    -- Binary Data
    binary_data BYTEA, -- For storing binary strings/byte arrays

    -- Network Address Types
    ip_address INET,
    mac_address MACADDR,

    -- UUID Type
    unique_id UUID DEFAULT gen_random_uuid(), -- Requires pgcrypto extension or similar for gen_random_uuid()

    -- JSON Types
    json_data JSON,
    jsonb_data JSONB,

    -- Array Types
    int_array INT[],
    text_array TEXT[],

    -- Custom ENUM Type (requires prior creation: CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');)
    user_mood mood,

    -- Geometric Types (examples)
    point_coords POINT,
    line_segment LSEG,

    -- Range Types (examples)
    int_range INT4RANGE,
    date_range DATERANGE
);
```

### Inserting Data

The `INSERT` statement is used to add new rows of data into a table.

#### Listing Column Names

It is not strictly necessary to list the column names in an `INSERT` statement, but it is **highly recommended**. If you omit the column names, you **must** provide a value for **every** column in the table, in the exact order they are defined. This is fragile and can easily break if the table structure changes.

By listing the column names, your `INSERT` statement becomes more readable, robust, and flexible.

#### Using the `DEFAULT` Keyword

Instead of omitting a column to use its default value, you can explicitly use the `DEFAULT` keyword in the `VALUES` clause. This can sometimes make the `INSERT` statement clearer, as it explicitly shows that the default value is being used for a specific column.

**Example:** Using `DEFAULT` with the `tasks` table from the `DEFAULT` constraint example.

```sql
-- Create the tasks table if it doesn't exist
CREATE TABLE IF NOT EXISTS tasks (
    task_id SERIAL PRIMARY KEY,
    task_description TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert a new task, explicitly using the default for the 'status' column
INSERT INTO tasks (task_description, status) VALUES ('Clean the kitchen', DEFAULT);
    -- In this case, the new row will have 'pending' as its status.
```

#### Inserting Multiple Rows

You can insert multiple rows into a table with a single `INSERT` statement by providing multiple sets of values, separated by commas.

**Example:** Inserting multiple tasks into the `tasks` table.

```sql
INSERT INTO tasks (task_description, status)
VALUES
    ('Walk the dog', 'completed'),
    ('Pay bills', DEFAULT),
    ('Schedule dentist appointment', 'pending');
```

#### Inserting NULL Values Explicitly

To insert a `NULL` value into a column that allows it (i.e., does not have a `NOT NULL` constraint), you can explicitly specify `NULL` in the `VALUES` clause.

**Example:** Inserting a new student where the email is not yet known.

```sql
-- Assuming a students table exists with student_id, first_name, last_name, email
CREATE TABLE IF NOT EXISTS students (
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);

INSERT INTO students (first_name, last_name, email) VALUES ('John', 'Doe', NULL);
```

**Example:** Inserting data into the `comprehensive_data_types` table.

```sql
INSERT INTO comprehensive_data_types (
    small_num,
    regular_num,
    large_num,
    exact_decimal,
    approx_float,
    double_precision_float,
    money_value,
    fixed_char,
    variable_char,
    long_text,
    just_date,
    just_time,
    datetime_no_tz,
    datetime_with_tz,
    duration,
    is_active,
    binary_data,
    ip_address,
    mac_address,
    json_data,
    jsonb_data,
    int_array,
    text_array,
    user_mood,
    point_coords,
    line_segment,
    int_range,
    date_range
) VALUES (
    100,
    10000,
    10000000000,
    12345.67890,
    3.14,
    3.1415926535,
    '500.25',
    'FixedText',
    'Variable text example',
    'This is a longer piece of text for the TEXT column.',
    '2023-10-27',
    '14:30:00',
    '2023-10-27 14:30:00',
    '2023-10-27 14:30:00+02',
    '1 day 2 hours 30 minutes',
    TRUE,
    E'\\xDEADBEEF', -- Corrected escape for bytea literal
    '192.168.1.1',
    '08:00:2b:01:02:03',
    '{"key": "value", "number": 123}',
    '{"key": "value", "number": 123}',
    '{1,2,3}',
    '{"apple", "banana"}',
    'happy',
    '(10,20)',
    '((1,1),(5,5))',
    '[1, 10)',
    '[2023-01-01, 2023-01-31)'
);
```

### Showing Table Structure

Use the `\d` meta-command in `psql` to view a table's structure.

**Example:** Show the structure of the `comprehensive_data_types` table.

```sql
\d comprehensive_data_types
```

### Dropping vs. Truncating a Table

*   **`DROP TABLE`**: This command completely removes the table, including its structure, data, indexes, constraints, and triggers. The table will no longer exist and you would need to recreate it from scratch.
    ```sql
    DROP TABLE <table_name>;
    ```

*   **`TRUNCATE TABLE`**: This command removes all rows from a table, but the table structure (columns, constraints, etc.) remains. It is a DDL (Data Definition Language) command and is generally much faster than `DELETE` for emptying a table, as it doesn't scan every row before removing it. It also does not fire any `DELETE` triggers.
    ```sql
    TRUNCATE TABLE <table_name>;
    ```

#### Key Differences:
*   **Structure:** `DROP` removes the table structure, `TRUNCATE` does not.
*   **Speed:** `TRUNCATE` is faster than `DELETE` for large tables.
*   **Triggers:** `TRUNCATE` does not fire `DELETE` triggers.
*   **Rollback:** `DROP` and `TRUNCATE` are DDL commands and cannot be easily rolled back in the same way DML commands like `DELETE` can be within a transaction.

## ALTER TABLE Commands

The `ALTER TABLE` command is used to add, delete, or modify columns in an existing table. You can also use it to add and drop various constraints on an existing table.

*   **Add a new column:**
    ```sql
    ALTER TABLE <table_name>
    ADD COLUMN <column_name> <data_type>;
    ```

*   **Drop a column:**
    ```sql
    ALTER TABLE <table_name>
    DROP COLUMN <column_name>;
    ```

*   **Rename a column:**
    ```sql
    ALTER TABLE <table_name>
    RENAME COLUMN <old_name> TO <new_name>;
    ```

*   **Change a column's data type:**
    ```sql
    ALTER TABLE <table_name>
    ALTER COLUMN <column_name> TYPE <new_data_type>;
    ```

*   **Add a NOT NULL constraint:**
    ```sql
    ALTER TABLE <table_name>
    ALTER COLUMN <column_name> SET NOT NULL;
    ```

*   **Drop a NOT NULL constraint:**
    ```sql
    ALTER TABLE <table_name>
    ALTER COLUMN <column_name> DROP NOT NULL;
    ```

*   **Add a `DEFAULT` constraint:**
    ```sql
    ALTER TABLE <table_name>
    ALTER COLUMN <column_name> SET DEFAULT <value>;
    ```

*   **Add a UNIQUE constraint:**
    ```sql
    ALTER TABLE <table_name>
    ADD CONSTRAINT <constraint_name> UNIQUE (<column_name>);
    ```

*   **Add a PRIMARY KEY constraint:**
    ```sql
    ALTER TABLE <table_name>
    ADD PRIMARY KEY (<column_name>);
    ```

*   **Drop a constraint (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK):**
    ```sql
    ALTER TABLE <table_name>
    DROP CONSTRAINT <constraint_name>;
    ```

*   **Rename a table:**
    ```sql
    ALTER TABLE <old_table_name> RENAME TO <new_table_name>;
    ```

*   **Reorder Columns (Limitation in PostgreSQL):**
    PostgreSQL does not provide a direct `ALTER TABLE` command to reorder columns within a table. The physical order of columns is not considered logically important in the relational model.

    *   **Workaround:** The recommended approach is to create a new table with the desired column order and migrate the data, or to create a `VIEW` with the columns in the desired order for querying.

    *   **Comparison with MySQL:** Other database systems, like MySQL, do provide syntax for this. For example:
        ```sql
        -- MySQL Example: Move a column to be the first column
        ALTER TABLE <table_name> MODIFY COLUMN <column_name> <column_definition> FIRST;

        -- MySQL Example: Move a column after another column
        ALTER TABLE <table_name> MODIFY COLUMN <column_name> <column_definition> AFTER <other_column_name>;
        ```

### `CHANGE` and `MODIFY` (MySQL Specific)

In MySQL, `ALTER TABLE` provides `CHANGE COLUMN` and `MODIFY COLUMN` clauses to alter existing columns. While both can change a column's definition, `CHANGE COLUMN` is also used to rename a column.

*   **`MODIFY COLUMN`:** Used to change the definition of an existing column (e.g., data type, constraints) without renaming it.

    **Example: Change data type and add NOT NULL constraint**
    ```sql
    ALTER TABLE products
    MODIFY COLUMN product_name VARCHAR(200) NOT NULL;
    ```

*   **`CHANGE COLUMN`:** Used to change the definition of an existing column and/or rename it. When renaming, you must specify both the old and new column names, along with the new column definition.

    **Example: Rename a column and change its data type**
    ```sql
    ALTER TABLE customers
    CHANGE COLUMN old_email_address new_email VARCHAR(255);
    ```

    **Example: Change data type without renaming (similar to MODIFY)**
    ```sql
    ALTER TABLE orders
    CHANGE COLUMN order_date order_date DATE NOT NULL;
    ```

## SELECT Queries

The `SELECT` statement is used to query the database and retrieve data that matches criteria that you specify.

*   **Select all columns from a table:**
    ```sql
    SELECT * FROM <table_name>;
    ```

*   **Select specific columns from a table:**
    ```sql
    SELECT <column1>, <column2> FROM <table_name>;
    ```

*   **Filter results with `WHERE`:**
    ```sql
    SELECT * FROM <table_name> WHERE <condition>;
    ```
    *Example:*
    ```sql
    SELECT * FROM students WHERE student_id = 1;
    ```

*   **Check for NULL values with `IS NULL` or `IS NOT NULL`:**
    ```sql
    SELECT * FROM <table_name> WHERE <column_name> IS NULL;
    SELECT * FROM <table_name> WHERE <column_name> IS NOT NULL;
    ```
    *Example:*
    ```sql
    SELECT * FROM employees WHERE phone_number IS NULL;
    ```

*   **Combine conditions with Logical Operators (`AND`, `OR`, `NOT`):**
    ```sql
    SELECT * FROM <table_name> WHERE <condition1> AND <condition2>;
    SELECT * FROM <table_name> WHERE <condition1> OR <condition2>;
    SELECT * FROM <table_name> WHERE NOT <condition>;
    ```
    *Example:*
    ```sql
    SELECT * FROM students WHERE enrollment_date > '2023-01-01' AND first_name = 'Alice';
    SELECT * FROM products WHERE price < 10 OR stock_quantity = 0;
    ```

*   **Match values in a list with `IN`:**
    ```sql
    SELECT * FROM <table_name> WHERE <column_name> IN (<value1>, <value2>, ...);
    ```
    *Example:*
    ```sql
    SELECT * FROM students WHERE student_id IN (1, 3, 5);
    ```

*   **Select values within a range with `BETWEEN`:**
    The `BETWEEN` operator selects values within a given range. The values can be numbers, text, or dates. The range is inclusive.
    ```sql
    SELECT * FROM <table_name> WHERE <column_name> BETWEEN <value1> AND <value2>;
    ```
    *Example:*
    ```sql
    SELECT * FROM products WHERE price BETWEEN 10.00 AND 20.00;
    SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-01-31';
    ```

*   **Sort results with `ORDER BY`:**
    ```sql
    SELECT * FROM <table_name> ORDER BY <column_name> [ASC | DESC];
    ```
    *`ASC` for ascending (default), `DESC` for descending.*

*   **Limit the number of results with `LIMIT`:**
    ```sql
    SELECT * FROM <table_name> LIMIT <number>;
    ```

*   **Skip results with `OFFSET`:**
    The `OFFSET` clause is used with `LIMIT` to skip a specified number of rows before beginning to return rows. This is commonly used for pagination.

    *Example: Get the second page of 5 students (skipping the first 5).*
    ```sql
    SELECT * FROM students
    ORDER BY student_id
    LIMIT 5 OFFSET 5;
    ```

*   **Limit results with `FETCH FIRST` / `OFFSET ... FETCH NEXT` (SQL Standard):**
    This syntax is part of the SQL Standard (SQL:2008 and later) and is supported by databases like Oracle, DB2, and increasingly PostgreSQL (as an alternative to `LIMIT`). It provides a standard way to limit results and perform pagination.

    *Example: Get the first 10 students.*
    ```sql
    SELECT * FROM students
    ORDER BY student_id
    FETCH FIRST 10 ROWS ONLY;
    ```

    *Example: Get the second page of 5 students (skipping the first 5).*
    ```sql
    SELECT * FROM students
    ORDER BY student_id
    OFFSET 5 ROWS
    FETCH NEXT 5 ROWS ONLY;
    ```

*   **Select unique values with `DISTINCT`:**
    ```sql
    SELECT DISTINCT <column_name> FROM <table_name>;
    ```

*   **Alias columns and tables with `AS`:**
    The `AS` keyword is used to rename a column or table with an alias. An alias only exists for the duration of that query.

    **Column Alias Example:**
    ```sql
    SELECT customer_name AS name, contact_person AS contact
    FROM customers;
    ```

    **Table Alias Example:**
    ```sql
    SELECT o.order_id, c.customer_name
    FROM orders AS o, customers AS c
    WHERE o.customer_id = c.customer_id;
    ```

*   **Filter with `LIKE` and Wildcards:**
    The `LIKE` operator is used in a `WHERE` clause to search for a specified pattern in a column. SQL wildcards are used to substitute for one or more characters in a string.

    *   **`%` (Percent sign):** Represents zero, one, or multiple characters.
        *Example: Find all customers whose names start with 'A'.*
        ```sql
        SELECT * FROM customers WHERE customer_name LIKE 'A%';
        ```
        *Example: Find all customers whose names contain 'an'.*
        ```sql
        SELECT * FROM customers WHERE customer_name LIKE '%an%';
        ```
        *Example: Find all customers whose names end with 'a'.*
        ```sql
        SELECT * FROM customers WHERE customer_name LIKE '%a';
        ```

    *   **`_` (Underscore):** Represents a single character.
        *Example: Find all customers whose names have 'a' as the second letter.*
        ```sql
        SELECT * FROM customers WHERE customer_name LIKE '_a%';
        ```
        *Example: Find all customers whose names start with 'B' and are 4 letters long.*
        ```sql
        SELECT * FROM customers WHERE customer_name LIKE 'B___';
        ```

*   **Group rows with `GROUP BY`:**
    The `GROUP BY` statement groups rows that have the same values in specified columns into summary rows. It is often used with aggregate functions (`COUNT`, `MAX`, `MIN`, `SUM`, `AVG`) to perform calculations on each group.

    *Example: Count the number of students in each course.*
    ```sql
    SELECT course_id, COUNT(student_id) AS number_of_students
    FROM enrollments
    GROUP BY course_id;
    ```

*   **Combining `WHERE` and `GROUP BY`:**
    The `WHERE` clause is used to filter individual rows *before* they are grouped, while `HAVING` filters groups *after* they have been formed.

    *Example: Count the number of students in each course, but only for enrollments made after a specific date.*
    ```sql
    SELECT course_id, COUNT(student_id) AS number_of_students
    FROM enrollments
    WHERE enrollment_date > '2023-01-01'
    GROUP BY course_id;
    ```

*   **Filter groups with `HAVING`:**
    The `HAVING` clause was added to SQL because the `WHERE` keyword could not be used with aggregate functions. `HAVING` enables you to filter groups based on the results of aggregate functions.

    *Example: Find courses that have more than 5 students.*
    ```sql
    SELECT course_id, COUNT(student_id) AS number_of_students
    FROM enrollments
    GROUP BY course_id
    HAVING COUNT(student_id) > 5;
    ```

*   **Summarize data with `ROLLUP`:**
    The `ROLLUP` extension to `GROUP BY` is used to generate subtotals and grand totals for a set of columns. It creates a hierarchy of groupings, from the most detailed level to a grand total.

    *Example: Get the count of students per course, and also the total count of all students.*
    ```sql
    SELECT course_id, COUNT(student_id) AS number_of_students
    FROM enrollments
    GROUP BY ROLLUP(course_id);
    ```
    *This will show the student count for each `course_id`, and a final row where `course_id` is `NULL` which represents the grand total of all students across all courses.*

    *   **MySQL `ROLLUP` Syntax:**
        In MySQL, `ROLLUP` is used directly within the `GROUP BY` clause, similar to the standard SQL syntax.
        ```sql
        SELECT course_id, COUNT(student_id) AS number_of_students
        FROM enrollments
        GROUP BY course_id WITH ROLLUP;
        ```
        *Note: The `WITH ROLLUP` modifier is appended to the `GROUP BY` clause in MySQL.*

## UPDATE Queries

The `UPDATE` statement is used to modify existing records in a table.

*   **Update a single column for a specific record:**
    ```sql
    UPDATE <table_name>
    SET <column1> = <value1>
    WHERE <condition>;
    ```
    *Example:*
    ```sql
    UPDATE students
    SET email = 'new.email@example.com'
    WHERE student_id = 1;
    ```

*   **Update multiple columns for a specific record:**
    ```sql
    UPDATE <table_name>
    SET <column1> = <value1>,
        <column2> = <value2>
    WHERE <condition>;
    ```
    *Example:*
    ```sql
    UPDATE students
    SET first_name = 'Johnny', last_name = 'Smith'
    WHERE student_id = 2;
    ```

*   **Update all records in a table (use with caution):**
    If you omit the `WHERE` clause, all records in the table will be updated.
    ```sql
    UPDATE <table_name>
    SET <column1> = <value1>;
    ```

*   **Using Numeric Functions in `UPDATE`:**
    You can use arithmetic operators and functions to update numeric columns based on their current values.
    ```sql
    UPDATE <table_name>
    SET <numeric_column> = <numeric_column> + <value>
    WHERE <condition>;
    ```
    *Example:*
    ```sql
    UPDATE products
    SET price = price * 1.10 -- Increase price by 10%
    WHERE category = 'Electronics';

    UPDATE products
    SET stock_quantity = stock_quantity - 5
    WHERE product_id = 101;
    ```

## DELETE Queries

The `DELETE` statement is used to remove existing records from a table.

*   **Delete specific rows based on a condition:**
    ```sql
    DELETE FROM <table_name>
    WHERE <condition>;
    ```
    *Example:*
    ```sql
    DELETE FROM tasks
    WHERE status = 'completed';
    ```

*   **Delete all rows from a table (use with extreme caution):**
    If you omit the `WHERE` clause, all records in the table will be deleted. This is irreversible.
    ```sql
    DELETE FROM <table_name>;
    ```

## Common SQL Functions and Constants

### Date/Time Functions

*   **`CURRENT_TIMESTAMP`** or **`NOW()`**: Returns the current date and time (with time zone) at which the current transaction started.
    *   **Usage in `INSERT`:**
        ```sql
        INSERT INTO orders (order_description, created_at) VALUES ('New order', CURRENT_TIMESTAMP);
        ```
    *   **Usage as a `DEFAULT` value:**
        ```sql
        CREATE TABLE tasks (
            task_id SERIAL PRIMARY KEY,
            task_description TEXT NOT NULL,
            created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
        );
        ```

*   **`CURRENT_DATE`**: Returns the current date.

*   **`CURRENT_TIME`**: Returns the current time of day.

## psql Meta-Commands and SQL Queries

These commands are used within the `psql` interactive terminal.

*   **List all databases:**
    ```sql
    \l
    ```
    or
    ```sql
    SELECT datname FROM pg_database;
    ```

*   **Connect to a different database:**
    ```sql
    \c <database_name>
    ```

*   **List all tables in the current database:**
    ```sql
    \dt
    ```

*   **Describe a table (columns, data types, etc.):**
    ```sql
    \d <table_name>
    ```

*   **Describe a table in detail (including constraints, etc.):**
    ```sql
    \d+ <table_name>
    ```

*   **List all schemas:**
    ```sql
    \dn
    ```

*   **List all users (roles):**
    ```sql
    \du
    ```

*   **List all views:**
    ```sql
    \dv
    ```

*   **List all functions:**
    ```sql
    \df
    ```

*   **Get help on psql meta-commands:**
    ```sql
    \?
    ```

*   **Get help on a specific SQL command:**
    ```sql
    \h <SQL_COMMAND>
    ```
    (e.g., `\h SELECT`)

*   **Quit psql:**
    ```sql
    \q
    ```
