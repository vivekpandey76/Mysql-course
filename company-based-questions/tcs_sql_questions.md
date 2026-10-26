# 🏢 TCS SQL Interview Questions

> **Company-based set: TCS** — ten questions reported by candidates on GeeksforGeeks, AmbitionBox, LinkedIn and Reddit, including TCS Digital rounds. These are the screening-round staples TCS keeps coming back to: Nth highest salary, self-joins for managers, `LEFT JOIN` counts that must keep zero rows, top-N per group, generating missing IDs, and cloning a table.
> Each problem follows the same flow: **Question → Dataset → Expected Output → Solution → Explanation.**

Every problem is self-contained: run the *Dataset* block, then run the solution to reproduce the expected output. All queries target **MySQL 8.0+**. Every Expected Output block was checked by running the solution against its dataset.

---

## 📋 Contents

| # | Problem | Core Concept |
|---|---------|--------------|
| 1 | [Second highest / Nth highest salary](#q1--second-highest--nth-highest-salary) | `LIMIT … OFFSET` vs `DENSE_RANK()` |
| 2 | [Employees and their managers](#q2--employees-and-their-managers) | Self-join with `LEFT JOIN` |
| 3 | [Total students in each department](#q3--total-students-in-each-department) | `LEFT JOIN` + `COUNT(col)` for zero rows |
| 4 | [Latest visit date for each patient](#q4--latest-visit-date-for-each-patient) | `MAX()` + `GROUP BY` vs `ROW_NUMBER()` |
| 5 | [Clone a table](#q5--clone-a-table) | `CREATE TABLE … LIKE` vs `CREATE TABLE … AS SELECT` |
| 6 | [Top 3 salaries in each department](#q6--top-3-salaries-in-each-department) | `DENSE_RANK()` top-N per group |
| 7 | [Missing employee IDs between 1 and 100](#q7--missing-employee-ids-between-1-and-100) | Recursive CTE number series + anti-join |
| 8 | [City pairs with routes in both directions](#q8--city-pairs-with-routes-in-both-directions) | Self-join on swapped columns, `LEAST`/`GREATEST` |
| 9 | [Employees with maximum and minimum salary](#q9--employees-with-maximum-and-minimum-salary) | Scalar subqueries with `IN` |
| 10 | [Product-wise total amount, including unsold products](#q10--product-wise-total-amount-including-unsold-products) | `LEFT JOIN` + `COALESCE(SUM(…), 0)` |

---

## Q1 — Second Highest / Nth Highest Salary

**Source:** GeeksforGeeks, AmbitionBox

**Question:** Find the **second highest** salary. Then generalise the query to the **Nth highest** salary. Salaries contain duplicates, so ties must be handled.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id   INT PRIMARY KEY,
    emp_name VARCHAR(50),
    salary   INT
);

INSERT INTO employees (emp_id, emp_name, salary) VALUES
(1, 'Amit',  50000),
(2, 'Riya',  70000),
(3, 'Karan', 60000),
(4, 'Neha',  70000),
(5, 'Rahul', 45000),
(6, 'Sneha', 60000),
(7, 'Vikas', 80000);
```

### ✅ Expected Output

**Second highest:**

| second_salary |
|---|
| 70000 |

**Nth highest (N = 3):**

| nth_salary |
|---|
| 60000 |

### 💡 Solution

**Method 1: `DISTINCT` + `LIMIT … OFFSET`**

```sql
SELECT (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS second_salary;
```

**Method 2: `DENSE_RANK()`**

```sql
WITH ranked_salary AS (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS ranking
    FROM employees
)
SELECT DISTINCT salary AS second_salary
FROM ranked_salary
WHERE ranking = 2;
```

**Nth highest (N = 3)**

```sql
-- Method 1: OFFSET is always N - 1
SELECT (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET 2
) AS nth_salary;

-- Method 2: change only the rank you filter on
WITH ranked_salary AS (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS ranking
    FROM employees
)
SELECT DISTINCT salary AS nth_salary
FROM ranked_salary
WHERE ranking = 3;
```

**Nth highest with N as a variable**

```sql
SET @n = 3;
SET @skip = @n - 1;

PREPARE nth_stmt FROM
  'SELECT DISTINCT salary AS nth_salary
   FROM employees
   ORDER BY salary DESC
   LIMIT 1 OFFSET ?';
EXECUTE nth_stmt USING @skip;
DEALLOCATE PREPARE nth_stmt;
```

### 🧠 Explanation

- **`DISTINCT` is what handles ties.** Riya and Neha both earn 70000. Without `DISTINCT`, `OFFSET 1` lands on the second 70000 row and wrongly reports 70000 as "second" only by luck; ask for the third and you'd get 70000 again instead of 60000.
- **`DENSE_RANK` and not `RANK`:** `RANK` gives the two 70000s rank 2 and then jumps to 4, so `WHERE ranking = 3` would return nothing. `DENSE_RANK` has no gaps, so rank N is always the Nth distinct salary.
- **Why wrap Method 1 in an outer `SELECT`:** if N is larger than the number of distinct salaries, a bare `LIMIT … OFFSET` returns **zero rows**. Wrapping it as a scalar subquery returns a single `NULL` row instead, which is what the LeetCode version of this question expects.
- **N as a variable:** MySQL doesn't accept an expression like `@n - 1` directly in `LIMIT`. Use a prepared statement (above) or a stored function. In the `DENSE_RANK` version you can just write `WHERE ranking = @n`.

---

## Q2 — Employees and Their Managers

**Source:** GeeksforGeeks, Reddit

**Question:** HR wants a reporting-line sheet for the org chart. Managers are employees too, stored in the same table and linked through `manager_id`. For every employee, return `emp_id`, `emp_name` and `manager_name`. The CEO has no manager and must still appear with `manager_name` as `NULL`. Order by `emp_id`.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(50),
    salary     INT,
    manager_id INT              -- emp_id of the manager; NULL for the CEO
);

INSERT INTO employees (emp_id, emp_name, salary, manager_id) VALUES
(1, 'Amit',  150000, NULL),
(2, 'Riya',   90000, 1),
(3, 'Karan',  85000, 1),
(4, 'Neha',   95000, 2),
(5, 'Rahul',  60000, 2),
(6, 'Sneha',  70000, 3),
(7, 'Vikas',  88000, 3),
(8, 'Pooja',  50000, 4);
```

### ✅ Expected Output

| emp_id | emp_name | manager_name |
|---|---|---|
| 1 | Amit | NULL |
| 2 | Riya | Amit |
| 3 | Karan | Amit |
| 4 | Neha | Riya |
| 5 | Rahul | Riya |
| 6 | Sneha | Karan |
| 7 | Vikas | Karan |
| 8 | Pooja | Neha |

### 💡 Solution

```sql
SELECT e.emp_id,
       e.emp_name,
       m.emp_name AS manager_name
FROM employees e
LEFT JOIN employees m
       ON e.manager_id = m.emp_id
ORDER BY e.emp_id;
```

### 🧠 Explanation

- **Self-join:** the same table is used twice under two aliases. `e` is the employee row, `m` is the manager row, and `e.manager_id = m.emp_id` links them.
- **`LEFT JOIN`, not `INNER JOIN`:** Amit's `manager_id` is `NULL`, so he has no match. An `INNER JOIN` returns only 7 rows and silently drops the CEO.
- **Common follow-up:** "Find employees who earn more than their manager." Same join, plus `WHERE e.salary > m.salary`. Here that returns Neha (95000 vs Riya's 90000) and Vikas (88000 vs Karan's 85000).

---

## Q3 — Total Students in Each Department

**Source:** GeeksforGeeks

**Question:** Count the students in each department. Departments with **zero students** must still appear with a count of 0.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS students;
DROP TABLE IF EXISTS departments;
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(50)
);
CREATE TABLE students (
    student_id   INT PRIMARY KEY,
    student_name VARCHAR(50),
    dept_id      INT
);

INSERT INTO departments (dept_id, dept_name) VALUES
(1, 'CSE'),
(2, 'ECE'),
(3, 'MECH'),
(4, 'CIVIL');

INSERT INTO students (student_id, student_name, dept_id) VALUES
(101, 'Aarav',  1),
(102, 'Diya',   1),
(103, 'Ishaan', 2),
(104, 'Kavya',  1),
(105, 'Rohan',  3),
(106, 'Ananya', 2);
```

### ✅ Expected Output

| dept_name | total_students |
|---|---|
| CSE | 3 |
| ECE | 2 |
| MECH | 1 |
| CIVIL | 0 |

### 💡 Solution

```sql
SELECT d.dept_name,
       COUNT(s.student_id) AS total_students
FROM departments d
LEFT JOIN students s
       ON d.dept_id = s.dept_id
GROUP BY d.dept_id, d.dept_name
ORDER BY d.dept_id;
```

### 🧠 Explanation

- **Start from `departments`:** the question is "every department", so that's the left table. Starting from `students` can never produce CIVIL.
- ⚠️ **`COUNT(s.student_id)`, not `COUNT(*)`.** CIVIL has no students, but the `LEFT JOIN` still produces one row for it with `NULL` in every `students` column. `COUNT(*)` counts that row and reports **CIVIL = 1** (verified). `COUNT(s.student_id)` skips `NULL`s and correctly reports 0.
- `d.dept_id` is in the `GROUP BY` so two departments that happen to share a name are never merged.

---

## Q4 — Latest Visit Date for Each Patient

**Source:** GeeksforGeeks, TCS Digital

**Question:** Return each patient with their **most recent** visit date.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS patient_visits;
CREATE TABLE patient_visits (
    visit_id     INT PRIMARY KEY,
    patient_id   INT,
    patient_name VARCHAR(50),
    visit_date   DATE
);

INSERT INTO patient_visits (visit_id, patient_id, patient_name, visit_date) VALUES
(1, 101, 'Arjun', '2024-01-10'),
(2, 102, 'Meera', '2024-02-15'),
(3, 101, 'Arjun', '2024-03-05'),
(4, 103, 'Sahil', '2024-01-20'),
(5, 102, 'Meera', '2024-04-01'),
(6, 101, 'Arjun', '2024-02-18'),
(7, 103, 'Sahil', '2024-05-12');
```

### ✅ Expected Output

| patient_id | patient_name | latest_visit_date |
|---|---|---|
| 101 | Arjun | 2024-03-05 |
| 102 | Meera | 2024-04-01 |
| 103 | Sahil | 2024-05-12 |

### 💡 Solution

**Method 1: `MAX()` + `GROUP BY`**

```sql
SELECT patient_id,
       patient_name,
       MAX(visit_date) AS latest_visit_date
FROM patient_visits
GROUP BY patient_id, patient_name
ORDER BY patient_id;
```

**Method 2: `ROW_NUMBER()`**

```sql
WITH ranked_visits AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY patient_id
                              ORDER BY visit_date DESC, visit_id DESC) AS rn
    FROM patient_visits
)
SELECT patient_id,
       patient_name,
       visit_date AS latest_visit_date
FROM ranked_visits
WHERE rn = 1
ORDER BY patient_id;
```

### 🧠 Explanation

- **Method 1 is the answer when you only need the date.** It's the shortest query and the cheapest to run.
- **Method 2 is the answer when you need the whole row** (visit_id, doctor, diagnosis…). `MAX()` gives you the date, but not the other columns from that same visit.
- **`ROW_NUMBER`, not `DENSE_RANK`, for "one row per group":** if a patient had two visits on the same latest date, `DENSE_RANK` gives both rank 1 and returns the patient twice. `ROW_NUMBER` with `visit_id` as a tie-breaker always returns exactly one row.

---

## Q5 — Clone a Table

**Source:** GeeksforGeeks

**Question:**
- (a) Create `employees_backup` with the same structure **and data** as `employees`.
- (b) Create `employees_empty` with the same structure but **no data**.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(50),
    department VARCHAR(50),
    salary     INT
);

INSERT INTO employees (emp_id, emp_name, department, salary) VALUES
(1, 'Amit',  'IT',      50000),
(2, 'Riya',  'HR',      40000),
(3, 'Karan', 'IT',      60000),
(4, 'Neha',  'Finance', 55000);
```

### ✅ Expected Result

| table | rows | columns | primary key on emp_id |
|---|---|---|---|
| employees_backup | 4 | same as employees | Yes (Method 1) / No (Method 2) |
| employees_empty | 0 | same as employees | Yes (Method 1) / No (Method 2) |

### 💡 Solution

**Method 1: `CREATE TABLE … LIKE` (keeps keys and indexes)**

```sql
-- (a) structure + data
DROP TABLE IF EXISTS employees_backup;
CREATE TABLE employees_backup LIKE employees;
INSERT INTO employees_backup
SELECT * FROM employees;

-- (b) structure only
DROP TABLE IF EXISTS employees_empty;
CREATE TABLE employees_empty LIKE employees;
```

**Method 2: `CREATE TABLE … AS SELECT` (columns and data only)**

```sql
-- (a) structure + data
DROP TABLE IF EXISTS employees_backup;
CREATE TABLE employees_backup AS
SELECT * FROM employees;

-- (b) structure only: a condition that is never true copies zero rows
DROP TABLE IF EXISTS employees_empty;
CREATE TABLE employees_empty AS
SELECT * FROM employees
WHERE 1 = 0;
```

**Check the result**

```sql
SELECT COUNT(*) FROM employees_backup;   -- 4
SELECT COUNT(*) FROM employees_empty;    -- 0
SHOW CREATE TABLE employees_backup;      -- compare keys between the two methods
```

### 🧠 Explanation

| | `CREATE TABLE … LIKE` | `CREATE TABLE … AS SELECT` |
|---|---|---|
| Column names and types | ✅ | ✅ |
| Data | ❌ (add `INSERT … SELECT`) | ✅ |
| Primary key, indexes, `AUTO_INCREMENT` | ✅ | ❌ |
| Foreign keys | ❌ | ❌ |

- **`LIKE` is the safer backup.** It copies the full table definition, including the primary key, so the backup behaves like the original. `AS SELECT` creates a plain table with no primary key, which is fine for a quick snapshot but not as a drop-in replacement.
- ⚠️ **Don't follow `CREATE TABLE … LIKE` with an `INSERT` when the question asks for an empty table.** `LIKE` already gives you the empty structure. Adding `INSERT INTO employees_empty SELECT * FROM employees` turns it into a second full backup.
- **`WHERE 1 = 0`** is the standard trick for "structure only" with `AS SELECT`: the condition is never true, so MySQL builds the columns but copies no rows.

---

## Q6 — Top 3 Salaries in Each Department

**Source:** LinkedIn

**Question:** For each department, return the employees earning one of the **top 3 distinct salaries**.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(50),
    department VARCHAR(50),
    salary     INT
);

INSERT INTO employees (emp_id, emp_name, department, salary) VALUES
( 1, 'Amit',   'IT',    90000),
( 2, 'Riya',   'IT',    85000),
( 3, 'Karan',  'IT',    85000),
( 4, 'Neha',   'IT',    70000),
( 5, 'Rahul',  'IT',    60000),
( 6, 'Sneha',  'HR',    50000),
( 7, 'Vikas',  'HR',    45000),
( 8, 'Pooja',  'HR',    40000),
( 9, 'Manoj',  'HR',    35000),
(10, 'Anjali', 'Sales', 65000),
(11, 'Deepak', 'Sales', 55000);
```

### ✅ Expected Output

| department | emp_name | salary |
|---|---|---|
| HR | Sneha | 50000 |
| HR | Vikas | 45000 |
| HR | Pooja | 40000 |
| IT | Amit | 90000 |
| IT | Riya | 85000 |
| IT | Karan | 85000 |
| IT | Neha | 70000 |
| Sales | Anjali | 65000 |
| Sales | Deepak | 55000 |

### 💡 Solution

```sql
WITH ranked_salary AS (
    SELECT emp_id,
           department,
           emp_name,
           salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM employees
)
SELECT department, emp_name, salary
FROM ranked_salary
WHERE ranking <= 3
ORDER BY department, salary DESC, emp_id;
```

### 🧠 Explanation

- `PARTITION BY department` restarts the ranking in each department; `ORDER BY salary DESC` puts the highest salary at rank 1.
- **Why IT returns 4 people:** Riya and Karan tie at 85000, so both get rank 2 and Neha's 70000 is rank 3. "Top 3 *distinct* salaries" means 90000, 85000 and 70000, which covers four employees.
- **Which ranking function to use depends on the wording:**
  - "Top 3 distinct salaries" → `DENSE_RANK` (IT returns 4 rows).
  - "Top 3 employees" → `ROW_NUMBER` with a tie-breaker (IT returns exactly 3 rows, Neha is dropped).
- Sales has only two employees, so it returns two rows. `<= 3` never forces three rows into a small group.

---

## Q7 — Missing Employee IDs Between 1 and 100

**Source:** LinkedIn

**Question:** List every ID from **1 to 100** that does **not** exist in the `employees` table.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id   INT PRIMARY KEY,
    emp_name VARCHAR(50)
);

INSERT INTO employees (emp_id, emp_name) VALUES
( 1,'Amit'),   ( 2,'Riya'),    ( 3,'Karan'),    ( 5,'Neha'),     ( 6,'Rahul'),
( 9,'Sneha'),  (10,'Vikas'),   (13,'Priya'),    (14,'Arjun'),    (15,'Pooja'),
(17,'Rohit'),  (18,'Anjali'),  (20,'Suresh'),   (21,'Meera'),    (22,'Akash'),
(25,'Kavita'), (26,'Manish'),  (28,'Nisha'),    (29,'Varun'),    (30,'Divya'),
(32,'Aman'),   (33,'Simran'),  (35,'Kunal'),    (36,'Isha'),     (37,'Deepak'),
(40,'Snehal'), (41,'Mohit'),   (42,'Tanya'),    (45,'Rakesh'),   (46,'Shreya'),
(48,'Nikhil'), (50,'Komal'),   (51,'Aditya'),   (53,'Swati'),    (54,'Harsh'),
(55,'Payal'),  (58,'Gaurav'),  (60,'Pallavi'),  (61,'Yash'),     (63,'Tanvi'),
(64,'Sahil'),  (65,'Aarti'),   (68,'Varsha'),   (70,'Pranav'),   (71,'Mansi'),
(73,'Abhishek'),(74,'Sakshi'), (75,'Rohan'),    (78,'Kiran'),    (80,'Shweta'),
(81,'Jay'),    (83,'Neelam'),  (85,'Ankit'),    (86,'Bhavna'),   (88,'Raj'),
(90,'Sonali'), (91,'Nitin'),   (93,'Preeti'),   (95,'Omkar'),    (96,'Ritu'),
(98,'Sanjay'), (99,'Kirti');
```

### ✅ Expected Output

38 rows (100 IDs minus the 62 that exist). First and last rows:

| missing_id |
|---|
| 4 |
| 7 |
| 8 |
| 11 |
| 12 |
| … |
| 94 |
| 97 |
| 100 |

Full list: 4, 7, 8, 11, 12, 16, 19, 23, 24, 27, 31, 34, 38, 39, 43, 44, 47, 49, 52, 56, 57, 59, 62, 66, 67, 69, 72, 76, 77, 79, 82, 84, 87, 89, 92, 94, 97, 100.

### 💡 Solution

```sql
WITH RECURSIVE all_ids AS (
    SELECT 1 AS id              -- anchor: start the series at 1

    UNION ALL

    SELECT id + 1               -- recursive step: add 1 each time
    FROM all_ids
    WHERE id < 100              -- stop at 100
)
SELECT a.id AS missing_id
FROM all_ids a
LEFT JOIN employees e
       ON a.id = e.emp_id
WHERE e.emp_id IS NULL
ORDER BY a.id;
```

### 🧠 Explanation

- **There is no table of 1–100 to join against, so build one.** The recursive CTE starts at 1 and keeps adding 1 until it reaches 100.
- **Anti-join:** `LEFT JOIN` keeps all 100 numbers, and `WHERE e.emp_id IS NULL` keeps only those with no matching employee.
- **Keep the stop condition a plain literal:** `WHERE id < 100` does the job. A subquery like `(SELECT 100)` works but adds nothing.
- **MySQL's recursion limit** is `cte_max_recursion_depth`, 1000 by default. Generating 1 to 5000 fails with an error until you raise it: `SET SESSION cte_max_recursion_depth = 10000;`.
- **Follow-up:** to check up to the highest existing ID instead of a fixed 100, replace `100` with `(SELECT MAX(emp_id) FROM employees)`.

---

## Q8 — City Pairs with Routes in Both Directions

**Question:** Return each city pair **once** where a route exists in **both directions** (A → B and B → A).

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS routes;
CREATE TABLE routes (
    source_city      VARCHAR(50),
    destination_city VARCHAR(50)
);

INSERT INTO routes (source_city, destination_city) VALUES
('Delhi',     'Hyderabad'),
('Hyderabad', 'Delhi'),
('Bangalore', 'Mumbai'),
('Mumbai',    'Bangalore'),
('Kolkata',   'Pune'),
('Pune',      'Kolkata');
```

### ✅ Expected Output

| source_city | destination_city |
|---|---|
| Bangalore | Mumbai |
| Delhi | Hyderabad |
| Kolkata | Pune |

### 💡 Solution

**Method 1: Self-join on swapped columns**

```sql
SELECT DISTINCT r1.source_city, r1.destination_city
FROM routes r1
JOIN routes r2
  ON r1.source_city      = r2.destination_city
 AND r1.destination_city = r2.source_city
WHERE r1.source_city < r1.destination_city
ORDER BY r1.source_city;
```

**Method 2: `LEAST` / `GREATEST` + `GROUP BY`**

```sql
SELECT LEAST(source_city, destination_city)    AS source_city,
       GREATEST(source_city, destination_city) AS destination_city
FROM routes
GROUP BY LEAST(source_city, destination_city),
         GREATEST(source_city, destination_city)
HAVING COUNT(DISTINCT source_city) = 2
ORDER BY source_city;
```

### 🧠 Explanation

- **Method 1:** the join matches a route with its reverse: r1's source is r2's destination **and** r1's destination is r2's source. Both conditions are needed.
- **`r1.source_city < r1.destination_city`** keeps one of the two directions. Without it, every pair appears twice (Delhi → Hyderabad and Hyderabad → Delhi).
- ⚠️ **A common bug is writing the same condition twice:** `ON r1.source_city = r2.destination_city AND r2.destination_city = r1.source_city`. That only checks one side. It happens to pass on this dataset, but add a single route `('Mumbai', 'Delhi')` and it returns **Delhi → Hyderabad twice** (verified), because any route ending in Delhi now counts as a match.
- **`DISTINCT`** protects against the table holding the same route more than once.
- **Method 2** needs no join. `LEAST`/`GREATEST` put every pair into the same alphabetical order, so Delhi → Hyderabad and Hyderabad → Delhi land in the same group. `COUNT(DISTINCT source_city) = 2` means both cities appeared as a source, which is the "both directions" check. A one-way route like `('Chennai', 'Goa')` has a count of 1 and is filtered out.

---

## Q9 — Employees with Maximum and Minimum Salary

**Question:** Return **all** employees who earn the highest salary and **all** employees who earn the lowest salary.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(50),
    department VARCHAR(50),
    salary     DECIMAL(10,2)
);

INSERT INTO employees (emp_id, emp_name, department, salary) VALUES
( 1, 'Amit',   'HR',        45000.00),
( 2, 'Riya',   'IT',        82000.00),
( 3, 'Karan',  'Sales',     38000.00),
( 4, 'Neha',   'IT',        95000.00),
( 5, 'Rahul',  'Finance',   61000.00),
( 6, 'Pooja',  'Marketing', 38000.00),
( 7, 'Vikram', 'Finance',   95000.00),
( 8, 'Sneha',  'HR',        52000.00),
( 9, 'Arjun',  'Sales',     38000.00),
(10, 'Divya',  'IT',        95000.00),
(11, 'Manish', 'Marketing', 70000.00),
(12, 'Kavya',  'Finance',   48000.00);
```

### ✅ Expected Output

| emp_id | emp_name | department | salary |
|---|---|---|---|
| 4 | Neha | IT | 95000.00 |
| 7 | Vikram | Finance | 95000.00 |
| 10 | Divya | IT | 95000.00 |
| 3 | Karan | Sales | 38000.00 |
| 6 | Pooja | Marketing | 38000.00 |
| 9 | Arjun | Sales | 38000.00 |

### 💡 Solution

**Method 1: `IN` with two scalar subqueries**

```sql
SELECT *
FROM employees
WHERE salary IN (
    (SELECT MAX(salary) FROM employees),
    (SELECT MIN(salary) FROM employees)
)
ORDER BY salary DESC, emp_id;
```

**Method 2: `UNION`**

```sql
SELECT * FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees)

UNION

SELECT * FROM employees
WHERE salary = (SELECT MIN(salary) FROM employees)

ORDER BY salary DESC, emp_id;
```

### 🧠 Explanation

- **Compare against the value, not the row.** `MAX(salary)` returns 95000, and `WHERE salary = 95000` returns every employee on that salary. `ORDER BY salary DESC LIMIT 1` would return only one of the three.
- **Method 1 reads the table once for the result** and is the easiest to extend (add a third value to the `IN` list).
- **`UNION`, not `UNION ALL`, in Method 2:** if every employee earned the same salary, MAX and MIN would be equal and `UNION ALL` would return everyone twice. `UNION` removes those duplicates.
- The final `ORDER BY` applies to the combined result, so it goes after the last `SELECT`.

---

## Q10 — Product-wise Total Amount, Including Unsold Products

**Question:** Return the total sales amount per product. Products with **no sales** must show `0`.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS products;
CREATE TABLE products (
    product_id   INT PRIMARY KEY,
    product_name VARCHAR(50)
);
CREATE TABLE sales (
    sale_id    INT PRIMARY KEY,
    product_id INT,
    amount     DECIMAL(10,2)
);

INSERT INTO products (product_id, product_name) VALUES
(1, 'Laptop'),
(2, 'Mobile'),
(3, 'Tablet'),
(4, 'Headphones'),
(5, 'Monitor');

INSERT INTO sales (sale_id, product_id, amount) VALUES
(1, 1, 55000.00),
(2, 2, 20000.00),
(3, 1, 60000.00),
(4, 4,  2500.00),
(5, 2, 18000.00);
```

### ✅ Expected Output

| product_name | total_amount |
|---|---|
| Laptop | 115000.00 |
| Mobile | 38000.00 |
| Tablet | 0.00 |
| Headphones | 2500.00 |
| Monitor | 0.00 |

### 💡 Solution

```sql
SELECT p.product_name,
       COALESCE(SUM(s.amount), 0) AS total_amount
FROM products p
LEFT JOIN sales s
       ON p.product_id = s.product_id
GROUP BY p.product_id, p.product_name
ORDER BY p.product_id;
```

### 🧠 Explanation

- **`LEFT JOIN` from `products`** keeps Tablet and Monitor even though they have no sales.
- **Why `COALESCE` is needed:** for an unsold product, every `s.amount` is `NULL`, and `SUM` of only `NULL`s is `NULL`, not 0. Without `COALESCE`, Tablet and Monitor show `NULL` (verified). `COALESCE(…, 0)` turns that into 0.
- `SUM(COALESCE(s.amount, 0))` gives the same result here. Wrapping the outside is the usual form because it states the intent directly: "if the total is missing, show 0".
- Same pattern as Q3: start from the table that defines "every", `LEFT JOIN` the facts, and make the empty case show 0 (`COUNT(col)` there, `COALESCE(SUM(…), 0)` here).

---

<div align="center">

**If this helped you, please ⭐ the repo and [subscribe on YouTube](https://www.youtube.com/watch?v=DwFu-cqVm7E). It genuinely helps! 🙌**

</div>
