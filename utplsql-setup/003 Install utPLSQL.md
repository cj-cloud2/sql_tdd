# Complete Step-by-Step utPLSQL Installation Guide for Oracle 23ai Free

I'll provide a comprehensive guide for installing utPLSQL with clear steps. Since you're new to Oracle, 
I'll explain everything in detail.

## Understanding Oracle Architecture Basics

Oracle 23ai Free uses a "multitenant" architecture:
1. There's a root container (CDB) that manages the database system
2. Your actual data goes in a "pluggable database" (PDB) - typically named `FREEPDB1`
3. We need to work in the PDB, not the root container

## Step 1: Download utPLSQL

1. Visit https://github.com/utPLSQL/utPLSQL/releases
2. Download the latest release (e.g., `utPLSQL.zip`)
3. Extract it to `C:\oracle\utPLSQL`
4. The installation scripts will be in `C:\oracle\utPLSQL\source`

## Step 2: Create appuser with Proper Privileges

Execute these commands in **SQL*Plus** (command prompt):

Start Command prompt
 --> sqlplus / as sysdba
 or --> sqlplus --> usname=sys as sysdba --> password=root

OR BETTER
connect to sqldeveloper un--> sys as sysdba

```sql
-- Connect to your PDB (not the root container) -step 1
CONNECT sys@localhost:1521/FREEPDB1 AS SYSDBA
-- Enter your SYS password when prompted

-- Connect to the PDB- step 2
ALTER SESSION SET CONTAINER = FREEPDB1;

-- Create appuser with all necessary privileges
--CREATE USER appuser IDENTIFIED BY YourPassword123;
CREATE USER appuser IDENTIFIED BY root;

-- Grant basic privileges
GRANT CONNECT, RESOURCE TO appuser;
GRANT CREATE SESSION, CREATE TABLE, CREATE PROCEDURE, CREATE FUNCTION TO appuser;
GRANT CREATE VIEW, CREATE TYPE, CREATE TRIGGER, CREATE SEQUENCE TO appuser;
GRANT UNLIMITED TABLESPACE TO appuser;



-- Additional privileges that might be needed for testing
GRANT CREATE ANY DIRECTORY, DROP ANY DIRECTORY TO appuser;
```






## Step 3: Create ut3 User for utPLSQL Framework

Execute these commands in **SQL*Plus**:

```sql
-- Ensure you're connected to your PDB
CONNECT sys@localhost:1521/FREEPDB1 AS SYSDBA;
ALTER SESSION SET CONTAINER = FREEPDB1;


--DROP USER UT3 CASCADE; (To delete existing user with schema)
-- Create ut3 user
--CREATE USER ut3 IDENTIFIED BY ut3Password123;
CREATE USER ut3 IDENTIFIED BY root;

-- Grant necessary privileges for utPLSQL installation
GRANT CONNECT, RESOURCE TO ut3;
GRANT CREATE VIEW, CREATE TYPE, CREATE TRIGGER, CREATE PROCEDURE TO ut3;
GRANT CREATE SEQUENCE, CREATE SYNONYM, CREATE TABLE TO ut3;
GRANT UNLIMITED TABLESPACE TO ut3;
-- Grant execute permission on DBMS_CRYPTO to UT3
GRANT EXECUTE ON DBMS_CRYPTO TO ut3;
-- Grant execute on other potentially required packages
GRANT EXECUTE ON DBMS_LOCK TO UT3;
GRANT EXECUTE ON DBMS_OUTPUT TO UT3;
GRANT EXECUTE ON UTL_FILE TO UT3;
```











## Step 4: Install utPLSQL in ut3 Schema

Execute this in **SQL*Plus**:

```sql
-- Ensure you're connected to your PDB
CONNECT sys@localhost:1521/FREEPDB1 AS SYSDBA;
ALTER SESSION SET CONTAINER = FREEPDB1;

-- Run the installation script  (This only worked from sqldeveloper)
-- @C:/oracle/utPLSQL/source/uninstall.sql
@C:/oracle/utPLSQL/source/install.sql 


-- When prompted: "Provide schema for the utPLSQL v3 (ut3)"
-- Enter: ut3
```


