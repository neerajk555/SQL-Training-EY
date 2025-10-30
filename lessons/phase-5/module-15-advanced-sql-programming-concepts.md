# Module 15: Advanced SQL Programming Concepts (MySQL)

## 1) Introduction

Scenario: You’re building robust analytics and ops tooling. You need dynamic queries, JSON handling, scheduled jobs, and strategies for large tables.

Definition (plain):
- Dynamic SQL: Build and run SQL strings at runtime (PREPARE/EXECUTE).
- JSON functions: Parse and manipulate JSON documents in SQL.
- Partitioning: Split large tables into manageable partitions for performance/maintenance.
- Event scheduler: Time-based job execution inside MySQL.

Connection to previous/next:
- Builds on all prior topics to enable production-grade patterns. Next: Professional practices.

Learning objectives:
- Use PREPARE/EXECUTE for dynamic SQL safely.
- Work with JSON columns using MySQL JSON functions.
- Understand table partitioning basics.
- Schedule recurring tasks with the event scheduler.

## 2) Concept explanation

Dynamic SQL:
```sql
SET @sql = 'SELECT * FROM users WHERE country_code = ?'; -- illustrative
-- MySQL PREPARE doesn’t bind ? like app drivers; embed literal safely (beware injection)
SET @country := 'US';
SET @sql = CONCAT('SELECT * FROM users WHERE country_code = ''', @country, '''');
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

JSON functions:
- JSON_EXTRACT(json, path) → alias `->` in MySQL 8
- JSON_UNQUOTE(JSON_EXTRACT(...))
- JSON_OBJECT, JSON_ARRAY, JSON_SET, JSON_REPLACE, JSON_REMOVE

Partitioning (range example):
```sql
ALTER TABLE orders
PARTITION BY RANGE (YEAR(order_date)) (
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```

Event scheduler:
```sql
SET GLOBAL event_scheduler = ON;
CREATE EVENT ev_purge_old_logs
ON SCHEDULE EVERY 1 DAY
DO
  DELETE FROM logs WHERE created_at < NOW() - INTERVAL 30 DAY;
```

Common misconceptions:
- “Dynamic SQL is as safe as parameterized queries.” No—MySQL PREPARE doesn’t bind parameters like app layers; avoid user inputs or sanitize carefully.
- “Partitioning makes all queries faster.” It helps certain range queries and maintenance; it can hurt if used incorrectly.
- “JSON columns replace normalized design.” Use JSON judiciously; index needs generated columns.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS events, products;
CREATE TABLE events (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  payload JSON,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100),
  attrs JSON
);

INSERT INTO events VALUES
 (1,'user_signup', JSON_OBJECT('user_id', 10, 'ref', 'email'), NOW()),
 (2,'purchase',    JSON_OBJECT('user_id', 10, 'amount', 25.00), NOW());

INSERT INTO products VALUES
 (1,'Mouse', JSON_OBJECT('color','black','wireless',true)),
 (2,'Keyboard', JSON_OBJECT('layout','US','backlit',true));
```

### Example A — JSON extraction and filtering
- Business question: “Find purchases over $20 and extract amount.”

SQL:
```sql
SELECT id, name,
       JSON_EXTRACT(payload, '$.amount') AS amount
FROM events
WHERE name = 'purchase'
  AND JSON_EXTRACT(payload, '$.amount') > 20;
```

Better with generated column:
```sql
ALTER TABLE events ADD COLUMN amount DECIMAL(10,2)
  AS (JSON_UNQUOTE(JSON_EXTRACT(payload, '$.amount')))
  STORED;
CREATE INDEX idx_events_amount ON events (amount);

SELECT id, name, amount FROM events WHERE amount > 20;
```

### Example B — Dynamic SQL (caution)
- Business question: “Query products by attribute name/value provided at runtime.”

SQL:
```sql
SET @attr := 'color';
SET @val := 'black';
SET @sql = CONCAT(
  'SELECT product_id, product_name
   FROM products
   WHERE JSON_UNQUOTE(JSON_EXTRACT(attrs, ''$.', @attr, ''')) = ''', @val, '''');
PREPARE s FROM @sql; EXECUTE s; DEALLOCATE PREPARE s;
```

Safer approach: Predefine allowed attributes and use CASE or generated columns.

### Example C — Partitioning sketch and event scheduler
- Business question: “Purge old events daily and partition by year.”

SQL (sketch):
```sql
ALTER TABLE events
PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p2025 VALUES LESS THAN (2026),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);

SET GLOBAL event_scheduler = ON;
CREATE EVENT IF NOT EXISTS ev_purge_events
ON SCHEDULE EVERY 1 DAY
DO
  DELETE FROM events WHERE created_at < NOW() - INTERVAL 90 DAY;
```

## 4) Common patterns and best practices

Use cases:
1) JSON payload logging with selective indexing via generated columns.
2) Periodic maintenance with events (purges).
3) Dynamic reporting when columns vary (prefer app-layer params or generated columns).
4) Partition pruning for large, time-series tables.

