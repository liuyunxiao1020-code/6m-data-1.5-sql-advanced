# 📚 Lesson 1.5: SQL Advanced

**Theme:** From Tables to Intelligence — joining, ranking, and structuring complex queries

---

## 📅 Lesson Overview

| Section | Duration | Topic / Activity |
|---------|----------|-----------------|
| **Part 1: Joins & Unions** | 55 min | INNER, LEFT, RIGHT, FULL JOIN; UNION; multi-table reporting |
| **Part 2: Window Functions** | 55 min | Running totals; RANK(); QUALIFY; row-level analytics |
| **Part 3: Subqueries & CTEs** | 55 min | Nested queries; IN/EXISTS; Derived tables; Common Table Expressions |

---

## 🎯 Learning Outcomes

By the end of this lesson, you will be able to:

1. **Execute** INNER, LEFT, RIGHT, and FULL OUTER JOINs to combine data across multiple tables.
2. **Apply** Window Functions (`SUM OVER`, `RANK OVER`) to compute running totals and rankings without losing row-level detail.
3. **Write** Subqueries using `IN`, `EXISTS`, and derived tables to filter results based on aggregated logic.
4. **Structure** complex analytical queries using Common Table Expressions (CTEs) for readability and reuse.

---

## 📂 Course Materials

| Material | Description | Est. Time |
|----------|-------------|-----------|
| [Pre-Class](./pre-class.md) | Metadata queries, JOIN types, DbGate setup | 30–45 min |
| [Lesson Plan](./lesson.md) | Instructor guide for the 3-hour hands-on session | 3 hours |
| [Post-Class](./post-class.md) | Insurance Auditor — multi-table analysis challenge | 45–60 min |
| [Reference](./reference.md) | Advanced SQL cheat sheet — JOINs, window functions, CTEs | As needed |

---

## 🛠️ Tools & Setup

- **[DuckDB](https://duckdb.org):** In-process analytical database engine.
- **[DbGate](https://dbgate.org):** Free, cross-platform database manager.
- **Dataset:** `unit-1-5.db` — SafeDrive Insurance (clients, cars, claims, addresses). Download link in [pre-class.md](./pre-class.md).
