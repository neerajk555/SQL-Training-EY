# Module 7: Set Operations and Advanced Filtering (MySQL)

## 1) Introduction

Scenario: You need “all users who bought from Category A OR B,” “products that sold in both Q1 and Q2,” and “items that sold in Q1 but NOT Q2.” Set operations and advanced filtering let you express these comparisons clearly.

Definition (plain):
- Set operation: Combine results of multiple queries (UNION, UNION ALL). 
- Intersections/differences: In MySQL, emulate INTERSECT/EXCEPT using joins/EXISTS (MySQL doesn’t natively support INTERSECT/EXCEPT).
- Advanced filtering: CASE expressions, complex boolean logic, NOT EXISTS anti-joins.

Connection to previous/next:
- Builds on SELECT/WHERE/JOINs and subqueries. Prepares you for window functions in Module 8.

Learning objectives:
- Use UNION vs UNION ALL appropriately.
- Emulate INTERSECT and EXCEPT using JOINs/EXISTS.
- Write robust boolean logic with parentheses.
- Use CASE for conditional transformations.

## 2) Concept explanation

UNION vs UNION ALL:
- UNION removes duplicates (extra work).
- UNION ALL keeps duplicates (faster, preserves counts).

Emulating INTERSECT (rows in both queries):
```sql
SELECT a.key
FROM ( ...queryA... ) a
INNER JOIN ( ...queryB... ) b ON a.key = b.key;
```

Emulating EXCEPT (rows in A but not in B):
```sql
SELECT a.key
FROM ( ...queryA... ) a
LEFT JOIN ( ...queryB... ) b ON a.key = b.key
WHERE b.key IS NULL;
```

CASE expression:
```sql
CASE 
  WHEN condition THEN value
  [WHEN condition THEN value]
  [ELSE value]
END AS alias
```

Common misconceptions:
- “UNION always preserves order.” No—use ORDER BY after the full UNION block.
- “NOT IN is same as NOT EXISTS.” NOT IN fails with NULLs; NOT EXISTS is safer.
- “CASE can appear anywhere.” It can be used in SELECT, WHERE, ORDER BY, etc., but must follow expression rules.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS sales_q1;
CREATE TABLE sales_q1 (
  product_id INT,
  qty INT
);
DROP TABLE IF EXISTS sales_q2;
CREATE TABLE sales_q2 (
  product_id INT,
  qty INT
);

INSERT INTO sales_q1 VALUES (1,10),(2,5),(3,8),(4,0);
INSERT INTO sales_q2 VALUES (2,6),(3,7),(4,2),(5,9);
```

### Example A — UNION vs UNION ALL
- Business question: “List all product_ids that sold in Q1 or Q2; preserve duplicates for volume analysis.”

SQL:
```sql
-- Unique set across quarters
SELECT product_id FROM sales_q1
UNION
SELECT product_id FROM sales_q2;

-- Preserve duplicates (may show product_id twice if it appears in both)
SELECT product_id FROM sales_q1
UNION ALL
SELECT product_id FROM sales_q2;
```

Expected output:
- UNION: 1,2,3,4,5
- UNION ALL: 1,2,3,4,2,3,4,5 (order unspecified without ORDER BY)

### Example B — INTERSECT emulation: sold in both Q1 and Q2
- Business question: “Which product_ids sold in both quarters?”

SQL:
```sql
SELECT q1.product_id
FROM (SELECT DISTINCT product_id FROM sales_q1) AS q1
JOIN (SELECT DISTINCT product_id FROM sales_q2) AS q2
  ON q1.product_id = q2.product_id
ORDER BY q1.product_id;
```

Expected output: 2, 3, 4

Why/alternatives:
- EXISTS-based pattern:
```sql
SELECT DISTINCT s1.product_id
FROM sales_q1 s1
WHERE EXISTS (
  SELECT 1 FROM sales_q2 s2
  WHERE s2.product_id = s1.product_id
);
```

### Example C — EXCEPT emulation: sold in Q1 but NOT Q2
- Business question: “Which product_ids sold in Q1 but not Q2?”

SQL:
```sql
SELECT q1.product_id
FROM (SELECT DISTINCT product_id FROM sales_q1) q1
LEFT JOIN (SELECT DISTINCT product_id FROM sales_q2) q2
  ON q2.product_id = q1.product_id
WHERE q2.product_id IS NULL
ORDER BY q1.product_id;
```

Expected output: 1

Advanced filtering with CASE (bonus):
```sql
SELECT product_id,
       qty,
       CASE 
         WHEN qty = 0 THEN 'none'
         WHEN qty < 5 THEN 'low'
         WHEN qty < 10 THEN 'medium'
         ELSE 'high'
       END AS qty_band