Templates:
```sql
-- Generated column for JSON and index
ALTER TABLE t ADD COLUMN field VARCHAR(50) AS (JSON_UNQUOTE(JSON_EXTRACT(json_col, '$.field'))) STORED;
CREATE INDEX idx_t_field ON t (field);
```

Performance notes:
- JSON_EXTRACT on the fly is slow at scale; use generated columns + indexes.
- Partitioning helps maintenance (drop old partitions) and range queries, but complicates indexing.

Do’s and Don’ts:
- Do: Whitelist attributes for dynamic queries.
- Do: Use events carefully (or external schedulers) and monitor.
- Don’t: Build SQL strings from untrusted inputs.
- Don’t: Assume partitioning is a silver bullet.

Troubleshooting:
- “The used command is not allowed” for events → Enable event_scheduler.
- JSON path errors → Validate JSON and paths (`'$.key'`).

## 5) Practice exercises

### Exercise 1: JSON filter
- Difficulty: Beginner
- Task: Return events where payload has user_id = 10.
- Solution:
```sql
SELECT * FROM events
WHERE JSON_EXTRACT(payload, '$.user_id') = 10;
```

### Exercise 2: Generated column
- Difficulty: Beginner
- Task: Add `user_id` generated column to events and index it.
- Solution:
```sql
ALTER TABLE events ADD COLUMN user_id INT
  AS (JSON_UNQUOTE(JSON_EXTRACT(payload, '$.user_id')))
  STORED;
CREATE INDEX idx_events_user ON events (user_id);
```

### Exercise 3: Dynamic attribute lookup
- Difficulty: Intermediate
- Task: Use PREPARE/EXECUTE to filter products by a given JSON attribute.

### Exercise 4: Partitioning plan
- Difficulty: Intermediate
- Task: Propose a partitioning scheme for a high-volume logs table (by month/year) and how to prune old data.

### Exercise 5: Scheduled purge
- Difficulty: Advanced
- Task: Create a daily event to remove events older than 30 days.

## 6) Self-assessment checklist
- Can you extract JSON fields and filter on them? (Ex. 1/2)
- Can you use generated columns to index JSON data? (Ex. 2)
- Can you write dynamic SQL safely (or avoid it)? (Ex. 3)
- Can you explain when to use partitioning and events? (Ex. 4/5)

## 7) Connection to real work
- Scenarios: Storing semi-structured event data, retention policies, flexible product attributes.
- Roles: Data Engineer, Platform Engineer, Backend.
- Interview questions:
  - Q: Why prefer generated columns for JSON indexing?
    - A: Allows standard indexes and faster queries versus on-the-fly extraction.
  - Q: Downsides of dynamic SQL?
    - A: SQL injection risk, plan cache issues, hard to optimize.
  - Q: When does partitioning help?
    - A: Time-range queries and archival; drop partitions quickly to delete old data.
