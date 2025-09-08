# Assignment on TDD with Stubs (Banking Example)


***

## **Step 1 (Overview Only)**

**Function:** `calculate_saving_interest(p_principal)`

- **RED:** Write a test expecting `1000 → 40`.
- **GREEN:** Stub saving interest rate as `0.04`. Return `principal × 0.04`.
- **REFACTOR:** Nothing much to clean.

✅ Test should pass.

***

## **Step 2 (Overview Only)**

**Function:** `calculate_fd_interest(p_principal, p_years)`

- **RED:** Write a test expecting `1000 principal, 2 years → 120`.
- **GREEN:** Stub FD interest rate as `0.06`. Return `principal × years × 0.06`.
- **REFACTOR:** Nothing much to clean.

✅ Test should pass.

***

## **Step 3 (Detailed)**

Now we **replace stubbed constants** with values stored in a **configuration table**.
This simulates moving away from "stub values" towards a maintainable design.

***

### **3.1 Prepare Interest Rate Table**

Create the table holding account types and their stubbed interest rates:

```sql
CREATE TABLE tdd_stub_interest_rates (
    account_type VARCHAR2(30) PRIMARY KEY,
    interest_rate NUMBER(5,4)
);

-- Insert stubbed rates
INSERT INTO tdd_stub_interest_rates VALUES ('SAVING', 0.04);
INSERT INTO tdd_stub_interest_rates VALUES ('FD', 0.06);

COMMIT;
```


***

### **3.2 Package Spec**

We extend our package with 3 test functions:

- 2 existing tests
- 1 new test to ensure stub replacement works

```sql
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    -- Test functions
    FUNCTION test_calculate_saving_interest RETURN VARCHAR2;
    FUNCTION test_calculate_fd_interest RETURN VARCHAR2;
    FUNCTION test_lookup_based_interest RETURN VARCHAR2;

    -- Application functions
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER;
    FUNCTION get_interest_rate(p_account_type VARCHAR2) RETURN NUMBER;
END tdd_stub_interest_calc;
/
```


***

### **3.3 RED – Write Test First**

Add a new test verifying rates come from lookup.

```sql
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    
    FUNCTION test_calculate_saving_interest RETURN VARCHAR2 IS
        l_result NUMBER;
    BEGIN
        l_result := calculate_saving_interest(1000);
        IF l_result = 40 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=40, Got=' || l_result;
        END IF;
    END;
    
    FUNCTION test_calculate_fd_interest RETURN VARCHAR2 IS
        l_result NUMBER;
    BEGIN
        l_result := calculate_fd_interest(1000, 2);
        IF l_result = 120 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=120, Got=' || l_result;
        END IF;
    END;

    -- NEW TEST for lookup-based rate usage
    FUNCTION test_lookup_based_interest RETURN VARCHAR2 IS
        l_rate NUMBER;
    BEGIN
        l_rate := get_interest_rate('SAVING');
        
        IF l_rate = 0.04 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=0.04, Got=' || l_rate;
        END IF;
    END;
    
    -- Existing business logic (currently constant, will switch to DB lookup soon)
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.04; -- still stubbed (RED)
    BEGIN
        RETURN p_principal * l_interest_rate;
    END;
    
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.06; -- still stubbed (RED)
    BEGIN
        RETURN p_principal * p_years * l_interest_rate;
    END;

    -- Not implemented yet
    FUNCTION get_interest_rate(p_account_type VARCHAR2) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20010, 'Not implemented yet');
    END;
    
END tdd_stub_interest_calc;
/
```

➡️ Run Test:

```sql
BEGIN
    dbms_output.put_line('Lookup test: ' || tdd_stub_interest_calc.test_lookup_based_interest);
END;
/
```

💥 Will **fail** because `get_interest_rate` not implemented. (Expected → RED)

***

### **3.4 GREEN – Implement DB Lookup**

Now, implement the rate retrieval using our new `tdd_stub_interest_rates` table
and refactor business functions to call it.

```sql
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    
    FUNCTION test_calculate_saving_interest RETURN VARCHAR2 IS
        l_result NUMBER;
    BEGIN
        l_result := calculate_saving_interest(1000);
        IF l_result = 40 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=40, Got=' || l_result;
        END IF;
    END;
    
    FUNCTION test_calculate_fd_interest RETURN VARCHAR2 IS
        l_result NUMBER;
    BEGIN
        l_result := calculate_fd_interest(1000, 2);
        IF l_result = 120 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=120, Got=' || l_result;
        END IF;
    END;

    FUNCTION test_lookup_based_interest RETURN VARCHAR2 IS
        l_rate NUMBER;
    BEGIN
        l_rate := get_interest_rate('SAVING');
        IF l_rate = 0.04 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=0.04, Got=' || l_rate;
        END IF;
    END;
    
    -- Switch to lookup-based implementation
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_interest_rate NUMBER;
    BEGIN
        l_interest_rate := get_interest_rate('SAVING');
        RETURN p_principal * l_interest_rate;
    END;
    
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_interest_rate NUMBER;
    BEGIN
        l_interest_rate := get_interest_rate('FD');
        RETURN p_principal * p_years * l_interest_rate;
    END;

    FUNCTION get_interest_rate(p_account_type VARCHAR2) RETURN NUMBER IS
        l_rate NUMBER;
    BEGIN
        SELECT interest_rate 
        INTO l_rate 
        FROM tdd_stub_interest_rates
        WHERE account_type = p_account_type;

        RETURN l_rate;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN 0; -- default if not found
    END;
    
END tdd_stub_interest_calc;
/
```


***

### **3.5 Run All Tests**

```sql
BEGIN
    dbms_output.put_line('Saving interest: ' || tdd_stub_interest_calc.test_calculate_saving_interest);
    dbms_output.put_line('FD interest: ' || tdd_stub_interest_calc.test_calculate_fd_interest);
    dbms_output.put_line('Lookup test: ' || tdd_stub_interest_calc.test_lookup_based_interest);
END;
/
```

✅ Output should be:

```
Saving interest: PASS
FD interest: PASS
Lookup test: PASS
```


***

### **3.6 REFACTOR**

- Removed stub constants, now pulling rates dynamically.
- Rates live in a table → easy to change without code edits.
- This simulates replacing test doubles/stubs with real dependencies.

***

# 🎓 Final Assignment Summary for Students

1. **Step 1 \& 2:** Build interest calculator using **stubbed constants** (overview only).
2. **Step 3:** Replace stubs with a **rate lookup table** using TDD cycle (detailed).
3. **Tests guide you through RED–GREEN–REFACTOR**.
4. ✅ At the end: flexible interest calculator, tests passing.

***
