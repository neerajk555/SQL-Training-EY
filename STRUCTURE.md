# 📂 Project Structure Documentation

This document explains the organization, naming conventions, and structure of the SQL Training Curriculum repository.

---

## 📁 Repository Layout

```
SQL-Training-EY/
├── README.md                    # Main entry point with course overview and index
├── LEARNING-PLAN.md            # Detailed learning path with strategies and outcomes
├── STRUCTURE.md                # This file - explains project organization
├── custom.prompt.md            # Guidelines for creating new lesson modules
│
└── lessons/                    # All course modules organized by phase
    ├── phase-1/               # Foundations (Modules 1-3)
    │   ├── module-01-introduction-to-databases-and-sql.md
    │   ├── module-02-select-fundamentals.md
    │   └── module-03-working-with-data-types-and-basic-functions.md
    │
    ├── phase-2/               # Intermediate SQL Skills (Modules 4-6)
    │   ├── module-04-aggregate-functions-and-grouping.md
    │   ├── module-05-joins-and-relationships.md
    │   └── module-06-subqueries-and-ctes.md
    │
    ├── phase-3/               # Advanced Query Techniques (Modules 7-9)
    │   ├── module-07-set-operations-and-advanced-filtering.md
    │   ├── module-08-window-functions-and-analytics.md
    │   └── module-09-data-modification-dml-operations.md
    │
    ├── phase-4/               # Database Design and Management (Modules 10-12)
    │   ├── module-10-database-objects-ddl-and-schema-design.md
    │   ├── module-11-indexes-and-query-optimization.md
    │   └── module-12-transactions-and-data-integrity.md
    │
    └── phase-5/               # Enterprise SQL (Modules 13-16)
        ├── module-13-stored-procedures-and-functions.md
        ├── module-14-triggers-and-event-driven-logic.md
        ├── module-15-advanced-sql-programming-concepts.md
        └── module-16-professional-sql-practices.md
```

---

## 📝 File Naming Conventions

### Module Files

**Pattern**: `module-{number}-{topic-name}.md`

**Rules**:
- Use two-digit module numbers (01-16) for proper sorting
- Use lowercase letters with hyphens separating words
- Keep filenames descriptive but concise
- Use `.md` extension for Markdown files

**Examples**:
- ✅ `module-01-introduction-to-databases-and-sql.md`
- ✅ `module-08-window-functions-and-analytics.md`
- ✅ `module-15-advanced-sql-programming-concepts.md`
- ❌ `Module_1_Intro.md` (inconsistent casing and underscores)
- ❌ `8-window-functions.md` (missing leading zero)

### Documentation Files

**Core Files** (root level):
- `README.md` - Main repository documentation and navigation
- `LEARNING-PLAN.md` - Comprehensive learning strategy guide
- `STRUCTURE.md` - Repository structure documentation (this file)
- `custom.prompt.md` - Lesson creation guidelines

**Naming Rules**:
- Use UPPERCASE for major documentation files
- Use kebab-case for multi-word docs
- Use `.md` extension

---

## 🗂️ Folder Organization

### Phase-Based Structure

The curriculum is divided into **5 phases**, each in its own folder:

| Folder | Phase Name | Focus | Modules |
|--------|-----------|-------|---------|
| `phase-1/` | Foundations | SQL basics | 1-3 |
| `phase-2/` | Intermediate Skills | Multi-table queries | 4-6 |
| `phase-3/` | Advanced Queries | Complex operations | 7-9 |
| `phase-4/` | Database Management | Design & optimization | 3-12 |
| `phase-5/` | Enterprise SQL | Professional practices | 13-16 |

### Why Phase-Based Organization?

1. **Clear Learning Progression**: Each phase builds on the previous
2. **Manageable Chunks**: 3-4 modules per phase reduces overwhelm
3. **Easy Navigation**: Students know exactly where they are in the curriculum
4. **Flexible Pacing**: Complete one phase before moving to the next
5. **Logical Grouping**: Related concepts stay together

---

## 📄 Module File Structure

Each module file follows a **consistent internal structure**:

