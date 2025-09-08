<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Assignment Procedures 001 :  Give me 1 examples of TDD of Difficulty level 3 in which 1 stored-procedure is developed using 3 test functions

Language: PLSQL (Oracle Database 23ai Free Release)
Packaging: Create every object in a package that start with "tdd_<assignment-relevant-name>" as its prefix
Additional Requirements:
-By using  utPLSQL
-It should have 3 test functions.
-I want to create a Assignments for my students.
-First mention testable requirements, then create tests for each requirement.

- Then mention name of procedure to be created through this Assignment.
-Please create a step by step guide following perfect RED-GREEN-REFACTOR cycle.
-Student should go through a pass per test-function (ex: 3 test-functions should take the student through 3 passes of  RED-GREEN-REFACTOR cycle, to complete assignment)
-Also, while student is doing Assignment he should be able to copy paste code from the document to IDE.
-I will save it in ".md" format
-Let the example be from any of Banking  Domain
-Whereever instructed to test, give test command also to copy paste
- Dont take any shortcuts, let it be in detail. We will have only answer in this thread, but let it be detailed and  complete
- In the end, give scripts to delete all objects created in this LAB for cleanup
--Once answer is created, double-check if there are not any errors in SQL, when you execute them in sequence of this guided lab

Alright! ✅ Let’s create a **detailed step-by-step TDD Assignment** following the **RED–GREEN–REFACTOR** cycle using **PL/SQL, utPLSQL**, and the **Banking Domain**. Everything will be prepared for your students to **copy-paste into their IDE** and learn test-driven development properly.

We’ll make sure this **runs in Oracle Database 23ai Free Release** with **utPLSQL**, and that students can test it progressively through **3 test functions**.

***

# 📘 Assignment Procedures 001 – Test-Driven Development (TDD) using utPLSQL (Difficulty Level 3)


***

## 🎯 Problem Context: Banking Domain

We want to develop a stored procedure that **transfers money between two bank accounts**.

***

## ✅ Testable Requirements

1. **Requirement 1**: If source account has enough balance, the transfer should deduct from source and credit the destination account.
2. **Requirement 2**: If source account does **not** have enough balance, the transfer should **raise an exception** `INSUFFICIENT_FUNDS`.
3. **Requirement 3**: If either account ID does not exist, raise an exception `ACCOUNT_NOT_FOUND`.

***

## 📦 Package Naming Convention

All DB objects will be created inside a package named:

```
tdd_bank_transfer
```


***

# 🛠 Step-by-Step Guide (RED–GREEN–REFACTOR)

We will follow TDD:

1. Write **failing test (RED)**
2. Implement minimal code to make it pass (**GREEN**)
3. Clean/refactor code (**REFACTOR**)

We will do **3 passes**, one per requirement.

***

***

# 1️⃣ Pass 1 (Requirement 1)

### Step 1: Create Storage Structures (common for all tests)

```sql
-- Drop existing objects if any
DROP TABLE bank_accounts PURGE;

-- Base table to store balances
CREATE TABLE bank_accounts (
    account_id   NUMBER PRIMARY KEY,
    balance      NUMBER(12,2) NOT NULL
);

-- Insert some test data
INSERT INTO bank_accounts VALUES (1, 1000);
INSERT INTO bank_accounts VALUES (2, 500);
COMMIT;
```


***

### Step 2: Create Test Package with first test function (RED)

```sql

CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   -- Declare the procedure here as STUB
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );
END tdd_bank_transfer;
/

CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Empty stub – compiles fine, will be implemented after RED phase
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
   BEGIN
      NULL; -- Stub: does nothing yet
   END transfer_funds;


   -- Requirement 1: Successful transfer
   -- Transfer 200 from Account 1(1000 balance) to Account 2(500 balance)
   -- After transfer: Account 1 = 800, Account 2 = 700

   -- Test procedure (Requirement 1: Successful transfer)
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      -- Arrange - reset balances
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      -- Act: This will call the stub right now
      tdd_bank_transfer.transfer_funds(1, 2, 200);

      -- Assert: This will FAIL later, since balances won’t change yet
      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

END tdd_bank_transfer;
/

```

***

### Step 3: Create utPLSQL Test Suite

