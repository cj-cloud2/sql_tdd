# Assignment: TDD with Stubs in PL/SQL

**Domain:** Banking (Loan Eligibility Check)
**Package Prefix:** `tdd_fake_loan_eligibility`
**Framework:** utPLSQL

***

## 🎯 Testable Requirement:

- **R1:** A function `check_loan_eligibility(p_customer_id)` should return `'ELIGIBLE'` if the customer’s credit score (retrieved from a stubbed function) is **>= 700**, otherwise return `'NOT ELIGIBLE'`.

***

## 🧪 Step-by-Step TDD (RED → GREEN → REFACTOR)

### Step 1: Write the Test First (RED)

We start by writing the failing test function.
Copy-Paste this into your IDE:

```sql
create or replace package tdd_fake_loan_eligibility_test is
  --%suite(Loan Eligibility Tests)

  --%test(Customer with high credit score should be eligible)
  procedure test_high_credit_score;

end tdd_fake_loan_eligibility_test;
/
create or replace package body tdd_fake_loan_eligibility_test is

  procedure test_high_credit_score is
    l_result varchar2(50);
  begin
    -- Act
    l_result := tdd_fake_loan_eligibility.check_loan_eligibility(101);

    -- Assert
    ut.expect(l_result).to_equal('ELIGIBLE');
  end test_high_credit_score;

end tdd_fake_loan_eligibility_test;
/
```

👉 Run the test now:

```sql
exec ut.run('tdd_fake_loan_eligibility_test');
```

❌ It will FAIL because `tdd_fake_loan_eligibility` does not exist yet.
*(This is the RED phase.)*

***

### Step 2: Create the Stub and Failing Function (still RED)

Now we create the main package **with a stubbed credit score function** that always returns a fixed value.

```sql
create or replace package tdd_fake_loan_eligibility is
  function check_loan_eligibility(p_customer_id number) return varchar2;
end tdd_fake_loan_eligibility;
/
create or replace package body tdd_fake_loan_eligibility is

  -- 🔹 STUB: simple fake credit score provider
  function stub_get_credit_score(p_customer_id number) return number is
  begin
    return 750; -- Stubbed fixed score (fake implementation)
  end stub_get_credit_score;

  function check_loan_eligibility(p_customer_id number) return varchar2 is
    l_score number;
    l_status varchar2(50);
  begin
    -- Call the stub (Dependency)
    l_score := stub_get_credit_score(p_customer_id);

    if l_score >= 700 then
      l_status := 'ELIGIBLE';
    else
      l_status := 'NOT ELIGIBLE';
    end if;

    return l_status;
  end check_loan_eligibility;

end tdd_fake_loan_eligibility;
/
```

👉 Run the test again:

```sql
exec ut.run('tdd_fake_loan_eligibility_test');
```

✅ This time it should PASS (GREEN stage).

***

### Step 3: Refactor

Now, let’s refactor:

- The stub should remain in place, but we clean up formatting.
- Later in real-world, the stub would be replaced with a DB call or API.

```sql
create or replace package body tdd_fake_loan_eligibility is

  -- Stubbed function (simulating external system / DB call)
  function stub_get_credit_score(p_customer_id number) return number is
  begin
    return 750; -- For testing purpose only
  end stub_get_credit_score;

  function check_loan_eligibility(p_customer_id number) return varchar2 is
  begin
    if stub_get_credit_score(p_customer_id) >= 700 then
      return 'ELIGIBLE';
    else
      return 'NOT ELIGIBLE';
    end if;
  end check_loan_eligibility;

end tdd_fake_loan_eligibility;
/
```

👉 Run tests again to ensure ✅ still GREEN.
This concludes the **RED → GREEN → REFACTOR cycle**.

***

## ✅ Deliverable for Students

- Test Case: `tdd_fake_loan_eligibility_test.test_high_credit_score`
- Implementation: `tdd_fake_loan_eligibility.check_loan_eligibility` with stub inside

***

📚 **Learning Outcomes**

- Understand how to use **TDD in Oracle PL/SQL** with utPLSQL
- Learn how to integrate **stubs** to isolate external dependencies
- Practice the **RED-GREEN-REFACTOR cycle**

***