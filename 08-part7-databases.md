# Part 7: Databases

---

## Chapter 54: SQL & Relational Database Theory

### Introduction

Structured Query Language (SQL) is the universal language for managing relational databases. Developed in the 1970s at IBM by Donald D. Chamberlin and Raymond F. Boyce, SQL was based on Codd's relational model. It became an ANSI standard in 1986 and ISO standard in 1987. SQL has evolved through multiple versions including SQL-89, SQL-92, SQL:1999 (added recursive CTEs, triggers), SQL:2003 (window functions, XML), SQL:2008 (TRUNCATE, INSTEAD OF triggers), SQL:2011 (temporal data), and SQL:2016 (JSON, row pattern matching).

### Why It Matters

SQL is the lingua franca of data. Every full-stack developer writes SQL daily — for application backends, analytics, reporting, data science, and system administration. Understanding SQL deeply separates junior from senior engineers.

### Real World Analogy

Think of SQL as the librarian's query system. The **database** is the library, **tables** are bookshelves, **rows** are individual books, **columns** are book properties (title, author, ISBN), **indexes** are the card catalog, and **queries** are the request slips you fill out to find books.

### Theory: Codd's 12 Rules

| # | Rule | Principle |
|---|------|-----------|
| 1 | Information | All data stored as values in tables (rows & columns) |
| 2 | Guaranteed Access | Every value accessible by table + PK + column |
| 3 | NULL Handling | NULLs represent missing/inapplicable data uniformly |
| 4 | Active Catalog | Database dictionary is stored as tables |
| 5 | Sub-language | At least one language with DDL, DML, DCL, TCL |
| 6 | View Updating | Views support insert/update/delete where possible |
| 7 | Set Operations | INSERT, UPDATE, DELETE operate on sets |
| 8 | Physical Independence | Storage changes don't affect applications |
| 9 | Logical Independence | Schema changes don't affect views/applications |
| 10 | Integrity Independence | Constraints stored in catalog, not applications |
| 11 | Distribution Independence | Works across distributed systems |
| 12 | Non-subversion | Low-level access cannot bypass integrity rules |

### SQL Dialects

| Feature | MySQL | PostgreSQL | SQLite | SQL Server | Oracle |
|---------|-------|-----------|--------|------------|--------|
| Auto-increment | AUTO_INCREMENT | SERIAL / IDENTITY | AUTOINCREMENT | IDENTITY | SEQUENCE |
| LIMIT | LIMIT x OFFSET y | LIMIT x OFFSET y | LIMIT x OFFSET y | OFFSET x FETCH y | OFFSET x FETCH y |
| String concat | CONCAT() / || | || | || | + | || |
| Full outer join | Supported | Supported | Not supported | Supported | Supported |
| JSON type | JSON (5.7+) | JSON/JSONB | JSON1 ext | JSON | JSON |
| UPSERT | INSERT ... ON DUPLICATE KEY UPDATE | INSERT ... ON CONFLICT DO UPDATE | INSERT OR REPLACE | MERGE | MERGE |
| Arrays | Not native | Native ARRAY | Not native | Not native | Nested tables |
| CTEs recursive | Yes (8.0+) | Yes | Yes | Yes | Yes |
| Window functions | Yes (8.0+) | Yes | Yes (3.25+) | Yes | Yes |
| License | GPL/Commercial | PostgreSQL License | Public Domain | Commercial | Commercial |

### DDL (Data Definition Language)

`sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  full_name VARCHAR(150) NOT NULL,
  age INT CHECK (age >= 0 AND age <= 150),
  role ENUM('admin', 'editor', 'viewer') DEFAULT 'viewer',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users MODIFY COLUMN email VARCHAR(320) NOT NULL;
ALTER TABLE users DROP COLUMN phone;

DROP TABLE IF EXISTS users;
TRUNCATE TABLE users;
`

> ⚠️ **Warning:** DROP removes the table AND data permanently. TRUNCATE removes all rows but preserves the table structure. Neither can be rolled back in MySQL (implicit commit).

### DML (Data Manipulation Language)

`sql
INSERT INTO users (email, full_name, age, role) VALUES ('alice@example.com', 'Alice Chen', 28, 'admin');
INSERT INTO users (email, full_name, age) VALUES ('bob@example.com', 'Bob Smith', 35), ('carol@example.com', 'Carol Davis', 42);
SELECT id, email, full_name FROM users WHERE age > 30 ORDER BY full_name;
UPDATE users SET role = 'editor' WHERE age >= 40;
DELETE FROM users WHERE age IS NULL;
`

### DCL (Data Control Language)

`sql
GRANT SELECT, INSERT ON myapp.* TO 'app_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON myapp.* TO 'admin'@'%' WITH GRANT OPTION;
REVOKE DELETE ON myapp.* FROM 'app_user'@'localhost';
FLUSH PRIVILEGES;
`

### TCL (Transaction Control Language)

`sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

BEGIN;
  INSERT INTO orders (user_id, total) VALUES (42, 150.00);
  SAVEPOINT order_created;
  INSERT INTO order_items (order_id, product_id, qty) VALUES (LAST_INSERT_ID(), 5, 2);
  ROLLBACK TO order_created;
COMMIT;
`

### Data Types

| Category | Types | Use Case |
|----------|-------|----------|
| Numeric | INT, BIGINT, SMALLINT, TINYINT, DECIMAL(p,s), FLOAT, DOUBLE | IDs, prices, coordinates |
| String | CHAR(n), VARCHAR(n), TEXT, MEDIUMTEXT, LONGTEXT | Names, descriptions, articles |
| Binary | BINARY, VARBINARY, BLOB, MEDIUMBLOB, LONGBLOB | Files, encrypted data |
| Date/Time | DATE, TIME, DATETIME, TIMESTAMP, YEAR | Birthdays, logs |
| Special | BOOLEAN, ENUM, SET, JSON, UUID | Flags, categories, documents |
| Geometry | POINT, LINESTRING, POLYGON, GEOMETRY | Maps, geofences |

### Constraints

| Constraint | Purpose | Example |
|-----------|---------|---------|
| PRIMARY KEY | Uniquely identifies each row | id INT PRIMARY KEY |
| FOREIGN KEY | Enforces referential integrity | FOREIGN KEY (user_id) REFERENCES users(id) |
| UNIQUE | Ensures all values are distinct | email VARCHAR(255) UNIQUE |
| NOT NULL | Column cannot contain NULL | name VARCHAR(100) NOT NULL |
| CHECK | Validates values against condition | CHECK (age >= 0) |
| DEFAULT | Provides default value | created_at TIMESTAMP DEFAULT NOW() |
| INDEX | Speeds up queries | INDEX idx_email (email) |
| ENUM | Restricts to predefined values | role ENUM('admin','user') |

### Relationships

**One-to-One:**
`sql
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE profiles (
  id INT PRIMARY KEY, user_id INT UNIQUE, bio TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
`

**One-to-Many:**
`sql
CREATE TABLE customers (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE orders (
  id INT PRIMARY KEY, customer_id INT NOT NULL, total DECIMAL(10,2),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);
`

**Many-to-Many (junction table):**
`sql
CREATE TABLE students (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE courses (id INT PRIMARY KEY, title VARCHAR(100));
CREATE TABLE enrollments (
  student_id INT, course_id INT, enrolled_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES students(id),
  FOREIGN KEY (course_id) REFERENCES courses(id)
);
`

### JOINs

`sql
CREATE TABLE departments (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE employees (
  id INT PRIMARY KEY, name VARCHAR(100), dept_id INT, manager_id INT,
  FOREIGN KEY (dept_id) REFERENCES departments(id)
);

INSERT INTO departments VALUES (1, 'Engineering'), (2, 'Sales'), (3, NULL);
INSERT INTO employees VALUES (1, 'Alice', 1, NULL), (2, 'Bob', 1, 1),
  (3, 'Carol', 2, 1), (4, 'Dave', NULL, 2);

-- INNER JOIN: only matching rows
SELECT e.name, d.name FROM employees e INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN: all employees, matching dept if exists
SELECT e.name, d.name FROM employees e LEFT JOIN departments d ON e.dept_id = d.id;

-- RIGHT JOIN: all departments, matching employees
SELECT e.name, d.name FROM employees e RIGHT JOIN departments d ON e.dept_id = d.id;

-- FULL OUTER JOIN: all rows from both tables
SELECT e.name, d.name FROM employees e FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- CROSS JOIN: Cartesian product
SELECT e.name, d.name FROM employees e CROSS JOIN departments d;

-- SELF JOIN: table joined with itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1 LEFT JOIN employees e2 ON e1.manager_id = e2.id;
`

### Subqueries

`sql
-- Non-correlated: runs once
SELECT name, salary FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated: runs per outer row
SELECT e1.name, e1.salary FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id);

-- EXISTS (efficient)
SELECT d.name FROM departments d WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- IN, ANY, ALL
SELECT name FROM employees WHERE dept_id IN (1, 2);
SELECT name FROM employees WHERE salary > ANY (SELECT salary FROM employees WHERE dept_id = 2);
SELECT name FROM employees WHERE salary > ALL (SELECT salary FROM employees WHERE dept_id = 2);
`

### Set Operations

`sql
SELECT city FROM customers UNION SELECT city FROM suppliers ORDER BY city;
SELECT city FROM customers UNION ALL SELECT city FROM suppliers;
SELECT city FROM customers INTERSECT SELECT city FROM suppliers;
SELECT city FROM customers EXCEPT SELECT city FROM suppliers;
`

### Aggregate Functions

`sql
SELECT
  COUNT(*) AS total_employees, COUNT(DISTINCT dept_id) AS dept_count,
  SUM(salary) AS payroll_total, AVG(salary) AS avg_salary,
  MIN(salary) AS min_salary, MAX(salary) AS max_salary
FROM employees;

SELECT dept_id, AVG(salary) AS avg_dept_salary
FROM employees GROUP BY dept_id HAVING AVG(salary) > 50000;
`

### SQL Execution Order

FROM → JOIN → WHERE → GROUP BY → HAVING → WINDOW → SELECT → DISTINCT → UNION → ORDER BY → LIMIT/OFFSET

> Why can't you use WHERE to filter aggregated results? WHERE runs before GROUP BY — use HAVING.

### Common Table Expressions (CTEs)

`sql
-- Simple CTE
WITH sales_summary AS (
  SELECT user_id, SUM(amount) AS total_spent FROM orders GROUP BY user_id
)
SELECT u.name, s.total_spent FROM users u JOIN sales_summary s ON u.id = s.user_id WHERE s.total_spent > 1000;

-- Recursive CTE (hierarchy)
WITH RECURSIVE org_chart AS (
  SELECT id, name, manager_id, 1 AS level FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.id, e.name, e.manager_id, oc.level + 1
  FROM employees e JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level;
`

### Window Functions

`sql
SELECT
  name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank,
  RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rank_with_gaps,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_no_gaps,
  NTILE(4)     OVER (ORDER BY salary DESC) AS salary_quartile,
  LAG(salary)  OVER (PARTITION BY department ORDER BY salary) AS prev_salary,
  LEAD(salary) OVER (PARTITION BY department ORDER BY salary) AS next_salary,
  FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS top_dept_salary,
  LAST_VALUE(salary)  OVER (PARTITION BY department ORDER BY salary DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS bottom_dept_salary
FROM employees;
`

### CASE, COALESCE, NULLIF

`sql
SELECT
  name, salary,
  CASE WHEN salary < 30000 THEN 'Entry' WHEN salary < 60000 THEN 'Mid'
       WHEN salary < 100000 THEN 'Senior' ELSE 'Executive' END AS level,
  COALESCE(phone, email, 'No Contact') AS primary_contact,
  NULLIF(age, 0) AS age_or_null
FROM employees;
`

### Common Mistakes

| Mistake | Why It Hurts | Fix |
|---------|-------------|-----|
| SELECT * in production | Extra I/O, can't use covering indexes | Specify columns explicitly |
| Not using JOINs properly | Cartesian products | Use explicit JOIN syntax |
| No indexes on FK columns | Full table scans on JOINs | Index all foreign key columns |
| Using WHERE on aggregated columns | Wrong logic | Use HAVING for group filters |
| Forgetting NULL comparisons | WHERE col = NULL returns nothing | Use IS NULL |
| Wrong execution order understanding | Unexpected results | Read: FROM → WHERE → GROUP → HAVING → SELECT → ORDER |

### Best Practices

- Always use explicit JOIN syntax (not implicit comma-joins)
- Use EXISTS instead of IN for subqueries with large result sets
- Prefer UNION ALL over UNION when duplicates are acceptable
- Use VARCHAR instead of TEXT when length is known
- Use DECIMAL for monetary values, not FLOAT
- Always specify column list in INSERT statements
- Write CTEs for complex queries to improve readability
- Analyze tables regularly for query planner statistics

### Interview Questions

1. **Difference between HAVING and WHERE?** WHERE filters rows before aggregation; HAVING filters groups after aggregation.
2. **INNER JOIN vs LEFT JOIN?** INNER JOIN returns only matching rows; LEFT JOIN returns all left rows with NULLs for non-matches.
3. **Correlated vs non-correlated subqueries?** Non-correlated runs once; correlated runs once per outer row.
4. **When use recursive CTE?** For hierarchical data: org charts, tree traversal, date series generation.
5. **Window functions vs GROUP BY?** Window functions compute across row sets without collapsing; GROUP BY collapses groups.

### Practical Exercises

1. Find employees earning more than their department average (correlated subquery).
2. Recursive CTE to generate every day in 2024.
3. Running total of sales per month using window functions.
4. Find duplicate email addresses in a users table.
5. Cross-tabulation of orders by month and product category (CASE + PIVOT).

### Mini Project: Employee Directory

1. Create employees(id, name, manager_id, department, salary, hire_date), seed 20+ employees.
2. Org chart with levels using recursive CTE.
3. Department salary stats (avg, median, max, min).
4. Top 3 earners per department (window function).
5. New hires in last 30 days.
6. Manager chain breadcrumb trail for any employee.

### Revision Notes

- SQL is declarative — specify WHAT, not HOW
- Execution order: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
- JOINs combine tables; subqueries nest queries; CTEs name subqueries
- Window functions enable analytics without collapsing rows
- COALESCE handles NULLs safely; CASE provides if-then-else
- Index FK columns, avoid SELECT *, use EXISTS over IN for large sets

---

## Chapter 55: MySQL

### Introduction

MySQL is the world's most popular open-source relational database. Created by Michael Widenius in 1995, acquired by Sun Microsystems (2008) then Oracle (2010), it powers Facebook, Twitter, YouTube, and WordPress. It's the "M" in the LAMP stack with over 10 million installations.

### Why It Matters

MySQL dominates web hosting and mid-market applications. Its ease of use, strong community, and excellent replication make it the default choice for countless applications. If you work at a startup or mid-size company, you're likely using MySQL or MariaDB.

### Real World Analogy

MySQL is a well-organized filing cabinet where each drawer (database) contains folders (tables) with categorized documents (rows). Labels (indexes) enable fast retrieval, and the clerk (storage engine) manages how files are stored.

### MySQL Architecture

┌─────────────────────────────────────┐
│ Clients (Connection Manager)        │
├─────────────────────────────────────┤
│ Query Cache (deprecated 8.0)       │
├─────────────────────────────────────┤
│ SQL Interface → Parser → Optimizer │
├─────────────────────────────────────┤
│ Execution Engine                    │
├─────────────────────────────────────┤
│ Storage Engine Abstraction          │
│  InnoDB  MyISAM  MEMORY  CSV  ...  │
└─────────────────────────────────────┘

