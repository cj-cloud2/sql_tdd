# MariaDB TDD SQL Lab: Multi-Column JOIN Operations

## Simple Blogging Website Domain

### **Lab Overview**

This lab guides students through Test-Driven Development (TDD) principles to master SQL JOIN operations in MariaDB 10. Students will implement JOIN statements connecting two tables using multiple columns while following the **RED-GREEN-REFACTOR** cycle.[^1][^2]

**Difficulty Level:** 3
**Duration:** 90-120 minutes
**End Product:** SQL JOIN statement connecting 2 tables on 2 columns

***

## **Business Requirements**

A simple blogging website needs to display comprehensive article information by joining user and article data. The system must:

1. **Show article details with author information** - Display articles along with their author's username and email
2. **Filter articles by specific author and status** - Find articles matching both author ID and publication status
3. **Generate author-article summary reports** - Create reports showing article counts and author activity

***

## **Testable Requirements**

Based on business needs, we define three testable requirements:

1. **R1:** Retrieve all published articles with their author's username and email address
2. **R2:** Find all articles by a specific author (user_id=2) that have 'published' status
3. **R3:** Display article count per author for users who have both published and draft articles

***

## **Pre-Setup Phase**

### **Database and Table Creation**

```sql
-- Create test database
CREATE DATABASE IF NOT EXISTS blog_tdd_lab;
USE blog_tdd_lab;

-- Create users table (authors)
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    registration_date DATE NOT NULL,
    status ENUM('active', 'inactive') DEFAULT 'active'
) ENGINE=InnoDB;

-- Create articles table  
CREATE TABLE articles (
    article_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    created_date DATE NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
) ENGINE=InnoDB;
```


### **Test Data Insertion**

```sql
-- Insert test users
INSERT INTO users (username, email, registration_date, status) VALUES
('john_doe', 'john@example.com', '2024-01-15', 'active'),
('jane_smith', 'jane@example.com', '2024-02-20', 'active'),
('bob_wilson', 'bob@example.com', '2024-03-10', 'active'),
('alice_brown', 'alice@example.com', '2024-04-05', 'inactive');

-- Insert test articles
INSERT INTO articles (user_id, title, content, status, created_date) VALUES
(1, 'Introduction to SQL', 'Learning SQL fundamentals...', 'published', '2024-05-01'),
(1, 'Advanced SQL Techniques', 'Exploring complex queries...', 'draft', '2024-05-15'),
(2, 'Web Development Basics', 'HTML, CSS, and JavaScript...', 'published', '2024-06-01'),
(2, 'React Components Guide', 'Building reusable components...', 'published', '2024-06-15'),
(2, 'Database Design Patterns', 'Effective database modeling...', 'draft', '2024-06-20'),
(3, 'Python Programming', 'Getting started with Python...', 'published', '2024-07-01'),
(4, 'Machine Learning Intro', 'Understanding ML concepts...', 'archived', '2024-07-15');

-- Verify test data
SELECT 'Users Table:', COUNT(*) as user_count FROM users;
SELECT 'Articles Table:', COUNT(*) as article_count FROM articles;
```


***

## **TDD Pass 1: Requirement R1**

### **Retrieve all published articles with author information**

#### **🔴 RED Phase - Write the Test**

```sql
-- Test 1: Verify published articles with author details
-- Expected: 4 published articles with username and email
SELECT 'TEST 1 - Published Articles with Authors' as test_name;

-- Create test validation query
SELECT 
    COUNT(*) as actual_count,
    'Expected: 4 published articles with author details' as expectation
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND a.status = 'published'
HAVING COUNT(*) = 4;

-- This should return 0 rows initially (test fails)
```

**Expected Result:** Test fails (0 rows returned) because we haven't implemented the solution yet.

#### **🟢 GREEN Phase - Implement Minimum Code**

```sql
-- Solution for R1: JOIN articles and users on user_id with published status
SELECT 
    a.article_id,
    a.title,
    a.status,
    a.created_date,
    u.username,
    u.email
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND a.status = 'published'
ORDER BY a.created_date;
```

**Expected Output:**