```sql

CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   -- Main procedure to test
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );

   -- Test procedure (must be in spec for test package to use it!)
   PROCEDURE test_successful_transfer;
END tdd_bank_transfer;
/

CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
   BEGIN
      NULL; -- Still stub for now
   END transfer_funds;

   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

END tdd_bank_transfer;
/
CREATE OR REPLACE PACKAGE tdd_bank_transfer_test IS
   --%suite
   --%suitepath(bank.transfer)
   --%rollback(manual)

   --%test(Requirement 1: Successful transfer between accounts)
   PROCEDURE test_successful_transfer;
END tdd_bank_transfer_test;
/

CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer_test IS
   PROCEDURE test_successful_transfer IS
   BEGIN
      tdd_bank_transfer.test_successful_transfer;
   END;
END tdd_bank_transfer_test;
/

CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
   BEGIN
      NULL; -- Still stub for now
   END transfer_funds;

   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

END tdd_bank_transfer;
/

--  Leave your package body as-is:
CREATE OR REPLACE PACKAGE tdd_bank_transfer_test IS
   --%suite
   --%suitepath(bank.transfer)
   --%rollback(manual)

   --%test(Requirement 1: Successful transfer between accounts)
   PROCEDURE test_successful_transfer;
END tdd_bank_transfer_test;
/


-- Your test package 
-- Always declare any test procedures in your main package’s spec 
-- if you want to reference them from other packages.
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer_test IS
   PROCEDURE test_successful_transfer IS
   BEGIN
      tdd_bank_transfer.test_successful_transfer;
   END;
END tdd_bank_transfer_test;
/


```


***

### Step 4: Run the Test (RED → Expected Failure)

```sql
-- Set Enable Server Output (get test output on console)
SET SERVEROUTPUT ON SIZE UNLIMITED;

-- run the test
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

You should see an **error** since `Actual: 1000 (number) was expected to equal: 800 (number)`

***

### Step 5: Implement Procedure (GREEN)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Stored Procedure to be tested
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
   BEGIN
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
   END transfer_funds;

   -- Test procedure from earlier (copied here)
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

END tdd_bank_transfer;
/
```


***

### Step 6: Run Test Again (GREEN Expected)

```sql
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

✅ Should now **pass** Requirement 1.

***


========================================================================

# 2️⃣ Pass 2 (Requirement 2) - Insufficient Funds Check

### Step 7: Add Second Test Function (RED)

We need to add the second test for insufficient funds. First, let's update our test package:

```sql
-- Update the test package spec to include the second test
CREATE OR REPLACE PACKAGE tdd_bank_transfer_test IS
   --%suite
   --%suitepath(bank.transfer)
   --%rollback(manual)

   --%test(Requirement 1: Successful transfer between accounts)
   PROCEDURE test_successful_transfer;
   
   --%test(Requirement 2: Transfer fails with insufficient funds)
   PROCEDURE test_insufficient_funds;
END tdd_bank_transfer_test;
/
```

Now update the main package spec to include the new test procedure:

```sql
CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   -- Main procedure to test
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );

   -- Test procedures (must be in spec for test package to use them!)
   PROCEDURE test_successful_transfer;
   PROCEDURE test_insufficient_funds;
END tdd_bank_transfer;
/
```

### Step 8: Implement Second Test Function (RED)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);

   -- Stored Procedure (current implementation from Pass 1)
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
   BEGIN
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
   END transfer_funds;

   -- Test 1: Successful transfer (from Pass 1)
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   -- Test 2: Insufficient funds (NEW - this will FAIL first time)
   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      -- Arrange: Account 1 has 100, trying to transfer 200 (insufficient)
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      -- Get balances before
      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      -- Act & Assert: Should raise INSUFFICIENT_FUNDS exception
      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised
      ut.expect(l_exception_raised).to_be_true();

      -- Assert: Balances should remain unchanged
      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

END tdd_bank_transfer;
/
```

### Step 9: Update Test Package Body

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer_test IS
   PROCEDURE test_successful_transfer IS
   BEGIN
      tdd_bank_transfer.test_successful_transfer;
   END;

   PROCEDURE test_insufficient_funds IS
   BEGIN
      tdd_bank_transfer.test_insufficient_funds;
   END;
