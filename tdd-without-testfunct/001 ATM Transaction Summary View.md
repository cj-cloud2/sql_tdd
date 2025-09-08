# TDD Assignment 001: ATM Transaction Summary View

## Difficulty Level: 2

## Business Requirements

**Domain**: Banking ATM System

**User Story**: As a bank operations manager, I need a view that displays ATM transaction summaries to monitor daily transaction volumes and amounts across different ATM locations.

**Business Requirements**:

1. The view should display basic ATM transaction information including ATM ID, location, and total transaction count
2. The view should include the total transaction amount for each ATM location

## Testable Requirements

Based on the business requirements, we need to create testable requirements:

**Requirement 1**: Create a view that shows ATM ID, location name, and total count of transactions per ATM
**Requirement 2**: Extend the view to include the total transaction amount for each ATM

## Database Setup

First, let's create our base tables with sample data:

```sql
-- Create ATM table
CREATE TABLE atm (
    atm_id VARCHAR(10) PRIMARY KEY,
    location_name VARCHAR(100) NOT NULL,
    branch_code VARCHAR(10) NOT NULL
);

-- Create transactions table
CREATE TABLE atm_transactions (
    transaction_id VARCHAR(20) PRIMARY KEY,
    atm_id VARCHAR(10) NOT NULL,
    transaction_type VARCHAR(20) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    transaction_date DATE NOT NULL,
    FOREIGN KEY (atm_id) REFERENCES atm(atm_id)
);

-- Insert sample ATM data
INSERT INTO atm VALUES ('ATM001', 'Main Street Branch', 'BR001');
INSERT INTO atm VALUES ('ATM002', 'Shopping Mall', 'BR002');
INSERT INTO atm VALUES ('ATM003', 'Airport Terminal', 'BR003');

-- Insert sample transaction data
INSERT INTO atm_transactions VALUES ('TXN001', 'ATM001', 'WITHDRAWAL', 500.00, '2025-08-30');
INSERT INTO atm_transactions VALUES ('TXN002', 'ATM001', 'DEPOSIT', 1000.00, '2025-08-30');
INSERT INTO atm_transactions VALUES ('TXN003', 'ATM001', 'WITHDRAWAL', 200.00, '2025-08-30');
INSERT INTO atm_transactions VALUES ('TXN004', 'ATM002', 'WITHDRAWAL', 300.00, '2025-08-30');
INSERT INTO atm_transactions VALUES ('TXN005', 'ATM002', 'DEPOSIT', 750.00, '2025-08-30');
INSERT INTO atm_transactions VALUES ('TXN006', 'ATM003', 'WITHDRAWAL', 400.00, '2025-08-30');
```


## TDD Implementation Guide

### Initial Setup: Create Base/Dummy View

Start with a basic view structure:

```sql
-- Create initial dummy view
CREATE VIEW atm_transaction_summary AS
SELECT 
    'ATM001' as atm_id,
    'Dummy Location' as location_name,
    0 as transaction_count
FROM (VALUES(0)) AS dummy(x);
```


## Pass 1: RED-GREEN-REFACTOR Cycle

### Requirement 1: Display ATM ID, location name, and transaction count

#### RED Phase: Create Test (Should Fail)

```sql
-- Test for Requirement 1: Check if view returns ATM transaction counts
SELECT 
    atm_id,
    location_name,
    transaction_count
FROM atm_transaction_summary
WHERE atm_id = 'ATM001';

-- Expected Result: 
-- ATM001 | Main Street Branch | 3
-- But current dummy view will return: ATM001 | Dummy Location | 0
```

**Run this test** - It will fail because our dummy view doesn't show the correct data.

#### GREEN Phase: Make Test Pass

```sql
-- Drop the dummy view
DROP VIEW atm_transaction_summary;

-- Create view that satisfies Requirement 1
CREATE VIEW atm_transaction_summary AS
SELECT 
    a.atm_id,
    a.location_name,
    COUNT(t.transaction_id) as transaction_count
FROM atm a
LEFT JOIN atm_transactions t ON a.atm_id = t.atm_id
GROUP BY a.atm_id, a.location_name;
```


#### Verify Test Now Passes

