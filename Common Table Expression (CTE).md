# Part 6: Common Table Expressions (CTEs) in MySQL

A **CTE (Common Table Expression)** is a temporary, named result set that exists only for the duration of a single query. Think of it as giving a subquery a name and a "scratchpad" — you write it once at the top, then reference it like a regular table in the rest of your query.

CTEs don't make a query do anything fundamentally different from a subquery — they make it **easier for a human to read, debug, and reuse**.

---

## Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT *
FROM cte_name;
```

You define the CTE with `WITH ... AS (...)`, then immediately use it in the `SELECT` that follows — almost like declaring a variable before using it.

---

## CTE vs Subquery — Same Logic, Different Readability

Both queries below return the **exact same result**: employees whose salary increased compared to the previous employee (ordered by `employee_id`).

**Subquery version:**
```sql
SELECT *
FROM (
    SELECT *, LAG(salary) OVER (ORDER BY employee_id) AS previous_salary
    FROM employees
) t
WHERE salary > previous_salary;
```

**CTE version:**
```sql
WITH previous_employee_salary AS (
    SELECT *, LAG(salary) OVER (ORDER BY employee_id) AS previous_salary
    FROM employees
)
SELECT *
FROM previous_employee_salary
WHERE salary > previous_salary;
```

> 💡 **Teaching Tip:** Notice the subquery is nested *inside* the `FROM` clause, forcing you to read inside-out. The CTE pulls that same logic *above* the main query and gives it a name (`previous_employee_salary`), so you read it top-to-bottom like a story: "first calculate this, then filter on it." As queries grow more complex, this difference in readability becomes huge.

**Rule of thumb:**
- **Subquery** → better for simple, one-off queries.
- **CTE** → better for complex queries because it improves readability and reusability.

---

## Practice Questions

### Q1. Using a CTE, find departments having more than 5 employees.

**Without CTE (plain GROUP BY + HAVING):**
```sql
SELECT department, COUNT(*) AS total_employee
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

**With CTE:**
```sql
WITH department_count AS (
    SELECT department, COUNT(*) AS total_employee
    FROM employees
    GROUP BY department
)
SELECT *
FROM department_count
WHERE total_employee > 5;
```

> 💡 **Teaching Tip:** You might wonder — why use a CTE here when `HAVING` already does the job in one step? Because `HAVING` filters at *aggregation time*, while wrapping it in a CTE lets you treat `total_employee` as a regular column afterward — for further joins, additional filters, or reuse later in the same query. For this simple case `HAVING` alone is perfectly fine; the CTE version exists to demonstrate the pattern you'll need once queries get more layered (see Q2 onward).

---

### Q2. Using Multiple CTEs, find departments whose average salary is greater than the company average salary.

```sql
WITH department_average AS (
    SELECT department, AVG(salary) AS average_salary
    FROM employees
    WHERE department IS NOT NULL
    GROUP BY department
),
company_average AS (
    SELECT AVG(salary) AS average_salary
    FROM employees
)
SELECT d.*, c.*
FROM department_average d
CROSS JOIN company_average c
WHERE d.average_salary > c.average_salary;
```

> 💡 **Teaching Tip:** This is where CTEs really start to shine. We need **two separate aggregations** — one grouped by department, one across the whole company — and then we need to compare them side by side. Doing this with nested subqueries would mean writing the company-average calculation inline and repeating it; with CTEs, each calculation gets its own clear name (`department_average`, `company_average`) and we simply `CROSS JOIN` them together. A `CROSS JOIN` is safe and correct here specifically because `company_average` returns exactly **one row** — every department row gets paired with that single company-wide number.

---

### Q3. Using a CTE, find employees who have received more than one bonus.

**First pass (filter after the join):**
```sql
WITH employee_bonus AS (
    SELECT employee_id, COUNT(*) AS total_bonus_received
    FROM bonuses
    GROUP BY employee_id
)
SELECT e.employee_name, b.total_bonus_received
FROM employees e
INNER JOIN employee_bonus b
    ON e.employee_id = b.employee_id
WHERE total_bonus_received > 1;
```

