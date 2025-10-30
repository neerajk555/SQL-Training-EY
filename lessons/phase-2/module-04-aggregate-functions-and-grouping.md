# Module 4: Aggregate Functions and Grouping (MySQL)

## 1) Introduction

Scenario: Leadership asks, “What was total revenue last week?”, “Average order value by category?”, and “Which categories had more than 100 units sold?” Aggregations turn raw rows into summaries that drive decisions.

Definition (plain):
- Aggregate function: Computes a single result from many rows (COUNT, SUM, AVG, MIN, MAX).
- GROUP BY: Collects rows into groups so you can aggregate per group.
- HAVING: Filters groups after aggregation (like WHERE for groups).

Connection to previous/next:
- Builds on SELECT/WHERE/ORDER BY. You’ll combine this with JOINs (Module 5), subqueries/CTEs (Module 6), and window functions (Module 8).

Learning objectives:
- Use COUNT/SUM/AVG/MIN/MAX correctly.
- Group by one or more columns.
- Filter groups with HAVING (vs WHERE).
- Understand NULL behavior in aggregates.
- Use ROLLUP for simple subtotals/grand totals in MySQL.

## 2) Concept explanation

Aggregate functions:
- COUNT(*) counts rows (includes NULLs). COUNT(col) counts non-NULL values only.
- SUM(col), AVG(col) ignore NULL values.
- MIN(col), MAX(col) ignore NULL and return min/max of non-NULLs.

Grouping:
```sql
SELECT group_col, AGG(expr) AS alias
FROM table
WHERE <row filters>
GROUP BY group_col
HAVING <group filters>
ORDER BY alias DESC;
```

Multiple group columns:
```sql
GROUP BY group_col1, group_col2
```

MySQL ROLLUP:
```sql
SELECT group_col, SUM(val) AS total
FROM table
GROUP BY group_col WITH ROLLUP;  -- adds grand total row (group_col = NULL)
```

Common misconceptions:
- “WHERE can filter aggregates.” No—WHERE filters rows before grouping; use HAVING to filter aggregate results.
- “COUNT(col) includes NULLs.” No—use COUNT(*) for all rows; COUNT(col) skips NULLs.
- “All selected columns must be aggregated.” In aggregated queries, any selected column must be in GROUP BY or be aggregated.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS order_items;
CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  category VARCHAR(50) NOT NULL,
  order_date DATE NOT NULL,
  quantity INT NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL
);

INSERT INTO order_items VALUES
  (3001, 1, 'Electronics', '2025-01-08', 2, 25.00),
  (3002, 2, 'Electronics', '2025-01-09', 1, 85.00),
  (3003, 3, 'Home',        '2025-01-10', 4, 19.99),
  (3004, 1, 'Electronics', '2025-01-11', 1, 25.00),
  (3005, 5, 'Stationery',  '2025-01-11', 10, 3.50),
  (3006, 5, 'Stationery',  '2025-01-12', 5, 3.50);
```

### Example A — Revenue per category
- Business question: “What is total revenue by category?”

SQL:
```sql
SELECT 
  category,
  SUM(quantity * unit_price) AS revenue
FROM order_items
GROUP BY category
ORDER BY revenue DESC;
```

Expected output:
| category    | revenue |
|-------------|--------:|
| Electronics | 160.00  |
| Stationery  | 52.50   |
| Home        | 79.96   |

Processing:
1) FROM order_items → all rows.
2) GROUP BY category → group rows per category.
3) SUM per group → revenue.
4) ORDER BY revenue.

Why/alternatives:
- Using an expression inside SUM is standard. Alternative: Create a computed column in a subquery and then SUM the alias.

### Example B — Average unit price and counts per day
- Business question: “Show average unit price and number of lines per day.”

SQL:
```sql
SELECT 
  order_date,
  AVG(unit_price) AS avg_price,
  COUNT(*) AS line_count
FROM order_items
GROUP BY order_date
ORDER BY order_date ASC;
```

Expected output (conceptual):
| order_date | avg_price | line_count |
|------------|----------:|----------:|
| 2025-01-08 | 25.00     | 1         |
| 2025-01-09 | 85.00     | 1         |
| 2025-01-10 | 19.99     | 1         |
| 2025-01-11 | 14.25     | 2         |
| 2025-01-12 | 3.50      | 1         |

Processing:
1) GROUP BY date.
2) AVG and COUNT per group.

Why/alternatives:
- COUNT(*) includes all rows; COUNT(unit_price) is same here since no NULLs.

### Example C — HAVING and ROLLUP
- Business question: “Which categories sold more than 5 units total? Also show a grand total.”

SQL:
```sql
SELECT 
  category,
  SUM(quantity) AS units
FROM order_items
GROUP BY category WITH ROLLUP
HAVING (category IS NOT NULL AND units > 5)
    OR category IS NULL  -- keep the grand total row
