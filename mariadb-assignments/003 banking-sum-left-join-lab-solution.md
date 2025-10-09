# TDD Step-by-Step Walkthrough LAB: Banking Account Balance LEFT JOIN Query

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)  
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking CBS (Core Banking System)  
**Objective:** Create a SQL LEFT JOIN statement with 2 tables joined on 1 column, with SUM() aggregation using Test-Driven Development (TDD) approach

## Business Requirements

### Background

Metropolitan Bank operates a core banking system that manages customer accounts and their transaction history. The bank needs to generate comprehensive account balance reports that include all customer accounts, even those with no transaction history, to ensure complete visibility of the bank's account portfolio for regulatory compliance and risk management.

### Core Business Requirements

1. **BR001**: Generate a report showing all customer accounts in the system, including accounts with zero transaction activity
2. **BR002**: Calculate the current balance for each account by summing all transaction amounts
3. **BR003**: Display account information with calculated balances for portfolio analysis and regulatory reporting

## Testable Requirements

Based on business requirements, we define three testable requirements following TDD methodology:

### TR001: Basic Account-Transaction JOIN

**Given** customer account data and transaction data exist  
**When** querying all accounts with their transaction information  
**Then** return all accounts including those with zero transactions using LEFT JOIN

### TR002: Single Column Join Implementation

**Given** transactions are linked to accounts by account_number  
**When** joining tables on the account_number column  
**Then** return accurate transaction data for each account including NULL values for accounts without transactions

### TR003: Balance Calculation with SUM Function

**Given** joined data from accounts and transactions  
**When** applying SUM aggregation to transaction amounts  
**Then** return the calculated account balance for each account (0 for accounts with no transactions)

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS banking_cbs_lab;
USE banking_cbs_lab;