### Storage Engines

| Engine | Transactions | Row Locking | FK | Full-text | Use Case |
|--------|-------------|-------------|----|-----------|----------|
| InnoDB | ✅ ACID | Row-level | ✅ | ✅ (5.6+) | Default for most workloads |
| MyISAM | ❌ | Table-level | ❌ | ✅ | Legacy, read-only archives |
| MEMORY | ❌ | Table-level | ❌ | ❌ | Temp tables, caching |
| CSV | ❌ | None | ❌ | ❌ | CSV data interchange |
| ARCHIVE | ❌ | None | ❌ | ❌ | Log storage, compression |
| NDB Cluster | ✅ | Row-level | ✅ | ❌ | High-availability, distributed |

### InnoDB Architecture

Key Components:
- **Buffer Pool**: Caches data & indexes in memory (70-80% of RAM)
- **Change Buffer**: Caches secondary index changes
- **Adaptive Hash Index**: Auto-built hash indexes for hot pages
- **Redo Log**: Ensures durability (ib_logfile0, ib_logfile1)
- **Undo Log**: Supports MVCC and rollbacks (stored in ibdata)
- **Doublewrite Buffer**: Prevents partial page writes
- **Log Buffer**: Writes redo logs before flush to disk

> The doublewrite buffer prevents torn pages. When MySQL writes a 16KB page, power failure could leave only part written. The doublewrite buffer writes a copy first, then the actual page, enabling recovery.

### Installation & Configuration

`ash
# Ubuntu/Debian
sudo apt update && sudo apt install mysql-server -y
sudo mysql_secure_installation

# macOS (Homebrew)
brew install mysql && brew services start mysql

# Windows — MSI installer from dev.mysql.com/downloads/installer/
`

**Key my.cnf settings:**
`ini
[mysqld]
port = 3306
datadir = /var/lib/mysql

# InnoDB
innodb_buffer_pool_size = 4G          # 70-80% of available RAM
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 1    # 1 = Full ACID, 2 = faster
innodb_file_per_table = 1
innodb_flush_method = O_DIRECT

# Connection
max_connections = 500
thread_cache_size = 32

# Temp
tmp_table_size = 256M
max_heap_table_size = 256M

# Slow query log
slow_query_log = 1
long_query_time = 2
slow_query_log_file = /var/log/mysql/slow.log

# Binary log
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
expire_logs_days = 7

# Character set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
`

### MySQL CLI

`ash
mysql -u root -p

# Common commands
SHOW DATABASES;
USE mydb;
SHOW TABLES;
DESCRIBE users;
SHOW CREATE TABLE users\G
SHOW PROCESSLIST;
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
`

### MySQL Data Types Deep Dive

| Type | Storage (bytes) | Range |
|------|----------------|-------|
| TINYINT | 1 | -128 to 127 |
| SMALLINT | 2 | -32,768 to 32,767 |
| MEDIUMINT | 3 | -8M to 8M |
| INT | 4 | -2B to 2B |
| BIGINT | 8 | -9.2E18 to 9.2E18 |
| FLOAT(p) | 4 (p≤24) / 8 | Approximate |
| DOUBLE | 8 | Approximate, 15-16 digits |
| DECIMAL(m,d) | Varies | Exact, m up to 65 |
| VARCHAR(n) | n+1 or n+2 | Up to 65,535 chars |
| CHAR(n) | n (fixed) | Up to 255 |
| TEXT | L+2 bytes | Up to 65,535 chars |
| MEDIUMTEXT | L+3 bytes | Up to 16.7M chars |
| LONGTEXT | L+4 bytes | Up to 4GB chars |
| DATE | 3 | 1000-01-01 to 9999-12-31 |
| DATETIME | 5-8 | Date + time |
| TIMESTAMP | 4 | 1970 to 2038-01-19 |
| JSON | As stored | Validates JSON automatically |
| ENUM | 1-2 bytes | Up to 65,535 values |

> ⚠️ **Warning:** TIMESTAMP range only goes through 2038 (Year 2038 problem). Use DATETIME for dates beyond 2038.

### Indexes

`sql
-- B-Tree (default)
CREATE INDEX idx_email ON users(email);

-- Composite B-Tree
CREATE INDEX idx_name_age ON users(last_name, first_name, age);

-- UNIQUE
CREATE UNIQUE INDEX idx_email_unique ON users(email);

-- FULLTEXT (InnoDB 5.6+)
CREATE FULLTEXT INDEX idx_content ON articles(title, body);

-- SPATIAL (InnoDB 5.7+)
CREATE SPATIAL INDEX idx_location ON places(coordinates);

-- DESCENDING (8.0+)
CREATE INDEX idx_created_desc ON orders(created_at DESC);

-- Invisible (8.0+) — test without removing
ALTER TABLE users ALTER INDEX idx_email INVISIBLE;
ALTER TABLE users ALTER INDEX idx_email VISIBLE;
`

**Cardinality:** Measure of uniqueness. Higher = more selective = better index.
`sql
SHOW INDEX FROM users;  -- Cardinality column
`

### EXPLAIN Plan Analysis

| Column | Meaning | Values (worst → best) |
|--------|---------|----------------------|
| type | Access method | ALL > index > range > ref > eq_ref > const |
| key | Index used | Name or NULL |
| rows | Rows examined | Lower is better |
| Extra | Additional info | Using where; Using index; Using temporary; Using filesort |

**Type values:**
- **ALL** — Full table scan (worst, avoid)
- **index** — Full index scan
- **range** — Index range scan
- **ref** — Non-unique index lookup
- **eq_ref** — Unique index lookup in JOIN
- **const** — PK lookup (WHERE id = 42)

`sql
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE email = 'alice@example.com';
`

### Performance Tuning

`ini
# Most important: buffer pool size
innodb_buffer_pool_size = 70-80% of RAM
innodb_log_file_size = 25% of buffer pool
innodb_flush_log_at_trx_commit = 1    # =1 ACID, =2 faster

# Per-connection buffers (keep small — multiplied by connections)
tmp_table_size = 256M
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 128K
`

### Partitioning

`sql
-- RANGE partition
CREATE TABLE orders (
  id BIGINT NOT NULL, user_id INT, total DECIMAL(10,2), created_at DATE NOT NULL
) PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p_2022 VALUES LESS THAN (2023),
  PARTITION p_2023 VALUES LESS THAN (2024),
  PARTITION p_2024 VALUES LESS THAN (2025),
  PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- LIST partition
CREATE TABLE stores (id INT NOT NULL, region ENUM('East','West'))
  PARTITION BY LIST COLUMNS(region) (
    PARTITION p_east VALUES IN ('East'), PARTITION p_west VALUES IN ('West')
  );

-- HASH partition
CREATE TABLE logs (id INT NOT NULL, created_at DATETIME)
  PARTITION BY HASH (id) PARTITIONS 8;
`

### Replication

`ini
# Primary my.cnf
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
gtid_mode = ON
enforce_gtid_consistency = ON
`

`sql
-- On primary
CREATE USER 'replica'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
SHOW MASTER STATUS;

-- On replica
CHANGE MASTER TO MASTER_HOST='192.168.1.10', MASTER_USER='replica',
  MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=123456789;
START SLAVE;
SHOW SLAVE STATUS\G
`

| Type | Guarantee | Performance | Use Case |
|------|-----------|-------------|----------|
| Async | None | Fastest | Analytics replicas |
| Semi-sync | ≥1 replica acked | Slightly slower | Production reads |
| Group (MGR) | Consensus (Paxos) | Slower | HA, multi-primary |

### Backup

`ash
# Logical backup
mysqldump -u root -p --single-transaction --quick mydb > mydb.sql
mysqldump -u root -p --all-databases --routines --events > all.sql

# Physical backup (XtraBackup)
xtrabackup --backup --target-dir=/backups/mysql/ --user=root --password=root
xtrabackup --prepare --target-dir=/backups/mysql/

# Point-in-Time Recovery
mysqlbinlog /var/log/mysql/mysql-bin.000023 --stop-datetime="2024-03-15 14:30:00" | mysql -u root -p
`

### Security

`sql
CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'strong_password!';
GRANT SELECT, INSERT, UPDATE ON myapp.* TO 'app_user'@'192.168.1.%';
ALTER USER 'app_user'@'%' REQUIRE SSL;
SHOW GRANTS FOR 'app_user'@'%';
DROP USER 'old_user'@'localhost';
`

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Using MyISAM | No transactions, table locks, crash data loss | Use InnoDB |
| SELECT * in production | Extra I/O, covering index bypassed | Specify columns |
| Not using utf8mb4 | Can't store emoji/CJK characters | Use utf8mb4 (not utf8) |
| Default buffer_pool_size | 128MB is terrible for any workload | Set to 70-80% of RAM |
| No EXPLAIN | Full table scans in production | Always explain queries |
| No FK indexes | Full table scans on JOINs | Index all FK columns |
| Oversized buffers | Memory exhaustion | Keep per-connection buffers small |

### Best Practices

- Use InnoDB with innodb_file_per_table=1 always
- Set charset utf8mb4, collation utf8mb4_unicode_ci
- EXPLAIN all queries before deploying
- Enable slow query log (long_query_time=2)
- Use pt-query-digest (Percona Toolkit) for analysis
- Enable GTID for simpler replication management
- Use ROW-based binary logging
- Monitor Threads_connected, Innodb_rows_read
- Test schema changes on a replica first
- Use pt-online-schema-change for large table alterations

### Interview Questions

1. **MyISAM vs InnoDB?** InnoDB supports transactions, row-level locking, FKs, crash recovery. MyISAM only table-level locking, no transactions.
2. **How does InnoDB MVCC work?** Stores rollback segment with undo logs. Each transaction gets a snapshot ID. Reading reconstructs rows from undo log as they existed at snapshot time.
3. **Leftmost prefix rule?** Composite index (a,b,c) used for WHERE on (a), (a,b), (a,b,c) but NOT (b) or (c) alone.
4. **Migrate large table without downtime?** Use pt-online-schema-change or gh-ost — create shadow table, sync via triggers/binlog, swap atomically.
5. **What causes deadlocks in InnoDB?** Two transactions hold locks the other needs. InnoDB detects immediately, rolls back the transaction with least progress. Fix by accessing tables in consistent order.

### Practical Exercises

1. Install MySQL 8.0, create a database, run mysql_secure_installation.
2. Set innodb_buffer_pool_size=4G, verify with SHOW VARIABLES.
3. Create a table with all index types (B-Tree, FULLTEXT, SPATIAL, composite, unique).
4. Generate 1M rows, benchmark SELECT with and without index.
5. Set up primary-replica replication via Docker.

### Mini Project: Library Management System

1. Tables: books(id, isbn, title, author, published_year, genre), members(id, name, email, join_date), loans(id, book_id, member_id, loan_date, return_date).
2. Add indexes: ISBN lookup, member email, loan dates.
3. Write queries: currently loaned books, overdue books (7+ days), most borrowed books, member history.
4. Export with mysqldump.
5. Use EXPLAIN to optimize the overdue books query.

### Revision Notes

- InnoDB is the default and only recommended engine
- Buffer pool size is the single most important performance setting
- Composite indexes follow leftmost prefix rule
- Query cache is removed in MySQL 8.0/8.3 — don't rely on it
- ALTER TABLE on large tables can lock for hours — use online DDL tools
- utf8mb4 is true UTF-8; utf8 is limited to 3 bytes (no emoji)
- Percona Toolkit (pt-*) is essential for MySQL operations

---

## Chapter 56: PostgreSQL

### Introduction

PostgreSQL is the world's most advanced open-source relational database. Created at UC Berkeley in 1986 as a successor to Ingres, it has over 35 years of active development. Known for standards compliance, extensibility, and advanced features: custom types, full-text search, PostGIS, and JSONB.

### Why It Matters

PostgreSQL is the database of choice for modern development. Rails, Django, and many ORMs default to it. It handles OLTP, OLAP, geospatial, JSON, and text search equally well. Its feature set rivals Oracle at zero cost.

### Real World Analogy

If MySQL is a filing cabinet, PostgreSQL is a Swiss Army knife. It's not just for storage — it can search text (full-text search), handle geography (PostGIS), store JSON like a document database, and act as a queue (SKIP LOCKED).

### Architecture

**Processes:**
- **Postmaster**: Main daemon, listens for connections, forks children
- **Backend process**: One per client connection
- **WAL writer**: Flushes WAL to disk
- **Checkpointer**: Writes dirty buffers to data files
- **Background writer**: Preemptively writes dirty buffers
- **Autovacuum launcher/workers**: Reclaims dead tuples
- **Stats collector**: Collects table/index access statistics

**Shared Memory:** shared_buffers, WAL buffers, lock space, clog, etc.

### Installation

`ash
# Ubuntu/Debian
sudo apt update && sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql

# macOS
brew install postgresql@16 && brew services start postgresql@16

# Windows — installer from postgresql.org/download/windows/

psql -U postgres -c "SELECT version();"
`

### psql CLI

`ash
# Connect
psql -h localhost -U postgres -d mydb
psql postgresql://user:password@localhost:5432/mydb

# Meta-commands
\l               # List databases
\c mydb          # Connect
\dt              # List tables
\d+ users        # Describe table
\di              # List indexes
\df              # List functions
\x               # Toggle expanded display
\e               # Editor for multi-line query
\q               # Quit

# Run SQL script
psql -U postgres -d mydb -f schema.sql
`

### Data Types

`sql
-- Standard
INT, BIGINT, SMALLINT, SERIAL, BIGSERIAL
DECIMAL(p,s), NUMERIC(p,s), REAL, DOUBLE PRECISION
VARCHAR(n), CHAR(n), TEXT
BOOLEAN, DATE, TIME, TIMESTAMP, TIMESTAMPTZ, INTERVAL
BYTEA

-- PostgreSQL-specific
UUID                             -- Universally unique identifiers
JSON / JSONB                     -- Binary JSON (faster, indexable)
HSTORE                           -- Key-value store
ARRAY                            -- Multi-dimensional arrays
CIDR / INET / MACADDR            -- Network addresses
TSVECTOR / TSQUERY               -- Full-text search
XML                              -- XML data
RANGE types (int4range, tsrange, daterange, numrange)
POINT, LINE, LSEG, BOX, PATH, POLYGON, CIRCLE  -- Geometric
INTERVAL                         -- Time duration
`

### JSONB Deep Dive

`sql
CREATE TABLE events (
  id BIGSERIAL PRIMARY KEY, payload JSONB NOT NULL, created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO events (payload) VALUES
  ('{"user": "Alice", "action": "login", "ip": "192.168.1.1", "tags": ["admin", "beta"]}'),
  ('{"user": "Bob", "action": "purchase", "amount": 49.99, "items": [{"sku": "ABC", "qty": 2}]}');

-- Query operators
SELECT * FROM events WHERE payload @> '{"action": "login"}';
SELECT * FROM events WHERE payload -> 'user' = '"Alice"';   -- JSON string
SELECT * FROM events WHERE payload ->> 'user' = 'Alice';    -- plain text
SELECT payload #> '{items, 0, sku}' FROM events;            -- JSON path
SELECT payload #>> '{items, 0, sku}' FROM events;           -- text path

-- GIN index
CREATE INDEX idx_events_payload ON events USING GIN (payload);
CREATE INDEX idx_events_payload_path ON events USING GIN (payload jsonb_path_ops);
`

> The difference between -> and ->>: -> returns JSON (with quotes for strings), ->> returns text (no quotes). @> checks if left JSONB contains right structure. jsonb_path_ops creates smaller, faster GIN for @> but doesn't support ?, ?|, ?&.

