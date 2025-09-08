#  Assignment Procedures 001  Test-Driven Development (TDD) without utPLSQL (Difficulty Level 3)

##  🎯 Problem Context: Banking Domain

We want to develop a stored procedure that **transfers money between two bank accounts**.

##  ✅ Testable Requirements

1. **Requirement 1**: If source account has enough balance, the transfer should deduct from source and credit the destination account.
2. **Requirement 2**: If source account does **not** have enough balance, the transfer should **raise an exception** `INSUFFICIENT_FUNDS`.
3. **Requirement 3**: If either account ID does not exist, raise an exception `ACCOUNT_NOT_FOUND`.

##  📦 Package Naming Convention

All DB objects will be created inside a package named:

```
tdd_bank_transfer
```

#  🛠 Step-by-Step Guide (RED–GREEN–REFACTOR)

We will follow TDD:

1. Write **failing test (RED)**
2. Implement minimal code to make it pass (**GREEN**)
3. Clean/refactor code (**REFACTOR**)

We will do **3 passes**, one per requirement.

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

-- Placeholder view for smooth test package creation
CREATE OR REPLACE VIEW v_bank_accounts AS SELECT * FROM bank_accounts;
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

   -- Test procedure declarations
   PROCEDURE test_successful_transfer;
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

   -- Requirement 1 test procedure
   PROCEDURE test_successful_transfer IS
      l_balance_src NUMBER;
      l_balance_dest NUMBER;
   BEGIN
      -- Arrange - reset balances
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      INSERT INTO bank_accounts VALUES (2, 500);
      COMMIT;

      -- Act: Calls stub
      tdd_bank_transfer.transfer_funds(1, 2, 200);

      -- Assert: This will FAIL for now
      SELECT balance INTO l_balance_src FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest FROM bank_accounts WHERE account_id=2;

      IF l_balance_src != 800 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance incorrect');
      END IF;

      IF l_balance_dest != 700 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance incorrect');
      END IF;
   END test_successful_transfer;

END tdd_bank_transfer;
/
```

***

### Step 3: Run First Test (RED - should fail)

```sql
BEGIN
   tdd_bank_transfer.test_successful_transfer;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
/
```

***

### Step 4: Implement Procedure (GREEN)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

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

      IF l_balance_src != 800 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance incorrect');
      END IF;

      IF l_balance_dest != 700 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance incorrect');
      END IF;
   END test_successful_transfer;

END tdd_bank_transfer;
/
```

***

### Step 5: Run First Test Again (GREEN - should pass)

```sql
BEGIN
   tdd_bank_transfer.test_successful_transfer;
   DBMS_OUTPUT.PUT_LINE('Test 1 passed');
END;
/
```

***

# 2️⃣ Pass 2 (Requirement 2) - Insufficient Funds Check

### Step 6: Update Package Spec with second test

```sql
CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );

   PROCEDURE test_successful_transfer;
   PROCEDURE test_insufficient_funds;
END tdd_bank_transfer;
/
```

***

### Step 7: Implement Second Test and Procedure Logic (RED and GREEN)

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);

   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
   BEGIN
      SELECT balance INTO l_source_balance FROM bank_accounts WHERE account_id = p_source_id;

      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account');
      END IF;

      UPDATE bank_accounts
         SET balance = balance - p_amount
       WHERE account_id = p_source_id;

      UPDATE bank_accounts
         SET balance = balance + p_amount
       WHERE account_id = p_dest_id;
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

      IF l_balance_src != 800 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance incorrect');
      END IF;

      IF l_balance_dest != 700 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance incorrect');
      END IF;
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

      IF NOT l_exception_raised THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Insufficient funds exception not raised');
      END IF;

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      IF l_balance_src_after != l_balance_src_before THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance changed despite insufficient funds');
      END IF;

      IF l_balance_dest_after != l_balance_dest_before THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance changed despite insufficient funds');
      END IF;
   END test_insufficient_funds;

END tdd_bank_transfer;
/
```

***

### Step 8: Run Both Tests

```sql
BEGIN
   tdd_bank_transfer.test_successful_transfer;
   tdd_bank_transfer.test_insufficient_funds;
   DBMS_OUTPUT.PUT_LINE('Tests 1 and 2 passed');
END;
/
```

***

# 3️⃣ Pass 3 (Requirement 3) - Account Not Found Validation

### Step 9: Update Package Spec with third test

```sql
CREATE OR REPLACE PACKAGE tdd_bank_transfer IS
   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   );

   PROCEDURE test_successful_transfer;
   PROCEDURE test_insufficient_funds;
   PROCEDURE test_account_not_found;
