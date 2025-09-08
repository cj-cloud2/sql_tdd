# Assignment: TDD with Stubs in PL/SQL (Extended)
**Domain:** Banking (Loan Eligibility Check)
**Package Prefix:** `tdd_fake_loan_eligibility`
**Framework:** Manual Testing (No utPLSQL)
***
## 🎯 Testable Requirements
- **R1:** A function `check_loan_eligibility(p_customer_id)` should return `'ELIGIBLE'` if the customer's credit score (retrieved from a stubbed function) is **>= 700**, otherwise return `'NOT ELIGIBLE'`.
- **R2:** A function `calculate_max_loan(p_customer_id)` should return the **maximum loan amount** for a customer:
    - If credit score ≥ 700 → max loan = `10x salary`
    - If credit score < 700 → max loan = `5x salary`
*(Salary will also be retrieved from a stub for now.)*
***
## 🧪 Step-by-Step TDD (RED → GREEN → REFACTOR)
***
### 🟢 First Requirement (R1) — Already Done in Cycle 1
*(We keep what we already built — students should still run through cycle once.)*
👉 **Test for R1** (copy-paste into IDE first):
```sql
create or replace package tdd_fake_loan_eligibility_test is
  procedure test_high_credit_score;
  procedure test_low_credit_score;
end tdd_fake_loan_eligibility_test;
/
create or replace package body tdd_fake_loan_eligibility_test is
  procedure test_high_credit_score is
    l_result varchar2(50);
    l_expected varchar2(50) := 'ELIGIBLE';
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.check_loan_eligibility(101);
    -- Assert
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_high_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_high_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20001, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_high_credit_score;

  procedure test_low_credit_score is
    l_result varchar2(50);
    l_expected varchar2(50) := 'NOT ELIGIBLE';
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.check_loan_eligibility(202);
    -- Assert
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_low_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_low_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20002, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_low_credit_score;
end tdd_fake_loan_eligibility_test;
/
```
***
👉 **Implementation for R1** (package with stubs):
```sql
create or replace package tdd_fake_loan_eligibility is
  function check_loan_eligibility(p_customer_id number) return varchar2;
  function calculate_max_loan(p_customer_id number) return number; -- R2 (to be developed next)
end tdd_fake_loan_eligibility;
/
create or replace package body tdd_fake_loan_eligibility is
  -- Stub for credit score
  function stub_get_credit_score(p_customer_id number) return number is
  begin
    if p_customer_id = 101 then
      return 750; -- High score
    elsif p_customer_id = 202 then
      return 650; -- Low score
    else
      return 600; -- Default low
    end if;
  end stub_get_credit_score;
  -- R1: eligibility function
  function check_loan_eligibility(p_customer_id number) return varchar2 is
  begin
    if stub_get_credit_score(p_customer_id) >= 700 then
      return 'ELIGIBLE';
    else
      return 'NOT ELIGIBLE';
    end if;
  end check_loan_eligibility;
  -- R2: loan calculation (yet to implement properly)
  function calculate_max_loan(p_customer_id number) return number is
  begin
    return null; -- temporary (RED)
  end calculate_max_loan;
end tdd_fake_loan_eligibility;
/
```
👉 Run tests:
```sql
set serveroutput on;
exec tdd_fake_loan_eligibility_test.test_high_credit_score;
exec tdd_fake_loan_eligibility_test.test_low_credit_score;
```
✅ R1 tests pass, R2 test not written yet. Next cycle!
***
### 🔴 Requirement 2 (R2) — New Cycle
***
#### Step 1: Write New Test (RED)
Copy into test package (add new test procedures):
```sql
create or replace package tdd_fake_loan_eligibility_test is
  procedure test_high_credit_score;
  procedure test_low_credit_score;
  procedure test_high_score_loan;
  procedure test_low_score_loan;
end tdd_fake_loan_eligibility_test;
/
create or replace package body tdd_fake_loan_eligibility_test is
  procedure test_high_credit_score is
    l_result varchar2(50);
    l_expected varchar2(50) := 'ELIGIBLE';
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.check_loan_eligibility(101);
    -- Assert
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_high_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_high_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20001, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_high_credit_score;

  procedure test_low_credit_score is
    l_result varchar2(50);
    l_expected varchar2(50) := 'NOT ELIGIBLE';
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.check_loan_eligibility(202);
    -- Assert
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_low_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_low_credit_score - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20002, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_low_credit_score;

  procedure test_high_score_loan is
    l_result number;
    l_expected number := 50000;
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.calculate_max_loan(101);
    -- Assert (expected: 10 x 5000 = 50000)
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_high_score_loan - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_high_score_loan - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20003, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_high_score_loan;

  procedure test_low_score_loan is
    l_result number;
    l_expected number := 20000;
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.calculate_max_loan(202);
    -- Assert (expected: 5 x 4000 = 20000)
    if l_result = l_expected then
      dbms_output.put_line('✅ PASS: test_low_score_loan - Expected: ' || l_expected || ', Got: ' || l_result);
    else
      dbms_output.put_line('❌ FAIL: test_low_score_loan - Expected: ' || l_expected || ', Got: ' || l_result);
      raise_application_error(-20004, 'Test failed: Expected ' || l_expected || ' but got ' || l_result);
    end if;
  end test_low_score_loan;
end tdd_fake_loan_eligibility_test;
/
```
👉 Run:
```sql
set serveroutput on;
exec tdd_fake_loan_eligibility_test.test_high_score_loan;
exec tdd_fake_loan_eligibility_test.test_low_score_loan;
```
❌ These will FAIL because `calculate_max_loan` still has a dummy return (RED phase).
***
#### Step 2: Implement with Stub (GREEN)
Extend package body with a **salary stub** and update logic:
```sql
create or replace package body tdd_fake_loan_eligibility is
  -- Stub for credit score
  function stub_get_credit_score(p_customer_id number) return number is
  begin
    if p_customer_id = 101 then
      return 750; -- High score
    elsif p_customer_id = 202 then
      return 650; -- Low score
    else
      return 600;
    end if;
  end stub_get_credit_score;
  -- Stub for salary
  function stub_get_salary(p_customer_id number) return number is
  begin
    if p_customer_id = 101 then
      return 5000;
    elsif p_customer_id = 202 then
      return 4000;
    else
      return 3000;
    end if;
  end stub_get_salary;
  function check_loan_eligibility(p_customer_id number) return varchar2 is
  begin
    if stub_get_credit_score(p_customer_id) >= 700 then
      return 'ELIGIBLE';
    else
      return 'NOT ELIGIBLE';
    end if;
  end check_loan_eligibility;
  function calculate_max_loan(p_customer_id number) return number is
    l_score number;
    l_salary number;
  begin
    l_score := stub_get_credit_score(p_customer_id);
    l_salary := stub_get_salary(p_customer_id);
    if l_score >= 700 then
      return l_salary * 10;
    else
      return l_salary * 5;
    end if;
  end calculate_max_loan;
end tdd_fake_loan_eligibility;
/
```
👉 Run tests again:
```sql
set serveroutput on;
exec tdd_fake_loan_eligibility_test.test_high_score_loan;
exec tdd_fake_loan_eligibility_test.test_low_score_loan;
```
✅ All new tests should pass now (GREEN).
***
#### Step 3: Refactor
Simplify formatting, keep stubs clean:
```sql
create or replace package body tdd_fake_loan_eligibility is
  function stub_get_credit_score(p_customer_id number) return number is
  begin
    case p_customer_id
      when 101 then return 750;
      when 202 then return 650;
      else return 600;
    end case;
  end stub_get_credit_score;
  function stub_get_salary(p_customer_id number) return number is
  begin
    case p_customer_id
      when 101 then return 5000;
      when 202 then return 4000;
      else return 3000;
    end case;
  end stub_get_salary;
  function check_loan_eligibility(p_customer_id number) return varchar2 is
  begin
    if stub_get_credit_score(p_customer_id) >= 700 then
      return 'ELIGIBLE';
    else
      return 'NOT ELIGIBLE';
    end if;
  end check_loan_eligibility;
  function calculate_max_loan(p_customer_id number) return number is
  begin
    if stub_get_credit_score(p_customer_id) >= 700 then
      return stub_get_salary(p_customer_id) * 10;
    else
      return stub_get_salary(p_customer_id) * 5;
    end if;
  end calculate_max_loan;
end tdd_fake_loan_eligibility;
/
```
👉 Run tests one last time:
```sql
set serveroutput on;
exec tdd_fake_loan_eligibility_test.test_high_credit_score;
exec tdd_fake_loan_eligibility_test.test_low_credit_score;
exec tdd_fake_loan_eligibility_test.test_high_score_loan;
exec tdd_fake_loan_eligibility_test.test_low_score_loan;
```
✅ Still GREEN after refactor.
***
## ✅ Deliverables for Students
- **Functions under test**:
    - `check_loan_eligibility(p_customer_id)`
    - `calculate_max_loan(p_customer_id)`
- **Stubbed dependencies**:
    - `stub_get_credit_score(p_customer_id)`
    - `stub_get_salary(p_customer_id)`
- **Tests**:
    - Eligibility (`ELIGIBLE` / `NOT ELIGIBLE`)
    - Loan calculation (`10x salary` / `5x salary`)
***
📚 **Learning Outcomes**
1. Completing **two full TDD cycles (RED–GREEN–REFACTOR)** in PL/SQL
2. Using **stubs to fake external dependencies** (credit score, salary)
3. Writing **unit tests with manual assertions** for business logic in banking domain
***