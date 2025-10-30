# 📖 SQL Quick Reference Guide

A comprehensive quick reference for MySQL syntax covered in this curriculum.

---

## 📋 Table of Contents

1. [Basic Query Structure](#basic-query-structure)
2. [Data Types](#data-types)
3. [SELECT Statements](#select-statements)
4. [Filtering & Conditions](#filtering--conditions)
5. [Aggregate Functions](#aggregate-functions)
6. [Joins](#joins)
7. [Subqueries & CTEs](#subqueries--ctes)
8. [Window Functions](#window-functions)
9. [Data Modification](#data-modification)
10. [Database Objects](#database-objects)
11. [Indexes](#indexes)
12. [Transactions](#transactions)
13. [Stored Procedures & Functions](#stored-procedures--functions)
14. [Common Functions](#common-functions)
15. [Best Practices](#best-practices)

---

## Basic Query Structure

### Complete SELECT Syntax Order
```sql
SELECT column_list
FROM table_name
WHERE conditions
GROUP BY column_list
HAVING group_conditions
ORDER BY column_list [ASC|DESC]
LIMIT number [OFFSET number];
```

### Database Operations
```sql
-- Create database
CREATE DATABASE database_name;

-- Use database
USE database_name;

-- Drop database
DROP DATABASE database_name;

-- Show databases
SHOW DATABASES;
```

---

## Data Types

### Numeric Types
```sql
INT                    -- Integer (-2,147,483,648 to 2,147,483,647)
BIGINT                 -- Large integer
DECIMAL(p,s)           -- Fixed-point (p=precision, s=scale)
FLOAT                  -- Floating-point
DOUBLE                 -- Double precision floating-point
```

### String Types
```sql
CHAR(n)                -- Fixed-length string
VARCHAR(n)             -- Variable-length string (max n characters)
TEXT                   -- Large text
ENUM('val1','val2')    -- Enumeration
```

### Date & Time Types
```sql
DATE                   -- Date (YYYY-MM-DD)
TIME                   -- Time (HH:MM:SS)
DATETIME               -- Date and time
TIMESTAMP              -- Timestamp
YEAR                   -- Year
```

### Other Types
```sql
BOOLEAN                -- True/False (stored as TINYINT)
JSON                   -- JSON document
BLOB                   -- Binary Large Object
```

---

## SELECT Statements

### Basic SELECT
```sql
-- Select all columns
SELECT * FROM customers;

-- Select specific columns
SELECT first_name, last_name, email FROM customers;

-- Select with alias
SELECT first_name AS fname, last_name AS lname FROM customers;

-- Select distinct values
SELECT DISTINCT country FROM customers;
```

### Calculated Columns
```sql
SELECT 
    product_name,
    price,
    quantity,
    price * quantity AS total_value
FROM products;
```

---

## Filtering & Conditions

### WHERE Clause
```sql
-- Comparison operators
WHERE age >= 18
WHERE status = 'Active'
WHERE salary BETWEEN 50000 AND 100000
WHERE department IN ('Sales', 'Marketing', 'IT')
WHERE email LIKE '%@gmail.com'
WHERE last_name LIKE 'S%'        -- Starts with S
WHERE last_name LIKE '%son'      -- Ends with son
WHERE last_name LIKE '%an%'      -- Contains an

-- NULL checks
WHERE manager_id IS NULL
WHERE phone IS NOT NULL

-- Logical operators
WHERE age >= 18 AND country = 'USA'
WHERE department = 'Sales' OR department = 'Marketing'
WHERE NOT status = 'Inactive'
```

### CASE Expressions
```sql
SELECT 
    name,
    salary,
    CASE 
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

---

## Aggregate Functions

### Basic Aggregates
```sql
COUNT(*)               -- Count all rows
COUNT(column)          -- Count non-NULL values
COUNT(DISTINCT column) -- Count unique values
SUM(column)            -- Sum of values
AVG(column)            -- Average of values
MIN(column)            -- Minimum value
MAX(column)            -- Maximum value
```

### GROUP BY
```sql
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- Multiple columns
SELECT 
    department,
    job_title,
    COUNT(*) AS count
FROM employees
GROUP BY department, job_title;
```

### HAVING Clause
```sql
SELECT 
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;
```

---

## Joins

### INNER JOIN
```sql
SELECT 
    o.order_id,
    c.customer_name,
    o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
```

### LEFT JOIN
```sql
SELECT 
    c.customer_name,
    o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

### RIGHT JOIN
```sql
SELECT 
    c.customer_name,
    o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

### FULL OUTER JOIN (MySQL Workaround)
```sql
SELECT * FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
UNION
SELECT * FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id;
```

### CROSS JOIN
```sql
SELECT * FROM colors CROSS JOIN sizes;
```

### SELF JOIN
```sql
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

### Multiple Joins
```sql
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id;
```

---

## Subqueries & CTEs

### Scalar Subquery
```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Column Subquery
```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE location = 'New York'
);
```

### Table Subquery
```sql
SELECT dept_name, avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) dept_stats
JOIN departments d ON dept_stats.department_id = d.department_id;
```

### Correlated Subquery
```sql
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

### Common Table Expression (CTE)
```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 80000
)
SELECT department_id, COUNT(*) AS count
FROM high_earners
GROUP BY department_id;
```

### Recursive CTE
```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy;
```

### EXISTS
```sql
SELECT customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

## Window Functions

### Basic Syntax
```sql
function_name() OVER (
    [PARTITION BY partition_columns]
    [ORDER BY order_columns]
    [ROWS or RANGE frame_specification]
)
```

### Ranking Functions
```sql
-- ROW_NUMBER: Unique sequential number
SELECT 
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- RANK: Rank with gaps for ties
SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK: Rank without gaps
SELECT 
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

### Aggregate Window Functions
```sql
-- Running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Moving average
SELECT 
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day
FROM orders;
```

### Partition By
```sql
SELECT 
    department,
    name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
```

### LAG and LEAD
```sql
-- Previous row value
SELECT 
    order_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY order_date) AS prev_amount
FROM orders;

-- Next row value
SELECT 
    order_date,
    amount,
    LEAD(amount, 1) OVER (ORDER BY order_date) AS next_amount
FROM orders;
```

### FIRST_VALUE and LAST_VALUE
```sql
SELECT 
    name,
    salary,
    FIRST_VALUE(salary) OVER (ORDER BY salary DESC) AS highest_salary,
    LAST_VALUE(salary) OVER (
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_salary
FROM employees;
```

---

## Data Modification

### INSERT
```sql
-- Single row
INSERT INTO customers (name, email, country)
VALUES ('John Doe', 'john@example.com', 'USA');

-- Multiple rows
INSERT INTO customers (name, email, country)
VALUES 
    ('Jane Smith', 'jane@example.com', 'UK'),
    ('Bob Johnson', 'bob@example.com', 'Canada');

-- From SELECT
INSERT INTO customers_backup
SELECT * FROM customers WHERE country = 'USA';
```

### UPDATE
```sql
-- Basic update
UPDATE customers
SET email = 'newemail@example.com'
WHERE customer_id = 1;

-- Multiple columns
UPDATE employees
SET salary = salary * 1.1, last_review = NOW()
WHERE department = 'Sales';

-- Update with JOIN
UPDATE orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
SET o.status = 'VIP'
WHERE c.customer_type = 'Premium';
```

### DELETE
```sql
-- Delete specific rows
DELETE FROM orders WHERE order_date < '2023-01-01';

-- Delete with subquery
DELETE FROM customers
WHERE customer_id NOT IN (SELECT DISTINCT customer_id FROM orders);

-- Delete all rows (but keep table)
DELETE FROM temp_table;
```

### REPLACE / INSERT ON DUPLICATE KEY
```sql
-- Replace existing or insert new
REPLACE INTO products (product_id, name, price)
VALUES (1, 'Widget', 29.99);

-- Insert or update on duplicate
INSERT INTO products (product_id, name, price)
VALUES (1, 'Widget', 29.99)
ON DUPLICATE KEY UPDATE price = 29.99;
```

---

## Database Objects

### CREATE TABLE
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE DEFAULT CURRENT_DATE,
    salary DECIMAL(10,2) CHECK (salary >= 0),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

### ALTER TABLE
```sql
-- Add column
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);

-- Modify column
ALTER TABLE employees MODIFY COLUMN phone VARCHAR(25);

-- Drop column
ALTER TABLE employees DROP COLUMN phone;

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary > 0);

-- Add foreign key
ALTER TABLE orders ADD FOREIGN KEY (customer_id) 
    REFERENCES customers(customer_id);
```

### DROP TABLE
```sql
-- Drop table
DROP TABLE temp_table;

-- Drop if exists
DROP TABLE IF EXISTS temp_table;

-- Truncate (faster than DELETE)
TRUNCATE TABLE temp_table;
```

### VIEWS
```sql
-- Create view
CREATE VIEW active_customers AS
SELECT customer_id, name, email
FROM customers
WHERE status = 'Active';

-- Use view
SELECT * FROM active_customers;

-- Drop view
DROP VIEW active_customers;
```

---

## Indexes

### CREATE INDEX
```sql
-- Single column index
CREATE INDEX idx_lastname ON employees(last_name);

-- Composite index
CREATE INDEX idx_name ON employees(last_name, first_name);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Full-text index
CREATE FULLTEXT INDEX idx_description ON products(description);
```

### DROP INDEX
```sql
DROP INDEX idx_lastname ON employees;
```

### SHOW INDEXES
```sql
SHOW INDEXES FROM employees;
```

---

## Transactions

### Basic Transaction
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

### Rollback
```sql
START TRANSACTION;

UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 1;

-- Something goes wrong
ROLLBACK;
```

### Savepoints
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
SAVEPOINT sp1;

UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
SAVEPOINT sp2;

-- Rollback to savepoint
ROLLBACK TO SAVEPOINT sp1;

COMMIT;
```

### Isolation Levels
```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## Stored Procedures & Functions

### Stored Procedure
```sql
DELIMITER //

CREATE PROCEDURE GetEmployeesByDept(IN dept_id INT)
BEGIN
    SELECT * FROM employees WHERE department_id = dept_id;
END //

DELIMITER ;

-- Call procedure
CALL GetEmployeesByDept(1);
```

### Stored Function
```sql
DELIMITER //

CREATE FUNCTION GetFullName(emp_id INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE full_name VARCHAR(100);
    SELECT CONCAT(first_name, ' ', last_name) INTO full_name
    FROM employees WHERE employee_id = emp_id;
    RETURN full_name;
END //

DELIMITER ;

-- Use function
SELECT GetFullName(1);
```

### Variables
```sql
DELIMITER //

CREATE PROCEDURE CalculateBonus(IN emp_id INT, OUT bonus DECIMAL(10,2))
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
    SET bonus = emp_salary * 0.10;
END //

DELIMITER ;
```

### Control Flow
```sql
DELIMITER //

CREATE PROCEDURE CheckSalary(IN emp_id INT)
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
    
    IF emp_salary >= 100000 THEN
        SELECT 'High earner';
    ELSEIF emp_salary >= 50000 THEN
        SELECT 'Medium earner';
    ELSE
        SELECT 'Low earner';
    END IF;
END //

DELIMITER ;
```

---

## Common Functions

### String Functions
```sql
CONCAT(str1, str2, ...)           -- Concatenate strings
SUBSTRING(str, pos, len)          -- Extract substring
UPPER(str)                        -- Convert to uppercase
LOWER(str)                        -- Convert to lowercase
TRIM(str)                         -- Remove leading/trailing spaces
LENGTH(str)                       -- String length
REPLACE(str, from, to)            -- Replace substring
LEFT(str, len)                    -- Leftmost characters
RIGHT(str, len)                   -- Rightmost characters
```

### Numeric Functions
```sql
ROUND(number, decimals)           -- Round number
CEIL(number)                      -- Round up
FLOOR(number)                     -- Round down
ABS(number)                       -- Absolute value
MOD(n, m)                         -- Modulo
POWER(base, exp)                  -- Exponentiation
SQRT(number)                      -- Square root
```

### Date Functions
```sql
NOW()                             -- Current date and time
CURDATE()                         -- Current date
CURTIME()                         -- Current time
DATE(datetime)                    -- Extract date part
YEAR(date)                        -- Extract year
MONTH(date)                       -- Extract month
DAY(date)                         -- Extract day
DATE_ADD(date, INTERVAL value)    -- Add interval
DATE_SUB(date, INTERVAL value)    -- Subtract interval
DATEDIFF(date1, date2)            -- Difference in days
DATE_FORMAT(date, format)         -- Format date
```

### Conditional Functions
```sql
IFNULL(expr, alt)                 -- Return alt if expr is NULL
COALESCE(expr1, expr2, ...)       -- Return first non-NULL
NULLIF(expr1, expr2)              -- Return NULL if equal
IF(condition, true_val, false_val) -- Inline if
```

---

## Best Practices

### Naming Conventions
```sql
-- Use lowercase with underscores
customers, order_items, customer_id

-- Plural for tables, singular for columns
customers (table) -> customer_id (column)

-- Descriptive names
created_at, is_active, total_amount
```

### Query Optimization
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT * FROM orders WHERE customer_id = 1;

-- Create indexes on frequently queried columns
CREATE INDEX idx_customer ON orders(customer_id);

-- Avoid SELECT *
SELECT customer_id, name FROM customers; -- Better

-- Use LIMIT for large result sets
SELECT * FROM orders LIMIT 100;
```

### Security
```sql
-- Never concatenate user input directly
-- Bad: "SELECT * FROM users WHERE id = " + userId
-- Good: Use prepared statements

-- Grant minimum necessary privileges
GRANT SELECT ON database.* TO 'readonly_user'@'localhost';

-- Regularly backup databases
mysqldump -u root -p database_name > backup.sql
```

### Code Style
```sql
-- Use consistent formatting
SELECT 
    customer_id,
    customer_name,
    email
FROM customers
WHERE status = 'Active'
ORDER BY customer_name;

-- Add comments for complex logic
-- Calculate 30-day moving average
SELECT 
    order_date,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS moving_avg
FROM orders;
```

---

## Common Patterns

### Top N per Group
```sql
WITH ranked_products AS (
    SELECT 
        category,
        product_name,
        sales,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM products
)
SELECT category, product_name, sales
FROM ranked_products
WHERE rn <= 3;
```

### Running Total
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

### Gap Detection
```sql
SELECT 
    t1.id + 1 AS gap_start,
    MIN(t2.id) - 1 AS gap_end
FROM table t1
LEFT JOIN table t2 ON t1.id >= t2.id - 1
GROUP BY t1.id
HAVING gap_start < MIN(t2.id);
```

### Pivoting Data
```sql
SELECT 
    customer_id,
    SUM(CASE WHEN YEAR(order_date) = 2023 THEN amount ELSE 0 END) AS sales_2023,
    SUM(CASE WHEN YEAR(order_date) = 2024 THEN amount ELSE 0 END) AS sales_2024
FROM orders
GROUP BY customer_id;
```

---

## Error Codes

Common MySQL error codes:
- **1062**: Duplicate entry for unique key
- **1064**: SQL syntax error
- **1146**: Table doesn't exist
- **1054**: Unknown column
- **1452**: Foreign key constraint fails
- **1364**: Field doesn't have a default value

---

## Useful Commands

```sql
-- Show current database
SELECT DATABASE();

-- Show all tables
SHOW TABLES;

-- Describe table structure
DESCRIBE table_name;

-- Show create table statement
SHOW CREATE TABLE table_name;

-- Show table status
SHOW TABLE STATUS;

-- Show processlist
SHOW PROCESSLIST;

-- Kill query
KILL QUERY process_id;
```

---

**For more detailed explanations, refer to the specific module in the [lessons](lessons/) folder.**

---

*Quick Reference Guide - SQL Training Curriculum*
