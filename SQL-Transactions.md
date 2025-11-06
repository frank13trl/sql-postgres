# SQL Transaction Control Language (TCL)

Transaction Control Language (TCL) commands are used to manage transactions in the database. A transaction is a sequence of operations performed as a single logical unit of work. The key purpose of transactions is to maintain the ACID properties (Atomicity, Consistency, Isolation, Durability) of a database.

## Key TCL Commands

### `BEGIN` vs `START TRANSACTION`

To initiate a transaction, you can use either `BEGIN` or `START TRANSACTION`. In PostgreSQL, they are functionally equivalent.

*   **`BEGIN`**: A PostgreSQL-specific command that is widely understood.
*   **`START TRANSACTION`**: The SQL standard command for beginning a transaction. It is generally preferred for better portability across different SQL databases.

### `COMMIT`

The `COMMIT` command is used to permanently save any changes made during the current transaction to the database. Once a transaction is committed, the changes cannot be undone by `ROLLBACK`.

**Example:**

```sql
BEGIN TRANSACTION; -- Start a transaction (syntax may vary, e.g., START TRANSACTION, BEGIN)

UPDATE accounts
SET balance = balance - 100
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE account_id = 2;

COMMIT; -- Permanently save the changes
```

### `ROLLBACK`

The `ROLLBACK` command is used to undo all changes made during the current transaction since the last `COMMIT` or `ROLLBACK`. This restores the database to the state it was in before the transaction began.

**Example:**

```sql
BEGIN TRANSACTION;

INSERT INTO orders (product_id, quantity) VALUES (101, 5);

-- Something goes wrong, decide to undo the insert
ROLLBACK; -- Undo the insert operation
```

### `SAVEPOINT`

A `SAVEPOINT` (or `SAVE TRANSACTION`) command is used to set a point within a transaction to which you can later roll back. This allows for partial rollbacks, meaning you can undo some changes without rolling back the entire transaction.

**Example:**

```sql
BEGIN TRANSACTION;

INSERT INTO logs (message) VALUES ('Operation A started');
SAVEPOINT savepoint_a;

UPDATE users SET status = 'active' WHERE user_id = 1;
SAVEPOINT savepoint_b;

-- Suppose an error occurs after savepoint_b, but before the end of the transaction
-- We want to undo only the changes made after savepoint_a
ROLLBACK TO savepoint_a; -- Undoes the UPDATE, but keeps the INSERT

-- Continue with other operations or COMMIT/ROLLBACK the entire transaction
COMMIT;
```

## Auto-Commit

By default, most database systems and clients operate in an "auto-commit" mode. In this mode, every single SQL statement is treated as a separate transaction and is automatically committed immediately after it is executed. This means you do not need to explicitly use the `COMMIT` command for each statement.

When you explicitly start a transaction with `BEGIN TRANSACTION` (or a similar command), you are temporarily disabling auto-commit for the duration of that transaction. This allows you to group multiple statements into a single, atomic unit of work. The transaction ends when you issue a `COMMIT` or `ROLLBACK` command, at which point the database returns to its default auto-commit mode.

### When to Change Auto-Commit Settings

You generally do not need to explicitly enable auto-commit, as it is the default behavior. You would only need to manually change the auto-commit setting in specific situations:

*   **Disabling Auto-Commit**: If you want to run multiple statements and have the option to `ROLLBACK` without starting an explicit transaction block, you can disable auto-commit. When auto-commit is disabled, you must explicitly use `COMMIT` to save your changes.
    ```sql
    -- This is not a standard SQL command and varies by client/driver
    SET autocommit = 0; -- or SET autocommit = OFF;
    ```

*   **Re-enabling Auto-Commit**: You would only need to explicitly re-enable auto-commit if you or an application had previously disabled it.
    ```sql
    -- This is not a standard SQL command and varies by client/driver
    SET autocommit = 1; -- or SET autocommit = ON;
    ```

## ACID Properties of Transactions

Transactions are crucial for maintaining the integrity and reliability of data in a database. They adhere to the following four properties:

*   **Atomicity**: A transaction is treated as a single, indivisible unit of work. Either all of its operations are completed successfully, or none of them are. There are no partial transactions.
*   **Consistency**: A transaction brings the database from one valid state to another. It ensures that all data integrity rules (constraints, triggers, etc.) are maintained before and after the transaction.
*   **Isolation**: The execution of concurrent transactions results in a system state that would be achieved if transactions were executed serially (one after another). This prevents interference between transactions.
*   **Durability**: Once a transaction has been committed, its changes are permanent and will survive system failures (e.g., power outages, crashes).