# SQL Training - Complete Learning Plan

## 📚 Overview

This comprehensive SQL training program is designed to take you from beginner to advanced SQL practitioner in approximately **60-80 hours** of focused study. Each module builds upon previous concepts, creating a structured learning path that mirrors real-world database work.

---

## 🎯 Learning Path Strategy

### How to Use This Plan

1. **Sequential Learning**: Complete modules in order - each builds on previous knowledge
2. **Time Management**: Each module includes estimated completion time
3. **Hands-On Practice**: Aim for 70% practice, 30% theory
4. **Self-Assessment**: Use checklists to verify readiness before advancing
5. **Review Cycles**: Revisit earlier modules when referenced in later ones

### Success Metrics

- ✅ Complete all practice exercises for each module
- ✅ Pass self-assessment checklists with 80%+ confidence
- ✅ Build a portfolio of working queries
- ✅ Understand both "how" and "why" for each concept

---

## 📊 Phase-by-Phase Breakdown

### **PHASE 1: Foundations** (12-16 hours)
*Goal: Build core SQL literacy and comfort with basic queries*

#### Prerequisites
- Computer with MySQL 8.x installed
- Basic understanding of spreadsheets/tables
- Willingness to experiment and make mistakes

#### Learning Outcomes
By the end of Phase 1, you will:
- Navigate database management systems confidently
- Write SELECT queries with filtering and sorting
- Work with common data types and basic functions
- Understand NULL handling and data type conversion
- Read and interpret simple database schemas

---

#### **Module 1: Introduction to Databases and SQL**
**Time Estimate:** 3-4 hours  
**Difficulty:** ⭐ Beginner

**What You'll Learn:**
- Database fundamentals (tables, rows, columns, keys)
- SQL categories (DDL, DML, DQL, DCL, TCL)
- Setting up MySQL environment
- Writing your first queries
- Basic database terminology

**Key Skills:**
- CREATE DATABASE and USE statements
- Understanding schema design basics
- Reading table structures
- Avoiding common syntax errors

**Prerequisites:** None

**Connects To:**
- All future modules depend on these foundations
- Essential for Module 2 (SELECT fundamentals)

---

#### **Module 2: SELECT Fundamentals**
**Time Estimate:** 4-5 hours  
**Difficulty:** ⭐ Beginner

**What You'll Learn:**
- SELECT statement anatomy
- WHERE clause filtering
- ORDER BY sorting
- LIMIT and OFFSET for pagination
- Basic operators (=, !=, <, >, <=, >=, LIKE, IN, BETWEEN)

**Key Skills:**
- Filtering data with precision
- Sorting results logically
- Combining multiple conditions (AND, OR, NOT)
- Pattern matching with LIKE and wildcards
- Handling NULL values in conditions

**Prerequisites:** Module 1

**Connects To:**
- Module 3: Functions will enhance your SELECT queries
- Module 4: Aggregates build on SELECT basics
- Module 5: JOINs extend SELECT to multiple tables

---

#### **Module 3: Working with Data Types and Basic Functions**
**Time Estimate:** 5-7 hours  
**Difficulty:** ⭐⭐ Beginner-Intermediate

**What You'll Learn:**
- MySQL data types (INT, VARCHAR, DATE, DECIMAL, etc.)
- String functions (CONCAT, SUBSTRING, UPPER, LOWER, TRIM)
- Numeric functions (ROUND, CEIL, FLOOR, ABS, MOD)
- Date/time functions (NOW, CURDATE, DATE_ADD, DATEDIFF)
- Type conversion (CAST, CONVERT)

**Key Skills:**
- Choosing appropriate data types
- Manipulating text data
- Performing date calculations
- Formatting output for reporting
- Handling NULL values with COALESCE and IFNULL

**Prerequisites:** Modules 1-2

**Connects To:**
- Module 4: Functions used in aggregations
- Module 8: Advanced functions in window functions
- Module 13: Functions in stored procedures

---

### **PHASE 2: Intermediate SQL Skills** (15-20 hours)
*Goal: Master data analysis and multi-table operations*

#### Prerequisites
- Completed Phase 1
- Comfort with basic SELECT queries
- Understanding of data types and functions

#### Learning Outcomes
By the end of Phase 2, you will:
- Summarize data with aggregate functions
- Combine data from multiple tables using JOINs
- Write subqueries and Common Table Expressions (CTEs)
- Analyze grouped data effectively
- Understand database relationships (1:1, 1:N, N:M)

---

#### **Module 4: Aggregate Functions and Grouping**
**Time Estimate:** 4-5 hours  
**Difficulty:** ⭐⭐ Intermediate

