# TDD Step-by-Step Walkthrough LAB: Banking Day-End Posting Stored Procedure (1 Insert + 2 Updates)

Language: MariaDB-10-SQL (ANSI SQL Compliant)
Difficulty Level: 5
Domain: Banking Day End Job
Objective: Create a stored procedure that executes a single atomic transaction performing 1 insert and 2 updates, developed via strict RED-GREEN-REFACTOR TDD across 3 passes.

Main takeaway: Students will implement a day-end posting procedure that logs the job, posts end-of-day interest accruals, updates ledger balances, and marks accounts as processed, all within a single ACID transaction using START TRANSACTION/COMMIT/ROLLBACK and handlers. The assignment is structured into 3 testable requirements, each driving one TDD pass.

## Business Requirements

Background

A retail bank runs a nightly “Day-End” batch that accrues daily interest for eligible deposit accounts and updates account processing flags. The bank must ensure ACID properties: if any step fails, no partial updates persist and the job must be logged for audit. The process should be idempotent for the same business date per account.

Core Business Requirements

1. BR001: Record a Day-End Job run with job metadata (business_date, start/end times, status) for audit and traceability.
2. BR002: Accrue daily interest for eligible active accounts and post accruals as ledger entries; update account balances accordingly in one transaction.
3. BR003: Mark accounts processed for the business date and finalize the job status to SUCCESS on commit or FAILED on rollback.

## Testable Requirements

TR001: Job Logging Start
Given a business_date is provided, when starting the Day-End Job, then insert a row in day_end_job with status STARTED and a generated job_id.

TR002: Interest Accrual and Ledger Posting (Atomic)
Given active accounts with interest_rate > 0, when day-end runs, then insert one ledger entry per eligible account for the calculated daily interest and update the account’s balance within the same transaction (no partial updates on failure).

TR003: Processing Flags and Job Finalization
Given accounts processed in TR002, when day-end completes, then set processed_on to business_date on those accounts and update the job status to SUCCESS; on any error, rollback all work and set job status to FAILED.

## Pre-Setup Phase

Run the following in a fresh session. Students can copy-paste directly.

Database and Tables

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS bank_day_end_lab;
USE bank_day_end_lab;

-- Day-End Job audit table
CREATE TABLE day_end_job (
    job_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    business_date DATE NOT NULL,
    started_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ended_at DATETIME NULL,
    status ENUM('STARTED','SUCCESS','FAILED') NOT NULL,
    message VARCHAR(255) NULL,
    UNIQUE KEY u_business_date (business_date)
) ENGINE=InnoDB;

-- Accounts master
CREATE TABLE accounts (
    account_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT UNSIGNED NOT NULL,
    account_no VARCHAR(20) NOT NULL UNIQUE,
    account_type ENUM('SAVINGS','CURRENT','TERM') NOT NULL,
    status ENUM('ACTIVE','DORMANT','CLOSED') NOT NULL DEFAULT 'ACTIVE',
    interest_rate DECIMAL(7,5) NOT NULL DEFAULT 0.00000, -- e.g., 0.03500 = 3.5% p.a.
    balance DECIMAL(18,2) NOT NULL DEFAULT 0.00,
    processed_on DATE NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_status (status),
    INDEX idx_processed (processed_on)
) ENGINE=InnoDB;

-- Ledger entries (journal)
CREATE TABLE ledger_entries (
    entry_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    account_id BIGINT UNSIGNED NOT NULL,
    entry_date DATE NOT NULL,
    entry_type ENUM('INTEREST_ACCRUAL','ADJUSTMENT') NOT NULL,
    amount DECIMAL(18,2) NOT NULL,
    description VARCHAR(255) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (account_id) REFERENCES accounts(account_id),
    INDEX idx_account_date (account_id, entry_date)
) ENGINE=InnoDB;
```

Seed Data

```sql
-- Active interest-bearing accounts
INSERT INTO accounts (customer_id, account_no, account_type, status, interest_rate, balance)
VALUES
(101, 'SB1001', 'SAVINGS', 'ACTIVE', 0.03500, 100000.00),
(102, 'SB1002', 'SAVINGS', 'ACTIVE', 0.02000, 50000.00),
(103, 'CA2001', 'CURRENT', 'ACTIVE', 0.00000, 250000.00),
(104, 'SB1003', 'SAVINGS', 'DORMANT', 0.03500, 20000.00),
(105, 'SB1004', 'SAVINGS', 'ACTIVE', 0.02500, 0.00);

