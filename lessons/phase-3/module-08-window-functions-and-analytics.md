# Module 8: Window Functions and Analytics (MySQL 8+)

## 1) Introduction

Scenario: “What’s the running total of revenue by month?”, “Rank products by sales within each category,” and “Compare each day’s sales to the previous day.” Window functions make these analytical tasks concise and performant.

Definition (plain):
- Window function: An aggregate-like function that returns a value per row while looking at other rows in a window (partition), without collapsing rows.
- PARTITION BY: Defines groups (like GROUP BY) for window calculation.
- ORDER BY in window: Defines ordering within the partition for functions like LAG/LEAD, RANK.
- Frame: ROWS/RANGE clause narrowing the window around the current row.

Connection to previous/next:
- Builds on aggregations and joins. Complements subqueries and CTEs.

Learning objectives:
- Use ROW_NUMBER, RANK, DENSE_RANK to rank rows.
- Compute running totals/averages with SUM/AVG OVER.
- Use LAG/LEAD for period-over-period comparisons.
- Control windows with PARTITION BY, ORDER BY, and frames.

## 2) Concept explanation

Syntax:
```sql
func() OVER (
  PARTITION BY col_list
  ORDER BY col_list
  [ROWS|RANGE frame_spec]
)
```

Common window functions:
- Ranking: ROW_NUMBER(), RANK(), DENSE_RANK()
- Navigation: LAG(expr, offset, default), LEAD(expr, offset, default)
- Aggregates over windows: SUM(), AVG(), MIN(), MAX(), COUNT()

Default frame:
- For ORDER BY, MySQL uses `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` by default (for numeric/date ORDER BY). Use ROWS for deterministic row-count frames.

Common misconceptions:
- “Window functions replace GROUP BY.” No—windows don’t collapse rows; GROUP BY does.
- “ORDER BY in SELECT is enough.” You need ORDER BY inside OVER() to define calculation order.
- “Running totals always mean cumulative over entire table.” Use PARTITION BY to reset per group.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS daily_sales;
CREATE TABLE daily_sales (
  sale_date DATE,
  category VARCHAR(50),
  revenue DECIMAL(10,2)
);

INSERT INTO daily_sales VALUES
 ('2025-01-01','Electronics',100.00),
 ('2025-01-02','Electronics', 75.00),
 ('2025-01-03','Electronics', 50.00),
 ('2025-01-01','Home',        20.00),
 ('2025-01-02','Home',        25.00),
 ('2025-01-03','Home',        30.00);
```

### Example A — Running total per category
- Business question: “Show cumulative revenue by category over time.”

SQL:
```sql
SELECT 
  category,
  sale_date,
  revenue,
  SUM(revenue) OVER (
    PARTITION BY category
    ORDER BY sale_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_rev
FROM daily_sales
ORDER BY category, sale_date;
```

Expected output (Electronics): 100, 175, 225; (Home): 20, 45, 75.

### Example B — Rank days by revenue within category
- Business question: “Rank each day by revenue within its category (1 = highest).”

SQL:
```sql
SELECT 
  category,
  sale_date,
  revenue,
  RANK() OVER (
    PARTITION BY category
    ORDER BY revenue DESC
  ) AS rev_rank
FROM daily_sales
ORDER BY category, rev_rank;
```

Expected output: Ranks 1 to 3 per category, ties get same rank (gaps allowed).

### Example C — Day-over-day change with LAG
- Business question: “How much did revenue change vs the previous day in each category?”

SQL:
```sql
SELECT 
  category,
  sale_date,
  revenue,
  LAG(revenue, 1, 0) OVER (
    PARTITION BY category
    ORDER BY sale_date
  ) AS prev_rev,
  revenue - LAG(revenue, 1, 0) OVER (
    PARTITION BY category
    ORDER BY sale_date
  ) AS delta
FROM daily_sales
ORDER BY category, sale_date;
```

Expected output: delta = 75-100 = -25, 50-75 = -25; Home: +5, +5.

Portability notes:
- PostgreSQL/SQL Server: Similar syntax; SQL Server requires ORDER BY for window aggregates to define frame semantics.

## 4) Common patterns and best practices

Use cases:
1) Running totals/averages per group.
2) Ranking and top-N per segment.
3) Period-over-period (PoP) comparisons via LAG/LEAD.
4) Percent-of-total: `value / SUM(value) OVER (PARTITION BY ...)`.

Templates:
```sql
-- Top-N per group using ROW_NUMBER
WITH ranked AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
  FROM daily_sales
)
SELECT * FROM ranked WHERE rn <= 2;

-- Share of category total
SELECT category, sale_date, revenue,
       revenue / SUM(revenue) OVER (PARTITION BY category) AS share
FROM daily_sales;
```

Performance notes:
- Window functions can be heavy; indexes on PARTITION BY and ORDER BY columns help.
- Use ROWS frames for deterministic row counts; RANGE depends on value equality and can be surprising.

Do’s and Don’ts:
- Do: Specify ORDER BY in OVER for navigational and cumulative functions.
- Do: Use PARTITION BY to reset windows per group.
- Don’t: Mix GROUP BY and window columns incorrectly—compute aggregates first in a CTE if needed.
- Don’t: Forget that window ORDER BY is independent from the final SELECT ORDER BY.

Troubleshooting:
- “Window function is not allowed in this context” → Some clauses disallow window functions; use subqueries/CTEs.
- Wrong running totals → Add an explicit ROWS frame.

## 5) Practice exercises

### Exercise 1: Running revenue
- Difficulty: Beginner
- Task: Compute running revenue per category by sale_date.
- Solution: See Example A.

### Exercise 2: Rank days by revenue
- Difficulty: Beginner
- Task: Rank days within each category by revenue descending.
- Solution: See Example B.

### Exercise 3: Day-over-day delta
- Difficulty: Intermediate
- Task: Show revenue delta vs previous day per category using LAG.
- Solution: See Example C.

### Exercise 4: Top-2 days per category
- Difficulty: Intermediate
- Task: Use ROW_NUMBER in a CTE to return top 2 revenue days per category.
- Solution:
```sql
WITH ranked AS (
  SELECT *, ROW_NUMBER() OVER (
           PARTITION BY category ORDER BY revenue DESC) AS rn
  FROM daily_sales
)
SELECT category, sale_date, revenue
FROM ranked
WHERE rn <= 2
ORDER BY category, rn;
```

### Exercise 5: Share of total
- Difficulty: Advanced
- Task: For each day, compute percent of category’s total revenue.
- Solution:
```sql
SELECT category, sale_date, revenue,
       revenue / SUM(revenue) OVER (PARTITION BY category) AS pct_of_cat
FROM daily_sales
ORDER BY category, sale_date;
```

## 6) Self-assessment checklist
- Can you compute running totals with SUM OVER? (Ex. 1)
- Can you rank rows within a group? (Ex. 2/4)
- Can you compute period-over-period deltas using LAG? (Ex. 3)
- Can you build share-of-total metrics? (Ex. 5)

## 7) Connection to real work
- Scenarios: Trend dashboards, leaderboards, conversion funnels.
- Roles: Analyst, BI Developer, Data Scientist.
- Interview questions:
  - Q: Difference between GROUP BY and window functions?
    - A: GROUP BY collapses rows; windows return a value per row while referencing others.
  - Q: Why specify ROWS vs RANGE?
    - A: ROWS counts physical rows; RANGE groups peers with equal ORDER BY values—can change results.
  - Q: How do you get top-N per group?
    - A: Rank with ROW_NUMBER() OVER (PARTITION BY ...) and filter by rn.