FROM sales_q2
ORDER BY qty DESC;
```

## 4) Common patterns and best practices

Use cases:
1) Combine similar datasets across periods/sources (UNION/ALL).
2) Intersection/difference analysis between cohorts.
3) Conditional bucketing using CASE (bands, tiers).
4) Anti-joins for “in A but not in B.”

Templates:
```sql
-- Combine two tables with the same schema
SELECT col_list FROM t1
UNION ALL
SELECT col_list FROM t2;

-- Anti-join (A minus B)
SELECT a.*
FROM A a
LEFT JOIN B b ON b.key = a.key
WHERE b.key IS NULL;
```

Performance notes:
- DISTINCT or UNION (without ALL) requires deduplication work.
- Use EXISTS for large anti-joins instead of NOT IN, especially with NULLs.
- Index join keys in INTERSECT/EXCEPT emulations.

Do’s and Don’ts:
- Do: Add ORDER BY only once, after the final UNION block.
- Do: Align column counts and types across UNION branches.
- Don’t: Use NOT IN with NULL-producing subqueries.
- Don’t: Forget parentheses when mixing AND/OR.

Troubleshooting:
- “Result set columns don’t match” → Same number and compatible types on both sides of UNION.
- “Unexpected empty result with NOT IN” → Check NULLs in the subquery; switch to NOT EXISTS.

## 5) Practice exercises

### Exercise 1: All products across quarters
- Difficulty: Beginner
- Task: Return unique product_ids that sold in either quarter.
- Solution:
```sql
SELECT product_id FROM sales_q1
UNION
SELECT product_id FROM sales_q2
ORDER BY product_id;
```

### Exercise 2: Intersection
- Difficulty: Beginner
- Task: Return product_ids that sold in both quarters (INTERSECT emulation).
- Solution:
```sql
SELECT q1.product_id
FROM (SELECT DISTINCT product_id FROM sales_q1) q1
JOIN (SELECT DISTINCT product_id FROM sales_q2) q2
  ON q2.product_id = q1.product_id
ORDER BY q1.product_id;
```

### Exercise 3: Difference
- Difficulty: Intermediate
- Task: Return product_ids that sold in Q1 but not Q2 (EXCEPT emulation).
- Solution:
```sql
SELECT q1.product_id
FROM (SELECT DISTINCT product_id FROM sales_q1) q1
LEFT JOIN (SELECT DISTINCT product_id FROM sales_q2) q2
  ON q2.product_id = q1.product_id
WHERE q2.product_id IS NULL
ORDER BY q1.product_id;
```

### Exercise 4: Quantity bands
- Difficulty: Intermediate
- Task: On `sales_q2`, return product_id, qty, and a CASE-based band: none/low/medium/high.
- Solution:
```sql
SELECT product_id, qty,
       CASE WHEN qty = 0 THEN 'none'
            WHEN qty < 5 THEN 'low'
            WHEN qty < 10 THEN 'medium'
            ELSE 'high'
       END AS qty_band
FROM sales_q2
ORDER BY qty DESC;
```

### Exercise 5: Advanced anti-join with EXISTS
- Difficulty: Advanced
- Task: Rewrite Exercise 3 using NOT EXISTS.
- Solution:
```sql
SELECT DISTINCT s1.product_id
FROM sales_q1 s1
WHERE NOT EXISTS (
  SELECT 1 FROM sales_q2 s2
  WHERE s2.product_id = s1.product_id
)
ORDER BY s1.product_id;
```

## 6) Self-assessment checklist
- Can you choose UNION vs UNION ALL? (Ex. 1)
- Can you emulate INTERSECT and EXCEPT in MySQL? (Ex. 2/3/5)
- Can you write clear CASE expressions? (Ex. 4)
- Can you avoid NOT IN + NULL pitfalls?

## 7) Connection to real work
- Scenarios: Cross-period comparisons, user cohort overlaps, campaign inclusion/exclusion rules.
- Roles: Analyst, Marketing Ops, Product, Data Engineer.
- Interview questions:
  - Q: When do you prefer UNION vs UNION ALL?
    - A: Use UNION ALL to keep duplicates (faster); UNION to remove duplicates when necessary.
  - Q: How do you get INTERSECT-like results in MySQL?
    - A: Join on the key or use EXISTS between the two result sets.
  - Q: Why is NOT EXISTS safer than NOT IN with subqueries?
    - A: NOT IN returns empty if the subquery yields NULL; NOT EXISTS doesn’t have this issue.