-- A closed account (should not be processed)
INSERT INTO accounts (customer_id, account_no, account_type, status, interest_rate, balance)
VALUES (106, 'SB9999', 'SAVINGS', 'CLOSED', 0.03000, 15000.00);
```

Computation Notes

- Daily interest (simple accrual for lab): daily_interest = ROUND(balance * (interest_rate) / 365, 2). This is a simplified teaching formula.
- Only accounts where status='ACTIVE' and interest_rate>0 are eligible for accrual in this lab.
- All changes in a single stored procedure transaction using START TRANSACTION / COMMIT / ROLLBACK.
- Use SQL security declarations and deterministic notes per CREATE PROCEDURE guidance.


## TDD Implementation – Pass 1: TR001 Job Logging Start

RED Phase – Create Empty Placeholder Stored Procedure

```sql
-- Ensure clean start
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
BEGIN
    -- Placeholder for TDD RED: returns a stub row (not production)
    SELECT 'NOT_IMPLEMENTED' AS status, p_business_date AS business_date;
END //
DELIMITER ;
```

RED Phase – Write Failing Test (expects a STARTED row in day_end_job)

```sql
DROP PROCEDURE IF EXISTS Test_TR001_JobLoggingStart;

DELIMITER //
CREATE PROCEDURE Test_TR001_JobLoggingStart()
BEGIN
    DECLARE v_bd DATE DEFAULT CURRENT_DATE;

    -- Pre-clean any prior run
    DELETE FROM day_end_job WHERE business_date = v_bd;

    -- Call placeholder (will not insert STARTED)
    CALL RunDayEndJob(v_bd);

    -- Assert: expect exactly 1 STARTED row for v_bd
    SELECT 
        'TR001_JobLoggingStart' AS test_name,
        CASE 
            WHEN EXISTS (
                SELECT 1 FROM day_end_job 
                WHERE business_date = v_bd AND status = 'STARTED'
            ) THEN 'PASS' ELSE 'FAIL'
        END AS result,
        (SELECT COUNT(*) FROM day_end_job WHERE business_date = v_bd AND status='STARTED') AS started_rows;
END //
DELIMITER ;

-- Run test - should FAIL (no STARTED row created)
CALL Test_TR001_JobLoggingStart();
```

Expected Result: FAIL

GREEN Phase – Minimal Code to Pass

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
BEGIN
    -- Minimal to pass TR001: insert STARTED row
    INSERT INTO day_end_job (business_date, status)
    VALUES (p_business_date, 'STARTED');

    -- Return the job_id to aid tests/debug
    SELECT job_id, business_date, status FROM day_end_job WHERE business_date = p_business_date;
END //
DELIMITER ;

-- Re-run the test - should PASS
CALL Test_TR001_JobLoggingStart();
```

Expected Result: PASS

REFACTOR Phase – Verify and Clean

```sql
-- Verify state
SELECT * FROM day_end_job WHERE business_date = CURRENT_DATE;

-- Keep test for next passes or drop if desired
-- DROP PROCEDURE IF EXISTS Test_TR001_JobLoggingStart;
```

Reference: CREATE PROCEDURE syntax characteristics and invocation via CALL.

## TDD Implementation – Pass 2: TR002 Interest Accrual and Ledger Posting (Atomic)

RED Phase – Create Failing Test