ORDER BY units DESC;
```

Expected output:
| category    | units |
|-------------|------:|
| Stationery  | 15    |
| Electronics | 3     |  ← filtered out (<=5) in actual result
| Home        | 4     |  ← filtered out (<=5) in actual result
| NULL        | 22    |  ← grand total

Actual expected result (after HAVING):
| category   | units |
|------------|------:|
| Stationery | 15    |
| NULL       | 22    |

Processing:
1) GROUP BY WITH ROLLUP → adds a NULL category row as grand total.
2) HAVING → filter groups to > 5, but keep the NULL (grand total) row.

Why/alternatives:
- You could run two queries (detail + total) and union them. ROLLUP is concise when supported.

Portability notes:
- PostgreSQL: Use GROUPING SETS/CUBE/ROLLUP syntax (supported). 
- SQL Server: GROUP BY ROLLUP is supported with slightly different semantics; check documentation.

## 4) Common patterns and best practices

Use cases:
1) Revenue, quantity, or counts by category/date/region.
2) Average order value (AOV) and conversion metrics.
3) Identifying outliers with MIN/MAX.
4) Threshold-based summaries using HAVING.
5) Executive summaries with ROLLUP.

Templates:
```sql
-- Top-N categories by revenue
SELECT category, SUM(amount) AS revenue
FROM sales
GROUP BY category
ORDER BY revenue DESC
LIMIT 5;

-- Filter groups by threshold
SELECT salesperson, COUNT(*) AS deals
FROM deals
WHERE close_date >= '2025-01-01'
GROUP BY salesperson
HAVING deals >= 10;

-- Daily metrics
SELECT order_date,
       COUNT(*) AS orders,
       SUM(total_amount) AS revenue,
       AVG(total_amount) AS avg_order
FROM orders
GROUP BY order_date;
```

Performance notes:
- Aggregations scan many rows; appropriate indexes on WHERE columns help.
- Avoid unnecessary DISTINCT inside aggregates unless needed.
- Precompute derived metrics in staging tables if the same heavy calculation repeats.

Do’s and Don’ts:
- Do: Understand COUNT(*) vs COUNT(col).
- Do: Put non-aggregate columns in GROUP BY.
- Don’t: Filter aggregates in WHERE; use HAVING.
- Don’t: Depend on ROLLUP row order; explicitly ORDER BY.

Troubleshooting:
- “In aggregated query without GROUP BY, nonaggregated columns are not allowed” → Put columns in GROUP BY or aggregate them.
- “Incorrect results due to NULLs” → Remember aggregates ignore NULL (except COUNT(*)). Use COALESCE if needed.

## 5) Practice exercises

### Exercise 1: Units and revenue by category
- Difficulty: Beginner
- Context: Category managers want totals.
- Sample data: Use `order_items` created above.
- Task: Return `category`, `SUM(quantity) AS units`, `SUM(quantity*unit_price) AS revenue`, sorted by revenue DESC.
- Expected output: Stationery highest units; revenue depends on prices.
- Solution:
```sql
SELECT category,
       SUM(quantity) AS units,
       SUM(quantity*unit_price) AS revenue
FROM order_items
GROUP BY category
ORDER BY revenue DESC;
```
- Extension: Format revenue with `ROUND(.,2)`.

### Exercise 2: Daily line counts
- Difficulty: Beginner
- Task: Return `order_date` and `COUNT(*) AS line_count` sorted ASC.
- Solution:
```sql
SELECT order_date, COUNT(*) AS line_count
FROM order_items
GROUP BY order_date
ORDER BY order_date ASC;
```

### Exercise 3: Average price per category
- Difficulty: Intermediate
- Task: Return `category`, `AVG(unit_price) AS avg_price` with 2-decimal rounding; sort by avg_price DESC.
- Solution:
```sql
SELECT category, ROUND(AVG(unit_price),2) AS avg_price
FROM order_items
GROUP BY category
ORDER BY avg_price DESC;
```

### Exercise 4: HAVING threshold
- Difficulty: Intermediate
- Task: Show categories with total units >= 5.
- Solution:
```sql
SELECT category, SUM(quantity) AS units
FROM order_items
GROUP BY category
HAVING units >= 5
ORDER BY units DESC;
```

### Exercise 5: Grand total with ROLLUP
- Difficulty: Advanced
- Task: Show units per category and a grand total row using ROLLUP.
- Expected output: Category rows plus a NULL category total.
- Solution:
```sql
SELECT category, SUM(quantity) AS units
FROM order_items
GROUP BY category WITH ROLLUP;
```
- Extension: Replace NULL in the rollup row with 'TOTAL' using `COALESCE(category, 'TOTAL')`.

## 6) Self-assessment checklist
- Can you explain COUNT(*) vs COUNT(col)? (Ex. 1)
- Can you group by one or more columns? (Ex. 1/3)
- Can you filter groups using HAVING? (Ex. 4)
- Can you produce a grand total with ROLLUP? (Ex. 5)
- Can you handle NULLs in aggregates intentionally?

## 7) Connection to real work
- Scenarios: Revenue dashboards, ops KPIs, sales leaderboards.
- Roles: Analyst, BI Developer, FP&A, Product/Operations.
- Interview questions:
  - Q: Why might COUNT(col) be lower than COUNT(*)?
    - A: COUNT(col) ignores NULL values.
  - Q: Difference between WHERE and HAVING?
    - A: WHERE filters rows pre-aggregation; HAVING filters groups post-aggregation.
  - Q: When to use ROLLUP?
    - A: Quick subtotals/grand totals when supported; otherwise UNION separate totals.
