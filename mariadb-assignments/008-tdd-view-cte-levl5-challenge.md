# TDD Challenge LAB: Deposit Account Balance Report View with CTE

---

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 5 (Expert)
**Domain:** Deposit Accounts Banking-Reports Module
**Objective:** Implement a view that generates deposit account balance reports using CTEs and TDD.

---

## Business Requirements

- BR001: Show current balances, account status, and last transaction details for active deposit accounts.
- BR002: Calculate interest earned and fees charged per account type.
- BR003: Provide aggregated transaction summaries: deposits, withdrawals, and activity patterns.

---

## Testable Requirements & Passes

### Pass 1: TR001 – Basic Account Balance Calculation

RED Phase:
- Create a placeholder view that returns a single column `message` and write a test expecting four columns: `account_id`, `account_number`, `account_name`, `current_balance`. The test should fail initially.

GREEN Phase – Tasks (replace descriptions with SQL):
- Task 1.1: Select `account_id`, `account_number`, `account_name`, and `current_balance` from `deposit_accounts` where `account_status` = 'ACTIVE', ordered by `account_id`.

REFACTOR Phase:
- Clean up the test and verify the view returns the correct rows.

### Pass 2: TR002 – Interest and Fee Calculation

RED Phase:
- Write a test that checks for columns `total_interest_earned` and `total_fees_charged` in the view; it should fail initially.

GREEN Phase – Tasks:
- Task 2.1: Use a CTE `interest_fee_summary` to aggregate `interest_earned` and `fees_charged` from `interest_calculations` where status = 'POSTED'.
- Task 2.2: Join this CTE to the view on `account_id` and include `COALESCE(...,0.00)` for both new columns.

REFACTOR Phase:
- Clean up test procedure and verify output.

### Pass 3: TR003 – Transaction Summary and Reporting

RED Phase:
- Write a test that ensures the view has columns `total_transactions` and `last_transaction_date`; it should fail.

GREEN Phase – Tasks:
- Task 3.1: Create a CTE `transaction_summary` aggregating completed transactions: count of transactions, sum of deposits, sum of withdrawals, and last transaction date grouped by `account_id`.
- Task 3.2: Join this CTE into the view, include `COALESCE` defaults, and add additional calculated fields (`net_growth`, `activity_status`, `balance_category`, etc.) as per business rules.

REFACTOR Phase:
- Remove test procedures and verify the comprehensive view.

---

## Challenge Instructions

1. **Only** replace the commented “Your query description” placeholders in Pass 1, Pass 2, and Pass 3 with the actual SQL queries.
2. Do not modify view or test names, input parameters, or table structures.
3. Follow RED → GREEN → REFACTOR for each pass.
4. After each pass, run the provided test to move from FAIL to PASS.

---

## Validation Tests

```sql
-- Pass 1
CALL Test_TR001_BasicBalanceCalculation();

-- Pass 2
CALL Test_TR002_InterestFeeCalculation();

-- Pass 3
CALL Test_TR003_TransactionSummary();
```

---

## Bonus Challenges

- Add an `account_health` category based on balances and activity.
- Parameterize the view to filter by `account_type` or date range without altering tests.
- Use `EXPLAIN` to optimize CTE performance; add indexes where beneficial.
- Write additional tests for edge cases: no transactions, fully paid, or invalid data.

---

## Additional Improvement Idea

**Interactive Query Templates:** Include inline SQL template snippets in comments that students can uncomment and modify to speed up development and reduce syntax errors. This scaffolding helps focus on core logic rather than typing boilerplate.
