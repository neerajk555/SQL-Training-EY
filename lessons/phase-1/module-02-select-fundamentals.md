# Module 2: Basic SQL Queries — SELECT Fundamentals (MySQL)

## 1) Introduction

Scenario: Product managers ask, “Which products are low on stock?”, “Show the most recent orders,” and “Find customers with ‘gmail.com’ emails.” SELECT is the core tool we’ll use daily to answer these questions.

Definition (plain):
- SELECT: The SQL statement used to read and shape data from tables.
- Clauses you’ll learn: FROM, WHERE, ORDER BY, LIMIT, DISTINCT; plus column aliases and expressions.

Connection to previous/next:
- Builds on Module 1 basics. You’ll use these patterns in every later module (aggregations, joins, window functions, updates, etc.).

Learning objectives:
- Write SELECT queries with clear column lists.
- Filter rows with WHERE using comparison and logical operators.
- Sort results with ORDER BY and cap rows with LIMIT.
- Use DISTINCT to remove duplicates and aliases to rename outputs.

## 2) Concept explanation

Sub-concepts:
- Projection: Choosing which columns to return (SELECT col1, col2).
- Filtering: Reducing rows with WHERE (e.g., price > 100).
- Sorting: Ordering results with ORDER BY col [ASC|DESC].
- Limiting: Using LIMIT N (and optionally OFFSET) to restrict row count.
- Aliases: `SELECT col AS better_name` or `SELECT expression AS metric`.
- DISTINCT: Collapse duplicates on the selected columns.

Syntax breakdown:
```sql
SELECT [DISTINCT] 
  column_list_or_expressions
FROM table_name
WHERE <conditions>
ORDER BY col1 [ASC|DESC], col2 [ASC|DESC]
LIMIT <row_count> [OFFSET <start>];  -- OFFSET is optional
```

Operators to know:
- Comparison: =, <>, !=, <, <=, >, >=
- Range/Set: BETWEEN a AND b, IN (v1, v2, ...)
- Pattern: LIKE 'prefix%', '%contains%', '_oneChar'
- Null checks: IS NULL / IS NOT NULL
- Logic: AND, OR, NOT  (AND has higher precedence than OR)

Common misconceptions:
- “`=` works with NULL.” No — use IS NULL/IS NOT NULL.
- “DISTINCT sorts results.” No — DISTINCT removes duplicates; add ORDER BY for sorting.
- “ORDER BY happens before WHERE.” No — WHERE filters first, then ORDER BY sorts the remaining rows.

ASCII flow (conceptual):
FROM → WHERE → SELECT → DISTINCT → ORDER BY → LIMIT

## 3) Detailed examples

We’ll use a small inventory/orders dataset.

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS products;
CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100) NOT NULL,
  category VARCHAR(50) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock INT NOT NULL
);

DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  product_id INT NOT NULL,
  order_date DATE NOT NULL,
  quantity INT NOT NULL,
  FOREIGN KEY (product_id) REFERENCES products(product_id)
);

INSERT INTO products VALUES
 (1,'Wireless Mouse','Electronics',25.00,50),
 (2,'Mechanical Keyboard','Electronics',85.00,12),
 (3,'Desk Lamp','Home',19.99,5),
 (4,'Water Bottle','Outdoors',15.00,0),
 (5,'Notebook','Stationery',3.50,300);

INSERT INTO orders VALUES
 (2001,1,'2025-01-08',2),
 (2002,2,'2025-01-09',1),
 (2003,3,'2025-01-10',4),
 (2004,1,'2025-01-11',1),
 (2005,5,'2025-01-11',10);
```

### Example A — Select with expressions and aliases
- Business question: “Show product name, price, and price with tax (8%).”

SQL:
```sql
SELECT 
  product_name,
  price,
  price * 1.08 AS price_with_tax  -- expression with alias
FROM products;
```

Expected output (excerpt):
| product_name         | price | price_with_tax |
|----------------------|------:|---------------:|
| Wireless Mouse       | 25.00 | 27.00          |
| Mechanical Keyboard  | 85.00 | 91.80          |

Processing:
1) FROM products
2) SELECT columns and compute expression

Why/alternatives:
- Expressions let you compute on the fly. You can also round with `ROUND(price * 1.08, 2)`.

### Example B — Filtering with WHERE (AND/OR)
- Business question: “Find Electronics priced over $50 OR anything with stock under 10.”

SQL:
```sql
SELECT product_id, product_name, category, price, stock
FROM products
WHERE (category = 'Electronics' AND price > 50)
   OR stock < 10
