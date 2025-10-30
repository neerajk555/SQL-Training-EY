# Module 5: Joins and Relationships (MySQL)

## 1) Introduction

Scenario: You have customers in one table and their orders in another. To answer “Which customer placed each order?” or “Which customers haven’t ordered yet?”, you’ll JOIN tables—one of the most important SQL skills.

Definition (plain):
- JOIN: Combine rows from two (or more) tables based on related columns.
- Primary key (PK): Uniquely identifies a row in the parent table.
- Foreign key (FK): References a parent PK from a child table to enforce relationships.

Connection to previous/next:
- Builds on SELECT and aggregations. You’ll join tables before grouping, filtering, and windowing. Later modules (CTEs, window functions) often follow joins.

Learning objectives:
- Write INNER JOIN, LEFT JOIN, RIGHT JOIN, and CROSS JOIN.
- Emulate FULL OUTER JOIN in MySQL (via UNION of LEFT/RIGHT anti-joins).
- Join across multiple tables and avoid accidental row multiplication.
- Understand primary/foreign keys and relationship cardinality.

## 2) Concept explanation

Common joins:
- INNER JOIN: Keep matching rows from both tables.
- LEFT JOIN: Keep all left rows; fill with NULLs when no right match.
- RIGHT JOIN: Keep all right rows; fill with NULLs for missing left.
- FULL OUTER JOIN: Keep all rows from both sides; MySQL lacks native support (emulate).
- CROSS JOIN: Cartesian product (every left row with every right row).
- SELF JOIN: Join a table to itself (e.g., employees and managers).

Syntax patterns:
```sql
-- Inner join
SELECT a.*, b.*
FROM a
INNER JOIN b ON a.key = b.key;

-- Left join
SELECT a.*, b.*
FROM a
LEFT JOIN b ON a.key = b.key;

-- MySQL full outer join emulation (example)
SELECT a.*, b.*
FROM a LEFT JOIN b ON a.key = b.key
UNION
SELECT a.*, b.*
FROM a RIGHT JOIN b ON a.key = b.key;
```

Common misconceptions:
- “LEFT JOIN filters missing matches.” No—LEFT JOIN preserves left rows even without matches (right columns become NULL). Add WHERE conditions carefully.
- “Join conditions belong in WHERE.” You can put them in WHERE, but using ON is clearer and avoids accidental filtering of unmatched rows with LEFT JOIN.
- “Duplicates mean a bug.” Sometimes 1-to-many relations legitimately create multiple joined rows; aggregate if you need 1 row per parent.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS customers;
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL
);

DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL
);

INSERT INTO customers VALUES
  (1,'Sara Jones'),
  (2,'Leo Park'),
  (3,'Mina Lee'),
  (4,'Kai Tan');

INSERT INTO orders VALUES
  (5001,1,'2025-01-05',120.00),
  (5002,1,'2025-01-06',55.50),
  (5003,2,'2025-01-06',250.00),
  (5004,2,'2025-01-10',10.00);
```

### Example A — INNER JOIN: orders with customer names
- Business question: “Show each order with the customer’s name.”

SQL:
```sql
SELECT 
  o.order_id,
  c.customer_name,
  o.order_date,
  o.total_amount
FROM orders AS o
INNER JOIN customers AS c
  ON o.customer_id = c.customer_id
ORDER BY o.order_id;
```

Expected output:
| order_id | customer_name | order_date | total_amount |
|---------:|---------------|------------|-------------:|
| 5001     | Sara Jones    | 2025-01-05 | 120.00       |
| 5002     | Sara Jones    | 2025-01-06 | 55.50        |
| 5003     | Leo Park      | 2025-01-06 | 250.00       |
| 5004     | Leo Park      | 2025-01-10 | 10.00        |

Processing:
1) FROM orders → base.
2) INNER JOIN customers ON matching customer_id.
3) SELECT and ORDER BY.

Why/alternatives:
- INNER JOIN focuses on matches. If you want customers without orders, use LEFT JOIN.

### Example B — LEFT JOIN: customers without orders
- Business question: “List all customers and their last order date (if any).”

SQL:
```sql
SELECT 
  c.customer_id,
  c.customer_name,
  MAX(o.order_date) AS last_order_date
FROM customers AS c
LEFT JOIN orders AS o
  ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY last_order_date DESC IS NULL, last_order_date DESC;
