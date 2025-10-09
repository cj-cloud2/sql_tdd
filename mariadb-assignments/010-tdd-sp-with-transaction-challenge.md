# TDD Challenge LAB: Banking Day-End Posting Stored Procedure

---

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 5 (Expert)
**Domain:** Banking Day End Job
**Objective:** Create a stored procedure that executes a single atomic transaction performing 1 insert and 2 updates using Test-Driven Development

---

## Business Requirements

A retail bank runs a nightly "Day-End" batch that accrues daily interest for eligible deposit accounts and updates account processing flags. The bank must ensure ACID properties: if any step fails, no partial updates persist and the job must be logged for audit.

### Core Business Requirements
1. **BR001:** Record a Day-End Job run with job metadata (business_date, start/end times, status) for audit and traceability
2. **BR002:** Accrue daily interest for eligible active accounts and post accruals as ledger entries; update account balances accordingly in one transaction
3. **BR003:** Mark accounts processed for the business date and finalize the job status to SUCCESS on commit or FAILED on rollback

---

## Testable Requirements

- **TR001:** Job Logging Start - Insert a row in day_end_job with status STARTED and a generated job_id
- **TR002:** Interest Accrual and Ledger Posting (Atomic) - Insert ledger entries and update account balances within the same transaction
- **TR003:** Processing Flags and Job Finalization - Set processed_on flags and update job status to SUCCESS/FAILED

---

## Pre-Setup Phase (Provided - Do Not Modify)

All database schema and seed data are provided in the instructor's solution file. Use that file to create the database structure. The setup includes:

- `day_end_job` table for audit logging
- `accounts` table with interest rates and balances  
- `ledger_entries` table for transaction journaling
- Sample accounts with various statuses and interest rates

**Interest Calculation Formula:** `daily_interest = ROUND(balance * interest_rate / 365, 2)`

**Eligibility Criteria:** Only accounts where `status='ACTIVE'` and `interest_rate > 0` are eligible for accrual

---

## Challenge Instructions

**Your task:** Replace each "Task X.X" description in Pass 1, Pass 2, and Pass 3 with the actual SQL code. Follow RED-GREEN-REFACTOR strictly.

---

## PASS 1: TR001 - Job Logging Start

### RED Phase
Create a placeholder stored procedure and failing test as shown in the solution. The procedure should initially return a stub result.

### GREEN Phase - Your Tasks

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
BEGIN
    -- Task 1.1: Insert a new record into day_end_job table
    -- Description: Insert a row with the provided business_date and status 'STARTED'
    -- Use the INSERT statement to create the initial job log entry
    -- [Replace this comment with your INSERT statement]

    -- Task 1.2: Return the created job information for verification
    -- Description: Select job_id, business_date, and status from day_end_job 
    -- where business_date matches the input parameter
    -- [Replace this comment with your SELECT statement]
END //
DELIMITER ;
```

### REFACTOR Phase
Run the test - it should now PASS. Verify the job record was created properly.

---

## PASS 2: TR002 - Interest Accrual and Ledger Posting (Atomic)

### RED Phase
Create a comprehensive test that checks for ledger entries and balance updates. The test should initially FAIL.

### GREEN Phase - Your Tasks

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
MODIFIES SQL DATA
BEGIN
    DECLARE v_job_id BIGINT UNSIGNED;
    DECLARE v_err INT DEFAULT 0;

    -- Error handlers provided in solution
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SET v_err = 1;
        ROLLBACK;
        UPDATE day_end_job 
        SET status='FAILED', ended_at=NOW(), message='SQLEXCEPTION - rolled back'
        WHERE job_id = v_job_id;
        SELECT 'FAILED' AS status, p_business_date AS business_date, 'EXCEPTION' AS note;
    END;

    START TRANSACTION;

    -- Task 2.1: Insert or update the job record with STARTED status
    -- Description: Insert into day_end_job with business_date and status 'STARTED'
    -- Use ON DUPLICATE KEY UPDATE to handle existing records for the same date
    -- Then capture the job_id using LAST_INSERT_ID()
    -- [Replace this comment with your INSERT...ON DUPLICATE KEY UPDATE statement]
    -- [Replace this comment with SET v_job_id = LAST_INSERT_ID();]

    -- Task 2.2: Insert interest accrual records into ledger_entries
    -- Description: Insert one row per eligible account (ACTIVE status, interest_rate > 0, amount > 0.00)
    -- Calculate daily interest as ROUND(balance * interest_rate / 365, 2)
    -- Include account_id, entry_date (p_business_date), entry_type 'INTEREST_ACCRUAL', 
    -- calculated amount, and appropriate description
    -- Use a subquery or JOIN to calculate amounts and filter eligible accounts
    -- [Replace this comment with your INSERT INTO ledger_entries statement]

    -- Task 2.3: Update account balances with accrued interest
    -- Description: Update accounts.balance by adding the total accrual amount
    -- Join with ledger_entries where entry_date = p_business_date and entry_type = 'INTEREST_ACCRUAL'
    -- Group by account_id and sum the amounts, then update the corresponding account balances
    -- [Replace this comment with your UPDATE accounts statement with JOIN]

    -- Task 2.4: Update job status to indicate completion
    -- Description: Update day_end_job set ended_at = NOW(), status = 'SUCCESS' 
    -- where job_id = v_job_id
    -- [Replace this comment with your UPDATE day_end_job statement]

    COMMIT;

    -- Return final job information
    SELECT job_id, business_date, status, ended_at, message
    FROM day_end_job WHERE job_id = v_job_id;
END //
DELIMITER ;
```

