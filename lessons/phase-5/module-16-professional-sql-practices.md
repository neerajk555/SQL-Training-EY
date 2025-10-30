# Module 16: Professional SQL Practices

## 1) Introduction

Scenario: Your team collaborates on analytics, apps, and pipelines. Consistent SQL practices, safe migrations, and security controls keep systems reliable and maintainable.

Definition (plain):
- Style guide: Conventions for naming, formatting, and commenting.
- Migration: Version-controlled changes to database schema and reference data.
- Least privilege: Grant only needed permissions.

Connection to previous modules:
- Pulls together querying, DDL, indexing, transactions, and programming features into team-ready workflows.

Learning objectives:
- Apply consistent naming/formatting/commenting.
- Use migrations (Flyway/Liquibase) to evolve schemas safely.
- Implement access control and guard against SQL injection.
- Document, test, and review SQL changes.

## 2) Concept explanation

Style conventions (suggested):
- Names: snake_case for tables/columns; singular table names for dimensions (product), plural for facts (orders)—pick a pattern and stick to it.
- Keys: `table_id` for PKs; FK as `other_table_id`.
- Formatting: Keywords UPPERCASE; align clauses on new lines; one concept per line.
- Comments: `-- why` and `/* longer explanation */` for complex logic.

Migrations:
- Tooling: Flyway, Liquibase, or in-house migration scripts.
- Process: Each change is a migration file (DDL/DML) with an ID, applied sequentially to each environment (dev → test → prod).
- Rollbacks: Provide down scripts when possible; or plan forward fixes.

Security & privacy:
- Accounts/roles: Separate app users from admin; grant minimal privileges.
- Avoid SQL injection: Use parameterized queries in apps; avoid dynamic SQL with untrusted inputs.
- PII: Mask in non-prod; encrypt or tokenize sensitive fields.

Testing & reviews:
- Unit tests for stored routines; expected result checks for queries.
- Peer review of complex SQL with EXPLAIN plans and sample data.
- Monitoring: Track slow queries and error logs.

Backups & recovery:
- Back up regularly; test restores.
- Use point-in-time recovery with binary logs when needed.

## 3) Detailed examples

### Example A — A readable query style
```sql
-- Goal: Top 5 products by revenue this month
SELECT 
  oi.product_id,
  SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items AS oi
JOIN orders AS o ON o.order_id = oi.order_id
WHERE o.order_date >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
  AND o.order_date <  DATE_FORMAT(CURDATE() + INTERVAL 1 MONTH, '%Y-%m-01')
GROUP BY oi.product_id
ORDER BY revenue DESC
LIMIT 5;
```

### Example B — Migration scripts (Flyway-style)
- V1__create_core_tables.sql
```sql
CREATE TABLE customers (...);
CREATE TABLE orders (...);
```
- V2__add_indexes.sql
```sql
CREATE INDEX idx_orders_date ON orders (order_date);
```
- V3__denorm_total.sql
```sql
ALTER TABLE orders ADD COLUMN total_amount DECIMAL(10,2) NOT NULL DEFAULT 0.00;
```

### Example C — Role-based grants (sketch)
```sql
CREATE USER 'reporter'@'%' IDENTIFIED BY 'strong_password';
GRANT SELECT ON mydb.* TO 'reporter'@'%';
```

## 4) Common patterns and best practices

Use cases:
1) Team-wide query/readability standards.
2) Controlled schema evolution via migrations.
3) Secure access for apps, analysts, and admins.
4) Reproducible envs and data refresh processes.

Templates:
```sql
-- Comment header for complex scripts
/* Purpose: Daily revenue aggregation
   Owner: data-team
   Run: nightly */

-- Roll-forward migration
START TRANSACTION;
-- DDL/DML changes here
COMMIT;
```

Performance notes:
- Review EXPLAIN and test with realistic row counts.
- Add indexes alongside schema changes when needed.

Do’s and Don’ts:
- Do: Keep SQL in version control.
- Do: Review and test queries before production.
- Don’t: Run ad-hoc DDL in prod shells without a migration record.
- Don’t: Grant broad privileges to app/service accounts.

Troubleshooting:
- Drift between environments → Use migration tools and baseline snapshots.
- Inconsistent naming → Enforce style in code reviews.

## 5) Practice exercises

### Exercise 1: Style refactor
- Difficulty: Beginner
- Task: Reformat a messy query to the house style (keywords, indentation, comments).

### Exercise 2: Migration creation
- Difficulty: Beginner
- Task: Write V1__create_reporting_tables.sql to create `daily_revenue(date PK, revenue)` and V2__populate_initial.sql to backfill from `orders`.

### Exercise 3: Least-privilege role
- Difficulty: Intermediate
- Task: Create a read-only role/user for analysts and grant SELECT to reporting schema only.

### Exercise 4: Injection-safe access
- Difficulty: Intermediate
- Task: Show how an application should parameterize `WHERE email = ?` queries; explain why dynamic SQL is unsafe.

### Exercise 5: Backup/restore drill
- Difficulty: Advanced
- Task: Document steps to back up the database and restore to a point in time.

## 6) Self-assessment checklist
- Can you apply a consistent SQL style? (Ex. 1)
- Can you write and organize migrations? (Ex. 2)
- Can you configure least-privilege access? (Ex. 3)
- Can you explain and avoid SQL injection?
- Can you outline backup/restore procedures?

## 7) Connection to real work
- Scenarios: Reliable releases, secure analytics, disaster recovery.
- Roles: DBA, Data/Platform Engineer, Backend Lead.
- Interview questions:
  - Q: Why store database changes in version control?
    - A: Traceability, reproducibility, and consistent environments.
  - Q: What does least privilege mean in practice?
    - A: Users get only the permissions they need—reduces blast radius.
  - Q: How do you ensure SQL quality in a team?
    - A: Style guides, reviews, tests, and monitoring.