-- Table 1: Customer Accounts (Left table)
CREATE TABLE customer_accounts (
    account_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT') NOT NULL,
    branch_code VARCHAR(10) NOT NULL,
    opening_date DATE NOT NULL,
    status ENUM('ACTIVE', 'INACTIVE', 'CLOSED') DEFAULT 'ACTIVE',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_account_number (account_number)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Account Transactions (Right table)
CREATE TABLE account_transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL,
    transaction_type ENUM('CREDIT', 'DEBIT') NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    transaction_date DATE NOT NULL,
    description VARCHAR(200),
    reference_number VARCHAR(50),
    status ENUM('COMPLETED', 'PENDING', 'FAILED') DEFAULT 'COMPLETED',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_account_number (account_number),
    INDEX idx_transaction_date (transaction_date)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```

### Test Data Insertion

```sql
-- Insert customer account data
INSERT INTO customer_accounts (account_number, customer_name, account_type, branch_code, opening_date, status) VALUES
('ACC001', 'John Smith', 'SAVINGS', 'BR001', '2024-01-15', 'ACTIVE'),
('ACC002', 'Sarah Johnson', 'CURRENT', 'BR001', '2024-02-01', 'ACTIVE'),
('ACC003', 'Michael Brown', 'SAVINGS', 'BR002', '2024-01-20', 'ACTIVE'),
('ACC004', 'Emily Davis', 'CURRENT', 'BR002', '2024-02-15', 'ACTIVE'),
('ACC005', 'Robert Wilson', 'FIXED_DEPOSIT', 'BR001', '2024-03-01', 'ACTIVE'),
('ACC006', 'Lisa Anderson', 'SAVINGS', 'BR003', '2024-03-15', 'INACTIVE'),
('ACC007', 'David Martinez', 'CURRENT', 'BR003', '2024-04-01', 'ACTIVE');

-- Insert transaction data (note: some accounts have no transactions)
INSERT INTO account_transactions (account_number, transaction_type, amount, transaction_date, description, reference_number, status) VALUES
-- ACC001 transactions
('ACC001', 'CREDIT', 5000.00, '2024-01-16', 'Initial Deposit', 'REF001', 'COMPLETED'),
('ACC001', 'DEBIT', 200.00, '2024-01-20', 'ATM Withdrawal', 'REF002', 'COMPLETED'),
('ACC001', 'CREDIT', 1500.00, '2024-02-01', 'Salary Credit', 'REF003', 'COMPLETED'),
-- ACC002 transactions  
('ACC002', 'CREDIT', 10000.00, '2024-02-02', 'Initial Deposit', 'REF004', 'COMPLETED'),
('ACC002', 'DEBIT', 2500.00, '2024-02-10', 'Business Payment', 'REF005', 'COMPLETED'),
-- ACC003 transactions
('ACC003', 'CREDIT', 3000.00, '2024-01-21', 'Initial Deposit', 'REF006', 'COMPLETED'),
('ACC003', 'DEBIT', 500.00, '2024-01-25', 'Online Transfer', 'REF007', 'COMPLETED'),
('ACC003', 'DEBIT', 100.00, '2024-02-05', 'Service Charge', 'REF008', 'COMPLETED'),
-- ACC004 has one transaction
('ACC004', 'CREDIT', 8000.00, '2024-02-16', 'Initial Deposit', 'REF009', 'COMPLETED');
-- ACC005, ACC006, ACC007 have no transactions
```

## TDD Implementation - Pass 1: TR001 Basic LEFT JOIN

### TR001: Basic Account-Transaction JOIN

**Given** customer account data and transaction data exist  
**When** querying all accounts with their transaction information  
**Then** return all accounts including those with zero transactions using LEFT JOIN

### RED Phase - Write Failing Test

```sql
-- TR001 RED: Create test that WILL FAIL initially
-- Test expects exactly 7 unique accounts (all accounts from customer_accounts table)

-- Expected test data verification
CREATE TEMPORARY TABLE tr001_expected_results (
    account_number VARCHAR(20),
    expected_row_count INT DEFAULT 1
);

INSERT INTO tr001_expected_results VALUES
('ACC001', 1), ('ACC002', 1), ('ACC003', 1), ('ACC004', 1), 
('ACC005', 1), ('ACC006', 1), ('ACC007', 1);

-- TEST TR001: This will FAIL because tr001_solution doesn't exist yet
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 7 rows, got ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr001_solution'), 0), ' (table missing)')
    END as result;
```

**Expected Result:** FAIL - table 'tr001_solution' doesn't exist

### GREEN Phase - Write Minimal Code to Pass

```sql
-- TR001 GREEN: Create minimal solution to make test pass
-- Use DISTINCT to avoid duplicates from multiple transactions per account
CREATE TEMPORARY TABLE tr001_solution AS
SELECT DISTINCT ca.account_number,
                ca.customer_name,
                ca.account_type,
                ca.branch_code
FROM customer_accounts ca
LEFT JOIN account_transactions at ON ca.account_number = at.account_number;

-- Run TR001 test again - should PASS now
SELECT 
    'TR001_TEST' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr001_expected_results) = 
             (SELECT COUNT(*) FROM tr001_solution) 
        THEN 'PASS' 
        ELSE CONCAT('FAIL - Expected 7 rows, got ', 
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

### TR002: Single Column Join Implementation

**Given** transactions are linked to accounts by account_number  
**When** joining tables on the account_number column  
**Then** return accurate transaction data for each account including NULL values for accounts without transactions

### RED Phase - Write Failing Test

```sql
-- TR002 RED: Test single-column join accuracy 
-- Expected: Some accounts should have NULL transaction_id (no transactions)

CREATE TEMPORARY TABLE tr002_expected_nulls (
    account_number VARCHAR(20),
    should_be_null BOOLEAN DEFAULT TRUE
);

-- Accounts that should have NULL transaction_id (no transactions in test data)
INSERT INTO tr002_expected_nulls VALUES
('ACC005', TRUE), ('ACC006', TRUE), ('ACC007', TRUE);

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
-- This will show multiple rows for accounts with multiple transactions
CREATE TEMPORARY TABLE tr002_solution AS
SELECT ca.account_number,
       ca.customer_name,
       ca.account_type,
       ca.branch_code,
       at.transaction_id,
       at.transaction_type,
       at.amount,
       at.transaction_date,
       at.description
FROM customer_accounts ca
LEFT JOIN account_transactions at ON ca.account_number = at.account_number;

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

## TDD Implementation - Pass 3: TR003 SUM Aggregation

### TR003: Balance Calculation with SUM Function

**Given** joined data from accounts and transactions  
**When** applying SUM aggregation to transaction amounts  
**Then** return the calculated account balance for each account (0 for accounts with no transactions)


### RED Phase - Write Failing Test

```sql
-- TR003 RED: Test SUM aggregation for account balances
-- Expected results based on our test data

CREATE TEMPORARY TABLE tr003_expected_balances (
    account_number VARCHAR(20),
    expected_balance DECIMAL(15,2)
);

-- Expected account balances based on test data calculations
-- ACC001: 5000.00 - 200.00 + 1500.00 = 6300.00
-- ACC002: 10000.00 - 2500.00 = 7500.00  
-- ACC003: 3000.00 - 500.00 - 100.00 = 2400.00
-- ACC004: 8000.00 = 8000.00
-- ACC005, ACC006, ACC007: 0.00 (no transactions)
INSERT INTO tr003_expected_balances VALUES
('ACC001', 6300.00), ('ACC002', 7500.00), ('ACC003', 2400.00), ('ACC004', 8000.00),
('ACC005', 0.00), ('ACC006', 0.00), ('ACC007', 0.00);

-- TEST TR003: This will FAIL because tr003_solution doesn't exist yet
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN NOT EXISTS (
            SELECT 1 FROM tr003_expected_balances e
            LEFT JOIN tr003_solution s ON e.account_number = s.account_number
            WHERE s.account_number IS NULL 
               OR ABS(e.expected_balance - s.account_balance) > 0.01
        ) THEN 'PASS'
        ELSE CONCAT('FAIL - Balance mismatch or missing table: ', 
                   COALESCE((SELECT COUNT(*) FROM information_schema.tables 
                            WHERE table_schema = DATABASE() AND table_name = 'tr003_solution'), 0))
    END as result;
```

**Expected Result:** FAIL - table 'tr003_solution' doesn't exist

### GREEN Phase - Write Final Code

```sql
-- TR003 GREEN: Create final solution with SUM aggregation
-- Calculate account balance using CASE to handle CREDIT/DEBIT properly
CREATE TEMPORARY TABLE tr003_solution AS
SELECT ca.account_number,
       ca.customer_name,
       ca.account_type,
       ca.branch_code,
       ca.status,
       COALESCE(
           SUM(CASE 
               WHEN at.transaction_type = 'CREDIT' THEN at.amount
               WHEN at.transaction_type = 'DEBIT' THEN -at.amount
               ELSE 0
           END), 0
       ) as account_balance
FROM customer_accounts ca
LEFT JOIN account_transactions at ON ca.account_number = at.account_number
                                  AND at.status = 'COMPLETED'
GROUP BY ca.account_number, ca.customer_name, ca.account_type, ca.branch_code, ca.status
ORDER BY ca.account_number;

-- Run TR003 test - should PASS now
SELECT 
    'TR003_TEST' as test_name,
    CASE 
        WHEN NOT EXISTS (
            SELECT 1 FROM tr003_expected_balances e
            LEFT JOIN tr003_solution s ON e.account_number = s.account_number
            WHERE s.account_number IS NULL 
               OR ABS(e.expected_balance - s.account_balance) > 0.01
        ) THEN 'PASS'
        ELSE 'FAIL - Balance mismatch found'
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
        WHEN (SELECT COUNT(*) FROM tr003_solution) = 7 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected 7 rows, got ', (SELECT COUNT(*) FROM tr003_solution))
    END as result;

-- Test 2: Zero balance verification  
SELECT 
    'Zero_Balance_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution WHERE account_balance = 0.00) = 3 THEN 'PASS'
        ELSE CONCAT('FAIL - Expected 3 zero-balance accounts, got ', 
                   (SELECT COUNT(*) FROM tr003_solution WHERE account_balance = 0.00))
    END as result;

-- Test 3: Specific balance verification
SELECT 
    'Balance_Accuracy_Test' as test_name,
    CASE 
        WHEN (SELECT account_balance FROM tr003_solution 
              WHERE account_number = 'ACC001') = 6300.00 THEN 'PASS'
        ELSE 'FAIL - ACC001 should have balance 6300.00'
    END as result;

-- Test 4: Negative balance handling verification
SELECT 
    'Negative_Balance_Test' as test_name,
    CASE 
        WHEN (SELECT COUNT(*) FROM tr003_solution WHERE account_balance < 0) = 0 THEN 'PASS'
        ELSE 'FAIL - No accounts should have negative balance with current test data'
    END as result;

-- Clean up test tables
DROP TEMPORARY TABLE IF EXISTS tr003_solution;
DROP TEMPORARY TABLE IF EXISTS tr003_expected_balances;
```

## Final Solution

```sql
-- Final Production Query: Banking Account Balance Report
-- LEFT JOIN with single-column join condition and SUM aggregation
SELECT ca.account_number,
       ca.customer_name,
       ca.account_type,
       ca.branch_code,
       ca.status,
       COALESCE(
           SUM(CASE 
               WHEN at.transaction_type = 'CREDIT' THEN at.amount
               WHEN at.transaction_type = 'DEBIT' THEN -at.amount
               ELSE 0
           END), 0
       ) as account_balance
FROM customer_accounts ca
LEFT JOIN account_transactions at ON ca.account_number = at.account_number
                                  AND at.status = 'COMPLETED'
GROUP BY ca.account_number, ca.customer_name, ca.account_type, ca.branch_code, ca.status
ORDER BY ca.account_number;
```

## Expected Output

| account_number | customer_name | account_type | branch_code | status | account_balance |
|:--|:--|:--|:--|:--|:--|
| ACC001 | John Smith | SAVINGS | BR001 | ACTIVE | 6300.00 |
| ACC002 | Sarah Johnson | CURRENT | BR001 | ACTIVE | 7500.00 |
| ACC003 | Michael Brown | SAVINGS | BR002 | ACTIVE | 2400.00 |
| ACC004 | Emily Davis | CURRENT | BR002 | ACTIVE | 8000.00 |
| ACC005 | Robert Wilson | FIXED_DEPOSIT | BR001 | ACTIVE | 0.00 |
| ACC006 | Lisa Anderson | SAVINGS | BR003 | INACTIVE | 0.00 |
| ACC007 | David Martinez | CURRENT | BR003 | ACTIVE | 0.00 |

## Key Learning Points

1. **Proper TDD Cycle**: Each test initially fails (RED), then passes with minimal implementation (GREEN), then code is improved (REFACTOR)
2. **LEFT JOIN Behavior**: Returns all rows from the left table (customer_accounts) even when no matching rows exist in the right table (account_transactions)
3. **Single Column Join**: JOIN condition uses only one column pair (account_number) for matching records
4. **SUM with CASE Statement**: Using `SUM(CASE WHEN condition THEN value ELSE 0 END)` for conditional aggregation
5. **COALESCE for NULL Handling**: `COALESCE()` converts NULL results to 0 for accounts with no transactions
6. **Banking Logic**: CREDIT transactions add to balance, DEBIT transactions subtract from balance
7. **GROUP BY Requirement**: When using aggregate functions like SUM(), non-aggregated columns must be in GROUP BY clause

## Cleanup Script

```sql
-- Cleanup: Remove all test data and tables
DROP TABLE IF EXISTS account_transactions;
DROP TABLE IF EXISTS customer_accounts;
DROP DATABASE IF EXISTS banking_cbs_lab;

-- Verify cleanup
SHOW DATABASES LIKE 'banking_cbs_lab';
-- Should return empty result set
```
