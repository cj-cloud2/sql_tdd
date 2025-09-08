# 🧹 Cleanup Script for `tdd_fake_loan_eligibility` Assignment

```sql
-- =========================================
-- Cleanup Script: Loan Eligibility TDD Lab
-- Will drop test packages and implementation
-- =========================================

-- 1. Drop test package body
begin
  execute immediate 'drop package body tdd_fake_loan_eligibility_test';
exception
  when others then
    if sqlcode != -4043 then -- package does not exist
      raise;
    end if;
end;
/

-- 2. Drop test package
begin
  execute immediate 'drop package tdd_fake_loan_eligibility_test';
exception
  when others then
    if sqlcode != -4043 then
      raise;
    end if;
end;
/

-- 3. Drop implementation package body
begin
  execute immediate 'drop package body tdd_fake_loan_eligibility';
exception
  when others then
    if sqlcode != -4043 then
      raise;
    end if;
end;
/

-- 4. Drop implementation package
begin
  execute immediate 'drop package tdd_fake_loan_eligibility';
exception
  when others then
    if sqlcode != -4043 then
      raise;
    end if;
end;
/

-- =========================================
-- Verification (should return no rows)
-- =========================================
select object_name, object_type
from user_objects
where object_name like 'TDD_FAKE_LOAN_ELIGIBILITY%'
order by object_type;
```


***

### ✅ Key Notes for Students:

- Run this script at the end of the assignment or when code gets messy.
- It **drops the test package** and **implementation package** but handles cases where objects don’t exist (so it won’t break if already cleaned).
- At the end, they can verify with `select` — it should return **no objects** with that prefix.

***