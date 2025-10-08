# TDD Challenge LAB: Banking CBS RIGHT JOIN with DATE_FORMAT

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)  
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking CBS (Core Banking System)  
**Objective:** Implement a RIGHT JOIN with a single-column join and apply DATE_FORMAT(), using the TDD (RED → GREEN → REFACTOR) approach. This challenge sheet removes solution SQL for Pass 1, 2, 3 and replaces them with plain-English tasks you must implement.

## Business Requirements

### Background
FirstNational Bank needs monthly statements showing all transaction activities with readable date formats. The report must include transactions even if matching account master records are missing, and date output should be customer-friendly.

### Core Business Requirements
- **BR001**: Show all transactions with associated account details
- **BR002**: Preserve transactions where account info is missing (accounts not found)
- **BR003**: Display dates in DD-MM-YYYY format

## Testable Requirements (TDD)
- **TR001**: RIGHT JOIN preserves all transaction rows
- **TR002**: Correct single-column join on account_number; unmatched accounts appear with NULL fields
- **TR003**: Dates are formatted using DD-MM-YYYY via DATE_FORMAT

## Pre-Setup Phase

### Database and Tables
```sql
CREATE DATABASE IF NOT EXISTS banking_cbs_lab;
USE banking_cbs_lab;

CREATE TABLE accounts (
    account_number VARCHAR(20) NOT NULL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    account_type ENUM('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT') NOT NULL,
    branch_code CHAR(4) NOT NULL,
    opening_date DATE NOT NULL,
    balance DECIMAL(15,2) DEFAULT 0.00,
    status ENUM('ACTIVE', 'DORMANT', 'CLOSED') DEFAULT 'ACTIVE',
    created_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

CREATE TABLE transactions (
    transaction_id INT AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL,
    transaction_date DATE NOT NULL,
    transaction_type ENUM('CREDIT', 'DEBIT') NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    description VARCHAR(200),
    reference_number VARCHAR(50),
    processed_by VARCHAR(20),
    INDEX idx_account_number (account_number),
    INDEX idx_transaction_date (transaction_date)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```

### Seed Data
```sql
INSERT INTO accounts (account_number, customer_name, account_type, branch_code, opening_date, balance, status) VALUES
('ACC001', 'John Smith', 'SAVINGS', 'BR01', '2024-01-15', 25000.00, 'ACTIVE'),
('ACC002', 'Mary Johnson', 'CURRENT', 'BR01', '2024-02-20', 15000.00, 'ACTIVE'),
('ACC003', 'Robert Davis', 'SAVINGS', 'BR02', '2024-01-10', 8000.00, 'DORMANT'),
('ACC005', 'Linda Wilson', 'FIXED_DEPOSIT', 'BR02', '2024-03-01', 50000.00, 'ACTIVE');

INSERT INTO transactions (account_number, transaction_date, transaction_type, amount, description, reference_number, processed_by) VALUES
('ACC001', '2025-01-15', 'CREDIT', 5000.00, 'Salary Credit', 'TXN001', 'SYS001'),
('ACC001', '2025-02-01', 'DEBIT', 1500.00, 'ATM Withdrawal', 'TXN002', 'ATM01'),
('ACC002', '2025-01-20', 'CREDIT', 2000.00, 'Online Transfer', 'TXN003', 'SYS002'),
('ACC002', '2025-02-10', 'DEBIT', 500.00, 'Bill Payment', 'TXN004', 'SYS001'),
('ACC003', '2025-01-25', 'CREDIT', 1000.00, 'Interest Credit', 'TXN005', 'SYS003'),
('ACC004', '2025-02-05', 'CREDIT', 3000.00, 'Cheque Deposit', 'TXN006', 'BR01'),
('ACC004', '2025-02-15', 'DEBIT', 800.00, 'Card Payment', 'TXN007', 'POS01'),
('ACC006', '2025-01-30', 'DEBIT', 2500.00, 'Wire Transfer', 'TXN008', 'SYS004');
```

## Challenge Tasks (Replace SQL with Plain-English Implementation Goals)

