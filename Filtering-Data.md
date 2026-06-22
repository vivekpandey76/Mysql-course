# 🗄️ MySQL Data Filtering — Query Reference

A structured reference for filtering, sorting, and retrieving data from a MySQL `employees` table. Covers `WHERE`, `LIKE`, `IN`, `BETWEEN`, `IS NULL`, `ORDER BY`, and `LIMIT`.

---

## 📋 Table of Contents

- [Table Schema](#-table-schema)
- [Basic SELECT](#-basic-select)
- [Filtering with WHERE](#-filtering-with-where)
- [Pattern Matching with LIKE](#-pattern-matching-with-like)
- [NULL Checks](#-null-checks)
- [Sorting with ORDER BY](#-sorting-with-order-by)
- [Limiting Results](#-limiting-results)
- [Combined Queries](#-combined-queries)

---

## 🏗️ Table Schema

> Assumed structure of the `employees` table used throughout this guide.

| Column              | Type         | Description                        |
|---------------------|--------------|------------------------------------|
| `employee_id`       | INT          | Primary key                        |
| `employee_name`     | VARCHAR      | Full name of the employee          |
| `department`        | VARCHAR      | Department (IT, HR, Finance, etc.) |
| `salary`            | DECIMAL      | Monthly salary in ₹                |
| `email`             | VARCHAR      | Employee email address             |
| `employment_status` | VARCHAR      | Active / Inactive / On Leave       |
| `manager_id`        | INT          | FK to manager's employee_id        |

---

## 📌 Basic SELECT

### Q1 — Retrieve all employee details

```sql
SELECT *
FROM employees;
```

### Q2 — Retrieve name, department, and salary only

```sql
SELECT employee_name, department, salary
FROM employees;
```

### Q3 — Find all unique departments

```sql
SELECT DISTINCT department
FROM employees
WHERE department IS NOT NULL;
```

> `DISTINCT` removes duplicate values. `IS NOT NULL` excludes rows with no department assigned.

---

## 🔍 Filtering with WHERE

### Q4 — Employees in the IT department

```sql
SELECT *
FROM employees
WHERE department = 'IT';
```

### Q5 — Employees earning more than ₹80,000

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

### Q6 — IT employees earning more than ₹80,000

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 80000;
```

> `AND` requires **both** conditions to be true.

### Q7 — Employees in IT or Finance

```sql
-- Using OR
SELECT *
FROM employees
WHERE department = 'IT' OR department = 'Finance';

-- Using IN (cleaner for multiple values)
SELECT *
FROM employees
WHERE department IN ('IT', 'Finance');
```

### Q8 — Employees who are NOT Active

```sql
-- Using not-equal operator
SELECT *
FROM employees
WHERE employment_status <> 'Active';

-- Using NOT keyword
SELECT *
FROM employees
WHERE NOT employment_status = 'Active';
```

### Q9 — Employees in IT, HR, or Finance

```sql
-- Using IN
SELECT *
FROM employees
WHERE department IN ('IT', 'HR', 'Finance');

-- Using OR (equivalent, verbose)
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR'
   OR department = 'Finance';
```

> ✅ Prefer `IN` over chained `OR` — it's cleaner and easier to maintain.

### Q10 — Employees with salary between ₹50,000 and ₹80,000

```sql
-- Using BETWEEN (inclusive)
SELECT *
FROM employees
WHERE salary BETWEEN 50000 AND 80000;

-- Using comparison operators (equivalent)
SELECT *
FROM employees
WHERE salary >= 50000 AND salary <= 80000;
```

---

## 🔎 Pattern Matching with LIKE

| Wildcard | Meaning                     |
|----------|-----------------------------|
| `%`      | Zero or more characters     |
| `_`      | Exactly one character       |

### Q11 — Employees whose name starts with 'A'

```sql
SELECT *
FROM employees
WHERE employee_name LIKE 'A%';
```

### Q12 — Employees with a Gmail address

```sql
SELECT *
FROM employees
WHERE email LIKE '%@gmail.com';
```

### Q13 — Active employees whose name starts with 'S'

```sql
SELECT *
FROM employees
WHERE employment_status = 'Active'
  AND employee_name LIKE 'S%';
```

---

## 🚫 NULL Checks

### Q14 — Employees with no manager assigned

```sql
SELECT *
FROM employees
WHERE manager_id IS NULL;
```

### Q15 — Employees with an email on record

```sql
SELECT *
FROM employees
WHERE email IS NOT NULL;
```

> ⚠️ Never use `= NULL` — it always returns false. Always use `IS NULL` / `IS NOT NULL`.

---

## 📊 Sorting with ORDER BY

### Q16 — Sort by salary (lowest to highest)

```sql
SELECT salary, employee_name
FROM employees
WHERE salary IS NOT NULL
ORDER BY salary ASC;
```

### Q17 — Sort by salary (highest to lowest)

```sql
SELECT salary, employee_name
FROM employees
WHERE salary IS NOT NULL
ORDER BY salary DESC;
```

### Q18 — Sort by department, then salary descending

```sql
SELECT salary, department, employee_name
FROM employees
WHERE department IS NOT NULL
ORDER BY department ASC, salary DESC;
```

> Multiple columns in `ORDER BY` are evaluated left to right. Here: sort alphabetically by department first, then within each department by salary descending.

---

## 🔢 Limiting Results

### Q19 — Top 5 highest-paid employees

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

---

## 🧩 Combined Queries

### Q20 — Top 3 highest-paid IT/Finance employees earning ₹70K–₹1L

```sql
SELECT *
FROM employees
WHERE department IN ('IT', 'Finance')
  AND salary BETWEEN 70000 AND 100000
ORDER BY salary DESC
LIMIT 3;
```

> This query chains `IN`, `BETWEEN`, `ORDER BY`, and `LIMIT` — a common pattern for ranked, filtered reports.

---

## 🧠 Quick Concept Cheatsheet

| Clause / Keyword  | Purpose                                      |
|-------------------|----------------------------------------------|
| `WHERE`           | Filter rows based on condition               |
| `AND` / `OR`      | Combine multiple conditions                  |
| `NOT`             | Negate a condition                           |
| `IN (...)`        | Match against a list of values               |
| `BETWEEN a AND b` | Range check (inclusive on both ends)         |
| `LIKE`            | Pattern match (`%` = any chars, `_` = one)   |
| `IS NULL`         | Check for missing/null values                |
| `DISTINCT`        | Return unique values only                    |
| `ORDER BY`        | Sort results (`ASC` default, `DESC` reverse) |
| `LIMIT n`         | Return only the first `n` rows               |

---