### Advanced Indexing

`sql
-- B-Tree (default)
CREATE INDEX idx_users_email ON users(email);

-- Hash (equality only, smaller than B-tree)
CREATE INDEX idx_users_status ON users USING HASH (status);

-- GiST — geometry, full-text, ranges
CREATE INDEX idx_locations_coords ON locations USING GiST (coords);

-- GIN — JSONB, arrays, full-text
CREATE INDEX idx_articles_fts ON articles USING GIN (search_vector);

-- SP-GiST — point clustering, quad-trees
CREATE INDEX idx_buildings_address ON buildings USING SP-GiST (address);

-- BRIN — very large sequential data (time-series)
CREATE INDEX idx_logs_created ON logs USING BRIN (created_at) WITH (pages_per_range = 32);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Covering index (11+)
CREATE INDEX idx_users_covering ON users(email) INCLUDE (name, role);

-- Concurrent (no blocking)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
`

> BRIN indexes are extremely space-efficient for time-series. A BRIN on created_at can be 100x smaller than B-tree while performing well for range queries.

### Full-Text Search

`sql
-- Stored generated tsvector
ALTER TABLE articles ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED;

CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);

-- Search with ranking
SELECT title, ts_rank(search_vector, query) AS rank
FROM articles, plainto_tsquery('english', 'database optimization tips') query
WHERE search_vector @@ query ORDER BY rank DESC LIMIT 10;

-- Highlight matches
SELECT title, ts_headline('english', body, query, 'MaxWords=50, MinWords=20')
FROM articles, to_tsquery('english', 'database & optimization') query
WHERE search_vector @@ query;
`

### Recursive CTEs

`sql
-- Org chart with path
WITH RECURSIVE org_tree AS (
  SELECT id, name, manager_id, 1 AS depth, ARRAY[id] AS path
  FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.id, e.name, e.manager_id, ot.depth + 1, ot.path || e.id
  FROM employees e JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY path;

-- Graph traversal (shortest path, BFS)
WITH RECURSIVE paths AS (
  SELECT node1, node2, ARRAY[node1, node2] AS path, 1 AS depth FROM graph WHERE node1 = 'A'
  UNION ALL
  SELECT p.node1, e.node2, p.path || e.node2, p.depth + 1
  FROM paths p JOIN graph e ON p.node2 = e.node1 WHERE NOT e.node2 = ANY(p.path)
)
SELECT * FROM paths WHERE node2 = 'G' ORDER BY depth LIMIT 1;
`

### Window Functions (Advanced)

`sql
SELECT
  department, name, salary,
  SUM(salary) OVER (ORDER BY salary ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS moving_sum_3,
  AVG(salary)  OVER (ORDER BY hire_date RANGE BETWEEN INTERVAL '30' DAY PRECEDING AND CURRENT ROW) AS rolling_30d,
  salary - LAG(salary) OVER (PARTITION BY department ORDER BY salary) AS diff_from_prev
FROM employees;
`

**Frame specs:** ROWS BETWEEN (physical), RANGE BETWEEN (logical by value), GROUPS BETWEEN (peer groups).

### Advanced SQL

`sql
-- LATERAL (for-each-row subquery)
SELECT u.name, recent_orders.total, recent_orders.order_date
FROM users u
CROSS JOIN LATERAL (
  SELECT total, order_date FROM orders WHERE user_id = u.id ORDER BY order_date DESC LIMIT 3
) recent_orders;

-- GROUPING SETS, CUBE, ROLLUP
SELECT department, role, SUM(salary) FROM employees
GROUP BY GROUPING SETS ((department, role), (department), (role), ());
SELECT department, role, SUM(salary) FROM employees GROUP BY ROLLUP (department, role);
SELECT department, role, SUM(salary) FROM employees GROUP BY CUBE (department, role);

-- FILTER clause (conditional aggregation)
SELECT department,
  COUNT(*) AS total,
  COUNT(*) FILTER (WHERE salary > 50000) AS above_50k,
  AVG(salary) FILTER (WHERE role = 'Senior') AS avg_senior_salary
FROM employees GROUP BY department;
`

### Stored Procedures, Functions & Triggers

`sql
-- Function returning table
CREATE OR REPLACE FUNCTION get_user_orders(p_user_id INT)
RETURNS TABLE(order_id BIGINT, total DECIMAL, order_date TIMESTAMPTZ)
LANGUAGE plpgsql AS 
BEGIN
  RETURN QUERY SELECT o.id, o.total, o.created_at
  FROM orders o WHERE o.user_id = p_user_id ORDER BY o.created_at DESC;
END;
;

-- Trigger function for audit logging
CREATE OR REPLACE FUNCTION audit_order_changes()
RETURNS TRIGGER LANGUAGE plpgsql AS 
BEGIN
  INSERT INTO audit_log (table_name, action, old_data, new_data, changed_by)
  VALUES ('orders', TG_OP, row_to_json(OLD), row_to_json(NEW), current_user);
  RETURN NEW;
END;
;

CREATE TRIGGER trg_orders_audit
  AFTER INSERT OR UPDATE OR DELETE ON orders
  FOR EACH ROW EXECUTE FUNCTION audit_order_changes();

-- DO block (anonymous)
DO  DECLARE v_count INT; BEGIN
  SELECT COUNT(*) INTO v_count FROM users WHERE active = true;
  RAISE NOTICE 'Active users: %', v_count;
END; ;
`

### Views & Materialized Views

`sql
-- Simple view
CREATE VIEW active_users AS SELECT id, name, email FROM users WHERE active = true;

-- Materialized view
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT DATE_TRUNC('month', order_date) AS month, SUM(total) AS revenue, COUNT(*) AS orders_count
FROM orders WHERE status = 'completed' GROUP BY 1 ORDER BY 1;

-- Non-blocking refresh (needs unique index)
CREATE UNIQUE INDEX idx_monthly_sales_month ON monthly_sales(month);
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales;
`

### Transactions & Two-Phase Commit

`sql
-- Savepoint
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  SAVEPOINT after_debit;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  ROLLBACK TO SAVEPOINT after_debit;
COMMIT;

-- Two-phase commit
PREPARE TRANSACTION 'txn_12345';
COMMIT PREPARED 'txn_12345';
-- or ROLLBACK PREPARED 'txn_12345';
`

### MVCC Internals

`sql
-- System columns: xmin (creator XID), xmax (deleter XID), ctid (location)
SELECT xmin, xmax, ctid, * FROM users;

-- VACUUM reclaims dead tuples
VACUUM ANALYZE users;

-- Monitor dead tuples
SELECT relname, n_dead_tup, n_live_tup, last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'users';

-- Autovacuum config
-- autovacuum_vacuum_threshold = 50
-- autovacuum_vacuum_scale_factor = 0.2
`

> **Interview Question:** What is transaction wraparound? PostgreSQL XIDs are 32-bit (~4B transactions). After that, XID wraps. Vacuum-freeze marks rows as frozen (special XID always visible) to prevent this. If not done, the DB shuts down to protect integrity.

### Performance Tuning

| Parameter | Recommendation | Purpose |
|-----------|---------------|---------|
| shared_buffers | 25% of RAM | In-memory data cache |
| effective_cache_size | 75% of RAM | Planner's OS cache estimate |
| work_mem | 4-64MB | Per-operation sort/hash memory |
| maintenance_work_mem | 10% of RAM (max 1GB) | VACUUM, CREATE INDEX |
| wal_buffers | 16-64MB | WAL write buffer |
| random_page_cost | 1.1 (SSD) / 4.0 (HDD) | Cost of random I/O |
| effective_io_concurrency | 200 (SSD) / 2 (HDD) | Parallel I/O |
| checkpoint_timeout | 15-30 min | Checkpoint frequency |
| max_wal_size | 2-4GB | WAL retention |

### EXPLAIN ANALYZE

`sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM users WHERE email = 'alice@example.com';
`

**Scan types:** Seq Scan (worst), Index Scan, Index Only Scan (best), Bitmap Scan.
**Join strategies:** Nested Loop (small sets), Hash Join (large unsorted), Merge Join (pre-sorted).

### Extensions

`sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;  -- Query tracking
CREATE EXTENSION IF NOT EXISTS auto_explain;         -- Auto-log slow queries
CREATE EXTENSION IF NOT EXISTS postgis;               -- Geographic objects
CREATE EXTENSION IF NOT EXISTS uuid-ossp;             -- UUID generation
CREATE EXTENSION IF NOT EXISTS pgcrypto;              -- Cryptography
CREATE EXTENSION IF NOT EXISTS pg_trgm;               -- Fuzzy string matching
CREATE EXTENSION IF NOT EXISTS hstore;                -- Key-value store
CREATE EXTENSION IF NOT EXISTS citext;                -- Case-insensitive text
`

**pg_stat_statements:**
`sql
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements WHERE query NOT LIKE '%pg_%' ORDER BY total_exec_time DESC LIMIT 10;
`

### Streaming Replication

`ini
# Primary
wal_level = replica
max_wal_senders = 5
wal_keep_size = 1024

# Replica
primary_conninfo = 'host=192.168.1.10 port=5432 user=replicator'
primary_slot_name = 'replica1'
hot_standby = on
`

`ash
pg_basebackup -h 192.168.1.10 -D /var/lib/postgresql/16/main -U replicator -P -X stream
SELECT * FROM pg_create_physical_replication_slot('replica1');
SELECT * FROM pg_stat_replication;
`

### Declarative Partitioning

`sql
CREATE TABLE measurements (
  id BIGSERIAL, measured_at TIMESTAMPTZ NOT NULL, sensor_id INT, value DOUBLE PRECISION
) PARTITION BY RANGE (measured_at);

CREATE TABLE measurements_2024_q1 PARTITION OF measurements
  FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
CREATE TABLE measurements_2024_q2 PARTITION OF measurements
  FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');
`

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| No ANALYZE | Poor query plans | Run ANALYZE or configure autovacuum |
| Default work_mem | Disk sorts | Increase to 16-64MB |
| No VACUUM | Bloat, wraparound | Enable autovacuum |
| Over-indexing | Slow writes | Index only WHERE/JOIN columns |
| Using SERIAL for PK | Lock contention | Use BIGSERIAL or UUID |
| random_page_cost=4 on SSD | Planner prefers seq scans | Set to 1.1 |
| Ignoring pg_stat_statements | Blind optimization | Monitor query metrics |

### Best Practices

- random_page_cost = 1.1 on SSD
- BIGINT or UUID for PK (not SERIAL on high-write)
- Enable pg_stat_statements everywhere
- Configure auto_explain (log_min_duration=1000ms)
- Create indexes CONCURRENTLY to avoid locks
- Partition tables over 100GB or 100M rows
- Use materialized views for expensive aggregations
- Use JSONB over JSON (faster, indexable)
- Use connection pooling (PgBouncer, Pgpool-II)

### Interview Questions

1. **MVCC in Postgres vs MySQL?** Postgres stores row versions in main table + VACUUM cleans up. InnoDB stores versions in rollback segment (undo tablespace).
2. **VACUUM vs VACUUM FULL?** VACUUM reclaims space for reuse but not to OS. VACUUM FULL rewrites table, returns space to OS, needs ACCESS EXCLUSIVE lock.
3. **GiST vs GIN?** GiST for multidimensional data (geometry, nearest-neighbor). GIN for composite values (arrays, JSONB, FTS). GIN is slower to build but faster to read.
4. **Postgres FTS vs Elasticsearch?** Postgres FTS handles basic search with stemming, ranking, dictionaries. Elasticsearch excels at distributed search, real-time indexing, relevance tuning.
5. **What's a replication slot?** Ensures primary retains WAL until all replicas received them. Without one, a disconnected replica might need full resync.

### Practical Exercises

1. Install PostgreSQL, configure shared_buffers, work_mem, random_page_cost.
2. Create JSONB column with 10K records, query with @>, ->, ->>.
3. Set up full-text search with tsvector and GIN index.
4. Recursive CTE to generate Fibonacci sequence.
5. Function to transfer money between accounts with error handling.
6. Set up pg_stat_statements, identify top 3 slowest queries.

### Mini Project: E-Commerce Analytics

1. Tables: products, customers, orders, order_items, reviews.
2. Materialized views: daily_sales (date, product_id, revenue, units) and customer_lifetime_value.
3. Set up MV refresh (cron or pg_cron).
4. Report queries: top products, customer segments, revenue trends.
5. Use EXPLAIN ANALYZE to compare query performance with and without MVs.
6. Covering indexes for common queries.

### Revision Notes

- PostgreSQL uses processes (not threads) — one per connection
- shared_buffers ~25% of RAM (not 70-80% like MySQL)
- MVCC + VACUUM is central to design
- JSONB + GIN = document DB inside Postgres
- GiST, GIN, SP-GiST, BRIN are specialized index types
- EXPLAIN ANALYZE BUFFERS is the performance tool
- Streaming replication is built-in
- Extensions make Postgres infinitely extensible

---

## Chapter 57: MongoDB (NoSQL)

### Introduction

MongoDB is the most popular NoSQL document database. Created by Dwight Merriman and Eliot Horowitz at 10gen (now MongoDB Inc.) in 2007. It stores data in flexible, JSON-like documents (BSON format) rather than rigid tables. MongoDB 4.0+ added multi-document ACID transactions.

### Why It Matters

MongoDB excels at rapid development, flexible schemas, and horizontal scaling. It is the default for apps with evolving data models, high write throughput, or simple read patterns. It powers everything from real-time analytics to content management to IoT.

### Real World Analogy

If SQL is a filing cabinet, MongoDB is a warehouse where each shelf (collection) holds varied items (documents) that do not need the same structure. Each document describes itself, like boxes with different labels.

### NoSQL Overview

| Type | Database | Data Model | Use Case |
|------|----------|------------|----------|
| Document | MongoDB, Couchbase, Firestore | JSON/BSON | General purpose, flexible schema |
| Key-Value | Redis, DynamoDB, Riak | Key to Value | Caching, sessions |
| Wide-Column | Cassandra, HBase, ScyllaDB | Column families | Time-series, analytics |
| Graph | Neo4j, ArangoDB, Dgraph | Nodes + Edges | Social networks, recommendations |
| Search | Elasticsearch, Solr | Inverted index | Text search, logs |
| Time-Series | InfluxDB, TimescaleDB | Timestamp + value | Monitoring, metrics |

### BSON Document Model

```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Alice Chen",
  email: "alice@example.com",
  age: 28,
  tags: ["admin", "premium"],
  address: { street: "123 Main St", city: "San Francisco" },
  createdAt: ISODate("2024-03-15T10:30:00Z"),
  balance: NumberDecimal("1500.00"),
  isActive: true
}
```

> Document size limit is 16MB. For larger data use GridFS (2MB chunks). Max nesting depth is 100 levels.

### CRUD Operations

```javascript
const { MongoClient } = require('mongodb');
const client = new MongoClient('mongodb://localhost:27017');
await client.connect();
const db = client.db('shop');
const users = db.collection('users');

// CREATE
await users.insertOne({ name: 'Alice Chen', email: 'alice@example.com', age: 28, tags: ['admin'] });
await users.insertMany([
  { name: 'Bob', email: 'bob@example.com', age: 35 },
  { name: 'Carol', email: 'carol@example.com', age: 42 }
]);

// READ
const user = await users.findOne({ email: 'alice@example.com' });
const adults = await users.find({ age: { $gte: 18 } }).sort({ age: -1 }).limit(10).toArray();

// UPDATE
await users.updateOne(
  { email: 'alice@example.com' },
  { $set: { age: 29 }, $push: { tags: 'vip' } }
);
await users.updateMany({ age: { $gte: 65 } }, { $set: { seniorDiscount: true } });

// REPLACE (entire document replacement)
await users.replaceOne(
  { email: 'alice@example.com' },
  { name: 'Alice Chen', email: 'alice+new@example.com', age: 29 }
);

// DELETE
await users.deleteOne({ email: 'bob@example.com' });
await users.deleteMany({ active: false });
```

