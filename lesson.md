# 🎓 Lesson 1.5: SQL Advanced — Instructor Guide

## Session Overview

| Item | Detail |
|------|--------|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Hands-On SQL in DbGate |
| **Prerequisites** | SQL DML (Lesson 1.4); able to write SELECT/WHERE/GROUP BY; `unit-1-5.db` downloaded |
| **Tools** | DuckDB + DbGate |
| **Dataset** | SafeDrive Insurance (`unit-1-5.db`) — 4 tables: `address`, `car`, `claim`, `client` |

### Agenda

| Time | Section | Focus |
|------|---------|-------|
| 0:00 – 0:55 | Part 1: Joins & Unions | Combining data from multiple tables |
| 0:55 – 1:00 | Break | — |
| 1:00 – 1:55 | Part 2: Window Functions | Row-level analytics without losing detail |
| 1:55 – 2:00 | Break | — |
| 2:00 – 2:55 | Part 3: Subqueries & CTEs | Nested logic and readable query structures |
| 2:55 – 3:00 | Wrap-Up | Key Takeaways & Post-Class Assignment Briefing |

> **Before class:** Use `SHOW TABLES;` and `DESCRIBE table_name;` to explore the database structure with students.

---

## 🏃 Part 1: Joins & Unions (55 min)

### 🎯 Learning Objective
Execute INNER, LEFT, RIGHT, and FULL OUTER JOINs to combine data across multiple tables, and use UNION to stack result sets.

### 📖 Theory Recap (10 min)

**Analogy:** Imagine a party with two guest lists — one from the host, one from the caterer.

| Join Type | The Party Analogy | What it returns |
|-----------|-------------------|-----------------|
| **INNER JOIN** | Only guests who appear on *both* lists | Matching rows only |
| **LEFT JOIN** | Everyone on the host's list; empty box for no caterer match | All left rows + matched right rows |
| **RIGHT JOIN** | Everyone on the caterer's list | All right rows + matched left rows |
| **FULL JOIN** | Everyone from both lists, even if unmatched | All rows from both tables |
| **UNION** | Combine two identical guest lists into one | Stacked rows (removes duplicates) |

### 🛠️ Hands-On Activity: "The Insurance Report" (35 min)

**Part A — Explore the schema (5 min):**
```sql
SHOW TABLES;
DESCRIBE claim;
```

**Part B — Build progressively (30 min):**

1. INNER JOIN `claim` and `car` on `car_id`:
```sql
SELECT c.id, c.claim_date, c.claim_amt, car.make, car.model
FROM claim c
INNER JOIN car ON c.car_id = car.id;
```

2. LEFT JOIN `client` to see all clients even without claims.

3. Build a master report joining all 4 tables (claim → car → client → address).

4. Use UNION to combine `claim_date` from two different year-ranges.

**Discussion Questions:**
- "Which clients have never made a claim? Which JOIN type surfaces this?"
- "What does a NULL `claim_amt` mean after a LEFT JOIN?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "I can use `WHERE table_a.id = table_b.id` instead of JOIN." → This works for INNER JOINs only. You cannot replicate LEFT/RIGHT/FULL JOINs with WHERE clauses.
- **Business Case:** Every business intelligence dashboard at companies like Salesforce, SAP, and Oracle runs on multi-table JOINs. A single dashboard query might join 8–12 tables simultaneously.

---

## 🏃 Part 2: Window Functions (55 min)

### 🎯 Learning Objective
Apply Window Functions (`SUM OVER`, `RANK OVER`) to perform running totals, rankings, and comparisons without collapsing rows.

### 📖 Theory Recap (10 min)

**Analogy:** `GROUP BY` collapses your data — like averaging exam scores by class, you lose individual results. Window functions are like adding a class-average column *next to* each student's score — the detail stays, but you gain the group context.

**Anatomy of a window function:**
```sql
function_name() OVER (
    PARTITION BY group_column   -- "For each group..."
    ORDER BY sort_column        -- "...in this order..."
)
```

| Function | Purpose |
|----------|---------|
| `SUM() OVER (...)` | Running total within a partition |
| `RANK() OVER (...)` | Rank within a partition (ties get same rank; gaps after) |
| `ROW_NUMBER() OVER (...)` | Unique sequential number regardless of ties |
| `QUALIFY` | Filter on window function results |

### 🛠️ Hands-On Activity: "The Risk Analyser" (35 min)

