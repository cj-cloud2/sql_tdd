# TDD Step-by-Step Walkthrough LAB: ATM Account Report User-Defined Function

---

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 3
**Domain:** Banking ATM
**Objective:** Create a user-defined function (UDF) to generate an account summary report using Test-Driven Development (TDD) approach.

---

## Business Requirements

### Background

A bank ATM system allows users to view account information and transaction summaries at any time. For audit and customer service, a concise account summary report must be available for each account, including balance and basic statistics about recent activity.

### Core Business Requirements

1. **BR001:** Users must be able to retrieve the current balance for any account at any ATM.
2. **BR002:** Users must be able to see the number of transactions (deposits and withdrawals) for a selected account over a given period.
3. **BR003:** The customer account summary must show, for the given account and period, the opening balance (before period), closing balance (after period), and total debits/credits.

---

## Testable Requirements

- **TR001:** The system provides a function that, given an account ID, returns the current balance (as maintained in the database).
- **TR002:** The function, when given an account ID and date range, returns the total count of posted transactions in that period.
- **TR003:** The function, when given an account ID and date range, returns a JSON summary: opening balance, closing balance, sum of credits and sum of debits for that period.

---

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS atm_lab;
USE atm_lab;

-- Create accounts table
CREATE TABLE accounts (
    account_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    account_number CHAR(12) NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT') NOT NULL,
    balance DECIMAL(15,2) NOT NULL DEFAULT 0.0,
    is_active BOOLEAN DEFAULT TRUE,
    opened_date DATE,
    created_date DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- Create account_transactions table
CREATE TABLE account_transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    account_id INT NOT NULL,
    txn_type ENUM('DEPOSIT','WITHDRAWAL') NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    txn_time DATETIME NOT NULL,
    FOREIGN KEY(account_id) REFERENCES accounts(account_id)
) ENGINE=InnoDB;
```

### Test Data Insertion

```sql
-- Insert accounts
INSERT INTO accounts (customer_id, account_number, account_type, balance, opened_date) VALUES
(1, '123000000001', 'SAVINGS', 1500.00, '2025-01-01'),
(2, '123000000002', 'CURRENT', 250.00, '2025-02-01'),
(3, '123000000003', 'SAVINGS', 3000.00, '2025-03-01');

-- Insert transactions
INSERT INTO account_transactions (account_id, txn_type, amount, txn_time) VALUES
(1, 'DEPOSIT', 1000.00, '2025-05-01 10:00:00'),
(1, 'WITHDRAWAL', 100.00, '2025-05-02 09:30:00'),
(1, 'DEPOSIT', 600.00, '2025-06-01 12:45:00'),
(2, 'DEPOSIT', 250.00, '2025-05-10 13:00:00'),
(3, 'DEPOSIT', 2000.00, '2025-04-01 15:00:00'),
(3, 'WITHDRAWAL', 700.00, '2025-06-11 18:10:00');
```

---

## TDD Implementation - Stepwise Walkthrough

### Pass 1: TR001 - Account Balance

#### RED Phase: Create Placeholder Function and Failing Test

```sql
-- Create empty function first
DELIMITER //
CREATE FUNCTION get_account_balance(p_account_id INT) RETURNS DECIMAL(15,2)
DETERMINISTIC
BEGIN
    RETURN 0.0; -- Placeholder
END//
DELIMITER ;

-- Test procedure for RED phase (expect failure, actual != db value)
DELIMITER //
CREATE PROCEDURE test_get_account_balance_red()
BEGIN
    DECLARE expected DECIMAL(15,2);
    DECLARE actual DECIMAL(15,2);
    SET expected = (SELECT balance FROM accounts WHERE account_id = 1);
    SET actual = get_account_balance(1);
    IF actual <> expected THEN
        SELECT 'FAIL' AS result, actual, expected;
    ELSE
        SELECT 'PASS' AS result;
    END IF;
END//
DELIMITER ;
-- Run this: CALL test_get_account_balance_red();
```

#### GREEN Phase: Minimal Implementation to Pass

```sql
-- Redefine function to return correct balance
DROP FUNCTION IF EXISTS get_account_balance;
DELIMITER //
CREATE FUNCTION get_account_balance(p_account_id INT) RETURNS DECIMAL(15,2)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE cur_balance DECIMAL(15,2);
    SELECT balance INTO cur_balance FROM accounts WHERE account_id = p_account_id;
    RETURN IFNULL(cur_balance, 0.0);
