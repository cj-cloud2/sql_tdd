# 📘 Assignment Views 002 – Full Guided TDD Example (Difficulty‑5)
***
## **Scenario (Banking Trade-Finance Domain)**
In the **Trade-Finance division** of a Bank, customers maintain **transactions** (DEBITS when they pay, CREDITS when deposits or collections happen). Only **ACTIVE customers** should be shown in financial summaries.
Business wants a reporting **view** that summarizes:
1. Total debits and credits of each **active customer**.
2. A computed **net balance** = credit – debit.
3. Ensure that inactive customers (like those blacklisted/closed) **should not appear**.
We will build this **view using Test-Driven Development (TDD)** with **manual testing**.
***
## **Learning Objectives**
- Practice the **RED–GREEN–REFACTOR cycle** in PL/SQL.
- Learn to write **tests first, code later**.
- Handle **deliberate bug introduction + fixing**.
- Use **manual testing** properly.
***
## **Testable Requirements**
1. **R1:** The view must return transactions only for **ACTIVE customers**.
2. **R2:** The view must correctly show **debit and credit totals** per customer.
3. **R3:** The view must correctly calculate **net_balance = credit - debit**.
4. **R4:** The view must **include customers with no transactions at all**, showing `0` debit, `0` credit, and `0` balance (❌ will fail first due to a bug in our initial view).
***
## **Step-by-Step Assignment Guide**
### ⚙️ Step 1. Create Schema Objects (Tables + Sample Data + Placeholder view)
```sql
-- Drop any leftover objects before creating
BEGIN
   EXECUTE IMMEDIATE 'DROP VIEW vw_tradefinance_summary';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE transactions CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE customers CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/
-- Customers table
CREATE TABLE customers (
    customer_id   NUMBER PRIMARY KEY,
    customer_name VARCHAR2(100),
    status        VARCHAR2(20) CHECK (status IN ('ACTIVE','INACTIVE'))
);
-- Transactions table
CREATE TABLE transactions (
    txn_id        NUMBER PRIMARY KEY,
    customer_id   NUMBER REFERENCES customers(customer_id),
    txn_type      VARCHAR2(10) CHECK (txn_type IN ('DEBIT','CREDIT')),
    amount        NUMBER(12,2)
);
-- Simple placeholder view so test procedures compile
CREATE OR REPLACE VIEW vw_tradefinance_summary AS
SELECT NULL AS customer_id,
       NULL AS customer_name,
       NULL AS total_debit,
       NULL AS total_credit,
       NULL AS net_balance
FROM dual WHERE 1=0; -- empty result
-- Sample Data
INSERT INTO customers VALUES (1, 'Alice Corp', 'ACTIVE');
INSERT INTO customers VALUES (2, 'Bob Traders', 'INACTIVE');
INSERT INTO customers VALUES (3, 'Global Finance', 'ACTIVE');
INSERT INTO customers VALUES (4, 'Dormant Textiles', 'ACTIVE'); -- No transactions (for R4 test)
-- Some transactions
INSERT INTO transactions VALUES (101, 1, 'DEBIT',  5000);
INSERT INTO transactions VALUES (102, 1, 'CREDIT', 10000);
INSERT INTO transactions VALUES (103, 2, 'CREDIT', 8000); -- Bob INACTIVE, should NOT appear
INSERT INTO transactions VALUES (104, 3, 'DEBIT',  2000);
INSERT INTO transactions VALUES (105, 3, 'CREDIT', 5000);
COMMIT;
```
***
### ⚙️ Step 2. Create Empty Business Package
```sql
CREATE OR REPLACE PACKAGE tdd_tradefinance_txn_summary IS
END tdd_tradefinance_txn_summary;
/
CREATE OR REPLACE PACKAGE BODY tdd_tradefinance_txn_summary IS
END tdd_tradefinance_txn_summary;
/
```
***
### ⚙️ Step 3. Create Test Package Skeleton (Manual Testing Procedures)
```sql
CREATE OR REPLACE PACKAGE tdd_tradefinance_txn_summary_test IS
  PROCEDURE test_active_customers_only;
  PROCEDURE test_debit_credit_totals;
  PROCEDURE test_net_balance;
  PROCEDURE test_customers_with_no_transactions;
  PROCEDURE run_all_tests;
END tdd_tradefinance_txn_summary_test;
/
```
***
### ⚙️ Step 4. Create Test Package Body with Manual Assertions
```sql
CREATE OR REPLACE PACKAGE BODY tdd_tradefinance_txn_summary_test IS
  PROCEDURE test_active_customers_only IS
    l_count INTEGER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_active_customers_only...');
    SELECT COUNT(*)
      INTO l_count
      FROM vw_tradefinance_summary
     WHERE customer_id = 2; -- Bob (INACTIVE)
    IF l_count = 0 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_active_customers_only');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_active_customers_only - INACTIVE customer found');
      RAISE_APPLICATION_ERROR(-20001, 'Test failed: test_active_customers_only');
    END IF;
  EXCEPTION WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('❌ ERROR in test_active_customers_only: ' || SQLERRM);
    RAISE;
  END;

  PROCEDURE test_debit_credit_totals IS
    l_debit  NUMBER;
    l_credit NUMBER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_debit_credit_totals...');
    SELECT total_debit, total_credit
      INTO l_debit, l_credit
      FROM vw_tradefinance_summary
     WHERE customer_id = 1; -- Alice
    IF l_debit = 5000 AND l_credit = 10000 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_debit_credit_totals');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_debit_credit_totals - Expected (5000,10000), Got (' || l_debit || ',' || l_credit || ')');
      RAISE_APPLICATION_ERROR(-20002, 'Test failed: test_debit_credit_totals');
    END IF;
  EXCEPTION WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('❌ ERROR in test_debit_credit_totals: ' || SQLERRM);
    RAISE;
  END;

  PROCEDURE test_net_balance IS
    l_net NUMBER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_net_balance...');
    SELECT net_balance
      INTO l_net
      FROM vw_tradefinance_summary
     WHERE customer_id = 3; -- Global Finance
    IF l_net = 3000 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_net_balance');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_net_balance - Expected 3000, Got ' || l_net);
      RAISE_APPLICATION_ERROR(-20003, 'Test failed: test_net_balance');
    END IF;
  EXCEPTION WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('❌ ERROR in test_net_balance: ' || SQLERRM);
    RAISE;
  END;

  PROCEDURE test_customers_with_no_transactions IS
    l_debit  NUMBER;
    l_credit NUMBER;
    l_net    NUMBER;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_customers_with_no_transactions...');
    SELECT total_debit, total_credit, net_balance
      INTO l_debit, l_credit, l_net
      FROM vw_tradefinance_summary
     WHERE customer_id = 4; -- Dormant Textiles no transactions
    IF l_debit = 0 AND l_credit = 0 AND l_net = 0 THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_customers_with_no_transactions');
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_customers_with_no_transactions - Expected (0,0,0), Got (' || l_debit || ',' || l_credit || ',' || l_net || ')');
      RAISE_APPLICATION_ERROR(-20004, 'Test failed: test_customers_with_no_transactions');
    END IF;
  EXCEPTION WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('❌ ERROR in test_customers_with_no_transactions: ' || SQLERRM);
    RAISE;
  END;

  PROCEDURE run_all_tests IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('=== Running All TradeFinance Summary View Tests ===');
    test_active_customers_only;
    test_debit_credit_totals;
    test_net_balance;
    test_customers_with_no_transactions;
    DBMS_OUTPUT.PUT_LINE('=== All TradeFinance Summary View Tests Completed ===');
  END;
END tdd_tradefinance_txn_summary_test;
/
```
***
### ⚙️ Step 5. RED Phase: Run tests BEFORE correct view exists
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_tradefinance_txn_summary_test.run_all_tests;
END;
/
```
Expected: Tests fail due to placeholder view with no data.
***
### ⚙️ Step 6. GREEN Phase: Implement the View for First 3 Requirements
```sql
CREATE OR REPLACE VIEW vw_tradefinance_summary AS
SELECT
    c.customer_id,
    c.customer_name,
    NVL(SUM(CASE WHEN t.txn_type = 'DEBIT'  THEN t.amount END),0) AS total_debit,
    NVL(SUM(CASE WHEN t.txn_type = 'CREDIT' THEN t.amount END),0) AS total_credit,
    NVL(SUM(CASE WHEN t.txn_type = 'CREDIT' THEN t.amount END),0) -
    NVL(SUM(CASE WHEN t.txn_type = 'DEBIT'  THEN t.amount END),0) AS net_balance
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
WHERE c.status = 'ACTIVE'
GROUP BY c.customer_id, c.customer_name;
```
***
### ⚙️ Step 7. Run tests again (Expect R4 fails)
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_tradefinance_txn_summary_test.run_all_tests;
END;
/
```
Expected:
- R1, R2, R3 pass
- R4 fails (customer with no txn missing)
***
### ⚙️ Step 8. REFACTOR Phase: Fix bug for R4 by changing JOIN to LEFT JOIN
```sql
CREATE OR REPLACE VIEW vw_tradefinance_summary AS
SELECT
    c.customer_id,
    c.customer_name,
    NVL(SUM(CASE WHEN t.txn_type = 'DEBIT'  THEN t.amount END),0) AS total_debit,
    NVL(SUM(CASE WHEN t.txn_type = 'CREDIT' THEN t.amount END),0) AS total_credit,
    NVL(SUM(CASE WHEN t.txn_type = 'CREDIT' THEN t.amount END),0) -
    NVL(SUM(CASE WHEN t.txn_type = 'DEBIT'  THEN t.amount END),0) AS net_balance
FROM customers c
LEFT JOIN transactions t ON c.customer_id = t.customer_id
WHERE c.status = 'ACTIVE'
GROUP BY c.customer_id, c.customer_name;
```
***
### ⚙️ Step 9. Run tests one last time
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_tradefinance_txn_summary_test.run_all_tests;
END;
/
```
Expected: All tests pass (R1-R4)
***
## 🧹 Cleanup Script (Important for Labs)
```sql
BEGIN
  EXECUTE IMMEDIATE 'DROP VIEW vw_tradefinance_summary';
