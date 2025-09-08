# Assignment – TDD Views 001
**Domain**: Banking ATM
**Topic**: Creating a view via TDD using manual testing
***
## 1. Problem Statement
Banks want to track **ATM withdrawal transactions**.
We need to build a VIEW to analyze withdrawals. The VIEW should expose:
- `account_id`
- `transaction_date`
- `amount`
- `status` (whether the withdrawal is `"SUCCESS"` or `"FAILURE"`)
We will build this **view** following strict **TDD (Test Driven Development)** cycles using **manual testing**.
***
## 2. Testable Requirements
1. **Requirement 1**: The view must return only transactions of type `'ATM_WITHDRAWAL'`.
2. **Requirement 2**: The column `status` must display correctly:
    - If `transaction_success = 'Y'` → status = `'SUCCESS'`
    - Else → status = `'FAILURE'`
3. **Requirement 3**: Withdrawals should only return **positive** amounts.
***
## 3. RED-GREEN-REFACTOR Cycles (Step by Step)
### 3.1. Setup – Base Schema Objects
```sql
-- Table used for ATM transactions
CREATE TABLE atm_transactions (
    transaction_id       NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    account_id           NUMBER NOT NULL,
    transaction_type     VARCHAR2(30) NOT NULL,
    transaction_date     DATE DEFAULT SYSDATE,
    amount               NUMBER(10,2),
    transaction_success  CHAR(1) CHECK (transaction_success IN ('Y','N'))
);
```
Insert seed test data:
```sql
INSERT INTO atm_transactions(account_id, transaction_type, amount, transaction_success)
VALUES (101, 'ATM_WITHDRAWAL', 500, 'Y');   -- Success withdrawal
INSERT INTO atm_transactions(account_id, transaction_type, amount, transaction_success)
VALUES (102, 'ATM_WITHDRAWAL', -200, 'N');  -- Negative failure withdrawal
INSERT INTO atm_transactions(account_id, transaction_type, amount, transaction_success)
VALUES (103, 'ONLINE_PAYMENT', 100, 'Y');   -- Not ATM withdrawal
COMMIT;
```
***
### 3.2. Create the Testing Package
We create a **testing package** containing 3 procedures (1 per requirement).
We use **dynamic SQL** so the package compiles even before the view exists.
```sql
CREATE OR REPLACE PACKAGE tdd_atm_withdraw_view IS
  PROCEDURE test_withdrawal_filter;
  PROCEDURE test_status_display;
  PROCEDURE test_amount_positive;
  PROCEDURE run_all_tests;
END tdd_atm_withdraw_view;
/
```
#### Package Body
```sql
CREATE OR REPLACE PACKAGE BODY tdd_atm_withdraw_view IS
  PROCEDURE test_withdrawal_filter IS
    l_count NUMBER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_withdrawal_filter...');
    EXECUTE IMMEDIATE
      'SELECT COUNT(*) FROM atm_withdraw_view WHERE transaction_type <> ''ATM_WITHDRAWAL'''
      INTO l_count;
    IF l_count = 0 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_withdrawal_filter - No non-ATM_WITHDRAWAL records found');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_withdrawal_filter - Found ' || l_count || ' non-ATM_WITHDRAWAL records');
      RAISE_APPLICATION_ERROR(-20001, 'Test failed: test_withdrawal_filter');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_withdrawal_filter - ' || SQLERRM);
      RAISE;
  END test_withdrawal_filter;

  PROCEDURE test_status_display IS
    l_status VARCHAR2(10);
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_status_display...');
    EXECUTE IMMEDIATE
      'SELECT status FROM atm_withdraw_view WHERE account_id = 101'
      INTO l_status;
    IF l_status = 'SUCCESS' THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_status_display - Status correctly shows SUCCESS');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_status_display - Expected SUCCESS, got ' || NVL(l_status, 'NULL'));
      RAISE_APPLICATION_ERROR(-20002, 'Test failed: test_status_display');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_status_display - ' || SQLERRM);
      RAISE;
  END test_status_display;

  PROCEDURE test_amount_positive IS
    l_count NUMBER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_amount_positive...');
    EXECUTE IMMEDIATE
      'SELECT COUNT(*) FROM atm_withdraw_view WHERE amount <= 0'
      INTO l_count;
    IF l_count = 0 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_amount_positive - No non-positive amounts found');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_amount_positive - Found ' || l_count || ' non-positive amount records');
      RAISE_APPLICATION_ERROR(-20003, 'Test failed: test_amount_positive');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_amount_positive - ' || SQLERRM);
      RAISE;
  END test_amount_positive;

  PROCEDURE run_all_tests IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('=== Running ATM Withdraw View Tests ===');
    test_withdrawal_filter;
    test_status_display;
    test_amount_positive;
    DBMS_OUTPUT.PUT_LINE('=== All Tests Completed Successfully ===');
  END run_all_tests;
END tdd_atm_withdraw_view;
/
```
***
## 4. RED-GREEN-REFACTOR Passes
### 🔴 **PASS 1: RED**
- Run:
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_atm_withdraw_view.run_all_tests;
END;
/
```
- At this point `atm_withdraw_view` does NOT exist.
- Test results: Failures due to missing object → this is RED phase.
***
### 🟢 **PASS 1: GREEN**
Create minimal view to satisfy **Requirement 1**:
```sql
CREATE OR REPLACE VIEW atm_withdraw_view AS
SELECT
    account_id,
    transaction_type,
    transaction_date,
    amount,
    transaction_success,
    NULL AS status