END//
DELIMITER ;
-- Run test again: CALL test_get_account_balance_red(); (should PASS now)
```

#### REFACTOR Phase: Cleanup

```sql
DROP PROCEDURE IF EXISTS test_get_account_balance_red;
```

---

### Pass 2: TR002 - Transaction Count for Period

#### RED Phase: New Test Procedure (Fails: function does not exist)

```sql
-- New function placeholder
DELIMITER //
CREATE FUNCTION get_txn_count_for_period(p_account_id INT, p_start DATETIME, p_end DATETIME) RETURNS INT
DETERMINISTIC
BEGIN
    RETURN -1; -- Placeholder
END//
DELIMITER ;

-- Test procedure
DELIMITER //
CREATE PROCEDURE test_get_txn_count_red()
BEGIN
    DECLARE expected INT;
    DECLARE actual INT;
    SET expected = (SELECT COUNT(*) FROM account_transactions WHERE account_id=1 AND txn_time BETWEEN '2025-05-01' AND '2025-07-01');
    SET actual = get_txn_count_for_period(1, '2025-05-01', '2025-07-01');
    IF actual <> expected THEN
        SELECT 'FAIL' AS result, actual, expected;
    ELSE
        SELECT 'PASS' AS result;
    END IF;
END//
DELIMITER ;
-- Run: CALL test_get_txn_count_red();
```

#### GREEN Phase: Make Test Pass

```sql
DROP FUNCTION IF EXISTS get_txn_count_for_period;
DELIMITER //
CREATE FUNCTION get_txn_count_for_period(p_account_id INT, p_start DATETIME, p_end DATETIME) RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE c INT;
    SELECT COUNT(*) INTO c FROM account_transactions
      WHERE account_id=p_account_id AND txn_time BETWEEN p_start AND p_end;
    RETURN c;
END//
DELIMITER ;
-- Run test again: CALL test_get_txn_count_red(); (should PASS)
```

#### REFACTOR Phase: Cleanup

```sql
DROP PROCEDURE IF EXISTS test_get_txn_count_red;
```

---

### Pass 3: TR003 - JSON Account Summary

#### RED Phase: Add Test (returns dummy string at first, fails)

```sql
-- Create placeholder
DELIMITER //
CREATE FUNCTION get_account_summary_report(
  p_account_id INT, p_start DATETIME, p_end DATETIME
) RETURNS TEXT
DETERMINISTIC
BEGIN
    RETURN '{}';
END//
DELIMITER ;

-- Example manual verification for RED: (Expected != actual)
SELECT get_account_summary_report(1, '2025-05-01', '2025-07-01') AS report_json;
-- Expect: opening balance as of 2025-05-01, closing as of 2025-07-01, total credits, total debits
```

#### GREEN Phase: Implement Correct Function

```sql
DROP FUNCTION IF EXISTS get_account_summary_report;
DELIMITER //
CREATE FUNCTION get_account_summary_report(
  p_account_id INT, p_start DATETIME, p_end DATETIME
) RETURNS TEXT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE opening DECIMAL(15,2);
    DECLARE closing DECIMAL(15,2);
    DECLARE credits DECIMAL(15,2);
    DECLARE debits DECIMAL(15,2);

    SELECT IFNULL(balance,0) INTO opening FROM accounts WHERE account_id = p_account_id;
    SELECT IFNULL(balance,0) INTO closing FROM accounts WHERE account_id = p_account_id;
    SELECT IFNULL(SUM(amount),0) INTO credits FROM account_transactions
        WHERE account_id=p_account_id AND txn_type='DEPOSIT' AND txn_time BETWEEN p_start AND p_end;
    SELECT IFNULL(SUM(amount),0) INTO debits FROM account_transactions
        WHERE account_id=p_account_id AND txn_type='WITHDRAWAL' AND txn_time BETWEEN p_start AND p_end;

    RETURN CONCAT('{',
      '"opening_balance":', opening, ',',
      '"closing_balance":', closing, ',',
      '"credits":', credits, ',',
      '"debits":', debits, '}');
END//
DELIMITER ;
-- SELECT get_account_summary_report(1, '2025-05-01', '2025-07-01');
```

#### REFACTOR Phase: Optional tweaks, verify output and cleanup

---

## Cleanup Script

```sql
-- Clean up all test data and schema
DROP TABLE IF EXISTS account_transactions;
DROP TABLE IF EXISTS accounts;
DROP DATABASE IF EXISTS atm_lab;
-- VERIFY: SHOW DATABASES LIKE 'atm_lab'; -- should return nothing
```