```sql
DROP PROCEDURE IF EXISTS Test_TR002_AccrualAtomic;

DELIMITER //
CREATE PROCEDURE Test_TR002_AccrualAtomic()
BEGIN
    DECLARE v_bd DATE DEFAULT CURRENT_DATE;

    -- Reset state for v_bd
    DELETE FROM ledger_entries WHERE entry_date = v_bd;
    DELETE FROM day_end_job WHERE business_date = v_bd;
    UPDATE accounts SET processed_on = NULL;

    -- Sanity snapshot balances
    SELECT account_id, balance INTO @a1_id, @a1_bal FROM accounts WHERE account_no='SB1001';
    SELECT account_id, balance INTO @a2_id, @a2_bal FROM accounts WHERE account_no='SB1002';
    SELECT account_id, balance INTO @a3_id, @a3_bal FROM accounts WHERE account_no='CA2001'; -- no interest
    SELECT account_id, balance INTO @aDorm_id, @aDorm_bal FROM accounts WHERE account_no='SB1003'; -- dormant
    SELECT account_id, balance INTO @aZero_id, @aZero_bal FROM accounts WHERE account_no='SB1004'; -- zero balance
    SELECT account_id, balance INTO @aClosed_id, @aClosed_bal FROM accounts WHERE account_no='SB9999'; -- closed

    -- Execute day-end
    CALL RunDayEndJob(v_bd);

    -- Expectations:
    -- 1) A STARTED row was created earlier (TR001 behavior) and the transaction posted:
    --    For ACTIVE & interest_rate>0 accounts only:
    --    • SB1001: accrual posted, ledger row exists, balance increased by ROUND(bal*rate/365,2)
    --    • SB1002: accrual posted similarly
    --    • CA2001: no ledger/accrual (rate=0)
    --    • SB1003: no accrual (DORMANT)
    --    • SB1004: accrual may be 0.00; for this lab, require a ledger row only if amount > 0.00
    --    • SB9999: no accrual (CLOSED)

    -- Check ledger entries count: expect exactly 2 accruals (SB1001, SB1002)
    SELECT 
        'TR002_LedgerCount' AS test_name,
        CASE 
            WHEN (SELECT COUNT(*) FROM ledger_entries 
                  WHERE entry_date = v_bd AND entry_type='INTEREST_ACCRUAL') = 2
            THEN 'PASS' ELSE 'FAIL'
        END AS result,
        (SELECT COUNT(*) FROM ledger_entries WHERE entry_date = v_bd AND entry_type='INTEREST_ACCRUAL') AS accrual_rows;

    -- Compute expected accruals
    SELECT ROUND(@a1_bal * 0.03500 / 365, 2) INTO @a1_accr;
    SELECT ROUND(@a2_bal * 0.02000 / 365, 2) INTO @a2_accr;

    -- Check balances updated atomically
    SELECT 
        'TR002_BalanceUpdate_SB1001' AS test_name,
        CASE 
            WHEN (SELECT balance FROM accounts WHERE account_id=@a1_id) = @a1_bal + @a1_accr
            THEN 'PASS' ELSE 'FAIL'
        END AS result,
        (SELECT balance FROM accounts WHERE account_id=@a1_id) AS new_balance,
        @a1_bal + @a1_accr AS expected_balance;

    SELECT 
        'TR002_BalanceUpdate_SB1002' AS test_name,
        CASE 
            WHEN (SELECT balance FROM accounts WHERE account_id=@a2_id) = @a2_bal + @a2_accr
            THEN 'PASS' ELSE 'FAIL'
        END AS result,
        (SELECT balance FROM accounts WHERE account_id=@a2_id) AS new_balance,
        @a2_bal + @a2_accr AS expected_balance;

    -- Check non-eligible accounts: no accrual rows
    SELECT 
        'TR002_NoAccrual_CA2001' AS test_name,
        CASE 
            WHEN NOT EXISTS (
                SELECT 1 FROM ledger_entries 
                WHERE account_id=@a3_id AND entry_date=v_bd AND entry_type='INTEREST_ACCRUAL'
            ) THEN 'PASS' ELSE 'FAIL'
        END AS result;

    SELECT 
        'TR002_NoAccrual_DORMANT' AS test_name,
        CASE 
            WHEN NOT EXISTS (
                SELECT 1 FROM ledger_entries 
                WHERE account_id=@aDorm_id AND entry_date=v_bd AND entry_type='INTEREST_ACCRUAL'
            ) THEN 'PASS' ELSE 'FAIL'
        END AS result;

    SELECT 
        'TR002_NoAccrual_CLOSED' AS test_name,
        CASE 
            WHEN NOT EXISTS (
                SELECT 1 FROM ledger_entries 
                WHERE account_id=@aClosed_id AND entry_date=v_bd AND entry_type='INTEREST_ACCRUAL'
            ) THEN 'PASS' ELSE 'FAIL'
        END AS result;
END //
DELIMITER ;

-- Run - should FAIL until transaction logic and accrual code exist
CALL Test_TR002_AccrualAtomic();
```

Expected Result: FAIL

GREEN Phase – Implement Transaction with 1 Insert + 2 Updates

- Insert: ledger_entries accrual row per eligible account (this is the “1 insert”).
- Update 1: accounts.balance increment by accrual amount.
- Update 2: day_end_job.ended_at/status on success, and on error set FAILED.

