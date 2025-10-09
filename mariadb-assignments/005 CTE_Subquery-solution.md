# TDD Step-by-Step Walkthrough LAB: Subqueries and CTEs in Banking CRM (MariaDB-10-SQL)


***

## Overview

**Language:** Mariadb-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 3 (Intermediate)
**Domain:** Banking and Finance, CRM
**Objective:** Use Test-Driven Development (TDD) to implement a complex query with subqueries and CTEs (Common Table Expressions) to generate an enriched Customer Transaction Report. Students will follow a strict RED-GREEN-REFACTOR cycle in multiple passes for multi-layered business and testable requirements.

***

## Business Requirements

A retail bank wants comprehensive analysis of customer behavior and engagement for CRM (Customer Relationship Management).
The system must:

1. **BR001:** Report each customer's total transaction value, only including their 'ACTIVE' or 'SETTLED' transactions.
2. **BR002:** Highlight customers with the highest single transaction in the last quarter, showing customer info and transaction details.
3. **BR003:** Provide the top 3 regions by aggregate transaction count for CRM campaign prioritization.

***

## Testable Requirements

Derived TDD objectives:

**TR001:** For all customers, display (customer_id, name, SUM of qualifying transactions) using a subquery or CTE (must only sum qualifying statuses).

**TR002:** Identify customer(s) with max transaction value in the last quarter, joining with customer details (use nested subquery or CTE).

**TR003:** Return top 3 regions ranked by transaction counts (use CTE for intermediate aggregation).

***

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS banking_crm_lab;
USE banking_crm_lab;

-- Table 1: Customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    region VARCHAR(30),
    joined_date DATE
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Transactions
CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    txn_date DATE NOT NULL,
    txn_type ENUM('DEPOSIT', 'WITHDRAWAL', 'LOAN', 'TRANSFER'),
    txn_status ENUM('ACTIVE', 'SETTLED', 'FAILED', 'CANCELLED'),
    txn_amount DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```


### Test Data Insertion

```sql
-- Insert customer data
INSERT INTO customers (customer_id, customer_name, region, joined_date) VALUES
(1, 'Alice', 'East', '2022-01-10'),
(2, 'Bob', 'West', '2021-09-20'),
(3, 'Charlie', 'North', '2020-07-05'),
(4, 'Diana', 'East', '2023-02-13'),
(5, 'Evan', 'South', '2020-02-23');

-- Insert transactions (cover various statuses, dates, and customers)
INSERT INTO transactions (customer_id, txn_date, txn_type, txn_status, txn_amount) VALUES
(1, '2025-08-01', 'DEPOSIT', 'ACTIVE', 900.00),
(1, '2025-08-12', 'LOAN', 'SETTLED', 4500.00),
(1, '2025-06-11', 'TRANSFER', 'FAILED', 500.00),
(2, '2025-09-09', 'DEPOSIT', 'ACTIVE', 1200.00),
(2, '2025-07-20', 'LOAN', 'SETTLED', 3500.00),
(3, '2025-08-25', 'WITHDRAWAL', 'SETTLED', 1500.00),
(3, '2025-08-15', 'WITHDRAWAL', 'CANCELLED', 1850.00),
(4, '2025-09-16', 'DEPOSIT', 'ACTIVE', 650.00),
(4, '2025-09-26', 'LOAN', 'ACTIVE', 4900.00),
(4, '2025-07-18', 'TRANSFER', 'SETTLED', 700.00),
(5, '2025-06-22', 'DEPOSIT', 'ACTIVE', 900.00),
(5, '2025-08-29', 'TRANSFER', 'ACTIVE', 1100.00),
(5, '2025-09-27', 'DEPOSIT', 'FAILED', 975.00);
```


***

## TDD Implementation - Pass 1: TR001 (Customer Transaction Sum using CTE/subquery)

### RED Phase - Write Failing Test

```sql
-- TR001 RED: Fails initially (solution table not created)
CREATE TEMPORARY TABLE tr001_expected_results (
    customer_id INT,
    expected_total DECIMAL(12,2)
);

