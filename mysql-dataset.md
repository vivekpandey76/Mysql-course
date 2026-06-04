# 🗄️ employer_database — MySQL Complete Course Dataset

> A **real-life, interview-style MySQL dataset** purpose-built for a full SQL course.
> Every table is crafted so you can practise **every topic** — from basic SELECT to Window Functions,
> CTEs, Duplicate detection & deletion, and complex multi-table JOINs.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Database Setup](#-database-setup)
- [Tables at a Glance](#-tables-at-a-glance)
- [Entity Relationship Overview](#-entity-relationship-overview)
- [Step-by-Step Table Creation](#-step-by-step-table-creation)
  - [Step 1 — managers](#step-1--managers-table)
  - [Step 2 — employees](#step-2--employees-table)
  - [Step 3 — bonuses](#step-3--bonuses-table)
  - [Step 4 — attendance](#step-4--attendance-table)
  - [Step 5 — projects](#step-5--projects-table)
  - [Step 6 — employee_projects](#step-6--employee_projects-table)
  - [Step 7 — salary_history](#step-7--salary_history-table)
  - [Step 8 — leaves_data](#step-8--leaves_data-table)
  - [Step 9 — clients](#step-9--clients-table)
  - [Step 10 — orders_data](#step-10--orders_data-table)
- [Duplicate Records Map](#-duplicate-records-map)
- [Dataset Design Notes](#-dataset-design-notes)

---

## 🌟 Overview

| Feature | Detail |
|---|---|
| **Database Name** | `employer_database` |
| **Total Tables** | 10 |
| **Total Rows** | 200+ |
| **Difficulty Levels** | Beginner → Advanced |
| **Special Data** | Duplicates in 4 tables · NULLs · `department` & `employment_status` in employees · Many-to-Many |
| **Topics Covered** | CRUD · Filtering · Aggregates · GROUP BY · HAVING · All JOINs · Subqueries · CTEs · Window Functions · Duplicates · NULL Handling |

---

## 🚀 Database Setup

```sql
-- Step 0: Create and select the database
CREATE DATABASE employer_database;
USE employer_database;
```

---

## 📊 Tables at a Glance

| # | Table | Purpose | Duplicates? |
|---|---|---|---|
| 1 | `managers` | Manager directory | — |
| 2 | `employees` | Core table — `department` column, `employment_status`, NULLs | ✅ Yes |
| 3 | `bonuses` | Bonus payouts — duplicate entries exist | ✅ Yes |
| 4 | `attendance` | Daily attendance — duplicate entries exist | ✅ Yes |
| 5 | `projects` | Project catalogue | — |
| 6 | `employee_projects` | Many-to-many: employee ↔ project — duplicate assignments exist | ✅ Yes |
| 7 | `salary_history` | Salary increment history | — |
| 8 | `leaves_data` | Leave type & days per employee | — |
| 9 | `clients` | Client master data | — |
| 10 | `orders_data` | Sales & revenue data | — |

---

## 🔗 Entity Relationship Overview

```
managers        (1) ──────────── (N) employees
employees       (1) ──────────── (N) bonuses
employees       (1) ──────────── (N) attendance
employees       (N) ──────────── (N) projects       [via employee_projects]
employees       (1) ──────────── (N) salary_history
employees       (1) ──────────── (N) leaves_data
employees       (1) ──────────── (N) orders_data
clients         (1) ──────────── (N) orders_data
```

---

## 🛠️ Step-by-Step Table Creation

---

### Step 1 — `managers` Table

> Manager directory. Some managers manage many employees; one manages none — useful for all JOIN types.

```sql
CREATE TABLE managers (
    manager_id       INT PRIMARY KEY,
    manager_name     VARCHAR(100),
    experience_years INT,
    city             VARCHAR(50),
    department       VARCHAR(50)
);

INSERT INTO managers (manager_id, manager_name, experience_years, city, department) VALUES
(1, 'Raj Sharma',    12, 'Mumbai',     'IT'),
(2, 'Ankit Verma',    9, 'Delhi',      'HR'),
(3, 'Priya Singh',   15, 'Pune',       'Finance'),
(4, 'Amit Patel',     7, 'Bangalore',  'Sales'),
(5, 'Neha Joshi',    11, 'Hyderabad',  'Marketing'),
(6, 'Rohit Mehta',    6, 'Chennai',    'Support'),
(7, 'Sneha Kapoor',  13, 'Mumbai',     'IT'),       -- second IT manager (same dept as manager 1)
(8, 'Kunal Shah',    10, 'Ahmedabad',  'Finance');  -- ⚠️ no employees assigned → LEFT JOIN practice
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `manager_id` | INT | Primary Key |
| `manager_name` | VARCHAR(100) | Full name |
| `experience_years` | INT | Years of experience |
| `city` | VARCHAR(50) | Work city |
| `department` | VARCHAR(50) | Department they manage |

---

### Step 2 — `employees` Table

> **The heart of the dataset.**
>
> Key design features:
> - ✅ `AUTO_INCREMENT` primary key
> - ✅ `department` stored directly as a text column — no separate table, no FK needed
> - ✅ `employment_status` column (`Active` / `Inactive` / `On Leave` / `Terminated`)
> - ⚠️ **Duplicate rows** — 5 employees inserted twice (rows 140–144) for dedup practice
> - ⚠️ NULL `salary` — for NULL handling queries
> - ⚠️ NULL `manager_id` — employees with no manager assigned
> - ⚠️ NULL `department` — for IS NULL / COALESCE / IFNULL practice
> - ⚠️ Same salary tie (₹70,000) — for RANK vs DENSE_RANK difference
> - ⚠️ Salary outliers — lowest ₹25,000 · highest ₹1,50,000

```sql
CREATE TABLE employees (
    employee_id       INT AUTO_INCREMENT PRIMARY KEY,
    employee_name     VARCHAR(100),
    gender            VARCHAR(10),
    department        VARCHAR(50),        -- stored directly, no FK needed
    salary            INT,                -- can be NULL
    manager_id        INT,                -- FK → managers; can be NULL
    hire_date         DATE,
    city              VARCHAR(50),
    state             VARCHAR(50),        -- can be NULL
    email             VARCHAR(100),       -- can be NULL
    employment_status VARCHAR(20)         -- Active | Inactive | On Leave | Terminated
);

INSERT INTO employees
    (employee_id, employee_name, gender, department, salary, manager_id,
     hire_date, city, state, email, employment_status)
VALUES
(101, 'Aarav Sharma',  'Male',   'IT',        75000,  1, '2021-01-15', 'Mumbai',    'Maharashtra', 'aarav@gmail.com',    'Active'),
(102, 'Priya Mehta',   'Female', 'HR',        55000,  2, '2020-03-10', 'Delhi',     'Delhi',       'priya@gmail.com',    'Active'),
(103, 'Rohit Jain',    'Male',   'Finance',   68000,  3, '2019-07-23', 'Pune',      'Maharashtra', 'rohit@gmail.com',    'Active'),
(104, 'Sneha Patil',   'Female', 'IT',        80000,  1, '2022-02-12', 'Mumbai',    'Maharashtra', 'sneha@gmail.com',    'Active'),
(105, 'Vikas Singh',   'Male',   'Sales',     45000,  4, '2021-11-05', 'Bangalore', 'Karnataka',   'vikas@gmail.com',    'Active'),
(106, 'Anjali Verma',  'Female', 'Marketing', 60000,  5, '2020-09-17', 'Hyderabad', 'Telangana',   'anjali@gmail.com',   'Active'),
(107, 'Karan Shah',    'Male',   'IT',        95000,  1, '2018-04-19', 'Mumbai',    'Maharashtra', 'karan@gmail.com',    'Active'),
(108, 'Meera Joshi',   'Female', 'HR',        53000,  2, '2021-06-14', 'Pune',      'Maharashtra', 'meera@gmail.com',    'Active'),
(109, 'Pooja Nair',    'Female', 'Finance',   72000,  3, '2019-12-01', 'Chennai',   'Tamil Nadu',  'pooja@gmail.com',    'Active'),
(110, 'Low Salary',    'Male',   'Support',   25000,  6, '2023-01-01', 'Pune',      'Maharashtra', 'low@gmail.com',      'Active'),  -- ⚠️ lowest salary
(111, 'Arjun Gupta',   'Male',   'IT',        78000,  1, '2020-08-11', 'Mumbai',    'Maharashtra', 'arjun@gmail.com',    'Active'),
(112, 'Rahul Kapoor',  'Male',   'Sales',     47000,  4, '2023-01-08', 'Delhi',     'Delhi',       'rahul@gmail.com',    'Active'),
(113, 'Simran Kaur',   'Female', 'Marketing', 62000,  5, '2022-10-09', 'Chandigarh','Punjab',      'simran@gmail.com',   'Active'),
(114, 'Aman Mishra',   'Male',   'HR',        54000,  2, '2021-04-30', 'Lucknow',   'UP',          'aman@gmail.com',     'Active'),
(115, 'Dev Malhotra',  'Male',   'Finance',   67000,  3, '2017-03-15', 'Delhi',     'Delhi',       'dev@gmail.com',      'Active'),
(116, 'Nisha Rao',     'Female', 'IT',        83000,  1, '2021-05-25', 'Bangalore', 'Karnataka',   'nisha@gmail.com',    'Active'),
(117, 'Ravi Kumar',    'Male',   'Support',   30000,  6, '2022-05-15', 'Chennai',   'Tamil Nadu',  'ravi@gmail.com',     'Active'),
(118, 'Yash Thakur',   'Male',   'Sales',     50000,  4, '2022-01-20', 'Mumbai',    'Maharashtra', 'yash@gmail.com',     'Active'),
(119, 'Ritika Sen',    'Female', 'Marketing', 59000,  5, '2019-11-13', 'Kolkata',   'West Bengal', 'ritika@gmail.com',   'Active'),
(120, 'Tanvi Desai',   'Female', 'HR',        52000,  2, '2019-09-27', 'Ahmedabad', 'Gujarat',     'tanvi@gmail.com',    'On Leave'),
(121, 'Mohit Arora',   'Male',   'IT',        91000,  7, '2018-07-07', 'Delhi',     'Delhi',       'mohit@gmail.com',    'Active'),
(122, 'Kriti Sharma',  'Female', 'Finance',   71000,  3, '2020-12-21', 'Pune',      'Maharashtra', 'kriti@gmail.com',    'Active'),
(123, 'Neha Iyer',     'Female', 'Sales',     49000,  4, '2023-03-18', 'Chennai',   'Tamil Nadu',  'nehaiyer@gmail.com', 'Active'),
(124, 'Varun Sethi',   'Male',   'Marketing', 61000,  5, '2020-01-05', 'Delhi',     'Delhi',       'varun@gmail.com',    'Active'),
(125, 'Harsh Pandey',  'Male',   'IT',        76000,  7, '2022-06-11', 'Mumbai',    'Maharashtra', 'harsh@gmail.com',    'On Leave'),
(126, 'Divya Pillai',  'Female', 'Support',   32000,  6, '2021-09-10', 'Hyderabad', 'Telangana',   'divya@gmail.com',    'On Leave'),
(127, 'Isha Kulkarni', 'Female', 'Finance',   69000,  3, '2021-08-14', 'Pune',      'Maharashtra', 'isha@gmail.com',     'Active'),
(128, 'Sonal Jain',    'Female', 'HR',        51000,  2, '2023-04-04', 'Jaipur',    'Rajasthan',   'sonal@gmail.com',    'Active'),
(129, 'Aisha Khan',    'Female', 'Sales',     46000,  4, '2022-02-28', 'Hyderabad', 'Telangana',   'aisha@gmail.com',    'Inactive'),
(130, 'Siddharth Roy', 'Male',   'IT',        87000,  1, '2018-05-17', 'Kolkata',   'West Bengal', 'sid@gmail.com',      'Active'),
(131, 'Preeti Das',    'Female', 'Marketing', 64000,  5, '2020-10-10', 'Kolkata',   'West Bengal', 'preeti@gmail.com',   'Terminated'),
(132, 'Rajat Saxena',  'Male',   'Finance',   73000,  3, '2019-06-06', 'Delhi',     'Delhi',       'rajat@gmail.com',    'Terminated'),
(133, 'Akash Yadav',   'Male',   'IT',        88000,  7, '2017-08-22', 'Lucknow',   'UP',          'akash@gmail.com',    'Active'),
(134, 'No Manager',    'Male',   'Finance',   65000,  NULL,'2020-02-20','Nagpur',   'Maharashtra', 'nomgr@gmail.com',    'Active'),   -- ⚠️ NULL manager_id
(135, 'Same Salary A', 'Female', 'IT',        70000,  1, '2021-03-03', 'Mumbai',    'Maharashtra', 'same1@gmail.com',    'Active'),  -- ⚠️ salary tie
(136, 'Same Salary B', 'Male',   'IT',        70000,  1, '2021-03-04', 'Mumbai',    'Maharashtra', 'same2@gmail.com',    'Active'),  -- ⚠️ salary tie
(137, 'High Salary',   'Female', 'IT',       150000,  1, '2016-05-15', 'Mumbai',    'Maharashtra', 'high@gmail.com',     'Active'),  -- ⚠️ highest salary
(138, 'Test User',     'Female', 'IT',          NULL,  1, '2022-09-01', 'Mumbai',   'Maharashtra', 'test@gmail.com',     'Inactive'), -- ⚠️ NULL salary
(139, 'NULL Employee', 'Male',   NULL,        40000,  NULL,'2023-02-10','Indore',   NULL,          NULL,                 'Inactive'), -- ⚠️ NULL dept, state, email, manager

-- ⚠️ DUPLICATE ROWS — exact copies of existing employees, for dedup practice
(140, 'Aarav Sharma',  'Male',   'IT',        75000,  1, '2021-01-15', 'Mumbai',    'Maharashtra', 'aarav@gmail.com',    'Active'),   -- duplicate of 101
(141, 'Rohit Jain',    'Male',   'Finance',   68000,  3, '2019-07-23', 'Pune',      'Maharashtra', 'rohit@gmail.com',    'Active'),   -- duplicate of 103
(142, 'Priya Mehta',   'Female', 'HR',        55000,  2, '2020-03-10', 'Delhi',     'Delhi',       'priya@gmail.com',    'Active'),   -- duplicate of 102
(143, 'Vikas Singh',   'Male',   'Sales',     45000,  4, '2021-11-05', 'Bangalore', 'Karnataka',   'vikas@gmail.com',    'Active'),   -- duplicate of 105
(144, 'Anjali Verma',  'Female', 'Marketing', 60000,  5, '2020-09-17', 'Hyderabad', 'Telangana',   'anjali@gmail.com',   'Active');   -- duplicate of 106
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `employee_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_name` | VARCHAR(100) | May be duplicated intentionally |
| `gender` | VARCHAR(10) | Male / Female |
| `department` | VARCHAR(50) | Stored as text directly — can be NULL |
| `salary` | INT | Can be NULL |
| `manager_id` | INT | FK → managers; can be NULL |
| `hire_date` | DATE | Date of joining |
| `city` | VARCHAR(50) | Work city |
| `state` | VARCHAR(50) | Can be NULL |
| `email` | VARCHAR(100) | Can be NULL |
| `employment_status` | VARCHAR(20) | Active / Inactive / On Leave / Terminated |

> 💡 `department` is a plain text column — no separate departments table, no FK. You can directly use `GROUP BY department` without any JOIN, making it perfect for beginner-level aggregate queries.

---

### Step 3 — `bonuses` Table

> Bonus payouts per employee.
> **Intentional duplicates**: 4 bonus entries repeated (same employee + amount + date).
> Some employees have **no bonus at all** — for LEFT JOIN / NOT IN / NOT EXISTS practice.

```sql
CREATE TABLE bonuses (
    bonus_id     INT PRIMARY KEY AUTO_INCREMENT,
    employee_id  INT,
    bonus_amount INT,
    bonus_date   DATE,
    bonus_type   VARCHAR(30)    -- Performance | Diwali | Joining | Referral
);

INSERT INTO bonuses (bonus_id, employee_id, bonus_amount, bonus_date, bonus_type) VALUES
-- Regular bonus records
(1,  101, 10000, '2024-01-10', 'Performance'),
(2,  114,  5000, '2024-01-15', 'Performance'),
(3,  119,  7000, '2024-02-01', 'Performance'),
(4,  102, 12000, '2024-02-05', 'Performance'),
(5,  126,  3000, '2024-02-10', 'Diwali'),
(6,  103, 15000, '2024-03-01', 'Performance'),
(7,  120,  8000, '2024-03-05', 'Performance'),
(8,  115,  4000, '2024-03-15', 'Diwali'),
(9,  104,  9000, '2024-04-01', 'Performance'),
(10, 105, 11000, '2024-04-10', 'Performance'),
(11, 106, 14000, '2024-04-20', 'Performance'),
(12, 122,  7500, '2024-05-01', 'Joining'),
(13, 107,  9500, '2024-05-10', 'Performance'),
(14, 108, 16000, '2024-05-20', 'Performance'),
(15, 109, 17000, '2024-06-01', 'Performance'),
(16, 112, 20000, '2024-06-15', 'Performance'),
(17, 131,  6000, '2024-07-01', 'Referral'),
(18, 128,  5500, '2024-07-10', 'Diwali'),

-- ⚠️ DUPLICATE bonus entries — same employee + amount + date
(19, 101, 10000, '2024-01-10', 'Performance'),  -- duplicate of row 1
(20, 103, 15000, '2024-03-01', 'Performance'),  -- duplicate of row 6
(21, 109, 17000, '2024-06-01', 'Performance'),  -- duplicate of row 15
(22, 112, 20000, '2024-06-15', 'Performance');  -- duplicate of row 16
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `bonus_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_id` | INT | FK → employees |
| `bonus_amount` | INT | Amount ₹ |
| `bonus_date` | DATE | Payout date |
| `bonus_type` | VARCHAR(30) | Performance / Diwali / Joining / Referral |

---

### Step 4 — `attendance` Table

> Daily attendance with status: `Present`, `Absent`, `WFH`, `Half Day`.
> **Intentional duplicates**: 5 rows where the same employee appears twice on the same date.

```sql
CREATE TABLE attendance (
    attendance_id   INT PRIMARY KEY AUTO_INCREMENT,
    employee_id     INT,
    attendance_date DATE,
    status          VARCHAR(20)    -- Present | Absent | WFH | Half Day
);

INSERT INTO attendance (attendance_id, employee_id, attendance_date, status) VALUES
-- January 1
(1,  101, '2025-01-01', 'Present'),
(2,  102, '2025-01-01', 'Absent'),
(3,  103, '2025-01-01', 'Present'),
(4,  104, '2025-01-01', 'WFH'),
(5,  105, '2025-01-01', 'Present'),
(6,  106, '2025-01-01', 'Present'),
(7,  107, '2025-01-01', 'Absent'),
(8,  108, '2025-01-01', 'Present'),
(9,  109, '2025-01-01', 'WFH'),
(10, 110, '2025-01-01', 'Present'),
-- January 2
(11, 111, '2025-01-02', 'Present'),
(12, 112, '2025-01-02', 'Absent'),
(13, 113, '2025-01-02', 'Present'),
(14, 114, '2025-01-02', 'Present'),
(15, 115, '2025-01-02', 'WFH'),
(16, 116, '2025-01-02', 'Present'),
(17, 117, '2025-01-02', 'Present'),
(18, 118, '2025-01-02', 'Absent'),
(19, 119, '2025-01-02', 'Half Day'),
(20, 120, '2025-01-02', 'Present'),
-- January 3
(21, 101, '2025-01-03', 'WFH'),
(22, 102, '2025-01-03', 'Present'),
(23, 103, '2025-01-03', 'Absent'),
(24, 104, '2025-01-03', 'Present'),
(25, 105, '2025-01-03', 'Present'),
(26, 114, '2025-01-03', 'Present'),
(27, 119, '2025-01-03', 'Present'),
(28, 126, '2025-01-03', 'Absent'),
(29, 131, '2025-01-03', 'WFH'),
(30, 136, '2025-01-03', 'Half Day'),

-- ⚠️ DUPLICATE attendance rows — same employee + date + status
(31, 101, '2025-01-01', 'Present'),    -- duplicate of row 1
(32, 107, '2025-01-01', 'Absent'),     -- duplicate of row 7
(33, 114, '2025-01-02', 'Present'),    -- duplicate of row 14
(34, 119, '2025-01-02', 'Half Day'),   -- duplicate of row 19
(35, 103, '2025-01-03', 'Absent');     -- duplicate of row 23
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `attendance_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_id` | INT | FK → employees |
| `attendance_date` | DATE | Date of record |
| `status` | VARCHAR(20) | Present / Absent / WFH / Half Day |

---

### Step 5 — `projects` Table

> Project catalogue with budgets and client assignments.

```sql
CREATE TABLE projects (
    project_id     INT PRIMARY KEY AUTO_INCREMENT,
    project_name   VARCHAR(100),
    project_budget INT,
    client_id      INT,
    start_date     DATE,
    status         VARCHAR(20)    -- Ongoing | Completed | On Hold
);

INSERT INTO projects (project_id, project_name, project_budget, client_id, start_date, status) VALUES
(1, 'CRM System',           500000, 1, '2025-01-01', 'Ongoing'),
(2, 'Banking App',          800000, 2, '2024-06-01', 'Ongoing'),
(3, 'Ecommerce Platform',   650000, 3, '2024-09-01', 'Completed'),
(4, 'Analytics Dashboard',  300000, 4, '2025-02-01', 'On Hold'),
(5, 'HR Management System', 250000, 5, '2025-03-01', 'Ongoing');
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `project_id` | INT AUTO_INCREMENT | Primary Key |
| `project_name` | VARCHAR(100) | Name of project |
| `project_budget` | INT | Budget ₹ |
| `client_id` | INT | FK → clients |
| `start_date` | DATE | Project start date |
| `status` | VARCHAR(20) | Ongoing / Completed / On Hold |

---

### Step 6 — `employee_projects` Table

> Many-to-many bridge between employees and projects.
> **Intentional duplicates**: 2 employees assigned to the same project twice.

```sql
CREATE TABLE employee_projects (
    ep_id         INT PRIMARY KEY AUTO_INCREMENT,
    employee_id   INT,
    project_id    INT,
    assigned_date DATE,
    role          VARCHAR(50)    -- Developer | Tester | Lead | Analyst
);

INSERT INTO employee_projects (ep_id, employee_id, project_id, assigned_date, role) VALUES
(1,  101, 1, '2025-01-01', 'Lead'),
(2,  101, 2, '2025-01-10', 'Developer'),   -- emp 101 on 2 projects
(3,  114, 5, '2025-01-11', 'Analyst'),
(4,  119, 2, '2025-01-15', 'Developer'),
(5,  102, 1, '2025-01-20', 'Tester'),
(6,  126, 3, '2025-01-22', 'Lead'),
(7,  131, 4, '2025-01-25', 'Analyst'),
(8,  103, 1, '2025-01-28', 'Developer'),
(9,  120, 2, '2025-02-01', 'Tester'),
(10, 127, 3, '2025-02-05', 'Developer'),
(11, 115, 5, '2025-02-10', 'Analyst'),
(12, 104, 1, '2025-02-12', 'Developer'),
(13, 132, 4, '2025-02-15', 'Tester'),
(14, 119, 2, '2025-02-18', 'Developer'),   -- ⚠️ emp 119 on project 2 again
(15, 105, 1, '2025-02-20', 'Lead'),
(16, 108, 2, '2025-02-25', 'Developer'),
(17, 109, 1, '2025-03-01', 'Tester'),

-- ⚠️ DUPLICATE project assignments — same employee + project + role
(18, 101, 1, '2025-01-01', 'Lead'),        -- duplicate of row 1
(19, 126, 3, '2025-01-22', 'Lead');        -- duplicate of row 6
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `ep_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_id` | INT | FK → employees |
| `project_id` | INT | FK → projects |
| `assigned_date` | DATE | Assignment date |
| `role` | VARCHAR(50) | Developer / Tester / Lead / Analyst |

---

### Step 7 — `salary_history` Table

> Tracks salary increments over time. Useful for LAG/LEAD window functions and % increment analysis.

```sql
CREATE TABLE salary_history (
    history_id     INT PRIMARY KEY AUTO_INCREMENT,
    employee_id    INT,
    old_salary     INT,
    new_salary     INT,
    increment_date DATE,
    increment_pct  DECIMAL(5,2)
);

INSERT INTO salary_history (history_id, employee_id, old_salary, new_salary, increment_date, increment_pct) VALUES
(1,  101,  65000,  75000, '2024-01-01', 15.38),
(2,  102,  72000,  80000, '2024-01-01', 11.11),
(3,  103,  85000,  95000, '2024-01-01', 11.76),
(4,  104,  70000,  78000, '2024-02-01', 11.43),
(5,  105,  76000,  83000, '2024-02-01',  9.21),
(6,  106,  82000,  91000, '2024-03-01', 10.98),
(7,  107,  69000,  76000, '2024-03-01', 10.14),
(8,  108,  79000,  87000, '2024-04-01', 10.13),
(9,  109,  81000,  88000, '2024-04-01',  8.64),
(10, 112, 130000, 150000, '2024-05-01', 15.38),
(11, 119,  60000,  68000, '2024-01-01', 13.33),
(12, 120,  65000,  72000, '2024-02-01',  7.69),
(13, 126,  40000,  45000, '2024-03-01', 12.50),
(14, 131,  55000,  60000, '2024-04-01',  9.09),
(15, 114,  50000,  55000, '2024-01-01', 10.00);
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `history_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_id` | INT | FK → employees |
| `old_salary` | INT | Salary before increment |
| `new_salary` | INT | Salary after increment |
| `increment_date` | DATE | When increment was applied |
| `increment_pct` | DECIMAL(5,2) | Percentage increment |

---

### Step 8 — `leaves_data` Table

> Leave records per employee across three leave types.

```sql
CREATE TABLE leaves_data (
    leave_id    INT PRIMARY KEY AUTO_INCREMENT,
    employee_id INT,
    leave_type  VARCHAR(50),    -- Sick Leave | Casual Leave | Paid Leave
    days        INT,
    leave_date  DATE
);

INSERT INTO leaves_data (leave_id, employee_id, leave_type, days, leave_date) VALUES
(1,  101, 'Sick Leave',   2, '2025-01-10'),
(2,  114, 'Casual Leave', 1, '2025-01-12'),
(3,  119, 'Paid Leave',   5, '2025-01-15'),
(4,  102, 'Sick Leave',   3, '2025-01-20'),
(5,  126, 'Casual Leave', 2, '2025-01-22'),
(6,  131, 'Paid Leave',   4, '2025-01-25'),
(7,  103, 'Sick Leave',   1, '2025-02-01'),
(8,  120, 'Paid Leave',   6, '2025-02-05'),
(9,  127, 'Casual Leave', 2, '2025-02-10'),
(10, 115, 'Sick Leave',   3, '2025-02-12'),
(11, 104, 'Paid Leave',   2, '2025-02-15'),
(12, 105, 'Casual Leave', 1, '2025-02-18'),
(13, 106, 'Sick Leave',   4, '2025-03-01'),
(14, 108, 'Paid Leave',   3, '2025-03-05'),
(15, 109, 'Casual Leave', 2, '2025-03-10');
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `leave_id` | INT AUTO_INCREMENT | Primary Key |
| `employee_id` | INT | FK → employees |
| `leave_type` | VARCHAR(50) | Sick Leave / Casual Leave / Paid Leave |
| `days` | INT | Number of days taken |
| `leave_date` | DATE | Leave start date |

---

### Step 9 — `clients` Table

> Client master data linked to projects and orders.

```sql
CREATE TABLE clients (
    client_id     INT PRIMARY KEY AUTO_INCREMENT,
    client_name   VARCHAR(100),
    country       VARCHAR(50),
    industry      VARCHAR(50),
    contact_email VARCHAR(100)
);

INSERT INTO clients (client_id, client_name, country, industry, contact_email) VALUES
(1, 'Infosys', 'India', 'IT',        'contact@infosys.com'),
(2, 'HDFC',    'India', 'Banking',   'contact@hdfc.com'),
(3, 'Amazon',  'USA',   'Ecommerce', 'contact@amazon.com'),
(4, 'TCS',     'India', 'IT',        'contact@tcs.com'),
(5, 'Wipro',   'India', 'IT',        'contact@wipro.com');
```

**Schema:**

| Column | Type | Notes |
|---|---|---|
| `client_id` | INT AUTO_INCREMENT | Primary Key |
| `client_name` | VARCHAR(100) | Company name |
| `country` | VARCHAR(50) | Country |
| `industry` | VARCHAR(50) | Sector |
| `contact_email` | VARCHAR(100) | Contact email |

---

### Step 10 — `orders_data` Table

> Sales orders placed by Sales employees for clients. Perfect for revenue analysis, monthly trends, running totals, and ranking.

```sql
CREATE TABLE orders_data (
    order_id     INT PRIMARY KEY AUTO_INCREMENT,
    employee_id  INT,
    client_id    INT,
    order_amount INT,
    order_date   DATE,
    order_status VARCHAR(20)    -- Completed | Pending | Cancelled
);

INSERT INTO orders_data (order_id, employee_id, client_id, order_amount, order_date, order_status) VALUES
-- January
(1,  126, 1, 50000, '2025-01-01', 'Completed'),
(2,  126, 2, 65000, '2025-01-05', 'Completed'),
(3,  127, 3, 70000, '2025-01-10', 'Completed'),
(4,  128, 4, 45000, '2025-01-12', 'Pending'),
(5,  129, 5, 30000, '2025-01-18', 'Completed'),
(6,  130, 1, 55000, '2025-01-22', 'Cancelled'),
-- February
(7,  126, 3, 80000, '2025-02-01', 'Completed'),
(8,  127, 2, 72000, '2025-02-06', 'Completed'),
(9,  128, 5, 39000, '2025-02-10', 'Completed'),
(10, 129, 4, 47000, '2025-02-18', 'Pending'),
(11, 130, 1, 60000, '2025-02-25', 'Completed'),
-- March
(12, 126, 2, 75000, '2025-03-05', 'Completed'),
(13, 127, 3, 82000, '2025-03-10', 'Completed'),
(14, 128, 4, 41000, '2025-03-12', 'Completed'),
(15, 129, 5, 35000, '2025-03-18', 'Completed'),
(16, 130, 2, 68000, '2025-03-22', 'Pending'),
(17, 126, 4, 90000, '2025-03-28', 'Completed'),
(18, 127, 1, 55000, '2025-03-30', 'Completed');
```


> ⭐ **Star this repo** if it helped you! Issues and PRs are always welcome.