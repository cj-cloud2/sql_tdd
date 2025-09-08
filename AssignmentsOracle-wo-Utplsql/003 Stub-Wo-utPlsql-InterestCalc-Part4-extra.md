# Assignment 2: TDD with utPLSQL (Banking Interest Calculator)


***

## 📚 Prerequisites

1. **utPLSQL Framework Installed** in Oracle 23ai Free.
2. Students run tests using:

```sql
begin
   ut.run('tdd_stub_interest_calc_ut');
end;
/
```


***

## **Step 1 – Saving Interest with Stub**

### Function to Implement

`calculate_saving_interest(p_principal NUMBER)`

- Stub rate = `0.04`.
- Formula: `interest = principal × 0.04`.

***

### **RED – Write Test First**

- Create **package for application**
- Create **test package with utPLSQL tests**

```sql
-- Application Package Spec
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
END tdd_stub_interest_calc;
/

-- Application Package Body (stubbed not implemented yet)
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20001, 'Not yet implemented');
    END;
END tdd_stub_interest_calc;
/

-- Test Package Spec
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc_ut IS
END tdd_stub_interest_calc_ut;
/

-- Test Package Body
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc_ut IS
    -- utPLSQL Annotation
    --%suite(Saving Interest Tests)
    
    --%test(Returns 40 when principal = 1000 at 4%)
    PROCEDURE test_calculate_saving_interest IS
       l_result NUMBER;
    BEGIN
        l_result := tdd_stub_interest_calc.calculate_saving_interest(1000);
        ut.expect(l_result).to_equal(40);
    END;
END tdd_stub_interest_calc_ut;
/
```

👉 Run:

```sql
BEGIN
   ut.run('tdd_stub_interest_calc_ut');
END;
/
```

❌ Test Fails → RED.

***

### **GREEN – Implement Stubbed Rate**

```sql
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.04;
    BEGIN
        RETURN p_principal * l_rate;
    END;
END tdd_stub_interest_calc;
/
```

✅ Run Tests → Should pass.

***

### **REFACTOR**

- Code clean. Stub rate still hardcoded.

***

***

## **Step 2 – FD Interest with Stub**

### Function to Implement

`calculate_fd_interest(p_principal NUMBER, p_years NUMBER)`

- Stubbed FD rate = `0.06`.
- Formula: `interest = principal × years × 0.06`.

***

### **RED – Add Test**

Extend application and test packages.

```sql
-- Extend Application Spec
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER;
END tdd_stub_interest_calc;
/

-- Extend Application Body
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.04;
    BEGIN
        RETURN p_principal * l_rate;
    END;

    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20002, 'Not yet implemented');
    END;
END tdd_stub_interest_calc;
/

-- Extend Test Package
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc_ut IS
    --%suite(Saving + FD Interest Tests)
    
    --%test(Saving Interest 1000 → 40)
    PROCEDURE test_calculate_saving_interest IS
       l_result NUMBER;
    BEGIN
        l_result := tdd_stub_interest_calc.calculate_saving_interest(1000);
        ut.expect(l_result).to_equal(40);
    END;
    
    --%test(FD Interest: 1000 principal, 2 years → 120 at 6%)
    PROCEDURE test_calculate_fd_interest IS
       l_result NUMBER;
    BEGIN
        l_result := tdd_stub_interest_calc.calculate_fd_interest(1000, 2);
        ut.expect(l_result).to_equal(120);
    END;
END tdd_stub_interest_calc_ut;
/
```

👉 Run:

```sql
BEGIN
   ut.run('tdd_stub_interest_calc_ut');
END;
/
```

❌ New test fails → RED.

***

### **GREEN – Implement FD Function**

```sql
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.04;
    BEGIN
        RETURN p_principal * l_rate;
    END;

    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.06;
    BEGIN
        RETURN p_principal * p_years * l_rate;
    END;
END tdd_stub_interest_calc;
/
```

✅ Run again → Both tests should pass.

***

### **REFACTOR**

- Both rates still stub constants.

***

***

## **Step 3 – Replace Stubs with Lookup Table**

### Requirement

Read rates for **SAVING** and **FD** from config table instead of constants.
Table: `tdd_stub_interest_rates(account_type, interest_rate)`

***

### **3.1 Create Table**

