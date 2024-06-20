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

#### Example

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

#### Explanation

- `START TRANSACTION;`: Begins a new transaction.
- `UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;`: Decreases the balance of the sender’s account.
- `UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;`: Increases the balance of the recipient’s account.
- `COMMIT;`: If both updates are successful, commits the transaction, making the changes permanent.
- `ROLLBACK;`: If an error occurs, the transaction is rolled back, and no changes are applied to the database.

In this example, the transaction ensures that either both account balances are updated, or neither is, preserving the consistency and integrity of the database.

## What is Atomicity?

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

#### Example Scenario

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


## Isolation

https://en.wikipedia.org/wiki/Isolation_(database_systems)

In the context of database transaction isolation levels, the terms Serializable, Repeatable Read, and Read Committed refer to different levels of isolation that control the visibility of data changes made by other transactions during the execution of a transaction. Here's a breakdown of each:

### 1. Serializable Isolation Level
- **Highest level of isolation**.
- Transactions are executed in such a way that the end result is as if they were executed one after the other, sequentially.
- **Prevents:** Dirty reads, non-repeatable reads, and phantom reads.
- **Implementation:** Typically implemented using locking mechanisms that prevent other transactions from accessing the rows being read or written.
- **Use Case:** Critical applications where data consistency is paramount and the cost of reduced concurrency is acceptable.

### 2. Snapshot Isolation Level
- Provides a consistent view of the database as it was at the start of the transaction.
- **Prevents:** Dirty reads, non-repeatable reads, and phantom reads within the context of the snapshot.
- **Implementation:** Uses a versioning mechanism, where each transaction sees a snapshot of the database at a point in time.
- **Use Case:** Scenarios where consistent reads are required without locking, allowing higher concurrency compared to Serializable isolation.

### 3. Repeatable Read Isolation Level
- Ensures that if a transaction reads a row, subsequent reads of that row will return the same data, even if other transactions modify it.
- **Prevents:** Dirty reads and non-repeatable reads.
- **Does not prevent:** Phantom reads. (Phantom reads occur when a transaction reads a set of rows that satisfy a condition, but another transaction inserts or deletes rows that satisfy the same condition before the initial transaction completes.)
- **Implementation:** Uses locks on rows that are read and written, but allows new rows to be inserted that match the query conditions.
- **Use Case:** Scenarios where repeatable reads are necessary, but the application can tolerate phantom reads.

### 4. Read Committed Isolation Level
- Ensures that any data read is committed at the moment it is read. Thus, it prevents dirty reads (i.e., reading uncommitted changes from other transactions).
- **Prevents:** Dirty reads.
- **Does not prevent:** Non-repeatable reads and phantom reads.
- **Implementation:** Uses locks on rows during read and write operations but releases them immediately after the operation is done.
- **Use Case:** Commonly used in applications where performance is a concern and some degree of data inconsistency is acceptable.

### 5. Read Uncommitted Isolation Level
- **Lowest level of isolation**.
- Transactions are allowed to read uncommitted changes made by other transactions.
- **Allows:** Dirty reads, non-repeatable reads, and phantom reads.
- **Use Case:** Scenarios where maximum performance is needed, and data consistency is not critical. This is rarely used due to the risk of reading uncommitted, potentially inconsistent data.


| Isolation Level      | Prevents Dirty Reads | Prevents Non-Repeatable Reads | Prevents Phantom Reads | Concurrency | Use Case                                           |
|----------------------|----------------------|------------------------------|------------------------|-------------|----------------------------------------------------|
| Serializable         | Yes                  | Yes                          | Yes                    | Low         | Critical applications requiring strict consistency |
| Repeatable Read      | Yes                  | Yes                          | No                     | Medium      | Scenarios needing repeatable reads, tolerating phantom reads |
| Read Committed       | Yes                  | No                           | No                     | High        | Commonly used where performance is key, some inconsistency acceptable |
| Read Uncommitted     | No                   | No                           | No                     | Very High   | Scenarios where performance is critical, and data consistency is not important |
| Snapshot Isolation   | Yes                  | Yes                          | Yes (within the snapshot) | High    | Scenarios requiring consistent reads without locking, allowing high concurrency |

Understanding these isolation levels helps in choosing the right one based on the specific needs for data consistency and system performance in your application.

