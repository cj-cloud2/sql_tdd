# TDD Challenge LAB: Banking Account Balance LEFT JOIN Query

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)  
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking CBS (Core Banking System)  
**Objective:** Create a SQL LEFT JOIN statement with 2 tables joined on 1 column, with SUM() aggregation using Test-Driven Development (TDD) approach

## Business Requirements

### Background

Metropolitan Bank operates a core banking system that manages customer accounts and their transaction history. The bank needs to generate comprehensive account balance reports that include all customer accounts, even those with no transaction history, to ensure complete visibility of the bank's account portfolio for regulatory compliance and risk management.

### Core Business Requirements

1. **BR001**: Generate a report showing all customer accounts in the system, including accounts with zero transaction activity
2. **BR002**: Calculate the current balance for each account by summing all transaction amounts
3. **BR003**: Display account information with calculated balances for portfolio analysis and regulatory reporting

## Testable Requirements

Based on business requirements, you need to implement three testable requirements following TDD methodology:

### TR001: Basic Account-Transaction JOIN

**Given** customer account data and transaction data exist  
**When** querying all accounts with their transaction information  
**Then** return all accounts including those with zero transactions using LEFT JOIN

### TR002: Single Column Join Implementation

**Given** transactions are linked to accounts by account_number  
**When** joining tables on the account_number column  
**Then** return accurate transaction data for each account including NULL values for accounts without transactions

### TR003: Balance Calculation with SUM Function

**Given** joined data from accounts and transactions  
**When** applying SUM aggregation to transaction amounts  
**Then** return the calculated account balance for each account (0 for accounts with no transactions)

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS banking_cbs_lab;
USE banking_cbs_lab;