INSERT INTO tr001_expected_results VALUES
(1, 5400.00),  -- Alice (900 + 4500)
(2, 4700.00),  -- Bob (1200 + 3500)
(3, 1500.00),  -- Charlie (only settled)
(4, 6250.00),  -- Diana (650 + 4900 + 700)
(5, 2000.00);  -- Evan (900 + 1100)

-- Assertion: solution must match expected totals for each customer
SELECT
    'TR001_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM tr001_expected_results e
            LEFT JOIN tr001_solution s ON e.customer_id = s.customer_id
            WHERE e.expected_total != s.txn_total OR s.customer_id IS NULL
        ) THEN 'PASS'
        ELSE 'FAIL - Customer transaction totals do not match'
    END as result;
```

**Expected Result:** FAIL - `tr001_solution` does not exist.

### GREEN Phase - Minimal Working Solution

```sql
-- CTE to isolate sum of qualifying transactions (ACTIVE/SETTLED)
CREATE TEMPORARY TABLE tr001_solution AS
WITH qualifying_transactions AS (
    SELECT customer_id, txn_amount
    FROM transactions
    WHERE txn_status IN ('ACTIVE', 'SETTLED')
)
SELECT c.customer_id, SUM(q.txn_amount) AS txn_total
FROM customers c
LEFT JOIN qualifying_transactions q ON c.customer_id = q.customer_id
GROUP BY c.customer_id;

-- Run TR001 test again - should PASS now
SELECT
    'TR001_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM tr001_expected_results e
            LEFT JOIN tr001_solution s ON e.customer_id = s.customer_id
            WHERE e.expected_total != s.txn_total OR s.customer_id IS NULL
        ) THEN 'PASS'
        ELSE 'FAIL'
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Clean and Verify

```sql
DROP TEMPORARY TABLE IF EXISTS tr001_solution;
DROP TEMPORARY TABLE IF EXISTS tr001_expected_results;
```


***

## TDD Implementation - Pass 2: TR002 (Maximum Transaction in Last Quarter via Nested Subquery/CTE)

### RED Phase - Write Failing Test

```sql
-- Compute last fiscal quarter (July-September for this example; adjust dates for current)
CREATE TEMPORARY TABLE tr002_expected_results (
    customer_id INT,
    max_txn_amount DECIMAL(12,2),
    expected_name VARCHAR(100)
);

INSERT INTO tr002_expected_results VALUES
(4, 4900.00, 'Diana');

-- Assertion: solution must match (Diana, 4900)
SELECT
    'TR002_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1
            FROM tr002_expected_results e
            LEFT JOIN tr002_solution s ON e.customer_id = s.customer_id
            WHERE s.max_txn_amount != e.max_txn_amount OR s.customer_name != e.expected_name
        ) THEN 'PASS'
        ELSE 'FAIL - Highest transaction amount not matching'
    END as result;
```

**Expected Result:** FAIL - `tr002_solution` not present.

### GREEN Phase - Solution using CTE and Nested Subquery

```sql
-- Find highest transaction in last quarter per customer
CREATE TEMPORARY TABLE tr002_solution AS
WITH last_quarter_txns AS (
    SELECT t.customer_id, t.txn_amount, c.customer_name
    FROM transactions t
    INNER JOIN customers c ON c.customer_id = t.customer_id
    WHERE t.txn_date BETWEEN '2025-07-01' AND '2025-09-30'
        AND t.txn_status IN ('ACTIVE', 'SETTLED')
)
SELECT customer_id, customer_name, MAX(txn_amount) AS max_txn_amount
FROM last_quarter_txns
GROUP BY customer_id, customer_name
ORDER BY max_txn_amount DESC
LIMIT 1;

-- Run TR002 test again - should PASS now
SELECT
    'TR002_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1
            FROM tr002_expected_results e
            LEFT JOIN tr002_solution s ON e.customer_id = s.customer_id
            WHERE s.max_txn_amount != e.max_txn_amount OR s.customer_name != e.expected_name
        ) THEN 'PASS'
        ELSE 'FAIL'
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Verify and Clean

```sql
DROP TEMPORARY TABLE IF EXISTS tr002_solution;
DROP TEMPORARY TABLE IF EXISTS tr002_expected_results;
```


***

## TDD Implementation - Pass 3: TR003 (Top Regions by Transaction Count using CTE)

### RED Phase - Write Failing Test

```sql
CREATE TEMPORARY TABLE tr003_expected_results (
    region VARCHAR(30),
    txn_count INT
);