MariaDB official guidance for CREATE PROCEDURE and transaction statements apply.

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
MODIFIES SQL DATA
BEGIN
    DECLARE v_job_id BIGINT UNSIGNED;
    DECLARE v_err INT DEFAULT 0;

    -- Error handlers: on any exception or warning, rollback and flag failure
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SET v_err = 1;
        ROLLBACK;
        -- Update job status to FAILED if job row exists
        UPDATE day_end_job 
        SET status='FAILED', ended_at=NOW(), message='SQLEXCEPTION - rolled back'
        WHERE job_id = v_job_id;
        SELECT 'FAILED' AS status, p_business_date AS business_date, 'EXCEPTION' AS note;
    END;

    DECLARE EXIT HANDLER FOR SQLWARNING
    BEGIN
        SET v_err = 1;
        ROLLBACK;
        UPDATE day_end_job 
        SET status='FAILED', ended_at=NOW(), message='SQLWARNING - rolled back'
        WHERE job_id = v_job_id;
        SELECT 'FAILED' AS status, p_business_date AS business_date, 'WARNING' AS note;
    END;

    START TRANSACTION;

    -- Insert STARTED row if not present (idempotent per business_date)
    INSERT INTO day_end_job (business_date, status)
    VALUES (p_business_date, 'STARTED')
    ON DUPLICATE KEY UPDATE job_id=LAST_INSERT_ID(job_id);

    -- Capture job_id (either inserted or existing)
    SET v_job_id = LAST_INSERT_ID();

    -- Interest Accrual: eligible = ACTIVE and interest_rate > 0 and not already processed for p_business_date
    -- For each eligible account, compute daily accrual; only insert ledger if accrual > 0.00
    -- 1 INSERT
    INSERT INTO ledger_entries (account_id, entry_date, entry_type, amount, description)
    SELECT 
        a.account_id,
        p_business_date AS entry_date,
        'INTEREST_ACCRUAL' AS entry_type,
        accr.amount,
        CONCAT('Daily interest accrual for ', a.account_no, ' on ', DATE_FORMAT(p_business_date, '%Y-%m-%d'))
    FROM accounts a
    JOIN (
        SELECT 
            account_id,
            -- ROUND to 2 decimals, ignore zero amounts
            ROUND(balance * (interest_rate) / 365, 2) AS amount
        FROM accounts
    ) accr ON accr.account_id = a.account_id
    WHERE a.status = 'ACTIVE'
      AND a.interest_rate > 0
      AND (a.processed_on IS NULL OR a.processed_on <> p_business_date)
      AND accr.amount > 0.00;

    -- 1st UPDATE: increment balances only for accounts that received a ledger accrual row
    UPDATE accounts a
    JOIN (
        SELECT le.account_id, SUM(le.amount) AS total_accr
        FROM ledger_entries le
        WHERE le.entry_date = p_business_date
          AND le.entry_type = 'INTEREST_ACCRUAL'
        GROUP BY le.account_id
    ) x ON x.account_id = a.account_id
    SET a.balance = a.balance + x.total_accr;

    -- 2nd UPDATE: job end success set later in TR003 after flags;
    -- For TR002 minimal pass, we can set ended_at & status tentatively, will finalize in TR003
    UPDATE day_end_job
       SET ended_at = NOW(), status = 'SUCCESS', message = 'Accrual completed (intermediate)'
     WHERE job_id = v_job_id;

    COMMIT;

    -- Return job status snapshot
    SELECT job_id, business_date, status, ended_at, message
    FROM day_end_job WHERE job_id = v_job_id;
END //
DELIMITER ;

-- Run TR002 test - should PASS now
CALL Test_TR002_AccrualAtomic();
```

Expected Result: PASS

REFACTOR Phase – Quick Validation

```sql
-- Verify ledger entries for today
SELECT * FROM ledger_entries WHERE entry_date = CURRENT_DATE ORDER BY account_id;

-- Keep the test for next pass or drop later
-- DROP PROCEDURE IF EXISTS Test_TR002_AccrualAtomic;
```

References: Transactions and rollback semantics in MariaDB ; stored procedure creation and invocation.

## TDD Implementation – Pass 3: TR003 Processing Flags and Job Finalization

RED Phase – Create Failing Test

```sql
DROP PROCEDURE IF EXISTS Test_TR003_FlagsAndFinalize;