-- Table 1: Customer Accounts (Left table)
CREATE TABLE customer_accounts (
    account_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT') NOT NULL,
    branch_code VARCHAR(10) NOT NULL,
    opening_date DATE NOT NULL,
    status ENUM('ACTIVE', 'INACTIVE', 'CLOSED') DEFAULT 'ACTIVE',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_account_number (account_number)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Account Transactions (Right table)
CREATE TABLE account_transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL,
    transaction_type ENUM('CREDIT', 'DEBIT') NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    transaction_date DATE NOT NULL,
    description VARCHAR(200),
    reference_number VARCHAR(50),
    status ENUM('COMPLETED', 'PENDING', 'FAILED') DEFAULT 'COMPLETED',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_account_number (account_number),
    INDEX idx_transaction_date (transaction_date)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```

### Test Data Insertion

```sql
-- Insert customer account data
INSERT INTO customer_accounts (account_number, customer_name, account_type, branch_code, opening_date, status) VALUES
('ACC001', 'John Smith', 'SAVINGS', 'BR001', '2024-01-15', 'ACTIVE'),
('ACC002', 'Sarah Johnson', 'CURRENT', 'BR001', '2024-02-01', 'ACTIVE'),
('ACC003', 'Michael Brown', 'SAVINGS', 'BR002', '2024-01-20', 'ACTIVE'),
('ACC004', 'Emily Davis', 'CURRENT', 'BR002', '2024-02-15', 'ACTIVE'),
('ACC005', 'Robert Wilson', 'FIXED_DEPOSIT', 'BR001', '2024-03-01', 'ACTIVE'),
('ACC006', 'Lisa Anderson', 'SAVINGS', 'BR003', '2024-03-15', 'INACTIVE'),
('ACC007', 'David Martinez', 'CURRENT', 'BR003', '2024-04-01', 'ACTIVE');

-- Insert transaction data (note: some accounts have no transactions)
INSERT INTO account_transactions (account_number, transaction_type, amount, transaction_date, description, reference_number, status) VALUES
-- ACC001 transactions
('ACC001', 'CREDIT', 5000.00, '2024-01-16', 'Initial Deposit', 'REF001', 'COMPLETED'),
('ACC001', 'DEBIT', 200.00, '2024-01-20', 'ATM Withdrawal', 'REF002', 'COMPLETED'),
('ACC001', 'CREDIT', 1500.00, '2024-02-01', 'Salary Credit', 'REF003', 'COMPLETED'),
-- ACC002 transactions  
('ACC002', 'CREDIT', 10000.00, '2024-02-02', 'Initial Deposit', 'REF004', 'COMPLETED'),
('ACC002', 'DEBIT', 2500.00, '2024-02-10', 'Business Payment', 'REF005', 'COMPLETED'),
-- ACC003 transactions
('ACC003', 'CREDIT', 3000.00, '2024-01-21', 'Initial Deposit', 'REF006', 'COMPLETED'),
('ACC003', 'DEBIT', 500.00, '2024-01-25', 'Online Transfer', 'REF007', 'COMPLETED'),
('ACC003', 'DEBIT', 100.00, '2024-02-05', 'Service Charge', 'REF008', 'COMPLETED'),
-- ACC004 has one transaction
('ACC004', 'CREDIT', 8000.00, '2024-02-16', 'Initial Deposit', 'REF009', 'COMPLETED');
-- ACC005, ACC006, ACC007 have no transactions
```

## Challenge Tasks

### Task 1: TDD Implementation - Pass 1: TR001 Basic LEFT JOIN

#### Challenge Requirements:
1. Write a **RED phase** test that creates expected results for 7 unique accounts
2. Write a **GREEN phase** query that:
   - Retrieves all account information from the customer_accounts table
   - Uses LEFT JOIN to combine with transaction data
   - Ensures all accounts are included, even those without transactions
   - Uses DISTINCT to avoid duplicate rows

#### Your Challenge:
Create SQL code that will pass the TR001 test showing exactly 7 accounts.

**Test Specification:**
- Expected: Exactly 7 unique accounts (all accounts from customer_accounts table)
- Method: Use LEFT JOIN between customer_accounts and account_transactions
- Join condition: Match on account_number column

---

### Task 2: TDD Implementation - Pass 2: TR002 Single Column Join Verification

#### Challenge Requirements:
1. Write a **RED phase** test that identifies accounts with no transactions (should have NULL transaction_id)
2. Write a **GREEN phase** query that:
   - Shows transaction details for each account
   - Displays NULL values for accounts without transactions
   - Demonstrates proper single-column join implementation

#### Your Challenge:
Create SQL code that shows transaction details with NULL values for accounts that have no transactions.

**Test Specification:**
- Expected: At least 3 accounts should have NULL transaction_id (ACC005, ACC006, ACC007)
- Method: LEFT JOIN showing individual transaction records
- Result: Multiple rows per account if they have multiple transactions

---

### Task 3: TDD Implementation - Pass 3: TR003 SUM Aggregation

#### Challenge Requirements:
1. Write a **RED phase** test with expected account balances:
   - ACC001: 6300.00 (5000.00 - 200.00 + 1500.00)
   - ACC002: 7500.00 (10000.00 - 2500.00)
   - ACC003: 2400.00 (3000.00 - 500.00 - 100.00)
   - ACC004: 8000.00 (8000.00)
   - ACC005, ACC006, ACC007: 0.00 (no transactions)

2. Write a **GREEN phase** query that:
   - Calculates account balance using SUM with CASE statement
   - Handles CREDIT transactions as positive amounts
   - Handles DEBIT transactions as negative amounts
   - Shows 0.00 for accounts with no transactions
   - Groups results properly to show one row per account

#### Your Challenge:
Create the final production query that calculates accurate account balances for all accounts.

**Test Specification:**
- Expected: 7 rows with calculated balances matching the expected values above
- Method: Use SUM with CASE statement for CREDIT/DEBIT logic
- Requirement: Include only COMPLETED transactions in balance calculation

---

## Expected Final Output Format

Your final query should produce results in this format:

| account_number | customer_name | account_type | branch_code | status | account_balance |
|:--|:--|:--|:--|:--|--:|
| ACC001 | John Smith | SAVINGS | BR001 | ACTIVE | 6300.00 |
| ACC002 | Sarah Johnson | CURRENT | BR001 | ACTIVE | 7500.00 |
| ACC003 | Michael Brown | SAVINGS | BR002 | ACTIVE | 2400.00 |
| ACC004 | Emily Davis | CURRENT | BR002 | ACTIVE | 8000.00 |
| ACC005 | Robert Wilson | FIXED_DEPOSIT | BR001 | ACTIVE | 0.00 |
| ACC006 | Lisa Anderson | SAVINGS | BR003 | INACTIVE | 0.00 |
| ACC007 | David Martinez | CURRENT | BR003 | ACTIVE | 0.00 |

## Additional Challenge Ideas

### Bonus Challenges (Optional):

1. **Performance Optimization**: Add appropriate indexes and explain how they improve query performance

2. **Error Handling**: Create additional test cases for edge cases:
   - What happens with PENDING or FAILED transactions?
   - How to handle accounts with only DEBIT transactions (negative balances)?

3. **Advanced Reporting**: Extend your solution to include:
   - Transaction count per account
   - Latest transaction date per account
   - Average transaction amount per account

4. **Data Validation**: Create tests to verify:
   - No duplicate account numbers in final result
   - All accounts from master table are included
   - Balance calculations are mathematically correct

### Real-World Scenarios:

1. **Audit Trail**: How would you modify the query to show both current balance and balance as of a specific date?

2. **Multi-Currency**: If accounts could have transactions in different currencies, how would you handle the balance calculation?

3. **Account Hierarchies**: If accounts could have sub-accounts, how would you aggregate balances at the parent account level?

## Key Learning Objectives

By completing this challenge, you should understand:

1. **TDD Methodology**: RED-GREEN-REFACTOR cycle with SQL
2. **LEFT JOIN Behavior**: Including all records from left table even without matches
3. **Single Column Joins**: Proper join syntax and conditions
4. **Aggregate Functions**: Using SUM with conditional logic
5. **NULL Handling**: Working with NULL values in aggregate functions
6. **Banking Logic**: CREDIT vs DEBIT transaction handling
7. **GROUP BY Requirements**: Proper grouping with aggregate functions

## Success Criteria

Your solution is complete when:
- [ ] All TDD tests pass (RED → GREEN → REFACTOR)
- [ ] Query returns exactly 7 rows
- [ ] All account balances match expected values
- [ ] Query handles accounts with no transactions correctly
- [ ] Code follows SQL best practices and is well-commented

## Cleanup Script

```sql
-- Cleanup: Remove all test data and tables
DROP TABLE IF EXISTS account_transactions;
DROP TABLE IF EXISTS customer_accounts;
DROP DATABASE IF EXISTS banking_cbs_lab;

-- Verify cleanup
SHOW DATABASES LIKE 'banking_cbs_lab';
-- Should return empty result set
```

---

**Good luck with your TDD SQL Challenge!** Remember to follow the RED-GREEN-REFACTOR cycle for each test requirement.