# TDD Challenge LAB: Banking Transaction Audit Trigger

---

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 3 (Intermediate)
**Domain:** Banking
**Objective:** Create a trigger that audits transaction inserts using Test-Driven Development (TDD)

---

## Business Requirements

The bank needs to improve traceability and compliance by ensuring every transaction insert is automatically audited with proper user tracking and timestamp recording.

### Core Business Requirements
1. **BR001:** Record all inserts to the `bank_transactions` table in the `transaction_audit` table for traceability
2. **BR002:** Ensure audit log stores the username of the inserter and timestamp
3. **BR003:** Prevent direct manual inserts to the audit table (enforce insertions only via trigger)

---

## Testable Requirements

- **TR001:** Each new transaction insert triggers a corresponding row in `transaction_audit`
- **TR002:** Audit entry contains accurate username and timestamp from the transaction insertion context
- **TR003:** Reject manual inserts into the audit table (direct inserts fail with error)

---

## Pre-Setup Phase (Provided - Do Not Modify)

```sql
-- Create required database
CREATE DATABASE IF NOT EXISTS tdd_bank_lab;
USE tdd_bank_lab;

-- Main banking transactions table
CREATE TABLE bank_transactions (
    txn_id INT AUTO_INCREMENT PRIMARY KEY,
    account_no VARCHAR(20) NOT NULL,
    txn_type ENUM('DEPOSIT','WITHDRAWAL') NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    created_by VARCHAR(30) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- Transaction audit table
CREATE TABLE transaction_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    txn_id INT NOT NULL,
    account_no VARCHAR(20) NOT NULL,
    txn_type ENUM('DEPOSIT','WITHDRAWAL') NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    audit_user VARCHAR(30) NOT NULL,
    audit_time DATETIME NOT NULL,
    FOREIGN KEY(txn_id) REFERENCES bank_transactions(txn_id)
) ENGINE=InnoDB;

-- Insert reference data
INSERT INTO bank_transactions (account_no, txn_type, amount, created_by)
VALUES ('A001', 'DEPOSIT', 1000.00, 'alice'),
       ('A002', 'WITHDRAWAL', 500.00, 'bob');
```

---

## Challenge Instructions

**Your task:** Replace each "Task X.X" description with the actual SQL code to implement the required functionality. Follow the RED-GREEN-REFACTOR cycle strictly.

---

## PASS 1: TR001 - Trigger Audit Row on Insert

### RED Phase
Create an empty trigger and a failing test:

```sql
-- RED: Create placeholder trigger (does nothing yet)
DELIMITER //
CREATE TRIGGER audit_bank_insert AFTER INSERT ON bank_transactions
FOR EACH ROW
BEGIN
    -- No-op for now
END;//
DELIMITER ;

-- RED: Failing test procedure
DELIMITER //
CREATE PROCEDURE Test_TR001_AuditRowOnInsert()
BEGIN
    DECLARE v_before INT DEFAULT 0;
    DECLARE v_after INT DEFAULT 0;

    SELECT COUNT(*) INTO v_before FROM transaction_audit;

    INSERT INTO bank_transactions(account_no, txn_type, amount, created_by)
    VALUES('A999','DEPOSIT',252.50,'testuser');

    SELECT COUNT(*) INTO v_after FROM transaction_audit;

    IF v_after = v_before+1 THEN
        SELECT 'TR001_PASS' AS result, v_before, v_after;
    ELSE
        SELECT 'TR001_FAIL' AS result, v_before, v_after;
    END IF;
END//
DELIMITER ;

-- Run test (should FAIL)
CALL Test_TR001_AuditRowOnInsert();
```

### GREEN Phase - Your Tasks

```sql
-- Task 1.1: Replace the empty trigger with working code
-- Description: Create an AFTER INSERT trigger that inserts a new row into transaction_audit 
-- containing the transaction ID, account number, transaction type, amount, created_by user, 
-- and current timestamp from the newly inserted transaction
DROP TRIGGER IF EXISTS audit_bank_insert;
DELIMITER //
CREATE TRIGGER audit_bank_insert AFTER INSERT ON bank_transactions
FOR EACH ROW
BEGIN
    -- [Replace this comment with your INSERT statement for transaction_audit table]
    -- Use NEW.txn_id, NEW.account_no, NEW.txn_type, NEW.amount, NEW.created_by, and NOW()
END;//
DELIMITER ;

-- Rerun test (should PASS)
CALL Test_TR001_AuditRowOnInsert();
```

### REFACTOR Phase
```sql
-- Verify audit matches transaction
SELECT * FROM transaction_audit WHERE audit_id = (SELECT MAX(audit_id) FROM transaction_audit);

-- Drop test procedure
DROP PROCEDURE IF EXISTS Test_TR001_AuditRowOnInsert;
```

---

## PASS 2: TR002 - Username and Timestamp Accuracy

### RED Phase
```sql
DELIMITER //
CREATE PROCEDURE Test_TR002_AuditValues()
BEGIN
    DECLARE v_user VARCHAR(30);
    DECLARE v_now DATETIME;

    INSERT INTO bank_transactions(account_no, txn_type, amount, created_by)
    VALUES('A888','DEPOSIT',120.00, 'qa_test');

    -- Task 2.1: Write a SELECT statement to get audit_user and audit_time 
    -- from the most recent transaction_audit record for account 'A888'
    -- Description: Select audit_user and audit_time from transaction_audit where 
    -- account_no equals 'A888', ordered by audit_id descending, limit 1 row
    -- [Replace this comment with your SELECT statement using INTO v_user, v_now]

    IF v_user = 'qa_test' AND v_now >= NOW() - INTERVAL 1 MINUTE THEN
        SELECT 'TR002_PASS' AS result, v_user, v_now;
    ELSE
        SELECT 'TR002_FAIL' AS result, v_user, v_now;
    END IF;
END//
DELIMITER ;

-- Run test (should initially FAIL if trigger implementation is incomplete)
CALL Test_TR002_AuditValues();
```

