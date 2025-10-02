# TDD Challenge LAB: Subqueries and CTEs in Banking CRM

## Overview

**Language:** MariaDB-10-SQL (ANSI SQL Compliant)  
**Difficulty Level:** 3 (Intermediate)  
**Domain:** Banking and Finance, CRM  
**Objective:** Use Test-Driven Development (TDD) to implement complex queries with subqueries and CTEs (Common Table Expressions) for Customer Transaction Analysis. Follow the RED-GREEN-REFACTOR cycle for each pass.

## Business Requirements

A retail bank wants comprehensive analysis of customer behavior and engagement for CRM (Customer Relationship Management). The system must:

1. **BR001**: Report each customer's total transaction value, only including their 'ACTIVE' or 'SETTLED' transactions
2. **BR002**: Highlight customers with the highest single transaction in the last quarter, showing customer info and transaction details  
3. **BR003**: Provide the top 3 regions by aggregate transaction count for CRM campaign prioritization

## Testable Requirements

Derived TDD objectives:

- **TR001**: For all customers, display (customer_id, name, SUM of qualifying transactions) using a subquery or CTE (must only sum qualifying statuses)
- **TR002**: Identify customer(s) with max transaction value in the last quarter, joining with customer details (use nested subquery or CTE)  
- **TR003**: Return top 3 regions ranked by transaction counts (use CTE for intermediate aggregation)

## Pre-Setup Phase

### Database and Table Creation

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS banking_crm_lab;
USE banking_crm_lab;

-- Table 1: Customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    region VARCHAR(30),
    joined_date DATE
) ENGINE=InnoDB CHARACTER SET=utf8mb4;

-- Table 2: Transactions
CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    txn_date DATE NOT NULL,
    txn_type ENUM('DEPOSIT', 'WITHDRAWAL', 'LOAN', 'TRANSFER'),
    txn_status ENUM('ACTIVE', 'SETTLED', 'FAILED', 'CANCELLED'),
    txn_amount DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```

### Test Data Insertion

```sql
-- Insert customer data
INSERT INTO customers (customer_id, customer_name, region, joined_date) VALUES
(1, 'Alice', 'East', '2022-01-10'),
(2, 'Bob', 'West', '2021-09-20'),
(3, 'Charlie', 'North', '2020-07-05'),
(4, 'Diana', 'East', '2023-02-13'),
(5, 'Evan', 'South', '2020-02-23');

