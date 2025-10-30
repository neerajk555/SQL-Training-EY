# Module 1: Introduction to Databases and SQL (MySQL)

## 1) Introduction

Real-world scenario: You’ve joined a growing e‑commerce startup. Leadership needs quick answers like “How many orders did we ship yesterday?”, “Which products are out of stock?”, and “Who are our top 5 customers?”. These answers live in a database, and SQL is the language we’ll use to ask those questions.

Plain definition:
- Database: An organized collection of data stored in tables.
- Table: Like a spreadsheet—with rows (records) and columns (attributes).
- Row: A single record (e.g., one customer).
- Column: A field/attribute about the record (e.g., customer_email).
- Primary Key (PK): A column or set of columns that uniquely identify a row.
- SQL (Structured Query Language): The standard language to store, find, and analyze data.

Connection to upcoming modules:
- In the next lessons, we’ll query data with SELECT, work with data types and functions, summarize with GROUP BY, combine tables with JOINs, and design robust schemas.

Learning objectives:
- Understand what databases, tables, rows, and columns are.
- Explain what SQL is and where it’s used.
- Run your first queries in MySQL.
- Recognize basic SQL categories (DDL, DML, DQL, DCL, TCL).
- Avoid common beginner pitfalls (NULLs, semicolons, and naming).

## 2) Concept explanation

Breakdown of key ideas:
- RDBMS (Relational Database Management System): Software like MySQL that stores data in related tables.
- Schema/Database: A named collection of tables and other objects. In MySQL, we commonly say “database,” which is similar to a schema in other systems.
- SQL categories:
  - DDL: Data Definition Language (CREATE, ALTER, DROP)
  - DML: Data Manipulation Language (INSERT, UPDATE, DELETE)
  - DQL: Data Query Language (SELECT)
  - DCL: Data Control Language (GRANT, REVOKE)
  - TCL: Transaction Control Language (COMMIT, ROLLBACK)
- Basic syntax conventions in MySQL:
  - Keywords in UPPERCASE: SELECT, FROM, WHERE
  - End statements with a semicolon (;)
  - Comments: `-- single line`, `# single line`, `/* multi-line */`

ASCII visuals:

Table (Customers)
+-------------+------------------+-----------------+
| customer_id | customer_name    | customer_email  |
+-------------+------------------+-----------------+
| 1           | Sara Jones       | sara@example.com|
| 2           | Leo Park         | leo@shop.com    |
+-------------+------------------+-----------------+

Relationships (One-to-Many):
Customers (1) ───< Orders (many)

Common misconceptions (and fixes):
- “SQL is case-sensitive.” In MySQL, keywords are not; table/column name case depends on OS and settings. Still, stick to one style.
- “NULL means zero or empty string.” No—NULL means “unknown/missing.” It behaves differently in comparisons and aggregates.
- “Semicolons are optional.” In interactive clients, missing semicolons often cause a hanging prompt. Always end statements with ;

## 3) Detailed examples

We’ll use a tiny e‑commerce dataset. Run the setup first:

```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS customers;
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL,
  customer_email VARCHAR(100)
);

DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

INSERT INTO customers (customer_id, customer_name, customer_email) VALUES
  (1, 'Sara Jones', 'sara@example.com'),
  (2, 'Leo Park', 'leo@shop.com'),
  (3, 'Mina Lee', NULL);

INSERT INTO orders (order_id, customer_id, order_date, total_amount) VALUES
  (101, 1, '2025-01-05', 120.00),
  (102, 1, '2025-01-06', 55.50),
  (103, 2, '2025-01-06', 250.00),
  (104, 2, '2025-01-10', 10.00);
```

### Example A — Your first SELECT
- Business question: “List all customers so I can see who is in the system.”

SQL (with comments):
```sql
-- Return every row and specified columns
SELECT 
  customer_id,      -- unique identifier
  customer_name,    -- person’s name
  customer_email    -- may be NULL
FROM customers;     -- table to read from
```

Sample input (customers):
| customer_id | customer_name | customer_email     |
|-------------|---------------|--------------------|
| 1           | Sara Jones    | sara@example.com   |
| 2           | Leo Park      | leo@shop.com       |
| 3           | Mina Lee      | NULL               |

Expected output:
| customer_id | customer_name | customer_email     |
|-------------|---------------|--------------------|
| 1           | Sara Jones    | sara@example.com   |
| 2           | Leo Park      | leo@shop.com       |
| 3           | Mina Lee      | NULL               |

How SQL processes this:
1) FROM customers → read the table.
2) SELECT columns → project three columns.
3) Return all rows (no WHERE filter applied).

Why this approach and alternatives:
- This is the simplest projection query. Alternative: `SELECT * FROM customers;` for all columns. In production, prefer listing columns to avoid surprises when schema changes.

### Example B — Filter and sort
- Business question: “Show orders above $50, newest first.”

SQL:
```sql
SELECT 
  order_id,
  customer_id,
  order_date,
  total_amount
FROM orders
WHERE total_amount > 50           -- filter rows
ORDER BY order_date DESC, order_id DESC;  -- stable order
```