**What You'll Learn:**
- Aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- GROUP BY clause mechanics
- HAVING clause for filtered aggregations
- DISTINCT keyword usage
- Grouping by multiple columns

**Key Skills:**
- Calculating business metrics (totals, averages, counts)
- Filtering aggregated results
- Understanding aggregation levels
- Avoiding GROUP BY pitfalls

**Prerequisites:** Modules 1-3

**Connects To:**
- Module 5: Aggregates with JOINs
- Module 8: Aggregates vs window functions
- Module 11: Performance optimization for aggregations

---

#### **Module 5: Joins and Relationships**
**Time Estimate:** 6-8 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- Relationship types (1:1, 1:N, N:M)
- INNER JOIN mechanics
- LEFT/RIGHT/FULL OUTER JOINs
- CROSS JOIN and SELF JOIN
- JOIN strategies and best practices

**Key Skills:**
- Connecting related tables
- Choosing the right JOIN type
- Handling NULL values in JOINs
- Multi-table queries (3+ tables)
- Join order optimization

**Prerequisites:** Modules 1-4

**Connects To:**
- Module 6: Subqueries as JOIN alternatives
- Module 10: Schema design for efficient JOINs
- Module 11: Index design for JOIN performance

---

#### **Module 6: Subqueries and CTEs**
**Time Estimate:** 5-7 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- Subquery types (scalar, column, table)
- Subquery locations (SELECT, WHERE, FROM, HAVING)
- Correlated vs non-correlated subqueries
- Common Table Expressions (CTEs)
- Recursive CTEs

**Key Skills:**
- Breaking complex queries into steps
- Improving query readability
- Solving hierarchical problems
- When to use subqueries vs JOINs
- Performance considerations

**Prerequisites:** Modules 1-5

**Connects To:**
- Module 8: CTEs with window functions
- Module 13: CTEs in stored procedures
- Module 15: Advanced CTE patterns

---

### **PHASE 3: Advanced Query Techniques** (12-16 hours)
*Goal: Master complex data manipulation and analytical queries*

#### Prerequisites
- Completed Phases 1-2
- Strong understanding of JOINs and subqueries
- Comfort with aggregate functions

#### Learning Outcomes
By the end of Phase 3, you will:
- Perform set operations (UNION, INTERSECT, EXCEPT)
- Use advanced filtering techniques
- Write analytical queries with window functions
- Perform INSERT, UPDATE, DELETE, and MERGE operations
- Handle complex data modifications safely

---

#### **Module 7: Set Operations and Advanced Filtering**
**Time Estimate:** 3-4 hours  
**Difficulty:** ⭐⭐ Intermediate

**What You'll Learn:**
- UNION, UNION ALL
- INTERSECT and EXCEPT (MySQL workarounds)
- Advanced WHERE techniques (EXISTS, NOT EXISTS)
- IN vs EXISTS performance
- Complex filtering patterns

**Key Skills:**
- Combining result sets
- Finding differences between datasets
- Optimizing existence checks
- Advanced pattern matching
- Multi-condition filtering

**Prerequisites:** Modules 1-6

**Connects To:**
- Module 9: Set operations in data modifications
- Module 11: Performance tuning for set operations

---

#### **Module 8: Window Functions and Analytics**
**Time Estimate:** 5-7 hours  
**Difficulty:** ⭐⭐⭐⭐ Advanced

**What You'll Learn:**
- Window function syntax (OVER clause)
- Ranking functions (ROW_NUMBER, RANK, DENSE_RANK)
- Aggregate window functions
- PARTITION BY and ORDER BY in windows
- Frame specifications (ROWS, RANGE)
- Lead, Lag, First_Value, Last_Value

**Key Skills:**
- Running totals and moving averages
- Ranking and top-N queries
- Period-over-period comparisons
- Advanced analytics
- Avoiding self-joins with windows

**Prerequisites:** Modules 1-6

**Connects To:**
- Module 15: Advanced analytical patterns
- Module 16: Reporting and dashboards

---

#### **Module 9: Data Modification - DML Operations**
**Time Estimate:** 4-5 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- INSERT statements (single, multiple, from SELECT)
- UPDATE with complex conditions
- DELETE with safety practices
- REPLACE and INSERT...ON DUPLICATE KEY UPDATE
- Bulk operations and performance

**Key Skills:**
- Safe data modifications
- Bulk loading techniques
- Conditional updates
- Cascading changes
- Rollback strategies

**Prerequisites:** Modules 1-7

**Connects To:**
- Module 12: Transactions for safe modifications
- Module 13: DML in stored procedures
- Module 14: Triggers on DML operations

---

### **PHASE 4: Database Design and Management** (12-16 hours)
*Goal: Design efficient databases and optimize performance*