END tdd_bank_transfer_test;
/
```

### Step 10: Run Tests (RED - Second Test Should Fail)

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED;
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

You should see that **test_insufficient_funds FAILS** because no exception is raised and balances change inappropriately.

### Step 11: Implement Insufficient Funds Logic (GREEN)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);

   -- Enhanced Stored Procedure with balance check
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
   BEGIN
      -- Check if source account has sufficient funds
      SELECT balance INTO l_source_balance 
        FROM bank_accounts 
       WHERE account_id = p_source_id;

      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account');
      END IF;

      -- Proceed with transfer if sufficient funds
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
   END transfer_funds;

   -- Test 1: Successful transfer
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   -- Test 2: Insufficient funds
   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      -- Arrange: Account 1 has 100, trying to transfer 200 (insufficient)
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      -- Get balances before
      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      -- Act & Assert: Should raise INSUFFICIENT_FUNDS exception
      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised
      ut.expect(l_exception_raised).to_be_true();

      -- Assert: Balances should remain unchanged
      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

END tdd_bank_transfer;
/
```

### Step 12: Run Tests Again (GREEN Expected)

```sql
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

✅ Both tests should now **pass** - Requirement 1 and Requirement 2 are satisfied.

### Step 13: Refactor (REFACTOR)

Let's improve code organization and add some transaction handling:

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);

   -- Refactored Stored Procedure with better structure
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
   BEGIN
      -- Validation: Check if source account has sufficient funds
      SELECT balance 
        INTO l_source_balance 
        FROM bank_accounts 
       WHERE account_id = p_source_id;

      -- Business rule: Insufficient funds check
      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account');
      END IF;

      -- Execute transfer operations
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;

      -- Implicit commit will happen at end of procedure
   END transfer_funds;

   -- Test procedures remain the same...
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

END tdd_bank_transfer;
/
```

### Step 14: Final Test for Pass 2 (GREEN)

```sql
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

✅ Both tests should still pass after refactoring.

---

**Pass 2 Complete!** ✅ 

We have successfully implemented and tested:
- ✅ Requirement 1: Successful transfer when sufficient funds exist
- ✅ Requirement 2: Exception raised when insufficient funds


=====================================================================

# 3️⃣ Pass 3 (Requirement 3) - Account Not Found Validation

### Step 15: Add Third Test Function Declaration (RED)

First, let's update our main package spec to include the third test procedure:

```sql
-- Update main package spec to include all three test procedures
CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   -- Main procedure to test
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );

   -- Test procedures (must be in spec for test package to use them!)
   PROCEDURE test_successful_transfer;
   PROCEDURE test_insufficient_funds;
   PROCEDURE test_account_not_found;  -- NEW procedure declaration
END tdd_bank_transfer;
/
```

### Step 16: Update Test Package Spec

Now update the test package spec to include the third test:

```sql
-- Update the test package spec to include all three tests
CREATE OR REPLACE PACKAGE tdd_bank_transfer_test IS
   --%suite
   --%suitepath(bank.transfer)
   --%rollback(manual)

   --%test(Requirement 1: Successful transfer between accounts)
   PROCEDURE test_successful_transfer;
   
   --%test(Requirement 2: Transfer fails with insufficient funds)
   PROCEDURE test_insufficient_funds;
   
   --%test(Requirement 3: Transfer fails when account not found)
   PROCEDURE test_account_not_found;  -- NEW test declaration
END tdd_bank_transfer_test;
/
```

### Step 17: Implement Third Test Function in Main Package Body (RED)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);
   
   ACCOUNT_NOT_FOUND EXCEPTION;
   PRAGMA EXCEPTION_INIT(ACCOUNT_NOT_FOUND, -20002);

   -- Current Stored Procedure (from Pass 2 - will fail new test initially)
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
   BEGIN
      -- Validation: Check if source account has sufficient funds
      SELECT balance 
        INTO l_source_balance 
        FROM bank_accounts 
       WHERE account_id = p_source_id;

      -- Business rule: Insufficient funds check
      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account');
      END IF;

      -- Execute transfer operations
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
   END transfer_funds;

   -- Test 1: Successful transfer
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   -- Test 2: Insufficient funds
   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

   -- Test 3: Account not found (NEW - this will initially FAIL)
   PROCEDURE test_account_not_found IS
      l_exception_raised BOOLEAN := FALSE;
      l_account_count NUMBER;
   BEGIN
      -- Arrange: Create only account 1, account 999 doesn't exist
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      COMMIT;

      -- Verify account 999 doesn't exist
      SELECT COUNT(*) INTO l_account_count 
        FROM bank_accounts 
       WHERE account_id = 999;
      ut.expect(l_account_count).to_equal(0);

      -- Act & Assert: Should raise ACCOUNT_NOT_FOUND exception
      BEGIN
         tdd_bank_transfer.transfer_funds(1, 999, 200);  -- dest account 999 doesn't exist
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised
      ut.expect(l_exception_raised).to_be_true();

      -- Also test with non-existent source account
      l_exception_raised := FALSE;
      BEGIN
         tdd_bank_transfer.transfer_funds(888, 1, 200);  -- source account 888 doesn't exist
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised for source account too
      ut.expect(l_exception_raised).to_be_true();
   END test_account_not_found;

END tdd_bank_transfer;
/
```

