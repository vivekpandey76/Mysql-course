# 🪟 Part 5: Window Functions in MySQL

> **Master MySQL Window Functions** — ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, PARTITION BY and more with real-world practice questions using the `employer_database`.

---

## 📋 Table of Contents

- [What is a Window Function?](#what-is-a-window-function)
- [When to Use Window Functions](#-when-to-use-window-functions)
- [Key Concepts](#-key-concepts)
- [ROW\_NUMBER vs RANK vs DENSE\_RANK](#-row_number-vs-rank-vs-dense_rank)
- [GROUP BY vs PARTITION BY](#-group-by-vs-partition-by)
- [LAG vs LEAD](#-lag-vs-lead)
- [Practice Questions & Solutions](#-practice-questions--solutions)
- [Cheatsheet](#-window-functions-cheatsheet)

---

## What is a Window Function?

A **Window Function** performs a calculation across a set of rows **related to the current row** — without collapsing the result into a single row like `GROUP BY` does.

```sql
-- Syntax
function_name() OVER (
    PARTITION BY column    -- optional: divide into groups
    ORDER BY column        -- optional: define row order
)
```

> 💡 **Why "Window"?** The function looks through a "window" of rows to compute each result. Every row keeps its identity while gaining calculated context from surrounding rows.

---

## ⏰ When to Use Window Functions

| Use Case | Example |
|---|---|
| Rank rows within a group | Top earner per department |
| Compare current row with previous/next row | Salary change vs last employee |
| Calculate running totals | Cumulative salary by department |
| Find the Nth highest/lowest value | 2nd highest salary |
| Display aggregate values alongside each row | Dept total salary next to each employee |
| Group-wise analysis while keeping all rows | Avg dept salary per employee row |
| Avoid subqueries for row-level calculations | Running sums, rankings |

---

## 🔑 Key Concepts

### `OVER()` — The Heart of Every Window Function

`OVER()` defines the **window** (set of rows) the function operates on.

```sql
SELECT *,
    ROW_NUMBER() OVER(ORDER BY salary DESC) AS row_num
FROM employees;
```

Without `OVER()`, you'd just have an aggregate. With `OVER()`, every row gets its own computed value.

---

### `PARTITION BY` — Divide and Conquer

Splits rows into **independent groups** so the window function restarts for each group.

```sql
SELECT *,
    RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

> Think of `PARTITION BY` as `GROUP BY` but without collapsing rows.

---

## 📊 ROW_NUMBER vs RANK vs DENSE_RANK

This is one of the most commonly asked interview topics. Here's the key distinction:

| Function | Handles Ties | Gaps in Ranking? | Use When |
|---|---|---|---|
| `ROW_NUMBER()` | Assigns unique numbers even to ties | No gaps | You need strictly unique row labels |
| `RANK()` | Same rank for ties | **Leaves gaps** | Standard competition ranking (1, 2, 2, 4) |
| `DENSE_RANK()` | Same rank for ties | **No gaps** | You want sequential ranks despite ties (1, 2, 2, 3) |

### Visual Example

Assume three employees earn 90k, 90k, and 70k:

| Salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 90,000 | 1 | 1 | 1 |
| 90,000 | 2 | 1 | 1 |
| 70,000 | 3 | 3 | 2 |

> ⚠️ **Interview Tip:** `RANK()` skips rank 2 above. `DENSE_RANK()` does not. When a question asks for the "Nth highest salary," always prefer `DENSE_RANK()` — it handles duplicates correctly.

---

## 🔄 GROUP BY vs PARTITION BY

| Feature | `GROUP BY` | `PARTITION BY` |
|---|---|---|
| Output rows | **Collapses** rows into one per group | **Keeps all rows** |
| Used with | Aggregate functions (`SUM`, `COUNT`, etc.) | Window functions (`RANK`, `LAG`, etc.) |
| Can display individual row data? | ❌ No | ✅ Yes |
| Typical use | Summary reports | Row-level analysis with group context |

```sql
-- GROUP BY: collapses rows — you only get one row per department
SELECT department
FROM employees
GROUP BY department;

-- PARTITION BY: keeps all rows — every employee row is preserved
SELECT *,
    RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS ranking
FROM employees;
```

> 💡 **Why does this matter?** Use `GROUP BY` when you want a summary. Use `PARTITION BY` when you want each row to carry group-level context (e.g., "what is this employee's rank within their department?").

---

## ↔️ LAG vs LEAD

Both functions let you **peek at other rows** from the current row's perspective — without a self-join.

| Function | Looks At | Returns |
|---|---|---|
| `LAG(col)` | The **previous** row | Value from the row before the current one |
| `LEAD(col)` | The **next** row | Value from the row after the current one |

```sql
SELECT *,
    LAG(salary)  OVER(ORDER BY employee_id) AS previous_salary,
    LEAD(salary) OVER(ORDER BY employee_id) AS next_salary
FROM employees;
```

**Sample Output:**

| employee_id | salary | previous_salary | next_salary |
|---|---|---|---|
| 1 | 50,000 | NULL | 60,000 |
| 2 | 60,000 | 50,000 | 45,000 |
| 3 | 45,000 | 60,000 | NULL |

> 💡 **Why NULL?** The first row has no previous row (`LAG` → NULL). The last row has no next row (`LEAD` → NULL). Always account for NULLs when filtering on these columns.

---

## 💼 Practice Questions & Solutions

### Q1. Assign a unique row number to each employee based on salary

```sql
SELECT *,
    ROW_NUMBER() OVER(ORDER BY salary DESC) AS unique_row_number
FROM employees;
```

> **Why `ROW_NUMBER()`?** We want a unique label for each row — even if two employees earn the same salary. `ROW_NUMBER()` never repeats a number, making it ideal for pagination or picking exactly one row.

---

### Q2. Rank employees based on salary using `RANK()`

```sql
SELECT *,
    RANK() OVER(ORDER BY salary DESC) AS ranking
FROM employees;
```

> **Why `RANK()`?** Employees with equal salaries get the same rank, but the next rank skips a number (like 1, 2, 2, 4). This mirrors real-world competition rankings.

---

### Q3. Rank employees based on salary using `DENSE_RANK()`

```sql
SELECT *,
    DENSE_RANK() OVER(ORDER BY salary DESC) AS ranking
FROM employees;
```

> **Why `DENSE_RANK()`?** Unlike `RANK()`, no numbers are skipped (1, 2, 2, 3). Use this when you want to find the "Nth highest salary" — gaps in `RANK()` would make that logic unreliable.

---

### Q4. Compare ROW_NUMBER vs RANK vs DENSE_RANK side by side

```sql
SELECT *,
    ROW_NUMBER() OVER(ORDER BY salary DESC) AS row_number_ranking,
    RANK()       OVER(ORDER BY salary DESC) AS rank_ranking,
    DENSE_RANK() OVER(ORDER BY salary DESC) AS dense_rank_ranking
FROM employees;
```

> **Why all three together?** This is the clearest way to see how they differ when duplicate salaries exist. Run this query on your dataset and observe which column skips numbers — that's `RANK()`.

---

### Q5. Find the employee with the 2nd highest salary

**Approach 1 — Simple `LIMIT` + `OFFSET`** *(works only when there are no duplicates)*

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Approach 2 — `DENSE_RANK()` subquery** *(handles duplicates correctly)*

```sql
SELECT *
FROM (
    SELECT *,
        DENSE_RANK() OVER(ORDER BY salary DESC) AS ranking
    FROM employees
) t
WHERE ranking = 2;
```

> **Why prefer Approach 2?** If two employees share the highest salary, `OFFSET 1` would return the second person at that same salary — not the true 2nd highest. `DENSE_RANK()` correctly identifies distinct salary levels, making it the robust production-ready solution.

---

### Q6. Display each employee's rank within their department based on salary

```sql
SELECT *,
    RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS ranking
FROM employees
WHERE department IS NOT NULL;
```

> **Why `PARTITION BY department`?** Without it, `RANK()` would rank all employees globally. `PARTITION BY` resets the rank counter for each department — so the highest earner in every department gets rank 1.

---

### Q7. Find the highest-paid employee from each department

**Approach 1 — Subquery**

```sql
SELECT *
FROM (
    SELECT *,
        DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM employees
    WHERE department IS NOT NULL
) t
WHERE ranking = 1;
```

**Approach 2 — CTE (cleaner & more readable)**

```sql
WITH ranked AS (
    SELECT *,
        DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM employees
    WHERE department IS NOT NULL
)
SELECT *
FROM ranked
WHERE ranking = 1;
```

> **Why `DENSE_RANK()` and not `MAX()`?** `MAX()` gives the salary value, but not the full employee row. `DENSE_RANK()` lets you retrieve the entire record. Using `ranking = 1` also handles ties — if two employees share the top salary in a department, both are returned.

---

### Q8. Find employees earning more than the previous employee

```sql
SELECT *
FROM (
    SELECT *,
        LAG(salary) OVER(ORDER BY employee_id) AS previous_salary
    FROM employees
) t
WHERE salary > previous_salary;
```

> **Why `LAG()`?** It fetches the salary of the row before the current one (ordered by `employee_id`), all without a self-join. The outer `WHERE` then filters only those who out-earn their predecessor.

---

### Q9. Find employees earning less than the next employee

```sql
SELECT *
FROM (
    SELECT *,
        LEAD(salary) OVER(ORDER BY employee_id) AS next_salary
    FROM employees
) t
WHERE salary < next_salary;
```

> **Why `LEAD()`?** It looks at the row *after* the current one. Employees where the next person earns more are identified in the outer `WHERE`. Note: the last employee always returns NULL for `next_salary` and is automatically excluded by the comparison.

---

### Q10. Display the salary difference between each employee and the previous employee

```sql
SELECT *,
    salary - LAG(salary) OVER(ORDER BY employee_id) AS salary_difference
FROM employees;
```

> **Why subtract `LAG()`?** This creates a derived column showing how much each employee's salary has changed relative to the one before them. Positive = higher than previous; negative = lower; NULL = first employee (no previous row exists).

---

### Q11. LAG vs LEAD — side-by-side comparison

```sql
SELECT *,
    LAG(salary)  OVER(ORDER BY employee_id) AS previous_salary,
    LEAD(salary) OVER(ORDER BY employee_id) AS next_salary
FROM employees;
```

> **Why show both together?** Running this query makes the difference immediately clear: `LAG` shifts values down (previous row's salary appears on the current row), while `LEAD` shifts values up (next row's salary appears on the current row). The first row has NULL for `LAG`; the last row has NULL for `LEAD`.

---

### Q12. GROUP BY vs PARTITION BY — side-by-side comparison

```sql
-- GROUP BY: collapses all rows — result has one row per department
SELECT department
FROM employees
GROUP BY department;

-- PARTITION BY: keeps all rows — every employee appears with their dept rank
SELECT *,
    RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS ranking
FROM employees;
```

> **Key insight:** `GROUP BY` answers "what are the departments?" (summary). `PARTITION BY` answers "what is each employee's rank within their department?" (row-level + group context). You cannot get both an individual employee row and a group summary in the same `GROUP BY` query — but `PARTITION BY` handles this effortlessly.

---

### Q13. Find departments where multiple employees share the same salary rank

```sql
WITH ranked AS (
    SELECT *,
        RANK() OVER(PARTITION BY department ORDER BY salary) AS ranking
    FROM employees
    WHERE department IS NOT NULL
)
SELECT department, ranking, COUNT(*) AS total_employee
FROM ranked
GROUP BY department, ranking
HAVING COUNT(*) > 1;
```

> **Why combine window functions with `GROUP BY`?** First, `RANK()` assigns ranks within each department (window function layer). Then, `GROUP BY` + `HAVING` finds which department-rank combinations have more than one employee (aggregation layer). This two-step pattern — window first, then aggregate — is a powerful and commonly tested interview technique.

---

### Q14. Display each employee along with the total salary expense of their department

```sql
SELECT *,
    SUM(salary) OVER(PARTITION BY department) AS total_salary_expense
FROM employees;
```

> **Why not `GROUP BY`?** A `GROUP BY` with `SUM()` would collapse rows — you'd lose individual employee data. Here, every employee row is preserved while also showing what their entire department costs. This is the classic "aggregate alongside detail" use case for window functions.

---

### Q15. Display a running total of salaries within each department

```sql
SELECT *,
    SUM(salary) OVER(PARTITION BY department ORDER BY employee_id) AS running_total
FROM employees;
```

> **Why add `ORDER BY` inside `OVER()`?** When you add `ORDER BY` to a `SUM()` window function, MySQL computes a **cumulative sum** — adding each row's salary to all previous rows within that partition. Without `ORDER BY`, you'd get the total for the entire partition on every row (as in Q14). The `ORDER BY` is what makes it a "running" total.

---

### Q16. Find employees earning more than the average salary of their department

```sql
SELECT *
FROM (
    SELECT *,
        AVG(salary) OVER(PARTITION BY department) AS average_salary
    FROM employees
) t
WHERE salary > average_salary;
```

> **Why not a correlated subquery?** A correlated subquery would compute the avg for every row — expensive on large tables. `AVG() OVER(PARTITION BY department)` computes it once per partition and attaches it to every row efficiently. The outer `WHERE` then filters high earners cleanly.

---

## 📌 Window Functions Cheatsheet

```sql
-- ROW_NUMBER: unique sequential number (no ties)
ROW_NUMBER() OVER(ORDER BY salary DESC)

-- RANK: same rank for ties, gaps after ties
RANK() OVER(ORDER BY salary DESC)

-- DENSE_RANK: same rank for ties, NO gaps
DENSE_RANK() OVER(ORDER BY salary DESC)

-- LAG: value from the PREVIOUS row
LAG(salary) OVER(ORDER BY employee_id)

-- LEAD: value from the NEXT row
LEAD(salary) OVER(ORDER BY employee_id)

-- SUM as running total (ORDER BY creates cumulative)
SUM(salary) OVER(PARTITION BY department ORDER BY employee_id)

-- SUM as partition total (no ORDER BY = full group total)
SUM(salary) OVER(PARTITION BY department)

-- AVG per partition (group average on every row)
AVG(salary) OVER(PARTITION BY department)
```

### Quick Decision Guide

```
Need unique row labels?                  → ROW_NUMBER()
Need ranks with gaps for ties?           → RANK()
Need ranks without gaps for ties?        → DENSE_RANK()
Need Nth highest/lowest value?           → DENSE_RANK() + WHERE ranking = N
Need previous row's value?               → LAG()
Need next row's value?                   → LEAD()
Need running total?                      → SUM() OVER(ORDER BY ...)
Need group total per row?                → SUM() OVER(PARTITION BY ...)
Need group avg per row?                  → AVG() OVER(PARTITION BY ...)
Need rank within a group?                → RANK/DENSE_RANK() OVER(PARTITION BY ...)
```

---

## 🗂️ Dataset Reference

All queries in this series use the `employer_database` with the `employees` table as the core entity.

```sql
USE employer_database;
```

| Column | Type | Notes |
|---|---|---|
| `employee_id` | INT | Primary key, sequential 1–52 |
| `first_name` | VARCHAR | — |
| `last_name` | VARCHAR | — |
| `department` | VARCHAR | Plain text, nullable |
| `salary` | DECIMAL | Used in all window function examples |
| `manager_id` | INT | Self-referencing FK |

---
