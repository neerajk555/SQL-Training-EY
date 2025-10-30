# Module 3: Working with Data Types and Basic Functions (MySQL)

## 1) Introduction

Scenario: You need to calculate discounts, format names, and prepare date-based reports. Getting data types and basic functions right makes your queries correct and reliable.

Definition (plain):
- Data type: The kind of value a column can store (number, text, date, etc.).
- Function: A built-in operation that transforms values (e.g., ROUND, CONCAT, DATE_ADD).

Connection to previous/next:
- Builds on SELECT and WHERE from Module 2. You’ll use these functions everywhere, including with GROUP BY and JOINs. Later modules add advanced functions (window, procedural, etc.).

Learning objectives:
- Choose the right MySQL data types for numbers, text, and dates.
- Use string, numeric, and date functions to transform data.
- Handle NULLs safely with COALESCE/IFNULL.
- Convert types explicitly with CAST/CONVERT.

## 2) Concept explanation

Common MySQL data types:
- Integers: TINYINT, SMALLINT, INT, BIGINT
- Fixed-point decimals: DECIMAL(p,s) — good for money
- Floating-point: FLOAT, DOUBLE — beware tiny rounding errors
- Text: CHAR(n) fixed-length, VARCHAR(n) variable-length, TEXT for long text
- Date/time: DATE, DATETIME, TIMESTAMP, TIME; YEAR
- Boolean: MySQL uses TINYINT(1) (0/1); TRUE = 1, FALSE = 0
- Enumerations: ENUM('A','B',...), SET — use sparingly for portability
- NULL: Unknown or missing value (not zero or empty string)

Basic functions (selected):
- String: CONCAT, CONCAT_WS, SUBSTRING, LEFT, RIGHT, LENGTH, LOWER, UPPER, TRIM, REPLACE
- Numeric: ROUND, CEIL, FLOOR, ABS, MOD
- Date/Time: NOW(), CURDATE(), DATE(), DATE_ADD, DATE_SUB, DATEDIFF, TIMESTAMPDIFF, EXTRACT
- Null-handling: COALESCE(x, y, ...), IFNULL(x, alt)
- Type conversion: CAST(expr AS TYPE), CONVERT(expr, TYPE)

Syntax examples:
```sql
SELECT 
  CONCAT(first_name, ' ', last_name) AS full_name,
  ROUND(price * 1.07, 2) AS price_with_tax,
  DATE_ADD(order_date, INTERVAL 7 DAY) AS ship_by,
  COALESCE(discount_pct, 0) AS safe_discount
FROM table_name;
```

Common misconceptions:
- “DECIMAL and FLOAT are the same.” No—use DECIMAL for money to avoid floating-point rounding issues.
- “NULL equals empty string.” No—use COALESCE/IFNULL to replace NULL when needed.
- “TIMESTAMP and DATETIME are interchangeable.” They store differently; TIMESTAMP is timezone-aware and limited range, DATETIME is not.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100),
  salary DECIMAL(10,2) NOT NULL,
  start_date DATE NOT NULL,
  department VARCHAR(50) NOT NULL
);

INSERT INTO employees VALUES
 (1,'Aria','Ng','aria@corp.com',70000.00,'2023-06-01','Engineering'),
 (2,'Ben','Ortiz',NULL,52000.00,'2024-01-15','Support'),
 (3,'Chloe','Park','chloe@corp.com',48000.50,'2023-09-20','Marketing'),
 (4,'Dev','Singh','dev@corp.com',120000.00,'2022-11-05','Engineering');
```

### Example A — String formatting and aliases
- Business question: “Show employee full names and email handles.”

SQL:
```sql
SELECT 
  CONCAT(first_name, ' ', last_name) AS full_name,
  email,
  LOWER(SUBSTRING(email, 1, LOCATE('@', email) - 1)) AS email_handle
FROM employees;
```

Expected output (NULL email produces NULL handle):
| full_name | email           | email_handle |
|-----------|-----------------|--------------|
| Aria Ng   | aria@corp.com   | aria         |
| Ben Ortiz | NULL            | NULL         |
| Chloe Park| chloe@corp.com  | chloe        |
| Dev Singh | dev@corp.com    | dev          |

Processing:
1) CONCAT creates full name.
2) SUBSTRING/LOCATE parse the email left of '@'.
3) LOWER normalizes case.

Why/alternatives:
- `CONCAT_WS(' ', first_name, last_name)` automatically ignores NULLs.

### Example B — Numeric math with rounding and NULL safety
- Business question: “Show salary, 10% bonus, and total compensation. Treat missing salary adjustments as 0.”

SQL:
```sql
SELECT 
  employee_id,
  salary,
  ROUND(salary * 0.10, 2) AS bonus,
  ROUND(salary + COALESCE(salary * 0.10, 0), 2) AS total_comp