### Query Operators

```javascript
// Comparison
{ age: { $eq: 25 } }    { age: { $ne: 25 } }    { age: { $gt: 18 } }
{ age: { $gte: 18 } }   { age: { $lt: 65 } }    { age: { $lte: 65 } }
{ status: { $in: ['A','B'] } }   { status: { $nin: ['C','D'] } }

// String and Existence
{ name: { $regex: /^alice/i } }
{ middleName: { $exists: false } }
{ age: { $type: 'int' } }

// Expression and Logical
{ $expr: { $gt: ['$spent', '$limit'] } }
{ $and: [{ age: { $gte: 18 } }, { status: 'active' }] }
{ $or:  [{ role: 'admin' }, { role: 'moderator' }] }

// Array
{ tags: { $all: ['admin', 'premium'] } }
{ ratings: { $elemMatch: { $gte: 4, $lte: 5 } } }
{ tags: { $size: 3 } }
```

### Update Operators

```javascript
// Fields
{ $set: { age: 29 } }           // Set values
{ $unset: { tempField: '' } }   // Remove field
{ $inc: { loginCount: 1 } }     // Increment
{ $mul: { price: 1.1 } }        // Multiply

// Arrays
{ $push: { tags: 'vip' } }                           // Add to end
{ $push: { scores: { $each: [85,90], $sort: -1 } } } // Push with modifiers
{ $pop: { messages: -1 } }                           // Remove first/last
{ $pull: { tags: 'temp' } }                          // Remove matching
{ $addToSet: { tags: 'new' } }                       // Add if not exists
```

### Aggregation Pipeline

```javascript
const pipeline = [
  { $match: { createdAt: { $gte: new Date('2024-01-01'), $lt: new Date('2025-01-01') } } },
  { $lookup: { from: 'users', localField: 'userId', foreignField: '_id', as: 'user' } },
  { $unwind: '$user' },
  { $addFields: { month: { $month: '$createdAt' }, year: { $year: '$createdAt' } } },
  { $group: {
      _id: { year: '$year', month: '$month' },
      totalRevenue: { $sum: '$total' }, orderCount: { $sum: 1 },
      avgOrderValue: { $avg: '$total' }, customers: { $addToSet: '$userId' }
  } },
  { $sort: { '_id.year': 1, '_id.month': 1 } },
  { $project: {
      _id: 0,
      period: { $concat: [{ $toString: '$_id.year' }, '-', { $toString: '$_id.month' }] },
      totalRevenue: { $round: ['$totalRevenue', 2] }, orderCount: 1,
      avgOrderValue: { $round: ['$avgOrderValue', 2] },
      uniqueCustomers: { $size: '$customers' }
  } }
];
const results = await orders.aggregate(pipeline).toArray();
```

**Key stages:** $match (filter early), $project (reshape), $group (aggregate), $sort, $lookup (join), $unwind (deconstruct arrays), $addFields (computed), $bucket, $facet (multi-pipeline), $out/$merge (write results).

### explain()

```javascript
const exp = await db.collection('users').find(
  { email: 'alice@example.com' }
).explain('executionStats');

console.log({
  stage: exp.queryPlanner.winningPlan.stage,
  nReturned: exp.executionStats.nReturned,
  totalDocsExamined: exp.executionStats.totalDocsExamined,
  executionTimeMillis: exp.executionStats.executionTimeMillis
});
```

**Stages (worst to best):** COLLSCAN to IXSCAN to FETCH to SHARD_MERGE to SORT to IDHACK to PROJECTION_COVERED.

### Indexes

```javascript
// Single, compound, unique
await users.createIndex({ email: 1 }, { unique: true });
await users.createIndex({ status: 1, age: -1, name: 1 });

// Multikey (auto on arrays)
await users.createIndex({ tags: 1 });

// Text index with weights
await articles.createIndex({ title: 'text', body: 'text' }, { weights: { title: 10, body: 5 } });

// Geospatial
await places.createIndex({ location: '2dsphere' });

// Hashed (for sharding)
await users.createIndex({ userId: 'hashed' });

// TTL (auto-expire)
await sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Sparse (only docs with field)
await users.createIndex({ optionalField: 1 }, { sparse: true });

// Partial filter
await users.createIndex({ age: 1 }, { partialFilterExpression: { age: { $gte: 18 } } });

// Hidden (test impact without removing)
await users.createIndex({ email: 1 }, { hidden: true });
```

### Schema Design Patterns

**Embedded (one-to-one / one-to-few):**
```javascript
{ _id: ObjectId("..."), name: "Alice", address: { street: "123 Main St", city: "SF" } }
```

**Referenced (one-to-many / many-to-many):**
```javascript
// User: { _id: ObjectId("..."), name: "Alice" }
// Order: { _id: ObjectId("..."), userId: ObjectId("..."), total: 150.00 }
```

**Bucket Pattern (time-series):**
```javascript
{
  sensorId: "temp-1", date: ISODate("2024-03-15"),
  readings: [{ time: ISODate("2024-03-15T10:00:00Z"), temp: 22.5 }],
  count: 1440, avgTemp: 22.6
}
```

> Model data based on query patterns, not relationships. If you read user + recent orders together, embed orders. If orders are queried independently, reference.

### Replication: Replica Sets

```javascript
// rs.initiate()
{ _id: "rs0", members: [
  { _id: 0, host: "mongo1:27017", priority: 2 },
  { _id: 1, host: "mongo2:27017", priority: 1 },
  { _id: 2, host: "mongo3:27017", arbiterOnly: true }
]}
```

**Read Preferences:** primary (default), primaryPreferred, secondary, secondaryPreferred, nearest.
**Write Concerns:** w=1 (default), w=majority, w=3, j=true (journaled).

### Sharding

```javascript
sh.enableSharding("shop");
sh.shardCollection("shop.orders", { userId: "hashed" });
sh.shardCollection("shop.orders", { createdAt: 1 }); // Range based
```

**Shard key criteria:** High cardinality, low frequency, even distribution, supports common queries.

> Choosing a bad shard key is the most common scaling mistake. Monotonically increasing keys with range sharding cause hot spots. Use hashed for write-heavy workloads.

### Multi-Document ACID Transactions

```javascript
const session = client.startSession();
try {
  await session.withTransaction(async () => {
    await accounts.updateOne({ userId: 'alice' }, { $inc: { balance: -100 } }, { session });
    await accounts.updateOne({ userId: 'bob' }, { $inc: { balance: 100 } }, { session });
  });
} finally {
  await session.endSession();
}
```

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| No indexes on query fields | COLLSCAN on every request | Create matching indexes |
| Oversized docs (>100KB) | Wasted cache, slow queries | Normalize or bucket pattern |
| Deep nesting (>5 levels) | Complex queries, slow updates | Flatten or reference |
| No replica set | Data loss on crash | Deploy replica set |
| Frequent $lookup | Slow aggregation | Embed or denormalize |
| Not setting w:majority | Rollbacks on failover | Use w:majority for critical data |
| Bad shard key | Hot spots, uneven distribution | Choose carefully, hash keys |

### Best Practices

- Always use replica sets (even in development)
- Index query patterns, not all fields
- Use $match early in aggregation pipelines
- Keep documents small and flat
- Use allowDiskUse(true) for large aggregations
- Set maxTimeMS on queries
- Use WiredTiger storage engine
- Enable TLS/SSL in production

### Interview Questions

1. **Consistency in replica sets?** Primary handles all writes, replicated via oplog. Secondaries replay oplog. Primary failure triggers voting for new primary.
2. **Embedding vs referencing?** Embedding = faster reads, atomic updates, no joins. Referencing = smaller docs, avoids 16MB limit, more flexible.
3. **WiredTiger concurrency?** Document-level concurrency with MVCC. Readers dont block writers. Snapshots for consistent reads.
4. **Aggregation pipeline vs MapReduce?** Pipeline is optimized C++ operations that can use indexes. MapReduce is deprecated as of 5.0.
5. **Choose a shard key?** High cardinality, low frequency, even distribution, aligned with common queries. Avoid monotonically increasing keys.

### Practical Exercises

1. Start local MongoDB with replica set (mongod --replSet rs0).
2. Model blog: users, posts, comments (embed comments, reference users).
3. Aggregation pipeline: top 10 users by order value this month.
4. Create 2dsphere index, run near queries.
5. Compare query using explain('executionStats') with/without index.
6. TTL index for expiring session data.

### Mini Project: Real-Time Analytics Dashboard

1. Collections: events (raw clickstream), sessions (user sessions), pageviews.
2. Bucket pattern: aggregate pageviews into hourly buckets.
3. Indexes: compound on (timestamp, eventType), TTL on raw events (30 days).
4. Aggregation: daily unique visitors, top referral sources, conversion funnel.
5. Use $facet for multiple metrics (totalVisitors, bounceRate, avgSessionDuration).
6. $merge results into daily_reports collection.

### Revision Notes

- Schema-less: documents in a collection can have different fields
- BSON is binary JSON with extra types (ObjectId, ISODate, NumberDecimal)
- 16MB document size limit, 100-level nesting
- _id is required and unique (auto ObjectId or custom)
- Aggregation pipeline (not MapReduce) for data processing
- Replica sets for HA with automatic failover
- Sharding for horizontal scaling
- Multi-document ACID transactions since 4.0
- WiredTiger: document-level locking, MVCC
- Shard keys are permanent: choose very carefully
- Embed for contains relationships, reference for belongs-to

---

## Chapter 58: Redis

### Introduction

Redis (Remote Dictionary Server) is an in-memory data structure store used as database, cache, and message broker. Created by Salvatore Sanfilippo in 2009, it is known for sub-millisecond latency, rich data structures, and built-in replication and clustering.

### Why It Matters

Redis is the backbone of modern high-performance applications. It powers caching, real-time analytics, leaderboards, session stores, message queues, rate limiting, and distributed locks.

### Real World Analogy

Redis is like a whiteboard in your office. You write things you need quick access to: phone numbers, quick notes, to-do lists. It is fast but volatile. The notebook (disk) stores everything permanently, but the whiteboard is 100x faster.

### Data Structures

| Type | Internal Encoding | Use Case |
|------|-------------------|----------|
| STRING | raw, int, embstr | Cache, counters, sessions, locks |
| LIST | quicklist | Queues, message buffer, timelines |
| SET | intset, hashtable | Tags, unique items, intersections |
| HASH | ziplist, hashtable | Object storage, user profiles |
| ZSET | ziplist, skiplist | Leaderboards, rate limiting, priority queues |
| BITMAP | raw string | User analytics, bloom filters |
| HYPERLOGLOG | raw string | Approximate unique counts (12KB fixed) |
| GEO | zset internally | Location near queries |
| STREAM | radix tree | Event sourcing, message queues, log aggregation |

### String Commands

```bash
SET user:1:name "Alice"
GET user:1:name

MSET user:1:name "Alice" user:1:email "alice@example.com"
MGET user:1:name user:1:email

INCR counter      # 1
INCRBY counter 10 # 11
DECR counter      # 10

SETNX lock:resource:42 "server-1"   # 1 (acquired)
SETNX lock:resource:42 "server-2"   # 0 (already locked)

SET session:abc123 "data" EX 3600   # Expires in 1 hour
TTL session:abc123                  # Remaining seconds

APPEND key "suffix"
STRLEN key
GETRANGE key 0 4
```

### List Commands

```bash
LPUSH queue "task-3"       # Push to head
RPUSH queue "task-1"       # Push to tail
LPOP queue                 # Remove and return head
RPOP queue                 # Remove and return tail
BLPOP queue 30             # Blocking pop (30s timeout)
BRPOP queue 0              # Block indefinitely

LRANGE queue 0 -1          # All elements
LLEN queue                 # List length
LINDEX queue 0             # Element at index 0
LINSERT queue BEFORE "task-2" "task-1.5"
LREM queue 2 "task-x"      # Remove 2 occurrences
LTRIM queue 0 99           # Trim to 100 elements
```

### Set Commands

```bash
SADD tags:post:42 "redis" "database" "tutorial"
SREM tags:post:42 "tutorial"
SMEMBERS tags:post:42          # All members
SISMEMBER tags:post:42 "redis" # 1 (true)
SCARD tags:post:42             # 2 (cardinality)

SUNION tags:post:1 tags:post:2         # Union
SINTER tags:python tags:tutorial       # Intersection
SDIFF tags:python tags:javascript      # Difference

SPOP tags:post:42              # Remove and return random
SRANDMEMBER tags:post:42 2     # 2 random members without removal
```

### Hash Commands

```bash
HSET user:1 name "Alice" age 28 email "alice@example.com"
HGET user:1 name
HGETALL user:1
HKEYS user:1
HVALS user:1
HEXISTS user:1 phone        # 0 (false)
HLEN user:1                 # 3 fields
HINCRBY user:1 login_count 1
HDEL user:1 temp_field
HMGET user:1 name email
```

> Use HASH for storing objects with many fields. Under 100 fields with names < 64 bytes, it is encoded as ziplist for excellent memory efficiency.

### Sorted Set Commands

```bash
ZADD leaderboard 1000 "alice" 850 "bob" 920 "carol"
ZRANGE leaderboard 0 -1 WITHSCORES         # All sorted ascending
ZREVRANGE leaderboard 0 2                  # Top 3 (descending)
ZRANK leaderboard "alice"                  # Position (0 = lowest)
ZREVRANK leaderboard "alice"               # Position (0 = highest)
ZSCORE leaderboard "alice"                 # 1000
ZINCRBY leaderboard "alice" 50             # Score +50
ZCARD leaderboard                          # 4 members
ZCOUNT leaderboard 900 1000                # Members in score range
ZRANGEBYSCORE leaderboard 800 1000         # Members by score
ZREMRANGEBYSCORE leaderboard 500 700       # Remove by score range
ZUNIONSTORE combined 2 set1 set2 WEIGHTS 1 2
ZINTERSTORE common 2 set1 set2
```

### Geo Commands

```bash
GEOADD places 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEODIST places "Palermo" "Catania" km       # 166.2742 km
GEORADIUS places 13.361389 38.115556 200 km WITHCOORD WITHDIST
GEORADIUSBYMEMBER places "Palermo" 200 km
GEOSEARCH places FROMMEMBER "Palermo" BYRADIUS 200 km
```

### Stream Commands

```bash
XADD mystream * sensor-id 1234 temperature 22.5
XRANGE mystream - +                          # All entries
XREAD COUNT 10 BLOCK 5000 STREAMS mystream 0

# Consumer groups
XGROUP CREATE mystream mygroup $
XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >
XACK mystream mygroup 1700000000000-0

XLEN mystream
```

### Pub/Sub

```bash
SUBSCRIBE notifications
PSUBSCRIBE notification:*      # Pattern match

PUBLISH notifications "User signed up!"
PUBLISH notification:email "Welcome email sent"
```

> Pub/Sub is fire-and-forget. If a subscriber disconnects, it misses all messages. For guaranteed delivery, use Streams.

### Transactions

```bash
MULTI
SET balance:alice 100
SET balance:bob 50
EXEC

WATCH balance:alice
val = GET balance:alice
if val >= 50:
    MULTI
    DECRBY balance:alice 50
    INCRBY balance:bob 50
    EXEC
else:
    UNWATCH
```

