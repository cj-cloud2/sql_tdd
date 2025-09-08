# TDD Assignment 002: Trade Finance Risk Assessment View

## Difficulty Level: 3

## Business Requirements

**Domain**: Banking Trade Finance

**User Story**: As a Trade Finance Operations Manager, I need a comprehensive risk assessment view that evaluates Letters of Credit (LC) transactions to monitor exposure levels, payment risks, and compliance requirements across different clients and countries.

**Business Requirements**:

1. The view should display basic LC information including LC number, client details, and LC status
2. The view should calculate risk exposure by including outstanding amounts and risk ratings based on country and client risk profiles
3. The view should provide compliance indicators showing document compliance status and days until LC expiry for proactive risk management

## Testable Requirements

Based on the business requirements, we need to create testable requirements:

**Requirement 1**: Create a view that shows LC number, client name, beneficiary country, LC amount, and current status
**Requirement 2**: Extend the view to include calculated risk metrics: outstanding exposure amount, country risk rating, and client risk score
**Requirement 3**: Add compliance indicators: document compliance percentage, days until expiry, and risk alert flag for LCs expiring within 30 days

## Database Setup

First, let's create our base tables with sample data:

```sql
-- Create clients table
CREATE TABLE trade_clients (
    client_id VARCHAR(10) PRIMARY KEY,
    client_name VARCHAR(100) NOT NULL,
    client_risk_score INTEGER NOT NULL CHECK (client_risk_score BETWEEN 1 AND 10),
    country_code VARCHAR(3) NOT NULL
);

-- Create country risk table
CREATE TABLE country_risk (
    country_code VARCHAR(3) PRIMARY KEY,
    country_name VARCHAR(50) NOT NULL,
    risk_rating VARCHAR(10) NOT NULL CHECK (risk_rating IN ('LOW', 'MEDIUM', 'HIGH', 'CRITICAL'))
);

-- Create letters of credit table
CREATE TABLE letters_of_credit (
    lc_number VARCHAR(20) PRIMARY KEY,
    client_id VARCHAR(10) NOT NULL,
    beneficiary_country VARCHAR(3) NOT NULL,
    lc_amount DECIMAL(15,2) NOT NULL,
    outstanding_amount DECIMAL(15,2) NOT NULL,
    lc_status VARCHAR(20) NOT NULL,
    issue_date DATE NOT NULL,
    expiry_date DATE NOT NULL,
    FOREIGN KEY (client_id) REFERENCES trade_clients(client_id),
    FOREIGN KEY (beneficiary_country) REFERENCES country_risk(country_code)
);

-- Create document compliance table
CREATE TABLE lc_documents (
    document_id VARCHAR(20) PRIMARY KEY,
    lc_number VARCHAR(20) NOT NULL,
    document_type VARCHAR(30) NOT NULL,
    required_status VARCHAR(10) NOT NULL CHECK (required_status IN ('REQUIRED', 'OPTIONAL')),
    compliance_status VARCHAR(15) NOT NULL CHECK (compliance_status IN ('COMPLIANT', 'NON_COMPLIANT', 'PENDING')),
    FOREIGN KEY (lc_number) REFERENCES letters_of_credit(lc_number)
);

-- Insert sample data
INSERT INTO country_risk VALUES ('USA', 'United States', 'LOW');
INSERT INTO country_risk VALUES ('IND', 'India', 'MEDIUM');
INSERT INTO country_risk VALUES ('PAK', 'Pakistan', 'HIGH');
INSERT INTO country_risk VALUES ('AFG', 'Afghanistan', 'CRITICAL');

INSERT INTO trade_clients VALUES ('TC001', 'Global Exports Ltd', 3, 'USA');
INSERT INTO trade_clients VALUES ('TC002', 'Asian Trade Corp', 6, 'IND');
INSERT INTO trade_clients VALUES ('TC003', 'Middle East Trading', 8, 'PAK');
INSERT INTO trade_clients VALUES ('TC004', 'International Commodities', 2, 'USA');

INSERT INTO letters_of_credit VALUES ('LC2025001', 'TC001', 'IND', 500000.00, 350000.00, 'ACTIVE', '2025-07-01', '2025-09-15');
INSERT INTO letters_of_credit VALUES ('LC2025002', 'TC002', 'USA', 750000.00, 750000.00, 'ACTIVE', '2025-08-01', '2025-09-30');
INSERT INTO letters_of_credit VALUES ('LC2025003', 'TC003', 'AFG', 1000000.00, 800000.00, 'ACTIVE', '2025-08-15', '2025-09-10');
INSERT INTO letters_of_credit VALUES ('LC2025004', 'TC004', 'PAK', 300000.00, 0.00, 'CLOSED', '2025-06-01', '2025-08-31');
INSERT INTO letters_of_credit VALUES ('LC2025005', 'TC001', 'USA', 400000.00, 400000.00, 'ACTIVE', '2025-08-20', '2025-10-15');

INSERT INTO lc_documents VALUES ('DOC001', 'LC2025001', 'Commercial Invoice', 'REQUIRED', 'COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC002', 'LC2025001', 'Bill of Lading', 'REQUIRED', 'COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC003', 'LC2025001', 'Packing List', 'REQUIRED', 'NON_COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC004', 'LC2025001', 'Certificate of Origin', 'OPTIONAL', 'PENDING');

INSERT INTO lc_documents VALUES ('DOC005', 'LC2025002', 'Commercial Invoice', 'REQUIRED', 'COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC006', 'LC2025002', 'Bill of Lading', 'REQUIRED', 'COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC007', 'LC2025002', 'Insurance Certificate', 'REQUIRED', 'COMPLIANT');

INSERT INTO lc_documents VALUES ('DOC008', 'LC2025003', 'Commercial Invoice', 'REQUIRED', 'NON_COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC009', 'LC2025003', 'Bill of Lading', 'REQUIRED', 'PENDING');
INSERT INTO lc_documents VALUES ('DOC010', 'LC2025003', 'Packing List', 'REQUIRED', 'NON_COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC011', 'LC2025003', 'Certificate of Origin', 'REQUIRED', 'NON_COMPLIANT');

INSERT INTO lc_documents VALUES ('DOC012', 'LC2025005', 'Commercial Invoice', 'REQUIRED', 'COMPLIANT');
INSERT INTO lc_documents VALUES ('DOC013', 'LC2025005', 'Bill of Lading', 'REQUIRED', 'COMPLIANT');
```