Sample input (orders):
| order_id | customer_id | order_date | total_amount |
|---------:|------------:|------------|-------------:|
| 101      | 1           | 2025-01-05 | 120.00       |
| 102      | 1           | 2025-01-06 | 55.50        |
| 103      | 2           | 2025-01-06 | 250.00       |
| 104      | 2           | 2025-01-10 | 10.00        |

Expected output:
| order_id | customer_id | order_date | total_amount |
|---------:|------------:|------------|-------------:|
| 104      | 2           | 2025-01-10 | 10.00        | ← filtered OUT (≤ 50)
| 103      | 2           | 2025-01-06 | 250.00       |
| 102      | 1           | 2025-01-06 | 55.50        |
| 101      | 1           | 2025-01-05 | 120.00       |

Actual expected result:
| order_id | customer_id | order_date | total_amount |
|---------:|------------:|------------|-------------:|
| 103      | 2           | 2025-01-06 | 250.00       |
| 102      | 1           | 2025-01-06 | 55.50        |
| 101      | 1           | 2025-01-05 | 120.00       |

Processing:
1) FROM orders → read table.
2) WHERE total_amount > 50 → keep rows 101, 102, 103.
3) ORDER BY date DESC then id DESC.
4) SELECT listed columns.

Why and alternatives:
- WHERE focuses on relevant rows; ORDER BY answers “newest first.” Alternative: Add `LIMIT 5` for just the top five records.

### Example C — Simple aggregate
- Business question: “How many orders does each customer have?”

SQL:
```sql
SELECT 
  customer_id,
  COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
ORDER BY order_count DESC;
```

Expected output:
| customer_id | order_count |
|------------:|------------:|
| 1           | 2           |
| 2           | 2           |

Processing:
1) FROM orders → all rows.
2) GROUP BY customer_id → form groups.
3) COUNT(*) per group.
4) ORDER BY result of aggregation.

Why and alternatives:
- GROUP BY is the standard way to summarize. Alternative: To include customers with zero orders, JOIN customers to orders (we’ll cover joins later) and use `COUNT(o.order_id)`.

Note for PostgreSQL/SQL Server:
- Basic SELECT/WHERE/ORDER BY/GROUP BY syntax is identical. Small differences include functions and quoting identifiers. MySQL allows backticks for identifiers; SQL Server uses square brackets; PostgreSQL uses double quotes.

## 4) Common patterns and best practices

Common use cases:
1) List data for review (SELECT with LIMIT).
2) Filter by date/amount/status.
3) Sort reports by most recent or highest value.
4) Count totals per category (GROUP BY).
5) Find missing data (WHERE column IS NULL).
6) Sample a few rows (ORDER BY RAND() LIMIT 5) — use sparingly.

Templates:
```sql
-- Top N newest records
SELECT col_list
FROM table_name
ORDER BY created_at DESC
LIMIT 10;

-- Records with missing required info
SELECT *
FROM table_name
WHERE important_col IS NULL;

-- Simple summary
SELECT category_col, COUNT(*) AS cnt
FROM table_name
GROUP BY category_col
ORDER BY cnt DESC;
```

Performance notes (beginner-friendly):
- Filtering early with WHERE reduces work.
- Sorting (ORDER BY) can be expensive; later you’ll learn how indexes help.
- Avoid SELECT * in large tables to reduce data scanned and transferred.

Do’s and Don’ts:
- Do: Use clear, consistent names (snake_case or lowerCamelCase; pick one).
- Do: Comment non-obvious logic.
- Do: Specify column lists instead of SELECT *.
- Don’t: Assume NULL equals empty string or zero.
- Don’t: Forget semicolons; many clients require them.

Troubleshooting tips:
- “ERROR 1146 (42S02): Table doesn’t exist” → Check database selection (`USE your_db;`) and spelling.
- “Unknown column” → Verify column names and aliases.
- “You have an error in your SQL syntax” → Look near the reported position; check commas, quotes, and semicolons.

## 5) Practice exercises (MySQL)

All exercises include full sample data and solutions. Run them in the `sql_training` database.

### Exercise 1: List all customers
- Difficulty: Beginner
- Business context: Support wants to verify all customer accounts.
- Sample data:
```sql
DROP TABLE IF EXISTS customers;
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL,
  customer_email VARCHAR(100)
);
INSERT INTO customers VALUES
  (1,'Sara Jones','sara@example.com'),
  (2,'Leo Park','leo@shop.com'),
  (3,'Mina Lee',NULL);
```
- Task: Return all rows and columns from `customers`.
- Expected output:
| customer_id | customer_name | customer_email     |
|-------------|---------------|--------------------|
| 1           | Sara Jones    | sara@example.com   |
| 2           | Leo Park      | leo@shop.com       |
| 3           | Mina Lee      | NULL               |
- Hints:
  1) Use SELECT and FROM.
  2) You can list columns or use `*`.
  3) Don’t forget the semicolon.