DELIMITER //
CREATE PROCEDURE Test_TR003_FlagsAndFinalize()
BEGIN
    DECLARE v_bd DATE DEFAULT CURRENT_DATE;

    -- Reset state for the date
    DELETE FROM ledger_entries WHERE entry_date = v_bd;
    DELETE FROM day_end_job WHERE business_date = v_bd;
    UPDATE accounts SET processed_on = NULL;

    -- Run job (should set processed_on for accounts that accrued)
    CALL RunDayEndJob(v_bd);

    -- Expect processed_on = v_bd for only eligible accounts that accrued (>0 amount):
    SELECT account_id INTO @a1_id FROM accounts WHERE account_no='SB1001';
    SELECT account_id INTO @a2_id FROM accounts WHERE account_no='SB1002';
    SELECT account_id INTO @a3_id FROM accounts WHERE account_no='CA2001';
    SELECT account_id INTO @aDorm_id FROM accounts WHERE account_no='SB1003';
    SELECT account_id INTO @aZero_id FROM accounts WHERE account_no='SB1004';
    SELECT account_id INTO @aClosed_id FROM accounts WHERE account_no='SB9999';

    -- Check flags
    SELECT 'TR003_ProcessedFlag_SB1001' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@a1_id) = v_bd THEN 'PASS' ELSE 'FAIL' END AS result;
    SELECT 'TR003_ProcessedFlag_SB1002' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@a2_id) = v_bd THEN 'PASS' ELSE 'FAIL' END AS result;

    -- Non-eligible or zero-accrual must remain NULL
    SELECT 'TR003_NoFlag_CA2001' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@a3_id) IS NULL THEN 'PASS' ELSE 'FAIL' END AS result;
    SELECT 'TR003_NoFlag_DORMANT' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@aDorm_id) IS NULL THEN 'PASS' ELSE 'FAIL' END AS result;

    -- SB1004: zero balance leads to 0.00 accrual => no ledger row => keep NULL
    SELECT 'TR003_NoFlag_ZeroAccrual' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@aZero_id) IS NULL THEN 'PASS' ELSE 'FAIL' END AS result;

    -- Closed must remain NULL
    SELECT 'TR003_NoFlag_CLOSED' AS test_name,
           CASE WHEN (SELECT processed_on FROM accounts WHERE account_id=@aClosed_id) IS NULL THEN 'PASS' ELSE 'FAIL' END AS result;

    -- Job finalization must be SUCCESS with ended_at set
    SELECT 'TR003_JobFinalStatus' AS test_name,
           CASE 
              WHEN EXISTS (SELECT 1 FROM day_end_job WHERE business_date = v_bd AND status='SUCCESS' AND ended_at IS NOT NULL)
              THEN 'PASS' ELSE 'FAIL'
           END AS result;
END //
DELIMITER ;

-- Run - should FAIL until flags and finalization are added
CALL Test_TR003_FlagsAndFinalize();
```

Expected Result: FAIL

GREEN Phase – Finalize SP: processed_on and SUCCESS/FAILED finalization

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
MODIFIES SQL DATA
BEGIN
    DECLARE v_job_id BIGINT UNSIGNED;
    DECLARE v_err INT DEFAULT 0;

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SET v_err = 1;
        ROLLBACK;
        UPDATE day_end_job 
           SET status='FAILED', ended_at=NOW(), message='SQLEXCEPTION - rolled back'
         WHERE job_id = v_job_id;
        SELECT 'FAILED' AS status, p_business_date AS business_date, 'EXCEPTION' AS note;
    END;

    DECLARE EXIT HANDLER FOR SQLWARNING
    BEGIN
        SET v_err = 1;
        ROLLBACK;
        UPDATE day_end_job 
           SET status='FAILED', ended_at=NOW(), message='SQLWARNING - rolled back'
         WHERE job_id = v_job_id;
        SELECT 'FAILED' AS status, p_business_date AS business_date, 'WARNING' AS note;
    END;

    START TRANSACTION;

    -- Insert STARTED if not exists, capture job_id
    INSERT INTO day_end_job (business_date, status)
    VALUES (p_business_date, 'STARTED')
    ON DUPLICATE KEY UPDATE job_id=LAST_INSERT_ID(job_id);
    SET v_job_id = LAST_INSERT_ID();

    -- Accrual INSERT
    INSERT INTO ledger_entries (account_id, entry_date, entry_type, amount, description)
    SELECT 
        a.account_id,
        p_business_date AS entry_date,
        'INTEREST_ACCRUAL' AS entry_type,
        accr.amount,
        CONCAT('Daily interest accrual for ', a.account_no, ' on ', DATE_FORMAT(p_business_date, '%Y-%m-%d'))
    FROM accounts a
    JOIN (
        SELECT 
            account_id,
            ROUND(balance * (interest_rate) / 365, 2) AS amount
        FROM accounts
    ) accr ON accr.account_id = a.account_id
    WHERE a.status = 'ACTIVE'
      AND a.interest_rate > 0
      AND (a.processed_on IS NULL OR a.processed_on <> p_business_date)
      AND accr.amount > 0.00;

    -- Balance UPDATE
    UPDATE accounts a
    JOIN (
        SELECT le.account_id, SUM(le.amount) AS total_accr
        FROM ledger_entries le
        WHERE le.entry_date = p_business_date
          AND le.entry_type = 'INTEREST_ACCRUAL'
        GROUP BY le.account_id
    ) x ON x.account_id = a.account_id
    SET a.balance = a.balance + x.total_accr;

    -- Processed flag UPDATE only for accounts that had accrual posted
    UPDATE accounts a
    JOIN (
        SELECT DISTINCT account_id
        FROM ledger_entries
        WHERE entry_date = p_business_date
          AND entry_type = 'INTEREST_ACCRUAL'
    ) y ON y.account_id = a.account_id
    SET a.processed_on = p_business_date;

    -- Finalize job success
    UPDATE day_end_job
       SET ended_at = NOW(), status = 'SUCCESS', message = 'Day-End completed'
     WHERE job_id = v_job_id;

    COMMIT;

    -- Return final status
    SELECT job_id, business_date, status, ended_at, message
    FROM day_end_job WHERE job_id = v_job_id;
END //
DELIMITER ;

-- Run TR003 test - should PASS now
CALL Test_TR003_FlagsAndFinalize();
```