## TDD Implementation Guide

### Initial Setup: Create Base/Dummy View

Start with a basic view structure:

```sql
-- Create initial dummy view
CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    'LC0000000' as lc_number,
    'Dummy Client' as client_name,
    'XXX' as beneficiary_country,
    0.00 as lc_amount,
    'UNKNOWN' as lc_status
FROM (VALUES(0)) AS dummy(x);
```


## Pass 1: RED-GREEN-REFACTOR Cycle

### Requirement 1: Display basic LC information

#### RED Phase: Create Test (Should Fail)

```sql
-- Test for Requirement 1: Check if view returns basic LC information
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected Result: 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE
-- But current dummy view will return: LC0000000 | Dummy Client | XXX | 0.00 | UNKNOWN
```

**Run this test** - It will fail because our dummy view doesn't show the correct data.

#### GREEN Phase: Make Test Pass

```sql
-- Drop the dummy view
DROP VIEW trade_finance_risk_assessment;

-- Create view that satisfies Requirement 1
CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    lc.lc_amount,
    lc.lc_status
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id;
```


#### Verify Test Now Passes

```sql
-- Test for Requirement 1: Check if view returns basic LC information
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected and Actual Result: 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE
```

**Run this test** - It should now pass!

#### REFACTOR Phase

```sql
-- Add proper ordering and ensure data consistency
DROP VIEW trade_finance_risk_assessment;

CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    lc.lc_amount,
    lc.lc_status
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id
ORDER BY lc.lc_number;
```


## Pass 2: RED-GREEN-REFACTOR Cycle

### Requirement 2: Include risk metrics

#### RED Phase: Create Test (Should Fail)

```sql
-- Test for Requirement 2: Check if view includes risk metrics
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status,
    outstanding_exposure,
    country_risk_rating,
    client_risk_score
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected Result: 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE | 350000.00 | MEDIUM | 3
-- But current view doesn't have risk columns, so this will fail
```

**Run this test** - It will fail because the risk metric columns don't exist.

#### GREEN Phase: Make Test Pass

```sql
-- Drop the existing view
DROP VIEW trade_finance_risk_assessment;

-- Create view that satisfies Requirements 1 and 2
CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    lc.lc_amount,
    lc.lc_status,
    lc.outstanding_amount as outstanding_exposure,
    cr.risk_rating as country_risk_rating,
    tc.client_risk_score
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id
INNER JOIN country_risk cr ON lc.beneficiary_country = cr.country_code
ORDER BY lc.lc_number;
```