- Solution:
```sql
SELECT *
FROM customers;
```
- Extension: Return only `customer_id, customer_name`.

### Exercise 2: Filter orders by amount
- Difficulty: Beginner
- Business context: Finance wants orders greater than $100.
- Sample data:
```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL
);
INSERT INTO orders VALUES
 (101,1,'2025-01-05',120.00),
 (102,1,'2025-01-06',55.50),
 (103,2,'2025-01-06',250.00),
 (104,2,'2025-01-10',10.00);
```
- Task: List `order_id, total_amount` for orders over 100, sorted by amount DESC.
- Expected output:
| order_id | total_amount |
|---------:|-------------:|
| 103      | 250.00       |
| 101      | 120.00       |
- Hints:
  1) WHERE filters rows.
  2) ORDER BY can sort DESC.
  3) Select just two columns.
- Solution:
```sql
SELECT order_id, total_amount
FROM orders
WHERE total_amount > 100
ORDER BY total_amount DESC;
```
- Extension: Add `LIMIT 1` to get the single biggest order.

### Exercise 3: Customers with missing emails
- Difficulty: Intermediate
- Business context: Marketing needs to fix missing contacts.
- Sample data: Use `customers` from Exercise 1.
- Task: Find customers where `customer_email` is NULL.
- Expected output:
| customer_id | customer_name | customer_email |
|-------------|---------------|----------------|
| 3           | Mina Lee      | NULL           |
- Hints:
  1) NULL is special; don’t use `=`.
  2) Use `IS NULL`.
  3) Return all columns.
- Solution:
```sql
SELECT *
FROM customers
WHERE customer_email IS NULL;
```
- Extension: Return the count of customers with missing emails.

### Exercise 4: Daily order totals
- Difficulty: Intermediate
- Business context: Ops wants to see how many orders happened per day.
- Sample data: Use `orders` from Exercise 2.
- Task: Return `order_date` and `COUNT(*) AS order_count` sorted by date ASC.
- Expected output:
| order_date | order_count |
|------------|------------:|
| 2025-01-05 | 1           |
| 2025-01-06 | 2           |
| 2025-01-10 | 1           |
- Hints:
  1) Use GROUP BY on the date.
  2) COUNT(*) counts rows per group.
  3) ORDER BY the date.
- Solution:
```sql
SELECT order_date, COUNT(*) AS order_count
FROM orders
GROUP BY order_date
ORDER BY order_date ASC;
```
- Extension: Also sum `total_amount` per day.

### Exercise 5: Top spending customer
- Difficulty: Advanced
- Business context: Sales wants to know who spends the most in total.
- Sample data: Use `customers` and `orders`.
- Task: Return the `customer_id` with the highest sum of `total_amount`.
- Expected output:
| customer_id | total_spent |
|------------:|------------:|
| 1 or 2      | 175.50 or 260.00 |  ← depends on data; with given data, customer 2 wins
- Hints:
  1) GROUP BY customer_id.
  2) SUM total_amount.
  3) ORDER BY sum DESC and LIMIT 1.
- Solution:
```sql
SELECT customer_id, SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 1;
```
- Extension: Show the customer’s name by joining to `customers` (we’ll cover JOINs in Module 5).

## 6) Self-assessment checklist
- Can you explain the difference between a database, table, row, and column? (Ex. 1)
- Can you run a SELECT to return specific columns? (Ex. 1–2)
- Can you filter rows with WHERE? (Ex. 2)
- Can you handle NULLs with IS NULL? (Ex. 3)
- Can you sort results with ORDER BY? (Ex. 2)
- Can you aggregate with COUNT and GROUP BY? (Ex. 4)
- Can you compute sums and pick the top result with LIMIT? (Ex. 5)
- Can you read error messages and fix simple syntax issues?
- Can you describe DDL vs. DML vs. DQL?
- Can you explain why SELECT * is risky?

## 7) Connection to real work

Real job scenarios:
1) Customer success reviews account lists (SELECT, WHERE, ORDER BY).
2) Finance compiles revenue reports by day (GROUP BY, SUM).
3) Operations checks incomplete records to improve data quality (IS NULL checks).

Typical roles using these skills:
- Data Analyst, Business Analyst, Product Analyst, Support Engineer, Operations Manager.

Sample interview questions with answers:
- Q: What’s the difference between WHERE and HAVING?
  - A: WHERE filters rows before grouping; HAVING filters groups after aggregation. For non-aggregated queries, use WHERE.
- Q: How does NULL affect COUNT(*) vs COUNT(column)?
  - A: COUNT(*) counts rows, including those with NULLs; COUNT(column) counts non-NULL values in that column.
- Q: Why avoid SELECT * in production queries?
  - A: It can pull unnecessary data, break code when schemas change, and prevent index-only scans.

PostgreSQL/SQL Server notes:
- Most of this module’s syntax is identical. MySQL uses backticks (optional) for identifiers; SQL Server uses brackets; PostgreSQL uses double quotes. Functions and date handling vary slightly and will be discussed when relevant.
