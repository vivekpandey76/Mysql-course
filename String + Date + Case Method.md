# Part 7: String + Date + CASE Functions in MySQL

Welcome to **Part 7** of the MySQL Course! In this section, we move from *filtering and aggregating* data to *transforming and presenting* it — using **String Functions**, **Date Functions**, and the **CASE Statement**.

By the end of this part, you'll be able to:
- Clean, format, and extract pieces of text data
- Perform date math (differences, formatting, extracting parts)
- Build conditional logic directly inside your SQL queries

We'll continue using our shared `employer_database` dataset (`employees`, `bonuses`, `projects`).

---

## Table of Contents
- [1. String Functions](#1-string-functions)
- [2. Date Functions](#2-date-functions)
- [3. CASE Statement](#3-case-statement)
- [4. Practice Questions](#4-practice-questions)
- [5. Cheatsheet](#5-cheatsheet)

---

## 1. String Functions

String functions let you **manipulate and format text data** — cleaning up messy inputs, extracting parts of a string, or combining multiple columns into one readable output.

### 1.1 `UPPER()` / `LOWER()`
Converts text to all uppercase or all lowercase.

```sql
SELECT UPPER(employee_name), LOWER(employee_name)
FROM employees;
```

> **Why?**
> Case-insensitive comparisons and clean display formatting both rely on this. For example, `WHERE UPPER(city) = 'MUMBAI'` guards against a user typing `mumbai`, `Mumbai`, or `MUMBAI` — they'll all match.

### 1.2 `LENGTH()`
Returns the number of **characters** (technically bytes) in a string.

```sql
SELECT employee_name, LENGTH(employee_name) AS name_length
FROM employees;
```

> **Why?**
> Useful for validation — e.g., flagging names that are suspiciously short, or checking if a phone number has the expected number of digits.

### 1.3 `CONCAT()`
Joins two or more strings into one.

```sql
SELECT CONCAT(employee_name, ' works in ', city) AS summary
FROM employees;
```

> **Why?**
> Raw tables store data in separate columns for good reason (normalization), but humans read *sentences*, not columns. `CONCAT()` is how you turn structured data back into a readable message — for reports, emails, or labels.

### 1.4 `SUBSTRING()`
Extracts a portion of a string, given a starting position and length.

```sql
SUBSTRING(string, start_position, length)

SELECT SUBSTRING('MySQL Course', 1, 5); -- 'MySQL'
```

> **Why?**
> Position-based extraction is essential when data follows a fixed pattern — like pulling the first 4 digits of a product code, or the area code from a phone number.

### 1.5 `LEFT()` / `RIGHT()`
Grabs a fixed number of characters from the **start** or **end** of a string.

```sql
SELECT LEFT('Raj Sharma', 3);  -- 'Raj'
SELECT RIGHT('Raj Sharma', 6); -- 'Sharma'
```

> **Why?**
> `LEFT()`/`RIGHT()` are simpler versions of `SUBSTRING()` for the common case of "just give me the first/last N characters" — no need to specify a starting position.

### 1.6 `REPLACE()`
Finds and replaces all occurrences of a substring.

```sql
SELECT REPLACE('raj.sharma@oldcompany.com', 'oldcompany', 'newcompany');
```

> **Why?**
> Extremely common in data cleaning — fixing typos across a column, standardizing formats (e.g., replacing `-` with `/` in dates stored as text), or masking sensitive substrings.

### 1.7 `TRIM()`
Removes leading and trailing spaces (whitespace) from a string.

```sql
SELECT TRIM('   Raj Sharma   ');  -- 'Raj Sharma'
```

> **Why?**
> Data entered manually (via forms, spreadsheets, imports) often has stray spaces. Untrimmed strings silently break comparisons — `' Raj' = 'Raj'` is `FALSE` in SQL, even though they look identical to the human eye.

### 1.8 `SUBSTRING_INDEX()`
Extracts a portion of a string **before or after** a specified occurrence of a delimiter.

```sql
SUBSTRING_INDEX(string, delimiter, occurrence)

SELECT SUBSTRING_INDEX('raj.sharma@company.com', '@', 1); -- 'raj.sharma'
SELECT SUBSTRING_INDEX('raj.sharma@company.com', '@', -1); -- 'company.com'
```

> **Why?**
> This is the go-to function for splitting delimited data (emails, full names, file paths) **without regex**. A **positive** occurrence counts from the left; a **negative** occurrence counts from the right. This distinction matters a lot — `1` gives you everything *before* the first delimiter, while `-1` gives you everything *after* the last one.

---

## 2. Date Functions

Date functions let you **calculate durations, extract parts of a date, and reformat how dates are displayed.**

### 2.1 `CURDATE()`
Returns today's date.

```sql
SELECT CURDATE();
```

> **Why?**
> Almost every "how recent" or "how old" calculation starts by anchoring to today's date. It's the reference point for relative queries like "employees who joined this year."

### 2.2 `DATEDIFF()`
Returns the difference between two dates **in days**.

```sql
SELECT DATEDIFF(CURDATE(), hire_date) AS days_employed
FROM employees;
```

> **Why?**
> `DATEDIFF()` always returns days — it doesn't know about months or years. If you need the difference in a different unit (months, years, hours), you need `TIMESTAMPDIFF()` instead (see below).

### 2.3 `DATE_FORMAT()`
Formats a date into a custom, human-readable string.

```sql
SELECT DATE_FORMAT(hire_date, '%d-%b-%Y') AS formatted_date
FROM employees;
```

**Common format specifiers:**

| Specifier | Meaning              | Example |
|-----------|----------------------|---------|
| `%d`      | Day (01–31)          | 07      |
| `%m`      | Month number (01–12) | 03      |
| `%b`      | Short month name     | Jan     |
| `%M`      | Full month name      | January |
| `%Y`      | 4-digit year         | 2025    |
| `%y`      | 2-digit year         | 25      |

> **Why?**
> Dates are stored internally in a fixed `YYYY-MM-DD` format for consistency and correct sorting — but that's rarely the format you want to *display* to a user. `DATE_FORMAT()` separates "how it's stored" from "how it's shown."

### 2.4 `YEAR()` / `MONTH()` / `MONTHNAME()` / `DAYNAME()`
Extracts individual parts of a date.

```sql
SELECT
    YEAR(hire_date)      AS hire_year,
    MONTH(hire_date)     AS hire_month_number,
    MONTHNAME(hire_date) AS hire_month_name,
    DAYNAME(hire_date)   AS hire_day_name
FROM employees;
```

> **Why?**
> These are essential for **grouping and trend analysis** — e.g., "how many employees joined each month" (`GROUP BY MONTHNAME(hire_date)`), or checking if a business event happened on a weekend (`DAYNAME(order_date) IN ('Saturday','Sunday')`).

### 2.5 `TIMESTAMPDIFF()`
Returns the difference between two dates in a **unit you specify** — days, months, or years.

```sql
TIMESTAMPDIFF(unit, start_date, end_date)

SELECT TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_employed
FROM employees;
```

> **Why?**
> Unlike `DATEDIFF()`, which is locked to days, `TIMESTAMPDIFF()` calculates *calendar-aware* differences. `TIMESTAMPDIFF(YEAR, ...)` correctly accounts for whether the person has actually completed a full year, not just divided the day-count by 365 — this matters for accuracy in tenure or age calculations.

### 2.6 `DATE_SUB()`
Subtracts a specified time interval from a date.

```sql
SELECT DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```

> **Why?**
> `DATE_SUB()` and the shorthand `CURDATE() - INTERVAL 30 DAY` do the same thing — both are valid. `DATE_SUB()` reads more explicitly for beginners, while the shorthand is more common in real-world queries once you're comfortable.

---

## 3. CASE Statement

The `CASE` statement lets you build **conditional logic directly inside SQL** — similar to `if / else if / else` in programming languages.

### Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN condition3 THEN result3
    ELSE default_result
END
```

> **Why?**
> Without `CASE`, you'd have to pull raw data into your application code and apply the logic there. `CASE` lets you **categorize, label, or transform** data at the database level — which is faster, keeps logic close to the data, and means every application reading from that query sees consistent categorization.

**Key rules:**
- Conditions are checked **top to bottom**, and the **first match wins** — later conditions are never checked once one is true.
- `ELSE` is optional. If omitted and no condition matches, the result is `NULL`.
- `CASE` can be used **anywhere an expression is allowed** — in `SELECT`, `WHERE`, `ORDER BY`, and even inside `SUM()`/`COUNT()`.

---

## 4. Practice Questions

All queries use the `employer_database` (`employees`, `bonuses`, `projects` tables).

### Q1. Display employee name in uppercase along with the length of the employee name.

```sql
SELECT
    UPPER(employee_name) AS employee_name,
    LENGTH(employee_name) AS employee_length
FROM employees;
```

---

### Q2. Display employee details in the format: `Raj Sharma (raj.sharma@company.com) - Mumbai`

```sql
SELECT
    CONCAT(employee_name, ' (', email, ') ', '- ', city) AS employee_name_format
FROM employees;
```

---

### Q3. Extract the username from employee email addresses.

```sql
SELECT
    SUBSTRING_INDEX(email, '@', 1) AS userName,
    email
FROM employees;
```

> **Why?**
> Occurrence `1` grabs everything to the *left* of the first `@` — the username portion.

---

### Q4. Find employees whose email domain is `gmail.com`.

```sql
-- Approach 1: Pattern matching
SELECT * FROM employees
WHERE email LIKE '%gmail.com';

-- Approach 2: Exact domain extraction
SELECT * FROM employees
WHERE SUBSTRING_INDEX(email, '@', -1) = 'gmail.com';
```

> **Why two approaches?**
> `LIKE '%gmail.com'` is quick to write but can false-positive on something like `notgmail.com` if data is messy. `SUBSTRING_INDEX(email, '@', -1)` isolates the **exact domain** after the `@`, so the comparison is precise. Prefer the second approach when accuracy matters more than brevity.

---

### Q5. Display employee initials. e.g., `Raj Sharma → RS`, `Priya Mehta → PM`

```sql
SELECT
    CONCAT(
        LEFT(TRIM(employee_name), 1),
        LEFT(SUBSTRING_INDEX(TRIM(employee_name), ' ', -1), 1)
    ) AS employee_initials
FROM employees;
```

> **Why?**
> This combines three functions, each solving one part of the problem:
> - `TRIM()` — guards against stray leading/trailing spaces breaking the logic.
> - `LEFT(..., 1)` — grabs the first letter of the *first* name.
> - `SUBSTRING_INDEX(..., ' ', -1)` — isolates the *last* word (last name) by splitting on space and taking everything after the final space, then `LEFT(..., 1)` grabs its first letter.
>
> Breaking it down step-by-step (as shown in the original notes) is a great way to build and debug this kind of nested logic before combining it into one query.

---

### Q6. Find employees who joined in the last 30 days.

```sql
-- Approach 1: Shorthand interval subtraction
SELECT * FROM employees
WHERE hire_date >= CURDATE() - INTERVAL 30 DAY;

-- Approach 2: Explicit DATE_SUB()
SELECT * FROM employees
WHERE hire_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```

> **Why?**
> Both compute the same cutoff date (today minus 30 days) and compare `hire_date` against it. Use whichever reads more clearly to you — many developers default to the shorthand once comfortable.

---

### Q7. Find employees who have completed more than 5 years in the company.

```sql
SELECT * FROM employees
WHERE TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) > 5;
```

---

### Q8. Display employee name, joining year, joining month name, and total years of experience.

```sql
SELECT
    employee_name,
    YEAR(hire_date) AS joining_year,
    MONTHNAME(hire_date) AS month_name,
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS total_years_experience
FROM employees;
```

---

### Q9. Display bonus details along with the day name on which the bonus was awarded.

```sql
SELECT
    *,
    DAYNAME(bonus_date) AS bonus_day
FROM bonuses;
```

---

### Q10. Display all projects along with their start date in the format: `01-Jan-2025`

```sql
SELECT
    *,
    DATE_FORMAT(start_date, '%d-%b-%Y') AS start_format_date
FROM projects;
```

---

### Q11. Categorize employees based on salary.

| Condition                        | Category      |
|-----------------------------------|---------------|
| Salary >= 100000                  | High Salary   |
| Salary BETWEEN 60000 AND 99999    | Medium Salary |
| Else                               | Low Salary    |

```sql
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High Salary'
        WHEN salary BETWEEN 60000 AND 99999 THEN 'Medium Salary'
        ELSE 'Low Salary'
    END AS employee_salary_ranking
FROM employees;
```

---

### Q12. Categorize employees based on experience.

| Condition        | Category  |
|-------------------|-----------|
| 0–2 Years          | Junior    |
| 3–5 Years          | Mid Level |
| More than 5 Years  | Senior    |

```sql
SELECT
    employee_name,
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_of_experience,
    CASE
        WHEN TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) <= 2 THEN 'Junior'
        WHEN TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) BETWEEN 3 AND 5 THEN 'Mid Level'
        ELSE 'Senior'
    END AS employee_experience_ranking
FROM employees;
```

> **Why?**
> Notice the experience calculation is repeated across the `SELECT` column and each `WHEN`. This works fine, but as queries grow, repeating logic like this becomes error-prone. A cleaner alternative (once you learn CTEs — see Part 6) is to calculate `years_of_experience` once in a CTE, then reference that column throughout the CASE statement.

---

### Q13. Find the total number of Active employees using CASE.

```sql
-- Approach 1: Simple COUNT with WHERE
SELECT COUNT(*) AS total_employees
FROM employees
WHERE employment_status = 'Active';

-- Approach 2: CASE inside SUM()
SELECT
    SUM(
        CASE
            WHEN employment_status = 'Active' THEN 1
            ELSE 0
        END
    ) AS total_employees
FROM employees;
```

> **Why learn the CASE + SUM() approach if WHERE already works?**
> This pattern becomes essential when you need **multiple conditional counts in a single row** — e.g., counting Active *and* Inactive *and* On Leave employees side-by-side without running three separate queries:
> ```sql
> SELECT
>     SUM(CASE WHEN employment_status = 'Active' THEN 1 ELSE 0 END) AS active_count,
>     SUM(CASE WHEN employment_status = 'Inactive' THEN 1 ELSE 0 END) AS inactive_count
> FROM employees;
> ```
> This is a foundational pattern for building **pivot-style summary reports** directly in SQL.

---

### Q14. Display project status with business-friendly labels.

| Raw Status  | Business Label         |
|-------------|--------------------------|
| Ongoing     | In Progress              |
| Completed   | Successfully Delivered   |
| On Hold     | Awaiting Approval        |

```sql
SELECT
    project_name,
    status,
    CASE
        WHEN status = 'Ongoing' THEN 'In Progress'
        WHEN status = 'Completed' THEN 'Successfully Delivered'
        WHEN status = 'On Hold' THEN 'Awaiting Approval'
    END AS project_status
FROM projects;
```

> **Why?**
> No `ELSE` here — deliberately. If a `status` value doesn't match any of the three listed, it returns `NULL`, which acts as a visible signal that an *unexpected* status value exists in the data (rather than silently mislabeling it). This is a useful debugging habit: only add `ELSE` when you genuinely have a sensible default for "everything else."

---

### Q15. Display employee names along with a custom message.

| Condition                                       | Message           |
|--------------------------------------------------|--------------------|
| Active AND salary > 100000                        | Star Employee      |
| Active                                             | Active Employee    |
| Else                                               | Needs Attention    |

```sql
SELECT
    employee_name,
    salary,
    employment_status,
    CASE
        WHEN employment_status = 'Active' AND salary > 100000 THEN 'Star Employee'
        WHEN employment_status = 'Active' THEN 'Active Employee'
        ELSE 'Needs Attention'
    END AS employee_label
FROM employees;
```

> **Why does order matter here?**
> This is the clearest example of "first match wins." If the more specific condition (`Active AND salary > 100000`) were placed *after* the general one (`Active`), it would **never be reached** — every Active + high-salary employee would already be caught by the broader `Active` condition first. **Always order CASE conditions from most specific to least specific.**

---

## 5. Cheatsheet

### String Functions

| Function              | Purpose                                   | Example                                      |
|------------------------|--------------------------------------------|-----------------------------------------------|
| `UPPER(str)`           | Convert to uppercase                       | `UPPER('raj')` → `RAJ`                        |
| `LOWER(str)`           | Convert to lowercase                       | `LOWER('RAJ')` → `raj`                        |
| `LENGTH(str)`          | Character count                            | `LENGTH('Raj')` → `3`                         |
| `CONCAT(a, b, ...)`    | Join strings                               | `CONCAT('Raj',' ','Sharma')` → `Raj Sharma`   |
| `SUBSTRING(str, s, l)` | Extract substring by position              | `SUBSTRING('MySQL',1,2)` → `My`               |
| `LEFT(str, n)`         | First n characters                         | `LEFT('Sharma',3)` → `Sha`                    |
| `RIGHT(str, n)`        | Last n characters                          | `RIGHT('Sharma',3)` → `rma`                   |
| `REPLACE(str, a, b)`   | Replace all occurrences                    | `REPLACE('a-b','-','/')` → `a/b`              |
| `TRIM(str)`            | Remove leading/trailing spaces             | `TRIM('  Raj  ')` → `Raj`                     |
| `SUBSTRING_INDEX(s,d,n)` | Split on delimiter, take before/after n  | `SUBSTRING_INDEX('a@b.com','@',1)` → `a`      |

### Date Functions

| Function                  | Purpose                                  |
|-----------------------------|--------------------------------------------|
| `CURDATE()`                 | Today's date                               |
| `DATEDIFF(d1, d2)`          | Difference in days                         |
| `DATE_FORMAT(date, fmt)`    | Custom formatted date                      |
| `YEAR(date)`                | Extract year                               |
| `MONTH(date)`               | Extract month number                       |
| `MONTHNAME(date)`           | Full month name                            |
| `DAYNAME(date)`             | Weekday name                               |
| `TIMESTAMPDIFF(unit, d1, d2)` | Difference in specified unit (YEAR/MONTH/DAY) |
| `DATE_SUB(date, INTERVAL n unit)` | Subtract interval from a date        |

### CASE Statement

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

**Rules to remember:**
1. Conditions evaluate top-to-bottom — first match wins.
2. Order conditions from **most specific → least specific**.
3. `ELSE` is optional; omitting it returns `NULL` for unmatched rows (useful for catching unexpected data).
4. `CASE` works anywhere an expression works — `SELECT`, `WHERE`, `ORDER BY`, and inside aggregate functions like `SUM()`/`COUNT()`.

---

## Key Takeaways

- **String functions** clean and reshape text — critical for presentation and for extracting structured pieces (like usernames or initials) from a single field.
- **Date functions** split into two families: ones that *extract parts* (`YEAR`, `MONTH`, `DAYNAME`) and ones that *calculate differences* (`DATEDIFF`, `TIMESTAMPDIFF`). Know which one you need — days-only vs. calendar-aware units.
- **CASE** turns raw data into business-readable categories directly in SQL, and combined with aggregate functions like `SUM()`, becomes a powerful tool for building conditional summaries and pivot-style reports.

---