### REFACTOR Phase
Run the TR002 test - it should now PASS. Verify ledger entries and balance updates.

---

## PASS 3: TR003 - Processing Flags and Job Finalization

### RED Phase
Create a test that verifies processed_on flags are set correctly and job finalization works. Should initially FAIL.

### GREEN Phase - Your Tasks

```sql
DROP PROCEDURE IF EXISTS RunDayEndJob;

DELIMITER //
CREATE PROCEDURE RunDayEndJob(IN p_business_date DATE)
MODIFIES SQL DATA
BEGIN
    DECLARE v_job_id BIGINT UNSIGNED;
    DECLARE v_err INT DEFAULT 0;

    -- Error handlers (same as before)
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

    -- Task 3.1: Insert/update job record (same as Task 2.1)
    -- [Replace this comment with your INSERT...ON DUPLICATE KEY UPDATE and job_id capture]

    -- Task 3.2: Insert accrual ledger entries (same as Task 2.2)
    -- [Replace this comment with your INSERT INTO ledger_entries statement]

    -- Task 3.3: Update account balances (same as Task 2.3)  
    -- [Replace this comment with your UPDATE accounts balance statement]

    -- Task 3.4: Set processed_on flags for accounts that received accruals
    -- Description: Update accounts.processed_on = p_business_date 
    -- for accounts that have ledger entries for the current business date with entry_type 'INTEREST_ACCRUAL'
    -- Use a JOIN with a subquery that selects DISTINCT account_id from ledger_entries
    -- [Replace this comment with your UPDATE accounts processed_on statement]

    -- Task 3.5: Finalize job status as SUCCESS
    -- Description: Update day_end_job set ended_at = NOW(), status = 'SUCCESS', 
    -- message = 'Day-End completed' where job_id = v_job_id
    -- [Replace this comment with your final UPDATE day_end_job statement]

    COMMIT;

    SELECT job_id, business_date, status, ended_at, message
    FROM day_end_job WHERE job_id = v_job_id;
END //
DELIMITER ;
```

### REFACTOR Phase
Run the TR003 test - it should now PASS. Verify processed_on flags and final job status.

---

## Validation Tests

After completing each pass, run these validation commands:

```sql
-- Test Pass 1: Job logging
CALL Test_TR001_JobLoggingStart();

-- Test Pass 2: Interest accrual and balance updates  
CALL Test_TR002_AccrualAtomic();

-- Test Pass 3: Processing flags and finalization
CALL Test_TR003_FlagsAndFinalize();

-- Manual verification
SELECT * FROM day_end_job WHERE business_date = CURRENT_DATE;
SELECT * FROM ledger_entries WHERE entry_date = CURRENT_DATE;
SELECT account_no, balance, processed_on FROM accounts WHERE status = 'ACTIVE';
```

---

## Expected Outcomes

- **Pass 1:** Job record created with STARTED status and unique job_id
- **Pass 2:** Ledger entries inserted for eligible accounts, balances updated atomically
- **Pass 3:** Processing flags set correctly, job status finalized as SUCCESS

---

## Transaction Requirements Met

The solution implements exactly "1 Insert + 2 Updates" within a single transaction:

1. **1 Insert:** `ledger_entries` table with interest accrual records
2. **Update 1:** `accounts.balance` incremented with accrued interest  
3. **Update 2:** `accounts.processed_on` set to business date

---

## Bonus Challenge Ideas

### 1. **Enhanced Error Handling**
- Add specific error codes for different failure scenarios
- Implement retry logic for transient errors
- Create detailed error logging with stack traces

### 2. **Advanced Transaction Features**
- Implement savepoints for partial rollback scenarios
- Add transaction isolation level considerations
- Create deadlock detection and resolution

### 3. **Performance Optimization**
- Add appropriate indexes for large account volumes
- Implement batch processing for high-volume scenarios
- Use EXPLAIN to analyze query execution plans

### 4. **Audit and Compliance**
- Add digital signatures to ledger entries
- Implement change tracking for all modifications
- Create regulatory reporting outputs

### 5. **Idempotency and Recovery**
- Enhance duplicate detection logic
- Add support for reprocessing failed dates
- Implement partial recovery for interrupted jobs

### 6. **Monitoring and Alerting**
- Create performance metrics collection
- Add business rule validation alerts
- Implement SLA monitoring for processing times

### 7. **Advanced Testing Framework**
- Create parameterized tests for edge cases
- Add stress testing with large datasets
- Implement property-based testing scenarios

---

## Assessment Rubric

- **Correctness (40%):** All three passes work correctly, tests transition from FAIL to PASS
- **Transaction Integrity (25%):** Proper use of START TRANSACTION/COMMIT/ROLLBACK with error handlers
- **TDD Methodology (20%):** Strict adherence to RED-GREEN-REFACTOR cycle
- **Code Quality (15%):** Clean, readable SQL with appropriate comments and error handling

---

## Submission Requirements

1. Complete stored procedure implementation for all three passes
2. Demonstrate all tests transitioning from FAIL to PASS
3. Show validation of ACID properties (atomicity, consistency)
4. Provide evidence of idempotency (running same date twice)
5. Optional: Implement and document one or more bonus challenges