```
+------------+------------------------+-----------+--------------+------------+------------------+
| article_id | title                  | status    | created_date | username   | email            |
+------------+------------------------+-----------+--------------+------------+------------------+
|          1 | Introduction to SQL    | published | 2024-05-01   | john_doe   | john@example.com |
|          3 | Web Development Basics | published | 2024-06-01   | jane_smith | jane@example.com |
|          4 | React Components Guide | published | 2024-06-15   | jane_smith | jane@example.com |
|          6 | Python Programming     | published | 2024-07-01   | bob_wilson | bob@example.com  |
+------------+------------------------+-----------+--------------+------------+------------------+
```


#### **🔵 REFACTOR Phase - Optimize and Clean**

```sql
-- Refactored solution with better readability and aliases
SELECT 
    a.article_id AS 'Article ID',
    a.title AS 'Article Title',
    u.username AS 'Author',
    u.email AS 'Author Email',
    a.created_date AS 'Publication Date'
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id 
WHERE a.status = 'published'
ORDER BY a.created_date ASC;
```


***

## **TDD Pass 2: Requirement R2**

### **Find articles by specific author with published status**

#### **🔴 RED Phase - Write the Test**

```sql
-- Test 2: Verify specific author's published articles
-- Expected: 2 published articles by jane_smith (user_id=2)
SELECT 'TEST 2 - Jane Smith Published Articles' as test_name;

-- Create test validation query
SELECT 
    COUNT(*) as actual_count,
    'Expected: 2 published articles by user_id=2' as expectation
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND u.user_id = 2
WHERE a.status = 'published'
HAVING COUNT(*) = 2;

-- This should return 0 rows initially (test fails)
```


#### **🟢 GREEN Phase - Implement Solution**

```sql
-- Solution for R2: JOIN with multiple conditions (user_id and status)
SELECT 
    a.article_id,
    a.title,
    u.username,
    u.email,
    a.status,
    a.created_date
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND u.user_id = 2
WHERE a.status = 'published'
ORDER BY a.created_date;
```

**Expected Output:**

```
+------------+------------------------+------------+------------------+-----------+--------------+
| article_id | title                  | username   | email            | status    | created_date |
+------------+------------------------+------------+------------------+-----------+--------------+
|          3 | Web Development Basics | jane_smith | jane@example.com | published | 2024-06-01   |
|          4 | React Components Guide | jane_smith | jane@example.com | published | 2024-06-15   |
+------------+------------------------+------------+------------------+-----------+--------------+
```


#### **🔵 REFACTOR Phase - Enhance Query**

```sql
-- Refactored with parameterized approach and better formatting
SELECT 
    CONCAT('Article #', a.article_id) AS 'Article Reference',
    a.title AS 'Title',
    CONCAT(u.username, ' (', u.email, ')') AS 'Author Details',
    DATE_FORMAT(a.created_date, '%Y-%m-%d') AS 'Published On'
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND u.user_id = 2
WHERE a.status = 'published'
ORDER BY a.created_date DESC;
```


***

## **TDD Pass 3: Requirement R3**

### **Author-article summary for multi-status authors**

#### **🔴 RED Phase - Write the Test**

```sql
-- Test 3: Verify authors with both published and draft articles
-- Expected: 2 authors (john_doe and jane_smith) with mixed status articles
SELECT 'TEST 3 - Multi-Status Author Summary' as test_name;

-- Create test validation query  
SELECT 
    COUNT(DISTINCT u.user_id) as authors_with_mixed_status,
    'Expected: 2 authors with both published and draft articles' as expectation
FROM articles a1
INNER JOIN users u ON a1.user_id = u.user_id AND a1.status = 'published'
INNER JOIN articles a2 ON u.user_id = a2.user_id AND a2.status = 'draft'
HAVING COUNT(DISTINCT u.user_id) = 2;

-- This should return 0 rows initially (test fails)
```


#### **🟢 GREEN Phase - Implement Complex JOIN**

```sql
-- Solution for R3: Multi-table JOIN with aggregation
SELECT 
    u.username,
    u.email,
    COUNT(CASE WHEN a.status = 'published' THEN 1 END) as published_count,
    COUNT(CASE WHEN a.status = 'draft' THEN 1 END) as draft_count,
    COUNT(*) as total_articles
FROM users u
INNER JOIN articles a ON u.user_id = a.user_id 
GROUP BY u.user_id, u.username, u.email
HAVING published_count > 0 AND draft_count > 0
ORDER BY total_articles DESC;
```

**Expected Output:**