```markdown
# Module X: [Topic Name]

## 1) Introduction
- Real-world scenario
- Plain definition
- Connection to other modules
- Learning objectives

## 2) Concept Explanation
- Breakdown of key ideas
- Syntax explanations
- Visual representations
- Common misconceptions

## 3) Detailed Examples
- Basic example
- Intermediate example
- Advanced example
(Each with: context, query, input data, output, explanation)

## 4) Common Patterns and Best Practices
- Use cases
- Template queries
- Performance considerations
- Do's and Don'ts
- Troubleshooting tips

## 5) Practice Exercises (5+)
- Exercise title
- Difficulty level
- Business context
- Sample data (CREATE TABLE, INSERT)
- Task description
- Expected output
- Hints (3 progressive)
- Detailed solution with explanation

## 6) Self-Assessment Checklist
- Skill verification questions
- "Can you..." statements

## 7) Real-World Connections
- Industry applications
- Career relevance
- What's next
```

---

## 🎯 Module Numbering System

### Sequential Module Numbers

- **Modules 01-03**: Phase 1 (Foundations)
- **Modules 04-06**: Phase 2 (Intermediate)
- **Modules 07-09**: Phase 3 (Advanced Queries)
- **Modules 10-12**: Phase 4 (Database Management)
- **Modules 13-16**: Phase 5 (Enterprise SQL)

### Module Number Format

- Always use **two-digit** format (01, 02, ... 16)
- Numbers are **consecutive** across all phases
- No gaps in numbering sequence

**Why Two Digits?**
- Ensures proper file sorting in file explorers
- Maintains consistent visual alignment
- Allows expansion to 99 modules if needed

---

## 🔗 Linking Conventions

### Internal Links

**Between Modules**:
```markdown
See [Module 5: Joins and Relationships](../phase-2/module-05-joins-and-relationships.md)
```

**To Documentation**:
```markdown
Review the [LEARNING-PLAN.md](../../LEARNING-PLAN.md) for detailed strategies.
```

**Within Same Phase**:
```markdown
After completing [Module 4](module-04-aggregate-functions-and-grouping.md)...
```

### External Links

```markdown
[MySQL Documentation](https://dev.mysql.com/doc/)
[Stack Overflow - SQL Tag](https://stackoverflow.com/questions/tagged/sql)
```

---

## 📊 Content Organization Principles

### 1. Sequential Learning Path

- Each module **builds on previous modules**
- Prerequisites are clearly stated
- Forward references prepare for upcoming topics

### 2. Self-Contained Modules

- Each module can be **reviewed independently**
- All necessary context is provided
- Sample data included in each module

### 3. Progressive Difficulty

- Examples progress from **basic → intermediate → advanced**
- Exercises increase in complexity
- Clear difficulty ratings help students gauge readiness

### 4. Comprehensive Coverage

- Every SQL concept is explained thoroughly
- Multiple examples for each concept
- Real-world applications demonstrated

### 5. Hands-On Focus

- Every module includes **5+ practice exercises**
- Sample data provided for all exercises
- Detailed solutions with explanations

---

## 🎨 Formatting Standards

### Markdown Conventions

**Headers**:
```markdown
# Module Title (H1 - once per file)
## Major Section (H2)
### Subsection (H3)
#### Detail (H4)
```

**Code Blocks**:
```markdown
```sql
SELECT * FROM customers;
```
```

**Tables**:
```markdown
| Column | Description |
|--------|-------------|
| id     | Primary key |
```

**Lists**:
```markdown
- Unordered item
1. Ordered item
```

**Emphasis**:
```markdown
*italic* or _italic_
**bold** or __bold__
`code` or ``inline code``
```

### SQL Code Style

**Formatting**:
- Keywords in UPPERCASE: `SELECT`, `FROM`, `WHERE`
- Table/column names in lowercase: `customers`, `order_id`
- Indent nested queries
- One clause per line for readability

**Example**:
```sql
SELECT 
    customer_id,
    customer_name,
    COUNT(*) AS order_count
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY customer_id, customer_name
ORDER BY order_count DESC;
```

---

## 🔄 Version Control Considerations

### Branching Strategy

- `main` - Stable, reviewed content
- `develop` - Work-in-progress modules
- `feature/*` - New module development

### Commit Messages

**Format**: `[Module XX] Brief description`

**Examples**:
- `[Module 05] Add advanced JOIN examples`
- `[Module 08] Fix window function syntax error`
- `[README] Update module index with time estimates`
- `[Structure] Add file naming conventions`

