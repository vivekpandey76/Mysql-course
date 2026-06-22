# employer_database — MySQL Dataset

## Database Setup

```sql
CREATE DATABASE employer_database;
USE employer_database;
```

---

## Tables

| # | Table | Rows | Duplicates |
|---|---|---|---|
| 1 | `employees` | 52 | Yes (IDs 48–52) |
| 2 | `bonuses` | 22 | Yes (rows 19–22) |
| 3 | `attendance` | 35 | Yes (rows 31–35) |
| 4 | `projects` | 5 | — |
| 5 | `employee_projects` | 19 | Yes (rows 18–19) |
| 6 | `salary_history` | 16 | — |
| 7 | `leaves_data` | 15 | — |
| 8 | `clients` | 5 | — |
| 9 | `orders_data` | 18 | — |

---

## Entity Relationships

```
employees (self) ────────────── (self) employees   [manager_id → employee_id]
employees      (1) ──────────── (N)    bonuses
employees      (1) ──────────── (N)    attendance
employees      (N) ──────────── (N)    projects     [via employee_projects]
employees      (1) ──────────── (N)    salary_history
employees      (1) ──────────── (N)    leaves_data
employees      (1) ──────────── (N)    orders_data
clients        (1) ──────────── (N)    orders_data
clients        (1) ──────────── (N)    projects
```

---

## employees

