# TDD Step-by-Step Challenge LAB: ATM Account Report User-Defined Function

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

## TDD Implementation - Challenge Tasks

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
    -- Task 1.1: Write a SQL query to get the expected balance from accounts table where account_id = 1
    -- Your query description: [Retrieve the balance column value from the accounts table for account with ID 1]
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
    -- Task 1.2: Write a SQL query to select balance into cur_balance variable from accounts table 
    -- where account_id matches the input parameter p_account_id
    -- Your query description: [Select the balance value from accounts table where account_id equals the input parameter and store it in cur_balance variable]
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
    -- Task 2.1: Write a SQL query to count all transactions from account_transactions table 
    -- where account_id=1 and transaction time is between '2025-05-01' and '2025-07-01'
    -- Your query description: [Count the number of records in account_transactions table where account_id equals 1 and txn_time falls between the specified date range]
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
    -- Task 2.2: Write a SQL query to count transactions from account_transactions table 
    -- where account_id matches p_account_id and txn_time is between p_start and p_end parameters
    -- Your query description: [Count all transaction records from account_transactions table where account_id equals the input parameter and txn_time falls within the specified date range parameters, then store the result in variable c]
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

    -- Task 3.1: Write a SQL query to get the current balance from accounts table for opening balance
    -- Your query description: [Select balance value from accounts table where account_id matches the input parameter and store it in opening variable]

    -- Task 3.2: Write a SQL query to get the current balance from accounts table for closing balance
    -- Your query description: [Select balance value from accounts table where account_id matches the input parameter and store it in closing variable]

    -- Task 3.3: Write a SQL query to calculate total credits (deposits) within the date range
    -- Your query description: [Sum all amount values from account_transactions table where account_id matches parameter, txn_type is 'DEPOSIT', and txn_time falls within the date range, then store result in credits variable]

    -- Task 3.4: Write a SQL query to calculate total debits (withdrawals) within the date range  
    -- Your query description: [Sum all amount values from account_transactions table where account_id matches parameter, txn_type is 'WITHDRAWAL', and txn_time falls within the date range, then store result in debits variable]

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

## Challenge Instructions

### Your Tasks

1. **Complete Pass 1**: Replace the query descriptions with actual SQL SELECT statements in the designated locations for Tasks 1.1 and 1.2.

2. **Complete Pass 2**: Replace the query descriptions with actual SQL SELECT statements in the designated locations for Tasks 2.1 and 2.2.

3. **Complete Pass 3**: Replace the query descriptions with actual SQL SELECT statements in the designated locations for Tasks 3.1, 3.2, 3.3, and 3.4.

### Expected Outcomes

- **Pass 1**: Function should return the correct account balance (1500.00 for account_id=1)
- **Pass 2**: Function should return correct transaction count (3 transactions for account_id=1 in the given period)  
- **Pass 3**: Function should return a properly formatted JSON string with opening balance, closing balance, total credits, and total debits

### Validation Tests

After completing each pass, run the provided test procedures to verify your implementation:

```sql
-- Test Pass 1
CALL test_get_account_balance_red();

-- Test Pass 2  
CALL test_get_txn_count_red();

-- Test Pass 3
SELECT get_account_summary_report(1, '2025-05-01', '2025-07-01');
```

### Bonus Challenges

1. **Error Handling**: Add proper error handling for invalid account IDs
2. **Edge Cases**: Test with accounts that have no transactions in the specified period
3. **Performance**: Add appropriate indexes to optimize query performance
4. **Validation**: Create additional test cases for different account types and date ranges
5. **Documentation**: Add comments explaining the business logic in each function

---

## Cleanup Script

```sql
-- Clean up all test data and schema
DROP TABLE IF EXISTS account_transactions;
DROP TABLE IF EXISTS accounts;
DROP DATABASE IF EXISTS atm_lab;
-- VERIFY: SHOW DATABASES LIKE 'atm_lab'; -- should return nothing
```

---

## Assessment Criteria

- **Correctness**: SQL queries produce expected results
- **TDD Compliance**: Following RED-GREEN-REFACTOR cycle
- **Code Quality**: Clean, readable SQL with proper formatting
- **Testing**: All test procedures pass successfully
- **Understanding**: Demonstration of SQL concepts (JOINs, aggregations, functions)