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

-- Placeholder trigger to enable first test smooth creation (may fail intentionally)
CREATE OR REPLACE TRIGGER tdd_savings_interest_trigger
    AFTER INSERT ON transactions
    FOR EACH ROW
BEGIN
    NULL; -- placeholder logic
END;
/

-- Insert test accounts
INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1001, 'SAVINGS', 1000.00, 0.0365); -- 3.65% annual interest

INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1002, 'CHECKING', 2000.00, 0.0000); -- No interest

INSERT INTO accounts (account_id, account_type, balance, interest_rate) 
VALUES (1003, 'SAVINGS', 5000.00, 0.0450); -- 4.50% annual interest

COMMIT;

-- Create test package specification
CREATE OR REPLACE PACKAGE tdd_savings_interest_tests AS
    PROCEDURE test_interest_calculation_basic;
    PROCEDURE test_no_interest_for_checking;
    PROCEDURE test_different_interest_rates;
END tdd_savings_interest_tests;
/

-- Create test package body without utPLSQL, using basic PL/SQL with explicit exception handling and output
CREATE OR REPLACE PACKAGE BODY tdd_savings_interest_tests AS

    PROCEDURE assert_equal(p_test_name IN VARCHAR2, p_actual IN NUMBER, p_expected IN NUMBER) IS
    BEGIN
        IF p_actual = p_expected THEN
            DBMS_OUTPUT.PUT_LINE('PASS: ' || p_test_name);
        ELSE
            DBMS_OUTPUT.PUT_LINE('FAIL: ' || p_test_name || ' Expected: ' || p_expected || ' Actual: ' || p_actual);
            ROLLBACK;
            RAISE_APPLICATION_ERROR(-20010, 'Test failed: ' || p_test_name);
        END IF;
    END assert_equal;

    PROCEDURE test_interest_calculation_basic IS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 500.00;
        l_test_name VARCHAR2(100) := 'test_interest_calculation_basic';
    BEGIN
        -- Get initial balance
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1001;

        -- Calculate expected daily interest on new balance
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0365 / 365);

        -- Insert deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1001, 'DEPOSIT', l_deposit_amount);

        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1001;

        -- Assert balance: original + deposit + interest (rounded)
        assert_equal(l_test_name, l_final_balance, ROUND(l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2), 2));

        ROLLBACK;
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('ERROR in ' || l_test_name || ': ' || SQLERRM);
            ROLLBACK;
            RAISE;
    END test_interest_calculation_basic;

    PROCEDURE test_no_interest_for_checking IS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_deposit_amount NUMBER := 300.00;
        l_test_name VARCHAR2(100) := 'test_no_interest_for_checking';
    BEGIN
        -- Get initial balance of checking account
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1002;

        -- Insert deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1002, 'DEPOSIT', l_deposit_amount);

        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1002;

        -- Assert balance: only original + deposit (no interest)
        assert_equal(l_test_name, l_final_balance, ROUND(l_initial_balance + l_deposit_amount, 2));

        ROLLBACK;
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('ERROR in ' || l_test_name || ': ' || SQLERRM);
            ROLLBACK;
            RAISE;
    END test_no_interest_for_checking;

    PROCEDURE test_different_interest_rates IS
        l_initial_balance NUMBER;
        l_final_balance NUMBER;
        l_expected_interest NUMBER;
        l_deposit_amount NUMBER := 1000.00;
        l_test_name VARCHAR2(100) := 'test_different_interest_rates';
    BEGIN
        -- Get initial balance for account 1003
        SELECT balance INTO l_initial_balance 
        FROM accounts WHERE account_id = 1003;

        -- Calculate expected daily interest with 4.50% rate
        l_expected_interest := (l_initial_balance + l_deposit_amount) * (0.0450 / 365);

        -- Insert deposit transaction
        INSERT INTO transactions (transaction_id, account_id, transaction_type, amount)
        VALUES (trans_seq.NEXTVAL, 1003, 'DEPOSIT', l_deposit_amount);

        -- Get final balance
        SELECT balance INTO l_final_balance 
        FROM accounts WHERE account_id = 1003;

        -- Assert balance with interest applied
        assert_equal(l_test_name, l_final_balance, ROUND(l_initial_balance + l_deposit_amount + ROUND(l_expected_interest, 2), 2));

        ROLLBACK;
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('ERROR in ' || l_test_name || ': ' || SQLERRM);
            ROLLBACK;
            RAISE;
    END test_different_interest_rates;

END tdd_savings_interest_tests;
/

-- Run tests manually (call each test one by one)
-- EXEC tdd_savings_interest_tests.test_interest_calculation_basic;
-- EXEC tdd_savings_interest_tests.test_no_interest_for_checking;
-- EXEC tdd_savings_interest_tests.test_different_interest_rates;

-- ------------------------------------------------------------------------------------
-- Minimal trigger implementation with error handling and correct interest calculation
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
    IF :NEW.transaction_type = 'DEPOSIT' AND :NEW.amount > 0 THEN
        BEGIN
            SELECT account_type, interest_rate, balance
            INTO l_account_type, l_interest_rate, l_current_balance
            FROM accounts 
            WHERE account_id = :NEW.account_id;

            l_new_balance := l_current_balance + :NEW.amount;

            IF l_account_type = 'SAVINGS' AND l_interest_rate > 0 THEN
                l_daily_interest := l_new_balance * (l_interest_rate / 365);
                l_new_balance := l_new_balance + ROUND(l_daily_interest, 2);
            END IF;

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

-- ------------------------------------------------------------------------------------
-- Cleanup Scripts
-- Drop the trigger
DROP TRIGGER tdd_savings_interest_trigger;

-- Drop the test package
DROP PACKAGE tdd_savings_interest_tests;

-- Drop tables (will also drop FK constraints)
DROP TABLE transactions;
DROP TABLE accounts;

-- Drop sequence
DROP SEQUENCE trans_seq;

-- Verify cleanup
SELECT object_name, object_type 
FROM user_objects 
WHERE object_name LIKE 'TDD_%' 
   OR object_name IN ('ACCOUNTS', 'TRANSACTIONS', 'TRANS_SEQ');
```