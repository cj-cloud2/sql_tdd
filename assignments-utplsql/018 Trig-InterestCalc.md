# TDD Assignment 001: Banking Domain Trigger Development

## Assignment Overview
This assignment will guide you through developing a database trigger using Test-Driven Development (TDD) methodology. You'll create a trigger that automatically calculates and updates interest on savings accounts when deposits are made.

## Testable Requirements

1. **Interest Calculation**: When a deposit is made to a savings account, the trigger should automatically calculate daily interest based on the new balance
2. **Interest Rate Application**: The interest should be calculated using the account's interest rate (stored as annual percentage)
3. **Balance Update**: The calculated interest should be added to the account balance immediately after the deposit

## Trigger to be Created
**Trigger Name**: `tdd_savings_interest_trigger`

This trigger will fire AFTER INSERT on the transactions table and automatically calculate and apply daily interest for savings account deposits.

---

## Prerequisites Setup

Before starting the TDD cycle, let's create the necessary database objects:

### Step 1: Create Base Tables

```sql
-- Create accounts table
CREATE TABLE accounts (
    account_id NUMBER PRIMARY KEY,
    account_type VARCHAR2(20) NOT NULL,
    balance NUMBER(15,2) DEFAULT 0,
    interest_rate NUMBER(5,4) DEFAULT 0,
    created_date DATE DEFAULT SYSDATE
);

-- Create transactions table
CREATE TABLE transactions (
    transaction_id NUMBER PRIMARY KEY,
    account_id NUMBER NOT NULL,
    transaction_type VARCHAR2(10) NOT NULL,
    amount NUMBER(15,2) NOT NULL,
    transaction_date DATE DEFAULT SYSDATE,
    CONSTRAINT fk_trans_account FOREIGN KEY (account_id) REFERENCES accounts(account_id)
);

-- Create sequence for transactions
CREATE SEQUENCE trans_seq START WITH 1 INCREMENT BY 1;
```

### Step 2: Create Test Data

```sql
-- Insert test accounts
INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1001, 'SAVINGS', 1000.00, 0.0365); -- 3.65% annual interest

INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1002, 'CHECKING', 2000.00, 0.0000); -- No interest

INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1003, 'SAVINGS', 5000.00, 0.0450); -- 4.50% annual interest

COMMIT;
```

---

## TDD Cycle Implementation

## RED-GREEN-REFACTOR Pass 1

### RED Phase 1: Create First Test Function

Create the test package specification:

```sql
CREATE OR REPLACE PACKAGE tdd_savings_interest_tests AS
    -- %suite(Savings Interest Trigger Tests)
    -- %suitepath(banking.triggers)
    
    -- %test(Test 1: Interest calculation for savings account deposit)
    PROCEDURE test_interest_calculation_basic;
    
    -- %test(Test 2: No interest for non-savings accounts)
    PROCEDURE test_no_interest_for_checking;
    
    -- %test(Test 3: Interest calculation with different rates)
    PROCEDURE test_different_interest_rates;
    
END tdd_savings_interest_tests;
/
```

Create the test package body with the first test:

```sql
CREATE OR REPLACE PACKAGE BODY tdd_savings_interest_tests AS
    
    PROCEDURE test_interest_calculation_basic AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 500.00;
    BEGIN
        -- Get initial balance
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Calculate expected daily interest on new balance
        -- Daily interest = (balance + deposit) * (annual_rate / 365)
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0365 / 365);
        
        -- Make deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1001, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Assert that balance includes original + deposit + interest
        ut.expect(l_final_balance).to_equal(
            l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2)
        );
        
        -- Rollback transaction
        ROLLBACK;
    END test_interest_calculation_basic;
    
    PROCEDURE test_no_interest_for_checking AS
    BEGIN
        -- Placeholder for second test
        NULL;
    END test_no_interest_for_checking;
    
    PROCEDURE test_different_interest_rates AS
    BEGIN
        -- Placeholder for third test
        NULL;
    END test_different_interest_rates;
    
END tdd_savings_interest_tests;
/
```