ORDER BY stock ASC, price DESC;
```

Expected output (ordered by lowest stock first):
| product_id | product_name        | category    | price | stock |
|-----------:|---------------------|-------------|------:|------:|
| 4          | Water Bottle        | Outdoors    | 15.00 | 0     |
| 3          | Desk Lamp           | Home        | 19.99 | 5     |
| 2          | Mechanical Keyboard | Electronics | 85.00 | 12    |

Processing:
1) FROM products → read rows.
2) WHERE applies compound logic.
3) ORDER BY sorts by stock then price.

Why/alternatives:
- Parentheses clarify order of evaluation. Alternative: Write two separate queries per condition and `UNION` them; WHERE is simpler and faster here.

### Example C — DISTINCT, LIKE, LIMIT/OFFSET
- Business question: “List unique product categories containing an ‘o’ (case-sensitive depends on collation). Show the first 2.”

SQL:
```sql
SELECT DISTINCT category
FROM products
WHERE category LIKE '%o%'
ORDER BY category ASC
LIMIT 2 OFFSET 0;  -- first two
```

Expected output:
| category   |
|------------|
| Electronics|
| Outdoors   |

Processing:
1) FROM → read products
2) WHERE → match LIKE pattern
3) SELECT DISTINCT → remove duplicates
4) ORDER BY → alphabetical
5) LIMIT/OFFSET → pagination

Why/alternatives:
- DISTINCT reduces duplicates in the projection. Alternative: `GROUP BY category` also returns unique categories; use DISTINCT when you don’t need aggregates.

Note for PostgreSQL/SQL Server:
- PostgreSQL uses the same LIMIT/OFFSET. SQL Server uses `OFFSET ... ROWS FETCH NEXT ... ROWS ONLY` with ORDER BY.

## 4) Common patterns and best practices

Use cases:
1) Top N newest orders: ORDER BY date DESC LIMIT N.
2) Search-like filters: WHERE name LIKE '%term%'.
3) Quality checks: WHERE important_col IS NULL or = '' (empty string)—note the difference.
4) De-duplication: SELECT DISTINCT col.
5) Simple KPIs: SELECT COUNT(*) FROM table WHERE condition.
6) Pagination: ORDER BY stable key + LIMIT/OFFSET.

Templates:
```sql
-- Pagination pattern (stable ordering by primary key)
SELECT columns
FROM table
WHERE condition
ORDER BY id ASC
LIMIT 50 OFFSET 0;

-- De-duplicate values
SELECT DISTINCT col
FROM table
WHERE condition;

-- Multi-condition filtering with precedence
SELECT *
FROM table
WHERE (cond_a AND cond_b)
   OR cond_c;
```

Performance notes:
- ORDER BY without an index can be slow on big tables. Pagination cost grows with large OFFSET.
- LIKE '%term%' can’t use typical indexes efficiently; consider full-text indexes in advanced cases.
- DISTINCT requires extra work to remove duplicates; only use when needed.

Do’s and Don’ts:
- Do: Use aliases for readability.
- Do: Use parentheses in complex WHERE logic.
- Don’t: Assume LIKE is case-insensitive; depends on collation. Test it.
- Don’t: Rely on implicit row order; always use ORDER BY for deterministic results.

Troubleshooting:
- “Unknown column” → Misspelled column or alias. Aliases in SELECT can’t be used in WHERE in the same level (use a subquery or repeat the expression).
- “LIMIT not working as expected” → Remember that LIMIT applies after ORDER BY.

## 5) Practice exercises

### Exercise 1: Price with tax and label
- Difficulty: Beginner
- Business context: Display catalog with tax-included price and a label.
- Sample data:
```sql
DROP TABLE IF EXISTS products;
CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100) NOT NULL,
  category VARCHAR(50) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock INT NOT NULL
);
INSERT INTO products VALUES
 (1,'Wireless Mouse','Electronics',25.00,50),
 (2,'Mechanical Keyboard','Electronics',85.00,12),
 (3,'Desk Lamp','Home',19.99,5);
```
- Task: Show `product_name`, `price`, and `ROUND(price*1.08,2) AS price_with_tax`. Add alias `AS label` with value `'Incl. Tax'`.
- Expected output:
| product_name        | price | price_with_tax | label     |
|---------------------|------:|---------------:|-----------|
| Wireless Mouse      | 25.00 | 27.00          | Incl. Tax |
| Mechanical Keyboard | 85.00 | 91.80          | Incl. Tax |
| Desk Lamp           | 19.99 | 21.59          | Incl. Tax |
- Hints: Use expressions and string literals with AS.
- Solution:
```sql
SELECT 
  product_name,
  price,
  ROUND(price*1.08,2) AS price_with_tax,
  'Incl. Tax' AS label
