# Module 6: Subqueries and CTEs (MySQL 8+)

## 1) Introduction

Scenario: You need “customers whose spending is above the overall average,” or “products with sales above their category’s average.” Subqueries and CTEs help express complex logic clearly and step by step.

Definition (plain):
- Subquery: A query nested inside another query (in SELECT, FROM, WHERE, or HAVING).
- CTE (Common Table Expression): A named, temporary result set defined with `WITH` used within a single statement.
- Recursive CTE: A CTE that references itself to traverse hierarchies.

Connection to previous/next:
- Builds on SELECT, GROUP BY, and JOINs. Later modules use CTEs with window functions and advanced logic.

Learning objectives:
- Write scalar, IN, EXISTS, and FROM-subqueries.
- Use CTEs to structure multi-step logic.
- Understand when CTEs improve readability and maintainability.
- Write a simple recursive CTE for hierarchies.

## 2) Concept explanation

Subquery types:
- Scalar subquery: Returns a single value.
- IN/NOT IN subquery: Returns a set of values for membership tests.
- EXISTS/NOT EXISTS subquery: Tests existence efficiently.
- FROM-subquery (derived table): Treat a subquery as a table to join or filter.

CTE syntax:
```sql
WITH cte_name AS (
  SELECT ...
)
SELECT ...
FROM cte_name;
```

Recursive CTE pattern:
```sql
WITH RECURSIVE r AS (
  -- anchor
  SELECT id, parent_id, 0 AS depth FROM nodes WHERE id = 1
  UNION ALL
  -- recursive step
  SELECT n.id, n.parent_id, r.depth + 1
  FROM nodes n
  JOIN r ON n.parent_id = r.id
)
SELECT * FROM r;
```

Common misconceptions:
- “Subqueries always run first.” SQL is declarative; the optimizer chooses an efficient plan.
- “CTEs are faster than subqueries.” Not inherently—CTEs are for readability; performance depends on the query and engine.
- “IN and EXISTS are identical.” EXISTS can be more efficient for correlated checks; IN materializes a set of values.

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
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL
);

INSERT INTO customers VALUES
 (1,'Sara Jones'),(2,'Leo Park'),(3,'Mina Lee'),(4,'Kai Tan');

INSERT INTO orders VALUES
 (6001,1,'2025-01-05',120.00),
 (6002,1,'2025-01-06',55.50),
 (6003,2,'2025-01-06',250.00),
 (6004,2,'2025-01-10',10.00);
```

### Example A — Scalar subquery: compare to overall average
- Business question: “Show orders above the overall average order amount.”

SQL:
```sql
SELECT order_id, customer_id, total_amount
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders)
ORDER BY total_amount DESC;
```

Expected output: Orders greater than the average (based on sample data, likely 6003 and 6001).

Why/alternatives:
- You could compute AVG first, then filter. The scalar subquery in WHERE keeps it in one statement.

### Example B — IN vs EXISTS: customers with orders
- Business question: “List customers who have placed orders.”

Using IN:
```sql
SELECT *
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
```

Using EXISTS (correlated):
```sql
SELECT c.*
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.customer_id = c.customer_id
);
```

Why/alternatives:
- EXISTS often performs well for existence checks; IN is concise but may need DISTINCT.

### Example C — CTE for multi-step logic
- Business question: “Show customers whose total spend is above the overall average spend per customer.”

SQL (CTEs):
```sql
WITH customer_totals AS (
  SELECT customer_id, SUM(total_amount) AS total_spent
  FROM orders
  GROUP BY customer_id
), overall AS (
  SELECT AVG(total_spent) AS avg_spend
  FROM customer_totals
)
SELECT c.customer_id, ct.total_spent
FROM customer_totals ct
JOIN overall o ON 1=1
JOIN customers c ON c.customer_id = ct.customer_id
WHERE ct.total_spent > o.avg_spend
ORDER BY ct.total_spent DESC;
```

Expected output: Customers above average total spend (likely customer 2).

Why/alternatives:
- CTEs improve readability vs nesting multiple levels of subqueries. You could also use a derived table (FROM-subquery) structure.

Bonus — Recursive CTE (simple hierarchy):
```sql
-- Example structure
DROP TABLE IF EXISTS org;
CREATE TABLE org (
  employee_id INT PRIMARY KEY,
  manager_id INT NULL,
  name VARCHAR(100) NOT NULL
);
INSERT INTO org VALUES
 (1,NULL,'CEO'),
 (2,1,'VP Eng'),(3,1,'VP Sales'),
 (4,2,'Eng Manager'),(5,4,'Engineer');

