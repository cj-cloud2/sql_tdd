# 📘 Assignment: TDD with Fakes in PL/SQL (HRMS Payroll – Extended)
## 🎯 Objective
In this assignment, you will practice **Test Driven Development (TDD)** using **manual testing** in Oracle Database. You will:
- Write **tests before code** (RED).
- Implement the simplest code to pass (GREEN).
- Refactor for clarity (REFACTOR).
- Use the **Fake pattern** instead of real database tables.
You will build functions for:
1. `calculate_bonus` (Fixed bonus).
2. `calculate_tax` (Tax based on salary).
***
## 📦 Package Name
All code goes inside a package named:
`tdd_fake_payroll`
***
# 🚀 Workflow – RED → GREEN → REFACTOR
We will go through **two passes (two functions)** to understand TDD, each time with three steps.
***
## PASS 1 – Bonus Calculation
***
### Step 1️⃣ RED – Write Test First
Create the **test package** (before writing production code):
```sql
CREATE OR REPLACE PACKAGE tdd_fake_payroll_test IS
  PROCEDURE test_calculate_bonus;
  PROCEDURE run_all_tests;
END tdd_fake_payroll_test;
/
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll_test IS
  PROCEDURE test_calculate_bonus IS
    l_bonus NUMBER;
    l_expected NUMBER := 1000;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_calculate_bonus...');
    -- ACT: Call function (not yet implemented)
    l_bonus := tdd_fake_payroll.calculate_bonus(1001);
    -- ASSERT: Should always give fixed bonus
    IF l_bonus = l_expected THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_calculate_bonus - Expected: ' || l_expected || ', Got: ' || l_bonus);
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_calculate_bonus - Expected: ' || l_expected || ', Got: ' || NVL(TO_CHAR(l_bonus), 'NULL'));
      RAISE_APPLICATION_ERROR(-20001, 'Test failed: test_calculate_bonus');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_calculate_bonus - ' || SQLERRM);
      RAISE;
  END;
  
  PROCEDURE run_all_tests IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('=== Running All Tests ===');
    test_calculate_bonus;
    DBMS_OUTPUT.PUT_LINE('=== All Tests Completed ===');
  END;
END tdd_fake_payroll_test;
/
```
👉 Run this test:
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_fake_payroll_test.run_all_tests;
END;
/
```
✅ It will **FAIL** since `tdd_fake_payroll.calculate_bonus` does not exist.
(That's expected in RED phase!)
***
### Step 2️⃣ GREEN – Implement Minimum Code
Now create the **package with Fake repository**:
```sql
CREATE OR REPLACE PACKAGE tdd_fake_payroll IS
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER;
END tdd_fake_payroll;
/
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll IS
  -- Fake repository instead of real Employee Table
  FUNCTION fake_get_employee_salary(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    IF p_emp_id = 1001 THEN
      RETURN 50000; -- Fake employee salary
    ELSE
      RETURN NULL;
    END IF;
  END;
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    -- Minimal logic: fixed bonus
    RETURN 1000;
  END;
END tdd_fake_payroll;
/
```
👉 Run tests:
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_fake_payroll_test.run_all_tests;
END;
/
```
✅ Now test passes → The code is GREEN.
***
### Step 3️⃣ REFACTOR – Cleanup without Breaking
Improve code readability while keeping the test intact:
```sql
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll IS
  -- Cleaner fake repo
  FUNCTION fake_get_employee_salary(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    CASE p_emp_id
      WHEN 1001 THEN RETURN 50000; -- Known fake employee
      ELSE RETURN NULL;
    END CASE;
  END;
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN 1000; -- For now bonus is always fixed
  END;
END tdd_fake_payroll;
/
```
👉 Rerun tests → still **PASSING**.
🎉 **End of Pass 1**
***
## PASS 2 – Tax Calculation
Now, let's add a **second function** (`calculate_tax`), which uses the **Fake salary**. Example:
Tax = 10% of salary, if employee exists. Otherwise 0.
***
### Step 1️⃣ RED – Write New Test First
Expand test package body:
```sql
CREATE OR REPLACE PACKAGE tdd_fake_payroll_test IS
  PROCEDURE test_calculate_bonus;
  PROCEDURE test_calculate_tax;
  PROCEDURE run_all_tests;
END tdd_fake_payroll_test;
/
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll_test IS
  PROCEDURE test_calculate_bonus IS
    l_bonus NUMBER;
    l_expected NUMBER := 1000;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_calculate_bonus...');
    l_bonus := tdd_fake_payroll.calculate_bonus(1001);
    IF l_bonus = l_expected THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_calculate_bonus - Expected: ' || l_expected || ', Got: ' || l_bonus);
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_calculate_bonus - Expected: ' || l_expected || ', Got: ' || NVL(TO_CHAR(l_bonus), 'NULL'));
      RAISE_APPLICATION_ERROR(-20001, 'Test failed: test_calculate_bonus');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_calculate_bonus - ' || SQLERRM);
      RAISE;
  END;
  
  PROCEDURE test_calculate_tax IS
    l_tax NUMBER;
    l_expected NUMBER := 5000;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Running test_calculate_tax...');
    -- ACT: Call function (not yet implemented)
    l_tax := tdd_fake_payroll.calculate_tax(1001);
    -- ASSERT: 10% of fake salary (50000) = 5000
    IF l_tax = l_expected THEN
      DBMS_OUTPUT.PUT_LINE('✅ PASS: test_calculate_tax - Expected: ' || l_expected || ', Got: ' || l_tax);
    ELSE
      DBMS_OUTPUT.PUT_LINE('❌ FAIL: test_calculate_tax - Expected: ' || l_expected || ', Got: ' || NVL(TO_CHAR(l_tax), 'NULL'));
      RAISE_APPLICATION_ERROR(-20002, 'Test failed: test_calculate_tax');
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('❌ ERROR: test_calculate_tax - ' || SQLERRM);
      RAISE;
  END;
  
  PROCEDURE run_all_tests IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('=== Running All Tests ===');
    test_calculate_bonus;
    test_calculate_tax;
    DBMS_OUTPUT.PUT_LINE('=== All Tests Completed ===');
  END;
END tdd_fake_payroll_test;
/
```
👉 Run tests:
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_fake_payroll_test.run_all_tests;
END;
/
```
✅ New test will **FAIL** because `calculate_tax` doesn't exist → RED.
***
### Step 2️⃣ GREEN – Implement Just Enough
Expand package:
```sql
CREATE OR REPLACE PACKAGE tdd_fake_payroll IS
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER;
  FUNCTION calculate_tax(p_emp_id NUMBER) RETURN NUMBER;
END tdd_fake_payroll;
/
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll IS
  FUNCTION fake_get_employee_salary(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    CASE p_emp_id
      WHEN 1001 THEN RETURN 50000;
      ELSE RETURN NULL;
    END CASE;
  END;
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN 1000;
  END;
  FUNCTION calculate_tax(p_emp_id NUMBER) RETURN NUMBER IS
    l_salary NUMBER;
  BEGIN
    l_salary := fake_get_employee_salary(p_emp_id);
    IF l_salary IS NOT NULL THEN
      RETURN l_salary * 0.10; -- 10% tax
    ELSE
      RETURN 0;
    END IF;
  END;
END tdd_fake_payroll;
/
```
👉 Run tests again:
```sql
SET SERVEROUTPUT ON
BEGIN
  tdd_fake_payroll_test.run_all_tests;
END;
/
```
✅ Now both tests **pass**.
***
### Step 3️⃣ REFACTOR – Cleaner Design
You may refactor calculation logic for readability:
```sql
CREATE OR REPLACE PACKAGE BODY tdd_fake_payroll IS
  FUNCTION fake_get_employee_salary(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    CASE p_emp_id
      WHEN 1001 THEN RETURN 50000;
      ELSE RETURN NULL;
    END CASE;
  END;
  FUNCTION calculate_bonus(p_emp_id NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN 1000;
  END;
  FUNCTION calculate_tax(p_emp_id NUMBER) RETURN NUMBER IS
    l_salary NUMBER := fake_get_employee_salary(p_emp_id);
  BEGIN
    RETURN NVL(l_salary, 0) * 0.10;
  END;
END tdd_fake_payroll;
/
```
👉 Run tests one last time → still PASS.
🎉 **End of Pass 2**
***
# 📑 Final Deliverables
1. `tdd_fake_payroll_test` package spec \& body.
2. `tdd_fake_payroll` package spec \& body.
3. Screenshot of passing test results.
***
# 🧭 Student Roadmap
- **Pass 1**: Write test for `calculate_bonus`, watch it fail (RED), implement function with fake repo (GREEN), clean up (REFACTOR).
- **Pass 2**: Add test for `calculate_tax`, watch it fail (RED), implement real logic with fake repo (GREEN), simplify using NVL (REFACTOR).
✅ Both functions complete, full RED-GREEN-REFACTOR cycles understood.
***
# 🧹 Cleanup Scripts
```sql
-- Drop test package
DROP PACKAGE tdd_fake_payroll_test;
/

-- Drop main package
DROP PACKAGE tdd_fake_payroll;
/

-- Verify cleanup
SELECT object_name, object_type, status 
FROM user_objects 
WHERE object_name IN ('TDD_FAKE_PAYROLL_TEST', 'TDD_FAKE_PAYROLL');
```