FROM atm_transactions
WHERE transaction_type = 'ATM_WITHDRAWAL';
```
- Run tests again:
✅ Requirement 1 passes
❌ Requirement 2 \& 3 fail
***
### 🟠 **PASS 1: REFACTOR**
View is clean. Proceed.
***
### 🔴 **PASS 2: RED**
- Run tests again → Requirement 2 still failing (status is NULL).
***
### 🟢 **PASS 2: GREEN**
Improve view to add status logic:
```sql
CREATE OR REPLACE VIEW atm_withdraw_view AS
SELECT
    account_id,
    transaction_type,
    transaction_date,
    amount,
    CASE WHEN transaction_success = 'Y' THEN 'SUCCESS'
         ELSE 'FAILURE'
    END AS status
FROM atm_transactions
WHERE transaction_type = 'ATM_WITHDRAWAL';
```
- Run tests again:
✅ Requirement 1 \& 2 pass
❌ Requirement 3 fails (negative amount still in results)
***
### 🟠 **PASS 2: REFACTOR**
Looks good. Move on.
***
### 🔴 **PASS 3: RED**
- Requirement 3 fails (negative amount still available in the view).
***
### 🟢 **PASS 3: GREEN**
Fix query to exclude non-positive amounts:
```sql
CREATE OR REPLACE VIEW atm_withdraw_view AS
SELECT
    account_id,
    transaction_type,
    transaction_date,
    amount,
    CASE WHEN transaction_success = 'Y' THEN 'SUCCESS'
         ELSE 'FAILURE'
    END AS status
FROM atm_transactions
WHERE transaction_type = 'ATM_WITHDRAWAL'
  AND amount > 0;
```
- Run tests again:
✅ All 3 tests pass
***
### 🟠 **PASS 3: REFACTOR**
Final cleanup is good – no further changes needed.
***
## 5. Deliverables for Student
- **One package**: `TDD_ATM_WITHDRAW_VIEW` (tests).
- **One view**: `ATM_WITHDRAW_VIEW` (final implementation).
- **Three RED–GREEN–REFACTOR cycles** demonstrated.
- Students must:
1. Run tests RED (fail)
2. Incrementally add logic to the view
3. Get GREEN pass for each step
***
## 🎯 Learning Objectives
- Practice TDD with **manual testing** for SQL views.
- Understand **incremental test-driven development** in PL/SQL.
- Experience real **banking ATM domain** modeling.
***
✅ Cleanup commands to delete all objects:
```sql
-- Drop the test package specification and body
BEGIN
  EXECUTE IMMEDIATE 'DROP PACKAGE tdd_atm_withdraw_view';
EXCEPTION
  WHEN OTHERS THEN
    IF SQLCODE != -4043 THEN -- ignore "object does not exist" error
      RAISE;
    END IF;
END;
/
-- Drop the view
BEGIN
  EXECUTE IMMEDIATE 'DROP VIEW atm_withdraw_view';
EXCEPTION
  WHEN OTHERS THEN
    IF SQLCODE != -4041 THEN -- ignore "view does not exist" error
      RAISE;
    END IF;
END;
/
-- Drop the table (optional if you want to cleanup everything)
BEGIN
  EXECUTE IMMEDIATE 'DROP TABLE atm_transactions CASCADE CONSTRAINTS';
EXCEPTION
  WHEN OTHERS THEN
    IF SQLCODE != -942 THEN -- ignore "table does not exist" error
      RAISE;
    END IF;
END;
/
```