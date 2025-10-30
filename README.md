# 🗄️ SQL Training Curriculum - Complete Learning Path# SQL Training Curriculum



[![MySQL](https://img.shields.io/badge/MySQL-8.x-blue.svg)](https://www.mysql.com/)Welcome! This repository contains a complete, beginner-friendly SQL curriculum designed for recent graduates. You’ll learn SQL step by step with real-world scenarios, clear explanations, and hands-on practice using MySQL syntax.

[![Difficulty](https://img.shields.io/badge/Difficulty-Beginner%20to%20Advanced-green.svg)]()

[![Time](https://img.shields.io/badge/Total%20Time-60--80%20hours-orange.svg)]()## How to use this course

- Start at Phase 1 and progress in order. Each lesson builds on the last.

Welcome! This repository contains a **comprehensive, beginner-friendly SQL curriculum** designed for recent graduates and aspiring data professionals. You'll learn SQL step by step with real-world scenarios, clear explanations, and hands-on practice using MySQL syntax.- Every module includes:

  - Introduction and learning objectives

---  - Concept breakdown with syntax and visuals

  - 3 worked examples (basic → intermediate → advanced)

## 📋 Table of Contents  - Common patterns, best practices, and troubleshooting

  - 5 exercises with sample data, hints, and detailed solutions

1. [Getting Started](#-getting-started)  - Self-assessment checklist and real-world connections

2. [Course Structure](#-course-structure)- You can run the sample data and solutions directly in MySQL (or in services like MySQL Workbench, Docker, or a cloud sandbox).

3. [Module Index](#-module-index)

4. [Quick Reference](#-quick-reference)## Prerequisites

5. [Progress Tracking](#-progress-tracking)- No prior SQL experience required.

6. [Resources](#-resources)- A local MySQL 8.x server or a cloud-based MySQL workspace.



---## Quick setup on Windows

1. Install MySQL Community Server (8.x): https://dev.mysql.com/downloads/mysql/

## 🚀 Getting Started2. Optionally install MySQL Workbench: https://dev.mysql.com/downloads/workbench/

3. Create a practice schema:

### Prerequisites   ```sql

- ✅ No prior SQL experience required   CREATE DATABASE sql_training;

- ✅ A local MySQL 8.x server or cloud-based MySQL workspace   USE sql_training;

- ✅ Basic understanding of spreadsheets/tables   ```

- ✅ Willingness to practice hands-on4. For each module, copy the sample `CREATE TABLE` and `INSERT` statements into your MySQL client and run the practice queries.



### Quick Setup on WindowsNote: When syntax differs from PostgreSQL or SQL Server, the lessons call it out explicitly so you can transfer your knowledge across systems.



1. **Install MySQL Community Server (8.x)**## Course map

   - Download: https://dev.mysql.com/downloads/mysql/

   - Follow installation wizard- Phase 1: Foundations

  1. Introduction to Databases and SQL

2. **[Optional] Install MySQL Workbench**  2. Basic SQL Queries — SELECT Fundamentals

   - Download: https://dev.mysql.com/downloads/workbench/  3. Working with Data Types and Basic Functions

   - Provides GUI for database management

- Phase 2: Intermediate SQL Skills

3. **Create Practice Database**  4. Aggregate Functions and Grouping

   ```sql  5. Joins and Relationships

   CREATE DATABASE sql_training;  6. Subqueries and CTEs

   USE sql_training;

   ```- Phase 3: Advanced Query Techniques

  7. Set Operations and Advanced Filtering

4. **Ready to Learn!**  8. Window Functions and Analytics

   - For each module, copy sample `CREATE TABLE` and `INSERT` statements  9. Data Modification — DML Operations

   - Run practice queries in MySQL client or Workbench

- Phase 4: Database Design and Management

> 💡 **Note**: When syntax differs from PostgreSQL or SQL Server, lessons call it out explicitly for cross-platform knowledge transfer.  10. Database Objects — DDL and Schema Design

  11. Indexes and Query Optimization

---  12. Transactions and Data Integrity



## 📚 Course Structure- Phase 5: Enterprise SQL and Real-World Applications

  13. Stored Procedures and Functions

### What's Included in Each Module  14. Triggers and Event-Driven Logic

  15. Advanced SQL Programming Concepts

Every module follows a proven learning structure:  16. Professional SQL Practices



- ✅ **Introduction** - Real-world scenarios showing why this matters## Where are the lessons?

- ✅ **Learning Objectives** - Clear goals for the moduleAll lessons are in the `lessons` folder, grouped by phase:

- ✅ **Concept Explanation** - Detailed breakdown with analogies and visuals- `lessons/phase-1` → Modules 1–3

- ✅ **Progressive Examples** - Basic → Intermediate → Advanced- `lessons/phase-2` → Modules 4–6

- ✅ **Common Patterns** - Reusable templates and best practices- `lessons/phase-3` → Modules 7–9

- ✅ **5+ Practice Exercises** - With sample data, hints, and detailed solutions- `lessons/phase-4` → Modules 10–12

- ✅ **Self-Assessment Checklist** - Verify readiness before moving forward- `lessons/phase-5` → Modules 13–16

- ✅ **Real-World Connections** - How professionals use these concepts

Each lesson is a standalone Markdown file you can read top-to-bottom or jump to sections as needed.

### Study Approach

## Tips for success

1. **Sequential Learning**: Complete modules in order - each builds on previous knowledge- Practice each exercise before reading the solution.

2. **Hands-On Focus**: 70% practice, 30% theory- Keep notes on mistakes and how you fixed them.

3. **Active Learning**: Type all queries manually (no copy-paste)- Use the self-assessment checklist to confirm you’re ready to move on.

4. **Incremental Progress**: Complete one module before starting the next- Revisit earlier modules anytime as references.

5. **Regular Review**: Revisit earlier modules every 2 weeks

Happy querying!

📖 **For detailed learning strategy, see [LEARNING-PLAN.md](LEARNING-PLAN.md)**

---

## 📖 Module Index

### **PHASE 1: Foundations** (12-16 hours)
*Master the basics and build confidence with SQL*

| Module | Topic | Time | Difficulty | File |
|--------|-------|------|-----------|------|
| 1 | Introduction to Databases and SQL | 3-4h | ⭐ | [View](lessons/phase-1/module-01-introduction-to-databases-and-sql.md) |
| 2 | SELECT Fundamentals | 4-5h | ⭐ | [View](lessons/phase-1/module-02-select-fundamentals.md) |
| 3 | Data Types and Basic Functions | 5-7h | ⭐⭐ | [View](lessons/phase-1/module-03-working-with-data-types-and-basic-functions.md) |

**Phase 1 Goals:**
- Navigate MySQL confidently
- Write SELECT queries with filtering and sorting
- Work with strings, numbers, and dates
- Understand NULL handling

---

### **PHASE 2: Intermediate SQL Skills** (15-20 hours)
*Analyze data and work with multiple tables*

| Module | Topic | Time | Difficulty | File |
|--------|-------|------|-----------|------|
| 4 | Aggregate Functions and Grouping | 4-5h | ⭐⭐ | [View](lessons/phase-2/module-04-aggregate-functions-and-grouping.md) |
| 5 | Joins and Relationships | 6-8h | ⭐⭐⭐ | [View](lessons/phase-2/module-05-joins-and-relationships.md) |
| 6 | Subqueries and CTEs | 5-7h | ⭐⭐⭐ | [View](lessons/phase-2/module-06-subqueries-and-ctes.md) |

**Phase 2 Goals:**
- Summarize data with aggregations
- Combine data from multiple tables
- Write subqueries and CTEs
- Understand database relationships

---

### **PHASE 3: Advanced Query Techniques** (12-16 hours)
*Master complex data manipulation and analytics*

| Module | Topic | Time | Difficulty | File |
|--------|-------|------|-----------|------|
| 7 | Set Operations and Advanced Filtering | 3-4h | ⭐⭐ | [View](lessons/phase-3/module-07-set-operations-and-advanced-filtering.md) |
| 8 | Window Functions and Analytics | 5-7h | ⭐⭐⭐⭐ | [View](lessons/phase-3/module-08-window-functions-and-analytics.md) |
| 9 | Data Modification - DML Operations | 4-5h | ⭐⭐⭐ | [View](lessons/phase-3/module-09-data-modification-dml-operations.md) |

**Phase 3 Goals:**
- Perform set operations (UNION, INTERSECT, EXCEPT)
- Write analytical queries with window functions
- Safely modify data (INSERT, UPDATE, DELETE)
- Handle complex transformations

---

### **PHASE 4: Database Design and Management** (12-16 hours)
*Design efficient databases and optimize performance*

| Module | Topic | Time | Difficulty | File |
|--------|-------|------|-----------|------|
| 10 | Database Objects - DDL and Schema Design | 4-5h | ⭐⭐⭐ | [View](lessons/phase-4/module-10-database-objects-ddl-and-schema-design.md) |
| 11 | Indexes and Query Optimization | 5-7h | ⭐⭐⭐⭐ | [View](lessons/phase-4/module-11-indexes-and-query-optimization.md) |
| 12 | Transactions and Data Integrity | 3-4h | ⭐⭐⭐ | [View](lessons/phase-4/module-12-transactions-and-data-integrity.md) |

**Phase 4 Goals:**
- Design normalized database schemas
- Create and manage database objects
- Optimize query performance
- Implement transactions and constraints

---

### **PHASE 5: Enterprise SQL and Real-World Applications** (9-12 hours)
*Master professional SQL programming*

| Module | Topic | Time | Difficulty | File |
|--------|-------|------|-----------|------|
| 13 | Stored Procedures and Functions | 3-4h | ⭐⭐⭐⭐ | [View](lessons/phase-5/module-13-stored-procedures-and-functions.md) |
| 14 | Triggers and Event-Driven Logic | 2-3h | ⭐⭐⭐⭐ | [View](lessons/phase-5/module-14-triggers-and-event-driven-logic.md) |
| 15 | Advanced SQL Programming Concepts | 2-3h | ⭐⭐⭐⭐ | [View](lessons/phase-5/module-15-advanced-sql-programming-concepts.md) |
| 16 | Professional SQL Practices | 2-3h | ⭐⭐⭐ | [View](lessons/phase-5/module-16-professional-sql-practices.md) |

**Phase 5 Goals:**
- Create stored procedures and functions
- Implement triggers and automation
- Apply advanced programming patterns
- Follow professional best practices

---

## 🎯 Quick Reference

### Total Course Statistics

| Metric | Value |
|--------|-------|
| **Total Modules** | 16 |
| **Total Study Time** | 60-80 hours |
| **Practice Exercises** | 80+ |
| **Difficulty Range** | Beginner to Advanced |
| **Prerequisites** | None |

### Recommended Study Schedule

| Week | Modules | Focus Area |
|------|---------|-----------|
| 1-2 | 1-3 | Foundations |
| 3-4 | 4-6 | Intermediate Skills |
| 5-6 | 7-9 | Advanced Queries |
| 7-8 | 10-12 | Database Management |
| 9-10 | 13-16 | Enterprise SQL |

### Key Skills by Phase

```
Phase 1: SQL Literacy
├── Reading data with SELECT
├── Filtering and sorting
└── Working with data types

Phase 2: Data Analysis
├── Aggregations and grouping
├── Multi-table JOINs
└── Subqueries and CTEs

Phase 3: Advanced Manipulation
├── Set operations
├── Window functions
└── Data modifications

Phase 4: Database Engineering
├── Schema design
├── Performance optimization
└── Transactions

Phase 5: Professional Development
├── Stored procedures
├── Triggers
└── Best practices
```

---

## 📊 Progress Tracking

### Module Completion Checklist

Track your progress as you complete each module:

**Phase 1: Foundations**
- [ ] Module 1: Introduction to Databases and SQL
- [ ] Module 2: SELECT Fundamentals
- [ ] Module 3: Data Types and Basic Functions

**Phase 2: Intermediate Skills**
- [ ] Module 4: Aggregate Functions and Grouping
- [ ] Module 5: Joins and Relationships
- [ ] Module 6: Subqueries and CTEs

**Phase 3: Advanced Queries**
- [ ] Module 7: Set Operations and Advanced Filtering
- [ ] Module 8: Window Functions and Analytics
- [ ] Module 9: Data Modification - DML Operations

**Phase 4: Database Management**
- [ ] Module 10: Database Objects - DDL and Schema Design
- [ ] Module 11: Indexes and Query Optimization
- [ ] Module 12: Transactions and Data Integrity

**Phase 5: Enterprise SQL**
- [ ] Module 13: Stored Procedures and Functions
- [ ] Module 14: Triggers and Event-Driven Logic
- [ ] Module 15: Advanced SQL Programming Concepts
- [ ] Module 16: Professional SQL Practices

📝 **Tip**: Copy this checklist to a personal document and check off modules as you complete them!

---

## 📚 Resources

### Essential Files
- 📖 **[LEARNING-PLAN.md](LEARNING-PLAN.md)** - Detailed learning path with outcomes and strategies
- 📁 **[lessons/](lessons/)** - All course modules organized by phase
- 📝 **[custom.prompt.md](custom.prompt.md)** - Lesson creation guidelines

### Where Are the Lessons?

```
lessons/
├── phase-1/     → Modules 1-3 (Foundations)
├── phase-2/     → Modules 4-6 (Intermediate)
├── phase-3/     → Modules 7-9 (Advanced Queries)
├── phase-4/     → Modules 10-12 (Database Management)
└── phase-5/     → Modules 13-16 (Enterprise SQL)
```

Each lesson is a **standalone Markdown file** - read top-to-bottom or jump to specific sections.

### External Resources

**📚 Recommended Books:**
- "Learning SQL" by Alan Beaulieu (O'Reilly)
- "SQL Performance Explained" by Markus Winand

**💻 Practice Platforms:**
- [LeetCode SQL](https://leetcode.com/problemset/database/) - Coding challenges
- [HackerRank SQL](https://www.hackerrank.com/domains/sql) - Practice problems
- [Mode Analytics Tutorial](https://mode.com/sql-tutorial/) - Interactive lessons

**🛠️ Tools:**
- [MySQL Workbench](https://www.mysql.com/products/workbench/) - Official GUI
- [DBeaver](https://dbeaver.io/) - Universal database tool
- [Docker MySQL](https://hub.docker.com/_/mysql) - Containerized environment

**👥 Communities:**
- [Stack Overflow - SQL Tag](https://stackoverflow.com/questions/tagged/sql)
- [Reddit - r/SQL](https://www.reddit.com/r/SQL/)
- [MySQL Community Forums](https://forums.mysql.com/)

### Certification Preparation

After completing this curriculum, you'll be prepared for:
- ✅ MySQL Certified Developer
- ✅ Oracle SQL Fundamentals Certification
- ✅ Entry-level database developer positions
- ✅ Data analyst roles requiring SQL proficiency

---

## 💡 Tips for Success

1. **🎯 Practice Daily** - Even 30 minutes daily beats 5-hour weekend sessions
2. **⌨️ Type Everything** - Don't copy-paste; muscle memory matters
3. **🔨 Break Things** - Experiment fearlessly and learn from errors
4. **❓ Ask Questions** - Use comments to document what you don't understand
5. **📁 Build Portfolio** - Save your best queries and solutions
6. **🔄 Review Regularly** - Revisit earlier modules every 2 weeks
7. **👥 Join Communities** - Learn from others' questions
8. **📊 Use Real Data** - Work with actual datasets when possible
9. **⚡ Think Performance** - Always consider query efficiency
10. **📝 Document Learning** - Keep a journal of "aha!" moments

---

## 🎓 What You'll Build

### Project Milestones

- **After Phase 1**: Simple inventory database with reporting queries
- **After Phase 2**: E-commerce analytics dashboard with JOINs
- **After Phase 3**: Sales performance analytics with window functions
- **After Phase 4**: Optimized schema with indexes and constraints
- **After Phase 5**: Production-ready application with procedures and triggers

---

## 🤝 Contributing

Found an error or have suggestions? Feel free to:
- Open an issue
- Submit a pull request
- Share your learning experience

---

## 📄 License

This curriculum is designed for educational purposes. Feel free to use, share, and adapt for learning.

---

## 🚀 Ready to Start?

**Begin your SQL journey with [Module 1: Introduction to Databases and SQL](lessons/phase-1/module-01-introduction-to-databases-and-sql.md)**

Or review the complete **[LEARNING-PLAN.md](LEARNING-PLAN.md)** for detailed strategies and outcomes.

---

**Happy querying! 🎉**
"# SQL-Training-EY" 
