# TDD Step-by-Step Walkthrough LAB: Banking Trade Finance LEFT JOIN Query

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 3 (Intermediate)
**Domain:** Banking Trade Finance
**Objective:** Create a SQL LEFT JOIN statement with 2 tables joined on 2 columns, with COUNT() aggregation using Test-Driven Development (TDD) approach

## Business Requirements

### Background

GlobalTrade Bank processes trade finance transactions including Letters of Credit (LC) and Documentary Collections. The bank needs to analyze transaction volumes by country and currency to support risk management and business development decisions.

### Core Business Requirements

1. **BR001**: Generate a report showing all countries where the bank has trade finance operations, including countries with no active transactions
2. **BR002**: Count the total number of active trade finance transactions per country
3. **BR003**: Display results grouped by both country code and currency code to support multi-currency analysis

## Testable Requirements

Based on business requirements, we define three testable requirements following TDD methodology :

### TR001: Basic Country-Transaction Join

**Given** country master data and transaction data exist
**When** querying all countries with their transaction counts
**Then** return all countries including those with zero transactions using LEFT JOIN

### TR002: Multi-Column Join Implementation

**Given** transactions are linked to countries by both country_code and currency_code
**When** joining tables on multiple columns
**Then** return accurate transaction counts per country-currency combination

### TR003: Aggregation with COUNT Function

**Given** joined data from countries and transactions
**When** applying COUNT aggregation
**Then** return the total number of active transactions per country-currency pair

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS trade_finance_lab;
USE trade_finance_lab;

-- Table 1: Country Master (Left table)
CREATE TABLE countries (
    country_code CHAR(3) NOT NULL,
    currency_code CHAR(3) NOT NULL,
    country_name VARCHAR(100) NOT NULL,
    region VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (country_code, currency_code)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Trade Finance Transactions (Right table)  
CREATE TABLE trade_transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    country_code CHAR(3) NOT NULL,
    currency_code CHAR(3) NOT NULL,
    transaction_type ENUM('LC', 'DC', 'BG') NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    status ENUM('ACTIVE', 'COMPLETED', 'CANCELLED') DEFAULT 'ACTIVE',
    transaction_date DATE NOT NULL,
    INDEX idx_country_currency (country_code, currency_code)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```


### Test Data Insertion

```sql
-- Insert country master data
INSERT INTO countries (country_code, currency_code, country_name, region) VALUES
('USA', 'USD', 'United States', 'North America'),
('USA', 'EUR', 'United States', 'North America'),
('GBR', 'GBP', 'United Kingdom', 'Europe'),
('GBR', 'USD', 'United Kingdom', 'Europe'), 
('DEU', 'EUR', 'Germany', 'Europe'),
('JPN', 'JPY', 'Japan', 'Asia'),
('SGP', 'SGD', 'Singapore', 'Asia'),
('SGP', 'USD', 'Singapore', 'Asia'),
('AUS', 'AUD', 'Australia', 'Oceania');

-- Insert transaction data (note: some countries have no transactions)
INSERT INTO trade_transactions (country_code, currency_code, transaction_type, amount, status, transaction_date) VALUES
('USA', 'USD', 'LC', 100000.00, 'ACTIVE', '2025-01-15'),
('USA', 'USD', 'DC', 75000.00, 'ACTIVE', '2025-02-01'), 
('USA', 'EUR', 'BG', 50000.00, 'ACTIVE', '2025-01-20'),
('GBR', 'GBP', 'LC', 80000.00, 'ACTIVE', '2025-01-10'),
('GBR', 'GBP', 'LC', 60000.00, 'COMPLETED', '2025-01-05'),
('DEU', 'EUR', 'DC', 45000.00, 'ACTIVE', '2025-02-10'),
('SGP', 'SGD', 'LC', 30000.00, 'ACTIVE', '2025-01-25'),
('SGP', 'USD', 'BG', 25000.00, 'CANCELLED', '2025-01-18');
```


## TDD Implementation - Pass 1: TR001 Basic LEFT JOIN

### RED Phase - Write Failing Test

```sql
-- TR001 RED: Create test that WILL FAIL initially
-- Test expects exactly 9 unique country-currency combinations

-- Expected test data verification
CREATE TEMPORARY TABLE tr001_expected_results (
    country_code CHAR(3),
    currency_code CHAR(3),
    expected_row_count INT DEFAULT 1
);

INSERT INTO tr001_expected_results VALUES
('USA', 'USD', 1), ('USA', 'EUR', 1), ('GBR', 'GBP', 1), 
('GBR', 'USD', 1), ('DEU', 'EUR', 1), ('JPN', 'JPY', 1), 
('SGP', 'SGD', 1), ('SGP', 'USD', 1), ('AUS', 'AUD', 1);

-- TEST TR001: This will FAIL because tr001_solution doesn't exist yet
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 9 rows, got ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr001_solution'), 0), ' (table missing)')
    END as result;
```

**Expected Result:** FAIL - table 'tr001_solution' doesn't exist

### GREEN Phase - Write Minimal Code to Pass

```sql
-- TR001 GREEN: Create minimal solution to make test pass
-- Use DISTINCT to avoid duplicates from multiple transactions per country-currency
CREATE TEMPORARY TABLE tr001_solution AS
SELECT DISTINCT c.country_code, 
                c.currency_code,
                c.country_name
FROM countries c
LEFT JOIN trade_transactions t ON c.country_code = t.country_code 
                               AND c.currency_code = t.currency_code;

-- Run TR001 test again - should PASS now
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 9 rows, got ', 
                   (SELECT COUNT(*) FROM tr001_solution))
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Verify and Clean