> Redis transactions have no rollback. If a command fails, subsequent commands still execute. Use WATCH for optimistic locking.

### Lua Scripting

```lua
-- Atomic transfer: KEYS[1] = from, KEYS[2] = to, ARGV[1] = amount
local balance = redis.call('GET', KEYS[1])
if not balance or tonumber(balance) < tonumber(ARGV[1]) then
  return -1  -- insufficient funds
end
redis.call('DECRBY', KEYS[1], ARGV[1])
redis.call('INCRBY', KEYS[2], ARGV[1])
return 0  -- success
```

```bash
EVAL "script_content" 2 account:alice account:bob 50
SCRIPT LOAD "script_content"           # Returns SHA
EVALSHA 4e6d3e... 2 account:alice account:bob 50
```

### Persistence

| Option | Mechanism | Performance | Data Safety |
|--------|-----------|-------------|-------------|
| RDB only | Snapshot at intervals | Best | Lose last N minutes |
| AOF every sec | Append log, fsync 1s | Good | Lose <=1 second |
| AOF always | fsync every write | Slowest | Zero data loss |
| RDB + AOF | Both active | Good | Best (Redis 7+ default) |
| None | No persistence | Maximum | No persistence |

### Replication

```ini
# replica.conf
replicaof 192.168.1.10 6379
replica-read-only yes
repl-backlog-size 100mb
```

```bash
INFO replication
# role:master, connected_slaves:2, slave0:state=online,lag=0
```

### Sentinel (HA)

```ini
# sentinel.conf
sentinel monitor mymaster 192.168.1.10 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

```bash
redis-sentinel /etc/redis/sentinel.conf
redis-cli -p 26379 SENTINEL masters
redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
```

### Redis Cluster

```ini
# cluster.conf on each node
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

```bash
redis-cli --cluster create 192.168.1.10:6379 192.168.1.11:6379 192.168.1.12:6379 \
  192.168.1.20:6379 192.168.1.21:6379 192.168.1.22:6379 --cluster-replicas 1
```

### Cache Patterns

**Cache-Aside (most common):**
```javascript
async function getUser(id) {
  const cacheKey = `user:${id}`;
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  const user = await db.findUser(id);
  if (user) await redis.setex(cacheKey, 3600, JSON.stringify(user));
  return user;
}

async function updateUser(id, data) {
  const user = await db.updateUser(id, data);
  await redis.del(`user:${id}`);  // Invalidate cache
  return user;
}
```

**Write-Through:** Write to cache AND database atomically.
**Write-Behind:** Write to cache, queue DB write asynchronously.

### Rate Limiting (Sliding Window)

```javascript
async function checkRateLimit(userId, maxRequests, windowSeconds) {
  const key = `ratelimit:${userId}`;
  const now = Date.now();
  const windowStart = now - windowSeconds * 1000;

  await redis.zremrangebyscore(key, 0, windowStart);
  const count = await redis.zcard(key);

  if (count >= maxRequests) {
    const oldest = await redis.zrange(key, 0, 0, 'WITHSCORES');
    const retryAfter = Math.ceil((parseInt(oldest[1]) + windowSeconds * 1000 - now) / 1000);
    return { allowed: false, retryAfter };
  }

  await redis.zadd(key, now, `${now}-${Math.random()}`);
  await redis.expire(key, windowSeconds);
  return { allowed: true, remaining: maxRequests - count - 1 };
}
```

### Performance & Pipelining

```javascript
// Without pipeline: N round trips
for (const key of keys) { await redis.get(key); }

// With pipeline: 1 round trip
const pipeline = redis.pipeline();
for (const key of keys) { pipeline.get(key); }
const results = await pipeline.exec();
```

> Pipelining improves throughput 10-100x for batch ops. With 10ms RTT, 1000 commands = 10s without pipeline, ~10ms with.

### Memory & Eviction Policies

| Policy | Behavior |
|--------|----------|
| noeviction | Return errors on writes when full |
| allkeys-lru | Evict least recently used keys |
| allkeys-lfu | Evict least frequently used keys |
| volatile-lru | Evict LRU among keys with TTL |
| volatile-ttl | Evict keys with shortest TTL |
| allkeys-random | Evict random keys |

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Using KEYS in production | Blocks Redis for seconds | Use SCAN with cursor |
| No maxmemory setting | OOM crash when full | Set maxmemory + policy |
| Storing > 10MB values | Blocks single-threaded ops | Split large values, compress |
| Redis as primary DB | Data loss risk | Use persistence + replication |
| No TTL on caches | Memory fills with stale data | Always set EXPIRE |
| Slow commands on huge sets | Blocks everything | Avoid KEYS, SMEMBERS on large sets |

### Best Practices

- Always set maxmemory with eviction policy
- Use SCAN instead of KEYS in production
- Batch with pipeline or transactions
- Use Lua scripts for atomic multi-key operations
- Set TTL on all cache keys
- Use HASH for objects (more efficient than JSON strings)
- Monitor INFO for used_memory, evicted_keys
- Keep keys short/readable: entity:id:field
- Use connection pooling (CPU cores x 2)
- Enable ACTIVE DEFRAG for long-running instances

### Interview Questions

1. **How does Redis handle single-threaded operations and still perform well?** Uses multiplexed I/O (epoll/kqueue) for thousands of connections. Operations are O(1) or O(log N). CPU-bound by compute, not I/O.
2. **Redis vs Memcached?** Redis: rich data structures, persistence, replication, clustering, Lua. Memcached: simpler (strings only), multi-threaded, purely volatile.
3. **RDB vs AOF?** RDB: snapshots, compact, fast recovery, minutes data loss. AOF: every write logged, durable, larger. Hybrid RDB+AOF is default in Redis 7+.
4. **Redis Cluster failover?** When master fails, replica with most current data promoted. Cluster available if majority of master slots covered.
5. **Redlock algorithm?** Acquires locks from majority of nodes (usually 5). Criticized for clock drift assumptions. Many prefer single-node with fencing tokens.

### Practical Exercises

1. Start Redis, insert 1000 keys, measure memory with INFO memory.
2. Implement sliding window rate limiter with sorted sets.
3. Set up master-replica replication, simulate failover.
4. Write Lua script for distributed lock with auto-expiry.
5. GEO commands for "find nearby restaurants" API.
6. Compare throughput with/without pipelining (1000 ops).

### Mini Project: Real-Time Leaderboard

1. ZSET: leaderboard:game:{gameId}. Score = (wins x 100 + points). ZINCRBY for updates.
2. APIs: GET /leaderboard/{gameId}?top=10, GET /rank/{gameId}/{userId}, GET /leaderboard/{gameId}/around/{userId}.
3. TTL for stale entries via ZREMRANGEBYSCORE.
4. "Top players this week" using timestamp-based scores.
5. Cache leaderboard response in STRING with 60s TTL.
6. Pub/Sub channel for real-time leaderboard change notifications.

### Revision Notes

- Single-threaded command execution: avoid O(N) on large keys
- Use hashes for objects (ziplist under 512 fields)
- Pipelining reduces round trips, no atomicity guarantee
- Lua scripts guarantee atomic execution
- Cluster uses 16384 hash slots and gossip protocol
- Sentinel provides HA without clustering
- Hybrid RDB+AOF default in Redis 7+
- Streams replace Pub/Sub for reliable messaging (replay, consumer groups, ACK)
- Always use connection pooling
- Monitor evicted_keys: if non-zero, increase memory or adjust policy

---

## Chapter 59: Normalization (1NF to 5NF)

### Introduction

Database normalization organizes data to minimize redundancy and dependency by decomposing tables based on functional dependencies. Formalized by Edgar Codd (1970) and extended by Boyce, Fagin, and others. Reduces update anomalies, saves storage, and improves data integrity.

### Why It Matters

Unnormalized databases suffer from three update anomalies:
- **Insert anomaly**: Cannot add data without adding unrelated data
- **Update anomaly**: Changing a value requires updating many rows
- **Delete anomaly**: Deleting a row removes unintended data

### Real World Analogy

Normalization is like organizing a library. Instead of writing the author's full bio on every book by that author, you create an author card with their bio and link the book via author ID. When the author wins an award, update one card instead of 50 books.

### 1NF (First Normal Form)

**Rule:** Each cell is atomic (single value). All entries in a column are the same type. Each row has a unique identifier (PK).

**Violation:**
```sql
-- Items and Prices contain comma-separated values
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer VARCHAR(100),
  items VARCHAR(500),     -- "Apple, Banana"
  prices VARCHAR(100)     -- "1.50, 0.75"
);
```

**Fix (1NF):**
```sql
CREATE TABLE orders (
  order_id INT,
  customer VARCHAR(100),
  item VARCHAR(100),
  price DECIMAL(10,2),
  PRIMARY KEY (order_id, item)
);
```

### 2NF (Second Normal Form)

**Rule:** 1NF + every non-key column depends on the ENTIRE primary key (no partial dependency).

**Violation:**
```sql
-- PK: (OrderID, ProductID), but ProductName depends only on ProductID
CREATE TABLE order_details (
  order_id INT,
  product_id INT,
  product_name VARCHAR(100),  -- Partial dependency on product_id only
  quantity INT,
  PRIMARY KEY (order_id, product_id)
);
```

**Fix (2NF):**
```sql
CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  quantity INT,
  PRIMARY KEY (order_id, product_id)
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100)
);
```

### 3NF (Third Normal Form)

**Rule:** 2NF + no transitive dependency (non-key column depends on another non-key column).

**Violation:**
```sql
CREATE TABLE employees (
  emp_id INT PRIMARY KEY,
  dept_id INT,
  dept_name VARCHAR(100),    -- Depends on dept_id, not emp_id
  dept_location VARCHAR(100) -- Depends on dept_id, not emp_id
);
```

**Fix (3NF):**
```sql
CREATE TABLE employees (
  emp_id INT PRIMARY KEY,
  dept_id INT REFERENCES departments(dept_id)
);

CREATE TABLE departments (
  dept_id INT PRIMARY KEY,
  dept_name VARCHAR(100),
  dept_location VARCHAR(100)
);
```

### BCNF (Boyce-Codd Normal Form)

**Rule:** 3NF + for every non-trivial functional dependency X -> Y, X must be a superkey.

**Violation:**
```sql
-- Each subject has one professor, and a student can take many subjects
-- But Subject -> Professor violates BCNF because Subject is not a superkey
CREATE TABLE enrollments (
  student_id INT,
  subject VARCHAR(100),
  professor VARCHAR(100),
  PRIMARY KEY (student_id, subject)
);
```

**Fix (BCNF):**
```sql
CREATE TABLE enrollments (
  student_id INT,
  subject VARCHAR(100),
  PRIMARY KEY (student_id, subject)
);

CREATE TABLE subject_professors (
  subject VARCHAR(100) PRIMARY KEY,
  professor VARCHAR(100)
);
```

### 4NF (Fourth Normal Form)

**Rule:** BCNF + no multi-valued dependencies (no two independent multi-valued facts).

**Violation:**
```sql
-- Skills and Languages are independent. Every skill paired with every language.
CREATE TABLE employee_attributes (
  emp_id INT,
  skill VARCHAR(100),
  language VARCHAR(100),
  PRIMARY KEY (emp_id, skill, language)
);
```

**Fix (4NF):**
```sql
CREATE TABLE employee_skills (emp_id INT, skill VARCHAR(100), PRIMARY KEY (emp_id, skill));
CREATE TABLE employee_languages (emp_id INT, language VARCHAR(100), PRIMARY KEY (emp_id, language));
```

### 5NF (Projection-Join Normal Form)

**Rule:** 4NF + every join dependency is implied by candidate keys. Cannot be decomposed further without losing information. Rarely needed in practice.

### Denormalization Trade-offs

| Aspect | Normalized (3NF) | Denormalized |
|--------|-----------------|--------------|
| Write performance | Faster (one place) | Slower (multiple copies) |
| Read performance | Slower (joins) | Faster (one table) |
| Data integrity | Higher | Lower |
| Storage | Less | More |
| Dev complexity | Moderate | Lower |

> Start at 3NF. Denormalize only when: (1) profiling confirms a read bottleneck, (2) data is read-heavy and rarely updated, (3) you understand write anomaly risks.

### Normal Forms Summary

| Form | Condition | Eliminates |
|------|-----------|------------|
| 1NF | Atomic columns, unique rows | Multi-valued columns |
| 2NF | Full dependency on PK | Partial dependencies |
| 3NF | No transitive dependencies | Transitive dependencies |
| BCNF | Every determinant is a superkey | Overlapping candidate key anomalies |
| 4NF | No multi-valued dependencies | Independent multi-valued facts |
| 5NF | All join dependencies implied by keys | Lossy joins |

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Over-normalizing (5NF/6NF) | Unnecessary complexity, slow reads | Stop at 3NF unless proven needed |
| Under-normalizing | Update anomalies, inconsistency | At minimum reach 3NF |
| Ignoring query patterns | Wrong schema for access patterns | Model around queries after 3NF |
| Not documenting functional dependencies | Incomplete normalization | Document FDs before designing |
| Mixing OLTP and OLAP schemas | Neither optimized | Normalize for OLTP, star schema for OLAP |

### Best Practices

- Default to 3NF for OLTP systems
- Use surrogate keys (auto-increment INT or UUID)
- Document functional dependencies before designing schema
- Test with real query patterns before denormalizing
- Enforce integrity at DB level with constraints
- Use materialized views as alternative to denormalization
- For analytics, use star schema (fact + dimension tables)

### Interview Questions

1. **3NF vs BCNF?** BCNF is stricter: every determinant must be a superkey. 3NF allows non-key to determine non-key in some cases. Every BCNF table is 3NF but not vice versa.
2. **When to denormalize?** When read performance is critical, data is mostly read-only, or joins become too expensive. Always measure first.
3. **Update anomalies?** Insert: cannot add dept without employee. Update: changing dept name needs many rows. Delete: removing employee removes department info.
4. **What is a functional dependency?** X -> Y means each value of X determines exactly one value of Y.
5. **Star schema vs normalized?** Star: central fact + denormalized dimensions, minimal joins, optimized for analytics. Normalized: many related tables, optimized for transactional integrity.

### Practical Exercises

1. Take unnormalized spreadsheet (orders with repeated customer info), normalize to 3NF.
2. Identify functional dependencies in flawed schema, fix them.
3. Convert 3NF schema to BCNF, explain trade-offs.
4. Design library catalog (books, authors, members, loans) normalized to 3NF.
5. Identify BCNF violations in social media schema (users, posts, likes, comments).

### Mini Project: Hospital Management Schema

1. Unnormalized: PatientVisit (PatientID, Name, DOB, Doctor, Diagnosis, Treatment, Room, RoomType).
2. Normalize to 1NF: atomic values, one treatment per row.
3. Normalize to 2NF: split patient data (PatientID -> Name, DOB) from visit data.
4. Normalize to 3NF: doctor specialization, room details.
5. Achieve BCNF if applicable.
6. Write SQL to join back for full patient visit report.
7. Compare query complexity: normalized vs denormalized.

### Revision Notes

- 1NF: atomic columns + PK
- 2NF: no partial dependencies on composite PK
- 3NF: no transitive dependencies
- BCNF: every determinant is a superkey
- 4NF: no independent multi-valued facts
- Default to 3NF; denormalize only for proven performance needs
- Functional dependencies drive normalization
- Normalization reduces write anomalies at the cost of read complexity

---

## Chapter 60: Indexing Deep Dive

### Introduction