```sql
CREATE TABLE employees (
    employee_id       INT PRIMARY KEY,
    employee_name     VARCHAR(100),
    gender            VARCHAR(10),
    department        VARCHAR(50),
    salary            INT,
    manager_id        INT,
    hire_date         DATE,
    city              VARCHAR(50),
    state             VARCHAR(50),
    email             VARCHAR(100),
    employment_status VARCHAR(20),
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

INSERT INTO employees (employee_id, employee_name, gender, department, salary, manager_id, hire_date, city, state, email, employment_status) VALUES
(1, 'Raj Sharma',   'Male',   'IT',        120000, NULL, '2012-03-01', 'Mumbai',    'Maharashtra', 'raj.sharma@company.com',   'Active'),
(2, 'Ankit Verma',  'Male',   'HR',         90000, NULL, '2015-06-15', 'Delhi',     'Delhi',       'ankit.verma@company.com',  'Active'),
(3, 'Priya Singh',  'Female', 'Finance',   100000, NULL, '2010-09-20', 'Pune',      'Maharashtra', 'priya.singh@company.com',  'Active'),
(4, 'Amit Patel',   'Male',   'Sales',      85000, NULL, '2017-01-10', 'Bangalore', 'Karnataka',   'amit.patel@company.com',   'Active'),
(5, 'Neha Joshi',   'Female', 'Marketing',  88000, NULL, '2014-04-05', 'Hyderabad', 'Telangana',   'neha.joshi@company.com',   'Active'),
(6, 'Rohit Mehta',  'Male',   'Support',    75000, NULL, '2018-07-22', 'Chennai',   'Tamil Nadu',  'rohit.mehta@company.com',  'Active'),
(7, 'Sneha Kapoor', 'Female', 'IT',        115000, NULL, '2011-11-30', 'Mumbai',    'Maharashtra', 'sneha.kapoor@company.com', 'Active'),
(8, 'Kunal Shah',   'Male',   'Finance',   105000, NULL, '2013-02-14', 'Ahmedabad', 'Gujarat',     'kunal.shah@company.com',   'Active'),
(9,  'Aarav Sharma',  'Male',   'IT',        75000,  1, '2021-01-15', 'Mumbai',    'Maharashtra', 'aarav@gmail.com',    'Active'),
(10, 'Priya Mehta',   'Female', 'HR',        55000,  2, '2020-03-10', 'Delhi',     'Delhi',       'priya@gmail.com',    'Active'),
(11, 'Rohit Jain',    'Male',   'Finance',   68000,  3, '2019-07-23', 'Pune',      'Maharashtra', 'rohit@gmail.com',    'Active'),
(12, 'Sneha Patil',   'Female', 'IT',        80000,  1, '2022-02-12', 'Mumbai',    'Maharashtra', 'sneha@gmail.com',    'Active'),
(13, 'Vikas Singh',   'Male',   'Sales',     45000,  4, '2021-11-05', 'Bangalore', 'Karnataka',   'vikas@gmail.com',    'Active'),
(14, 'Anjali Verma',  'Female', 'Marketing', 60000,  5, '2020-09-17', 'Hyderabad', 'Telangana',   'anjali@gmail.com',   'Active'),
(15, 'Karan Shah',    'Male',   'IT',        95000,  1, '2018-04-19', 'Mumbai',    'Maharashtra', 'karan@gmail.com',    'Active'),
(16, 'Meera Joshi',   'Female', 'HR',        53000,  2, '2021-06-14', 'Pune',      'Maharashtra', 'meera@gmail.com',    'Active'),
(17, 'Pooja Nair',    'Female', 'Finance',   72000,  3, '2019-12-01', 'Chennai',   'Tamil Nadu',  'pooja@gmail.com',    'Active'),
(18, 'Low Salary',    'Male',   'Support',   25000,  6, '2023-01-01', 'Pune',      'Maharashtra', 'low@gmail.com',      'Active'),
(19, 'Arjun Gupta',   'Male',   'IT',        78000,  1, '2020-08-11', 'Mumbai',    'Maharashtra', 'arjun@gmail.com',    'Active'),
(20, 'Rahul Kapoor',  'Male',   'Sales',     47000,  4, '2023-01-08', 'Delhi',     'Delhi',       'rahul@gmail.com',    'Active'),
(21, 'Simran Kaur',   'Female', 'Marketing', 62000,  5, '2022-10-09', 'Chandigarh','Punjab',      'simran@gmail.com',   'Active'),
(22, 'Aman Mishra',   'Male',   'HR',        54000,  2, '2021-04-30', 'Lucknow',   'UP',          'aman@gmail.com',     'Active'),
(23, 'Dev Malhotra',  'Male',   'Finance',   67000,  3, '2017-03-15', 'Delhi',     'Delhi',       'dev@gmail.com',      'Active'),
(24, 'Nisha Rao',     'Female', 'IT',        83000,  1, '2021-05-25', 'Bangalore', 'Karnataka',   'nisha@gmail.com',    'Active'),
(25, 'Ravi Kumar',    'Male',   'Support',   30000,  6, '2022-05-15', 'Chennai',   'Tamil Nadu',  'ravi@gmail.com',     'Active'),
(26, 'Yash Thakur',   'Male',   'Sales',     50000,  4, '2022-01-20', 'Mumbai',    'Maharashtra', 'yash@gmail.com',     'Active'),
(27, 'Ritika Sen',    'Female', 'Marketing', 59000,  5, '2019-11-13', 'Kolkata',   'West Bengal', 'ritika@gmail.com',   'Active'),
(28, 'Tanvi Desai',   'Female', 'HR',        52000,  2, '2019-09-27', 'Ahmedabad', 'Gujarat',     'tanvi@gmail.com',    'On Leave'),
(29, 'Mohit Arora',   'Male',   'IT',        91000,  7, '2018-07-07', 'Delhi',     'Delhi',       'mohit@gmail.com',    'Active'),
(30, 'Kriti Sharma',  'Female', 'Finance',   71000,  3, '2020-12-21', 'Pune',      'Maharashtra', 'kriti@gmail.com',    'Active'),
(31, 'Neha Iyer',     'Female', 'Sales',     49000,  4, '2023-03-18', 'Chennai',   'Tamil Nadu',  'nehaiyer@gmail.com', 'Active'),
(32, 'Varun Sethi',   'Male',   'Marketing', 61000,  5, '2020-01-05', 'Delhi',     'Delhi',       'varun@gmail.com',    'Active'),
(33, 'Harsh Pandey',  'Male',   'IT',        76000,  7, '2022-06-11', 'Mumbai',    'Maharashtra', 'harsh@gmail.com',    'On Leave'),
(34, 'Divya Pillai',  'Female', 'Support',   32000,  6, '2021-09-10', 'Hyderabad', 'Telangana',   'divya@gmail.com',    'On Leave'),
(35, 'Isha Kulkarni', 'Female', 'Finance',   69000,  3, '2021-08-14', 'Pune',      'Maharashtra', 'isha@gmail.com',     'Active'),
(36, 'Sonal Jain',    'Female', 'HR',        51000,  2, '2023-04-04', 'Jaipur',    'Rajasthan',   'sonal@gmail.com',    'Active'),
(37, 'Aisha Khan',    'Female', 'Sales',     46000,  4, '2022-02-28', 'Hyderabad', 'Telangana',   'aisha@gmail.com',    'Inactive'),
(38, 'Siddharth Roy', 'Male',   'IT',        87000,  1, '2018-05-17', 'Kolkata',   'West Bengal', 'sid@gmail.com',      'Active'),
(39, 'Preeti Das',    'Female', 'Marketing', 64000,  5, '2020-10-10', 'Kolkata',   'West Bengal', 'preeti@gmail.com',   'Terminated'),
(40, 'Rajat Saxena',  'Male',   'Finance',   73000,  3, '2019-06-06', 'Delhi',     'Delhi',       'rajat@gmail.com',    'Terminated'),
(41, 'Akash Yadav',   'Male',   'IT',        88000,  7, '2017-08-22', 'Lucknow',   'UP',          'akash@gmail.com',    'Active'),
(42, 'No Manager',    'Male',   'Finance',   65000, NULL, '2020-02-20', 'Nagpur',  'Maharashtra', 'nomgr@gmail.com',    'Active'),
(43, 'Same Salary A', 'Female', 'IT',        70000,  1, '2021-03-03', 'Mumbai',    'Maharashtra', 'same1@gmail.com',    'Active'),
(44, 'Same Salary B', 'Male',   'IT',        70000,  1, '2021-03-04', 'Mumbai',    'Maharashtra', 'same2@gmail.com',    'Active'),
(45, 'High Salary',   'Female', 'IT',       150000,  1, '2016-05-15', 'Mumbai',    'Maharashtra', 'high@gmail.com',     'Active'),
(46, 'Test User',     'Female', 'IT',          NULL,  1, '2022-09-01', 'Mumbai',   'Maharashtra', 'test@gmail.com',     'Inactive'),
(47, 'NULL Employee', 'Male',    NULL,       40000, NULL, '2023-02-10', 'Indore',  NULL,          NULL,                 'Inactive'),
(48, 'Aarav Sharma',  'Male',   'IT',        75000,  1, '2021-01-15', 'Mumbai',    'Maharashtra', 'aarav@gmail.com',    'Active'),
(49, 'Rohit Jain',    'Male',   'Finance',   68000,  3, '2019-07-23', 'Pune',      'Maharashtra', 'rohit@gmail.com',    'Active'),
(50, 'Priya Mehta',   'Female', 'HR',        55000,  2, '2020-03-10', 'Delhi',     'Delhi',       'priya@gmail.com',    'Active'),
(51, 'Vikas Singh',   'Male',   'Sales',     45000,  4, '2021-11-05', 'Bangalore', 'Karnataka',   'vikas@gmail.com',    'Active'),
(52, 'Anjali Verma',  'Female', 'Marketing', 60000,  5, '2020-09-17', 'Hyderabad', 'Telangana',   'anjali@gmail.com',   'Active');
```