FROM employees;
```

Expected output (excerpt):
| employee_id | salary   | bonus   | total_comp |
|------------:|---------:|--------:|-----------:|
| 1           | 70000.00 | 7000.00 | 77000.00   |

Processing:
1) Compute 10% bonus.
2) Add to salary; ROUND for display.

Why/alternatives:
- COALESCE ensures NULL-safe math. DECIMAL avoids floating rounding issues.

### Example C — Dates: tenure and future dates
- Business question: “How many days has each employee worked, and what’s their 1-year anniversary date?”

SQL:
```sql
SELECT 
  employee_id,
  start_date,
  DATEDIFF(CURDATE(), start_date) AS days_worked,
  DATE_ADD(start_date, INTERVAL 1 YEAR) AS one_year_anniv
FROM employees
ORDER BY days_worked DESC;
```

Expected output (conceptual):
| employee_id | start_date | days_worked | one_year_anniv |
|------------:|-----------|------------:|----------------|
| 4           | 2022-11-05| ...         | 2023-11-05     |
| 1           | 2023-06-01| ...         | 2024-06-01     |

Processing:
1) DATEDIFF from today.
2) DATE_ADD for 1-year mark.

Why/alternatives:
- `TIMESTAMPDIFF(DAY, start_date, CURDATE())` provides similar results; choose based on readability.

Portability notes:
- PostgreSQL: `NOW()` returns timestamp; use `CURRENT_DATE` or `::date` casts. DATE_ADD becomes `start_date + INTERVAL '1 year'`.
- SQL Server: Use `GETDATE()` and `DATEADD(YEAR, 1, start_date)`; DATEDIFF has signature `DATEDIFF(day, start_date, GETDATE())`.

## 4) Common patterns and best practices

Use cases:
1) Clean and standardize text for reporting (LOWER/UPPER/TRIM/REPLACE).
2) Monetary calculations with DECIMAL and ROUND.
3) Date rollups: week starts, month starts, anniversaries.
4) Null safety with COALESCE/IFNULL.
5) Casting strings to numbers or dates before comparison.

Templates:
```sql
-- Null-safe label with COALESCE
SELECT COALESCE(nickname, first_name) AS display_name FROM people;

-- Month start from any date
SELECT DATE_FORMAT(order_date, '%Y-%m-01') AS month_start FROM orders; -- MySQL-specific

-- Safe numeric casting
SELECT CAST(price_str AS DECIMAL(10,2)) AS price FROM staging_prices;
```

Performance notes:
- Functions on columns in WHERE can prevent index usage (e.g., `WHERE LOWER(email) = 'x'`). Prefer transforming the parameter or using generated columns.
- DATE functions on large tables can be expensive—precompute or index when possible.

Do’s and Don’ts:
- Do: Use DECIMAL for currency.
- Do: Handle NULLs intentionally.
- Don’t: Store dates as strings if you plan to filter/sort by date.
- Don’t: Assume TRIM removes internal spaces—it removes leading/trailing; use REPLACE for internal.

Troubleshooting:
- “Invalid date” → Check format 'YYYY-MM-DD' for DATE in MySQL.
- “Truncated incorrect DECIMAL value” → CAST text to DECIMAL before math.
- “Function does not exist” → Some functions are DB-specific; verify MySQL equivalents.

## 5) Practice exercises

### Exercise 1: Clean names
- Difficulty: Beginner
- Context: HR wants consistent employee display names.
- Sample data:
```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100),
  salary DECIMAL(10,2) NOT NULL,
  start_date DATE NOT NULL,
  department VARCHAR(50) NOT NULL
);
INSERT INTO employees VALUES
 (1,'Aria','Ng','aria@corp.com',70000.00,'2023-06-01','Engineering'),
 (2,'Ben','Ortiz',NULL,52000.00,'2024-01-15','Support'),
 (3,'Chloe','Park','chloe@corp.com',48000.50,'2023-09-20','Marketing');
