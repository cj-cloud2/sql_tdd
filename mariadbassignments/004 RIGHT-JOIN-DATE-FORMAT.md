# TDD Step-by-Step Walkthrough LAB: Banking CBS RIGHT JOIN with DATE_FORMAT

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)  
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking CBS (Core Banking System)  
**Objective:** Create a SQL RIGHT JOIN statement with 2 tables joined on 1 column, with DATE_FORMAT() function using Test-Driven Development (TDD) approach

## Business Requirements

### Background

FirstNational Bank operates a Core Banking System (CBS) that manages customer accounts and transaction processing. The bank needs to generate monthly statements showing all transaction activities, including accounts with no transactions, formatted with readable date displays for customer-friendly reports.

### Core Business Requirements

1. **BR001**: Generate a comprehensive report showing all transactions with their associated account information
2. **BR002**: Include accounts that have no transactions to identify dormant accounts
3. **BR003**: Display transaction dates in a customer-friendly format (DD-MM-YYYY) instead of the standard ISO format

## Testable Requirements

Based on business requirements, we define three testable requirements following TDD methodology:

### TR001: Basic Account-Transaction RIGHT JOIN

**Given** account master data and transaction data exist  
**When** querying all transactions with their account information using RIGHT JOIN  
**Then** return all transactions including those where account information might be missing

### TR002: Single Column Join Implementation  

**Given** transactions are linked to accounts by account_number  
**When** joining tables on a single column (account_number)  
**Then** return accurate results with proper relationship mapping

### TR003: DATE_FORMAT Function Implementation

**Given** joined data contains transaction dates  
**When** applying DATE_FORMAT to display dates in DD-MM-YYYY format  
**Then** return formatted dates that are customer-readable instead of ISO format

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS banking_cbs_lab;
USE banking_cbs_lab;

