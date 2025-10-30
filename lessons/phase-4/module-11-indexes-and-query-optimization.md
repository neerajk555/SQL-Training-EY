# Module 11: Indexes and Query Optimization (MySQL)

## 1) Introduction

Scenario: Queries slow down as data grows. You need to find the right rows fast and avoid full scans. Indexes and basic optimization techniques make queries scale.

Definition (plain):
- Index: A data structure (often B-Tree) that speeds up lookups and sorting by specific columns.
- Query plan: The strategy the database uses to execute a query.
- EXPLAIN: A tool to see the plan and diagnose performance.

Connection to previous/next:
- Builds on DDL and joins. Next module covers transactions.

Learning objectives:
- Choose columns to index based on access patterns.
- Use composite indexes effectively (column order matters).
- Read EXPLAIN output to spot full scans (type=ALL) and missing indexes.
- Avoid common anti-patterns that defeat index usage.

## 2) Concept explanation

Index types (common):
- B-Tree (default): Good for equality and range queries.
- Hash (Memory engine): Equality only.
- Full-text: Text search; separate feature with MATCH() AGAINST().

When to index:
- Columns used in WHERE, JOIN ... ON, ORDER BY, GROUP BY.
- High selectivity (many distinct values) is most beneficial.
- Composite indexes should match common predicates with leftmost prefix.

Composite index rule (leftmost prefix):
- Index (a, b, c) can support filters on (a), (a,b), (a,b,c) efficiently, but not (b) alone.

EXPLAIN basics (columns vary by version):
- type: ALL (full scan), index, range, ref, eq_ref, const, system (best → const/system)
- key: index chosen; key_len: bytes used
- rows: estimated rows examined
- Extra: Using where; Using index; Using filesort; Using temporary

Common misconceptions:
- “Index everything.” Indexes speed reads but slow writes and use space.
- “ORDER BY always uses index.” Only if the ORDER BY matches index order and direction with compatible WHERE.
- “LIKE '%term%' uses index.” Leading wildcard prevents B-Tree use.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS users;
CREATE TABLE users (
  user_id INT PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  country_code CHAR(2) NOT NULL,
  created_at DATETIME NOT NULL,
  INDEX idx_users_email (email),
  INDEX idx_users_country_created (country_code, created_at)
);

INSERT INTO users
SELECT 
  seq AS user_id,
  CONCAT('user', seq, '@example.com') AS email,
  CASE WHEN seq % 3 = 0 THEN 'US' WHEN seq % 3 = 1 THEN 'IN' ELSE 'GB' END AS country_code,
  DATE_ADD('2025-01-01', INTERVAL seq MINUTE) AS created_at
FROM (
  SELECT 1 AS seq UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
) s -- small demo; imagine thousands
;
```

### Example A — WHERE + ORDER BY using composite index
- Business question: “Get newest US users.”

SQL:
```sql
EXPLAIN
SELECT user_id, email
FROM users
WHERE country_code = 'US'
ORDER BY created_at DESC
LIMIT 10;
```

Expect the index `idx_users_country_created` to be considered. Matching leftmost `country_code` and ordering by `created_at` enables index range scan.

### Example B — LIKE pattern and index
- Business question: “Find emails ending with '@example.com'.”

SQL:
```sql
EXPLAIN
SELECT user_id
FROM users
WHERE email LIKE '%@example.com';
```

Leading wildcard prevents typical B-Tree usage; expect a scan. Alternative: store domain in a separate column and index it.

### Example C — Avoid functions on indexed columns
- Business question: “Find users created on 2025-01-01.”

SQL (anti-pattern):
```sql
EXPLAIN
SELECT *
FROM users
WHERE DATE(created_at) = '2025-01-01';
```

The function on `created_at` may prevent index usage. Prefer a range:
```sql
EXPLAIN
SELECT *
FROM users
WHERE created_at >= '2025-01-01' AND created_at < '2025-01-02';
```

## 4) Common patterns and best practices

Use cases:
1) Speed up lookups by exact key or ranges.
2) Support ORDER BY/GROUP BY without extra sorts.
3) Optimize joins by indexing join keys on both sides.

Templates:
```sql
-- Create index
CREATE INDEX idx_table_col ON table_name (col);

