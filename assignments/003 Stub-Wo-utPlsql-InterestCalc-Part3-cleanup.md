# 🧹 Cleanup Script for Assignment

```sql
-- Drop Package Body and Spec
BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE BODY tdd_stub_interest_calc';
EXCEPTION
   WHEN OTHERS THEN
      IF SQLCODE != -4043 THEN -- ignore "package body does not exist"
         RAISE;
      END IF;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP PACKAGE tdd_stub_interest_calc';
EXCEPTION
   WHEN OTHERS THEN
      IF SQLCODE != -4043 THEN -- ignore "package does not exist"
         RAISE;
      END IF;
END;
/

-- Drop the Interest Rates Table
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE tdd_stub_interest_rates';
EXCEPTION
   WHEN OTHERS THEN
      IF SQLCODE != -942 THEN -- ignore "table does not exist"
         RAISE;
      END IF;
END;
/
```


***

### 🔍 What this does:

- Drops **`tdd_stub_interest_calc`** package (spec \& body).
- Drops **`tdd_stub_interest_rates`** table.
- Uses exception handling so the script **does not fail** if the objects don’t exist (clean re-runs).

***