1. **Running total of claims per car:**
```sql
SELECT id, car_id, claim_date, claim_amt,
       SUM(claim_amt) OVER (PARTITION BY car_id ORDER BY claim_date) AS running_total
FROM claim;
```

2. **Rank clients by total claim amount within their state:**
```sql
SELECT cl.name, a.state, SUM(c.claim_amt) AS total_claimed,
       RANK() OVER (PARTITION BY a.state ORDER BY SUM(c.claim_amt) DESC) AS state_rank
FROM claim c
JOIN car ON c.car_id = car.id
JOIN client cl ON car.client_id = cl.id
JOIN address a ON cl.address_id = a.id
GROUP BY cl.name, a.state;
```

3. **Use QUALIFY to find the top-claiming client per state:**
```sql
-- Add: QUALIFY state_rank = 1
```

**Discussion Questions:**
- "What happens to the running total when you remove `ORDER BY` from the window?"
- "What's the difference between RANK() and ROW_NUMBER() when two clients have the same total?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "I can filter window function results with WHERE." → Window functions are evaluated *after* WHERE. Use `QUALIFY` (DuckDB) or a subquery/CTE to filter on window results.
- **Business Case:** Spotify uses window functions to compute "listening streaks" — a running count of consecutive days a user listened — to drive engagement notifications.

---

## 🏃 Part 3: Subqueries & CTEs (55 min)

### 🎯 Learning Objective
Write subqueries using `IN` and `EXISTS`, and structure complex multi-step queries using Common Table Expressions (CTEs) for readability.

### 📖 Theory Recap (10 min)

**Analogy:** A subquery is like a sticky note — quick, inline, but messy if overused. A CTE is like a whiteboard — you write the result once, name it, and reference it as many times as needed.

| Technique | When to use | Readability |
|-----------|------------|-------------|
| **Subquery (WHERE IN)** | Simple filtering on aggregated result | Medium |
| **Subquery (EXISTS)** | Check existence efficiently | Medium |
| **Derived Table (FROM (...))** | Pre-filter before main query | Medium |
| **CTE (WITH ...)** | Multi-step logic; reference the result multiple times | High ✅ |

### 🛠️ Hands-On Activity: "The Insurance Auditor" (35 min)

**Step 1 — Subquery with IN:**
```sql
-- Find cars with below-average resale value in their category
SELECT * FROM car
WHERE resale_value < (
    SELECT AVG(resale_value) FROM car WHERE category = car.category
);
```

**Step 2 — Build a 4-stage CTE pipeline:**
```sql
WITH
market_avg AS (
    SELECT category, AVG(resale_value) AS avg_resale
    FROM car GROUP BY category
),
claim_totals AS (
    SELECT car_id, SUM(claim_amt) AS total_claimed
    FROM claim GROUP BY car_id
),
client_enriched AS (
    SELECT cl.name, cl.id, a.state, ct.total_claimed
    FROM claim_totals ct
    JOIN car ON ct.car_id = car.id
    JOIN client cl ON car.client_id = cl.id
    JOIN address a ON cl.address_id = a.id
),
state_ranked AS (
    SELECT *, RANK() OVER (PARTITION BY state ORDER BY total_claimed DESC) AS state_rank
    FROM client_enriched
)
SELECT * FROM state_ranked WHERE state_rank <= 2;
```

**Discussion Questions:**
- "Why is the CTE easier to debug than nesting all this as subqueries?"
- "If you needed to use `claim_totals` in two different parts of the query, how would that change if you used a subquery vs. a CTE?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "CTEs are faster than subqueries." → Not necessarily. CTEs are primarily a readability tool. Performance depends on the database engine's optimiser.
- **Business Case:** Insurance companies like Allianz run exactly this type of CTE-based query daily — identifying top-claiming policyholders by region for fraud detection and risk-based pricing adjustments.

---

## 🎯 Wrap-Up (5 min)

### Key Takeaways
1. **JOINs are the core of multi-table analysis.** INNER for matches only; LEFT to preserve your primary table. Know the difference before every query.
2. **Window functions add context without losing rows** — they're the key to running totals, rankings, and percentiles.
3. **CTEs are readability tools** — use them to break complex queries into named, logical steps that can be reviewed and debugged independently.

### Next Steps
- **Post-Class:** Complete the [Insurance Auditor Project](./post-class.md) — 3 targeted challenges + a multi-table CTE capstone (45–60 min).
- **Next Lesson:** Lesson 1.6 transitions from SQL to Python, introducing NumPy as the numerical foundation for data science.