-- Composite index for common filter + sort
CREATE INDEX idx_country_created ON users (country_code, created_at);

-- Covering index (select only indexed columns)
CREATE INDEX idx_email_only ON users (email);
SELECT email FROM users WHERE email > 'm' ORDER BY email; -- can be index-only
```

Performance notes:
- Validate with EXPLAIN; look for `type` better than ALL and reasonable `rows`.
- Keep indexes small; avoid indexing very wide text columns unless necessary.
- Update/delete performance degrades with too many indexes.

Do’s and Don’ts:
- Do: Index foreign keys and frequent WHERE columns.
- Do: Align composite index with the query’s predicates (leftmost prefix).
- Don’t: Wrap indexed columns in functions in WHERE.
- Don’t: Expect indexes to help with leading-wildcard LIKE.

Troubleshooting:
- “Using filesort” in Extra → ORDER BY executed via sort; consider index to cover order.
- “Using temporary” → GROUP BY/ORDER BY may need temp tables; tune indexes or query.
- EXPLAIN plan still scans many rows → Revisit predicates and index design.

## 5) Practice exercises

### Exercise 1: Add an index for common query
- Difficulty: Beginner
- Task: If you often filter users by country_code and order by created_at, create a composite index in that order.
- Solution:
```sql
CREATE INDEX idx_users_country_created ON users (country_code, created_at);
```

### Exercise 2: Range query by created_at
- Difficulty: Beginner
- Task: Write a range query for a single day without wrapping created_at with DATE().
- Solution:
```sql
SELECT *
FROM users
WHERE created_at >= '2025-01-01' AND created_at < '2025-01-02';
```

### Exercise 3: Optimize domain lookup
- Difficulty: Intermediate
- Task: Add a `email_domain` generated column and index it; query by domain.
- Solution:
```sql
ALTER TABLE users ADD COLUMN email_domain VARCHAR(255)
  AS (SUBSTRING_INDEX(email, '@', -1)) STORED;
CREATE INDEX idx_users_email_domain ON users (email_domain);

SELECT * FROM users WHERE email_domain = 'example.com';
```

### Exercise 4: Covering index query
- Difficulty: Intermediate
- Task: Create an index on (email) and query only email with ORDER BY email.
- Solution: See template; EXPLAIN should show Using index.

### Exercise 5: Join optimization
- Difficulty: Advanced
- Setup: Add table `user_logs(user_id, created_at)`; index (user_id, created_at).
- Task: Query logs for US users last day efficiently via join on indexed keys.
- Sketch:
```sql
SELECT ul.*
FROM user_logs ul
JOIN users u ON u.user_id = ul.user_id
WHERE u.country_code = 'US'
  AND ul.created_at >= NOW() - INTERVAL 1 DAY;
```

## 6) Self-assessment checklist
- Can you choose columns to index based on queries? (Ex. 1/5)
- Can you read EXPLAIN to spot scans and missing indexes? (Examples)
- Can you use composite indexes effectively? (Ex. 1)
- Can you avoid function-wrapped predicates and leading wildcards?

## 7) Connection to real work
- Scenarios: Speeding up dashboards, reducing timeouts, scaling APIs.
- Roles: Data/DB Engineer, SRE, Backend Engineer.
- Interview questions:
  - Q: How does column order in a composite index matter?
    - A: It supports leftmost prefixes; predicates must start with the leftmost column(s) to use the index fully.
  - Q: Why can `LIKE '%term%'` be slow?
    - A: Leading wildcard disables index range scans; consider full-text or other approaches.
  - Q: What do `Using filesort` and `Using temporary` mean in EXPLAIN?
    - A: Extra work for sorting and temp tables; consider better indexes or query rewrites.