Indexes are auxiliary data structures that speed up data retrieval at the cost of additional storage and slower writes. An index is a sorted copy of selected columns organized for fast search: O(log n) instead of O(n) full table scans.

### Why It Matters

Without indexes, every query is a full table scan. On a 10M row table, a sequential scan takes ~30 seconds. With a B-tree index, the same lookup takes under 10ms: a 3000x improvement.

### Real World Analogy

An index is like the back-of-book index. Instead of reading every page to find "Database Normalization", you check the index, get page numbers 142-148, and turn directly there.

### B-Tree Index Internals

```
Root:    [50, 100, 150]
       /     |       \
Level 1: [10,30] [60,80] [120,140,170]
         /  |    /   |    /   |   |   \
Leaves: data data data data data data data
```

**Properties:**
- Balanced: all leaves at same depth
- Each node has sorted keys + pointers to children
- Leaf nodes contain data pointers (or data in clustered indexes)
- Fan-out is typically hundreds: depth stays at 3-4 even for billions of rows
- Node size is usually 8KB (database page size)

**Operations:**
- **Search:** O(log_b N): traverse root to leaf
- **Insert:** locate leaf, insert, split if full, propagate upward
- **Delete:** locate key, remove, rebalance if underfull

### Index Types

| Type | Internal Structure | Best For | Limitations |
|------|-------------------|----------|-------------|
| B-Tree | Balanced tree | Equality + range + sorting | Default for everything |
| Hash | Hash table | Equality lookups | No range, no sorting |
| GiST | Generalized tree | Geometry, full-text, ranges | Slower for simple equality |
| GIN | Inverted index | Arrays, JSONB, full-text | Slower to build, larger |
| SP-GiST | Space-partitioned | Point clustering, network | Less common |
| BRIN | Block range summaries | Large sequential data (logs) | Poor for random lookups |
| Full-text | Inverted + stemmer | Text search, ranking | Text only |
| Spatial | R-Tree variant | Geographic coordinates | Geometry only |

### Composite Index Rules (Leftmost Prefix)

```sql
CREATE INDEX idx_name_dob ON employees(last_name, first_name, dob);
```

**Can use index for:**
- `WHERE last_name = 'Smith'` -- yes
- `WHERE last_name = 'Smith' AND first_name = 'John'` -- yes
- `WHERE last_name = 'Smith' AND first_name = 'John' AND dob = '1990-01-01'` -- yes
- `WHERE first_name = 'John'` -- NO (skips leading column)
- `WHERE last_name = 'Smith' AND dob = '1990-01-01'` -- partial (gap, only last_name used)

**Sorting:**
```sql
-- Index: (department, salary DESC)
ORDER BY department ASC, salary DESC    -- yes (matches)
ORDER BY department DESC, salary ASC    -- yes (reverse scan)
ORDER BY salary DESC                     -- no (leading column missing)
```

> Put high-selectivity columns first. For WHERE status='active' AND email='alice@ex.com', index (email, status) even though status appears first in the query. The planner handles column reordering for equality conditions.

### Covering Index

Contains ALL columns needed by a query, avoiding table (heap) access.

```sql
-- Without covering: Index Scan -> Heap Fetch -> Result
CREATE INDEX idx_email ON users(email);
SELECT email, name FROM users WHERE email = 'alice@example.com';

-- With covering: Index Only Scan -> Result (no heap access)
CREATE INDEX idx_email_covering ON users(email) INCLUDE (name, role);
```

**Benefits:** fewer disk reads, less cache pressure, no table locking for reads.

### Partial & Expression Indexes

```sql
-- Partial: only index active users
CREATE INDEX idx_active_email ON users(email) WHERE active = true;

-- Expression: case-insensitive email lookup
CREATE INDEX idx_lower_email ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Expression: date from timestamp
CREATE INDEX idx_order_date ON orders((created_at::date));
```

### Clustered vs Non-Clustered

| Feature | Clustered (InnoDB) | Non-Clustered (Postgres default) |
|---------|-------------------|----------------------------------|
| Leaf nodes | Full row data | Pointers (TID) to heap |
| Per table | 1 | Unlimited |
| PK | Clustered by default | Separate from data |
| Secondary index | PK lookup -> row (double) | TID -> row |
| Data order | Sorted by index | Heap (insertion order) |
| Insert speed | Slower (maintains order) | Faster (append) |

### When to Index (and NOT)

**DO index:** WHERE equality, range queries, IN, ORDER BY, JOIN conditions, IS NULL, expression (with expression index).

**DO NOT index:** Negative queries (WHERE status != 'active'), LIKE '%prefix' (wildcard prefix), low-cardinality columns (boolean), functions without expression index, small tables (<1000 rows).

### Index Maintenance

```sql
-- PostgreSQL
REINDEX INDEX idx_users_email;
REINDEX TABLE users;

-- MySQL
OPTIMIZE TABLE users;

-- Find unused indexes (PostgreSQL)
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes WHERE idx_scan = 0 ORDER BY tablename;
```

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| No FK indexes | Full scans on JOINs | Index all foreign keys |
| Separate vs composite | Only first column used efficiently | Create composite for multi-column queries |
| Low-cardinality columns | Index never used (too many rows per value) | Only index high-selectivity columns |
| Over-indexing | Slow writes, large storage | Target actual query patterns |
| Not using covering indexes | Extra heap lookups | Add INCLUDE columns |
| Ignoring partial indexes | Wasted index space | Use WHERE clause in CREATE INDEX |
| Not monitoring unused indexes | Unnecessary maintenance | Check usage stats monthly |

### Best Practices

- Index WHERE, JOIN, ORDER BY columns
- Composite indexes: lead with highest selectivity
- Use covering indexes (INCLUDE) for critical query paths
- Use partial indexes for filtered subsets
- CREATE INDEX CONCURRENTLY (Postgres) to avoid locking
- Run ANALYZE after significant data changes
- Monitor index usage quarterly, drop unused ones
- Avoid low-cardinality indexes unless in composite
- For time-series: BRIN (Postgres) or DESC (MySQL 8+)
- Use EXPLAIN ANALYZE before and after creating indexes

### Interview Questions

1. **How does a B-tree work internally?** Balanced tree: root -> internal nodes -> leaves. Sorted keys + pointers. Search O(log_b N). Depth 3-4 for billions of rows. Fan-out of hundreds.
2. **Clustered vs non-clustered?** Clustered stores data rows in index order (one per table). Non-clustered stores pointers to data. InnoDB PK is clustered. Postgres uses heap with non-clustered indexes.
3. **Composite index leftmost prefix?** Index on (a,b,c) used for (a), (a,b), (a,b,c) but not (b) alone or (c) alone.
4. **What is a covering index?** Contains all columns for a query. Enables index-only scan. No heap access needed. Uses INCLUDE for extra columns.
5. **When does an index hurt performance?** Small tables (full scan faster), heavy writes (each write updates all indexes), low-cardinality columns, rarely-queried columns.
6. **How do partial indexes work?** CREATE INDEX ... WHERE condition. Only rows matching condition are indexed. Smaller and faster. Great for active=true, status='active' queries.

### Practical Exercises

1. Create a 10M row table, compare SELECT performance with/without index.
2. Design composite indexes for a complex WHERE + ORDER BY query.
3. Use pg_stat_user_indexes to find and drop unused indexes.
4. Create partial index, verify it is used (EXPLAIN).
5. Create covering index, verify index-only scan in EXPLAIN output.

### Mini Project: E-Commerce Index Optimization

1. Tables: products(10M), orders(50M), order_items, customers(5M).
2. Identify slow queries from slow query log.
3. Design indexes: composite for (status, created_at), partial for active products, covering for order summary queries.
4. Measure before/after with EXPLAIN ANALYZE.
5. Monitor index usage, drop unused ones.

### Revision Notes

- B-tree is the default for most use cases
- Composite: lead with highest selectivity
- Covering: no heap access
- Partial: smaller index for subsets
- Hash: equality only, smaller than B-tree
- GIN: arrays, JSONB, full-text
- BRIN: huge sequential data
- GiST: geometry, ranges
- Every index slows writes: only create what is needed
- Monitor and drop unused indexes

---

## Chapter 61: Transactions & ACID

### Introduction

A transaction is a unit of work treated as a single, indivisible operation. The ACID properties ensure reliable processing of database transactions, even during failures, crashes, and concurrent access.

### Why It Matters

Without transactions, concurrent operations cause race conditions, partial updates leave data inconsistent, and system crashes corrupt data. Every financial transaction, e-commerce order, and critical update depends on ACID guarantees.

### Real World Analogy

A bank transfer: debit Alice $100, credit Bob $100. If the power fails after the debit but before the credit, $100 disappears. A transaction ensures both complete or neither happens, maintaining consistency.

### ACID Properties

| Property | Meaning | Purpose |
|----------|---------|---------|
| **A**tomicity | All or nothing | Prevents partial updates |
| **C**onsistency | Valid state to valid state | Maintains data integrity |
| **I**solation | Concurrent transactions dont interfere | Prevents race conditions |
| **D**urability | Committed changes survive failures | Ensures data persistence |

### Transaction Control SQL

```sql
-- Basic transaction
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 10;
ROLLBACK;

-- Savepoints (partial rollback)
BEGIN;
INSERT INTO orders (user_id, total) VALUES (42, 150.00);
SAVEPOINT after_order;
INSERT INTO order_items (order_id, product_id, qty) VALUES (LAST_INSERT_ID(), 5, 2);
-- If order_items fails:
ROLLBACK TO after_order;
-- Fix and retry
COMMIT;
```

### Isolation Levels

| Level | Dirty Read | Non-repeatable Read | Phantom Read | Lost Update |
|-------|-----------|---------------------|--------------|-------------|
| READ UNCOMMITTED | Possible | Possible | Possible | Possible |
| READ COMMITTED | Prevented | Possible | Possible | Possible |
| REPEATABLE READ | Prevented | Prevented | Prevented (PG) / Possible (MySQL) | Prevented |
| SERIALIZABLE | Prevented | Prevented | Prevented | Prevented |

### Read Phenomena

- **Dirty Read**: Read uncommitted data from another transaction that may be rolled back.
- **Non-repeatable Read**: Read same row twice, get different values (another transaction updated it between reads).
- **Phantom Read**: Same query returns different row sets (another transaction inserted/deleted rows).
- **Lost Update**: Two transactions read same value, both modify it, one overwrites the other.

### How Isolation Levels Work

**READ UNCOMMITTED:** No locks for reads. Writers lock rows. Most permissive, rarely used.

**READ COMMITTED:** Each statement sees a fresh snapshot. Implementation:
- PostgreSQL: Each query gets a new snapshot. No read locks.
- MySQL/InnoDB: Consistent read using undo log. Snapshot per statement.

**REPEATABLE READ:** Transaction sees a consistent snapshot from the first query.
- PostgreSQL: Snapshot at start of transaction. Prevents phantoms (uses snapshot).
- MySQL/InnoDB: Snapshot at first read. Phantom rows can appear (non-locking reads).

**SERIALIZABLE:** Transactions execute as if serial (one after another).
- PostgreSQL: Serializable Snapshot Isolation (SSI) with conflict detection and rollback.
- MySQL: Lock-based serialization (all reads are locking reads).

### MVCC (Multi-Version Concurrency Control)

MVCC is the mechanism behind isolation. Instead of locks, each transaction sees a snapshot of data.

**PostgreSQL MVCC:**
- Each row has xmin (creating XID) and xmax (deleting XID)
- Each transaction has a snapshot of in-progress XIDs
- A row is visible if its xmin is committed and xmax is not committed (or > transaction XID)
- Dead tuples cleaned by VACUUM

**InnoDB MVCC:**
- Undo logs in rollback segments
- Each transaction gets a transaction ID
- Consistent read reconstructs row from undo log
- No vacuum needed (undo log is cleaned when no transaction needs it)

### Deadlocks

A deadlock occurs when two transactions each hold locks the other needs.

```
Transaction 1: UPDATE accounts SET balance=x WHERE id=1;
Transaction 2: UPDATE accounts SET balance=y WHERE id=2;
Transaction 1: UPDATE accounts SET balance=z WHERE id=2;  -- Waits for T2
Transaction 2: UPDATE accounts SET balance=w WHERE id=1;  -- Waits for T1 (DEADLOCK!)
```

**Detection:** InnoDB detects cycles immediately and rolls back the transaction that made the least progress. PostgreSQL detects deadlocks via timeout.

**Prevention:**
- Access tables/rows in consistent order
- Keep transactions short
- Use lock timeouts
- Consider SERIALIZABLE isolation with retry logic

### Two-Phase Commit (XA Transactions)

```sql
-- PostgreSQL two-phase commit
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
PREPARE TRANSACTION 'txn_001';

-- Later, if all participants prepared:
COMMIT PREPARED 'txn_001';
-- Or if any failed:
ROLLBACK PREPARED 'txn_001';
```

Used for distributed transactions across multiple databases or message queues. Rare in practice — most modern apps use compensating transactions or saga patterns instead.

### Transaction Best Practices by Database

| Database | Default Isolation | Notes |
|----------|------------------|-------|
| PostgreSQL | READ COMMITTED | Can use REPEATABLE READ or SERIALIZABLE. No dirty reads even at READ UNCOMMITTED. |
| MySQL/InnoDB | REPEATABLE READ | READ COMMITTED is recommended for most apps. REPEATABLE READ uses more memory. |
| MongoDB | Read uncommitted (pre-4.0), snapshot for transactions | Multi-doc transactions since 4.0, but slower. |
| SQLite | SERIALIZABLE | Only one writer at a time (table-level locking). |
| SQL Server | READ COMMITTED | Snapshot isolation available as an option. |
| Oracle | READ COMMITTED | Only READ COMMITTED and SERIALIZABLE supported. |

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Not using transactions for multi-step ops | Partial updates, data corruption | Wrap in BEGIN/COMMIT |
| Holding transactions too long | Blocking, lock contention | Keep short |
| Wrong isolation level | Phantom reads or deadlocks | Choose minimum needed |
| No error handling/rollback | Committed partial work | Try/Catch with ROLLBACK |
| Ignoring deadlocks | Failed requests | Implement retry logic |
| Forgetting SAVEPOINT usage | Full rollback when partial is enough | Use savepoints |
| Not setting lock timeout | Indefinite waiting | SET lock_timeout |

### Best Practices

- Use transactions for all multi-statement operations
- Keep transactions as short as possible
- Choose the minimum isolation level that meets requirements
- Implement retry logic for serialization failures
- Set reasonable lock timeouts
- Access tables in consistent order to prevent deadlocks
- Monitor lock contention (pg_locks, SHOW ENGINE INNODB STATUS)
- Test with concurrent load before production

### Interview Questions

1. **What is the difference between READ COMMITTED and REPEATABLE READ?** READ COMMITTED: each statement sees latest committed data, non-repeatable reads possible. REPEATABLE READ: consistent snapshot for entire transaction, same row returns same value.
2. **How does MVCC work?** Each transaction sees a snapshot of committed data. Row versions track creating/deleting transaction IDs. Readers do not block writers. Old versions are cleaned up (VACUUM or undo log).
3. **What causes dirty reads and how do you prevent them?** Dirty reads occur when a transaction reads uncommitted data. Prevented by READ COMMITTED isolation or higher.
4. **Explain the phantom read problem.** A transaction re-executes a query and gets different rows because another transaction inserted rows matching the WHERE clause. REPEATABLE READ prevents this in PostgreSQL (snapshot) but not in MySQL (unless using locking reads).
5. **How do you handle deadlocks in production?** Implement retry logic with exponential backoff. Keep transactions short. Access resources in consistent order. Monitor with SHOW ENGINE INNODB STATUS / pg_locks.

