# Module 14: Triggers and Event-Driven Logic (MySQL)

## 1) Introduction

Scenario: You want to audit changes, keep derived totals in sync, or enforce complex rules automatically when data changes. Triggers run on events like INSERT/UPDATE/DELETE.

Definition (plain):
- Trigger: A stored program that executes automatically in response to table events.
- Timing: BEFORE or AFTER the event.
- Event: INSERT, UPDATE, DELETE.

Connection to previous/next:
- Complements procedures/functions and transactions. Next, advanced programming concepts and professional practices.

Learning objectives:
- Create BEFORE/AFTER triggers using NEW/OLD values.
- Implement audit logs and derived columns.
- Understand performance and recursion risks.

## 2) Concept explanation

Syntax:
```sql
DELIMITER $$
CREATE TRIGGER trg_name
BEFORE|AFTER INSERT|UPDATE|DELETE ON table_name
FOR EACH ROW
BEGIN
  -- use NEW.col (insert/update) and OLD.col (update/delete)
END $$
DELIMITER ;
```

Common use cases:
- Audit trails: Record who changed what, when.
- Derived/calculated columns: Update denormalized totals.
- Data validation: Enforce rules not captured by constraints.

Cautions:
- Triggers can hide business logic and make debugging harder.
- Bulk operations may fire many triggers; watch performance.
- Avoid recursive writes (trigger updating same table causing loops).

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS orders, order_items, order_totals, audit_log;

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL
);

CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  quantity INT NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL
);

CREATE TABLE order_totals (
  order_id INT PRIMARY KEY,
  total_amount DECIMAL(10,2) NOT NULL DEFAULT 0.00
);

CREATE TABLE audit_log (
  id INT AUTO_INCREMENT PRIMARY KEY,
  table_name VARCHAR(50),
  action VARCHAR(10),
  row_id VARCHAR(50),
  changed_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Example A — Maintain order total on item changes

Trigger on INSERT to order_items:
```sql
DELIMITER $$
CREATE TRIGGER trg_items_ins
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
  INSERT INTO order_totals (order_id, total_amount)
  VALUES (NEW.order_id, NEW.quantity * NEW.unit_price)
  ON DUPLICATE KEY UPDATE
    total_amount = total_amount + (NEW.quantity * NEW.unit_price);
END $$
DELIMITER ;
```

Trigger on DELETE:
```sql
DELIMITER $$
CREATE TRIGGER trg_items_del
AFTER DELETE ON order_items
FOR EACH ROW
BEGIN
  UPDATE order_totals
  SET total_amount = GREATEST(total_amount - (OLD.quantity * OLD.unit_price), 0)
  WHERE order_id = OLD.order_id;
END $$
DELIMITER ;
```

### Example B — Audit updates to orders

```sql
DELIMITER $$
CREATE TRIGGER trg_orders_upd
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
  INSERT INTO audit_log (table_name, action, row_id)
  VALUES ('orders', 'UPDATE', CONCAT('order_id=', NEW.order_id));
END $$
DELIMITER ;
```

Test data:
```sql
INSERT INTO orders VALUES (1,101,'2025-01-01');
INSERT INTO order_items VALUES (1,1,2,25.00), (1,2,1,85.00);
SELECT * FROM order_totals; -- should reflect 135.00

UPDATE orders SET order_date = '2025-01-02' WHERE order_id = 1;
SELECT * FROM audit_log;
```

## 4) Common patterns and best practices

Use cases:
1) Keep summary tables up-to-date on row changes.
2) Capture audit trails for compliance.
3) Validate domain-specific rules (with caution).

Templates:
```sql
-- BEFORE INSERT validation (simple)
DELIMITER $$
CREATE TRIGGER trg_products_bi
BEFORE INSERT ON products
FOR EACH ROW
BEGIN
  IF NEW.price < 0 THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Price cannot be negative';
  END IF;
END $$
DELIMITER ;
```

Performance notes:
- Keep trigger logic short; move heavy work to async processes if possible.
- Index keys used in trigger updates.

Do’s and Don’ts:
- Do: Document triggers thoroughly.
- Do: Use AFTER triggers for logging and totals; BEFORE for validation.
- Don’t: Create triggers that update the same table without safeguards.
- Don’t: Put complex business flows entirely in triggers.

Troubleshooting:
- “Can’t update table used in trigger” → Avoid recursive updates; use a different table.
- Silent failures → Use `SIGNAL` to raise explicit errors.

## 5) Practice exercises

### Exercise 1: Total maintenance on UPDATE
- Difficulty: Beginner
- Task: Add an AFTER UPDATE trigger on `order_items` to adjust totals when quantity or unit_price changes.
- Solution sketch:
```sql
DELIMITER $$
CREATE TRIGGER trg_items_upd
AFTER UPDATE ON order_items
FOR EACH ROW
BEGIN
  UPDATE order_totals
  SET total_amount = total_amount
                    - (OLD.quantity * OLD.unit_price)
                    + (NEW.quantity * NEW.unit_price)
  WHERE order_id = NEW.order_id;
END $$
DELIMITER ;
```

### Exercise 2: Audit deletes
- Difficulty: Beginner
- Task: Write an AFTER DELETE trigger on `orders` to log deletion.

### Exercise 3: Validation before insert
- Difficulty: Intermediate
- Task: Prevent inserting order_items with quantity <= 0 using BEFORE INSERT + SIGNAL.

### Exercise 4: Prevent recursive updates
- Difficulty: Intermediate
- Task: Discuss why triggers should not update their own table and propose alternatives.

### Exercise 5: Consolidated audit trigger
- Difficulty: Advanced
- Task: Create a generic audit trigger for `products` capturing insert/update/delete with action and primary key.

## 6) Self-assessment checklist
- Can you write BEFORE/AFTER triggers using NEW/OLD? (Ex. 1–3)
- Can you maintain summary tables via triggers? (Ex. 1)
- Can you raise errors using SIGNAL? (Ex. 3)
- Can you explain trigger performance and debugging considerations?

## 7) Connection to real work
- Scenarios: Compliance audit logs, denormalized total maintenance, data validations.
- Roles: DB Developer, Data Engineer, Platform Engineer.
- Interview questions:
  - Q: When choose BEFORE vs AFTER triggers?
    - A: BEFORE for validation/modification before write; AFTER for actions requiring the final state.
  - Q: Risks of heavy trigger logic?
    - A: Slows writes, hard to debug, risk of recursion or lock contention.
  - Q: How to test triggers safely?
    - A: Use isolated test data and transactions; verify side effects and roll back.