-- Table 1: Account Master (Left table - may have missing info)
CREATE TABLE accounts (
    account_number VARCHAR(20) NOT NULL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT') NOT NULL,
    branch_code CHAR(4) NOT NULL,
    opening_date DATE NOT NULL,
    balance DECIMAL(15,2) DEFAULT 0.00,
    status ENUM('ACTIVE', 'DORMANT', 'CLOSED') DEFAULT 'ACTIVE',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Transaction Details (Right table - primary data source)
CREATE TABLE transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL,
    transaction_date DATE NOT NULL,
    transaction_type ENUM('CREDIT', 'DEBIT') NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    description VARCHAR(200),
    reference_number VARCHAR(50),
    processed_by VARCHAR(20),
    INDEX idx_account_number (account_number),
    INDEX idx_transaction_date (transaction_date)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```

### Test Data Insertion

```sql
-- Insert account master data (note: some accounts may be missing for some transactions)
INSERT INTO accounts (account_number, customer_name, account_type, branch_code, opening_date, balance, status) VALUES
('ACC001', 'John Smith', 'SAVINGS', 'BR01', '2024-01-15', 25000.00, 'ACTIVE'),
('ACC002', 'Mary Johnson', 'CURRENT', 'BR01', '2024-02-20', 15000.00, 'ACTIVE'),
('ACC003', 'Robert Davis', 'SAVINGS', 'BR02', '2024-01-10', 8000.00, 'DORMANT'),
('ACC005', 'Linda Wilson', 'FIXED_DEPOSIT', 'BR02', '2024-03-01', 50000.00, 'ACTIVE');

-- Insert transaction data (includes transactions for accounts that may not exist in accounts table)
INSERT INTO transactions (account_number, transaction_date, transaction_type, amount, description, reference_number, processed_by) VALUES
('ACC001', '2025-01-15', 'CREDIT', 5000.00, 'Salary Credit', 'TXN001', 'SYS001'),
('ACC001', '2025-02-01', 'DEBIT', 1500.00, 'ATM Withdrawal', 'TXN002', 'ATM01'),
('ACC002', '2025-01-20', 'CREDIT', 2000.00, 'Online Transfer', 'TXN003', 'SYS002'),
('ACC002', '2025-02-10', 'DEBIT', 500.00, 'Bill Payment', 'TXN004', 'SYS001'),
('ACC003', '2025-01-25', 'CREDIT', 1000.00, 'Interest Credit', 'TXN005', 'SYS003'),
('ACC004', '2025-02-05', 'CREDIT', 3000.00, 'Cheque Deposit', 'TXN006', 'BR01'),
('ACC004', '2025-02-15', 'DEBIT', 800.00, 'Card Payment', 'TXN007', 'POS01'),
('ACC006', '2025-01-30', 'DEBIT', 2500.00, 'Wire Transfer', 'TXN008', 'SYS004');
```

## TDD Implementation - Pass 1: TR001 Basic RIGHT JOIN

### RED Phase - Write Failing Test

```sql
-- TR001 RED: Create test that WILL FAIL initially
-- Test expects exactly 8 transactions (all transactions should appear)

-- Expected test data verification
CREATE TEMPORARY TABLE tr001_expected_results (
    transaction_id INT,
    expected_exists BOOLEAN DEFAULT TRUE
);

INSERT INTO tr001_expected_results VALUES
(1, TRUE), (2, TRUE), (3, TRUE), (4, TRUE), 
(5, TRUE), (6, TRUE), (7, TRUE), (8, TRUE);

-- TEST TR001: This will FAIL because tr001_solution doesn't exist yet
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 8 rows, got ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr001_solution'), 0), ' (table missing)')
    END as result;
```

**Expected Result:** FAIL - table 'tr001_solution' doesn't exist

### GREEN Phase - Write Minimal Code to Pass

```sql
-- TR001 GREEN: Create minimal solution to make test pass
-- RIGHT JOIN ensures all transactions appear, even if account info is missing
CREATE TEMPORARY TABLE tr001_solution AS
SELECT t.transaction_id,
       t.account_number,
       a.customer_name,
       t.transaction_type,
       t.amount
FROM accounts a
RIGHT JOIN transactions t ON a.account_number = t.account_number;

-- Run TR001 test again - should PASS now
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 8 rows, got ', 
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

## TDD Implementation - Pass 2: TR002 Single Column Join Verification

### RED Phase - Write Failing Test

```sql
-- TR002 RED: Test single column join accuracy
-- Expected: Some transactions should have NULL customer_name (accounts not in master)

CREATE TEMPORARY TABLE tr002_expected_nulls (
    account_number VARCHAR(20),
    should_be_null BOOLEAN DEFAULT TRUE
);

-- Accounts that should have NULL customer_name (not in accounts table)
INSERT INTO tr002_expected_nulls VALUES
('ACC004', TRUE), ('ACC006', TRUE);

-- TEST TR002: This will FAIL because tr002_solution doesn't exist yet  
SELECT 
    'TR002_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr002_expected_nulls) <= 
             (SELECT COUNT(*) FROM tr002_solution WHERE customer_name IS NULL)
        THEN 'PASS'
        ELSE CONCAT('FAIL - Expected at least 2 NULL customer_name rows, got ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr002_solution'), 0), ' (table missing)')
    END as result;
```

**Expected Result:** FAIL - table 'tr002_solution' doesn't exist

### GREEN Phase - Write Code to Pass

```sql
-- TR002 GREEN: Create solution showing single column join with NULLs
CREATE TEMPORARY TABLE tr002_solution AS
SELECT t.transaction_id,
       t.account_number,
       a.customer_name,
       a.account_type,
       a.status,
       t.transaction_type,
       t.amount,
       t.description
FROM accounts a
RIGHT JOIN transactions t ON a.account_number = t.account_number
ORDER BY t.transaction_date;

-- Run TR002 test - should PASS now
SELECT 
    'TR002_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr002_expected_nulls) <= 
             (SELECT COUNT(*) FROM tr002_solution WHERE customer_name IS NULL)
        THEN 'PASS'
        ELSE CONCAT('FAIL - Expected at least 2 NULL customer_name rows, got ', 
                   (SELECT COUNT(*) FROM tr002_solution WHERE customer_name IS NULL))
    END as result;
```

**Expected Result:** PASS

### REFACTOR Phase - Verify and Clean

```sql
-- Verify TR002 solution and clean up
DROP TEMPORARY TABLE IF EXISTS tr002_solution;
DROP TEMPORARY TABLE IF EXISTS tr002_expected_nulls;
```

## TDD Implementation - Pass 3: TR003 DATE_FORMAT Implementation 

### RED Phase - Write Failing Test

```sql
-- TR003 RED: Test DATE_FORMAT function - this should FAIL initially
-- Expected: Date format should be DD-MM-YYYY, not the default YYYY-MM-DD

CREATE TEMPORARY TABLE tr003_expected_formats (
    transaction_id INT,
    expected_date_format VARCHAR(10)
);

-- Expected formatted dates in DD-MM-YYYY format
INSERT INTO tr003_expected_formats VALUES
(1, '15-01-2025'), (2, '01-02-2025'), (3, '20-01-2025'), (4, '10-02-2025'),
(5, '25-01-2025'), (6, '05-02-2025'), (7, '15-02-2025'), (8, '30-01-2025');

-- TEST TR003: This will FAIL initially because DATE_FORMAT is not applied
-- Create a temporary solution WITHOUT DATE_FORMAT to make it fail
CREATE TEMPORARY TABLE tr003_failing_solution AS
SELECT t.transaction_id,
       t.account_number,
       a.customer_name,
       t.transaction_date,  -- This will be in YYYY-MM-DD format, causing test to FAIL
       t.transaction_type,
       t.amount,
       t.description
FROM accounts a
RIGHT JOIN transactions t ON a.account_number = t.account_number
ORDER BY t.transaction_date;

-- This test will FAIL because dates are not formatted
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN EXISTS (
            SELECT 1 FROM tr003_expected_formats e
            JOIN tr003_failing_solution s ON e.transaction_id = s.transaction_id
            WHERE e.expected_date_format = s.transaction_date
        ) THEN 'PASS'
        ELSE 'FAIL - DATE_FORMAT not applied, dates in wrong format'
    END as result;

-- Clean up the failing solution
DROP TEMPORARY TABLE tr003_failing_solution;
```

**Expected Result:** FAIL - DATE_FORMAT not applied, dates in wrong format

### GREEN Phase - Write Final Code

```sql
-- TR003 GREEN: Create final solution WITH DATE_FORMAT
CREATE TEMPORARY TABLE tr003_solution AS
SELECT t.transaction_id,
       t.account_number,
       a.customer_name,
       a.account_type,
       a.branch_code,
       DATE_FORMAT(t.transaction_date, '%d-%m-%Y') as formatted_transaction_date,
       t.transaction_type,
       t.amount,
       t.description,
       t.reference_number
FROM accounts a
RIGHT JOIN transactions t ON a.account_number = t.account_number
ORDER BY t.transaction_date;

-- Run TR003 test - should PASS now
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN NOT EXISTS (
            SELECT 1 FROM tr003_expected_formats e
            LEFT JOIN tr003_solution s ON e.transaction_id = s.transaction_id
            WHERE s.transaction_id IS NULL 
               OR e.expected_date_format != s.formatted_transaction_date
        ) THEN 'PASS'
        ELSE 'FAIL - DATE_FORMAT mismatch found'
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
        WHEN (SELECT COUNT(*) FROM tr003_solution) = 8 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected 8 rows, got ', (SELECT COUNT(*) FROM tr003_solution))
    END as result;

-- Test 2: NULL handling verification (RIGHT JOIN preserves all transactions)
SELECT 
    'NULL_Handling_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution WHERE customer_name IS NULL) >= 2 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected at least 2 NULL customer_name rows, got ', 
                   (SELECT COUNT(*) FROM tr003_solution WHERE customer_name IS NULL))
    END as result;

-- Test 3: DATE_FORMAT verification
SELECT 
    'DATE_FORMAT_Test' as test_name,
    CASE 
        WHEN (SELECT formatted_transaction_date FROM tr003_solution 
              WHERE transaction_id = 1) = '15-01-2025' THEN 'PASS'
        ELSE 'FAIL - DATE_FORMAT not working correctly'
    END as result;

-- Test 4: RIGHT JOIN verification (all transactions preserved)
SELECT 
    'RIGHT_JOIN_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution) = 
             (SELECT COUNT(*) FROM transactions) THEN 'PASS'
        ELSE 'FAIL - RIGHT JOIN should preserve all transactions'
    END as result;

-- Clean up test tables
DROP TEMPORARY TABLE IF EXISTS tr003_solution;
DROP TEMPORARY TABLE IF EXISTS tr003_expected_formats;
```

## Final Solution

```sql
-- Final Production Query: Banking CBS Transaction Report
-- RIGHT JOIN with single column join condition and DATE_FORMAT function
SELECT t.transaction_id,
       t.account_number,
       COALESCE(a.customer_name, 'Account Not Found') as customer_name,
       a.account_type,
       a.branch_code,
       DATE_FORMAT(t.transaction_date, '%d-%m-%Y') as transaction_date,
       t.transaction_type,
       t.amount,
       t.description,
       t.reference_number,
       t.processed_by
FROM accounts a
RIGHT JOIN transactions t ON a.account_number = t.account_number
ORDER BY t.transaction_date, t.transaction_id;
```

## Expected Output

| transaction_id | account_number | customer_name | account_type | branch_code | transaction_date | transaction_type | amount | description | reference_number | processed_by |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| 1 | ACC001 | John Smith | SAVINGS | BR01 | 15-01-2025 | CREDIT | 5000.00 | Salary Credit | TXN001 | SYS001 |
| 3 | ACC002 | Mary Johnson | CURRENT | BR01 | 20-01-2025 | CREDIT | 2000.00 | Online Transfer | TXN003 | SYS002 |
| 5 | ACC003 | Robert Davis | SAVINGS | BR02 | 25-01-2025 | CREDIT | 1000.00 | Interest Credit | TXN005 | SYS003 |
| 8 | ACC006 | Account Not Found | NULL | NULL | 30-01-2025 | DEBIT | 2500.00 | Wire Transfer | TXN008 | SYS004 |
| 2 | ACC001 | John Smith | SAVINGS | BR01 | 01-02-2025 | DEBIT | 1500.00 | ATM Withdrawal | TXN002 | ATM01 |
| 6 | ACC004 | Account Not Found | NULL | NULL | 05-02-2025 | CREDIT | 3000.00 | Cheque Deposit | TXN006 | BR01 |
| 4 | ACC002 | Mary Johnson | CURRENT | BR01 | 10-02-2025 | DEBIT | 500.00 | Bill Payment | TXN004 | SYS001 |
| 7 | ACC004 | Account Not Found | NULL | NULL | 15-02-2025 | DEBIT | 800.00 | Card Payment | TXN007 | POS01 |

## Key Learning Points

1. **Proper TDD Cycle**: Each test initially fails (RED), then passes with minimal implementation (GREEN), then code is improved (REFACTOR)
2. **RIGHT JOIN Behavior**: Returns all rows from the right table (transactions) even when no matching rows exist in the left table (accounts)
3. **Single Column Joins**: JOIN conditions can be simple with just one column match using the ON operator
4. **DATE_FORMAT Function**: Transforms standard ISO date format (YYYY-MM-DD) to custom format (DD-MM-YYYY) for user-friendly display
5. **Test-Driven DATE_FORMAT**: The RED phase demonstrates that without DATE_FORMAT, dates appear in default format, failing business requirements
6. **NULL Handling**: RIGHT JOIN produces NULL values for left table columns when no match exists
7. **Customer-Friendly Output**: Using COALESCE to replace NULL customer names with meaningful messages

## Cleanup Script

```sql
-- Cleanup: Remove all test data and tables
DROP TABLE IF EXISTS transactions;
DROP TABLE IF EXISTS accounts;
DROP DATABASE IF EXISTS banking_cbs_lab;

-- Verify cleanup
SHOW DATABASES LIKE 'banking_cbs_lab';
-- Should return empty result set
```