```sql
-- Verify TR001 solution works and clean up
DROP TEMPORARY TABLE IF EXISTS tr001_solution;
DROP TEMPORARY TABLE IF EXISTS tr001_expected_results;
```


## TDD Implementation - Pass 2: TR002 Multi-Column Join Verification

### RED Phase - Write Failing Test

```sql
-- TR002 RED: Test multi-column join accuracy
-- Expected: Some countries should have NULL transaction_id (no transactions)

CREATE TEMPORARY TABLE tr002_expected_nulls (
    country_code CHAR(3),
    currency_code CHAR(3),
    should_be_null BOOLEAN DEFAULT TRUE
);

-- Countries that should have NULL transaction_id (no transactions in test data)
INSERT INTO tr002_expected_nulls VALUES
('GBR', 'USD', TRUE), ('JPN', 'JPY', TRUE), ('AUS', 'AUD', TRUE);

-- TEST TR002: This will FAIL because tr002_solution doesn't exist yet  
SELECT 
    'TR002_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr002_expected_nulls) <= 
             (SELECT COUNT(*) FROM tr002_solution WHERE transaction_id IS NULL)
        THEN 'PASS'
        ELSE CONCAT('FAIL - Expected at least 3 NULL transaction_id rows, got ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr002_solution'), 0), ' (table missing)')
    END as result;
```

**Expected Result:** FAIL - table 'tr002_solution' doesn't exist

### GREEN Phase - Write Code to Pass

```sql
-- TR002 GREEN: Create solution showing transaction details with NULLs
-- This will show multiple rows for countries with multiple transactions
CREATE TEMPORARY TABLE tr002_solution AS
SELECT c.country_code,
       c.currency_code, 
       c.country_name,
       t.transaction_id,
       t.transaction_type,
       t.amount,
       t.status
FROM countries c
LEFT JOIN trade_transactions t ON c.country_code = t.country_code 
                               AND c.currency_code = t.currency_code;

-- Run TR002 test - should PASS now
SELECT 
    'TR002_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr002_expected_nulls) <= 
             (SELECT COUNT(*) FROM tr002_solution WHERE transaction_id IS NULL)
        THEN 'PASS'
        ELSE CONCAT('FAIL - Expected at least 3 NULL transaction_id rows, got ', 
                   (SELECT COUNT(*) FROM tr002_solution WHERE transaction_id IS NULL))
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Verify and Clean

```sql
-- Verify TR002 solution and clean up
DROP TEMPORARY TABLE IF EXISTS tr002_solution;
DROP TEMPORARY TABLE IF EXISTS tr002_expected_nulls;
```


## TDD Implementation - Pass 3: TR003 COUNT Aggregation

### RED Phase - Write Failing Test

```sql
-- TR003 RED: Test COUNT aggregation for active transactions
-- Expected results based on our test data

CREATE TEMPORARY TABLE tr003_expected_counts (
    country_code CHAR(3),
    currency_code CHAR(3),
    expected_active_count INT
);

-- Expected active transaction counts based on test data
INSERT INTO tr003_expected_counts VALUES
('USA', 'USD', 2), ('USA', 'EUR', 1), ('GBR', 'GBP', 1), ('GBR', 'USD', 0),
('DEU', 'EUR', 1), ('JPN', 'JPY', 0), ('SGP', 'SGD', 1), ('SGP', 'USD', 0), 
('AUS', 'AUD', 0);

-- TEST TR003: This will FAIL because tr003_solution doesn't exist yet
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN NOT EXISTS (
            SELECT 1 FROM tr003_expected_counts e
            LEFT JOIN tr003_solution s ON e.country_code = s.country_code 
                                       AND e.currency_code = s.currency_code
            WHERE s.country_code IS NULL 
               OR e.expected_active_count != s.active_transaction_count
        ) THEN 'PASS'
        ELSE CONCAT('FAIL - Count mismatch or missing table: ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr003_solution'), 0))
    END as result;
```

**Expected Result:** FAIL - table 'tr003_solution' doesn't exist

### GREEN Phase - Write Final Code

```sql
-- TR003 GREEN: Create final solution with COUNT aggregation
CREATE TEMPORARY TABLE tr003_solution AS
SELECT c.country_code,
       c.currency_code,
       c.country_name,
       c.region,
       COUNT(CASE WHEN t.status = 'ACTIVE' THEN t.transaction_id END) as active_transaction_count