**Better version (filter inside the CTE):**
```sql
WITH employee_bonus AS (
    SELECT employee_id, COUNT(*) AS total_bonus_received
    FROM bonuses
    GROUP BY employee_id
    HAVING COUNT(*) > 1
)
SELECT e.employee_name, b.total_bonus_received
FROM employees e
INNER JOIN employee_bonus b
    ON e.employee_id = b.employee_id;
```

> 💡 **Teaching Tip:** Both versions return the same result, but the second is the better habit to build. Filtering with `HAVING COUNT(*) > 1` *inside* the CTE means MySQL only joins employees who already qualify — the CTE itself is smaller before it ever touches the `employees` table. Filtering with `WHERE total_bonus_received > 1` *after* the join means every employee with at least one bonus gets joined first, and the unwanted rows are discarded afterward. As a general principle: **filter as early as possible** in your CTE so later steps work with the smallest, most relevant data set.

---

### Q4. Using a CTE, find the top 2 highest-paid employees from each department.

**Subquery version:**
```sql
SELECT *
FROM (
    SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM employees
    WHERE department IS NOT NULL
) t
WHERE ranking <= 2;
```

**CTE version:**
```sql
WITH ranked AS (
    SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM employees
    WHERE department IS NOT NULL
)
SELECT *
FROM ranked
WHERE ranking <= 2;
```

> 💡 **Teaching Tip:** This is the single most common reason CTEs and window functions are taught together: **you can't filter directly on a window function's output in the same `SELECT` it's defined in** (you can't write `WHERE ranking <= 2` next to `RANK() OVER (...)` — MySQL evaluates `WHERE` before window functions are calculated). Wrapping the window function in a CTE (or subquery) solves this by calculating the ranking first, then filtering on it as a regular column in the outer query.

---

### Q5. Using Multiple CTEs, find employees whose salary is greater than the average salary of their department, and display their department rank.

```sql
WITH department_average AS (
    SELECT *, AVG(salary) OVER (PARTITION BY department) AS average_salary
    FROM employees
    WHERE department IS NOT NULL
),
department_ranking AS (
    SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM department_average
)
SELECT *
FROM department_ranking
WHERE salary > average_salary;
```

> 💡 **Teaching Tip:** This is **chained CTEs** — each CTE builds on the one before it. `department_average` adds an `average_salary` column using a window function (note: `AVG(...) OVER (PARTITION BY department)` keeps every row instead of collapsing rows like `GROUP BY` would). `department_ranking` then takes that *already-enriched* result and adds a `ranking` column on top of it. By the time we reach the final `SELECT`, both `average_salary` and `ranking` already exist as plain columns, so the final filter is a simple, readable comparison. This step-by-step layering — each CTE adding one clear piece of logic — is exactly why CTEs are preferred for complex, multi-stage queries.

---

## CTE vs Subquery — Quick Reference

| CTE                                  | Subquery                          |
|---------------------------------------|------------------------------------|
| More readable                         | Can become difficult to read       |
| Reusable within the query             | Not easily reusable                |
| Better for complex queries            | Better for simple queries          |
| Commonly used with Window Functions   | Mostly used for simple filtering   |

---

## When to Use a CTE

1. **To simplify complex queries** — break a tangled nested subquery into a clear, named step.
2. **To improve query readability** — read top-to-bottom instead of inside-out.
3. **To break a query into smaller steps** — each CTE handles one piece of logic.
4. **To reuse intermediate results** — reference the same CTE more than once in the outer query.
5. **To work with Window Functions more efficiently** — calculate the window function in a CTE, then filter/aggregate on it afterward (since you can't filter on a window function directly in the same `SELECT`).

---

## Cheatsheet

```sql
-- Single CTE
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name;

-- Multiple CTEs (comma-separated)
WITH cte_one AS (
    SELECT ...
),
cte_two AS (
    SELECT ...
)
SELECT *
FROM cte_one
JOIN cte_two ON ...;

-- Chained CTEs (each builds on the previous one)
WITH step_one AS (
    SELECT ...
),
step_two AS (
    SELECT * FROM step_one ...
)
SELECT * FROM step_two;
```