### File History

- Keep module files in **one file per module**
- Avoid breaking modules into multiple files
- Use comments within SQL code for explanations

---

## 📦 Future Expansion

### Adding New Modules

1. **Determine Phase**: Where does the new content fit?
2. **Number Assignment**: Use next available number
3. **File Naming**: Follow `module-{number}-{topic}.md` convention
4. **Update Documentation**:
   - Add to README.md module index
   - Update LEARNING-PLAN.md with details
   - Update phase completion times

### Adding Supplementary Content

**Potential Additions**:
```
SQL-Training-EY/
├── exercises/           # Additional practice problems
├── solutions/           # Separate solution files
├── datasets/            # Larger sample datasets
├── projects/            # Capstone project ideas
└── cheatsheets/        # Quick reference guides
```

---

## 🎓 Educational Design Principles

### Cognitive Load Management

- **Chunking**: Content divided into digestible phases
- **Scaffolding**: Each module builds incrementally
- **Worked Examples**: Step-by-step demonstrations
- **Practice Variety**: Multiple exercise types

### Active Learning

- **Hands-On Exercises**: 70% of learning time
- **Problem-Solving**: Real-world scenarios
- **Self-Assessment**: Checklists for mastery verification
- **Reflection**: "What you learned" sections

### Accessibility

- **Clear Language**: No unnecessary jargon
- **Visual Aids**: ASCII diagrams and tables
- **Multiple Examples**: Different learning styles
- **Hints System**: Progressive assistance without giving away answers

---

## 🛠️ Tools and Technologies

### Recommended Development Tools

**For Contributors**:
- **VS Code** - Markdown editing with preview
- **Git** - Version control
- **MySQL Workbench** - Testing SQL examples
- **Markdown linters** - Consistency checking

**For Students**:
- **MySQL 8.x** - Database engine
- **MySQL Workbench** or **DBeaver** - GUI tools
- **Text Editor** - For taking notes
- **Git** (optional) - For cloning repository

---

## 📋 Quality Checklist

### Before Adding/Updating a Module

- [ ] Follows file naming convention
- [ ] Placed in correct phase folder
- [ ] Includes all required sections
- [ ] Has 5+ practice exercises
- [ ] SQL code tested in MySQL 8.x
- [ ] Links to other modules work
- [ ] No spelling/grammar errors
- [ ] Consistent formatting with other modules
- [ ] Updated in README.md index
- [ ] Updated in LEARNING-PLAN.md

---

## 🔍 Finding Content

### Quick Navigation

**To Find a Specific Module**:
1. Check README.md module index
2. Navigate to appropriate phase folder
3. Open module file

**To Find a Specific Topic**:
1. Use text search in repository
2. Check LEARNING-PLAN.md topic index
3. Review module learning objectives

**To Find Examples**:
- Search for "Example:" in module files
- Look in section 3 of each module
- Check practice exercises (section 5)

---

## 📞 Support and Maintenance

### Issue Reporting

**Types of Issues**:
- **Content Errors**: SQL syntax, logic mistakes
- **Broken Links**: 404 or incorrect references
- **Clarity Issues**: Confusing explanations
- **Missing Content**: Gaps in coverage

**Reporting Format**:
```
Title: [Module XX] Brief issue description
Body:
- Location: Module X, Section Y
- Issue: Description of problem
- Suggested Fix: (if applicable)
```

### Content Updates

**Trigger for Updates**:
- MySQL version changes
- SQL standard updates
- User feedback on clarity
- Discovered errors
- Better examples found

---

## 📚 Related Documentation

- **[README.md](README.md)** - Main entry point and course overview
- **[LEARNING-PLAN.md](LEARNING-PLAN.md)** - Detailed learning strategies
- **[custom.prompt.md](custom.prompt.md)** - Content creation guidelines

---

## 🎯 Summary

This repository is designed to be:

- ✅ **Well-Organized**: Clear folder and file structure
- ✅ **Easy to Navigate**: Consistent naming and linking
- ✅ **Scalable**: Room for growth and expansion
- ✅ **Student-Friendly**: Logical progression and clear signposting
- ✅ **Maintainable**: Clear conventions and documentation

**Following these conventions ensures a high-quality learning experience for all students.**

---

*For questions about structure or organization, refer to this document or open an issue.*