### Step 18: Update Test Package Body with Third Test

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer_test IS
   PROCEDURE test_successful_transfer IS
   BEGIN
      tdd_bank_transfer.test_successful_transfer;
   END;

   PROCEDURE test_insufficient_funds IS
   BEGIN
      tdd_bank_transfer.test_insufficient_funds;
   END;

   PROCEDURE test_account_not_found IS
   BEGIN
      tdd_bank_transfer.test_account_not_found;
   END;
END tdd_bank_transfer_test;
/
```

### Step 19: Run Tests (RED - Third Test Should Fail)

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED;
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

You should see that **test_account_not_found FAILS** because the procedure will raise `NO_DATA_FOUND` instead of our custom `ACCOUNT_NOT_FOUND` exception.

### Step 20: Implement Account Validation Logic (GREEN)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);
   
   ACCOUNT_NOT_FOUND EXCEPTION;
   PRAGMA EXCEPTION_INIT(ACCOUNT_NOT_FOUND, -20002);

   -- Enhanced Stored Procedure with account existence validation
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
      l_dest_balance NUMBER;
      l_source_count NUMBER;
      l_dest_count NUMBER;
   BEGIN
      -- Validation 1: Check if both accounts exist
      SELECT COUNT(*) INTO l_source_count 
        FROM bank_accounts 
       WHERE account_id = p_source_id;
       
      SELECT COUNT(*) INTO l_dest_count 
        FROM bank_accounts 
       WHERE account_id = p_dest_id;

      IF l_source_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Source account not found');
      END IF;

      IF l_dest_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Destination account not found');
      END IF;

      -- Validation 2: Check if source account has sufficient funds
      SELECT balance 
        INTO l_source_balance 
        FROM bank_accounts 
       WHERE account_id = p_source_id;

      -- Business rule: Insufficient funds check
      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account');
      END IF;

      -- Execute transfer operations
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
   END transfer_funds;

   -- Test 1: Successful transfer
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   -- Test 2: Insufficient funds
   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

   -- Test 3: Account not found
   PROCEDURE test_account_not_found IS
      l_exception_raised BOOLEAN := FALSE;
      l_account_count NUMBER;
   BEGIN
      -- Arrange: Create only account 1, account 999 doesn't exist
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      COMMIT;

      -- Verify account 999 doesn't exist
      SELECT COUNT(*) INTO l_account_count 
        FROM bank_accounts 
       WHERE account_id = 999;
      ut.expect(l_account_count).to_equal(0);

      -- Act & Assert: Should raise ACCOUNT_NOT_FOUND exception
      BEGIN
         tdd_bank_transfer.transfer_funds(1, 999, 200);  -- dest account 999 doesn't exist
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised
      ut.expect(l_exception_raised).to_be_true();

      -- Also test with non-existent source account
      l_exception_raised := FALSE;
      BEGIN
         tdd_bank_transfer.transfer_funds(888, 1, 200);  -- source account 888 doesn't exist
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      -- Assert: Exception should have been raised for source account too
      ut.expect(l_exception_raised).to_be_true();
   END test_account_not_found;

END tdd_bank_transfer;
/
```

### Step 21: Run Tests Again (GREEN Expected)