FROM countries c
LEFT JOIN trade_transactions t ON c.country_code = t.country_code 
                               AND c.currency_code = t.currency_code
GROUP BY c.country_code, c.currency_code, c.country_name, c.region
ORDER BY c.region, c.country_name, c.currency_code;

-- Run TR003 test - should PASS now
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN NOT EXISTS (
            SELECT 1 FROM tr003_expected_counts e
            LEFT JOIN tr003_solution s ON e.country_code = s.country_code 
                                       AND e.currency_code = s.currency_code
            WHERE s.country_code IS NULL 
               OR e.expected_active_count != s.active_transaction_count
        ) THEN 'PASS'
        ELSE 'FAIL - Count mismatch found'
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Final Verification

```sql
-- Run comprehensive test suite
SELECT 'COMPREHENSIVE_TEST_SUITE' as test_phase;

-- Test 1: Row count verification
SELECT 
    'Row_Count_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution) = 9 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected 9 rows, got ', (SELECT COUNT(*) FROM tr003_solution))
    END as result;

-- Test 2: NULL handling verification  
SELECT 
    'NULL_Handling_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution WHERE active_transaction_count = 0) = 4 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected 4 zero-count rows, got ', 
                   (SELECT COUNT(*) FROM tr003_solution WHERE active_transaction_count = 0))
    END as result;

-- Test 3: Data accuracy verification
SELECT 
    'Data_Accuracy_Test' as test_name,
    CASE 
        WHEN (SELECT active_transaction_count FROM tr003_solution 
              WHERE country_code = 'USA' AND currency_code = 'USD') = 2 THEN 'PASS'
        ELSE 'FAIL - USA-USD should have 2 active transactions'
    END as result;

-- Test 4: Multi-transaction country verification
SELECT 
    'Multi_Transaction_Test' as test_name,
    CASE 
        WHEN (SELECT active_transaction_count FROM tr003_solution 
              WHERE country_code = 'GBR' AND currency_code = 'GBP') = 1 THEN 'PASS'
        ELSE 'FAIL - GBR-GBP should have 1 active transaction (1 ACTIVE, 1 COMPLETED)'
    END as result;

-- Clean up test tables
DROP TEMPORARY TABLE IF EXISTS tr003_solution;
DROP TEMPORARY TABLE IF EXISTS tr003_expected_counts;
```


## Final Solution

```sql
-- Final Production Query: Banking Trade Finance Analysis
-- LEFT JOIN with dual-column join condition and COUNT aggregation
SELECT c.country_code,
       c.currency_code,
       c.country_name,
       c.region,
       COUNT(CASE WHEN t.status = 'ACTIVE' THEN t.transaction_id END) as active_transaction_count
FROM countries c
LEFT JOIN trade_transactions t ON c.country_code = t.country_code 
                               AND c.currency_code = t.currency_code
GROUP BY c.country_code, c.currency_code, c.country_name, c.region
ORDER BY c.region, c.country_name, c.currency_code;
```


## Expected Output

| country_code | currency_code | country_name | region | active_transaction_count |
| :-- | :-- | :-- | :-- | :-- |
| AUS | AUD | Australia | Oceania | 0 |
| JPN | JPY | Japan | Asia | 0 |
| SGP | SGD | Singapore | Asia | 1 |
| SGP | USD | Singapore | Asia | 0 |
| DEU | EUR | Germany | Europe | 1 |
| GBR | GBP | United Kingdom | Europe | 1 |
| GBR | USD | United Kingdom | Europe | 0 |
| USA | EUR | United States | North America | 1 |
| USA | USD | United States | North America | 2 |

## Key Learning Points

1. **Proper TDD Cycle**: Each test initially fails (RED), then passes with minimal implementation (GREEN), then code is improved (REFACTOR)
2. **LEFT JOIN Behavior**: Returns all rows from the left table (countries) even when no matching rows exist in the right table (transactions)
3. **Multi-Column Joins**: JOIN conditions can include multiple column pairs using AND operator
4. **Duplicate Row Handling**: Without GROUP BY, LEFT JOIN creates one row per matching transaction; use DISTINCT or GROUP BY to get unique combinations
5. **COUNT with Conditions**: Using `COUNT(CASE WHEN condition THEN column END)` counts only rows meeting specific criteria
6. **Test Data Design**: Tests use specific expected values that will fail if implementation is incorrect

## Cleanup Script

```sql
-- Cleanup: Remove all test data and tables
DROP TABLE IF EXISTS trade_transactions;
DROP TABLE IF EXISTS countries;
DROP DATABASE IF EXISTS trade_finance_lab;

-- Verify cleanup
SHOW DATABASES LIKE 'trade_finance_lab';
-- Should return empty result set
```