-- Insert transactions (cover various statuses, dates, and customers)
INSERT INTO transactions (customer_id, txn_date, txn_type, txn_status, txn_amount) VALUES
(1, '2025-08-01', 'DEPOSIT', 'ACTIVE', 900.00),
(1, '2025-08-12', 'LOAN', 'SETTLED', 4500.00),
(1, '2025-06-11', 'TRANSFER', 'FAILED', 500.00),
(2, '2025-09-09', 'DEPOSIT', 'ACTIVE', 1200.00),
(2, '2025-07-20', 'LOAN', 'SETTLED', 3500.00),
(3, '2025-08-25', 'WITHDRAWAL', 'SETTLED', 1500.00),
(3, '2025-08-15', 'WITHDRAWAL', 'CANCELLED', 1850.00),
(4, '2025-09-16', 'DEPOSIT', 'ACTIVE', 650.00),
(4, '2025-09-26', 'LOAN', 'ACTIVE', 4900.00),
(4, '2025-07-18', 'TRANSFER', 'SETTLED', 700.00),
(5, '2025-06-22', 'DEPOSIT', 'ACTIVE', 900.00),
(5, '2025-08-29', 'TRANSFER', 'ACTIVE', 1100.00),
(5, '2025-09-27', 'DEPOSIT', 'FAILED', 975.00);
```

## Challenge Tasks

### Task 1: TR001 - Customer Transaction Totals using CTE/Subquery

#### Challenge Requirements:
**RED Phase**: Create a test that expects specific transaction totals for each customer:
- Alice (ID 1): 5400.00 (sum of ACTIVE/SETTLED only)
- Bob (ID 2): 4700.00
- Charlie (ID 3): 1500.00
- Diana (ID 4): 6250.00
- Evan (ID 5): 2000.00

**GREEN Phase**: Your task is to implement a query that:
- Uses a CTE or subquery to filter transactions by status ('ACTIVE' or 'SETTLED' only)
- Joins customers with their qualifying transactions
- Calculates the sum of transaction amounts for each customer
- Returns customer_id and total transaction amount

**What you need to implement (plain English)**:
Create a query structure that first isolates qualifying transactions in a CTE, then joins this with the customers table to calculate totals. The CTE should filter transactions to include only those with status 'ACTIVE' or 'SETTLED'. Use LEFT JOIN to ensure all customers appear even if they have no qualifying transactions.

---

### Task 2: TR002 - Maximum Transaction in Last Quarter

#### Challenge Requirements:
**RED Phase**: Create a test expecting Diana (customer_id 4) to have the highest single transaction of 4900.00 in the last quarter (July-September 2025).

**GREEN Phase**: Your task is to implement a query that:
- Uses CTE and/or nested subqueries to find transactions in the last quarter (2025-07-01 to 2025-09-30)
- Filters for ACTIVE/SETTLED transactions only
- Joins with customer information
- Finds the customer with the maximum single transaction amount
- Returns customer details with their highest transaction amount

**What you need to implement (plain English)**:
Create a CTE that filters transactions for the specific date range and qualifying statuses. Join this with customer data to get customer names. Use aggregation to find the maximum transaction per customer, then order and limit to get the top customer with the highest single transaction.

---

### Task 3: TR003 - Top Regions by Transaction Count

#### Challenge Requirements:
**RED Phase**: Create a test expecting these regional transaction counts:
- East: 5 transactions
- South: 2 transactions  
- West: 2 transactions

**GREEN Phase**: Your task is to implement a query that:
- Uses a CTE to aggregate transaction counts by region
- Joins customers with transactions (ACTIVE/SETTLED only)
- Groups by region and counts transactions
- Returns top 3 regions ordered by transaction count descending

**What you need to implement (plain English)**:
Create a CTE that joins customers and transactions, filtering for qualifying transaction statuses. Group the results by customer region and count the number of transactions. Order by count in descending order and limit to top 3 regions.

---

## Expected Final Output Format

### TR001 Results:
| customer_id | customer_name | txn_total |
|:--|:--|--:|
| 4 | Diana | 6250.00 |
| 1 | Alice | 5400.00 |
| 2 | Bob | 4700.00 |
| 5 | Evan | 2000.00 |
| 3 | Charlie | 1500.00 |

### TR002 Results:
| customer_id | customer_name | max_txn_amount |
|:--|:--|--:|
| 4 | Diana | 4900.00 |

### TR003 Results:
| region | txn_count |
|:--|--:|
| East | 5 |
| South | 2 |
| West | 2 |

## Additional Challenge Ideas

### Advanced Extensions:
1. **Multi-Quarter Analysis**: Extend TR002 to show top customer for each quarter
2. **Customer Segmentation**: Create CTEs to categorize customers as High/Medium/Low value based on transaction totals
3. **Trend Analysis**: Use window functions with CTEs to show month-over-month growth per region
4. **Complex Filtering**: Add CTEs to exclude dormant customers (no transactions in last 90 days)

### Performance Challenges:
1. **Index Strategy**: Propose and justify indexes for optimal CTE performance
2. **Query Optimization**: Rewrite solutions using different CTE approaches and compare execution plans
3. **Memory Usage**: Implement materialized vs non-materialized CTE versions

### Real-World Scenarios:
1. **Fraud Detection**: Use nested CTEs to identify unusual transaction patterns
2. **Risk Assessment**: Create hierarchical CTEs for customer risk scoring
3. **Campaign Targeting**: Build CTEs to identify customers for specific marketing campaigns

### Testing Robustness:
1. **Edge Cases**: Test with customers having no transactions, all failed transactions, or transactions outside date ranges
2. **Data Validation**: Add tests to verify CTE intermediate results match expected values
3. **Null Handling**: Test behavior when customer or transaction data contains NULLs

## Key Learning Objectives

By completing this challenge, you will master:

1. **CTE Fundamentals**: Creating and using Common Table Expressions for complex data transformations
2. **Subquery Techniques**: Nested queries for multi-step data analysis
3. **TDD with SQL**: RED-GREEN-REFACTOR methodology for database development
4. **Banking Analytics**: Real-world customer behavior analysis patterns
5. **Performance Awareness**: Understanding when to use CTEs vs subqueries vs temporary tables
6. **Data Filtering**: Complex conditional logic across multiple tables
7. **Aggregation Mastery**: GROUP BY, SUM, COUNT, MAX with proper joins

## Success Criteria

Your solution is complete when:
- [ ] All TDD tests pass (RED → GREEN → REFACTOR)
- [ ] TR001 returns correct totals for all 5 customers
- [ ] TR002 identifies Diana as having the highest transaction (4900.00)
- [ ] TR003 ranks regions correctly by transaction count
- [ ] CTEs are used appropriately for intermediate results
- [ ] Code follows SQL best practices and includes comments
- [ ] All temporary test tables are cleaned up after each pass

## TDD Test Template Structure

Each pass should follow this pattern:

```sql
-- RED Phase: Create failing test
CREATE TEMPORARY TABLE tr00X_expected_results (...);
INSERT INTO tr00X_expected_results VALUES (...);
-- Test assertion that fails initially

-- GREEN Phase: Implement solution
CREATE TEMPORARY TABLE tr00X_solution AS
WITH your_cte AS (
    -- Your CTE implementation here
)
SELECT ... FROM your_cte ...;
-- Re-run test assertion - should pass

-- REFACTOR Phase: Clean up
DROP TEMPORARY TABLE IF EXISTS tr00X_solution;
DROP TEMPORARY TABLE IF EXISTS tr00X_expected_results;
```

## Cleanup Script

```sql
-- Remove all test data and business tables
DROP TABLE IF EXISTS transactions;
DROP TABLE IF EXISTS customers;
DROP DATABASE IF EXISTS banking_crm_lab;

-- Verify cleanup (should return zero)
SHOW DATABASES LIKE 'banking_crm_lab';
```

---

**Good luck with your CTE and Subquery Challenge!** Remember to follow the TDD cycle religiously: write failing tests first, implement minimal working solutions, then refactor for clarity and performance.