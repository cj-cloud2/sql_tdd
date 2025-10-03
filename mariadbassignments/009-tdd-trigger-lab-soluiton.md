# TDD Step-by-Step Walkthrough LAB: Banking Transaction Audit Trigger (MariaDB)

---

### Language: MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking
**Objective:** Create a trigger that audits transaction inserts with TDD (Test-Driven Development), strictly following the RED-GREEN-REFACTOR cycle.

---

## Business Requirements

The bank wants to improve traceability and compliance by ensuring that every new entry in the `bank_transactions` table is also recorded in `transaction_audit` table. The audit must include who inserted the transaction and the time of insertion. 

### Core Business Requirements
1. **BR001:** Record all inserts to the `bank_transactions` table in the `transaction_audit` table for traceability.
2. **BR002:** Ensure audit log stores the username of the inserter and timestamp.
3. **BR003:** Prevent direct manual inserts to the audit table (enforce insertions only via trigger).

---

## Testable Requirements

Based on business requirements, define testable requirements for TDD:

- **TR001:** Each new transaction insert triggers a corresponding row in `transaction_audit`.
- **TR002:** Audit entry contains accurate username and timestamp from the transaction insertion context.
- **TR003:** Reject manual inserts into the audit table (direct inserts fail with error).

---

## Pre-Setup Phase
### Database and Table Creation
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

```

-- Insert reference users
```sql
INSERT INTO bank_transactions (account_no, txn_type, amount, created_by)
VALUES ('A001', 'DEPOSIT', 1000.00, 'alice'),
       ('A002', 'WITHDRAWAL', 500.00, 'bob');
```

---

# TDD Implementation: 3 Passes (RED-GREEN-REFACTOR for Each Requirement)

----

## **PASS 1: TR001 - Trigger Audit Row on Insert**

#### **RED Phase:**
Create an empty trigger, and a test SP that expects an audit record (which should fail).

```sql
-- RED: Create placeholder trigger (this does nothing yet)
DELIMITER //
CREATE TRIGGER audit_bank_insert AFTER INSERT ON bank_transactions
FOR EACH ROW
BEGIN
    -- No-op for now
END;//
DELIMITER ;

-- RED: Failing test SP (should expect row in audit table after insert, but none will be present)
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

-- Run it (should FAIL, result=TR001_FAIL, because trigger is empty)
CALL Test_TR001_AuditRowOnInsert();
```

#### **GREEN Phase:**
Add minimal working trigger code.
```sql
-- GREEN: Implement trigger to insert audit row
DROP TRIGGER IF EXISTS audit_bank_insert;
DELIMITER //
CREATE TRIGGER audit_bank_insert AFTER INSERT ON bank_transactions
FOR EACH ROW
BEGIN
    INSERT INTO transaction_audit
        (txn_id, account_no, txn_type, amount, audit_user, audit_time)
    VALUES (NEW.txn_id, NEW.account_no, NEW.txn_type, NEW.amount, NEW.created_by, NOW());
END;//
DELIMITER ;

-- Rerun the test SP (should PASS, result=TR001_PASS)
CALL Test_TR001_AuditRowOnInsert();
```

#### **REFACTOR Phase:**
Clean up test and validate result accuracy.
```sql
-- Verify audit matches transaction (check last row)
SELECT * FROM transaction_audit WHERE audit_id = (SELECT MAX(audit_id) FROM transaction_audit);

-- Drop test SP (optional cleanup)
DROP PROCEDURE IF EXISTS Test_TR001_AuditRowOnInsert;


```

---

## **PASS 2: TR002 - Username and Timestamp are Accurate**

#### **RED Phase:**
Test for correct user/timestamp mapping. (Should FAIL with wrong user/timestamp if implementation is incomplete.)
```sql
DELIMITER //
CREATE PROCEDURE Test_TR002_AuditValues()
BEGIN
    DECLARE v_user VARCHAR(30);
    DECLARE v_now DATETIME;

    INSERT INTO bank_transactions(account_no, txn_type, amount, created_by)
    VALUES('A888','DEPOSIT',120.00, 'qa_test');

    SELECT audit_user, audit_time INTO v_user, v_now
    FROM transaction_audit
    WHERE account_no='A888'
    ORDER BY audit_id DESC
    LIMIT 1;

    IF v_user = 'qa_test' AND v_now >= NOW() - INTERVAL 1 MINUTE THEN
        SELECT 'TR002_PASS' AS result, v_user, v_now;
    ELSE
        SELECT 'TR002_FAIL' AS result, v_user, v_now;
    END IF;
END//
DELIMITER ;

-- Run (should FAIL if values not correct)
CALL Test_TR002_AuditValues();
```

#### **GREEN Phase:**
If the trigger uses NEW.created_by and NOW(), this should pass. Fix otherwise.
```sql
-- If needed, edit the trigger to use NEW.created_by (already done), so just rerun test
CALL Test_TR002_AuditValues();
```

#### **REFACTOR Phase:**
Clean up.
```sql
-- Delete test procedure
DROP PROCEDURE IF EXISTS Test_TR002_AuditValues;

-- Optional: Check audit integrity manually
SELECT audit_user, audit_time FROM transaction_audit WHERE account_no='A888';
```

---

## **PASS 3: TR003 - Prevent Manual Audit Inserts**

#### **RED Phase:**
Try direct insert -- test expects signal error (should fail now as any insert is allowed).
```sql
DELIMITER //
CREATE PROCEDURE Test_TR003_ManualAuditInsert()
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
        SELECT 'TR003_PASS' AS result; -- pass if insert failed

    INSERT INTO transaction_audit
        (txn_id, account_no, txn_type, amount, audit_user, audit_time)
    VALUES (999, 'A777', 'DEPOSIT', 5.00, 'hacker', NOW());

    -- If insert succeeded, then fail
    SELECT 'TR003_FAIL' AS result;
END//
DELIMITER ;
-- Run (should FAIL, because manual insert is allowed)
CALL Test_TR003_ManualAuditInsert();
```

#### **GREEN Phase:**
Add BEFORE INSERT trigger to block manual entries.
```sql
DELIMITER //
CREATE TRIGGER prevent_manual_audit_before_insert
BEFORE INSERT ON transaction_audit
FOR EACH ROW
BEGIN
    IF (CURRENT_USER() NOT LIKE '%mariadb.sys%' AND NEW.audit_user != 'TRIGGER') THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Manual insert to audit table prohibited!';
    END IF;
END;//
DELIMITER ;

-- Rerun test (should PASS)
CALL Test_TR003_ManualAuditInsert();
```

#### **REFACTOR Phase:**
Remove test proc and check validation.
```sql
DROP PROCEDURE IF EXISTS Test_TR003_ManualAuditInsert;

-- Try manual insert (should error):
INSERT INTO transaction_audit (txn_id, account_no, txn_type, amount, audit_user, audit_time)
VALUES (111, 'A666', 'WITHDRAWAL', 1.00, 'badguy', NOW());
-- Should throw 'Manual insert to audit table prohibited!'

```





---

# Cleanup Script
```sql
DROP TRIGGER IF EXISTS audit_bank_insert;
DROP TRIGGER IF EXISTS prevent_manual_audit_before_insert;
DROP TABLE IF EXISTS transaction_audit;
DROP TABLE IF EXISTS bank_transactions;
DROP DATABASE IF EXISTS tdd_bank_lab;
```

