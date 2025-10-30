# Module 9: Data Modification — DML Operations (MySQL)

## 1) Introduction

Scenario: You need to load new products, update stock, and remove discontinued items safely. DML (Data Manipulation Language) lets you INSERT, UPDATE, and DELETE data correctly and predictably.

Definition (plain):
- INSERT: Add new rows.
- UPDATE: Change existing rows.
- DELETE: Remove rows.
- INSERT ... SELECT: Insert results of a query.
- ON DUPLICATE KEY UPDATE: Upsert-like behavior in MySQL.

Connection to previous/next:
- You’ll use DML after cleaning and analyzing data. Next, we’ll learn DDL/schema design, indexing, and transactions.

Learning objectives:
- Write safe INSERT/UPDATE/DELETE with WHERE.
- Use INSERT ... SELECT to copy/transform data.
- Upsert with ON DUPLICATE KEY UPDATE.
- Understand safe-update practices and audit-friendly patterns.

## 2) Concept explanation

INSERT variants:
```sql
INSERT INTO t (col1, col2) VALUES (v1, v2), (v3, v4);
INSERT INTO t SELECT col1, col2 FROM other;
INSERT IGNORE INTO t ...; -- ignores certain errors (e.g., duplicate keys)
REPLACE INTO t ...;        -- delete+insert semantics when key conflict occurs (use carefully)
```

UPDATE:
```sql
UPDATE t
SET col = expr, col2 = expr2
WHERE condition; -- always include WHERE unless truly updating all rows
```

DELETE:
```sql
DELETE FROM t
WHERE condition; -- always include WHERE unless truly deleting all rows
```

Upsert (MySQL-specific):
```sql
INSERT INTO t (id, val)
VALUES (1,'a')
ON DUPLICATE KEY UPDATE val = VALUES(val);
```

Common misconceptions:
- “Leaving out WHERE is fine.” Dangerous—can update/delete all rows.
- “REPLACE is the same as UPDATE.” No—REPLACE deletes and inserts; it can reset auto-increment and affect FKs.
- “INSERT IGNORE is harmless.” It can hide data issues; prefer explicit handling.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS inventory;
CREATE TABLE inventory (
  sku VARCHAR(20) PRIMARY KEY,
  product_name VARCHAR(100) NOT NULL,
  stock INT NOT NULL,
  price DECIMAL(10,2) NOT NULL
);

INSERT INTO inventory VALUES
 ('SKU-1','Wireless Mouse',50,25.00),
 ('SKU-2','Keyboard',12,85.00),
 ('SKU-3','Desk Lamp',5,19.99);
```

### Example A — INSERT single and multiple rows
- Business question: “Load two new products.”

SQL:
```sql
INSERT INTO inventory (sku, product_name, stock, price) VALUES
 ('SKU-4','Water Bottle',0,15.00),
 ('SKU-5','Notebook',300,3.50);
```

Verify:
```sql
SELECT * FROM inventory ORDER BY sku;
```

### Example B — UPDATE with safe WHERE
- Business question: “Increase price by 5% for products under $20.”

SQL:
```sql
UPDATE inventory
SET price = ROUND(price * 1.05, 2)
WHERE price < 20.00;
```

Verify:
```sql
SELECT sku, product_name, price FROM inventory ORDER BY sku;
```

### Example C — Upsert with ON DUPLICATE KEY UPDATE
- Business question: “Load stock counts daily—insert new SKUs, update existing.”

SQL:
```sql
INSERT INTO inventory (sku, product_name, stock, price) VALUES
 ('SKU-3','Desk Lamp',7,19.99), -- existing
 ('SKU-6','USB-C Cable',100,8.00) -- new
ON DUPLICATE KEY UPDATE
  stock = VALUES(stock),
  price = VALUES(price);
```

Verify:
```sql
SELECT * FROM inventory WHERE sku IN ('SKU-3','SKU-6') ORDER BY sku;
```

Portability notes:
- PostgreSQL: Use INSERT ... ON CONFLICT (key) DO UPDATE.
- SQL Server: Use MERGE (with caution) or separate UPDATE/INSERT logic.

## 4) Common patterns and best practices

Use cases:
1) Batch loading with INSERT ... SELECT.
2) Safe updates using precise WHERE filters.
3) Idempotent upserts for pipelines.
4) Soft deletes (mark disabled_at) instead of hard DELETE.
5) Audit columns (created_at, updated_at, updated_by).

Templates:
```sql
-- Insert from another table
INSERT INTO t_dest (id, val)
SELECT id, val
FROM t_src
WHERE valid_flag = 1;

