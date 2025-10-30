# Module 13: Stored Procedures and Functions (MySQL)

## 1) Introduction

Scenario: You need to encapsulate multi-step logic (e.g., creating an order with validations) or reusable calculations (e.g., price after tax). Stored procedures and functions let you move logic closer to the data.

Definition (plain):
- Stored procedure: A saved routine that executes one or more SQL statements; may have IN/OUT parameters, doesn’t return a scalar value directly.
- Stored function: A routine that returns a single value and can be used in SQL expressions.

Connection to previous/next:
- Builds on DML and transactions. Next: Triggers and event-driven logic.

Learning objectives:
- Create procedures with parameters and control flow.
- Create functions for reusable expressions.
- Handle errors and use transactions inside procedures.

## 2) Concept explanation

Stored procedure syntax (simplified):
```sql
DELIMITER $$
CREATE PROCEDURE proc_name(IN p_in INT, OUT p_out INT)
BEGIN
  -- statements
  SET p_out = p_in + 1;
END $$
DELIMITER ;
```

Stored function syntax:
```sql
DELIMITER $$
CREATE FUNCTION fn_name(p_val DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  RETURN ROUND(p_val * 1.08, 2);
END $$
DELIMITER ;
```

Notes:
- Change DELIMITER in clients to define routines with internal semicolons.
- Functions must not modify data (by convention and restrictions); procedures can.
- Mark functions DETERMINISTIC/NON-DETERMINISTIC for optimizer and replication.

Common misconceptions:
- “Functions can run DML freely.” In MySQL, functions are limited; use procedures for data changes.
- “DELIMITER is SQL.” It’s a client command (e.g., MySQL CLI) to help parse multi-statement routines.

## 3) Detailed examples

Setup:
```sql
CREATE DATABASE IF NOT EXISTS sql_training;
USE sql_training;

DROP TABLE IF EXISTS taxes;
CREATE TABLE taxes (
  region VARCHAR(10) PRIMARY KEY,
  rate DECIMAL(5,4) NOT NULL
);
INSERT INTO taxes VALUES ('US',0.0750),('IN',0.1800),('GB',0.2000);
```

### Example A — A simple stored function
- Business question: “Return a tax-included price at 8%.”

SQL:
```sql
DELIMITER $$
CREATE OR REPLACE FUNCTION fn_price_with_tax(p_price DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  RETURN ROUND(p_price * 1.08, 2);
END $$
DELIMITER ;

SELECT fn_price_with_tax(100.00) AS example; -- 108.00
```

### Example B — Procedure with IN/OUT and transaction
- Business question: “Adjust tax rate atomically; return old rate.”

SQL:
```sql
DELIMITER $$
CREATE OR REPLACE PROCEDURE sp_set_tax(IN p_region VARCHAR(10), IN p_rate DECIMAL(5,4), OUT p_old DECIMAL(5,4))
BEGIN
  DECLARE v_old DECIMAL(5,4);
  START TRANSACTION;
  SELECT rate INTO v_old FROM taxes WHERE region = p_region FOR UPDATE;
  UPDATE taxes SET rate = p_rate WHERE region = p_region;
  COMMIT;
  SET p_old = v_old;
END $$
DELIMITER ;

-- Call it
SET @old := NULL;
CALL sp_set_tax('US', 0.0800, @old);
SELECT @old AS previous_rate;
```

### Example C — Function with lookup
- Business question: “Compute price with region-specific tax.”

SQL:
```sql
DELIMITER $$
CREATE OR REPLACE FUNCTION fn_price_with_region_tax(p_price DECIMAL(10,2), p_region VARCHAR(10))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  DECLARE v_rate DECIMAL(5,4);
  SELECT rate INTO v_rate FROM taxes WHERE region = p_region;
  RETURN ROUND(p_price * (1 + v_rate), 2);
END $$
DELIMITER ;

SELECT fn_price_with_region_tax(100.00, 'IN') AS price_in_inr_tax; -- 118.00
```

## 4) Common patterns and best practices

Use cases:
1) Encapsulate multi-statement operations (procedures).
2) Reusable calculations (functions).
3) Parameterized routines for application APIs.

Templates:
```sql
-- Error handling sketch (MySQL handler)
DELIMITER $$
CREATE PROCEDURE sp_example()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    ROLLBACK;
  END;
  START TRANSACTION;
  -- do stuff
  COMMIT;
END $$
DELIMITER ;
```

Performance notes:
- Don’t overuse procedural code for tasks better done with set-based SQL.
- Functions in WHERE can prevent index usage; consider computed columns.

Do’s and Don’ts:
- Do: Keep business logic cohesive and testable.
- Do: Use transactions in procedures that modify multiple tables.
- Don’t: Put heavy loops in SQL; prefer set operations.
- Don’t: Return gigantic result sets from procedures—paginate.

Troubleshooting:
- “This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA” → Add appropriate characteristics.
- “You have an error in your SQL syntax” → Check DELIMITER usage and routine body.

## 5) Practice exercises

### Exercise 1: Simple function
- Difficulty: Beginner
- Task: Create `fn_discounted_price(p_price, p_pct)` returning rounded price after discount.
- Solution:
```sql
DELIMITER $$
CREATE FUNCTION fn_discounted_price(p_price DECIMAL(10,2), p_pct DECIMAL(5,2))
RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
  RETURN ROUND(p_price * (1 - p_pct/100), 2);
END $$
DELIMITER ;
```

### Exercise 2: Procedure to set tax
- Difficulty: Beginner
- Task: Write `sp_set_tax` as in Example B and return old rate via OUT param.

### Exercise 3: Region-aware function
- Difficulty: Intermediate
- Task: Write `fn_price_with_region_tax` using the taxes table.

### Exercise 4: Safe update with handler
- Difficulty: Intermediate
- Task: In a procedure, wrap an UPDATE in a transaction and add an EXIT HANDLER to ROLLBACK on error.

### Exercise 5: Validation inside procedure
- Difficulty: Advanced
- Task: Procedure `sp_adjust_balance(p_acct, p_delta)` that ensures balance never < 0; ROLLBACK on violation.

## 6) Self-assessment checklist
- Can you create functions with DETERMINISTIC/RETURNS? (Ex. 1/3)
- Can you write procedures with IN/OUT parameters and transactions? (Ex. 2/4)
- Can you use handlers for errors?

## 7) Connection to real work
- Scenarios: Encapsulate billing logic, tax calculations, batch operations.
- Roles: DB Developer, Backend Engineer, Data Engineer.
- Interview questions:
  - Q: When should you prefer a function vs a procedure?
    - A: Function returns a scalar used in expressions; procedure performs actions and may produce result sets.
  - Q: Why mark a function DETERMINISTIC?
    - A: Helps optimizer/replication know results depend only on inputs.
  - Q: How do you handle errors in procedures?
    - A: Use handlers, and wrap steps in START TRANSACTION/COMMIT with ROLLBACK on failure.
