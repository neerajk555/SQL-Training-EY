# Module 10: Database Objects — DDL and Schema Design (MySQL)

## 1) Introduction

Scenario: Your startup’s data keeps growing. You need tables that enforce data quality, relationships that prevent orphaned records, and names people understand. Good schema design saves time and avoids bugs.

Definition (plain):
- DDL (Data Definition Language): SQL statements that create/alter/drop objects (tables, views, indexes, constraints).
- Schema/database: Container for related objects.
- Constraint: Rule the database enforces (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL).

Connection to previous/next:
- You’ve been querying existing tables; now you’ll design them. Next modules cover indexing/optimization and transactions.

Learning objectives:
- Create tables with appropriate data types and constraints.
- Model relationships with PK/FK and choose cardinalities.
- Understand normalization vs denormalization trade-offs.
- Use views for reusable query logic.

## 2) Concept explanation

Core objects:
- Tables: Store data in rows/columns.
- Constraints: PK, FK, UNIQUE, CHECK, NOT NULL, DEFAULT.
- Views: Named, stored queries that present data like a table.
- Indexes: Structures that accelerate data access (Next module covers details).

Example DDL syntax:
```sql
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL CHECK (total_amount >= 0),
  CONSTRAINT fk_orders_customers
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

Notes:
- MySQL CHECK constraints are parsed but enforced only in recent versions; verify your version (8.0.16+ starts enforcement for InnoDB). Historically, CHECK was ignored.
- Use NOT NULL for required fields and sensible DEFAULTS when appropriate.

Normalization vs denormalization:
- Normalization: Reduce redundancy via separate tables and FKs (1NF, 2NF, 3NF). Pros: integrity, smaller updates; Cons: more joins.
- Denormalization: Duplicate some data for performance. Pros: speed; Cons: potential inconsistency.

Visual (ASCII) e-commerce schema:

customers (1) ───< orders (many) ───< order_items (many)
                         │
                         └──> payments (many)

products (1) ───< order_items (many)

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS order_items, payments, orders, products, customers;

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100) NOT NULL,
  category VARCHAR(50) NOT NULL,
  price DECIMAL(10,2) NOT NULL CHECK (price >= 0)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL CHECK (total_amount >= 0),
  CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  quantity INT NOT NULL CHECK (quantity > 0),
  unit_price DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (order_id, product_id),
  CONSTRAINT fk_items_order FOREIGN KEY (order_id)
    REFERENCES orders(order_id),
  CONSTRAINT fk_items_product FOREIGN KEY (product_id)
    REFERENCES products(product_id)
);

CREATE TABLE payments (
  payment_id INT PRIMARY KEY,
  order_id INT NOT NULL,
  payment_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  amount DECIMAL(10,2) NOT NULL,
  method ENUM('card','cash','wallet') NOT NULL,
  CONSTRAINT fk_payments_order FOREIGN KEY (order_id)
    REFERENCES orders(order_id)
);
```

### Example A — Insert sample data and verify constraints
```sql
INSERT INTO customers VALUES
 (1,'Sara Jones','sara@example.com', NOW()),
 (2,'Leo Park','leo@shop.com', NOW());

INSERT INTO products VALUES
 (1,'Wireless Mouse','Electronics',25.00),
 (2,'Keyboard','Electronics',85.00);

INSERT INTO orders VALUES
 (9001,1,'2025-01-05',120.00),
 (9002,2,'2025-01-06',55.50);

INSERT INTO order_items VALUES
 (9001,1,2,25.00),
 (9001,2,1,85.00),
 (9002,2,1,55.50);

INSERT INTO payments VALUES
 (1,9001,NOW(),120.00,'card'),
 (2,9002,NOW(),55.50,'wallet');
```

Try a violating insert (should fail):
```sql
-- Order referencing a non-existent customer
INSERT INTO orders VALUES (9003,999,'2025-01-10',10.00);
```

### Example B — View for common reporting
- Business question: “Create a reusable view of order summaries with customer names.”

SQL:
```sql
CREATE OR REPLACE VIEW v_order_summary AS
SELECT 
  o.order_id,
  c.customer_name,
  o.order_date,
  o.total_amount
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id;

SELECT * FROM v_order_summary ORDER BY order_id;
```

### Example C — Evolving schema: ALTER TABLE
- Business question: “Add a status to orders and default to 'placed'.”