---

## bonuses

```sql
CREATE TABLE bonuses (
    bonus_id     INT PRIMARY KEY AUTO_INCREMENT,
    employee_id  INT,
    bonus_amount INT,
    bonus_date   DATE,
    bonus_type   VARCHAR(30)
);

INSERT INTO bonuses (bonus_id, employee_id, bonus_amount, bonus_date, bonus_type) VALUES
(1,  9,  10000, '2024-01-10', 'Performance'),
(2,  22,  5000, '2024-01-15', 'Performance'),
(3,  27,  7000, '2024-02-01', 'Performance'),
(4,  10, 12000, '2024-02-05', 'Performance'),
(5,  34,  3000, '2024-02-10', 'Diwali'),
(6,  11, 15000, '2024-03-01', 'Performance'),
(7,  28,  8000, '2024-03-05', 'Performance'),
(8,  23,  4000, '2024-03-15', 'Diwali'),
(9,  12,  9000, '2024-04-01', 'Performance'),
(10, 13, 11000, '2024-04-10', 'Performance'),
(11, 14, 14000, '2024-04-20', 'Performance'),
(12, 30,  7500, '2024-05-01', 'Joining'),
(13, 15,  9500, '2024-05-10', 'Performance'),
(14, 16, 16000, '2024-05-20', 'Performance'),
(15, 17, 17000, '2024-06-01', 'Performance'),
(16, 20, 20000, '2024-06-15', 'Performance'),
(17, 39,  6000, '2024-07-01', 'Referral'),
(18, 36,  5500, '2024-07-10', 'Diwali'),
(19, 9,  10000, '2024-01-10', 'Performance'),
(20, 11, 15000, '2024-03-01', 'Performance'),
(21, 17, 17000, '2024-06-01', 'Performance'),
(22, 20, 20000, '2024-06-15', 'Performance');
```