```

Expected output:
| customer_id | customer_name | last_order_date |
|------------:|---------------|-----------------|
| 1           | Sara Jones    | 2025-01-06      |
| 2           | Leo Park      | 2025-01-10      |
| 3           | Mina Lee      | NULL            |
| 4           | Kai Tan       | NULL            |

Processing:
1) LEFT JOIN preserves all customers.
2) Group and MAX to get last order per customer.
3) ORDER BY puts NULLs last using an expression.

Why/alternatives:
- Use WHERE carefully with LEFT JOIN; `WHERE o.order_id IS NULL` finds customers with no orders.

### Example C — FULL OUTER JOIN emulation
- Business question: “Show all customers and all orders even if there is no match.”

SQL (emulate FULL OUTER JOIN in MySQL):
```sql
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
UNION
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.total_amount
FROM customers c
RIGHT JOIN orders o ON o.customer_id = c.customer_id;
```

Expected output:
- Contains inner matches, plus any customers without orders, plus any orders without customers.

Why/alternatives:
- Use UNION ALL + filters when you need to retain duplicates; here UNION removes duplicates from overlapping INNER results.

Portability notes:
- PostgreSQL/SQL Server support FULL OUTER JOIN natively.

## 4) Common patterns and best practices

Use cases:
1) Enriching facts (orders) with dimensions (customers/products).
2) Finding unmatched rows (data quality checks).
3) Multi-table reports (orders + items + products + categories).
4) Self-joins (employees → managers; categories → parents).
5) Many-to-many bridges (orders ↔ order_items ↔ products).

Templates:
```sql
-- Unmatched rows: customers with no orders
SELECT c.*
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.customer_id IS NULL;

-- Multi-join chain
SELECT o.order_id, c.customer_name, p.product_name, oi.quantity
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p ON p.product_id = oi.product_id
JOIN customers c ON c.customer_id = o.customer_id;

-- Self join (manager relationship)
SELECT e.employee_id, e.name, m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON m.employee_id = e.manager_id;
```

Performance notes:
- Ensure join keys are indexed (PK/FK). Missing indexes cause costly scans.
- Beware Cartesian products from missing or incorrect ON conditions.
- Reduce rows early with WHERE on the base (leftmost) table when possible.

Do’s and Don’ts:
- Do: Use explicit JOIN ... ON, not comma joins.
- Do: Qualify columns with table aliases to avoid ambiguity.
- Don’t: Put join conditions in WHERE if doing LEFT JOIN and you need to preserve null-extended rows (use ON).
- Don’t: Assume FULL OUTER JOIN exists in MySQL—emulate if necessary.

Troubleshooting:
- “Column ambiguously defined” → Qualify with table alias.
- “Too many rows after join” → Check 1-to-many relationships and duplicate keys.
- “Missing rows with LEFT JOIN” → A WHERE condition referencing the right table may be filtering null-extended rows; move to ON or adjust predicate.

## 5) Practice exercises

### Exercise 1: Order list with customer names
- Difficulty: Beginner
- Sample data: Use `customers`, `orders` from setup.
- Task: Show `order_id, customer_name, order_date, total_amount` using INNER JOIN.
- Solution:
```sql
SELECT o.order_id, c.customer_name, o.order_date, o.total_amount
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```

### Exercise 2: Customers with no orders
- Difficulty: Beginner
- Task: List customers who have never placed an order.
- Expected output: `Mina Lee`, `Kai Tan`.
- Solution:
```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.customer_id IS NULL;
```

### Exercise 3: Last order date per customer
- Difficulty: Intermediate
- Task: Show `customer_id, customer_name, last_order_date` using LEFT JOIN + MAX.
- Solution:
```sql
SELECT c.customer_id, c.customer_name, MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY last_order_date DESC IS NULL, last_order_date DESC;
```

### Exercise 4: Orders with missing customers
- Difficulty: Intermediate
- Context: Data quality. Assume an order might reference a non-existent customer.
- Task: Return orders where the customer is missing.
- Solution:
```sql
SELECT o.*
FROM orders o
LEFT JOIN customers c ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL;
```

### Exercise 5: Full outer join emulation
- Difficulty: Advanced
- Task: Return all customers and all orders, whether matched or not (emulate FULL OUTER JOIN).
- Solution:
```sql
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
UNION
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.total_amount
FROM customers c
RIGHT JOIN orders o ON o.customer_id = c.customer_id;
```
- Extension: Add a column `match_flag` to indicate 'MATCHED' vs 'UNMATCHED' using UNION ALL and predicates.

## 6) Self-assessment checklist
- Can you write INNER/LEFT/RIGHT joins? (Ex. 1–3)
- Can you find unmatched rows with LEFT JOIN + WHERE ... IS NULL? (Ex. 2/4)
- Can you emulate FULL OUTER JOIN in MySQL? (Ex. 5)
- Can you join multiple tables safely and avoid duplicates?
- Can you explain PK/FK relationships and why they matter?

## 7) Connection to real work
- Scenarios: Customer order enrichment, product catalog joins, data completeness checks.
- Roles: Analyst, Data Engineer, BI Developer, Support Engineer.
- Interview questions:
  - Q: When would a LEFT JOIN return fewer rows than the left table?
    - A: When a subsequent WHERE condition filters out NULL-extended rows. Keep such predicates in ON or adjust your logic.
  - Q: How do you handle many-to-many joins safely?
    - A: Join through the bridge table and aggregate if you need 1 row per parent; validate expected duplication.
  - Q: Why are indexes important for joins?
    - A: They allow fast lookups on join keys, reducing scans and improving performance.