FROM products;
```
- Extension: Add a column `price_category` that shows 'Premium' when price > 50 else 'Standard' (use CASE in Module 3, or simple WHERE filters here).

### Exercise 2: Filter by pattern
- Difficulty: Beginner
- Business context: Find categories that include the letter 'e'.
- Sample data: Use `products` above.
- Task: Return DISTINCT `category` where the name contains 'e'; sort ASC.
- Expected output:
| category    |
|-------------|
| Electronics |
| Home        |
- Hints: DISTINCT + WHERE + LIKE.
- Solution:
```sql
SELECT DISTINCT category
FROM products
WHERE category LIKE '%e%'
ORDER BY category ASC;
```
- Extension: Limit to 1 result and discuss which one appears and why.

### Exercise 3: Low stock or expensive
- Difficulty: Intermediate
- Business context: Flag items either low on stock (< 10) OR expensive (> 80).
- Sample data: Use `products` above.
- Task: Return product_id, product_name, stock, price for those items. Sort by stock ASC then price DESC.
- Expected output (with given data):
| product_id | product_name        | stock | price |
|-----------:|---------------------|------:|------:|
| 3          | Desk Lamp           | 5     | 19.99 |
| 2          | Mechanical Keyboard | 12    | 85.00 |
- Hints: Use parentheses with OR/AND as needed.
- Solution:
```sql
SELECT product_id, product_name, stock, price
FROM products
WHERE stock < 10 OR price > 80
ORDER BY stock ASC, price DESC;
```
- Extension: Add a condition to exclude category 'Home'.

### Exercise 4: Recent orders
- Difficulty: Intermediate
- Business context: Show the 3 most recent orders.
- Sample data:
```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  product_id INT NOT NULL,
  order_date DATE NOT NULL,
  quantity INT NOT NULL
);
INSERT INTO orders VALUES
 (2001,1,'2025-01-08',2),
 (2002,2,'2025-01-09',1),
 (2003,3,'2025-01-10',4),
 (2004,1,'2025-01-11',1),
 (2005,5,'2025-01-11',10);
```
- Task: Return all columns for the 3 newest orders.
- Expected output: The three rows with the latest `order_date` (and highest `order_id` to break ties).
- Hints: ORDER BY date DESC, id DESC; LIMIT 3.
- Solution:
```sql
SELECT *
FROM orders
ORDER BY order_date DESC, order_id DESC
LIMIT 3;
```
- Extension: Implement pagination — next page with `OFFSET 3`.

### Exercise 5: Unique buyers by day
- Difficulty: Advanced
- Business context: Count how many distinct products were ordered per day.
- Sample data: Use `orders` above.
- Task: Return `order_date` and `COUNT(DISTINCT product_id)` sorted by date.
- Expected output:
| order_date | unique_products |
|------------|----------------:|
| 2025-01-08 | 1               |
| 2025-01-09 | 1               |
| 2025-01-10 | 1               |
| 2025-01-11 | 2               |
- Hints: COUNT(DISTINCT ...) + GROUP BY.
- Solution:
```sql
SELECT order_date, COUNT(DISTINCT product_id) AS unique_products
FROM orders
GROUP BY order_date
ORDER BY order_date ASC;
```
- Extension: Also return total quantity per day.

## 6) Self-assessment checklist
- Can you use SELECT to return specific columns and expressions? (Ex. 1)
- Can you filter with WHERE using AND/OR and parentheses? (Ex. 3)
- Can you use LIKE, IN, and BETWEEN appropriately? (Exercises 2/3 variations)
- Can you apply DISTINCT to remove duplicates? (Ex. 2)
- Can you sort deterministically with ORDER BY and LIMIT? (Ex. 4)
- Can you compute distinct counts? (Ex. 5)

## 7) Connection to real work
- Job scenarios:
  1) Product analytics: identify low-stock items for restock.
  2) Marketing: segment customers by email patterns to tailor campaigns.
  3) Operations: daily checks of latest orders and anomalies.
- Roles: Data Analyst, Business Analyst, Product Ops, Support Engineer.
- Interview questions:
  - Q: What’s the difference between WHERE and HAVING?
    - A: WHERE filters rows before aggregation; HAVING filters groups after aggregation.
  - Q: Why is ORDER BY necessary even if rows “look” sorted by default?
    - A: Without ORDER BY, SQL returns rows in unspecified order; only ORDER BY guarantees deterministic order.
  - Q: When should you use DISTINCT vs GROUP BY?
    - A: DISTINCT for deduplicating projections; GROUP BY when you need aggregates.