### Run First Test (Should FAIL):

```sql
EXEC ut.run('tdd_savings_interest_tests.test_interest_calculation_basic');
```

**Expected Result**: TEST FAILS because the trigger doesn't exist yet.

### GREEN Phase 1: Create Minimal Trigger Implementation

```sql
CREATE OR REPLACE TRIGGER tdd_savings_interest_trigger
    AFTER INSERT ON transactions
    FOR EACH ROW
DECLARE
    l_account_type VARCHAR2(20);
    l_interest_rate NUMBER;
    l_current_balance NUMBER;
    l_daily_interest NUMBER;
BEGIN
    -- Only process deposits
    IF :NEW.transaction_type = 'DEPOSIT' THEN
        -- Get account details
        SELECT account_type, interest_rate, balance
        INTO l_account_type, l_interest_rate, l_current_balance
        FROM accounts 
        WHERE account_id = :NEW.account_id;
        
        -- Calculate and apply interest only for savings accounts
        IF l_account_type = 'SAVINGS' AND l_interest_rate > 0 THEN
            -- Update balance first with deposit
            UPDATE accounts 
            SET balance = balance + :NEW.amount
            WHERE account_id = :NEW.account_id;
            
            -- Calculate daily interest on new balance
            l_daily_interest := (l_current_balance + :NEW.amount) * (l_interest_rate / 365);
            
            -- Add interest to balance
            UPDATE accounts 
            SET balance = balance + ROUND(l_daily_interest, 2)
            WHERE account_id = :NEW.account_id;
        ELSE
            -- Just update balance for non-savings accounts
            UPDATE accounts 
            SET balance = balance + :NEW.amount
            WHERE account_id = :NEW.account_id;
        END IF;
    END IF;
END;
/
```

### Run First Test Again (Should PASS):

```sql
EXEC ut.run('tdd_savings_interest_tests.test_interest_calculation_basic');
```

**Expected Result**: TEST PASSES

### REFACTOR Phase 1

The current implementation works but can be optimized. For now, let's proceed to the next test.

---

## RED-GREEN-REFACTOR Pass 2

### RED Phase 2: Implement Second Test Function

Update the test package body to include the second test:

```sql
CREATE OR REPLACE PACKAGE BODY tdd_savings_interest_tests AS
    
    PROCEDURE test_interest_calculation_basic AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 500.00;
    BEGIN
        -- Get initial balance
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Calculate expected daily interest on new balance
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0365 / 365);
        
        -- Make deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1001, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Assert that balance includes original + deposit + interest
        ut.expect(l_final_balance).to_equal(
            l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2)
        );
        
        -- Rollback transaction
        ROLLBACK;
    END test_interest_calculation_basic;
    
    PROCEDURE test_no_interest_for_checking AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_deposit_amount NUMBER := 300.00;
    BEGIN
        -- Get initial balance of checking account
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1002;
        
        -- Make deposit to checking account
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1002, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1002;
        
        -- Assert that balance only includes original + deposit (no interest)
        ut.expect(l_final_balance).to_equal(l_initial_balance + l_deposit_amount);
        
        -- Rollback transaction
        ROLLBACK;
    END test_no_interest_for_checking;
    
    PROCEDURE test_different_interest_rates AS
    BEGIN
        -- Placeholder for third test
        NULL;
    END test_different_interest_rates;
    
END tdd_savings_interest_tests;
/
```

### Run Second Test (Should PASS):

```sql
EXEC ut.run('tdd_savings_interest_tests.test_no_interest_for_checking');
```

**Expected Result**: TEST PASSES (our current trigger already handles this case)

### GREEN Phase 2

The test already passes with our current implementation. No changes needed.

### REFACTOR Phase 2

Let's optimize the trigger to reduce code duplication:

