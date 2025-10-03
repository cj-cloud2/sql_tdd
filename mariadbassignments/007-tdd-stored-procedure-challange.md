# TDD Challenge LAB: Loan Account Balance Report Stored Procedure

---

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)
**Difficulty Level:** 5 (Expert)
**Domain:** Loans Accounts Banking
**Objective:** Implement a stored procedure that generates a comprehensive loan account balance report using a strict TDD approach.

---

## Business Requirements (Context)

- Generate loan account balance reports showing current outstanding amount, payment status, and health indicators.
- Calculate overdue amounts and penalties for loans past due, per loan type.
- Project next payment due dates and amounts for active loans.

---

## Pre-Setup (Provided)

All schema and sample data are provided in the instructor’s solution file. Use that file to create the database and seed data. Do not modify schema or seed data for the core tasks.

---

## Rules for This Challenge

- Replace every SQL query ONLY in Pass 1, Pass 2, and Pass 3 with working SQL.
- Follow RED → GREEN → REFACTOR for each pass.
- Do NOT change procedure names, input parameters, or table/column names.
- Where you see a "Your query description" comment, replace it with an actual SQL statement.

---

## Pass 1: TR001 – Basic Loan Balance Calculation

### RED Phase

Create the placeholder procedure and the failing test as shown in the solution file. Do not alter test logic. Your task starts where the comments indicate.

In the GREEN phase below, replace the queries described in plain English with actual SQL.

### GREEN Phase – Tasks

Inside GetLoanBalanceReport, implement the following queries in place of the descriptions:

- Task 1.1: Select each active loan account’s ID, customer ID, loan type, and compute current balance as principal amount minus the sum of principal paid on processed payments (treat missing sums as zero). Group by account and order by account ID.

- Task 1.2 (Test helper): In the associated test procedure, count the number of active loan accounts and store it in a variable for comparison.

- Task 1.3 (Optional verification): Query the procedure’s output to ensure it returns one row per active account.

### REFACTOR

Remove temporary objects and keep the procedure minimal and readable.

---

## Pass 2: TR002 – Overdue Amount Calculation with Penalties

### RED Phase

Create the failing test that checks for the existence of overdue_amount and penalty_amount in the procedure output.

### GREEN Phase – Tasks

Extend GetLoanBalanceReport to add two calculated columns:

- Task 2.1: Calculate overdue_amount as the sum of scheduled amounts for this account where the due date is before today and the schedule entry is unpaid. Treat missing sums as zero.

- Task 2.2: Calculate penalty_amount by multiplying the overdue_amount by the account’s penalty rate percentage, rounded to two decimals. Treat missing values as zero.

- Task 2.3: Ensure the result still returns one row per active account and remains ordered by account ID. Avoid double-counting by grouping correctly and using appropriate joins.

### REFACTOR

Keep calculation logic in coherent blocks (e.g., use a subquery or derived table for overdue sums). Ensure readability and correct grouping.

---

## Pass 3: TR003 – Payment Projection and Reporting

### RED Phase

Create the failing test that checks for next_due_date and next_payment_amount in the procedure output.

### GREEN Phase – Tasks

Extend GetLoanBalanceReport with next payment projections:

- Task 3.1: For each account, find the nearest upcoming unpaid schedule entry on or after today and output its due date as next_due_date.

- Task 3.2: Output the scheduled_amount for that nearest upcoming unpaid entry as next_payment_amount. If no such entry exists, use a fallback (e.g., return a default message or zero amount as per the solution’s behavior).

- Task 3.3: Maintain one row per active account, appropriate grouping, and ordering that prioritizes accounts with overdue amounts.

### REFACTOR

Isolate calculations in derived tables (paid_totals, overdue_calc, next_payment). Prefer COALESCE for null safety.

---

## What You Must Edit (Only Where Described)

Replace each "Your query description" comment in Pass 1, Pass 2, and Pass 3 with the actual SQL statements that perform the described operation. Keep all other code intact.

---

## Validation

After each pass, run the provided test procedure for that pass. All tests must transition from FAIL to PASS without changing the tests.

- Pass 1: Should return one row per active account with correct current balance.
- Pass 2: Should include overdue_amount and penalty_amount with correct values.
- Pass 3: Should include next_due_date and next_payment_amount for each active account when applicable.

---

## Bonus Challenges (Optional)

- Add defensive checks for invalid data (e.g., negative balances) and surface warnings.
- Create additional tests for edge cases: no payments, fully paid schedules, suspended accounts, or no upcoming payments.
- Add indexes if needed and use EXPLAIN to analyze the procedure query plan.
- Return an additional column "account_health" that categorizes accounts (e.g., OK, OVERDUE, AT_RISK) based on thresholds.
- Parameterize the procedure to filter by loan_type or customer_id without changing tests.

---

## Download

A challenge sheet has been generated for you to download and distribute to learners.
