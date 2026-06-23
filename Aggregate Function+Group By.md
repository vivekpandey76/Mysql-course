# 📊 Part 2 — Aggregate Functions + GROUP BY + HAVING

> **MySQL Course | employer_database**  
> This module covers how to summarize and group data using aggregate functions, GROUP BY, and HAVING — essential tools for data analysis and reporting.

---

## 🧠 Core Concepts

### What is an Aggregate Function?
We use **aggregate functions** when we want to perform a calculation on **multiple rows** and return a **single value**.

| Function | Description |
|----------|-------------|
| `COUNT()` | Counts the number of rows |
| `SUM()` | Returns the total sum of a column |
| `AVG()` | Returns the average value of a column |
| `MAX()` | Returns the highest value |
| `MIN()` | Returns the lowest value |

---

### What is GROUP BY?
We use **GROUP BY** when we want to **divide rows into groups** and perform calculations on **each group separately**.

> Without GROUP BY, aggregate functions collapse all rows into one result.  
> With GROUP BY, aggregate functions run independently for each group.

---

### What is HAVING?
We use **HAVING** to **filter groups** after `GROUP BY` and aggregate functions have been applied.

| Clause | Filters | Runs When |
|--------|---------|-----------|
| `WHERE` | Individual rows | Before grouping |
| `HAVING` | Groups | After grouping |

---

## 🔄 SQL Order of Execution

Understanding **how MySQL processes a query** is key to writing correct SQL:

```
FROM      →  Choose the table(s) and perform joins
WHERE     →  Filter individual rows
GROUP BY  →  Create groups from the remaining rows
HAVING    →  Filter groups after aggregation
SELECT    →  Choose the columns to return
DISTINCT  →  Remove duplicate rows from the result
ORDER BY  →  Sort the final result
LIMIT     →  Restrict the number of rows returned
```

> 💡 **Key Insight:** `WHERE` runs *before* grouping, so you cannot use aggregate functions inside `WHERE`. That's exactly what `HAVING` is for.

---

## 📝 COUNT(*) vs COUNT(column) vs COUNT(1)

This is one of the most commonly misunderstood concepts in SQL:

| Expression | What It Counts | Includes NULLs? |
|------------|----------------|-----------------|
| `COUNT(*)` | All rows in the result set | ✅ Yes |
| `COUNT(1)` | All rows — `1` is just a constant value, every row qualifies | ✅ Yes |
| `COUNT(column)` | Only rows where the specified column is **NOT NULL** | ❌ No |

> `COUNT(*)` and `COUNT(1)` produce identical results. Use `COUNT(*)` as the standard convention.

---

## ✅ Practice Questions

### Section 1 — Basic Aggregate Functions

---

#### Q1: Find the total number of employees in the company.

**Goal:** Count every row in the `employees` table, regardless of NULLs.

```sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

> 💡 `COUNT(*)` counts all rows including those with NULL values in any column. Perfect for getting the total headcount.

---

#### Q2: Find how many employees have a manager assigned.

**Goal:** Count only employees where `manager_id` is not NULL.

```sql
SELECT COUNT(manager_id) AS total_manager_assigned
FROM employees;
```

> 💡 `COUNT(column)` skips NULLs. So if some employees have no manager (`manager_id IS NULL`), those rows are excluded from the count — giving us only the employees who actually have a manager assigned.

---

#### Q3: Understanding COUNT(*) vs COUNT(column) vs COUNT(1)

```sql
-- Counts all rows (including rows where any column is NULL)
SELECT COUNT(*) FROM employees;

-- Also counts all rows (1 is a constant, so every row qualifies)
SELECT COUNT(1) FROM employees;

-- Counts only rows where manager_id is NOT NULL
SELECT COUNT(manager_id) FROM employees;
```

> 💡 Think of it this way: `COUNT(*)` asks "how many rows exist?", while `COUNT(column)` asks "how many rows have a value in this column?"

---

#### Q4: Find the total salary expense of the company.

**Goal:** Add up all salaries across all employees.

```sql
SELECT SUM(salary) AS total_salary_expense
FROM employees;
```

> 💡 `SUM()` ignores NULL values automatically. Only rows with an actual salary value are included in the total.

---

#### Q5: Find the average salary of employees.

**Goal:** Calculate the mean salary across all employees.

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

> 💡 `AVG()` = `SUM(salary) / COUNT(salary)`. It also ignores NULLs — it won't treat a NULL salary as 0.

---

#### Q6: Find the highest and lowest salary in the company.

**Goal:** Return both extremes in a single query.

```sql
SELECT 
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM employees;
```

> 💡 You can use multiple aggregate functions in a single `SELECT`. This returns one row with two summary values.

---

### Section 2 — GROUP BY

---

#### Q7: Find the total number of employees in each department.

**Goal:** Break down headcount by department.

```sql
SELECT department, COUNT(*) AS total_department_employee
FROM employees
WHERE department IS NOT NULL
GROUP BY department;
```

> 💡 `WHERE department IS NOT NULL` filters out rows with no department *before* grouping happens. Without this, NULL would appear as its own group.

---

#### Q8: Find the average salary of employees in each city.

**Goal:** Get salary averages per city location.

```sql
SELECT city, AVG(salary) AS average_salary
FROM employees
GROUP BY city;
```

> 💡 Each city becomes a group, and `AVG()` runs independently within each group. The result has one row per city.

---

#### Q9: Find the total number of Active employees in each department.

**Goal:** Count only Active employees, grouped by department.

```sql
SELECT department, COUNT(*) AS total_active_employee
FROM employees
WHERE employment_status = 'Active'
GROUP BY department;
```

> 💡 `WHERE` filters rows first — only `Active` employees enter the grouping step. So `COUNT(*)` inside each group only counts active employees.

---

#### Q10: Find the total salary expense of employees working in Mumbai for each department.

**Goal:** Filter by city first, then group by department.

```sql
SELECT department, SUM(salary) AS total_salary_expense
FROM employees
WHERE city = 'Mumbai'
GROUP BY department;
```

> 💡 First, `WHERE city = 'Mumbai'` keeps only Mumbai employees. Then `GROUP BY department` splits them into department groups, and `SUM(salary)` totals each group.

---

### Section 3 — HAVING

---

#### Q11: Find departments having more than 5 employees.

**Goal:** Filter groups based on count — use HAVING, not WHERE.

```sql
-- Using alias in HAVING (MySQL supports this)
SELECT department, COUNT(*) AS total_employee_department
FROM employees
GROUP BY department
HAVING total_employee_department > 5;