-- Based on test data
INSERT INTO tr003_expected_results VALUES
('East', 5), -- Alice + Diana: 3 + 2 qualifying
('South', 2),
('West', 2);

-- Assertion: solution must show correct top-3 regions by transaction count
SELECT
    'TR003_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1
            FROM tr003_expected_results e
            LEFT JOIN tr003_solution s ON e.region = s.region
            WHERE e.txn_count != s.txn_count OR s.region IS NULL
        ) THEN 'PASS'
        ELSE 'FAIL - Regional transaction counts do not match'
    END as result;
```

**Expected Result:** FAIL - `tr003_solution` does not exist.

### GREEN Phase - Working Solution

```sql
-- Use CTE to compute transaction counts by region
CREATE TEMPORARY TABLE tr003_solution AS
WITH txn_counts AS (
    SELECT c.region, COUNT(t.transaction_id) AS txn_count
    FROM customers c
    JOIN transactions t ON c.customer_id = t.customer_id
    WHERE t.txn_status IN ('ACTIVE', 'SETTLED')
    GROUP BY c.region
)
SELECT region, txn_count
FROM txn_counts
ORDER BY txn_count DESC
LIMIT 3;

-- Run TR003 test again - should PASS now
SELECT
    'TR003_TEST' as test_name,
    CASE
        WHEN NOT EXISTS (
            SELECT 1
            FROM tr003_expected_results e
            LEFT JOIN tr003_solution s ON e.region = s.region
            WHERE e.txn_count != s.txn_count OR s.region IS NULL
        ) THEN 'PASS'
        ELSE 'FAIL'
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Clean Test Tables

```sql
DROP TEMPORARY TABLE IF EXISTS tr003_solution;
DROP TEMPORARY TABLE IF EXISTS tr003_expected_results;
```


***

## Final Solution (Production Query Example)

```sql
-- Customer Transaction Sum
WITH qualifying_txns AS (
    SELECT customer_id, txn_amount
    FROM transactions
    WHERE txn_status IN ('ACTIVE', 'SETTLED')
)
SELECT c.customer_id, c.customer_name, SUM(q.txn_amount) AS txn_total
FROM customers c
LEFT JOIN qualifying_txns q ON c.customer_id = q.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY txn_total DESC;

-- Highest Single Transaction in Last Quarter
WITH last_quarter AS (
    SELECT t.customer_id, t.txn_amount, c.customer_name
    FROM transactions t
    JOIN customers c ON t.customer_id = c.customer_id
    WHERE t.txn_date BETWEEN '2025-07-01' AND '2025-09-30'
    AND t.txn_status IN ('ACTIVE', 'SETTLED')
)
SELECT customer_id, customer_name, MAX(txn_amount) AS max_txn_amount
FROM last_quarter
GROUP BY customer_id, customer_name
ORDER BY max_txn_amount DESC
LIMIT 1;

-- Top Regions by Transaction Count
WITH txn_by_region AS (
    SELECT c.region, COUNT(t.transaction_id) AS txn_count
    FROM customers c
    JOIN transactions t ON c.customer_id = t.customer_id
    WHERE t.txn_status IN ('ACTIVE', 'SETTLED')
    GROUP BY c.region
)
SELECT region, txn_count
FROM txn_by_region
ORDER BY txn_count DESC
LIMIT 3;
```


***

## Expected Output

### TR001 Example