#### Verify Test Now Passes

```sql
-- Test for Requirement 2: Check if view includes risk metrics
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status,
    outstanding_exposure,
    country_risk_rating,
    client_risk_score
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected and Actual Result: 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE | 350000.00 | MEDIUM | 3
```

**Run this test** - It should now pass!

#### REFACTOR Phase

```sql
-- Clean up data types and add better formatting
DROP VIEW trade_finance_risk_assessment;

CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    ROUND(lc.lc_amount, 2) as lc_amount,
    lc.lc_status,
    ROUND(lc.outstanding_amount, 2) as outstanding_exposure,
    cr.risk_rating as country_risk_rating,
    tc.client_risk_score
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id
INNER JOIN country_risk cr ON lc.beneficiary_country = cr.country_code
ORDER BY lc.lc_number;
```


## Pass 3: RED-GREEN-REFACTOR Cycle

### Requirement 3: Add compliance indicators

#### RED Phase: Create Test (Should Fail)

```sql
-- Test for Requirement 3: Check if view includes compliance indicators
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status,
    outstanding_exposure,
    country_risk_rating,
    client_risk_score,
    document_compliance_pct,
    days_until_expiry,
    risk_alert_flag
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected Result: 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE | 350000.00 | MEDIUM | 3 | 66.67 | 16 | Y
-- But current view doesn't have compliance columns, so this will fail
```

**Run this test** - It will fail because the compliance indicator columns don't exist.

#### GREEN Phase: Make Test Pass

```sql

-- Drop the existing view
DROP VIEW IF EXISTS trade_finance_risk_assessment;

-- Create view that satisfies all Requirements 1, 2, and 3 (CORRECTED VERSION)
CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    ROUND(lc.lc_amount, 2) as lc_amount,
    lc.lc_status,
    ROUND(lc.outstanding_amount, 2) as outstanding_exposure,
    cr.risk_rating as country_risk_rating,
    tc.client_risk_score,
    ROUND(
        CASE 
            WHEN COUNT(CASE WHEN d.required_status = 'REQUIRED' THEN 1 END) = 0 THEN 100.00
            ELSE CAST(COUNT(CASE WHEN d.compliance_status = 'COMPLIANT' AND d.required_status = 'REQUIRED' THEN 1 END) AS DECIMAL(10,2)) * 100.0 /
                 CAST(COUNT(CASE WHEN d.required_status = 'REQUIRED' THEN 1 END) AS DECIMAL(10,2))
        END, 2
    ) as document_compliance_pct,
    DATEDIFF('DAY', CURRENT_DATE, lc.expiry_date) as days_until_expiry,
    CASE 
        WHEN DATEDIFF('DAY', CURRENT_DATE, lc.expiry_date) <= 30 
        AND lc.lc_status = 'ACTIVE' 
        THEN 'Y' 
        ELSE 'N' 
    END as risk_alert_flag
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id
INNER JOIN country_risk cr ON lc.beneficiary_country = cr.country_code
LEFT JOIN lc_documents d ON lc.lc_number = d.lc_number
GROUP BY 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    lc.lc_amount,
    lc.lc_status,
    lc.outstanding_amount,
    cr.risk_rating,
    tc.client_risk_score,
    lc.expiry_date
ORDER BY lc.lc_number;



```


#### Verify Test Now Passes

```sql

-- Test for Requirement 3: Check if view includes compliance indicators
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status,
    outstanding_exposure,
    country_risk_rating,
    client_risk_score,
    document_compliance_pct,
    days_until_expiry,
    risk_alert_flag
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025001';

-- Expected and Actual Result (dates may vary based on current date): 
-- LC2025001 | Global Exports Ltd | IND | 500000.00 | ACTIVE | 350000.00 | MEDIUM | 3 | 66.67 | 16 | Y


```

**Run this test** - It should now pass!

#### REFACTOR Phase