EXCEPTION WHEN OTHERS THEN
  IF SQLCODE != -942 THEN RAISE; END IF;
END;
/
BEGIN
  EXECUTE IMMEDIATE 'DROP PACKAGE tdd_tradefinance_txn_summary_test';
EXCEPTION WHEN OTHERS THEN
  IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
  EXECUTE IMMEDIATE 'DROP PACKAGE tdd_tradefinance_txn_summary';
EXCEPTION WHEN OTHERS THEN
  IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
  EXECUTE IMMEDIATE 'DROP TABLE transactions CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN
  IF SQLCODE != -942 THEN RAISE; END IF;
END;
/
BEGIN
  EXECUTE IMMEDIATE 'DROP TABLE customers CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN
  IF SQLCODE != -942 THEN RAISE; END IF;
END;
/
```
***
## 🎯 Final Assignment Summary for Students
In this lab you:
1. **Created tables and sample data with a placeholder view** to allow tests to compile.
2. **Wrote manual test procedures first** simulating utPLSQL style.
3. Verified **RED phase** (tests fail initially due to placeholder view).
4. Implemented **view step by step** until tests passed.
5. Learned to fix a **deliberate bug (INNER JOIN vs LEFT JOIN)** to cover edge cases.
6. Practiced **4 test-driven passes**, one per requirement.
***