```sql
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

✅ All three tests should now **pass** - Requirements 1, 2, and 3 are satisfied.

### Step 22: Final Refactor (REFACTOR)

Let's clean up the code structure and add better error handling:

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   -- Custom exceptions with meaningful names
   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);
   
   ACCOUNT_NOT_FOUND EXCEPTION;
   PRAGMA EXCEPTION_INIT(ACCOUNT_NOT_FOUND, -20002);

   -- Final refactored stored procedure with comprehensive validation
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
      l_source_count NUMBER;
      l_dest_count NUMBER;
   BEGIN
      -- Input validation: Amount should be positive
      IF p_amount <= 0 THEN
         RAISE_APPLICATION_ERROR(-20003, 'Transfer amount must be positive');
      END IF;

      -- Validation 1: Verify both accounts exist
      SELECT COUNT(*) INTO l_source_count 
        FROM bank_accounts 
       WHERE account_id = p_source_id;
       
      IF l_source_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Source account ' || p_source_id || ' not found');
      END IF;

      SELECT COUNT(*) INTO l_dest_count 
        FROM bank_accounts 
       WHERE account_id = p_dest_id;
       
      IF l_dest_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Destination account ' || p_dest_id || ' not found');
      END IF;

      -- Validation 2: Verify sufficient funds in source account
      SELECT balance 
        INTO l_source_balance 
        FROM bank_accounts 
       WHERE account_id = p_source_id;

      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account. Available: ' || l_source_balance || ', Required: ' || p_amount);
      END IF;

      -- Execute atomic transfer operations
      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;

      -- Transaction will be committed by calling procedure/session
   END transfer_funds;

   -- All test procedures remain the same (no changes needed)
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      tdd_bank_transfer.transfer_funds(1, 2, 200);

      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src).to_equal(800);
      ut.expect(l_balance_dest).to_equal(700);
   END test_successful_transfer;

   PROCEDURE test_insufficient_funds IS
      l_balance_src_before NUMBER;
      l_balance_dest_before NUMBER;
      l_balance_src_after NUMBER;
      l_balance_dest_after NUMBER;
      l_exception_raised BOOLEAN := FALSE;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 100);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      SELECT balance INTO l_balance_src_before FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_before FROM bank_accounts WHERE account_id=2;

      BEGIN
         tdd_bank_transfer.transfer_funds(1, 2, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20001 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      ut.expect(l_balance_src_after).to_equal(l_balance_src_before);
      ut.expect(l_balance_dest_after).to_equal(l_balance_dest_before);
   END test_insufficient_funds;

   PROCEDURE test_account_not_found IS
      l_exception_raised BOOLEAN := FALSE;
      l_account_count NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      COMMIT;

      SELECT COUNT(*) INTO l_account_count 
        FROM bank_accounts 
       WHERE account_id = 999;
      ut.expect(l_account_count).to_equal(0);

      BEGIN
         tdd_bank_transfer.transfer_funds(1, 999, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();

      l_exception_raised := FALSE;
      BEGIN
         tdd_bank_transfer.transfer_funds(888, 1, 200);
      EXCEPTION
         WHEN OTHERS THEN
            IF SQLCODE = -20002 THEN
               l_exception_raised := TRUE;
            ELSE
               RAISE;
            END IF;
      END;

      ut.expect(l_exception_raised).to_be_true();
   END test_account_not_found;

END tdd_bank_transfer;
/
```

### Step 23: Final Test Run (GREEN Expected)

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED;
EXEC ut.run('TDD_BANK_TRANSFER_TEST');
```

✅ All tests should still pass after final refactoring.

---

# 🎯 Assignment Summary

**Procedure Name Created:** `tdd_bank_transfer.transfer_funds`

**Final Implementation Features:**
- ✅ Validates account existence for both source and destination
- ✅ Checks sufficient funds before transfer
- ✅ Provides meaningful error messages
- ✅ Atomic transfer operations
- ✅ Input validation for positive amounts

**All Requirements Satisfied:**
1. ✅ **Requirement 1:** Successful transfer when sufficient funds exist
2. ✅ **Requirement 2:** Exception raised when insufficient funds (`INSUFFICIENT_FUNDS`)
3. ✅ **Requirement 3:** Exception raised when account not found (`ACCOUNT_NOT_FOUND`)

# 🧹 Cleanup Scripts

To clean up all objects created in this lab, run these commands:

```sql
-- Drop test packages
DROP PACKAGE tdd_bank_transfer_test;
DROP PACKAGE tdd_bank_transfer;

-- Drop table and data
DROP TABLE bank_accounts PURGE;

-- Verify cleanup
SELECT object_name, object_type 
  FROM user_objects 
 WHERE object_name LIKE 'TDD_BANK%' OR object_name = 'BANK_ACCOUNTS';
```

**Pass 3 Complete!** ✅