| customer_id | customer_name | txn_total |
| :-- | :-- | :-- |
| 4 | Diana | 6250.00 |
| 2 | Bob | 4700.00 |
| 1 | Alice | 5400.00 |
| 5 | Evan | 2000.00 |
| 3 | Charlie | 1500.00 |

### TR002 Example

| customer_id | customer_name | max_txn_amount |
| :-- | :-- | :-- |
| 4 | Diana | 4900.00 |

### TR003 Example

| region | txn_count |
| :-- | :-- |
| East | 5 |
| West | 2 |
| South | 2 |


***

## Cleanup Script

```sql
-- Remove all test data and business tables
DROP TABLE IF EXISTS transactions;
DROP TABLE IF EXISTS customers;
DROP DATABASE IF EXISTS banking_crm_lab;

-- Verify cleanup (should return zero)
SHOW DATABASES LIKE 'banking_crm_lab';
```


***

## Key Learnings

- Every requirement enforced a strict TDD (RED-GREEN-REFACTOR) cycle
- CTEs (`WITH` clause) were used for readable and efficient composition of intermediate results, enabling complex filtering and multi-step analysis[^1][^2][^3]
- Subqueries and CTEs allow modular development of testable analytics in real-world banking CRM scenarios[^4][^5][^6][^7]
- This exercise is a precursor to Spark/Hadoop lineage tracking – each SQL block demonstrates explicit derivation of results
- All code is copy-paste ready for MariaDB SQL IDEs (10.2.2+ for CTEs)
- Tests are granular so students discover and fix failures step by step

***

**References:**

- [MariaDB official CTE documentation], [Tutorial], [Performance recommendations], [Subquery documentation][^2][^8][^3][^7]
<span style="display:none">[^10][^11][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21][^9]</span>

<div align="center">⁂</div>

[^1]: https://www.getgalaxy.io/learn/glossary/how-to-use-cte-in-mariadb

[^2]: https://www.mariadbtutorial.com/mariadb-basics/mariadb-cte/

[^3]: https://mariadb.com/docs/server/reference/sql-statements/data-manipulation/selecting-data/common-table-expressions

[^4]: https://www.geeksforgeeks.org/mariadb/mariadb-subqueries/

[^5]: https://www.getgalaxy.io/learn/glossary/how-to-use-subqueries-in-mariadb

[^6]: https://moldstud.com/articles/p-understanding-scalar-vs-table-subqueries-in-mariadb-a-comprehensive-guide

[^7]: https://mariadb.com/docs/server/reference/sql-statements/data-manipulation/selecting-data/joins-subqueries/subqueries

[^8]: https://moldstud.com/articles/p-how-to-optimize-subqueries-in-mariadb-for-faster-execution-performance-tips

[^9]: 002-Join-count-tradefinance.md

[^10]: https://www.dbvis.com/thetable/a-complete-guide-to-the-mysql-cte-mechanism/

[^11]: https://mariadb.org/wp-content/uploads/2017/05/Window-Functions-presentation-MariaDB-Foundation-NY-Developer-Meeting.pdf

[^12]: https://dev.mysql.com/doc/refman/8.4/en/with.html

[^13]: https://airbyte.com/data-engineering-resources/common-table-expression

[^14]: https://hightouch.com/sql-dictionary/sql-common-table-expression-cte

[^15]: https://mariadb.com/resources/blog/connect-by-is-dead-long-live-cte-in-mariadb-server-10-2/

[^16]: https://stackoverflow.com/questions/65317742/woes-using-update-with-a-cte-in-mysql-mariadb

[^17]: https://ceur-ws.org/Vol-1864/paper_6.pdf

[^18]: https://dev.mysql.com/doc/refman/8.1/en/exists-and-not-exists-subqueries.html

[^19]: https://www.academia.edu/41867081/MariaDB_and_MySQL_Common_Table_Expressions_and_Window_Functions_Revealed_Daniel_Bartholomew

[^20]: https://www.atlassian.com/data/sql/using-common-table-expressions

[^21]: https://www.linode.com/docs/guides/how-to-work-with-mysql-subqueries/