-- Soft delete pattern
UPDATE users
SET disabled_at = NOW(), disabled_by = 'admin'
WHERE user_id = 123;

-- Upsert
INSERT INTO products (product_id, name, price)
VALUES (?,?,?)
ON DUPLICATE KEY UPDATE price = VALUES(price);
```

Performance notes:
- Batch inserts are faster than many single-row inserts.
- Indexes speed reads but slow writes; balance appropriately.
- Wrap multi-step changes in transactions (Module 12) for atomicity.

Do’s and Don’ts:
- Do: Always have a WHERE in UPDATE/DELETE (unless intended).
- Do: Test updates with SELECT of the same WHERE first.
- Don’t: Use REPLACE casually; it can delete and reinsert.
- Don’t: Hide errors with INSERT IGNORE unless you’ve planned for it.

Troubleshooting:
- “Duplicate entry for key PRIMARY” → Use ON DUPLICATE KEY UPDATE for upserts.
- “0 rows affected” → Check WHERE condition; values may already match.
- “Foreign key constraint fails” → Insert parents first; maintain referential integrity.

## 5) Practice exercises

### Exercise 1: Insert a new product
- Difficulty: Beginner
- Task: Insert SKU-7 'Mouse Pad' with stock 40, price 6.00.
- Solution:
```sql
INSERT INTO inventory (sku, product_name, stock, price)
VALUES ('SKU-7','Mouse Pad',40,6.00);
```

### Exercise 2: Price increase
- Difficulty: Beginner
- Task: Increase all items priced < $10 by 10%; ROUND to 2 decimals.
- Solution:
```sql
UPDATE inventory
SET price = ROUND(price * 1.10, 2)
WHERE price < 10.00;
```

### Exercise 3: Upsert daily stock
- Difficulty: Intermediate
- Task: Upsert SKU-2 with stock=20, price=84.00 and SKU-8 'Headphones' stock=25 price=45.00.
- Solution:
```sql
INSERT INTO inventory (sku, product_name, stock, price) VALUES
 ('SKU-2','Keyboard',20,84.00),
 ('SKU-8','Headphones',25,45.00)
ON DUPLICATE KEY UPDATE
  stock = VALUES(stock),
  price = VALUES(price);
```

### Exercise 4: Discontinue item (soft delete)
- Difficulty: Intermediate
- Setup:
```sql
ALTER TABLE inventory ADD COLUMN discontinued_at DATETIME NULL;
```
- Task: Soft delete SKU-4 by setting `discontinued_at = NOW()`.
- Solution:
```sql
UPDATE inventory
SET discontinued_at = NOW()
WHERE sku = 'SKU-4';
```

### Exercise 5: Copy low-stock items into a restock table
- Difficulty: Advanced
- Setup:
```sql
DROP TABLE IF EXISTS restock;
CREATE TABLE restock (
  sku VARCHAR(20) PRIMARY KEY,
  needed_qty INT NOT NULL
);
```
- Task: Insert SKUs with stock < 10 into `restock` with needed_qty = 20 - stock (min 0).
- Solution:
```sql
INSERT INTO restock (sku, needed_qty)
SELECT sku, GREATEST(20 - stock, 0) AS needed_qty
FROM inventory
WHERE stock < 10;
```

## 6) Self-assessment checklist
- Can you write safe INSERT, UPDATE, DELETE with WHERE? (Ex. 1–2)
- Can you upsert with ON DUPLICATE KEY UPDATE? (Ex. 3)
- Can you implement soft deletes and audit fields? (Ex. 4)
- Can you INSERT ... SELECT to transform data? (Ex. 5)

## 7) Connection to real work
- Scenarios: Nightly data loads, pricing updates, product lifecycle management.
- Roles: Data Engineer, DB Developer, Ops Engineer.
- Interview questions:
  - Q: Difference between REPLACE and ON DUPLICATE KEY UPDATE?
    - A: REPLACE deletes then inserts (can affect FKs/autoinc); ON DUPLICATE updates in place.
  - Q: How do you prevent accidental mass updates/deletes?
    - A: Always include WHERE; test with SELECT first; use safe-updates mode in clients.
  - Q: Why prefer soft deletes?
    - A: Preserves history and supports recovery/audit.