WITH RECURSIVE tree AS (
  SELECT employee_id, manager_id, name, 0 AS depth
  FROM org
  WHERE employee_id = 1 -- root
  UNION ALL
  SELECT o.employee_id, o.manager_id, o.name, t.depth + 1
  FROM org o
  JOIN tree t ON o.manager_id = t.employee_id
)
SELECT * FROM tree ORDER BY depth, employee_id;
```

## 4) Common patterns and best practices

Use cases:
1) Thresholds based on averages or totals (scalar subqueries).
2) Existence checks (EXISTS) to filter parents with/without children.
3) Multi-step transformations (CTEs) for clarity and reuse.
4) Hierarchical expansions (recursive CTEs) for org charts, categories.

Templates:
```sql
-- Parent rows with at least one child
SELECT p.*
FROM parents p
WHERE EXISTS (
  SELECT 1 FROM children c WHERE c.parent_id = p.id
);

-- Multi-step CTE pipeline
WITH step1 AS (
  SELECT ...
), step2 AS (
  SELECT ... FROM step1
)
SELECT ... FROM step2;
```

Performance notes:
- Correlated subqueries run per row; EXISTS is efficient for boolean existence.
- CTEs don’t guarantee materialization in MySQL 8; optimizer may inline—performance varies.
- Index join/filter columns used inside subqueries.

Do’s and Don’ts:
- Do: Prefer EXISTS over IN for large lists or correlated presence checks.
- Do: Use CTEs to clarify intent and break problems into steps.
- Don’t: Over-nest subqueries when a JOIN or CTE is clearer.
- Don’t: Forget that NOT IN behaves unexpectedly with NULLs (use NOT EXISTS or handle NULLs explicitly).

Troubleshooting:
- “Operand should contain 1 column(s)” → Scalar subquery must return exactly one column.
- “Subquery returns more than 1 row” in a scalar context → Use an aggregate or LIMIT 1.
- NOT IN with NULL returns no rows → Use NOT EXISTS or filter NULLs in the subquery.

## 5) Practice exercises

### Exercise 1: Orders above average amount
- Difficulty: Beginner
- Task: Return orders with `total_amount` > overall AVG.
- Solution:
```sql
SELECT *
FROM orders
WHERE total_amount > (SELECT AVG(total_amount) FROM orders);
```

### Exercise 2: Customers without orders (EXISTS)
- Difficulty: Intermediate
- Task: Return customers who have NOT placed any orders using NOT EXISTS.
- Solution:
```sql
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o
  WHERE o.customer_id = c.customer_id
);
```

### Exercise 3: Top spender via CTE
- Difficulty: Intermediate
- Task: Using a CTE, compute total spend per customer and return the top 1.
- Solution:
```sql
WITH totals AS (
  SELECT customer_id, SUM(total_amount) AS total_spent
  FROM orders
  GROUP BY customer_id
)
SELECT *
FROM totals
ORDER BY total_spent DESC
LIMIT 1;
```

### Exercise 4: Category average comparison
- Difficulty: Advanced
- Sample data (extend order_items from Module 4).
- Task: For each category, list product_id with total revenue greater than that category’s average revenue per product (use CTEs or correlated subqueries).
- Sketch solution (pattern):
```sql
WITH per_product AS (
  SELECT product_id, category, SUM(quantity*unit_price) AS revenue
  FROM order_items
  GROUP BY product_id, category
), per_category AS (
  SELECT category, AVG(revenue) AS cat_avg
  FROM per_product
  GROUP BY category
)
SELECT p.*
FROM per_product p
JOIN per_category c ON c.category = p.category
WHERE p.revenue > c.cat_avg;
```

### Exercise 5: Recursive CTE — org depth
- Difficulty: Advanced
- Task: Using the `org` table from the example, return employee_id, name, and depth from CEO down.
- Solution: See the recursive CTE in Example C; modify the root as needed.

## 6) Self-assessment checklist
- Can you write scalar subqueries in WHERE? (Ex. 1)
- Can you choose between IN and EXISTS? (Ex. 2)
- Can you structure multi-step logic with CTEs? (Ex. 3/4)
- Can you write a simple recursive CTE? (Ex. 5)
- Can you avoid NOT IN + NULL pitfalls?

## 7) Connection to real work
- Scenarios: Peer benchmarks (above/below average), filtering parents by child criteria, hierarchical roll-ups.
- Roles: Analyst, BI Developer, Data Engineer.
- Interview questions:
  - Q: When is EXISTS preferred over IN?
    - A: When checking existence with correlated conditions, especially on large datasets.
  - Q: Do CTEs improve performance?
    - A: Not inherently—they improve readability; performance depends on the optimizer and indexes.
  - Q: How do you prevent NOT IN from returning zero rows due to NULLs?
    - A: Use NOT EXISTS or exclude NULLs in the subquery.
