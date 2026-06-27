# Part 4: Subqueries in MySQL

## What Is a Subquery?

A **subquery** is a query written **inside another query**. The inner query runs first, and its result is passed to the outer query.

```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
--              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
--              This inner query runs first
```

---

## JOIN vs Subquery — When to Use Which?

| Situation | Use |
|-----------|-----|
| You want to **combine columns** from multiple tables | `JOIN` |
| You want the **result of one query** to act as input to another | `Subquery` |
| You want to check **existence** of a related row | `EXISTS` subquery |
| You want to filter against a **list of values** from another table | `IN` subquery |

> **Rule of thumb:** If you need data *from* another table in your SELECT columns → use JOIN. If you only need data *from* another table to *filter* your main table → a subquery often reads more naturally.

---

## Types of Subqueries

| Type | Returns | Common Operators |
|------|---------|-----------------|
| **Single Row** | Exactly one row, one column | `=`, `>`, `<`, `>=`, `<=` |
| **Multi Row** | Multiple rows, one column | `IN`, `NOT IN`, `ANY`, `ALL` |
| **Correlated** | Depends on the outer query (reruns per row) | `=`, `>`, `EXISTS` |

---

## Questions & Solutions

### Q1. Find employees whose salary is greater than the average salary of the company

**Concept:** Single Row Subquery — the inner query returns one value (`AVG`), so we compare with `>`.

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary) AS average_salary
    FROM employees
);
```

> **Why subquery here?** You can't use `WHERE salary > AVG(salary)` directly — aggregate functions aren't allowed in `WHERE`. The subquery solves this by computing the average first.

---

### Q2. Find employees earning the highest salary in the company

**Concept:** Single Row Subquery — `MAX()` returns exactly one value.

```sql
SELECT *
FROM employees
WHERE salary = (
    SELECT MAX(salary) AS highest_salary
    FROM employees
);
```

> **Why not `ORDER BY salary DESC LIMIT 1`?** That returns only one employee. This approach correctly returns *all* employees tied for the highest salary.

---

### Q3. Find employees who work in the same department as 'Aarav Sharma'

**Concept:** Multi Row Subquery with `IN` — used because `Aarav Sharma` could theoretically belong to more than one department (e.g., if there are multiple employees with the same name).

```sql
SELECT *
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    WHERE employee_name = 'Aarav Sharma'
);
```

> **Why `IN` instead of `=`?** If the inner query returns more than one row, `=` will throw an error. `IN` safely handles one or many values.

---

### Q4. Find employees whose salary is greater than all employees in the Support department

**Concept:** Multi Row Subquery with `ALL` — the employee's salary must be greater than *every* salary in the Support department.

```sql
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Support'
);
```

> **`> ALL` vs `> MAX()`?** They are logically equivalent here. `> ALL(subquery)` is often more readable when the intent is "beat everyone in this group."

---

### Q5. Find employees working in departments where at least one employee is currently on leave

**Concept:** Multi Row Subquery with `IN` — get the list of departments that have at least one person on leave, then find all employees in those departments.

```sql
SELECT *
FROM employees
WHERE department IN (
    SELECT DISTINCT department
    FROM employees
    WHERE employment_status = 'On Leave'
);
```

> **Why `DISTINCT`?** Without it, if 5 people in the Sales department are on leave, "Sales" appears 5 times in the subquery result. `DISTINCT` keeps the list clean — though `IN` handles duplicates fine, it's good practice for clarity and performance.

---

### Q6. Find employees who have received bonuses

**Concept:** Three valid approaches — choose based on what you need in your output.

**Method 1 — JOIN** *(use when you need bonus details in the output)*
```sql
SELECT e.employee_name, b.bonus_amount
FROM employees e
INNER JOIN bonuses b ON e.employee_id = b.employee_id;
```

**Method 2 — IN Subquery** *(use when you only need employee details)*
```sql
SELECT *
FROM employees
WHERE employee_id IN (
    SELECT DISTINCT employee_id
    FROM bonuses
);
```

**Method 3 — EXISTS** *(most semantically clear: "does at least one bonus row exist for this employee?")*
```sql
SELECT *
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM bonuses b
    WHERE b.employee_id = e.employee_id
);
```

> **Why `SELECT 1` in EXISTS?** EXISTS only cares whether *any* row is returned — not what columns are in it. Writing `SELECT 1` makes this intent explicit and avoids unnecessary column scanning.

---

### Q7. Find employees who never received any bonus

**Concept:** Two valid approaches using exclusion logic.

**Method 1 — LEFT JOIN with NULL check**
```sql
SELECT e.employee_name, b.bonus_id
FROM employees e
LEFT JOIN bonuses b ON e.employee_id = b.employee_id
WHERE b.bonus_id IS NULL;
```

**Method 2 — NOT EXISTS** *(most readable: "no bonus row exists for this employee")*
```sql
SELECT *
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM bonuses b
    WHERE b.employee_id = e.employee_id
);
```

> **Why prefer `NOT EXISTS` over `NOT IN`?** `NOT IN` behaves unexpectedly when the subquery contains `NULL` values — it returns no rows at all. `NOT EXISTS` handles NULLs safely. **Always prefer `NOT EXISTS` over `NOT IN` for exclusion checks.**

---

### Q8. Find employees whose salary is greater than the average salary of their own department

**Concept:** Correlated Subquery — the inner query references the outer query (`e.department`), so it re-executes *for each row* of the outer query.

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(salary) AS average_salary
    FROM employees e1
    WHERE e1.department = e.department  -- reference to outer query
);
```