#### Prerequisites
- Completed Phases 1-3
- Understanding of all query types
- Comfort with complex multi-table operations

#### Learning Outcomes
By the end of Phase 4, you will:
- Design normalized database schemas
- Create and manage database objects (tables, views, indexes)
- Optimize query performance
- Implement data integrity constraints
- Understand transaction management

---

#### **Module 10: Database Objects - DDL and Schema Design**
**Time Estimate:** 4-5 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- CREATE TABLE with constraints
- ALTER TABLE operations
- DROP, TRUNCATE considerations
- Normalization (1NF, 2NF, 3NF, BCNF)
- Views and their uses
- Temporary tables

**Key Skills:**
- Schema design principles
- Defining primary and foreign keys
- Creating indexes
- Denormalization strategies
- View creation and management

**Prerequisites:** Modules 1-9

**Connects To:**
- Module 11: Index design
- Module 12: Constraint enforcement
- Module 15: Advanced schema patterns

---

#### **Module 11: Indexes and Query Optimization**
**Time Estimate:** 5-7 hours  
**Difficulty:** ⭐⭐⭐⭐ Advanced

**What You'll Learn:**
- Index types (B-tree, Hash, Full-text)
- Creating effective indexes
- Understanding EXPLAIN plans
- Query optimization techniques
- Index maintenance
- Covering indexes

**Key Skills:**
- Reading execution plans
- Identifying performance bottlenecks
- Index selection strategies
- Query rewriting for performance
- Monitoring query performance

**Prerequisites:** Modules 1-10

**Connects To:**
- Module 16: Production optimization practices

---

#### **Module 12: Transactions and Data Integrity**
**Time Estimate:** 3-4 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- ACID properties
- BEGIN, COMMIT, ROLLBACK
- Transaction isolation levels
- Locking mechanisms
- Deadlock handling
- Constraints (CHECK, UNIQUE, NOT NULL, FK)

**Key Skills:**
- Safe multi-statement operations
- Choosing isolation levels
- Implementing business rules
- Error handling in transactions
- Data consistency patterns

**Prerequisites:** Modules 1-10

**Connects To:**
- Module 13: Transactions in procedures
- Module 14: Transactions in triggers
- Module 16: Production transaction patterns

---

### **PHASE 5: Enterprise SQL and Real-World Applications** (9-12 hours)
*Goal: Master enterprise-level SQL programming and best practices*

#### Prerequisites
- Completed Phases 1-4
- Strong foundation in all SQL categories
- Understanding of database design and optimization

#### Learning Outcomes
By the end of Phase 5, you will:
- Create stored procedures and functions
- Implement triggers for automated logic
- Apply advanced programming patterns
- Follow professional SQL standards
- Build production-ready database solutions

---

#### **Module 13: Stored Procedures and Functions**
**Time Estimate:** 3-4 hours  
**Difficulty:** ⭐⭐⭐⭐ Advanced

**What You'll Learn:**
- Stored procedure syntax
- IN, OUT, INOUT parameters
- User-defined functions (UDFs)
- Control flow (IF, CASE, LOOP)
- Error handling (DECLARE, HANDLER)
- Procedure vs function differences

**Key Skills:**
- Encapsulating business logic
- Creating reusable code
- Parameter handling
- Complex procedural logic
- Security considerations

**Prerequisites:** Modules 1-12

**Connects To:**
- Module 14: Procedures in triggers
- Module 16: Production code organization

---

#### **Module 14: Triggers and Event-Driven Logic**
**Time Estimate:** 2-3 hours  
**Difficulty:** ⭐⭐⭐⭐ Advanced

**What You'll Learn:**
- BEFORE and AFTER triggers
- INSERT, UPDATE, DELETE triggers
- NEW and OLD pseudo-records
- Trigger use cases
- Trigger limitations and pitfalls

**Key Skills:**
- Automating data validation
- Audit trail implementation
- Cascading logic
- When to use (and avoid) triggers
- Debugging trigger issues

**Prerequisites:** Modules 1-13

**Connects To:**
- Module 16: Production trigger patterns

---

#### **Module 15: Advanced SQL Programming Concepts**
**Time Estimate:** 2-3 hours  
**Difficulty:** ⭐⭐⭐⭐ Advanced

**What You'll Learn:**
- Dynamic SQL
- Pivoting and unpivoting data
- Recursive queries (advanced)
- JSON operations in MySQL
- Regular expressions
- Advanced string manipulation

**Key Skills:**
- Building flexible queries
- Complex data transformations
- Hierarchical data processing
- Semi-structured data handling
- Advanced pattern matching

**Prerequisites:** Modules 1-14

**Connects To:**
- Module 16: Real-world applications

---