```
+------------+------------------+-----------------+-------------+----------------+
| username   | email            | published_count | draft_count | total_articles |
+------------+------------------+-----------------+-------------+----------------+
| jane_smith | jane@example.com |               2 |           1 |              3 |
| john_doe   | john@example.com |               1 |           1 |              2 |
+------------+------------------+-----------------+-------------+----------------+
```


#### **🔵 REFACTOR Phase - Final Optimization**

```sql
-- Refactored with enhanced reporting and status breakdown
SELECT 
    UPPER(u.username) AS 'Author',
    u.email AS 'Contact',
    COUNT(CASE WHEN a.status = 'published' THEN 1 END) AS 'Published',
    COUNT(CASE WHEN a.status = 'draft' THEN 1 END) AS 'Drafts',
    COUNT(CASE WHEN a.status = 'archived' THEN 1 END) AS 'Archived',
    COUNT(*) AS 'Total Articles',
    ROUND(
        (COUNT(CASE WHEN a.status = 'published' THEN 1 END) / COUNT(*)) * 100, 1
    ) AS 'Published %'
FROM users u
INNER JOIN articles a ON u.user_id = a.user_id
GROUP BY u.user_id, u.username, u.email
HAVING COUNT(CASE WHEN a.status = 'published' THEN 1 END) > 0 
   AND COUNT(CASE WHEN a.status = 'draft' THEN 1 END) > 0
ORDER BY `Published %` DESC, `Total Articles` DESC;
```


***

## **Final Validation Tests**

```sql
-- Comprehensive test suite to validate all requirements
SELECT '=== FINAL VALIDATION SUITE ===' as test_header;

-- Test 1 Validation
SELECT 
    'R1 VALIDATION' as requirement,
    COUNT(*) as result_count,
    CASE WHEN COUNT(*) = 4 THEN 'PASS' ELSE 'FAIL' END as status
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id 
WHERE a.status = 'published';

-- Test 2 Validation  
SELECT 
    'R2 VALIDATION' as requirement,
    COUNT(*) as result_count,
    CASE WHEN COUNT(*) = 2 THEN 'PASS' ELSE 'FAIL' END as status
FROM articles a
INNER JOIN users u ON a.user_id = u.user_id AND u.user_id = 2
WHERE a.status = 'published';

-- Test 3 Validation
SELECT 
    'R3 VALIDATION' as requirement,
    COUNT(*) as result_count,
    CASE WHEN COUNT(*) = 2 THEN 'PASS' ELSE 'FAIL' END as status
FROM (
    SELECT u.user_id
    FROM users u
    INNER JOIN articles a ON u.user_id = a.user_id
    GROUP BY u.user_id
    HAVING COUNT(CASE WHEN a.status = 'published' THEN 1 END) > 0 
       AND COUNT(CASE WHEN a.status = 'draft' THEN 1 END) > 0
) validated_authors;
```


***

## **Key Learning Outcomes**

### **JOIN Concepts Mastered**

1. **INNER JOIN syntax** with multiple conditions using AND operator[^2][^1]
2. **Multi-column JOIN conditions** connecting tables on user_id and status fields[^5][^6]
3. **JOIN with aggregation functions** for summary reporting[^7]
4. **Conditional aggregation** using CASE statements within COUNT functions[^1]

### **TDD Benefits Demonstrated**

1. **Failing tests first** ensure requirements are clearly defined[^4][^3]
2. **Incremental development** with immediate feedback on correctness[^8]
3. **Refactoring confidence** through comprehensive test coverage[^9]
4. **Documentation through tests** showing expected behavior[^10]

***

## **Cleanup Script**

```sql
-- Clean up all test artifacts
USE blog_tdd_lab;

-- Drop tables in correct order (foreign key constraints)
DROP TABLE IF EXISTS articles;
DROP TABLE IF EXISTS users;

-- Drop test database
DROP DATABASE IF EXISTS blog_tdd_lab;

-- Verify cleanup
SHOW DATABASES LIKE 'blog_tdd_lab';
-- Should return empty result set

SELECT 'Lab cleanup completed successfully!' as cleanup_status;
```


***

## **Extension Challenges**

For advanced students who complete the lab early:

1. **Add LEFT JOIN** to show users without any articles
2. **Implement RIGHT JOIN** to display all articles including orphaned ones
3. **Create FULL OUTER JOIN simulation** using UNION of LEFT and RIGHT joins
4. **Add performance testing** with EXPLAIN statements for query optimization

***
