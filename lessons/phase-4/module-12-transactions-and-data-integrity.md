# Module 12: Transactions and Data Integrity (MySQL)

## 1) Introduction

Scenario: You’re transferring stock between warehouses or charging a customer and creating an order. Both actions must succeed together—or not at all. Transactions ensure consistency even when failures occur.

Definition (plain):
- Transaction: A sequence of statements that execute as a single atomic unit.
- ACID: Atomicity, Consistency, Isolation, Durability—properties of reliable transactions.
- Isolation level: Controls how concurrent transactions interact (visibility of uncommitted/committed changes).

Connection to previous/next:
- Builds on DML and constraints. Next, we’ll cover stored procedures, triggers, and professional practices.

Learning objectives:
- Use START TRANSACTION/COMMIT/ROLLBACK.
- Understand isolation levels and MySQL defaults.
- Avoid common pitfalls like lost updates and dirty reads.
- Use constraints and FKs to maintain integrity.

## 2) Concept explanation

ACID:
- Atomicity: All or nothing.
- Consistency: Constraints preserved.
- Isolation: Transactions don’t interfere improperly.
- Durability: Committed changes persist through crashes.

MySQL defaults (InnoDB):
- autocommit = ON by default; each statement is its own transaction unless you START TRANSACTION.
- Default isolation = REPEATABLE READ (InnoDB). Others: READ UNCOMMITTED, READ COMMITTED, SERIALIZABLE.

Transaction control:
```sql
START TRANSACTION;  -- or BEGIN
-- DML statements here
COMMIT;             -- make changes permanent
-- or
ROLLBACK;           -- undo changes
```

Isolation levels (high level):
- READ UNCOMMITTED: Allows dirty reads (rarely desirable).
- READ COMMITTED: No dirty reads; nonrepeatable reads possible.
- REPEATABLE READ (MySQL default): No dirty or nonrepeatable reads; phantom reads possible (InnoDB has gap locking to reduce phantoms).
- SERIALIZABLE: Highest isolation; may reduce concurrency.

Set isolation level:
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
-- ...
COMMIT;
```

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS accounts;
CREATE TABLE accounts (
  account_id INT PRIMARY KEY,
  holder VARCHAR(100) NOT NULL,
  balance DECIMAL(10,2) NOT NULL CHECK (balance >= 0)
);

INSERT INTO accounts VALUES
 (1,'Alice',100.00),
 (2,'Bob',50.00);
```

### Example A — Atomic transfer
- Business question: “Transfer $20 from Alice to Bob atomically.”

SQL:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 20.00 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 20.00 WHERE account_id = 2;
-- Check balances (optional)
SELECT * FROM accounts WHERE account_id IN (1,2) FOR UPDATE;
COMMIT;
```

If any step fails (e.g., insufficient funds rule), ROLLBACK to undo partial changes.

### Example B — Rollback on error
- Business question: “Prevent negative balances.”

SQL pattern:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 2000.00 WHERE account_id = 1;
-- Validate business rule
SELECT CASE WHEN MIN(balance) < 0 THEN 1 ELSE 0 END AS bad
FROM accounts WHERE account_id = 1;
-- If bad = 1, then
ROLLBACK; -- else COMMIT
```

In practice, enforce with CHECK constraint or stored procedure logic.

### Example C — Isolation levels and reads
- Business question: “View balances consistently during concurrent updates.”

Demo (conceptual):
1) Session A: `SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; START TRANSACTION; SELECT * FROM accounts;`
2) Session B: Update balances and commit.
3) Session A: Re-select same rows and observe repeatable snapshot behavior.

## 4) Common patterns and best practices

Use cases:
1) Financial transfers, inventory moves, multi-table writes.
2) Batch imports with all-or-nothing semantics.
3) Idempotent retry-safe operations.

Templates:
```sql
-- Transaction with checks
START TRANSACTION;
UPDATE t SET ... WHERE ...;
-- optional validation queries
IF (/* validation fails */) THEN ROLLBACK; ELSE COMMIT; END IF;
```

Performance notes:
- Keep transactions short—long transactions hold locks and increase contention.
- Index rows you update to reduce lock scope.
- Avoid user interaction mid-transaction.

Do’s and Don’ts:
- Do: Explicitly start and commit/rollback for multi-statement operations.
- Do: Set the appropriate isolation level for your workload.
- Don’t: Leave transactions open; release locks quickly.
- Don’t: Use SERIALIZABLE casually; it can throttle concurrency.

Troubleshooting:
- Deadlock found when trying to get lock → Re-run the transaction; design to reduce lock conflicts (update rows in a consistent order; add indexes).
- Lock wait timeout exceeded → Inspect blocking transactions; shorten transactions; add indexes.

## 5) Practice exercises

### Exercise 1: Safe transfer
- Difficulty: Beginner
- Task: Transfer $10 from account 2 to 1 with START TRANSACTION/COMMIT.
- Solution:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 10.00 WHERE account_id = 2;
UPDATE accounts SET balance = balance + 10.00 WHERE account_id = 1;
COMMIT;
```

### Exercise 2: Rollback on failure
- Difficulty: Beginner
- Task: Attempt to subtract $500 from Bob; detect negative and ROLLBACK.
- Solution sketch: Use a validation SELECT before COMMIT; if negative, ROLLBACK.

### Exercise 3: Isolation level test
- Difficulty: Intermediate
- Task: In two sessions, show that REPEATABLE READ returns the same snapshot until commit.
- Solution: Follow Example C steps.

### Exercise 4: Prevent negative balances with constraints
- Difficulty: Intermediate
- Task: Enforce non-negative balance via CHECK or trigger (Module 14 for triggers).
- Solution:
```sql
ALTER TABLE accounts
MODIFY balance DECIMAL(10,2) NOT NULL CHECK (balance >= 0);
```

### Exercise 5: Multi-table atomic write
- Difficulty: Advanced
- Task: Within one transaction, insert a new order and related items; ROLLBACK on any error.
- Solution: Wrap INSERTS in START TRANSACTION/COMMIT and handle errors.

## 6) Self-assessment checklist
- Can you use START TRANSACTION/COMMIT/ROLLBACK? (Ex. 1/2)
- Can you explain ACID? (Concept)
- Can you describe isolation levels and MySQL defaults? (Ex. 3)
- Can you design transactions to avoid deadlocks and long locks?

## 7) Connection to real work
- Scenarios: Payments, order booking, ledger updates.
- Roles: Backend Engineer, DB Engineer, FinTech Ops.
- Interview questions:
  - Q: What is a deadlock and how do you mitigate it?
    - A: Cyclic lock dependency; mitigate by consistent update ordering, proper indexes, shorter transactions, retry logic.
  - Q: Why is autocommit important in MySQL?
    - A: Each statement commits automatically unless you start a transaction explicitly.
  - Q: When would you use READ COMMITTED over REPEATABLE READ?
    - A: To reduce contention and allow readers to see committed changes immediately; trade off repeatable snapshots.