#### **Module 16: Professional SQL Practices**
**Time Estimate:** 2-3 hours  
**Difficulty:** ⭐⭐⭐ Intermediate-Advanced

**What You'll Learn:**
- Code organization and documentation
- Naming conventions
- Version control for databases
- Testing strategies
- Security best practices
- Performance monitoring
- Code review guidelines

**Key Skills:**
- Writing maintainable SQL
- Documenting database logic
- Following industry standards
- Security awareness
- Collaboration practices

**Prerequisites:** Modules 1-15

**Connects To:**
- All previous modules (applies best practices retroactively)

---

## 🎓 Certification Preparation

After completing this curriculum, you'll be prepared for:
- **MySQL Certified Developer**
- **Oracle SQL Fundamentals Certification**
- Entry-level database developer positions
- Data analyst roles requiring SQL proficiency

---

## 📈 Progress Tracking

### Weekly Study Plan (Recommended)

**Weeks 1-2: Phase 1 (Foundations)**
- Week 1: Modules 1-2
- Week 2: Module 3 + review

**Weeks 3-4: Phase 2 (Intermediate Skills)**
- Week 3: Modules 4-5
- Week 4: Module 6 + review

**Weeks 5-6: Phase 3 (Advanced Queries)**
- Week 5: Modules 7-8
- Week 6: Module 9 + review

**Weeks 7-8: Phase 4 (Database Management)**
- Week 7: Modules 10-11
- Week 8: Module 12 + review

**Weeks 9-10: Phase 5 (Enterprise SQL)**
- Week 9: Modules 13-14
- Week 10: Modules 15-16 + final review

---

## 🔄 Review Cycles

### After Phase 1 (Week 2)
- Revisit Module 1 terminology
- Practice SELECT queries from scratch
- Test data type knowledge

### After Phase 2 (Week 4)
- Review Phases 1-2 completely
- Practice complex JOINs and subqueries
- Self-test on aggregate functions

### After Phase 3 (Week 6)
- Review all query types
- Practice window functions
- Test DML operation safety

### After Phase 4 (Week 8)
- Review schema design principles
- Practice optimization techniques
- Test transaction understanding

### After Phase 5 (Week 10)
- Complete review of all modules
- Build capstone project
- Prepare for certification

---

## 🛠️ Hands-On Project Recommendations

### After Phase 1: Build a Simple Inventory Database
- Create tables for products and categories
- Write SELECT queries for reporting

### After Phase 2: E-commerce Analytics Dashboard
- Design customer, orders, and products schema
- Write analytical queries with JOINs and aggregations

### After Phase 3: Sales Performance Analytics
- Implement window functions for rankings
- Build time-series analysis queries

### After Phase 4: Complete Schema with Optimization
- Design normalized schema for business domain
- Create indexes and optimize queries

### After Phase 5: Production-Ready Application
- Implement stored procedures for business logic
- Add triggers for audit trails
- Document and test thoroughly

---

## 📚 Additional Resources

### Books
- "Learning SQL" by Alan Beaulieu (O'Reilly)
- "SQL Performance Explained" by Markus Winand

### Online Practice
- LeetCode SQL problems
- HackerRank SQL challenges
- Mode Analytics SQL tutorial

### Tools
- MySQL Workbench (GUI)
- DBeaver (Multi-database tool)
- Docker (Isolated environments)

### Communities
- Stack Overflow (SQL tag)
- Reddit: r/SQL
- MySQL Community Forums

---

## ⚡ Quick Tips for Success

1. **Practice Daily**: Even 30 minutes daily beats 5-hour weekend sessions
2. **Type Everything**: Don't copy-paste - muscle memory matters
3. **Break Things**: Experiment and learn from errors
4. **Ask Questions**: Use comments to document what you don't understand
5. **Build Portfolio**: Save your best queries and solutions
6. **Review Regularly**: Revisit earlier modules every 2 weeks
7. **Join Communities**: Learn from others' questions and solutions
8. **Real Data**: Work with actual datasets when possible
9. **Performance Awareness**: Always consider query efficiency
10. **Document Learning**: Keep a journal of "aha!" moments

---

## 🎯 Learning Objectives by Phase

| Phase | Key Objective | Skills Acquired |
|-------|--------------|-----------------|
| 1 | SQL Literacy | Reading data, basic filtering, understanding schemas |
| 2 | Data Analysis | Aggregations, multi-table queries, complex filtering |
| 3 | Advanced Queries | Analytics, data modification, set operations |
| 4 | Database Design | Schema design, optimization, data integrity |
| 5 | Enterprise Skills | Procedural SQL, automation, best practices |

---

**Ready to start? Head to [Module 1](lessons/phase-1/module-01-introduction-to-databases-and-sql.md) and begin your SQL journey!**