### Practical Exercises

1. Open two SQL sessions. Execute UPDATE in one without COMMIT, try reading in the other. Observe isolation behavior.
2. Create a deadlock scenario: two transactions updating two rows in opposite order.
3. Implement a money transfer stored procedure with SAVEPOINT rollback on error.
4. Measure throughput difference between READ COMMITTED and SERIALIZABLE.
5. Test MVCC: long-running SELECT sees consistent snapshot vs short query seeing latest data.

### Mini Project: Banking Transaction System

1. Tables: accounts(id, user_id, balance, version), transactions(id, from_account, to_account, amount, status, created_at).
2. Implement transfer: BEGIN, debit, credit, INSERT transaction record, COMMIT.
3. Handle: insufficient funds, concurrent transfers (optimistic locking with version column), deadlock retry.
4. Write script to simulate 100 concurrent transfers, measure failure rate.
5. Compare performance at READ COMMITTED vs REPEATABLE READ vs SERIALIZABLE.

### Revision Notes

- ACID: Atomicity, Consistency, Isolation, Durability
- Isolation levels: READ UNCOMMITTED < READ COMMITTED < REPEATABLE READ < SERIALIZABLE
- MVCC enables non-blocking reads: readers never block writers
- Deadlocks: detect and retry, or prevent with consistent resource ordering
- Keep transactions short: do not hold locks longer than needed
- Savepoints allow partial rollback within a transaction
- Two-phase commit for distributed XA transactions
- SERIALIZABLE: strongest guarantees, highest contention, use only when needed
- Each database implements isolation differently: understand your DBs specific behavior

---

## Chapter 62: Query Optimization

### Introduction

Query optimization is the process of tuning SQL queries and database configuration to improve performance. It involves understanding how the query planner works, reading EXPLAIN plans, and applying indexing strategies.

### Why It Matters

A poorly written query on a 10M row table can take 30 seconds. The same query optimized can run in 10ms — a 3000x improvement. Query optimization is the highest-leverage performance activity for database-backed applications.

### Real World Analogy

Query optimization is like choosing the right route for a delivery truck. You can take local roads (full table scan) or the highway (index scan). The GPS (query planner) chooses based on traffic data (statistics), but sometimes you need to know the shortcuts.

### Understanding EXPLAIN

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT u.name, o.total
FROM users u JOIN orders o ON u.id = o.user_id
WHERE u.email = 'alice@example.com';

-- MySQL
EXPLAIN FORMAT=JSON
SELECT u.name, o.total
FROM users u JOIN orders o ON u.id = o.user_id
WHERE u.email = 'alice@example.com';

-- SQLite
EXPLAIN QUERY PLAN
SELECT * FROM users WHERE email = 'alice@example.com';
```

### Key EXPLAIN Metrics

| Metric | What It Tells You | Target |
|--------|------------------|--------|
| rows (estimated) | How many rows planner thinks it will examine | Low |
| actual rows | Actual rows examined (with ANALYZE) | Low |
| loops | How many times the node runs | 1 |
| buffers (shared hit) | Pages found in cache | High hit rate |
| buffers (shared read) | Pages read from disk | Low |
| execution time | Actual wall-clock time | Low |
| planning time | Time spent planning | Low |

### Scan Types

**Sequential Scan (Seq Scan):** Reads every row in the table. Fast for small tables or when retrieving most rows. Bad for selective queries on large tables.

**Index Scan:** Uses B-tree index to find row pointer, then reads full row from heap. Good for selective queries.

**Index Only Scan:** All needed columns in the index. No heap access. Best performance.

**Bitmap Index Scan (+ Bitmap Heap Scan):** Finds multiple index entries, sorts by physical location, reads heap in physical order. Better than Index Scan when many rows (>5%) match.

**Bitmap Heap Scan with Recheck:** When visibility map is fuzzy, rechecks conditions for each row.

### Join Strategies

| Strategy | Algorithm | When It Is Used |
|----------|-----------|-----------------|
| Nested Loop | For each outer row, scan inner | Small result sets, good indexes |
| Hash Join | Build hash table of inner, probe outer | Large, unsorted data, no indexes |
| Merge Join | Sort both sides, then merge | Pre-sorted data (ORDER BY or index), large sets |

```sql
-- Force specific join (PostgreSQL)
SET enable_hashjoin = off;
SET enable_mergejoin = off;
SET enable_nestloop = off;

-- Hints (Oracle, MySQL 8+)
SELECT /*+ HASH_JOIN(u o) */ * FROM users u JOIN orders o ON u.id = o.user_id;
```

### Indexing Strategies

**B-Tree Index Rule of Thumb:**
- Equality conditions: single column index is fine
- Equality + range: composite index, equality columns first
- ORDER BY: index on the ORDER BY column
- GROUP BY: index on GROUP BY columns
- JOIN: index on foreign key column

**Common patterns:**

```sql
-- Typical e-commerce query
SELECT * FROM orders
WHERE status = 'pending' AND created_at > NOW() - INTERVAL '7 days'
ORDER BY created_at DESC LIMIT 50;

-- Best index: (status, created_at DESC)
CREATE INDEX idx_orders_status_created ON orders(status, created_at DESC);
```

### Query Rewriting

Replace these patterns for better performance:

| Anti-Pattern | Bad | Good |
|-------------|-----|------|
| IN with subquery | WHERE id IN (SELECT user_id FROM ...) | WHERE EXISTS (SELECT 1 FROM ... WHERE user_id = id) |
| NOT IN with subquery | WHERE id NOT IN (SELECT user_id FROM active_users) | WHERE NOT EXISTS (SELECT 1 FROM active_users WHERE user_id = id) |
| OR on different columns | WHERE status = 'x' OR priority = 'y' | UNION ALL with separate queries |
| LIKE prefix wildcard | WHERE name LIKE '%smith%' | Full-text search or pg_trgm |
| SELECT * | SELECT * FROM users | SELECT id, name, email |
| Function on indexed column | WHERE YEAR(created_at) = 2024 | WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01' |

### Common Query Performance Problems

**N+1 Query Problem:**
```javascript
// Bad: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  const orders = await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
}

// Good: Single query
const usersWithOrders = await db.query(`
  SELECT u.*, o.id as order_id, o.total
  FROM users u LEFT JOIN orders o ON u.id = o.user_id
`);
```

**Pagination with OFFSET:**
```sql
-- Bad: OFFSET skips rows (scanning all previous rows)
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 100000;

-- Good: Keyset (cursor-based) pagination
SELECT * FROM users WHERE id > 100000 ORDER BY id LIMIT 20;
```

**Unnecessary Sorting:**
```sql
-- Bad: Extra sort step
SELECT * FROM users ORDER BY name;

-- Good: Index on name avoids sort
CREATE INDEX idx_users_name ON users(name);
```

### Slow Query Log

```ini
# PostgreSQL postgresql.conf
log_min_duration_statement = 1000   # Log queries > 1 second
log_line_prefix = '%t [%p-%l] %u %d '

# MySQL my.cnf
slow_query_log = 1
long_query_time = 2
slow_query_log_file = /var/log/mysql/slow.log
log_queries_not_using_indexes = 1
```

### Profiling Tools

| Tool | Database | Purpose |
|------|----------|---------|
| pg_stat_statements | PostgreSQL | Query performance statistics |
| auto_explain | PostgreSQL | Auto-log slow queries with plans |
| pgBadger | PostgreSQL | Analyze log files, generate reports |
| slow query log + pt-query-digest | MySQL | Analyze slow queries |
| performance_schema | MySQL | Detailed query instrumentation |
| Sysbench, pgbench | Both | Load testing and benchmarking |

**Using pg_stat_statements:**
```sql
SELECT query, calls, total_exec_time, mean_exec_time, rows,
       shared_blks_hit, shared_blks_read, shared_blks_dirtied
FROM pg_stat_statements
WHERE query NOT LIKE '%pg_%'
ORDER BY total_exec_time DESC
LIMIT 10;
```

### Optimization Workflow

1. **Identify slow queries**: slow query log, pg_stat_statements, application tracing
2. **EXPLAIN ANALYZE**: understand the plan
3. **Identify the bottleneck**: Seq scan? Sort? Nested loop on wrong side?
4. **Fix**: add index, rewrite query, update statistics
5. **Verify**: re-run EXPLAIN ANALYZE, compare timings
6. **Monitor**: ensure fix works under production load

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| No EXPLAIN before optimizing | Fixing wrong problem | Always check |
| Adding indexes without monitoring | Wasted writes, storage | Check index usage after 1 week |
| Over-optimizing rare queries | Wasted effort, complexity | Optimize the 10 slowest queries |
| Ignoring N+1 in ORMs | Many small queries | Use eager loading, batch queries |
| Wrong join type | Bad plans | Understand join strategies |
| No statistics (ANALYZE) | Bad plans | ANALYZE after bulk changes |
| SELECT * everywhere | No covering indexes | Specify needed columns |
| OFFSET for pagination | Slower as page grows | Use cursor-based pagination |

### Best Practices

- Enable slow query logging in all environments
- Use EXPLAIN ANALYZE (not just EXPLAIN) to see actual costs
- Regularly ANALYZE tables to update statistics
- Monitor with pg_stat_statements or performance_schema
- Profile before optimizing: measure, dont guess
- Optimize the 10 slowest queries first (Pareto principle)
- Rewrite queries before adding indexes where possible
- Use parameterized queries (avoid query cache pollution and injection)
- Be careful with ORMs: understand what SQL they generate

### Interview Questions

1. **How do you identify and fix a slow query?** Enable slow query log. Find slow query. Run EXPLAIN ANALYZE. Check for seq scans, sorts, row estimates. Add index or rewrite query. Re-check.
2. **What is the difference between Index Scan and Index Only Scan?** Index Scan: find entry, fetch row from heap. Index Only Scan: all needed columns in index, no heap access.
3. **When would the query planner choose a sequential scan over an index?** When table is small (<1000 rows), or query returns large % of table (>10-20%), or planner thinks index would be slower based on statistics.
4. **How do you debug an N+1 query problem?** Log all queries with duration. Notice repeated identical queries in loop. Eager load or batch join. For Rails: includes/eager_load. For Django: select_related/prefetch_related.
5. **What is the difference between nested loop, hash join, and merge join?** Nested Loop: for each outer row, scan inner. Best for small sets with indexes. Hash Join: build hash table of inner, probe with outer. Best for large unsorted data. Merge Join: sort both, merge. Best for pre-sorted large data.

### Practical Exercises

1. Enable slow query log, run some queries, review the log.
2. Take a slow query with a sequential scan, EXPLAIN ANALYZE it, then add an index, re-EXPLAIN.
3. Rewrite a query with IN subquery to use EXISTS, compare plans.
4. Implement cursor-based pagination vs OFFSET pagination, benchmark latency.
5. Use pg_stat_statements to find top 5 slowest queries in a test database.

### Mini Project: Query Performance Dashboard

1. Create a 5M row orders table with various indexes.
2. Generate 10 query patterns (aggregations, joins, filters, sorts).
3. Use EXPLAIN ANALYZE to measure each.
4. Profile bottlenecks, add indexes, rewrite queries.
5. Track improvements in a spreadsheet (baseline vs optimized).
6. Implement pg_stat_statements to continuously monitor query performance.

### Revision Notes

- EXPLAIN (ANALYZE, BUFFERS) is the primary tool
- Seq scan = bad on large tables (but normal on small ones)
- Nested loop = good for small sets with indexes
- Hash join = good for large unsorted sets
- Merge join = good for pre-sorted data
- Covering indexes avoid heap lookups
- Partial indexes save space for filtered queries
- ANALYZE maintains statistics for the planner
- Enable slow query log in every environment
- Fix the Pareto 20% of queries causing 80% of the slow time
- N+1 queries are the #1 ORM performance killer
- Use cursor pagination for large offset pagination

---

## Chapter 63: Backup & Recovery

### Introduction

Database backups protect against data loss from hardware failure, software bugs, human error, and security incidents. A backup strategy defines what to back up, how often, where to store it, and how to restore it.

### Why It Matters

Hard drives fail. Application bugs corrupt data. Operators delete tables by accident. Ransomware encrypts databases. Without tested backups, these events mean permanent data loss. A backup is not a backup until it has been successfully restored.

### Real World Analogy

Backups are like insurance. You pay the premium (time, storage, cost) and hope you never need it. But when disaster strikes, the difference between a good and bad backup is the difference between recovering in minutes and losing everything.

### Backup Types

| Type | Description | Size | Recovery Time | Frequency |
|------|-------------|------|---------------|-----------|
| Full | Complete copy of all data | Largest | Slowest | Weekly/daily |
| Incremental | Changes since last backup | Smallest | Medium | Hourly/daily |
| Differential | Changes since last full backup | Medium | Fast | Daily |
| Transaction log | Every individual change | Very small | Point-in-time | Continuous |

### Logical vs Physical Backup

| Aspect | Logical Backup | Physical Backup |
|--------|---------------|-----------------|
| Content | SQL statements (INSERT, CREATE) | Raw filesystem copy |
| Speed | Slow for large databases | Fast (file copy) |
| Portability | Cross-version, cross-platform | Same version, same platform |
| Granularity | Table-level possible | Database-level |
| Size | Larger (includes index rebuilds) | Compact |
| Tool MySQL | mysqldump, mysqlpump | XtraBackup |
| Tool Postgres | pg_dump, pg_dumpall | pg_basebackup |

### PostgreSQL Backup

**Logical Backup:**
```bash
# Custom format (compressed, parallel restore)
pg_dump -U admin -d mydb --format=custom -f mydb_backup.dump

# Plain SQL
pg_dump -U admin -d mydb > mydb.sql

# Restore
pg_restore -U admin -d mydb --jobs=4 mydb_backup.dump
psql -U admin -d mydb < mydb.sql
```

**Physical Backup:**
```bash
pg_basebackup -D /backups/pgsql/ -X stream -P -U replicator
```

**Point-in-Time Recovery:**
```ini
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backups/wal/%f'
```

```bash
# Recovery: create recovery.signal + postgresql.auto.conf
# restore_command = 'cp /backups/wal/%f %p'
# recovery_target_time = '2024-03-15 14:30:00 UTC'
```

### MySQL Backup

**Logical:**
```bash
# InnoDB consistent backup (MVCC, no locking)
mysqldump -u root -p --single-transaction --quick mydb > mydb.sql

# mysqlpump (parallel, 5.7+)
mysqlpump -u root -p mydb --default-parallelism=4 > mydb.sql

# Restore
mysql -u root -p mydb < mydb.sql
```

**Physical (XtraBackup):**
```bash
xtrabackup --backup --target-dir=/backups/mysql/ --user=root
xtrabackup --prepare --target-dir=/backups/mysql/
```

**PITR:**
```bash
mysqlbinlog /var/log/mysql/mysql-bin.000023 \
  --stop-datetime="2024-03-15 14:30:00" | mysql -u root -p