SQL:
```sql
ALTER TABLE orders
ADD COLUMN status ENUM('placed','shipped','delivered','cancelled') NOT NULL DEFAULT 'placed';

-- Update existing orders
UPDATE orders SET status = 'delivered' WHERE order_id = 9001;
```

## 4) Common patterns and best practices

Use cases:
1) Enforce integrity with PK/FK/UNIQUE/NOT NULL.
2) Use composite PKs for line-item tables.
3) Create views for common joins/filters.
4) Add audit columns (created_at, updated_at).

Templates:
```sql
-- Audit columns
ALTER TABLE customers
  ADD COLUMN updated_at DATETIME NULL,
  ADD COLUMN updated_by VARCHAR(100) NULL;

-- Unique composite constraint
ALTER TABLE products ADD CONSTRAINT uq_name_cat UNIQUE (product_name, category);
```

Performance notes:
- Choose data types wisely (DECIMAL for currency, appropriate string lengths).
- Index FK columns and frequent filter columns.
- Views don’t store data (in MySQL)—they are saved queries; performance depends on underlying tables and indexes.

Do’s and Don’ts:
- Do: Name constraints (fk_..., uq_...) for clarity.
- Do: Use NOT NULL when appropriate.
- Don’t: Overuse ENUM if portability matters.
- Don’t: Store multiple values in one column (violates normalization).

Troubleshooting:
- “Cannot add or update a child row: a foreign key constraint fails” → Insert parent first; check FK values.
- “Duplicate entry” on UNIQUE → Check data quality or adjust uniqueness rules.
- “CHECK constraint ignored” → Verify MySQL version and storage engine.

## 5) Practice exercises

### Exercise 1: Create a categories table and link products
- Difficulty: Beginner
- Task: Create `categories(category_id PK, category_name UNIQUE)` and add FK from `products(category)` to `categories(category_name)` or refactor products to store `category_id` (preferred).
- Solution (preferred refactor sketch):
```sql
ALTER TABLE products DROP COLUMN category;
CREATE TABLE categories (
  category_id INT PRIMARY KEY,
  category_name VARCHAR(50) UNIQUE NOT NULL
);
ALTER TABLE products ADD COLUMN category_id INT NOT NULL;
ALTER TABLE products ADD CONSTRAINT fk_prod_cat
  FOREIGN KEY (category_id) REFERENCES categories(category_id);
```

### Exercise 2: Add NOT NULL + DEFAULTs
- Difficulty: Beginner
- Task: Ensure `order_items.unit_price` is NOT NULL with a sensible default.
- Solution:
```sql
ALTER TABLE order_items
MODIFY unit_price DECIMAL(10,2) NOT NULL;
```

### Exercise 3: Composite primary key
- Difficulty: Intermediate
- Task: Add PK (order_id, product_id) to `order_items` if missing.
- Solution: Already present in setup; practice writing it.

### Exercise 4: Create a reporting view
- Difficulty: Intermediate
- Task: Create a view `v_daily_rev` with `sale_date` and `SUM(total_amount)` from orders.
- Solution:
```sql
CREATE OR REPLACE VIEW v_daily_rev AS
SELECT order_date AS sale_date, SUM(total_amount) AS revenue
FROM orders
GROUP BY order_date;
```

### Exercise 5: Enforce email uniqueness and length
- Difficulty: Advanced
- Task: Ensure `customers.email` is UNIQUE, non-empty, and at most 255 chars.
- Solution: Unique + NOT NULL + application-level checks; CHECK can enforce length in newer MySQL.
```sql
ALTER TABLE customers
MODIFY email VARCHAR(255) NOT NULL,
ADD UNIQUE KEY uq_customers_email (email);
```

## 6) Self-assessment checklist
- Can you create tables with PK/FK/UNIQUE/NOT NULL? (Ex. 1–2)
- Can you explain normalization and when to denormalize?
- Can you create views to encapsulate common joins? (Ex. 4)
- Can you evolve schemas safely with ALTER TABLE? (Ex. 2/3)

## 7) Connection to real work
- Scenarios: Launching new features (new tables), enforcing data quality, building analytics views.
- Roles: Data Engineer, DB Developer, Platform Engineer.
- Interview questions:
  - Q: Why prefer INT PKs over natural keys?
    - A: Stability, performance, and flexibility if natural keys change.
  - Q: Pros/cons of denormalization?
    - A: Pros: speed, fewer joins; Cons: duplication, potential inconsistency.
  - Q: When to use composite keys?
    - A: Line-item tables where the natural identity is parent_id + child_id.