---

## attendance

```sql
CREATE TABLE attendance (
    attendance_id   INT PRIMARY KEY AUTO_INCREMENT,
    employee_id     INT,
    attendance_date DATE,
    status          VARCHAR(20)
);

INSERT INTO attendance (attendance_id, employee_id, attendance_date, status) VALUES
(1,  9,  '2025-01-01', 'Present'),
(2,  10, '2025-01-01', 'Absent'),
(3,  11, '2025-01-01', 'Present'),
(4,  12, '2025-01-01', 'WFH'),
(5,  13, '2025-01-01', 'Present'),
(6,  14, '2025-01-01', 'Present'),
(7,  15, '2025-01-01', 'Absent'),
(8,  16, '2025-01-01', 'Present'),
(9,  17, '2025-01-01', 'WFH'),
(10, 18, '2025-01-01', 'Present'),
(11, 19, '2025-01-02', 'Present'),
(12, 20, '2025-01-02', 'Absent'),
(13, 21, '2025-01-02', 'Present'),
(14, 22, '2025-01-02', 'Present'),
(15, 23, '2025-01-02', 'WFH'),
(16, 24, '2025-01-02', 'Present'),
(17, 25, '2025-01-02', 'Present'),
(18, 26, '2025-01-02', 'Absent'),
(19, 27, '2025-01-02', 'Half Day'),
(20, 28, '2025-01-02', 'Present'),
(21, 9,  '2025-01-03', 'WFH'),
(22, 10, '2025-01-03', 'Present'),
(23, 11, '2025-01-03', 'Absent'),
(24, 12, '2025-01-03', 'Present'),
(25, 13, '2025-01-03', 'Present'),
(26, 22, '2025-01-03', 'Present'),
(27, 27, '2025-01-03', 'Present'),
(28, 34, '2025-01-03', 'Absent'),
(29, 39, '2025-01-03', 'WFH'),
(30, 44, '2025-01-03', 'Half Day'),
(31, 9,  '2025-01-01', 'Present'),
(32, 15, '2025-01-01', 'Absent'),
(33, 22, '2025-01-02', 'Present'),
(34, 27, '2025-01-02', 'Half Day'),
(35, 11, '2025-01-03', 'Absent');
```

---

## projects

```sql
CREATE TABLE projects (
    project_id     INT PRIMARY KEY AUTO_INCREMENT,
    project_name   VARCHAR(100),
    project_budget INT,
    client_id      INT,
    start_date     DATE,
    status         VARCHAR(20)
);

INSERT INTO projects (project_id, project_name, project_budget, client_id, start_date, status) VALUES
(1, 'CRM System',           500000, 1, '2025-01-01', 'Ongoing'),
(2, 'Banking App',          800000, 2, '2024-06-01', 'Ongoing'),
(3, 'Ecommerce Platform',   650000, 3, '2024-09-01', 'Completed'),
(4, 'Analytics Dashboard',  300000, 4, '2025-02-01', 'On Hold'),
(5, 'HR Management System', 250000, 5, '2025-03-01', 'Ongoing');
```

---

## employee_projects