> **Why is this correlated?** The subquery can't run independently — it depends on `e.department` from the outer row being evaluated. MySQL runs this inner query once per employee row, substituting that employee's department each time.
>
> **Performance note:** Correlated subqueries can be slow on large tables since they execute N times (once per row). Consider rewriting with a JOIN + GROUP BY for better performance at scale.

---

### Q9. Find employees who are assigned to at least one project

**Concept:** EXISTS subquery — checks for the *presence* of a matching row in `employee_projects`.

**Method 1 — EXISTS**
```sql
SELECT *
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM employee_projects ep
    WHERE ep.employee_id = e.employee_id
);
```

**Method 2 — INNER JOIN**
```sql
SELECT e.employee_name, ep.project_id
FROM employees e
INNER JOIN employee_projects ep ON e.employee_id = ep.employee_id;
```

> **JOIN vs EXISTS here?** If an employee has 3 projects, JOIN returns 3 rows for that employee. EXISTS returns them only once. Use EXISTS when you just want to know *if* they're assigned, without duplicating rows.

---

### Q10. Find employees who earn more than their manager

**Concept:** Correlated Subquery — for each employee, the subquery looks up *that employee's* manager's salary using `e.manager_id`.

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT m.salary
    FROM employees m
    WHERE m.employee_id = e.manager_id
);
```

> **Why a self-referencing subquery?** Managers live in the same `employees` table (via `manager_id`). The correlated subquery lets us look up the manager's salary dynamically for each employee row.
>
> This can also be written as a self-join: `JOIN employees m ON e.manager_id = m.employee_id WHERE e.salary > m.salary` — both approaches work; the subquery version often reads more naturally.

---

## EXISTS vs IN — Key Differences

| | `IN` | `EXISTS` |
|--|------|----------|
| **What it does** | Compares a value against a **list of values** | Checks whether a subquery **returns at least one row** |
| **How it works** | Runs the subquery once, builds a list, then checks membership | Runs the subquery once per outer row, stops as soon as one match is found |
| **NULLs** | ⚠️ Dangerous — `NOT IN` with NULLs returns no rows | ✅ Safe — NULLs don't affect the result |
| **Best for** | Filtering against a **known, small list** | Checking **existence** of a related row |
| **Performance** | Faster when the subquery result set is small | Faster when the subquery result set is large (short-circuits early) |

```sql
-- IN: did this employee_id appear in the bonuses table?
WHERE employee_id IN (SELECT employee_id FROM bonuses)

-- EXISTS: does at least one bonus row exist for this employee?
WHERE EXISTS (SELECT 1 FROM bonuses b WHERE b.employee_id = e.employee_id)
```

> **Practical advice:** Use `IN` when your subquery returns a simple list of IDs or values. Use `EXISTS` when you're checking for the *presence* of a related record — especially in correlated scenarios. **Always use `NOT EXISTS` instead of `NOT IN`** to avoid NULL-related bugs.

---

## SQL Execution Order Reference

```
FROM → WHERE (subquery runs here) → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Subqueries in the `WHERE` clause run **before** the outer query filters its rows — this is why they work as dynamic filters.

---

## Quick Reference — Which Subquery Type to Use?

| Scenario | Subquery Type | Operator |
|----------|--------------|----------|
| Compare against one computed value (AVG, MAX) | Single Row | `=`, `>`, `<` |
| Compare against values from another table | Multi Row | `IN`, `NOT IN` |
| Must beat every value in a group | Multi Row | `ALL` |
| Must beat at least one value in a group | Multi Row | `ANY` |
| Check if a related row exists | Correlated | `EXISTS` |
| Check if no related row exists | Correlated | `NOT EXISTS` |
| Filter per-row based on the outer query | Correlated | any |