## Step 4.1: Verify installation

Execute this in **SQL*Plus**:
```sql
-- Connect to your PDB
CONNECT sys@localhost:1521/FREEPDB1 AS SYSDBA;
ALTER SESSION SET CONTAINER = FREEPDB1;

-- Check if UT3 user exists and has objects
SELECT COUNT(*) FROM all_objects WHERE owner = 'UT3';

-- Check if the main utPLSQL package exists
SELECT object_name, object_type, status 
FROM all_objects 
WHERE owner = 'UT3' AND object_name LIKE 'UT%' 
ORDER BY object_type, object_name;
```





## Step 5: Grant appuser Access to utPLSQL

Execute these commands in **SQL*Plus**:

```sql
-- Connect to your PDB
CONNECT sys@localhost:1521/FREEPDB1 AS SYSDBA

ALTER SESSION SET CONTAINER = FREEPDB1;

-- Grant execute privileges on utPLSQL to appuser
GRANT EXECUTE ON ut3.ut TO appuser;
GRANT EXECUTE ON ut3.ut_runner TO appuser;

-- Grant these if you plan to use more advanced features
GRANT EXECUTE ON ut3.ut_expectation TO appuser;
GRANT EXECUTE ON ut3.ut_annotations TO appuser;
GRANT INHERIT PRIVILEGES ON USER ut3 TO appuser;
```

## Step 6: Verify Installation

Execute these commands in **SQL*Plus** as appuser:

```sql
-- Connect as appuser
--CONNECT appuser/YourPassword123@localhost:1521/FREEPDB1
CONNECT appuser/root@localhost:1521/FREEPDB1



-- Test if utPLSQL is accessible
BEGIN
  ut3.ut.run();
END;
/
```

If you see output about tests being run (even if it says no tests found), the installation was successful.

## Step 7: Create a Simple Test (Optional)

Execute these commands in **SQL*Plus** as appuser:

```sql
-- Connect as appuser
CONNECT appuser/YourPassword123@localhost:1521/FREEPDB1

-- Create a simple test package
CREATE OR REPLACE PACKAGE test_example AS
  -- %suite(Example Tests)
  
  -- %test(Check that 1+1=2)
  PROCEDURE test_addition;
END test_example;
/

CREATE OR REPLACE PACKAGE BODY test_example AS
  PROCEDURE test_addition IS
  BEGIN
    ut3.ut.expect(1 + 1).to_equal(2);
  END test_addition;
END test_example;
/

-- Set Enable Server Output
SET SERVEROUTPUT ON SIZE UNLIMITED;
-- Run the test (For Normal Output)
EXEC ut3.ut.run('test_example');



-- Run the test (For XML Output)
SET SERVEROUTPUT ON SIZE UNLIMITED
BEGIN
  ut3.ut.run(
    ut_varchar2_list('test_example'),
    ut3.ut_junit_reporter()
  );
END;

```

## Troubleshooting Tips

1. If you get connection errors, ensure:
   - Your Oracle database is running
   - You're using the correct port (1521) and service name (FREEPDB1)

2. If you get "user does not exist" errors:
   - Ensure you're connected to the PDB (FREEPDB1), not the root container

3. If you get permission errors:
   - Ensure you're executing the installation as SYSDBA

## Using SQL Developer Instead

All these steps can also be executed in SQL Developer:
1. Open SQL Developer
2. Create connections for SYS (with SYSDBA role) and appuser
3. Open worksheets for each connection
4. Execute the same SQL commands

## Final Notes

1. Remember that ut3 owns the utPLSQL framework code
2. appuser can use the framework but doesn't own it
3. This separation is a security best practice
4. You can now create your application tables, procedures, and tests in the appuser schema

This setup gives you a proper environment for development with utPLSQL for unit testing. The appuser has all necessary privileges for development, and ut3 cleanly contains the testing framework.