```sql
CREATE TABLE employee_projects (
    ep_id         INT PRIMARY KEY AUTO_INCREMENT,
    employee_id   INT,
    project_id    INT,
    assigned_date DATE,
    role          VARCHAR(50)
);

INSERT INTO employee_projects (ep_id, employee_id, project_id, assigned_date, role) VALUES
(1,  9,  1, '2025-01-01', 'Lead'),
(2,  9,  2, '2025-01-10', 'Developer'),
(3,  22, 5, '2025-01-11', 'Analyst'),
(4,  27, 2, '2025-01-15', 'Developer'),
(5,  10, 1, '2025-01-20', 'Tester'),
(6,  34, 3, '2025-01-22', 'Lead'),
(7,  39, 4, '2025-01-25', 'Analyst'),
(8,  11, 1, '2025-01-28', 'Developer'),
(9,  28, 2, '2025-02-01', 'Tester'),
(10, 35, 3, '2025-02-05', 'Developer'),
(11, 23, 5, '2025-02-10', 'Analyst'),
(12, 12, 1, '2025-02-12', 'Developer'),
(13, 40, 4, '2025-02-15', 'Tester'),
(14, 27, 2, '2025-02-18', 'Developer'),
(15, 13, 1, '2025-02-20', 'Lead'),
(16, 16, 2, '2025-02-25', 'Developer'),
(17, 17, 1, '2025-03-01', 'Tester'),
(18, 9,  1, '2025-01-01', 'Lead'),
(19, 34, 3, '2025-01-22', 'Lead');
```

---

## salary_history

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
(1,  9,  65000, 75000, '2024-01-01', 15.38),
(2,  10, 48000, 55000, '2024-02-01', 14.58),
(3,  11, 60000, 68000, '2024-01-01', 13.33),
(4,  12, 72000, 80000, '2024-02-01', 11.11),
(5,  13, 40000, 45000, '2024-03-01', 12.50),
(6,  14, 53000, 60000, '2024-03-01', 13.21),
(7,  15, 85000, 95000, '2024-04-01', 11.76),
(8,  16, 47000, 53000, '2024-04-01', 12.77),
(9,  17, 65000, 72000, '2024-05-01', 10.77),
(10, 20, 42000, 47000, '2024-03-01', 11.90),
(11, 22, 48000, 54000, '2024-01-01', 12.50),
(12, 23, 60000, 67000, '2024-02-01', 11.67),
(13, 27, 52000, 59000, '2024-04-01', 13.46),
(14, 28, 46000, 52000, '2024-03-01', 13.04),
(15, 34, 28000, 32000, '2024-05-01', 14.29),
(16, 39, 58000, 64000, '2024-02-01', 10.34);
```

---

## leaves_data

```sql
CREATE TABLE leaves_data (
    leave_id    INT PRIMARY KEY AUTO_INCREMENT,
    employee_id INT,
    leave_type  VARCHAR(50),
    days        INT,
    leave_date  DATE
);

INSERT INTO leaves_data (leave_id, employee_id, leave_type, days, leave_date) VALUES
(1,  9,  'Sick Leave',   2, '2025-01-10'),
(2,  22, 'Casual Leave', 1, '2025-01-12'),
(3,  27, 'Paid Leave',   5, '2025-01-15'),
(4,  10, 'Sick Leave',   3, '2025-01-20'),
(5,  34, 'Casual Leave', 2, '2025-01-22'),
(6,  39, 'Paid Leave',   4, '2025-01-25'),
(7,  11, 'Sick Leave',   1, '2025-02-01'),
(8,  28, 'Paid Leave',   6, '2025-02-05'),
(9,  35, 'Casual Leave', 2, '2025-02-10'),
(10, 23, 'Sick Leave',   3, '2025-02-12'),
(11, 12, 'Paid Leave',   2, '2025-02-15'),
(12, 13, 'Casual Leave', 1, '2025-02-18'),
(13, 14, 'Sick Leave',   4, '2025-03-01'),
(14, 16, 'Paid Leave',   3, '2025-03-05'),
(15, 17, 'Casual Leave', 2, '2025-03-10');
```

---

## clients

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

---

## orders_data

```sql
CREATE TABLE orders_data (
    order_id     INT PRIMARY KEY AUTO_INCREMENT,
    employee_id  INT,
    client_id    INT,
    order_amount INT,
    order_date   DATE,
    order_status VARCHAR(20)
);