-- Using the aggregate function directly in HAVING (standard SQL)
SELECT department, COUNT(*) AS total_employee_department
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

> 💡 Both versions work in MySQL. Why can't we use `WHERE COUNT(*) > 5`? Because `WHERE` runs *before* grouping, and at that point, aggregates haven't been calculated yet. `HAVING` runs *after* aggregation.

---

#### Q12: Find cities whose average salary is greater than ₹70,000.

**Goal:** Filter groups by an aggregate condition.

```sql
-- Using alias in HAVING
SELECT city, AVG(salary) AS average_salary
FROM employees
GROUP BY city
HAVING average_salary > 70000;

-- Using the aggregate function directly
SELECT city, AVG(salary) AS average_salary
FROM employees
GROUP BY city
HAVING AVG(salary) > 70000;
```

> 💡 Notice the pattern: `GROUP BY city` creates one group per city, `AVG(salary)` calculates the average for each, and `HAVING` keeps only the cities that pass the condition.

---

#### Q13: Find departments whose average salary lies between ₹60,000 and ₹80,000.

**Goal:** Use HAVING with a range condition.

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING average_salary BETWEEN 60000 AND 80000;
```

> 💡 `BETWEEN` is inclusive on both ends. Departments with an average of exactly ₹60,000 or exactly ₹80,000 are included.

---

#### Q14: Find departments having at least 4 employees who are currently Active.

**Goal:** Combine WHERE (row filter) with HAVING (group filter).

```sql
SELECT department, COUNT(*) AS total_employee_department
FROM employees
WHERE employment_status = 'Active'
GROUP BY department
HAVING COUNT(*) >= 4;
```

> 💡 This query demonstrates using `WHERE` and `HAVING` together:  
> - `WHERE` filters out non-active employees *before* grouping  
> - `HAVING` filters out departments with fewer than 4 active employees *after* grouping

---

### Section 4 — Finding Duplicates

---

#### Q15: Find Duplicate Records By Email

**Goal:** Identify email addresses that appear more than once.

```sql
SELECT email, COUNT(*) AS duplicate_email_count
FROM employees
GROUP BY email
HAVING duplicate_email_count > 1;
```

> 💡 This is a classic duplicate-detection pattern. `GROUP BY email` groups all rows sharing the same email, `COUNT(*)` counts how many rows are in each group, and `HAVING > 1` keeps only the emails that appear more than once — meaning they're duplicated.

---

## 🗂️ Quick Reference Cheatsheet

```sql
-- Basic aggregate on entire table
SELECT COUNT(*), SUM(col), AVG(col), MAX(col), MIN(col)
FROM table_name;

-- Aggregate per group
SELECT group_col, COUNT(*)
FROM table_name
GROUP BY group_col;

-- Filter rows BEFORE grouping
SELECT group_col, COUNT(*)
FROM table_name
WHERE condition
GROUP BY group_col;

-- Filter groups AFTER aggregation
SELECT group_col, COUNT(*)
FROM table_name
GROUP BY group_col
HAVING COUNT(*) > n;

-- Combine both
SELECT group_col, COUNT(*)
FROM table_name
WHERE row_condition
GROUP BY group_col
HAVING COUNT(*) > n;
```

---

## 📌 Key Rules to Remember

- **`WHERE` vs `HAVING`** — `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation. You cannot use aggregate functions in `WHERE`.
- **`COUNT(*)` vs `COUNT(col)`** — `COUNT(*)` counts all rows; `COUNT(col)` only counts non-NULL values.
- **Columns in SELECT with GROUP BY** — Every column in `SELECT` that is *not* an aggregate must appear in `GROUP BY`.
- **Alias in HAVING** — MySQL allows using a `SELECT` alias in `HAVING`, but standard SQL requires repeating the aggregate expression.
- **NULL in groups** — NULLs form their own group in `GROUP BY`. Use `WHERE col IS NOT NULL` to exclude them.

---


