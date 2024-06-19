## Transaction

A database transaction is a sequence of one or more SQL operations executed as a single unit of work. These operations can include `INSERT`, `UPDATE`, `DELETE`, and `SELECT` statements. The key characteristic of a transaction is that it ensures the ACID properties, which stand for Atomicity, Consistency, Isolation, and Durability. These properties help maintain the integrity and reliability of the database.

### ACID Properties

1. **Atomicity**:
   - Ensures that all operations within a transaction are completed successfully. If any operation fails, the entire transaction is rolled back, and no changes are applied to the database.
   - Example: Transferring money from one account to another involves two operations: debiting the sender's account and crediting the recipient's account. Both operations must succeed, or neither should be applied.

2. **Consistency**:
   - Ensures that a transaction brings the database from one valid state to another, maintaining all predefined rules, such as constraints, cascades, and triggers.
   - Example: After a transaction, the total balance across all accounts should remain unchanged.

3. **Isolation**:
   - Ensures that transactions are executed in isolation from one another. The changes made by a transaction are not visible to other transactions until the transaction is committed.
   - Example: If two transactions are updating the same account balance, each transaction will see the account balance as it was before the other transaction started, preventing conflicts.

4. **Durability**:
   - Ensures that once a transaction is committed, the changes are permanent and will survive system failures, such as crashes or power outages.
   - Example: After committing a transaction, the changes to the account balances are stored permanently in the database, even if the system crashes immediately after the commit.

### Lifecycle of a Transaction

1. **Begin Transaction**:
   - Marks the start of a new transaction.
   - SQL: `BEGIN TRANSACTION;` or `START TRANSACTION;`.

2. **Perform Operations**:
   - Execute one or more SQL operations as part of the transaction.
   - SQL: `INSERT`, `UPDATE`, `DELETE`, `SELECT`.

3. **Commit or Rollback**:
   - **Commit**: Finalizes the transaction, making all changes permanent.
     - SQL: `COMMIT;`.
   - **Rollback**: Reverts all changes made during the transaction, restoring the database to its previous state.
     - SQL: `ROLLBACK;`.

### Example

Here’s a simple example of a database transaction that transfers money from one account to another:

```sql
-- Start the transaction
START TRANSACTION;

-- Decrease the balance from the sender's account
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- Increase the balance in the recipient's account
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Commit the transaction if all operations are successful
COMMIT;

-- If an error occurs, rollback the transaction
ROLLBACK;
```

### Explanation

- `START TRANSACTION;`: Begins a new transaction.
- `UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;`: Decreases the balance of the sender’s account.
- `UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;`: Increases the balance of the recipient’s account.
- `COMMIT;`: If both updates are successful, commits the transaction, making the changes permanent.
- `ROLLBACK;`: If an error occurs, the transaction is rolled back, and no changes are applied to the database.

In this example, the transaction ensures that either both account balances are updated, or neither is, preserving the consistency and integrity of the database.

### What is Atomicity?

Atomicity is one of the four ACID properties of database transactions, which ensure that database operations are processed reliably. Atomicity specifically guarantees that a series of database operations within a transaction are treated as a single "atomic" unit. This means that either all operations within the transaction are completed successfully, or none of them are applied to the database. There are no partial transactions.

In essence, atomicity ensures the "all-or-nothing" principle:

- **All Operations Complete Successfully**: If every operation in the transaction is executed without error, the transaction is committed, and all changes are permanently applied to the database.
- **Any Operation Fails**: If any operation within the transaction fails, the entire transaction is rolled back, and the database is restored to the state it was in before the transaction began.

### What Happens When a SQL Server Crashes in the Middle of a Transaction?

When a SQL server crashes in the middle of a transaction, atomicity ensures that the transaction is handled in a way that maintains the integrity of the database. Here’s what typically happens:

1. **Incomplete Transactions Are Rolled Back**: Upon recovery, the SQL server will identify any transactions that were incomplete at the time of the crash. These transactions will be rolled back, meaning any partial changes made by the transaction will be undone.

2. **Transaction Log**: SQL servers use transaction logs to keep track of all operations performed by transactions. The transaction log records every operation so that in the event of a crash, the server can determine the state of each transaction (whether it was committed or not).

3. **Recovery Process**:
   - **Redo Phase**: The server will redo any committed transactions that were not fully written to the database before the crash. This ensures that all committed changes are applied.
   - **Undo Phase**: The server will undo any transactions that were not committed before the crash. This removes any partial changes, maintaining the atomicity and consistency of the database.