```sql
-- Test for Requirement 1: Check if view returns ATM transaction counts
SELECT 
    atm_id,
    location_name,
    transaction_count
FROM atm_transaction_summary
WHERE atm_id = 'ATM001';

-- Expected and Actual Result: 
-- ATM001 | Main Street Branch | 3
```

**Run this test** - It should now pass!

#### REFACTOR Phase

```sql
-- The current implementation is clean and efficient, no refactoring needed for now
-- View structure is simple and meets the requirement
```


## Pass 2: RED-GREEN-REFACTOR Cycle

### Requirement 2: Include total transaction amount for each ATM

#### RED Phase: Create Test (Should Fail)

```sql
-- Test for Requirement 2: Check if view includes total transaction amounts
SELECT 
    atm_id,
    location_name,
    transaction_count,
    total_amount
FROM atm_transaction_summary
WHERE atm_id = 'ATM001';

-- Expected Result: 
-- ATM001 | Main Street Branch | 3 | 1700.00
-- But current view doesn't have total_amount column, so this will fail
```

**Run this test** - It will fail because the `total_amount` column doesn't exist.

#### GREEN Phase: Make Test Pass

```sql
-- Drop the existing view
DROP VIEW atm_transaction_summary;

-- Create view that satisfies both Requirements 1 and 2
CREATE VIEW atm_transaction_summary AS
SELECT 
    a.atm_id,
    a.location_name,
    COUNT(t.transaction_id) as transaction_count,
    COALESCE(SUM(t.amount), 0) as total_amount
FROM atm a
LEFT JOIN atm_transactions t ON a.atm_id = t.atm_id
GROUP BY a.atm_id, a.location_name;
```


#### Verify Test Now Passes

```sql
-- Test for Requirement 2: Check if view includes total transaction amounts
SELECT 
    atm_id,
    location_name,
    transaction_count,
    total_amount
FROM atm_transaction_summary
WHERE atm_id = 'ATM001';

-- Expected and Actual Result: 
-- ATM001 | Main Street Branch | 3 | 1700.00
```

**Run this test** - It should now pass!

#### REFACTOR Phase

```sql
-- Add some formatting and ensure consistency
DROP VIEW atm_transaction_summary;

CREATE VIEW atm_transaction_summary AS
SELECT 
    a.atm_id,
    a.location_name,
    COUNT(t.transaction_id) as transaction_count,
    COALESCE(ROUND(SUM(t.amount), 2), 0.00) as total_amount
FROM atm a
LEFT JOIN atm_transactions t ON a.atm_id = t.atm_id
GROUP BY a.atm_id, a.location_name
ORDER BY a.atm_id;
```


## Final Verification Tests

Run these comprehensive tests to ensure both requirements are satisfied:

```sql
-- Test 1: Verify all ATMs are included with correct transaction counts
SELECT 
    atm_id,
    location_name,
    transaction_count,
    total_amount
FROM atm_transaction_summary
ORDER BY atm_id;

-- Expected Results:
-- ATM001 | Main Street Branch | 3 | 1700.00
-- ATM002 | Shopping Mall | 2 | 1050.00  
-- ATM003 | Airport Terminal | 1 | 400.00

-- Test 2: Verify specific ATM data
SELECT 
    location_name,
    transaction_count,
    total_amount
FROM atm_transaction_summary
WHERE atm_id = 'ATM002';

-- Expected Result:
-- Shopping Mall | 2 | 1050.00
```


## Assignment Completion Checklist

- [x] **Pass 1 Complete**: View shows ATM ID, location name, and transaction count
- [x] **Pass 2 Complete**: View includes total transaction amount
- [x] **RED-GREEN-REFACTOR**: Each requirement followed proper TDD cycle
- [x] **Tests Pass**: All verification tests return expected results


## Cleanup Script

```sql
-- Cleanup script - Run this to clean up all created objects
DROP VIEW IF EXISTS atm_transaction_summary;
DROP TABLE IF EXISTS atm_transactions;
DROP TABLE IF EXISTS atm;
```


## Learning Objectives Achieved

1. **TDD Methodology**: Followed strict RED-GREEN-REFACTOR cycle for each requirement
2. **View Development**: Created a functional view with aggregation and joins
3. **Test-Driven Development**: Wrote tests before implementation and ensured they guide development
4. **SQL Aggregation**: Used COUNT and SUM functions with GROUP BY
5. **Error Handling**: Used COALESCE to handle NULL values appropriately