```
- Task: Return `UPPER(last_name) || ', ' || first_name` style in MySQL using CONCAT and UPPER.
- Expected output:
| display_name |
|--------------|
| NG, Aria     |
| ORTIZ, Ben   |
| PARK, Chloe  |
- Hints: CONCAT(', ') and UPPER.
- Solution:
```sql
SELECT CONCAT(UPPER(last_name), ', ', first_name) AS display_name
FROM employees;
```
- Extension: Trim accidental spaces with TRIM.

### Exercise 2: Email or fallback
- Difficulty: Beginner
- Context: Show email if present, otherwise 'missing@corp.com'.
- Sample data: Use employees above.
- Task: Return `COALESCE(email,'missing@corp.com')` as `safe_email`.
- Expected output: One row will show the fallback value.
- Solution:
```sql
SELECT COALESCE(email, 'missing@corp.com') AS safe_email
FROM employees;
```
- Extension: Extract the email domain for non-NULL emails.

### Exercise 3: Compensation math
- Difficulty: Intermediate
- Context: Finance wants salary, 5% bonus, and total rounded to 2 decimals.
- Task: Return `salary`, `salary*0.05 AS bonus`, `ROUND(salary*1.05,2) AS total`.
- Expected output: Rounded totals.
- Solution:
```sql
SELECT salary,
       salary*0.05 AS bonus,
       ROUND(salary*1.05,2) AS total
FROM employees;
```
- Extension: Cap the bonus at 4,000 using `LEAST(bonus, 4000)` (MySQL supports LEAST).

### Exercise 4: Tenure in months
- Difficulty: Intermediate
- Context: HR needs months of service for benefits.
- Task: Use `TIMESTAMPDIFF(MONTH, start_date, CURDATE()) AS months_service`.
- Expected output: Integer months since start_date.
- Solution:
```sql
SELECT employee_id,
       TIMESTAMPDIFF(MONTH, start_date, CURDATE()) AS months_service
FROM employees
ORDER BY months_service DESC;
```
- Extension: Also show full years using `FLOOR(months_service/12)`.

### Exercise 5: Parsing strings
- Difficulty: Advanced
- Context: A staging table has names like 'Last|First'. Build clean columns.
- Sample data:
```sql
DROP TABLE IF EXISTS staging_names;
CREATE TABLE staging_names (
  raw_name VARCHAR(100)
);
INSERT INTO staging_names VALUES
 ('Ng|Aria'), ('Ortiz|Ben'), ('Park|Chloe'), ('Singh|Dev');
```
- Task: Return `first_name` and `last_name` by splitting on '|'.
- Expected output:
| first_name | last_name |
|------------|-----------|
| Aria       | Ng        |
| Ben        | Ortiz     |
| Chloe      | Park      |
| Dev        | Singh     |
- Hints: Use `SUBSTRING_INDEX(raw_name, '|', 1)` and `SUBSTRING_INDEX(raw_name, '|', -1)`.
- Solution:
```sql
SELECT 
  SUBSTRING_INDEX(raw_name, '|', -1) AS first_name,
  SUBSTRING_INDEX(raw_name, '|', 1)  AS last_name
FROM staging_names;
```
- Extension: Handle rows without a '|' by returning NULLs safely.

## 6) Self-assessment checklist
- Can you choose DECIMAL for money and explain why? (Ex. 3)
- Can you format names with CONCAT/UPPER/LOWER/TRIM? (Ex. 1)
- Can you handle NULLs using COALESCE/IFNULL? (Ex. 2)
- Can you perform date math with DATE_ADD/DATEDIFF/TIMESTAMPDIFF? (Ex. 4)
- Can you parse strings using SUBSTRING/LOCATE/SUBSTRING_INDEX? (Ex. 5)
- Can you CAST text to numeric before math?

## 7) Connection to real work
- Job scenarios:
  1) Finance reports clean monetary values with consistent rounding.
  2) Marketing standardizes names/emails for CRM imports.
  3) HR computes tenure-based benefits windows.
- Roles: Analyst, Data Engineer (staging/cleaning), Finance Ops, HR Ops.
- Interview questions:
  - Q: When do you use DECIMAL vs FLOAT?
    - A: DECIMAL for exact precision (currency), FLOAT/DOUBLE for scientific/approximate calculations.
  - Q: How do NULLs interact with concatenation and arithmetic?
    - A: CONCAT with NULL → NULL unless CONCAT_WS; arithmetic with NULL → NULL unless handled via COALESCE/IFNULL.
  - Q: Difference between TIMESTAMP and DATETIME in MySQL?
    - A: TIMESTAMP stores UTC seconds with timezone conversion and has a smaller range; DATETIME stores calendar date/time without timezone, broader display range.