```sql
-- Optimize the subquery and add better error handling
DROP VIEW trade_finance_risk_assessment;

CREATE VIEW trade_finance_risk_assessment AS
SELECT 
    lc.lc_number,
    tc.client_name,
    lc.beneficiary_country,
    ROUND(lc.lc_amount, 2) as lc_amount,
    lc.lc_status,
    ROUND(lc.outstanding_amount, 2) as outstanding_exposure,
    cr.risk_rating as country_risk_rating,
    tc.client_risk_score,
    COALESCE(doc_stats.compliance_percentage, 0.00) as document_compliance_pct,
    DATEDIFF('DAY', CURRENT_DATE, lc.expiry_date) as days_until_expiry,
    CASE 
        WHEN DATEDIFF('DAY', CURRENT_DATE, lc.expiry_date) <= 30 
        AND lc.lc_status = 'ACTIVE' 
        THEN 'Y' 
        ELSE 'N' 
    END as risk_alert_flag
FROM letters_of_credit lc
INNER JOIN trade_clients tc ON lc.client_id = tc.client_id
INNER JOIN country_risk cr ON lc.beneficiary_country = cr.country_code
LEFT JOIN (
    SELECT 
        lc_number,
        ROUND(
            CASE 
                WHEN COUNT(CASE WHEN required_status = 'REQUIRED' THEN 1 END) = 0 THEN 100.00
                ELSE CAST(COUNT(CASE WHEN compliance_status = 'COMPLIANT' AND required_status = 'REQUIRED' THEN 1 END) AS DECIMAL(10,2)) * 100.0 / 
                     CAST(COUNT(CASE WHEN required_status = 'REQUIRED' THEN 1 END) AS DECIMAL(10,2))
            END, 2
        ) as compliance_percentage
    FROM lc_documents 
    GROUP BY lc_number
) doc_stats ON lc.lc_number = doc_stats.lc_number
ORDER BY lc.lc_number;
```


## Final Verification Tests

Run these comprehensive tests to ensure all requirements are satisfied:

```sql
-- Test 1: Verify all LCs with complete risk assessment data
SELECT 
    lc_number,
    client_name,
    beneficiary_country,
    lc_amount,
    lc_status,
    outstanding_exposure,
    country_risk_rating,
    client_risk_score,
    document_compliance_pct,
    days_until_expiry,
    risk_alert_flag
FROM trade_finance_risk_assessment
ORDER BY lc_number;

-- Test 2: Verify high-risk LCs (alert flag = 'Y' or critical country risk)
SELECT 
    lc_number,
    client_name,
    outstanding_exposure,
    country_risk_rating,
    document_compliance_pct,
    days_until_expiry,
    risk_alert_flag
FROM trade_finance_risk_assessment
WHERE risk_alert_flag = 'Y' 
   OR country_risk_rating = 'CRITICAL'
   OR document_compliance_pct < 50.00
ORDER BY days_until_expiry ASC;

-- Test 3: Verify compliance calculations for specific LC
SELECT 
    lc_number,
    document_compliance_pct,
    days_until_expiry
FROM trade_finance_risk_assessment
WHERE lc_number = 'LC2025003';

-- Expected Result for LC2025003:
-- Should show low compliance percentage (0.00% as all required docs are non-compliant or pending)
-- Should show risk_alert_flag = 'Y' if expiry is within 30 days


```


## Assignment Completion Checklist

- [x] **Pass 1 Complete**: View shows basic LC information (number, client, country, amount, status)
- [x] **Pass 2 Complete**: View includes risk metrics (outstanding exposure, country risk, client risk score)
- [x] **Pass 3 Complete**: View includes compliance indicators (document compliance %, days until expiry, risk alert flag)
- [x] **RED-GREEN-REFACTOR**: Each requirement followed proper TDD cycle with failing tests, implementation, and refactoring
- [x] **Complex Calculations**: Implemented compliance percentage calculation and risk alerting logic
- [x] **Tests Pass**: All verification tests return expected results


## Learning Objectives Achieved

1. **Advanced TDD**: Followed strict RED-GREEN-REFACTOR cycle for three complex requirements
2. **Complex View Development**: Created view with multiple table joins and subqueries
3. **Calculated Fields**: Implemented percentage calculations, date arithmetic, and conditional logic
4. **Risk Assessment Logic**: Built business rule-based risk indicators and alerts
5. **Data Aggregation**: Used GROUP BY with CASE statements for compliance calculations
6. **Performance Optimization**: Refactored subquery into LEFT JOIN for better performance

## Cleanup Script

```sql
-- Cleanup script - Run this to clean up all created objects
DROP VIEW IF EXISTS trade_finance_risk_assessment;
DROP TABLE IF EXISTS lc_documents;
DROP TABLE IF EXISTS letters_of_credit;
DROP TABLE IF EXISTS country_risk;
DROP TABLE IF EXISTS trade_clients;
```