END tdd_bank_transfer;
/
```

***

### Step 10: Implement Third Test and Procedure Logic

```sql
CREATE OR REPLACE PACKAGE BODY tdd_bank_transfer IS

   INSUFFICIENT_FUNDS EXCEPTION;
   PRAGMA EXCEPTION_INIT(INSUFFICIENT_FUNDS, -20001);

   ACCOUNT_NOT_FOUND EXCEPTION;
   PRAGMA EXCEPTION_INIT(ACCOUNT_NOT_FOUND, -20002);

   PROCEDURE transfer_funds (
      p_source_id IN NUMBER,
      p_dest_id   IN NUMBER,
      p_amount    IN NUMBER
   ) IS
      l_source_balance NUMBER;
      l_source_count NUMBER;
      l_dest_count NUMBER;
   BEGIN
      IF p_amount <= 0 THEN
         RAISE_APPLICATION_ERROR(-20003, 'Transfer amount must be positive');
      END IF;

      SELECT COUNT(*) INTO l_source_count FROM bank_accounts WHERE account_id = p_source_id;
      IF l_source_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Source account ' || p_source_id || ' not found');
      END IF;

      SELECT COUNT(*) INTO l_dest_count FROM bank_accounts WHERE account_id = p_dest_id;
      IF l_dest_count = 0 THEN
         RAISE_APPLICATION_ERROR(-20002, 'Destination account ' || p_dest_id || ' not found');
      END IF;

      SELECT balance INTO l_source_balance FROM bank_accounts WHERE account_id = p_source_id;
      IF l_source_balance < p_amount THEN
         RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in source account. Available: ' || l_source_balance || ', Required: ' || p_amount);
      END IF;

      UPDATE bank_accounts SET balance = balance - p_amount WHERE account_id = p_source_id;
      UPDATE bank_accounts SET balance = balance + p_amount WHERE account_id = p_dest_id;
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

      IF l_balance_src != 800 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance incorrect');
      END IF;

      IF l_balance_dest != 700 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance incorrect');
      END IF;
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

      IF NOT l_exception_raised THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Insufficient funds exception not raised');
      END IF;

      SELECT balance INTO l_balance_src_after FROM bank_accounts WHERE account_id=1;
      SELECT balance INTO l_balance_dest_after FROM bank_accounts WHERE account_id=2;

      IF l_balance_src_after != l_balance_src_before THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source balance changed despite insufficient funds');
      END IF;

      IF l_balance_dest_after != l_balance_dest_before THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination balance changed despite insufficient funds');
      END IF;
   END test_insufficient_funds;

   PROCEDURE test_account_not_found IS
      l_exception_raised BOOLEAN := FALSE;
      l_account_count NUMBER;
   BEGIN
      EXECUTE IMMEDIATE 'DELETE FROM bank_accounts';
      INSERT INTO bank_accounts VALUES (1, 1000);
      COMMIT;

      SELECT COUNT(*) INTO l_account_count FROM bank_accounts WHERE account_id = 999;

      IF l_account_count != 0 THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test environment failure: Account 999 should not exist');
      END IF;

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

      IF NOT l_exception_raised THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Destination account not found exception not raised');
      END IF;

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

      IF NOT l_exception_raised THEN
         RAISE_APPLICATION_ERROR(-20010, 'Test failed: Source account not found exception not raised');
      END IF;
   END test_account_not_found;

END tdd_bank_transfer;
/
```

***

### Step 11: Run All Tests

```sql
BEGIN
   tdd_bank_transfer.test_successful_transfer;
   tdd_bank_transfer.test_insufficient_funds;
   tdd_bank_transfer.test_account_not_found;
   DBMS_OUTPUT.PUT_LINE('All tests passed');
END;
/
```

***

#  🧹 Cleanup Scripts

```sql
DROP PACKAGE tdd_bank_transfer;
DROP TABLE bank_accounts PURGE;
DROP VIEW v_bank_accounts;
```

***

This completes the detailed TDD assignment in PL/SQL for a banking domain money transfer procedure with 3 test functions, converted to manual test assertions and exceptions in plain PL/SQL without utPLSQL. The RED-GREEN-REFACTOR cycles are followed through 3 passes, and cleanup scripts are provided at the end.

Students can copy and paste these blocks step-by-step in their SQL IDE to learn test-driven development fundamentals in PL/SQL with manual test validation. 

Each test procedure raises errors on assertion failures, or outputs success messages on passing tests, enabling iterative development and verification. 

***
