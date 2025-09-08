# Assignment on TDD with Stubs

*Language: PL/SQL (Oracle 23ai Free Release)*
*Packaging Standard: All objects start with `tdd_stub_interest_*`*

***

## **Scenario**

You are tasked with building a **simple banking system** component to calculate interest.
However, the **source of interest rates** is not yet implemented (external service).
So, you must **stub** the interest rate provider and use **Test-Driven Development (TDD)** to build the package step by step.

- In our exercise, the "stub" will return fixed interest rates.
- Students will write tests first, then implement logic until **GREEN**.
- At the end of two cycles (for two functions), they will have a working interest calculator package.

***

# 🚦 Step 1: First TDD Cycle (Function 1)

### Function to Implement:

`calculate_saving_interest(p_principal NUMBER)`

- Uses **stub interest rate for saving account = 4%**.
- Formula: `interest = principal * 0.04`

***

## **RED – Write the Test First**

We don’t have any implementation yet.

```sql
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    -- Student test functions (manual test harness)
    FUNCTION test_calculate_saving_interest RETURN VARCHAR2;
    
    -- Application functions (to be implemented later)
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
END tdd_stub_interest_calc;
/

CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    
    -- TEST: For Saving Account, 1000 principal should give 40 interest
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
    
    -- Stub: Just raise not implemented now
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20001, 'Not implemented yet');
    END;
    
END tdd_stub_interest_calc;
/
```


### Run the Test

```sql
BEGIN
    dbms_output.put_line(tdd_stub_interest_calc.test_calculate_saving_interest);
END;
/
```

👉 This will **FAIL** (RED) since function raises exception.

***

## **GREEN – Implement with Stub**

Now, implement with fixed stub rate = 4%.

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
    
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.04; -- stub value
    BEGIN
        RETURN p_principal * l_interest_rate;
    END;
    
END tdd_stub_interest_calc;
/
```

✅ Run Test – Should output `PASS`.

***

## **REFACTOR**

- Code is simple. No cleanup needed.
- But note: `l_interest_rate` is a stub constant (hardcoded for now).
- Later, we could replace it with external service.

***

# 🚦 Step 2: Second TDD Cycle (Function 2)

### Function to Implement:

`calculate_fd_interest(p_principal NUMBER, p_years NUMBER)`

- Fixed stub rate for FD = **6%**.
- Formula: `interest = principal × years × interest_rate`.

***

## **RED – Write Test First**

Add second test:

```sql
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    FUNCTION test_calculate_saving_interest RETURN VARCHAR2;
    FUNCTION test_calculate_fd_interest RETURN VARCHAR2;
    
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER;
END tdd_stub_interest_calc;
/

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
    
    -- Test for FD Interest
    FUNCTION test_calculate_fd_interest RETURN VARCHAR2 IS
        l_result NUMBER;
    BEGIN
        -- Expectation: 1000 principal, 2 years, 6% = 120
        l_result := calculate_fd_interest(1000, 2);
        
        IF l_result = 120 THEN
            RETURN 'PASS';
        ELSE
            RETURN 'FAIL: Expected=120, Got=' || l_result;
        END IF;
    END;
    
    -- Implemented function
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.04; -- stubbed
    BEGIN
        RETURN p_principal * l_interest_rate;
    END;
    
    -- Not implemented yet (RED)
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20002, 'Not implemented yet');
    END;
    
END tdd_stub_interest_calc;
/
```


### Run test:

```sql
BEGIN
    dbms_output.put_line(tdd_stub_interest_calc.test_calculate_fd_interest);
END;
/
```

👉 This will **FAIL** (RED).

***

## **GREEN – Implement Now**

Implement FD function with stub 6%.

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
    
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.04;
    BEGIN
        RETURN p_principal * l_interest_rate;
    END;
    
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_interest_rate CONSTANT NUMBER := 0.06;
    BEGIN
        RETURN p_principal * p_years * l_interest_rate;
    END;
    
END tdd_stub_interest_calc;
/
```

✅ Run tests again:

```sql
BEGIN
    dbms_output.put_line('Saving interest: ' || tdd_stub_interest_calc.test_calculate_saving_interest);
    dbms_output.put_line('FD interest: ' || tdd_stub_interest_calc.test_calculate_fd_interest);
END;
/
```

Output should be:

```
Saving interest: PASS
FD interest: PASS
```


***

## **REFACTOR**

- Both rates are stubbed with fixed values.
- In real-world, these would come from an external service/config table.
- For now, assignment ends here.

***

# 🎓 Assignment Summary for Students

1. Follow **RED-GREEN-REFACTOR** twice:
    - 1st Pass: Saving Interest Function
    - 2nd Pass: FD Interest Function
2. Use **stubs** for interest rates.
3. Tests are manual functions returning `PASS/FAIL`.
4. At the end: Both functions tested successfully.

***