```sql
CREATE TABLE tdd_stub_interest_rates (
    account_type VARCHAR2(30) PRIMARY KEY,
    interest_rate NUMBER(5,4)
);

INSERT INTO tdd_stub_interest_rates VALUES ('SAVING', 0.04);
INSERT INTO tdd_stub_interest_rates VALUES ('FD', 0.06);
COMMIT;
```


***

### **RED – Add Test**

Update UT package to add a lookup test.

```sql
-- Extend Application Spec
CREATE OR REPLACE PACKAGE tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER;
    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER;
    FUNCTION get_interest_rate(p_account_type VARCHAR2) RETURN NUMBER;
END tdd_stub_interest_calc;
/

-- Extend App Body (not implemented yet)
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.04;
    BEGIN
        RETURN p_principal * l_rate;
    END;

    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_rate CONSTANT NUMBER := 0.06;
    BEGIN
        RETURN p_principal * p_years * l_rate;
    END;

    FUNCTION get_interest_rate(p_account_type VARCHAR2) RETURN NUMBER IS
    BEGIN
        RAISE_APPLICATION_ERROR(-20003, 'Not yet implemented');
    END;
END tdd_stub_interest_calc;
/

-- Extend UT Package with DB lookup test
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc_ut IS
    --%suite(Interest Calculator with Lookup)
    
    --%test(Saving Interest 1000 → 40)
    PROCEDURE test_calculate_saving_interest IS
       l_result NUMBER;
    BEGIN
        l_result := tdd_stub_interest_calc.calculate_saving_interest(1000);
        ut.expect(l_result).to_equal(40);
    END;
    
    --%test(FD Interest 1000 × 2 years → 120)
    PROCEDURE test_calculate_fd_interest IS
       l_result NUMBER;
    BEGIN
        l_result := tdd_stub_interest_calc.calculate_fd_interest(1000, 2);
        ut.expect(l_result).to_equal(120);
    END;

    --%test(Lookup SAVING rate → 0.04)
    PROCEDURE test_lookup_saving_rate IS
       l_rate NUMBER;
    BEGIN
        l_rate := tdd_stub_interest_calc.get_interest_rate('SAVING');
        ut.expect(l_rate).to_equal(0.04);
    END;
END tdd_stub_interest_calc_ut;
/
```

👉 Run:

```sql
BEGIN
   ut.run('tdd_stub_interest_calc_ut');
END;
/
```

❌ Lookup test fails → RED.

***

### **GREEN – Implement with Table Lookup**

```sql
CREATE OR REPLACE PACKAGE BODY tdd_stub_interest_calc AS
    FUNCTION calculate_saving_interest(p_principal NUMBER) RETURN NUMBER IS
        l_rate NUMBER;
    BEGIN
        l_rate := get_interest_rate('SAVING');
        RETURN p_principal * l_rate;
    END;

    FUNCTION calculate_fd_interest(p_principal NUMBER, p_years NUMBER) RETURN NUMBER IS
        l_rate NUMBER;
    BEGIN
        l_rate := get_interest_rate('FD');
        RETURN p_principal * p_years * l_rate;
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
            RETURN 0;
    END;
END tdd_stub_interest_calc;
/
```

✅ Run:

```sql
BEGIN
   ut.run('tdd_stub_interest_calc_ut');
END;
/
```

All tests should pass 🎉

***

### **REFACTOR**

- Constants removed, replaced by **DB lookup**.
- Assignment now mirrors production-style architecture.

***

# 🧹 Cleanup Script

```sql
BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE BODY tdd_stub_interest_calc';
EXCEPTION WHEN OTHERS THEN IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE tdd_stub_interest_calc';
EXCEPTION WHEN OTHERS THEN IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE BODY tdd_stub_interest_calc_ut';
EXCEPTION WHEN OTHERS THEN IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE tdd_stub_interest_calc_ut';
EXCEPTION WHEN OTHERS THEN IF SQLCODE != -4043 THEN RAISE; END IF;
END;
/
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE tdd_stub_interest_rates';
EXCEPTION WHEN OTHERS THEN IF SQLCODE != -942 THEN RAISE; END IF;
END;
/
```


***

✅ Now  you experience **Step 1 \& 2 with stubs**, and **Step 3 with real lookup**, this time fully automated with utPLSQL.

***