### Pass 1: TR001 Basic RIGHT JOIN
- RED: Define a failing test that expects the total number of transactions to be exactly eight.
- GREEN: Implement a query that joins transactions to accounts using a RIGHT JOIN so that all transactions are returned, even if there is no matching account, and select key columns to verify presence.
- REFACTOR: Clean up temporary artifacts after verification.

What to implement (plain English instead of SQL):
- Create a result set that lists every transaction
- For each transaction, attach account details when available
- Ensure that transactions without a corresponding account still appear in the result
- Verify the row count matches the number of rows in the transactions table

### Pass 2: TR002 Single Column Join Verification
- RED: Define a failing test asserting that at least two rows should have missing account details because the account numbers don’t exist in the accounts table.
- GREEN: Implement a query that joins by the single column account_number and returns columns showing NULLs for account fields when no match exists; order results by date for readability.
- REFACTOR: Remove temporary tables used in testing.

What to implement (plain English instead of SQL):
- Produce a detailed list of transactions joined with accounts using only the account_number column
- Include customer name, account type, and status where available
- Show NULLs for account-related columns when the account isn’t found (e.g., for ACC004 and ACC006)
- Order the results by transaction date

### Pass 3: TR003 DATE_FORMAT Implementation
- RED: Define a failing test that compares expected DD-MM-YYYY date strings against unformatted dates to demonstrate the failure.
- GREEN: Implement a query that outputs a formatted date column using the DD-MM-YYYY pattern, keeping all RIGHT JOIN behavior intact.
- REFACTOR: Run a compact test suite that verifies row counts, NULL handling, and the correctness of at least one formatted date.

What to implement (plain English instead of SQL):
- Create a column that shows each transaction’s date in DD-MM-YYYY format
- Preserve all transactions from the transactions table
- Ensure at least one sample transaction’s formatted date matches the expected formatted value

## Expected Final Output (for your reference)

| transaction_id | account_number | customer_name | account_type | branch_code | transaction_date | transaction_type | amount | description | reference_number | processed_by |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| 1 | ACC001 | John Smith | SAVINGS | BR01 | 15-01-2025 | CREDIT | 5000.00 | Salary Credit | TXN001 | SYS001 |
| 3 | ACC002 | Mary Johnson | CURRENT | BR01 | 20-01-2025 | CREDIT | 2000.00 | Online Transfer | TXN003 | SYS002 |
| 5 | ACC003 | Robert Davis | SAVINGS | BR02 | 25-01-2025 | CREDIT | 1000.00 | Interest Credit | TXN005 | SYS003 |
| 8 | ACC006 | Account Not Found | NULL | NULL | 30-01-2025 | DEBIT | 2500.00 | Wire Transfer | TXN008 | SYS004 |
| 2 | ACC001 | John Smith | SAVINGS | BR01 | 01-02-2025 | DEBIT | 1500.00 | ATM Withdrawal | TXN002 | ATM01 |
| 6 | ACC004 | Account Not Found | NULL | NULL | 05-02-2025 | CREDIT | 3000.00 | Cheque Deposit | TXN006 | BR01 |
| 4 | ACC002 | Mary Johnson | CURRENT | BR01 | 10-02-2025 | DEBIT | 500.00 | Bill Payment | TXN004 | SYS001 |
| 7 | ACC004 | Account Not Found | NULL | NULL | 15-02-2025 | DEBIT | 800.00 | Card Payment | TXN007 | POS01 |

## Additional Challenge Ideas

- Performance: Ask learners to propose indexes and justify them
- Robustness: Add tests for empty transactions table and for all transactions belonging to missing accounts
- Usability: Ask to replace NULL customer_name with a label (e.g., "Account Not Found") using a NULL-safe function
- QA Extensions: Add unit tests that assert ordering (date then id) and that the RIGHT JOIN row count equals the transactions row count

## Success Criteria
- All RED tests initially fail, then pass after implementation
- Row count equals the number of transactions
- At least two rows have missing account details
- Dates are rendered as DD-MM-YYYY
- Code follows clear TDD structure with cleanup

## Cleanup
```sql
DROP TABLE IF EXISTS transactions;
DROP TABLE IF EXISTS accounts;
DROP DATABASE IF EXISTS banking_cbs_lab;
```

Good luck! Follow the RED → GREEN → REFACTOR cycle and keep tests focused and minimal.