### Example Scenario

Consider a transaction that transfers money from Account A to Account B:

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

If the server crashes after the first `UPDATE` but before the `COMMIT`, the database could be in an inconsistent state. However, due to atomicity and the use of transaction logs:

1. Upon restarting, the SQL server checks the transaction log.
2. The server detects that the transaction was not committed.
3. The server rolls back the `UPDATE` to Account A, restoring its balance to the original state.
4. The server ensures no partial updates remain, maintaining database integrity.

In summary, atomicity ensures that even in the event of a crash, the database can recover to a consistent state by rolling back any incomplete transactions. This guarantees that no partial transactions are left in the database, preserving the all-or-nothing nature of transactions.


### Read Phenomenons in Database Transactions

#### 1. Dirty Read
**Definition:** A dirty read occurs when a transaction reads data that has been written by another transaction but not yet committed. If the other transaction is rolled back, the data read by the first transaction becomes invalid.

**Example:**

```sql
-- Transaction 1
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Transaction 2
SELECT balance FROM accounts WHERE account_id = 1; -- Dirty Read
-- Transaction 1
ROLLBACK;
```

In this example, Transaction 2 reads the balance that Transaction 1 has updated but not yet committed. If Transaction 1 rolls back, Transaction 2's read becomes invalid.

**Prevention:** Set the isolation level to `READ COMMITTED`.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

#### 2. Non-repeatable Read
**Definition:** A non-repeatable read occurs when a transaction reads the same row twice and gets different values because another transaction has modified and committed the data in between the two reads.

**Example:**

```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE account_id = 1; -- Read 1
-- Transaction 2
BEGIN;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 1;
COMMIT;
-- Transaction 1
SELECT balance FROM accounts WHERE account_id = 1; -- Read 2
COMMIT;
```

In this example, Transaction 1 reads the balance twice, but between the two reads, Transaction 2 updates and commits the balance.

**Prevention:** Set the isolation level to `REPEATABLE READ`.

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

#### 3. Phantom Reads
**Definition:** A phantom read occurs when a transaction reads a set of rows that satisfy a condition, but another transaction inserts or deletes rows that satisfy the same condition before the first transaction is complete.

**Example:**

```sql
-- Transaction 1
BEGIN;
SELECT COUNT(*) FROM accounts WHERE balance > 1000; -- Read 1
-- Transaction 2
BEGIN;
INSERT INTO accounts (account_id, balance) VALUES (5, 1500);
COMMIT;
-- Transaction 1
SELECT COUNT(*) FROM accounts WHERE balance > 1000; -- Read 2
COMMIT;
```

In this example, Transaction 1 reads the count of accounts with a balance greater than 1000 twice. Between the two reads, Transaction 2 inserts a new account that satisfies the condition.

**Prevention:** Set the isolation level to `SERIALIZABLE`.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

#### 4. Lost Updates
**Definition:** A lost update occurs when two transactions both read the same data and then update it based on the value read. The second update overwrites the first update without knowing about the changes made by the first transaction.

**Example:**

```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE account_id = 1; -- Read
-- Transaction 2
BEGIN;
SELECT balance FROM accounts WHERE account_id = 1; -- Read
-- Transaction 1
UPDATE accounts SET balance = balance + 100 WHERE account_id = 1;
COMMIT;
-- Transaction 2
UPDATE accounts SET balance = balance + 200 WHERE account_id = 1;
COMMIT;
```

In this example, both transactions read the same balance, but the second transaction's update overwrites the first transaction's update.

**Prevention:** Use locking mechanisms or higher isolation levels.

```sql
-- Using Explicit Locks
BEGIN;
SELECT balance FROM accounts WHERE account_id = 1 FOR UPDATE;
-- Perform update
COMMIT;
```

### Summary of Isolation Levels

1. **READ UNCOMMITTED**: Allows dirty reads.
2. **READ COMMITTED**: Prevents dirty reads but allows non-repeatable reads and phantom reads.
3. **REPEATABLE READ**: Prevents dirty reads and non-repeatable reads but allows phantom reads.
4. **SERIALIZABLE**: Prevents dirty reads, non-repeatable reads, and phantom reads.

These isolation levels ensure the consistency and integrity of data in concurrent transactions.