INSERT INTO orders_data (order_id, employee_id, client_id, order_amount, order_date, order_status) VALUES
(1,  13, 1, 50000, '2025-01-05', 'Completed'),
(2,  20, 2, 65000, '2025-01-08', 'Completed'),
(3,  26, 3, 70000, '2025-01-12', 'Completed'),
(4,  31, 4, 45000, '2025-01-15', 'Pending'),
(5,  37, 5, 30000, '2025-01-20', 'Completed'),
(6,  13, 2, 55000, '2025-01-25', 'Cancelled'),
(7,  13, 3, 80000, '2025-02-03', 'Completed'),
(8,  20, 1, 72000, '2025-02-07', 'Completed'),
(9,  26, 5, 39000, '2025-02-12', 'Completed'),
(10, 31, 2, 47000, '2025-02-18', 'Pending'),
(11, 37, 4, 60000, '2025-02-24', 'Completed'),
(12, 13, 1, 75000, '2025-03-04', 'Completed'),
(13, 20, 3, 82000, '2025-03-09', 'Completed'),
(14, 26, 4, 41000, '2025-03-14', 'Completed'),
(15, 31, 5, 35000, '2025-03-19', 'Completed'),
(16, 37, 3, 68000, '2025-03-23', 'Pending'),
(17, 13, 4, 90000, '2025-03-28', 'Completed'),
(18, 20, 2, 55000, '2025-03-31', 'Completed');
```

---

## Employee ID Reference Map

| ID | Name | Department | manager_id |
|---|---|---|---|
| 1 | Raj Sharma | IT | NULL |
| 2 | Ankit Verma | HR | NULL |
| 3 | Priya Singh | Finance | NULL |
| 4 | Amit Patel | Sales | NULL |
| 5 | Neha Joshi | Marketing | NULL |
| 6 | Rohit Mehta | Support | NULL |
| 7 | Sneha Kapoor | IT | NULL |
| 8 | Kunal Shah | Finance | NULL |
| 9 | Aarav Sharma | IT | 1 |
| 10 | Priya Mehta | HR | 2 |
| 11 | Rohit Jain | Finance | 3 |
| 12 | Sneha Patil | IT | 1 |
| 13 | Vikas Singh | Sales | 4 |
| 14 | Anjali Verma | Marketing | 5 |
| 15 | Karan Shah | IT | 1 |
| 16 | Meera Joshi | HR | 2 |
| 17 | Pooja Nair | Finance | 3 |
| 18 | Low Salary | Support | 6 |
| 19 | Arjun Gupta | IT | 1 |
| 20 | Rahul Kapoor | Sales | 4 |
| 21 | Simran Kaur | Marketing | 5 |
| 22 | Aman Mishra | HR | 2 |
| 23 | Dev Malhotra | Finance | 3 |
| 24 | Nisha Rao | IT | 1 |
| 25 | Ravi Kumar | Support | 6 |
| 26 | Yash Thakur | Sales | 4 |
| 27 | Ritika Sen | Marketing | 5 |
| 28 | Tanvi Desai | HR | 2 |
| 29 | Mohit Arora | IT | 7 |
| 30 | Kriti Sharma | Finance | 3 |
| 31 | Neha Iyer | Sales | 4 |
| 32 | Varun Sethi | Marketing | 5 |
| 33 | Harsh Pandey | IT | 7 |
| 34 | Divya Pillai | Support | 6 |
| 35 | Isha Kulkarni | Finance | 3 |
| 36 | Sonal Jain | HR | 2 |
| 37 | Aisha Khan | Sales | 4 |
| 38 | Siddharth Roy | IT | 1 |
| 39 | Preeti Das | Marketing | 5 |
| 40 | Rajat Saxena | Finance | 3 |
| 41 | Akash Yadav | IT | 7 |
| 42 | No Manager | Finance | NULL |
| 43 | Same Salary A | IT | 1 |
| 44 | Same Salary B | IT | 1 |
| 45 | High Salary | IT | 1 |
| 46 | Test User | IT | 1 |
| 47 | NULL Employee | NULL | NULL |
| 48 | Aarav Sharma | IT | 1 |
| 49 | Rohit Jain | Finance | 3 |
| 50 | Priya Mehta | HR | 2 |
| 51 | Vikas Singh | Sales | 4 |
| 52 | Anjali Verma | Marketing | 5 |

---

## Duplicate Records Map

| Table | Duplicate Logic | Affected Rows |
|---|---|---|
| `employees` | Same name + dept + salary + email | IDs 48–52 (copies of 9–14) |
| `bonuses` | Same employee + amount + date | Rows 19–22 |
| `attendance` | Same employee + date + status | Rows 31–35 |
| `employee_projects` | Same employee + project + role + date | Rows 18–19 |