Expected Result: PASS

REFACTOR Phase – Comprehensive Smoke

```sql
-- Basic smoke: counts per date
SELECT 
    (SELECT COUNT(*) FROM ledger_entries WHERE entry_date = CURRENT_DATE AND entry_type='INTEREST_ACCRUAL') AS accrual_rows,
    (SELECT COUNT(*) FROM accounts WHERE processed_on = CURRENT_DATE) AS processed_accounts;

-- Idempotency quick check: re-run should not duplicate for same date
CALL RunDayEndJob(CURRENT_DATE);

-- Verify no duplicate accruals for same date due to processed_on guard
SELECT account_id, COUNT(*) AS n
FROM ledger_entries
WHERE entry_date = CURRENT_DATE AND entry_type='INTEREST_ACCRUAL'
GROUP BY account_id
HAVING COUNT(*) > 1;
```

References: MariaDB CREATE PROCEDURE characteristics; transactions, commit and rollback; handlers and atomicity requirements.

## How This Meets “1 Insert + 2 Updates” Transaction

Within a single START TRANSACTION ... COMMIT block:

- 1 Insert: Insert into ledger_entries per eligible account for interest accrual.
- Update 1: Update accounts.balance with the posted accrual sums.
- Update 2: Update accounts.processed_on to the business date for accounts that accrued.

Additionally, the job audit row is inserted/updated as part of the same procedure (STARTED and final SUCCESS/FAILED), but the instructional constraint is satisfied by the accrual insert plus two distinct updates on accounts.

## How to Run The Lab (Student Steps)

1) Run “Pre-Setup Phase” scripts to create schema and seed data.
2) Execute Pass 1 scripts in order: placeholder SP → TR001 test (fail) → implement minimal SP → run TR001 test (pass).
3) Execute Pass 2 scripts: TR002 test (fail) → implement accrual with transaction and atomic updates → run TR002 test (pass).
4) Execute Pass 3 scripts: TR003 test (fail) → finalize flags and job finalization → run TR003 test (pass).
5) Optional refactor checks, idempotency verification.
6) Use CALL RunDayEndJob('YYYY-MM-DD') for a specific business date.

## Cleanup Script

```sql
USE bank_day_end_lab;

-- Drop procedures
DROP PROCEDURE IF EXISTS RunDayEndJob;
DROP PROCEDURE IF EXISTS Test_TR001_JobLoggingStart;
DROP PROCEDURE IF EXISTS Test_TR002_AccrualAtomic;
DROP PROCEDURE IF EXISTS Test_TR003_FlagsAndFinalize;

-- Drop tables
DROP TABLE IF EXISTS ledger_entries;
DROP TABLE IF EXISTS accounts;
DROP TABLE IF EXISTS day_end_job;

-- Drop database
DROP DATABASE IF EXISTS bank_day_end_lab;

-- Verify cleanup (these should return empty)
SHOW DATABASES LIKE 'bank_day_end_lab';
```