```sql
CREATE OR REPLACE TRIGGER tdd_savings_interest_trigger
    AFTER INSERT ON transactions
    FOR EACH ROW
DECLARE
    l_account_type VARCHAR2(20);
    l_interest_rate NUMBER;
    l_current_balance NUMBER;
    l_daily_interest NUMBER;
    l_new_balance NUMBER;
BEGIN
    -- Only process deposits
    IF :NEW.transaction_type = 'DEPOSIT' THEN
        -- Get account details
        SELECT account_type, interest_rate, balance
        INTO l_account_type, l_interest_rate, l_current_balance
        FROM accounts 
        WHERE account_id = :NEW.account_id;
        
        -- Calculate new balance after deposit
        l_new_balance := l_current_balance + :NEW.amount;
        
        -- Calculate interest for savings accounts
        IF l_account_type = 'SAVINGS' AND l_interest_rate > 0 THEN
            l_daily_interest := l_new_balance * (l_interest_rate / 365);
            l_new_balance := l_new_balance + ROUND(l_daily_interest, 2);
        END IF;
        
        -- Update account balance once
        UPDATE accounts 
        SET balance = l_new_balance
        WHERE account_id = :NEW.account_id;
    END IF;
END;
/
```

### Run All Tests After Refactoring:

```sql
EXEC ut.run('tdd_savings_interest_tests');
```

**Expected Result**: Both tests should still pass.

---

## RED-GREEN-REFACTOR Pass 3

### RED Phase 3: Implement Third Test Function

Update the test package body to include the third test:

```sql
CREATE OR REPLACE PACKAGE BODY tdd_savings_interest_tests AS
    
    PROCEDURE test_interest_calculation_basic AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 500.00;
    BEGIN
        -- Get initial balance
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Calculate expected daily interest on new balance
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0365 / 365);
        
        -- Make deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1001, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1001;
        
        -- Assert that balance includes original + deposit + interest
        ut.expect(l_final_balance).to_equal(
            l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2)
        );
        
        -- Rollback transaction
        ROLLBACK;
    END test_interest_calculation_basic;
    
    PROCEDURE test_no_interest_for_checking AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_deposit_amount NUMBER := 300.00;
    BEGIN
        -- Get initial balance of checking account
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1002;
        
        -- Make deposit to checking account
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1002, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1002;
        
        -- Assert that balance only includes original + deposit (no interest)
        ut.expect(l_final_balance).to_equal(l_initial_balance + l_deposit_amount);
        
        -- Rollback transaction
        ROLLBACK;
    END test_no_interest_for_checking;
    
    PROCEDURE test_different_interest_rates AS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 1000.00;
    BEGIN
        -- Test with account 1003 which has 4.50% interest rate
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1003;
        
        -- Calculate expected daily interest with 4.50% rate
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0450 / 365);
        
        -- Make deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1003, 'DEPOSIT', l_deposit_amount);
        
        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1003;
        
        -- Assert correct interest calculation with different rate
        ut.expect(l_final_balance).to_equal(
            l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2)
        );
        
        -- Rollback transaction
        ROLLBACK;
    END test_different_interest_rates;
    
END tdd_savings_interest_tests;
/
```

### Run Third Test (Should PASS):

```sql
EXEC ut.run('tdd_savings_interest_tests.test_different_interest_rates');
```

**Expected Result**: TEST PASSES (our current trigger should handle different interest rates)

### GREEN Phase 3

The test passes with our current implementation. No changes needed.

### REFACTOR Phase 3

Let's add some error handling and make the trigger more robust:

```sql
CREATE OR REPLACE TRIGGER tdd_savings_interest_trigger
    AFTER INSERT ON transactions
    FOR EACH ROW
DECLARE
    l_account_type VARCHAR2(20);
    l_interest_rate NUMBER;
    l_current_balance NUMBER;
    l_daily_interest NUMBER;
    l_new_balance NUMBER;
BEGIN
    -- Only process deposits
    IF :NEW.transaction_type = 'DEPOSIT' AND :NEW.amount > 0 THEN
        BEGIN
            -- Get account details
            SELECT account_type, interest_rate, balance
            INTO l_account_type, l_interest_rate, l_current_balance
            FROM accounts 
            WHERE account_id = :NEW.account_id;
            
            -- Calculate new balance after deposit
            l_new_balance := l_current_balance + :NEW.amount;
            
            -- Calculate interest for savings accounts
            IF l_account_type = 'SAVINGS' AND l_interest_rate > 0 THEN
                l_daily_interest := l_new_balance * (l_interest_rate / 365);
                l_new_balance := l_new_balance + ROUND(l_daily_interest, 2);
            END IF;
            
            -- Update account balance
            UPDATE accounts 
            SET balance = l_new_balance
            WHERE account_id = :NEW.account_id;
            
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                RAISE_APPLICATION_ERROR(-20001, 'Account not found: ' || :NEW.account_id);
            WHEN OTHERS THEN
                RAISE_APPLICATION_ERROR(-20002, 'Error processing interest: ' || SQLERRM);
        END;
    END IF;
END;
/
```

### Final Test Run - All Tests:

```sql
EXEC ut.run('tdd_savings_interest_tests');
```

**Expected Result**: All three tests should pass.

---

## Manual Testing

Let's verify our trigger works correctly with some manual tests:

### Test 1: Savings Account Deposit

```sql
-- Check current balance
SELECT account_id, balance FROM accounts WHERE account_id = 1001;

-- Make a deposit
INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
VALUES (trans_seq.NEXTVAL, 1001, 'DEPOSIT', 100.00);

-- Check new balance (should include interest)
SELECT account_id, balance FROM accounts WHERE account_id = 1001;

ROLLBACK;
```

### Test 2: Checking Account Deposit

```sql
-- Check current balance
SELECT account_id, balance FROM accounts WHERE account_id = 1002;

-- Make a deposit
INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
VALUES (trans_seq.NEXTVAL, 1002, 'DEPOSIT', 100.00);

-- Check new balance (should NOT include interest)
SELECT account_id, balance FROM accounts WHERE account_id = 1002;

ROLLBACK;
```

---

## Complete Test Suite Execution

Run the entire test suite to ensure everything works:

```sql
-- Run all tests
EXEC ut.run('tdd_savings_interest_tests');

-- Run with detailed output
SELECT * FROM TABLE(ut.run('tdd_savings_interest_tests'));
```

---

## Cleanup Scripts

Execute these scripts to clean up all objects created during this assignment:

```sql
-- Drop the trigger
DROP TRIGGER tdd_savings_interest_trigger;

-- Drop the test package
DROP PACKAGE tdd_savings_interest_tests;

-- Drop the tables (this will also drop the foreign key constraint)
DROP TABLE transactions;
DROP TABLE accounts;

-- Drop the sequence
DROP SEQUENCE trans_seq;

-- Verify cleanup
SELECT object_name, object_type 
FROM user_objects 
WHERE object_name LIKE 'TDD_%' 
   OR object_name IN ('ACCOUNTS', 'TRANSACTIONS', 'TRANS_SEQ');
```

---

## Summary

This assignment successfully demonstrated the TDD approach for trigger development with three complete RED-GREEN-REFACTOR cycles:

1. **Pass 1**: Basic interest calculation for savings accounts
2. **Pass 2**: Verification that checking accounts don't receive interest  
3. **Pass 3**: Testing different interest rates work correctly

The final trigger automatically calculates and applies daily interest to savings accounts when deposits are made, while leaving other account types unaffected. Each test function guided the development process and ensured the trigger meets all specified requirements.

**Key Learning Points**:
- TDD helps ensure code correctness through systematic testing
- Each test drives the minimum implementation needed
- Refactoring improves code quality while maintaining functionality
- Comprehensive test coverage provides confidence in the solution