```

### MongoDB & Redis Backup

**MongoDB:**
```bash
mongodump --uri="mongodb://localhost:27017/mydb" --gzip --archive=/backups/mongo/mydb.gz
mongorestore --uri="mongodb://localhost:27017/mydb" --gzip --archive=/backups/mongo/mydb.gz
```

**Redis:**
```bash
redis-cli BGSAVE
cp /var/lib/redis/dump.rdb /backups/redis/
```

### 3-2-1 Rule and Retention

**3 copies, 2 different media, 1 off-site.**

| Backup Type | Retention | Purpose |
|-------------|-----------|---------|
| Hourly | 24-48 hours | Quick recovery |
| Daily | 30 days | Common recovery point |
| Weekly | 3-6 months | Longer retention |
| Monthly | 1-2 years | Compliance |

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Never testing backups | Corruption goes unnoticed | Monthly restore tests |
| Same server storage | Server failure = backup loss | Separate storage |
| No monitoring | Failures undetected | Alerting on backup status |
| Unencrypted backups | Data exposure | Encrypt all backups |
| Single backup only | Corruption propagates | 3-2-1 rule |
| No PITR | Only last backup time | Enable WAL/binlog archiving |

### Best Practices

- Follow the 3-2-1 rule
- Automate backups with cron or K8s CronJobs
- Encrypt at rest (AES-256) and in transit (TLS)
- Monitor success/failure with alerts
- Test restores monthly
- Enable PITR in production
- Document and practice restore procedures
- Use immutable backups (S3 Object Lock) for ransomware protection
- Retention policy balances cost vs recovery needs

### Interview Questions

1. **3-2-1 rule?** 3 copies, 2 media, 1 off-site.
2. **Logical vs physical?** Logical: SQL, portable, slower. Physical: raw files, fast, platform-specific.
3. **What is PITR?** Restore to any moment using continuous WAL/binlog archiving. Enables recovery to right before an accident.
4. **Why test backups?** Files can corrupt silently. Only restore validates integrity. Many discover corruption during actual disasters.
5. **Incremental vs differential?** Incremental: changes since any last backup. Differential: changes since last full. Differential is simpler for recovery (full + 1 diff).

### Practical Exercises

1. Full backup with pg_dump, drop a table, restore it.
2. Set up WAL archiving, perform PITR.
3. Use XtraBackup, prepare and restore.
4. Encrypt backup with GPG, verify decryption.
5. Automated backup script with cron, test restore.

### Mini Project: Automated Backup System

1. Nightly pg_dump with custom format + gzip.
2. Encrypt with GPG.
3. Upload to S3 (or remote server).
4. Retention: 7 daily, 4 weekly, 12 monthly.
5. Monitoring alerts.
6. Monthly restore test to staging database.

### Revision Notes

- 3-2-1: 3 copies, 2 media, 1 off-site
- Logical (pg_dump/mysqldump): portable, table-level
- Physical (pg_basebackup/XtraBackup): fast, DB-level
- PITR requires continuous WAL/binlog archiving
- Test restores are mandatory
- Encrypt backups at rest and in transit
- Automate scheduling, encryption, upload, monitoring
- Ransomware: immutable/air-gapped backups
- Document and practice restore procedure

---

## Chapter 64: Scaling Databases

### Introduction

As data and traffic grow, databases must scale to handle increased load. Scaling strategies range from simple vertical upgrades to complex distributed architectures. The right approach depends on your workload, budget, and consistency requirements.

### Why It Matters

Every successful application eventually hits database limits. Understanding when and how to scale prevents outages, maintains performance, and controls costs. Premature scaling adds complexity; delayed scaling causes downtime.

### Real World Analogy

Scaling a database is like expanding a restaurant. At first, you add more tables (vertical scaling: bigger server). Then you add a takeout window (read replicas: offload orders). Finally, you open a second location (sharding: split customers by region).

### Vertical Scaling (Scale Up)

Adding more resources (CPU, RAM, disk) to a single server.

**Pros:** Simple, no code changes, strong consistency, ACID across all data.

**Cons:** Hardware limits, expensive at high end, single point of failure, diminishing returns.

```yaml
# Common vertical scaling path
Development:  2 CPU, 4GB RAM, 100GB SSD
Small:        4 CPU, 16GB RAM, 500GB SSD
Medium:       8 CPU, 64GB RAM, 1TB SSD
Large:        32 CPU, 256GB RAM, 4TB NVMe
X-Large:      96 CPU, 1TB RAM, 10TB NVMe (limit)
```

> When to scale vertically: first step for most apps. Cost-effective at smaller sizes. Do this BEFORE considering sharding.

### Read Replicas

Offload read queries from the primary to replica databases.

```yaml
Architecture:
  Primary: handles writes + critical reads
  Replica 1: handles read queries (analytics, reporting)
  Replica 2: handles read queries (API reads)
  Replica 3: standby for failover
```

**Trade-offs:**
- Replication lag: reads may return stale data
- No write scaling: all writes still go to primary
- Connection management: need read/write splitting logic

```javascript
// Read/write splitting (application level)
const db = {
  async write(query, params) {
    return pool.write.query(query, params);  // Primary
  },
  async read(query, params) {
    if (query.requiresFreshData) {
      return pool.write.query(query, params);  // Primary for fresh reads
    }
    return pool.read.query(query, params);     // Replica for stale-ok reads
  }
};
```

> Use read replicas when read:write ratio exceeds 5:1. Many apps are 20:1 or higher.

### Caching

Add a cache layer (Redis, Memcached) to reduce database load.

| Cache Pattern | Description | Use Case |
|--------------|-------------|----------|
| Cache-aside | App checks cache first, loads on miss | General purpose |
| Read-through | Cache loads from DB on miss | Simple reads |
| Write-through | Write to cache and DB simultaneously | Consistency critical |
| Write-behind | Write to cache, async write to DB | High write throughput |

> Caching is the most cost-effective scaling strategy. A Redis instance with 32GB RAM can handle millions of reads/second, reducing DB load by 90%+.

### Connection Pooling

Reuse database connections instead of opening new ones per request.

**Tools:** PgBouncer, Pgpool-II (Postgres), ProxySQL (MySQL), HikariCP (Java).

**Without pool:** Each request opens/closes a connection. At 1000 req/s with 10ms connect time, 10 seconds spent connecting.

**With pool:** Pre-opened connections reused. Zero connection overhead.

```ini
# PgBouncer configuration
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction         # Best for web apps
max_client_conn = 500
default_pool_size = 20          # Per database pool
max_db_connections = 50
```

### Partitioning (Table-Level)

Split a single table into smaller physical segments while appearing as one logical table.

```sql
-- PostgreSQL declarative partitioning
CREATE TABLE orders (
  id BIGSERIAL, user_id INT, total DECIMAL(10,2), created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_q1 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
CREATE TABLE orders_2024_q2 PARTITION OF orders
  FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- MySQL partitioning
CREATE TABLE orders (
  id BIGINT NOT NULL,
  created_at DATE NOT NULL,
  total DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p2022 VALUES LESS THAN (2023),
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025)
);
```

**Benefits:** Easier archiving (drop old partitions), faster range scans (partition pruning), parallel scanning.

**Limitations:** All partitions on same server (no horizontal scaling), complex cross-partition queries.

### Sharding (Horizontal Scaling)

Distribute data across multiple database instances.

```yaml
Shard 1 (users 1-1000000):  Primary + Replica
Shard 2 (users 1000001-2000000): Primary + Replica
Shard 3 (users 2000001-3000000): Primary + Replica
Shard N (...)
```

**Shard Key Selection:**
```javascript
// Hash-based sharding
function getShard(userId, totalShards) {
  return crypto.createHash('md5').update(String(userId)).digest().readUInt32BE(0) % totalShards;
}

// Range-based sharding
function getShard(userId) {
  if (userId <= 1_000_000) return 'shard1';
  if (userId <= 2_000_000) return 'shard2';
  return 'shard3';
}

// Directory-based (lookup table)
async function getShard(userId) {
  const shard = await shardMap.findOne({ userId });
  return shard.shardId;
}
```

**Sharding Challenges:**
- Cross-shard queries (joins, aggregations) are complex/inefficient
- Resharding (adding/removing shards) is difficult
- Distributed transactions (2PC) are slow
- Schema changes must be applied to all shards
- Shard key cannot change after initial sharding

| Shard Strategy | Cardinality | Distribution | Query Support | Complexity |
|---------------|-------------|--------------|---------------|------------|
| Hash | High | Even | Point queries only | Low |
| Range | High | Uneven (hot spots) | Range queries | Medium |
| Directory | Arbitrary | Controlled | Flexible | High |
| Geo (location) | Medium | Region-based | Geographic | Medium |
| Time-based | Low | Uneven (recent hot) | Time-range | Low |

### Distributed SQL Databases

Newer databases that provide horizontal scaling with ACID guarantees:

| Database | Protocol | Consistency | Sharding |
|----------|----------|-------------|----------|
| CockroachDB | PostgreSQL | Serializable | Auto, range-based |
| YugabyteDB | PostgreSQL/HBase | Strong | Auto, hash/range |
| Google Spanner | Custom | External consistency | Auto, range-based |
| TiDB | MySQL | Snapshot isolation | Auto, range-based |
| Vitess (MySQL proxy) | MySQL | Configurable | Manual, key-based |

### Scaling Decision Tree

```
Is DB overloaded?
├── Yes → Measure: CPU? Memory? Disk I/O? Connection count?
│
├── CPU high → Check slow queries → Optimize queries + indexes
│   └── Still high → Vertical scaling → Increase CPU
│       └── Still high → Read replicas (if read-heavy) → Caching
│           └── Still high → Sharding (if write-heavy)
│
├── Memory high → Increase buffer pool size
│   └── Still high → Vertical scaling → More RAM
│       └── Still high → Partition large tables
│
├── Disk I/O high → Optimize queries (fewer rows scanned)
│   └── Still high → Faster SSD → Read replicas
│       └── Still high → Sharding
│
├── Connections maxed → Connection pooling → Increase max connections
│   └── Still maxed → Read replicas → Sharding
│
└── Data too large → Archive old data → Partitioning
    └── Still large → Sharding
```

### Scaling Approaches Comparison

| Approach | Complexity | Cost | Read Scale | Write Scale | Consistency |
|----------|-----------|------|------------|-------------|-------------|
| Vertical | None | Medium | Limited | Limited | Full ACID |
| Read replicas | Low | Medium | High | None | Eventual |
| Caching | Low | Low | Very high | None | Stale |
| Connection pooling | Low | Free | Medium | Medium | Full ACID |
| Partitioning | Medium | Free | Medium | Medium | Full ACID |
| Sharding | High | High | High | High | Per-shard ACID |
| Distributed SQL | Medium | High | Very high | Very high | Serializable |

### Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Sharding prematurely | Unnecessary complexity, cross-shard issues | Try vertical + caching + replicas first |
| Bad shard key (hot spots) | Uneven load, one shard overwhelmed | Use hashed shard keys |
| No resharding plan | Hard to add/remove shards | Plan for rebalancing from day one |
| Cross-shard joins | Slow, complex workarounds | Design data model to avoid |
| Ignoring replication lag | Stale reads from replicas | Route critical reads to primary |
| No connection pooling | Connection storms, resource exhaustion | Always pool connections |
| Not partitioning large tables | Unmanageable table sizes | Partition tables over 100GB |
| Over-caching | Stale data, memory waste | Set TTL, invalidate on writes |

### Best Practices

- Start with vertical scaling and query optimization (cheapest, simplest)
- Add connection pooling (PgBouncer, ProxySQL) before hitting connection limits
- Add caching (Redis) when read patterns are repetitive
- Add read replicas when read:write ratio exceeds 5:1
- Partition tables over 100GB or 100M rows
- Shard only when write throughput exceeds a single node
- Choose shard keys with high cardinality and even distribution
- Plan resharding strategy before sharding
- Consider distributed SQL (CockroachDB, YugabyteDB) for new projects
- Monitor: query latency, connection count, replication lag, cache hit rate
- Automate: failover, resharding, schema changes

### Interview Questions

1. **Vertical vs horizontal scaling?** Vertical: bigger server. Limited by hardware, expensive at top end, simple. Horizontal: many servers. Complex, elastic, theoretically unlimited.
2. **How do read replicas work?** Primary handles writes, replicates to replicas. Reads go to replicas. Trade-off: replica lag (eventual consistency). Tools: PgBouncer, ProxySQL for routing.
3. **What makes a good shard key?** High cardinality (many unique values), even distribution across shards, aligned with common query patterns. Avoid monotonically increasing keys with range sharding.
4. **Challenges of cross-shard queries?** Joins across shards require scatter-gather. Aggregations need merge on coordinator. Transactions require 2PC. Better to design data to avoid cross-shard operations.
5. **When should you NOT shard?** When a single node handles the load. When queries need frequent cross-shard access. When team lacks operational maturity for distributed systems.
6. **How does Vitess differ from CockroachDB?** Vitess is a MySQL proxy that adds sharding on top of MySQL. It does not change the storage engine. CockroachDB is a distributed SQL database with built-in replication, sharding, and consensus (Raft).

### Practical Exercises

1. Set up read replicas: configure replication between two PostgreSQL instances, split read/write in application code.
2. Implement connection pooling with PgBouncer: configure pools, test concurrent connections.
3. Partition a table: create a time-series table with 1M rows, partition by month, query and compare performance.
4. Implement hash-based sharding in application code for user data (mock shards as separate files).
5. Set up Redis caching for a slow query, measure before/after latency.

### Mini Project: Multi-Tier Scaling Architecture

1. Application with 100K daily active users.
2. Single PostgreSQL instance: optimize queries, add indexes.
3. Add Redis caching for frequently accessed data (user profiles, product lists).
4. Add PgBouncer for connection pooling.
5. Add read replicas: one for analytics queries, one for API reads.
6. Partition the 10M-row orders table by month.
7. Implement application-level read/write splitting.
8. Load test each step, measure: queries/sec, latency P50/P99, connection count.

### Revision Notes

- Vertical first: simplest, cheapest at small scale
- Connection pooling: always do this first
- Caching: most cost-effective performance multiplier
- Read replicas: scale reads, no write benefit
- Partitioning: manage large tables, no horizontal scaling
- Sharding: horizontal scale, high complexity
- Shard key: choose carefully (permanent)
- Cross-shard operations: avoid, design around
- Distributed SQL: CockroachDB, YugabyteDB for new builds
- Monitor everything before scaling
- Automate failover, resharding, deployments

---

## Part 7 Summary

| Chapter | Focus | Key Topics |
|---------|-------|------------|
| 7.1 | SQL & Relational Theory | Codd rules, JOINs, subqueries, CTEs, window functions, normalization |
| 7.2 | MySQL | InnoDB, indexes, EXPLAIN, replication, my.cnf tuning |
| 7.3 | PostgreSQL | MVCC, JSONB, full-text search, BRIN/GIN/GiST, autovacuum |
| 7.4 | MongoDB | Document model, aggregation pipeline, replica sets, sharding |
| 7.5 | Redis | Data structures, persistence, Sentinel, Cluster, Lua scripting |
| 7.6 | Normalization | 1NF-5NF, functional dependencies, denormalization trade-offs |
| 7.7 | Indexing | B-tree internals, composite/covering/partial indexes, index types |
| 7.8 | Transactions & ACID | Isolation levels, MVCC, deadlocks, two-phase commit |
| 7.9 | Query Optimization | EXPLAIN ANALYZE, scan types, join strategies, profiling |
| 7.10 | Backup & Recovery | 3-2-1 rule, logical/physical, PITR, retention |
| 7.11 | Scaling | Vertical, read replicas, caching, partitioning, sharding |

**Core Principles:**
- Choose the right database for the job (relational for structured, document for flexible, in-memory for speed)
- Normalize to 3NF by default, denormalize only when proven needed
- Always use indexes on WHERE/JOIN/ORDER BY columns
- Understand MVCC and isolation levels of your database
- Enable slow query logging and use EXPLAIN ANALYZE
- Follow the 3-2-1 backup rule and test restores
- Scale vertically first, then caching + replicas, then sharding
- Monitor everything: latency, throughput, connections, disk, memory