### GREEN Phase
```sql
-- If your Task 1.1 was implemented correctly, this should now PASS
-- Rerun test to verify
CALL Test_TR002_AuditValues();
```

### REFACTOR Phase
```sql
-- Clean up
DROP PROCEDURE IF EXISTS Test_TR002_AuditValues;
```

---

## PASS 3: TR003 - Prevent Manual Audit Inserts

### RED Phase
```sql
DELIMITER //
CREATE PROCEDURE Test_TR003_ManualAuditInsert()
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
        SELECT 'TR003_PASS' AS result; -- Pass if insert failed

    INSERT INTO transaction_audit
        (txn_id, account_no, txn_type, amount, audit_user, audit_time)
    VALUES (999, 'A777', 'DEPOSIT', 5.00, 'hacker', NOW());

    -- If insert succeeded, then fail
    SELECT 'TR003_FAIL' AS result;
END//
DELIMITER ;

-- Run test (should FAIL because manual insert is currently allowed)
CALL Test_TR003_ManualAuditInsert();
```

### GREEN Phase - Your Tasks

```sql
-- Task 3.1: Create a BEFORE INSERT trigger on transaction_audit to prevent manual inserts
-- Description: Create a trigger that signals an error when someone tries to insert directly 
-- into transaction_audit table. Use SIGNAL SQLSTATE '45000' with an appropriate error message
DELIMITER //
CREATE TRIGGER prevent_manual_audit_before_insert
BEFORE INSERT ON transaction_audit
FOR EACH ROW
BEGIN
    -- [Replace this comment with your condition check and SIGNAL statement]
    -- Hint: Check if insertion is not coming from the system/trigger context
    -- Use SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Your error message here';
END;//
DELIMITER ;

-- Rerun test (should PASS)
CALL Test_TR003_ManualAuditInsert();
```

### REFACTOR Phase
```sql
DROP PROCEDURE IF EXISTS Test_TR003_ManualAuditInsert;

-- Verify manual insert prevention (should throw error):
INSERT INTO transaction_audit (txn_id, account_no, txn_type, amount, audit_user, audit_time)
VALUES (111, 'A666', 'WITHDRAWAL', 1.00, 'badguy', NOW());
```

---

## Validation Tests

After completing each pass, run these commands to verify your implementation:

```sql
-- Test 1: Insert a transaction and verify audit record
INSERT INTO bank_transactions(account_no, txn_type, amount, created_by) 
VALUES('TEST001', 'DEPOSIT', 100.00, 'validator');

-- Check if audit record was created
SELECT * FROM transaction_audit WHERE account_no = 'TEST001';

-- Test 2: Try manual audit insert (should fail)
INSERT INTO transaction_audit (txn_id, account_no, txn_type, amount, audit_user, audit_time)
VALUES (999, 'MANUAL', 'DEPOSIT', 1.00, 'manual_user', NOW());
```

---

## Expected Outcomes

- **Pass 1:** Every insert into `bank_transactions` automatically creates a corresponding audit record
- **Pass 2:** Audit records contain accurate username and timestamp information
- **Pass 3:** Direct manual inserts into `transaction_audit` are blocked with an error

---

## Bonus Challenge Ideas

### 1. **Enhanced Audit Information**
- Add column tracking the source IP address or session ID
- Include the original SQL statement that triggered the audit

### 2. **Audit Integrity Checks**
- Create a function to validate audit completeness
- Add checksums to detect audit tampering

### 3. **Performance Optimization**
- Add appropriate indexes for audit queries
- Implement audit log rotation for large datasets

### 4. **Advanced Security**
- Create different audit levels based on transaction amounts
- Implement audit log encryption for sensitive data

### 5. **Monitoring and Alerting**
- Create a view showing audit statistics
- Add triggers for unusual audit patterns (e.g., too many failed login attempts)

### 6. **Interactive Testing Framework**
- Build a comprehensive test suite with multiple edge cases
- Create parameterized tests for different user types and transaction scenarios

---

## Cleanup Script

```sql
DROP TRIGGER IF EXISTS audit_bank_insert;
DROP TRIGGER IF EXISTS prevent_manual_audit_before_insert;
DROP TABLE IF EXISTS transaction_audit;
DROP TABLE IF EXISTS bank_transactions;
DROP DATABASE IF EXISTS tdd_bank_lab;
```

---

## Assessment Rubric

- **Correctness (40%):** Triggers work as specified, all tests pass
- **TDD Adherence (25%):** Proper RED-GREEN-REFACTOR cycle followed
- **Code Quality (20%):** Clean, readable trigger code with appropriate error handling
- **Security (15%):** Proper prevention of manual audit manipulation

---

## Submission Requirements

1. Complete trigger implementation for all three passes
2. Demonstrate all tests transitioning from FAIL to PASS
3. Provide verification of manual insert prevention
4. Optional: Implement one or more bonus challenges with documentation