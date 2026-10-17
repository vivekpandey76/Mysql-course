# 🏢 Accenture SQL Interview Questions

> **Company-based set: Accenture** — fifteen questions reported by candidates on AmbitionBox for Data Engineer, Data Engineering Analyst, SQL Developer and Software Engineer roles. A mix of query problems (latest login per country, never-sold products, pivots, de-duplication, rolling averages) and the theory questions Accenture uses as screening filters (`RANK` vs `DENSE_RANK`, `WHERE` vs `HAVING`, `DELETE`/`TRUNCATE`/`DROP`, query optimisation).
> Each problem follows the same flow: **Question → Dataset → Expected Output → Solution → Explanation.**

Every problem is self-contained: run the *Dataset* block, then run the solution to reproduce the expected output. All queries target **MySQL 8.0+** and were executed against a local MySQL 9.6 server. Every Expected Output block is real query output.

---

## 📋 Contents

| # | Problem | Core Concept |
|---|---------|--------------|
| 1 | [Person who logged in last in each country](#q1--person-who-logged-in-last-in-each-country) | `DENSE_RANK()` + `PARTITION BY` |
| 2 | [`RANK` vs `DENSE_RANK` vs `ROW_NUMBER`](#q2--rank-vs-dense_rank-vs-row_number) | Ranking functions and ties |
| 3 | [Products that were never sold](#q3--products-that-were-never-sold) | Anti-join: `LEFT JOIN … IS NULL` / `NOT EXISTS` |
| 4 | [Total age of people older than 30](#q4--total-age-of-people-older-than-30) | `WHERE` filter vs conditional aggregation |
| 5 | [PIVOT and UNPIVOT](#q5--pivot-and-unpivot) | `SUM(CASE …)` pivot, `UNION ALL` unpivot |
| 6 | [SQL query optimisation techniques](#q6--sql-query-optimisation-techniques) | Indexes, `EXPLAIN`, filtering early |
| 7 | [Second highest salary in each department](#q7--second-highest-salary-in-each-department) | `DENSE_RANK()` top-N per group |
| 8 | [Find and remove duplicate rows by email](#q8--find-and-remove-duplicate-rows-by-email) | `GROUP BY … HAVING` + self-join `DELETE` |
| 9 | [Row counts for each join type](#q9--row-counts-for-each-join-type-on-val) | Join cardinality with duplicates and `NULL`s |
| 10 | [Rolling 7-day average of daily orders](#q10--rolling-7-day-average-of-daily-orders) | Recursive date spine + `RANGE` frame |
| 11 | [`WHERE` vs `HAVING`](#q11--difference-between-where-vs-having) | Row filter vs group filter |
| 12 | [Departments with more than 5 employees](#q12--departments-with-more-than-5-employees) | `GROUP BY` + `HAVING COUNT(*)` |
| 13 | [`DELETE` vs `TRUNCATE` vs `DROP`](#q13--difference-between-delete-truncate-and-drop) | DML vs DDL |
| 14 | [`INNER JOIN` vs `LEFT JOIN`](#q14--difference-between-inner-join-vs-left-join) | Matched rows vs all left rows |
| 15 | [The big spenders](#q15--the-big-spenders) | `HAVING` against an average of aggregates |

---

## Q1 — Person Who Logged In Last in Each Country

**Source:** AmbitionBox

**Question:** From the `person_logins` table, find the person (or people) who logged in **most recently** in each country.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS person_logins;
CREATE TABLE person_logins (
    login_id    INT PRIMARY KEY,
    person_name VARCHAR(30) NOT NULL,
    country     VARCHAR(20) NOT NULL,
    login_time  DATETIME NULL
);

INSERT INTO person_logins VALUES
( 1,'Aarav Shah',    'India',   '2025-06-01 09:10:00'),
( 2,'Aarav Shah',    'India',   '2025-06-03 18:45:00'),
( 3,'Diya Nair',     'India',   '2025-06-03 21:05:00'),
( 4,'Kabir Singh',   'India',   NULL),
( 5,'Emma Clarke',   'UK',      '2025-06-02 08:00:00'),
( 6,'Oliver Hughes', 'UK',      '2025-06-04 11:30:00'),
( 7,'Emma Clarke',   'UK',      '2025-06-04 11:30:00'),
( 8,'Liam Walsh',    'Ireland', '2025-06-01 07:15:00'),
( 9,'Sofia Rossi',   'Italy',   '2025-05-30 22:40:00'),
(10,'Marco Bianchi', 'Italy',   '2025-06-02 13:00:00'),
(11,'Sofia Rossi',   'Italy',   '2025-06-01 10:00:00'),
(12,'Diya Nair',     'India',   '2025-06-02 12:00:00'),
(13,'Noah Becker',   'Germany', NULL);
```

### ✅ Expected Output

| person_name | country | login_time |
|---|---|---|
| Diya Nair | India | 2025-06-03 21:05:00 |
| Liam Walsh | Ireland | 2025-06-01 07:15:00 |
| Marco Bianchi | Italy | 2025-06-02 13:00:00 |
| Emma Clarke | UK | 2025-06-04 11:30:00 |
| Oliver Hughes | UK | 2025-06-04 11:30:00 |

### 💡 Solution

**Method 1: Subquery**

```sql
SELECT person_name, country, login_time
FROM (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY country ORDER BY login_time DESC) AS ranking
    FROM person_logins
    WHERE login_time IS NOT NULL
) t
WHERE ranking = 1
ORDER BY country, person_name;
```

**Method 2: CTE**

```sql
WITH ranked_person AS (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY country ORDER BY login_time DESC) AS ranking
    FROM person_logins
    WHERE login_time IS NOT NULL
)
SELECT person_name, country, login_time
FROM ranked_person
WHERE ranking = 1
ORDER BY country, person_name;
```

### 🧠 Explanation

- `PARTITION BY country` restarts the ranking for every country, and `ORDER BY login_time DESC` puts the latest login at rank 1.
- **Why `DENSE_RANK` and not `ROW_NUMBER`:** Emma Clarke and Oliver Hughes both logged in at `2025-06-04 11:30:00` in the UK. `DENSE_RANK` gives both rank 1, so both are returned. `ROW_NUMBER` would pick one of them arbitrarily and silently drop the other.
- **Why filter `login_time IS NOT NULL`:** Germany has only a `NULL` login, and India has one too. Filtering them out means a country with no real login simply doesn't appear, instead of reporting a `NULL` as the "latest" login.
- Both methods produce the same plan. The CTE version reads top-down, which is easier to explain in an interview.

---

## Q2 — RANK vs DENSE_RANK vs ROW_NUMBER

**Source:** AmbitionBox, Data Engineer

**Question:** Explain the difference between `ROW_NUMBER()`, `RANK()` and `DENSE_RANK()`, and show all three on the same data.

### 📖 Theory

| Function | Tied values | Gaps after a tie? | Example on `96, 96, 91` |
|---|---|---|---|
| `ROW_NUMBER()` | Different numbers, even when values are equal | n/a | 1, 2, 3 |
| `RANK()` | Same rank | **Yes**, skips the next ranks | 1, 1, 3 |
| `DENSE_RANK()` | Same rank | **No**, next rank is consecutive | 1, 1, 2 |

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS consultant_scores;
CREATE TABLE consultant_scores (
    consultant_id   INT PRIMARY KEY,
    consultant      VARCHAR(20) NOT NULL,
    utilisation_pct INT NULL
);

INSERT INTO consultant_scores VALUES
(1,'Ananya', 96),(2,'Bharat', 96),(3,'Chetan', 91),(4,'Devika', 88),
(5,'Eshan',  88),(6,'Farah',  88),(7,'Gopal',  80),(8,'Hema',   75),
(9,'Irfan',  NULL),(10,'Jaya', 70);
```

### ✅ Expected Output

| consultant | utilisation_pct | row_num | rnk | dense_rnk |
|---|---|---|---|---|
| Ananya | 96 | 1 | 1 | 1 |
| Bharat | 96 | 2 | 1 | 1 |
| Chetan | 91 | 3 | 3 | 2 |
| Devika | 88 | 4 | 4 | 3 |
| Eshan | 88 | 5 | 4 | 3 |
| Farah | 88 | 6 | 4 | 3 |
| Gopal | 80 | 7 | 7 | 4 |
| Hema | 75 | 8 | 8 | 5 |
| Jaya | 70 | 9 | 9 | 6 |

### 💡 Solution

```sql
SELECT consultant,
       utilisation_pct,
       ROW_NUMBER() OVER (ORDER BY utilisation_pct DESC, consultant_id) AS row_num,
       RANK()       OVER (ORDER BY utilisation_pct DESC)                AS rnk,
       DENSE_RANK() OVER (ORDER BY utilisation_pct DESC)                AS dense_rnk
FROM consultant_scores
WHERE utilisation_pct IS NOT NULL;
```

### 🧠 Explanation

- Look at the three 88s: `RANK` gives them all 4, then **jumps to 7** for Gopal (positions 5 and 6 are "used up"). `DENSE_RANK` gives them 3 and moves on to 4.
- **Give each column its own alias.** Three columns all named `ranking` are legal in a result set but impossible to tell apart or reference later.
- **`ROW_NUMBER` needs a tie-breaker.** Without `consultant_id` in its `ORDER BY`, Ananya and Bharat can swap row numbers between runs. Adding a unique column makes the numbering repeatable.
- **Which one to use:** `ROW_NUMBER` for "exactly one row per group" (de-duplication, latest record). `DENSE_RANK` for "Nth highest value" questions (Q7). `RANK` for leaderboards where a tie should push the next person down.
- Irfan's `NULL` is filtered out. Left in, MySQL sorts `NULL` last under `DESC`, which would rank him 10th instead of excluding him.

---

## Q3 — Products That Were Never Sold

**Source:** AmbitionBox, Azure Data Engineer

**Question:** Given a `products` table and an `orders` table, list the products that have **never been ordered**.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
CREATE TABLE products (
    product_id   INT PRIMARY KEY,
    product_name VARCHAR(30) NOT NULL,
    category     VARCHAR(20) NOT NULL
);
CREATE TABLE orders (
    order_id   INT PRIMARY KEY,
    product_id INT NULL,
    qty        INT NOT NULL,
    status     VARCHAR(12) NOT NULL,
    order_date DATE NOT NULL
);

INSERT INTO products VALUES
(1,'Laptop Stand',   'Accessories'),
(2,'USB-C Hub',      'Accessories'),
(3,'Noise Headset',  'Audio'),
(4,'4K Monitor',     'Displays'),
(5,'Ergo Chair',     'Furniture'),
(6,'Webcam Pro',     'Video'),
(7,'Desk Lamp',      'Furniture'),
(8,'Wireless Mouse', 'Accessories');

INSERT INTO orders VALUES
(101, 1,    2, 'DELIVERED','2025-04-02'),
(102, 2,    1, 'DELIVERED','2025-04-03'),
(103, 1,    1, 'DELIVERED','2025-04-05'),
(104, 4,    1, 'CANCELLED','2025-04-06'),
(105, NULL, 3, 'DELIVERED','2025-04-07'),
(106, 8,    5, 'DELIVERED','2025-04-09'),
(107, 2,    2, 'RETURNED', '2025-04-10'),
(108, 99,   1, 'DELIVERED','2025-04-11'),
(109, 8,    1, 'DELIVERED','2025-04-12');
```

### ✅ Expected Output

| product_id | product_name |
|---|---|
| 3 | Noise Headset |
| 5 | Ergo Chair |
| 6 | Webcam Pro |
| 7 | Desk Lamp |

### 💡 Solution

**Method 1: `LEFT JOIN` + `IS NULL`**

```sql
SELECT p.product_id, p.product_name
FROM products p
LEFT JOIN orders o
       ON p.product_id = o.product_id
WHERE o.order_id IS NULL
ORDER BY p.product_id;
```

**Method 2: `NOT EXISTS`**

```sql
SELECT p.product_id, p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.product_id = p.product_id
)
ORDER BY p.product_id;
```

### 🧠 Explanation

- **Method 1:** the `LEFT JOIN` keeps every product. Products with no order get `NULL` in every `orders` column, and `WHERE o.order_id IS NULL` keeps only those. Check a column that can never be `NULL` on a real match, like the primary key `order_id`.
- **Method 2:** `NOT EXISTS` stops at the first matching order, so it never multiplies rows. What goes in the `SELECT` inside it doesn't matter; `SELECT 1` is the convention.
- ⚠️ **Don't use `NOT IN` here.** `WHERE product_id NOT IN (SELECT product_id FROM orders)` returns **0 rows** on this data, because order 105 has a `NULL` `product_id`. `x NOT IN (…, NULL)` can never be true, so every product is filtered out. That's verified output, and it's the most common follow-up Accenture interviewers ask.
- Order 108 references product 99, which doesn't exist. It doesn't affect the result because we start from `products`.
- Product 4 was ordered but cancelled. It counts as "sold" here. If the interviewer says only delivered orders count, move `AND o.status = 'DELIVERED'` into the `ON` clause (not the `WHERE`).

---

## Q4 — Total Age of People Older Than 30

**Source:** AmbitionBox, Data Engineer

**Question:** Return the number of people older than 30 and the sum of their ages. Then show the same numbers **per country**, including countries with nobody over 30.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS people_profile;
CREATE TABLE people_profile (
    person_id INT PRIMARY KEY,
    full_name VARCHAR(30) NOT NULL,
    country   VARCHAR(20) NOT NULL,
    age       INT NULL
);

INSERT INTO people_profile VALUES
( 1,'Rahul Das',    'India',     34),
( 2,'Meera Pillai', 'India',     29),
( 3,'Vivek Menon',  'India',     30),
( 4,'Sara Khan',    'India',     41),
( 5,'Tom Price',    'UK',        52),
( 6,'Lucy Hale',    'UK',        NULL),
( 7,'Jack Wood',    'UK',        31),
( 8,'Ana Lopez',    'Spain',     27),
( 9,'Pablo Ruiz',   'Spain',     30),
(10,'Nisha Rao',    'India',     38),
(11,'Chen Wei',     'Singapore', 45);
```

### ✅ Expected Output

**Overall:**

| people_over_30 | total_age |
|---|---|
| 6 | 241 |

**Per country:**

| country | people_over_30 | total_age_over_30 |
|---|---|---|
| India | 3 | 113 |
| Singapore | 1 | 45 |
| Spain | 0 | 0 |
| UK | 2 | 83 |

### 💡 Solution

**Overall:**

```sql
SELECT COUNT(*) AS people_over_30,
       SUM(age) AS total_age
FROM people_profile
WHERE age > 30;
```

**Per country:**

```sql
SELECT country,
       COUNT(CASE WHEN age > 30 THEN 1 END)        AS people_over_30,
       SUM(CASE WHEN age > 30 THEN age ELSE 0 END) AS total_age_over_30
FROM people_profile
GROUP BY country
ORDER BY country;
```

### 🧠 Explanation

- **"Older than 30" means `> 30`, not `>= 30`.** Vivek Menon and Pablo Ruiz are exactly 30 and are excluded.
- **Why the per-country query can't use `WHERE age > 30`:** Spain has nobody over 30, so a `WHERE` filter would remove all of Spain's rows and Spain would disappear from the output. Moving the condition inside `CASE` keeps every row in the group and only counts the ones that qualify.
- `COUNT(CASE WHEN age > 30 THEN 1 END)` works because `CASE` without an `ELSE` returns `NULL`, and `COUNT` ignores `NULL`s.
- `ELSE 0` in the `SUM` makes Spain show `0` instead of `NULL`. `COALESCE(SUM(CASE WHEN age > 30 THEN age END), 0)` gives the same result.
- Lucy Hale's `NULL` age fails `age > 30` (the comparison is unknown, not true), so she is never counted. No extra `NULL` handling is needed.

---

## Q5 — PIVOT and UNPIVOT

**Source:** AmbitionBox, SQL Developer

**Question:** Turn `quarterly_revenue` into one row per region with a column per quarter plus a total (**pivot**: rows to columns). Then turn that wide result back into one row per region per quarter (**unpivot**: columns to rows).

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS quarterly_revenue;
CREATE TABLE quarterly_revenue (
    row_id  INT PRIMARY KEY,
    region  VARCHAR(10) NOT NULL,
    quarter CHAR(2) NOT NULL,
    revenue INT NOT NULL
);

INSERT INTO quarterly_revenue VALUES
( 1,'North','Q1',120),( 2,'North','Q2',135),( 3,'North','Q3',150),( 4,'North','Q4',160),
( 5,'South','Q1', 90),( 6,'South','Q2', 60),( 7,'South','Q2', 45),( 8,'South','Q4',110),
( 9,'East', 'Q1', 70),(10,'East', 'Q2', 75),(11,'East', 'Q3', 80),(12,'East', 'Q4', 85),
(13,'West', 'Q3', 40);
```

### ✅ Expected Output

**Pivot:**

| region | q1 | q2 | q3 | q4 | total |
|---|---|---|---|---|---|
| North | 120 | 135 | 150 | 160 | 565 |
| East | 70 | 75 | 80 | 85 | 310 |
| South | 90 | 105 | NULL | 110 | 305 |
| West | NULL | NULL | 40 | NULL | 40 |

**Unpivot back:**

| region | quarter | revenue |
|---|---|---|
| East | Q1 | 70 |
| East | Q2 | 75 |
| East | Q3 | 80 |
| East | Q4 | 85 |
| North | Q1 | 120 |
| North | Q2 | 135 |
| North | Q3 | 150 |
| North | Q4 | 160 |
| South | Q1 | 90 |
| South | Q2 | 105 |
| South | Q4 | 110 |
| West | Q3 | 40 |

### 💡 Solution

**Pivot (rows → columns):**

```sql
SELECT region,
       SUM(CASE WHEN quarter = 'Q1' THEN revenue END) AS q1,
       SUM(CASE WHEN quarter = 'Q2' THEN revenue END) AS q2,
       SUM(CASE WHEN quarter = 'Q3' THEN revenue END) AS q3,
       SUM(CASE WHEN quarter = 'Q4' THEN revenue END) AS q4,
       SUM(revenue)                                   AS total
FROM quarterly_revenue
GROUP BY region
ORDER BY total DESC;
```

**Unpivot (columns → rows):**

```sql
WITH pivot_revenue AS (
    SELECT region,
           SUM(CASE WHEN quarter = 'Q1' THEN revenue END) AS q1,
           SUM(CASE WHEN quarter = 'Q2' THEN revenue END) AS q2,
           SUM(CASE WHEN quarter = 'Q3' THEN revenue END) AS q3,
           SUM(CASE WHEN quarter = 'Q4' THEN revenue END) AS q4
    FROM quarterly_revenue
    GROUP BY region
),
unpivoted AS (
    SELECT region, 'Q1' AS quarter, q1 AS revenue FROM pivot_revenue
    UNION ALL
    SELECT region, 'Q2', q2 FROM pivot_revenue
    UNION ALL
    SELECT region, 'Q3', q3 FROM pivot_revenue
    UNION ALL
    SELECT region, 'Q4', q4 FROM pivot_revenue
)
SELECT region, quarter, revenue
FROM unpivoted
WHERE revenue IS NOT NULL
ORDER BY region, quarter;
```

### 🧠 Explanation

- **MySQL has no `PIVOT` / `UNPIVOT` keywords** (SQL Server and Oracle do). The portable pattern is `SUM(CASE …)` with `GROUP BY` to pivot, and one `SELECT` per column joined with `UNION ALL` to unpivot.
- **Why `SUM` and not `MAX`:** South has two Q2 rows (60 and 45). `SUM` gives 105. `MAX` would give 60 and silently lose 45.
- **The `NULL`s are meaningful:** a `CASE` with no `ELSE` returns `NULL`, so West shows `NULL` for quarters with no revenue rather than a misleading `0`.
- **Unpivot needs `WHERE revenue IS NOT NULL`.** Without it, the `UNION ALL` produces 16 rows, including fake rows like `West, Q1, NULL` that never existed in the source. Filtering them out gives back exactly the real data.
- **A CTE only lives for one statement.** That's why the unpivot query defines `pivot_revenue` again in its own `WITH` instead of reusing it from the pivot query.
- `UNION ALL`, not `UNION`: we want every row, and `UNION` would pay for a de-duplication step it doesn't need.
- Unpivot doesn't fully reverse pivot: South's two Q2 rows come back as one row of 105. Pivoting aggregates, and aggregation can't be undone.

---

## Q6 — SQL Query Optimisation Techniques

**Source:** AmbitionBox, Data Engineer

**Question:** How would you speed up a slow SQL query?

### 📖 Answer

| # | Technique | Why it helps |
|---|---|---|
| 1 | **Index columns used in `WHERE`, `JOIN` and `ORDER BY`** | Lets MySQL jump to matching rows instead of scanning the whole table. |
| 2 | **Select only the columns you need**, not `SELECT *` | Less data read and sent over the network; can allow an index-only (covering) read. |
| 3 | **Filter early with `WHERE`** | Fewer rows reach the joins, `GROUP BY` and sorting steps. |
| 4 | **Check the execution plan with `EXPLAIN`** | Shows full table scans, unused indexes, temporary tables and filesorts. |
| 5 | **Use `EXISTS` / `NOT EXISTS` instead of `IN` / `NOT IN` when appropriate** | `EXISTS` stops at the first match, and `NOT EXISTS` is safe with `NULL`s (see Q3). |

### 💡 Example

```sql
-- 1. Look at the plan first
EXPLAIN
SELECT order_id, order_date
FROM orders
WHERE product_id = 8;

-- 2. If it shows a full scan ("Table scan on orders", or type = ALL in the
--    classic table format), add an index on the filter column
CREATE INDEX idx_orders_product_id ON orders (product_id);

-- 3. Run EXPLAIN again: it now shows
--    "Index lookup on orders using idx_orders_product_id" (type = ref)
```

### 🧠 Explanation

- **Avoid functions on indexed columns in `WHERE`.** `WHERE YEAR(order_date) = 2025` can't use an index on `order_date`; `WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01'` can.
- **Indexes aren't free.** Each one slows down `INSERT`, `UPDATE` and `DELETE`, and takes disk space. Index what you filter and join on, not everything.
- **Don't over-claim the `EXISTS` vs `IN` point.** Modern MySQL (8.0+) often rewrites `IN (subquery)` into the same semi-join as `EXISTS`. The real difference that always holds is `NOT IN` returning nothing when the subquery has a `NULL`.
- See [Part 13: Indexes](../docs/13-indexes.md) for composite indexes, prefix indexes and reading `EXPLAIN` output in detail.

---

## Q7 — Second Highest Salary in Each Department

**Source:** AmbitionBox, Data Engineering Analyst

**Question:** Find the employee(s) with the **second highest salary** in each department.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS dept_salaries;
CREATE TABLE dept_salaries (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(20) NOT NULL,
    department VARCHAR(20) NOT NULL,
    salary     INT NULL
);

INSERT INTO dept_salaries VALUES
( 1,'Arjun',  'Engineering',150000),
( 2,'Bela',   'Engineering',150000),
( 3,'Chirag', 'Engineering',120000),
( 4,'Dev',    'Engineering',120000),
( 5,'Esha',   'Engineering', 90000),
( 6,'Faiz',   'Finance',    110000),
( 7,'Gita',   'Finance',     95000),
( 8,'Hari',   'Finance',      NULL),
( 9,'Ira',    'HR',          70000),
(10,'Jai',    'Sales',       80000),
(11,'Kiran',  'Sales',       72000),
(12,'Lata',   'Sales',       60000);
```

### ✅ Expected Output

| emp_name | department | salary |
|---|---|---|
| Chirag | Engineering | 120000 |
| Dev | Engineering | 120000 |
| Gita | Finance | 95000 |
| Kiran | Sales | 72000 |

### 💡 Solution

```sql
WITH ranked_dept AS (
    SELECT emp_name, department, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS ranking
    FROM dept_salaries
    WHERE salary IS NOT NULL
)
SELECT emp_name, department, salary
FROM ranked_dept
WHERE ranking = 2
ORDER BY department, emp_name;
```

### 🧠 Explanation

- **"Second highest salary" means the second highest distinct value.** In Engineering, Arjun and Bela tie at 150000, so the second highest salary is 120000, earned by Chirag and Dev. `DENSE_RANK` gets this right.
- With `ROW_NUMBER`, Bela would be "second" (150000), which is wrong. With `RANK`, the 120000 earners get rank 3 and `ranking = 2` returns nothing for Engineering.
- **HR is missing on purpose.** Ira is the only person in HR, so there is no second highest salary.
- Hari's `NULL` salary is filtered out so it can't occupy a rank.
- Change `ranking = 2` to `ranking = N` for the Nth highest, or `ranking <= 3` for the top 3 per department.

---

## Q8 — Find and Remove Duplicate Rows By email

**Source:** AmbitionBox, Senior Software Engineer

**Question:** Find emails that appear more than once in `customer_signups`, then delete the duplicates while **keeping the earliest signup** (lowest `signup_id`) for each email.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS customer_signups;
CREATE TABLE customer_signups (
    signup_id   INT PRIMARY KEY,
    email       VARCHAR(40) NULL,
    full_name   VARCHAR(20) NOT NULL,
    signup_date DATE NOT NULL
);

INSERT INTO customer_signups VALUES
(1,'asha@mail.com',  'Asha',  '2025-01-05'),
(2,'ravi@mail.com',  'Ravi',  '2025-01-06'),
(3,'ASHA@mail.com ', 'Asha',  '2025-01-09'),
(4,'meena@mail.com', 'Meena', '2025-01-10'),
(5,'ravi@mail.com',  'Ravi',  '2025-01-06'),
(6,'ravi@mail.com',  'Ravi',  '2025-01-12'),
(7, NULL,            'Tarun', '2025-01-13'),
(8, NULL,            'Uma',   '2025-01-14'),
(9,'john@mail.com',  'John',  '2025-01-15');
```

### ✅ Expected Output

**Step 1: find duplicates**

| clean_email | cnt |
|---|---|
| asha@mail.com | 2 |
| ravi@mail.com | 3 |

**Step 2: table after the delete** (3 rows deleted)

| signup_id | email | full_name | signup_date |
|---|---|---|---|
| 1 | asha@mail.com | Asha | 2025-01-05 |
| 2 | ravi@mail.com | Ravi | 2025-01-06 |
| 4 | meena@mail.com | Meena | 2025-01-10 |
| 7 | NULL | Tarun | 2025-01-13 |
| 8 | NULL | Uma | 2025-01-14 |
| 9 | john@mail.com | John | 2025-01-15 |

### 💡 Solution

**Step 1: find duplicates**

```sql
SELECT LOWER(TRIM(email)) AS clean_email,
       COUNT(*)           AS cnt
FROM customer_signups
WHERE email IS NOT NULL
GROUP BY LOWER(TRIM(email))
HAVING COUNT(*) > 1
ORDER BY clean_email;
```

**Step 2: delete duplicates, keep the lowest `signup_id`**

```sql
DELETE c1
FROM customer_signups c1
JOIN customer_signups c2
  ON LOWER(TRIM(c1.email)) = LOWER(TRIM(c2.email))
 AND c1.signup_id > c2.signup_id;
```

### 🧠 Explanation

- **Clean the key before comparing it.** Row 3 is `'ASHA@mail.com '`, with capitals and a trailing space. Grouping on the raw `email` only finds `ravi@mail.com` (verified), because MySQL 8's default collation ignores case but **not** trailing spaces. `LOWER(TRIM(email))` catches the Asha duplicate too. Real signup data always has this kind of mess, so interviewers like to see you handle it.
- **How the `DELETE` works:** the self-join pairs each row with every row that has the same email and a **smaller** `signup_id`. Any row that finds such a partner is a later duplicate and gets deleted. The row with the lowest id has no smaller partner, so it survives.
- **`NULL` emails are left alone.** `NULL = NULL` is not true, so Tarun and Uma never pair up and neither is deleted. That's correct: two missing emails aren't evidence of a duplicate customer.
- **Always run Step 1 (or a `SELECT` version of Step 2) before the `DELETE`**, and do it inside a transaction on real data. A wrong join condition here deletes good rows with no undo.
- **Alternative with a window function:**

  ```sql
  DELETE FROM customer_signups
  WHERE signup_id IN (
      SELECT signup_id FROM (
          SELECT signup_id,
                 ROW_NUMBER() OVER (PARTITION BY LOWER(TRIM(email)) ORDER BY signup_id) AS rn
          FROM customer_signups
          WHERE email IS NOT NULL
      ) t
      WHERE rn > 1
  );
  ```

  The extra derived table `t` is required: MySQL won't let a `DELETE` read directly from the table it is deleting from in a subquery.

---

## Q9 — Row Counts for Each Join Type on val

**Source:** AmbitionBox, Data Engineering Analyst

**Question:** Two tables are joined on `val`. Both have duplicate values and `NULL`s. How many rows does each join type return? Write one query that shows all the counts.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS join_left_vals;
DROP TABLE IF EXISTS join_right_vals;
CREATE TABLE join_left_vals  (id INT PRIMARY KEY, val INT NULL);
CREATE TABLE join_right_vals (id INT PRIMARY KEY, val INT NULL);

INSERT INTO join_left_vals VALUES
(1,1),(2,1),(3,2),(4,3),(5,NULL);

INSERT INTO join_right_vals VALUES
(1,1),(2,1),(3,1),(4,2),(5,4),(6,NULL);
```

### ✅ Expected Output

| join_type | row_count |
|---|---|
| INNER | 7 |
| LEFT | 9 |
| RIGHT | 9 |
| FULL OUTER | 11 |
| CROSS | 30 |

### 💡 Solution

```sql
SELECT 'INNER' AS join_type, COUNT(*) AS row_count
FROM join_left_vals l
JOIN join_right_vals r ON l.val = r.val

UNION ALL

SELECT 'LEFT', COUNT(*)
FROM join_left_vals l
LEFT JOIN join_right_vals r ON l.val = r.val

UNION ALL

SELECT 'RIGHT', COUNT(*)
FROM join_left_vals l
RIGHT JOIN join_right_vals r ON l.val = r.val

UNION ALL

-- MySQL has no FULL OUTER JOIN: LEFT JOIN rows + right rows with no match
SELECT 'FULL OUTER',
       (SELECT COUNT(*) FROM join_left_vals l LEFT JOIN join_right_vals r ON l.val = r.val)
     + (SELECT COUNT(*) FROM join_right_vals r
        WHERE NOT EXISTS (SELECT 1 FROM join_left_vals l WHERE l.val = r.val))

UNION ALL

SELECT 'CROSS', COUNT(*)
FROM join_left_vals l
CROSS JOIN join_right_vals r;
```

### 🧠 Explanation

Work it out by hand first. That's what the interviewer is really testing.

| val | Left rows | Right rows | Matched pairs |
|---|---|---|---|
| 1 | 2 | 3 | 2 × 3 = **6** |
| 2 | 1 | 1 | 1 × 1 = **1** |
| 3 | 1 | 0 | 0 (left only) |
| 4 | 0 | 1 | 0 (right only) |
| NULL | 1 | 1 | 0 (`NULL = NULL` is not true) |

- **INNER = 7:** only matched pairs, 6 + 1. Duplicates multiply: each of the two left 1s joins to all three right 1s.
- **LEFT = 9:** 7 matched + 2 unmatched left rows (val 3 and the left `NULL`).
- **RIGHT = 9:** 7 matched + 2 unmatched right rows (val 4 and the right `NULL`).
- **FULL OUTER = 11:** 7 matched + 2 left-only + 2 right-only. MySQL doesn't support `FULL OUTER JOIN`, so it's built from a `LEFT JOIN` plus the right rows that have no match.
- **CROSS = 30:** every left row with every right row, 5 × 6. No join condition, so `NULL`s don't matter.
- **The `NULL` rows never match each other** in any join with `=`. They only appear as unmatched rows in outer joins.

---

## Q10 — Rolling 7-Day Average of Daily Orders

**Source:** AmbitionBox

**Question:** Calculate the **rolling 7-day average** of daily order amounts. Days with no orders must count as 0.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS daily_orders;
CREATE TABLE daily_orders (
    order_id    INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date  DATE NOT NULL,
    amount      INT NOT NULL
);

-- Multiple orders per day; no orders on 2025-03-05 and 2025-03-09 (gap days)
INSERT INTO daily_orders VALUES
(201, 1, '2025-03-01', 500),
(202, 2, '2025-03-01', 750),
(203, 3, '2025-03-01', 300),
(204, 1, '2025-03-02', 420),
(205, 4, '2025-03-03', 900),
(206, 2, '2025-03-03', 150),
(207, 5, '2025-03-04', 600),
(208, 3, '2025-03-04', 275),
(209, 1, '2025-03-04', 830),
(210, 6, '2025-03-04', 410),
(211, 2, '2025-03-06', 520),
(212, 4, '2025-03-06', 330),
(213, 5, '2025-03-07', 700),
(214, 6, '2025-03-08', 250),
(215, 1, '2025-03-08', 640),
(216, 3, '2025-03-08', 380),
(217, 2, '2025-03-10', 910),
(218, 5, '2025-03-10', 220),
(219, 4, '2025-03-11', 560),
(220, 6, '2025-03-12', 480),
(221, 1, '2025-03-12', 350),
(222, 3, '2025-03-12', 790);
```

### ✅ Expected Output

| order_date | total_amount | rolling_7d_avg |
|---|---|---|
| 2025-03-01 | 1550 | 1550.00 |
| 2025-03-02 | 420 | 985.00 |
| 2025-03-03 | 1050 | 1006.67 |
| 2025-03-04 | 2115 | 1283.75 |
| 2025-03-05 | 0 | 1027.00 |
| 2025-03-06 | 850 | 997.50 |
| 2025-03-07 | 700 | 955.00 |
| 2025-03-08 | 1270 | 915.00 |
| 2025-03-09 | 0 | 855.00 |
| 2025-03-10 | 1130 | 866.43 |
| 2025-03-11 | 560 | 644.29 |
| 2025-03-12 | 1620 | 875.71 |

### 💡 Solution

```sql
WITH RECURSIVE daily_sales AS (
    SELECT order_date, SUM(amount) AS total_amount
    FROM daily_orders
    GROUP BY order_date
),
calendar AS (
    SELECT MIN(order_date) AS order_date
    FROM daily_sales

    UNION ALL

    SELECT DATE_ADD(order_date, INTERVAL 1 DAY)
    FROM calendar
    WHERE order_date < (SELECT MAX(order_date) FROM daily_sales)
)
SELECT c.order_date,
       COALESCE(d.total_amount, 0) AS total_amount,
       ROUND(AVG(COALESCE(d.total_amount, 0)) OVER (
           ORDER BY c.order_date
           RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
       ), 2) AS rolling_7d_avg
FROM calendar c
LEFT JOIN daily_sales d
       ON c.order_date = d.order_date
ORDER BY c.order_date;
```

### 🧠 Explanation

- **Step 1, `daily_sales`:** collapse to one row per day. The table has several orders per day, and a window over raw rows would treat "previous 6 rows" as "previous 6 orders", not days.
- **Step 2, `calendar`:** a recursive CTE generates every date from the first to the last order date, including the gap days 03-05 and 03-09. `WITH RECURSIVE` must be written once at the start, even though only `calendar` is recursive.
- **Step 3:** `LEFT JOIN` the calendar to the daily totals, so gap days get `0` via `COALESCE`, then average over the current day plus the 6 before it.
- ⚠️ **Skip the calendar and the average is wrong.** Windowing only over days that have orders, 03-06's "7-day" average becomes **1197.00** instead of **997.50**, because the empty day 03-05 is missing and the window quietly reaches back an extra day. That's verified output.
- **`RANGE … INTERVAL 6 DAY` vs `ROWS 6 PRECEDING`:** once the calendar has no gaps, both give the same result here. `RANGE` is the safer choice because it measures in days, not rows, so it stays correct even if a date were ever missing.
- The first 6 days have partial windows (03-01 averages only itself). If a partial window is misleading for your report, filter out rows before the 7th day.
- For the full progression of rolling-window patterns, see [Bonus Set 1: The Rolling Sum Pattern](../bonus-problems/set-01-rolling-sum-pattern.md).

---

## Q11 — Difference Between WHERE VS HAVING

**Source:** AmbitionBox

**Question:** What is the difference between `WHERE` and `HAVING`? Show an example of each.

### 📖 Theory

| | `WHERE` | `HAVING` |
|---|---|---|
| Filters | Individual rows | Groups |
| Runs | **Before** `GROUP BY` | **After** `GROUP BY` |
| Aggregate functions (`COUNT`, `SUM`…) | ❌ Not allowed | ✅ Allowed |
| Works without `GROUP BY`? | Yes | Technically yes in MySQL, but it's meant for groups |

Logical order of execution: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(20) NOT NULL,
    department VARCHAR(20) NULL
);

INSERT INTO employees VALUES
( 1,'Aman',    'Engineering'),
( 2,'Bhavna',  'Engineering'),
( 3,'Chirag',  'Engineering'),
( 4,'Deepa',   'Engineering'),
( 5,'Ekta',    'Engineering'),
( 6,'Farhan',  'Engineering'),
( 7,'Gaurav',  'Engineering'),
( 8,'Harsha',  'Sales'),
( 9,'Isha',    'Sales'),
(10,'Jatin',   'Sales'),
(11,'Komal',   'Sales'),
(12,'Lokesh',  'Sales'),
(13,'Mansi',   'Support'),
(14,'Naveen',  'Support'),
(15,'Ojas',    'Support'),
(16,'Pooja',   'Support'),
(17,'Qadir',   'Support'),
(18,'Riya',    'Support'),
(19,'Sahil',   'HR'),
(20,'Tanvi',   'HR'),
(21,'Uday',    NULL),
(22,'Varsha',  NULL),
(23,'Yash',    NULL),
(24,'Zoya',    NULL),
(25,'Arnav',   NULL),
(26,'Bina',    NULL);
```

### ✅ Expected Output

**`WHERE`: employees in Sales**

| emp_id | emp_name | department |
|---|---|---|
| 8 | Harsha | Sales |
| 9 | Isha | Sales |
| 10 | Jatin | Sales |
| 11 | Komal | Sales |
| 12 | Lokesh | Sales |

**`HAVING`: departments with fewer than 3 employees**

| department | total_count |
|---|---|
| HR | 2 |

### 💡 Solution

```sql
-- WHERE filters rows
SELECT emp_id, emp_name, department
FROM employees
WHERE department = 'Sales';

-- HAVING filters groups
SELECT department, COUNT(*) AS total_count
FROM employees
WHERE department IS NOT NULL
GROUP BY department
HAVING COUNT(*) < 3;
```

### 🧠 Explanation

- `WHERE COUNT(*) < 3` is an error: when `WHERE` runs, the groups don't exist yet, so there is nothing to count.
- The second query uses **both**: `WHERE` first removes the rows with no department, then `HAVING` filters the groups that are left.
- **Rule of thumb:** if a condition can go in `WHERE`, put it there. Filtering rows before grouping is cheaper than grouping everything and filtering afterwards (see Q6).

---

## Q12 — Departments With More Than 5 Employees

**Source:** AmbitionBox

**Question:** Write a query to find the departments that have **more than 5 employees**.

### 🗄️ Dataset

Uses the same `employees` table as [Q11](#q11--difference-between-where-vs-having).

### ✅ Expected Output

| department | total_count |
|---|---|
| Engineering | 7 |
| Support | 6 |

### 💡 Solution

```sql
SELECT department, COUNT(*) AS total_count
FROM employees
WHERE department IS NOT NULL
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY total_count DESC;
```

### 🧠 Explanation

- `GROUP BY department` makes one group per department, `COUNT(*)` counts the rows in each, and `HAVING COUNT(*) > 5` keeps only the large ones.
- **"More than 5" is `> 5`.** Sales has exactly 5 employees and is correctly excluded.
- ⚠️ **Why `WHERE department IS NOT NULL` matters:** six employees have no department. `GROUP BY` puts all `NULL`s into one group, so without the filter a row `NULL | 6` appears in the result (verified). "Unassigned" isn't a department.

---

## Q13 — Difference Between DELETE, TRUNCATE AND DROP

**Source:** AmbitionBox

**Question:** What is the difference between `DELETE`, `TRUNCATE` and `DROP`?

### 📖 Theory

| | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Selected rows (`WHERE` allowed) or all rows | All rows | The whole table: structure, data, indexes |
| `WHERE` clause | ✅ | ❌ | ❌ |
| Table structure kept? | ✅ | ✅ | ❌ |
| Speed on large tables | Slow (row by row, logged) | Fast (recreates the table) | Fast |
| Resets `AUTO_INCREMENT` | ❌ | ✅ | n/a (table is gone) |
| Fires `DELETE` triggers | ✅ | ❌ | ❌ |
| Rollback in a transaction (MySQL) | ✅ | ❌ (implicit commit) | ❌ (implicit commit) |

### 🗄️ Dataset

A throwaway table, so the demo doesn't destroy the `employees` table used in Q11 and Q12:

```sql
DROP TABLE IF EXISTS employees_demo;
CREATE TABLE employees_demo (
    emp_id     INT AUTO_INCREMENT PRIMARY KEY,
    emp_name   VARCHAR(20) NOT NULL,
    department VARCHAR(20) NULL
);

INSERT INTO employees_demo (emp_name, department) VALUES
('Aman','Engineering'),('Harsha','Sales'),('Mansi','Support');
```

### 💡 Solution

```sql
-- DELETE: remove selected rows
DELETE FROM employees_demo WHERE department = 'Sales';

-- DELETE with no WHERE removes every row, but AUTO_INCREMENT keeps counting
DELETE FROM employees_demo;
INSERT INTO employees_demo (emp_name, department) VALUES ('Riya','Support');
SELECT * FROM employees_demo;   -- emp_id = 4

-- TRUNCATE: removes every row and resets AUTO_INCREMENT
TRUNCATE TABLE employees_demo;
INSERT INTO employees_demo (emp_name, department) VALUES ('Riya','Support');
SELECT * FROM employees_demo;   -- emp_id = 1

-- DROP: the table itself is gone
DROP TABLE employees_demo;
SHOW FULL TABLES;               -- employees_demo is no longer listed
```

### ✅ Expected Output

After `DELETE FROM employees_demo;` and a new insert:

| emp_id | emp_name | department |
|---|---|---|
| 4 | Riya | Support |

After `TRUNCATE TABLE employees_demo;` and a new insert:

| emp_id | emp_name | department |
|---|---|---|
| 1 | Riya | Support |

### 🧠 Explanation

- The two outputs above are the clearest way to show the difference in an interview: same insert, but `DELETE` continues from id 4 while `TRUNCATE` starts again at 1.
- **Pick by intent:** `DELETE` to remove some rows, `TRUNCATE` to empty a table and start fresh (staging tables, test data), `DROP` when the table shouldn't exist at all.
- In MySQL, `TRUNCATE` and `DROP` cause an implicit commit, so a `ROLLBACK` can't bring the data back. Treat both as permanent.
- `SHOW FULL TABLES` lists tables and views with a `Table_type` column, a quick way to confirm a `DROP` worked.

---

## Q14 — Difference Between INNER JOIN VS LEFT JOIN

**Source:** AmbitionBox

**Question:** What is the difference between `INNER JOIN` and `LEFT JOIN`? Show both on departments and employees.

### 📖 Theory

- **`INNER JOIN`:** returns only rows that have a match in **both** tables.
- **`LEFT JOIN`:** returns **all** rows from the left table; where there is no match, the right table's columns are `NULL`.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS emp_master;
DROP TABLE IF EXISTS dept_master;
CREATE TABLE dept_master (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(20) NOT NULL
);
CREATE TABLE emp_master (
    emp_id   INT PRIMARY KEY,
    emp_name VARCHAR(20) NOT NULL,
    dept_id  INT NULL
);

INSERT INTO dept_master VALUES
(10,'Engineering'),
(20,'Finance'),
(30,'HR'),
(40,'Marketing');

INSERT INTO emp_master VALUES
(1,'Anil',   10),
(2,'Bhumi',  10),
(3,'Charu',  20),
(4,'Dinesh', 30),
(5,'Esha',   NULL),
(6,'Firoz',  99);
```

### ✅ Expected Output

**`INNER JOIN`:**

| dept_id | dept_name | emp_id | emp_name |
|---|---|---|---|
| 10 | Engineering | 1 | Anil |
| 10 | Engineering | 2 | Bhumi |
| 20 | Finance | 3 | Charu |
| 30 | HR | 4 | Dinesh |

**`LEFT JOIN`:**

| dept_id | dept_name | emp_id | emp_name |
|---|---|---|---|
| 10 | Engineering | 1 | Anil |
| 10 | Engineering | 2 | Bhumi |
| 20 | Finance | 3 | Charu |
| 30 | HR | 4 | Dinesh |
| 40 | Marketing | NULL | NULL |

### 💡 Solution

```sql
-- INNER JOIN: only departments that have employees
SELECT d.dept_id, d.dept_name, e.emp_id, e.emp_name
FROM dept_master d
JOIN emp_master e
  ON d.dept_id = e.dept_id
ORDER BY d.dept_id, e.emp_id;

-- LEFT JOIN: every department, with or without employees
SELECT d.dept_id, d.dept_name, e.emp_id, e.emp_name
FROM dept_master d
LEFT JOIN emp_master e
       ON d.dept_id = e.dept_id
ORDER BY d.dept_id, e.emp_id;
```

### 🧠 Explanation

- **Marketing** has no employees. `INNER JOIN` drops it; `LEFT JOIN` keeps it with `NULL` employee columns.
- **Esha** (`dept_id NULL`) and **Firoz** (`dept_id 99`, a department that doesn't exist) appear in **neither** result, because `dept_master` is the left table. To see employees without a valid department, flip it: `FROM emp_master e LEFT JOIN dept_master d`.
- Name the columns instead of `d.*, m.*`. Both tables have a `dept_id`, and `SELECT *` would return it twice.
- ⚠️ A filter on the right table in `WHERE` (e.g. `WHERE e.emp_name LIKE 'A%'`) removes the `NULL` rows and quietly turns a `LEFT JOIN` back into an `INNER JOIN`. Put such filters in the `ON` clause to keep the unmatched rows.

---

## Q15 — The Big Spenders

**Source:** AmbitionBox

**Question:** List the name and city of customers whose **total order amount** is greater than the **average total order amount across all customers**.

### 🗄️ Dataset

```sql
DROP TABLE IF EXISTS customer_orders;
DROP TABLE IF EXISTS customer;
CREATE TABLE customer (
    customer_id INT PRIMARY KEY,
    name        VARCHAR(20) NOT NULL,
    city        VARCHAR(20) NOT NULL
);
CREATE TABLE customer_orders (
    order_id    INT PRIMARY KEY,
    customer_id INT NOT NULL,
    amount      INT NOT NULL
);

INSERT INTO customer VALUES
(1,'Rohit',  'Delhi'),
(2,'Simran', 'Mumbai'),
(3,'Aditya', 'Pune');

INSERT INTO customer_orders VALUES
(101, 1, 2500),
(102, 1, 1200),
(103, 2, 4000),
(104, 3,  900),
(105, 3,  600);
```

### ✅ Expected Output

Average of per-customer totals = (3700 + 4000 + 1500) / 3 = **3066.67**

| name | city |
|---|---|
| Rohit | Delhi |
| Simran | Mumbai |

### 💡 Solution

```sql
SELECT c.name, c.city
FROM customer c
JOIN customer_orders o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.city
HAVING SUM(o.amount) > (
    SELECT AVG(total_amount)
    FROM (
        SELECT customer_id, SUM(amount) AS total_amount
        FROM customer_orders
        GROUP BY customer_id
    ) t
)
ORDER BY c.customer_id;
```

### 🧠 Explanation

- **Two levels of aggregation:** first `SUM` per customer (3700, 4000, 1500), then `AVG` of those totals (3066.67). You can't write `AVG(SUM(amount))` directly in MySQL, so the inner derived table `t` does the first level.
- **`HAVING`, not `WHERE`:** the condition compares `SUM(o.amount)`, an aggregate, so it can only be checked after `GROUP BY`.
- ⚠️ **Don't compare against `AVG(amount)` from the orders table.** That's the average *order* (1840), not the average *customer total*. On this data it happens to return the same two customers, which is exactly why the bug survives testing. Add one customer with many small orders and the answers diverge. Always check which average the question asks for.
- `c.customer_id` is in the `GROUP BY` so two customers with the same name and city are never merged into one.

---

<div align="center">

**If this helped you, please ⭐ the repo and [subscribe on YouTube](https://www.youtube.com/watch?v=DwFu-cqVm7E). It genuinely helps! 🙌**

</div>
