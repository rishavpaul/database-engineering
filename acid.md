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
