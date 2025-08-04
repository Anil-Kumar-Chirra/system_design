# Complete SQL Cheatsheet: Basic to Advanced

## Table of Contents
1. [SQL Fundamentals](#sql-fundamentals)
2. [Data Types & Schema Design](#data-types--schema-design)
3. [Basic CRUD Operations](#basic-crud-operations)
4. [Filtering & Sorting](#filtering--sorting)
5. [Joins](#joins)
6. [Aggregations & Grouping](#aggregations--grouping)
7. [Subqueries](#subqueries)
8. [Window Functions](#window-functions)
9. [Common Table Expressions (CTEs)](#common-table-expressions-ctes)
10. [Advanced SQL Techniques](#advanced-sql-techniques)
11. [Performance & Optimization](#performance--optimization)
12. [Interview Questions](#interview-questions)

---

## SQL Fundamentals

### Database Structure Hierarchy
```
Database
├── Tables (Relations)
│   ├── Columns (Attributes)
│   └── Rows (Records/Tuples)
├── Views
├── Indexes
└── Stored Procedures
```

### Basic Syntax Structure
```sql
SELECT column1, column2
FROM table_name
WHERE condition
GROUP BY column
HAVING group_condition
ORDER BY column
LIMIT number;
```

---

## Data Types & Schema Design

### Common Data Types
```sql
-- Numeric Types
INT, BIGINT, SMALLINT          -- Integer types
DECIMAL(10,2), NUMERIC(10,2)   -- Exact decimal
FLOAT, DOUBLE, REAL            -- Floating point

-- String Types
VARCHAR(255)                   -- Variable length string
CHAR(10)                      -- Fixed length string
TEXT                          -- Large text data

-- Date/Time Types
DATE                          -- YYYY-MM-DD
TIME                          -- HH:MM:SS
DATETIME, TIMESTAMP           -- Date and time combined

-- Boolean
BOOLEAN                       -- TRUE/FALSE

-- JSON (Modern databases)
JSON, JSONB                   -- JSON data
```

### Schema Design Example
```sql
-- Users table
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Posts table
CREATE TABLE posts (
    post_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Comments table
CREATE TABLE comments (
    comment_id INT PRIMARY KEY AUTO_INCREMENT,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    content TEXT NOT NULL,
    parent_comment_id INT DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (parent_comment_id) REFERENCES comments(comment_id)
);
```

---

## Basic CRUD Operations

### CREATE (INSERT)
```sql
-- Single row insert
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john@example.com', 'hashed_password');

-- Multiple rows insert
INSERT INTO users (username, email, password_hash)
VALUES 
    ('alice', 'alice@example.com', 'hash1'),
    ('bob', 'bob@example.com', 'hash2'),
    ('charlie', 'charlie@example.com', 'hash3');

-- Insert from SELECT
INSERT INTO archived_posts (post_id, title, content)
SELECT post_id, title, content 
FROM posts 
WHERE status = 'archived';

-- Insert with ON DUPLICATE KEY UPDATE (MySQL)
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john@example.com', 'new_hash')
ON DUPLICATE KEY UPDATE 
    password_hash = VALUES(password_hash),
    updated_at = CURRENT_TIMESTAMP;
```

### READ (SELECT)
```sql
-- Basic SELECT
SELECT user_id, username, email FROM users;

-- SELECT with aliases
SELECT 
    u.username AS user_name,
    COUNT(p.post_id) AS post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id, u.username;

-- SELECT DISTINCT
SELECT DISTINCT status FROM posts;

-- Conditional SELECT
SELECT 
    CASE 
        WHEN status = 'published' THEN 'Public'
        WHEN status = 'draft' THEN 'Private'
        ELSE 'Unknown'
    END AS visibility,
    COUNT(*) as count
FROM posts
GROUP BY status;
```

### UPDATE
```sql
-- Basic UPDATE
UPDATE users 
SET email = 'newemail@example.com'
WHERE user_id = 1;

-- UPDATE with JOIN
UPDATE posts p
JOIN users u ON p.user_id = u.user_id
SET p.status = 'archived'
WHERE u.username = 'inactive_user';

-- Conditional UPDATE
UPDATE posts 
SET status = CASE 
    WHEN created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR) THEN 'archived'
    WHEN status = 'draft' AND created_at < DATE_SUB(NOW(), INTERVAL 30 DAY) THEN 'expired'
    ELSE status
END;
```

### DELETE
```sql
-- Basic DELETE
DELETE FROM comments 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- DELETE with JOIN
DELETE p 
FROM posts p
JOIN users u ON p.user_id = u.user_id
WHERE u.username = 'deleted_user';

-- Safe DELETE with LIMIT
DELETE FROM logs 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 30 DAY)
LIMIT 1000;
```

---

## Filtering & Sorting

### WHERE Clause Operations
```sql
-- Comparison operators
SELECT * FROM posts WHERE post_id = 1;
SELECT * FROM posts WHERE post_id != 1;
SELECT * FROM posts WHERE post_id > 10;
SELECT * FROM posts WHERE post_id BETWEEN 10 AND 20;

-- String operations
SELECT * FROM users WHERE username LIKE 'john%';        -- Starts with 'john'
SELECT * FROM users WHERE username LIKE '%doe';         -- Ends with 'doe'
SELECT * FROM users WHERE username LIKE '%admin%';      -- Contains 'admin'
SELECT * FROM users WHERE username REGEXP '^[a-z]+$';   -- Regex pattern

-- NULL handling
SELECT * FROM posts WHERE content IS NULL;
SELECT * FROM posts WHERE content IS NOT NULL;
SELECT * FROM posts WHERE COALESCE(content, '') != '';

-- IN operator
SELECT * FROM posts WHERE status IN ('published', 'featured');
SELECT * FROM posts WHERE user_id IN (
    SELECT user_id FROM users WHERE created_at > '2023-01-01'
);

-- Date/Time filtering
SELECT * FROM posts WHERE DATE(created_at) = '2023-12-01';
SELECT * FROM posts WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);
SELECT * FROM posts WHERE YEAR(created_at) = 2023;
SELECT * FROM posts WHERE MONTH(created_at) = 12;
```

### ORDER BY & LIMIT
```sql
-- Basic sorting
SELECT * FROM posts ORDER BY created_at DESC;
SELECT * FROM posts ORDER BY title ASC, created_at DESC;

-- Custom sorting
SELECT * FROM posts 
ORDER BY 
    CASE status
        WHEN 'featured' THEN 1
        WHEN 'published' THEN 2
        WHEN 'draft' THEN 3
        ELSE 4
    END,
    created_at DESC;

-- Pagination
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 20;  -- Skip 20, take 10 (page 3)

-- Top N per group
SELECT * FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
    FROM posts
) ranked
WHERE rn <= 3;  -- Top 3 posts per user
```

---

## Joins

### Visual Join Representation
```
TABLE A          TABLE B
┌─────────┐     ┌─────────┐
│ ID │ Val│     │ ID │ Val│
├────┼────┤     ├────┼────┤
│ 1  │ A  │     │ 1  │ X  │
│ 2  │ B  │     │ 3  │ Y  │
│ 3  │ C  │     │ 4  │ Z  │
└────┴────┘     └────┴────┘

INNER JOIN: Returns only matching records
┌────┬────┬────┬────┐
│ ID │Val │ ID │Val │
├────┼────┼────┼────┤
│ 1  │ A  │ 1  │ X  │
│ 3  │ C  │ 3  │ Y  │
└────┴────┴────┴────┘

LEFT JOIN: All from A + matching from B
┌────┬────┬─────┬─────┐
│ ID │Val │ ID  │ Val │
├────┼────┼─────┼─────┤
│ 1  │ A  │ 1   │ X   │
│ 2  │ B  │NULL │NULL │
│ 3  │ C  │ 3   │ Y   │
└────┴────┴─────┴─────┘
```

### Join Types with Examples
```sql
-- INNER JOIN (most common)
SELECT u.username, p.title, p.created_at
FROM users u
INNER JOIN posts p ON u.user_id = p.user_id
WHERE p.status = 'published';

-- LEFT JOIN (all users, even without posts)
SELECT u.username, COUNT(p.post_id) as post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id, u.username;

-- RIGHT JOIN (rarely used)
SELECT u.username, p.title
FROM users u
RIGHT JOIN posts p ON u.user_id = p.user_id;

-- FULL OUTER JOIN (MySQL doesn't support, use UNION)
SELECT u.username, p.title
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
UNION
SELECT u.username, p.title
FROM users u
RIGHT JOIN posts p ON u.user_id = p.user_id;

-- CROSS JOIN (Cartesian product)
SELECT u.username, s.status_name
FROM users u
CROSS JOIN (SELECT 'active' as status_name UNION SELECT 'inactive') s;

-- Self JOIN (hierarchical data)
SELECT 
    c1.content as comment,
    c2.content as reply
FROM comments c1
LEFT JOIN comments c2 ON c1.comment_id = c2.parent_comment_id
WHERE c1.parent_comment_id IS NULL;

-- Multiple JOINs
SELECT 
    u.username,
    p.title,
    COUNT(c.comment_id) as comment_count
FROM users u
INNER JOIN posts p ON u.user_id = p.user_id
LEFT JOIN comments c ON p.post_id = c.post_id
WHERE p.status = 'published'
GROUP BY u.user_id, p.post_id;
```

---

## Aggregations & Grouping

### Aggregate Functions
```sql
-- Basic aggregates
SELECT 
    COUNT(*) as total_posts,
    COUNT(DISTINCT user_id) as unique_authors,
    AVG(CHAR_LENGTH(content)) as avg_content_length,
    MIN(created_at) as first_post,
    MAX(created_at) as latest_post,
    SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published_count
FROM posts;

-- String aggregation (MySQL)
SELECT 
    user_id,
    GROUP_CONCAT(title ORDER BY created_at DESC SEPARATOR ' | ') as all_titles
FROM posts
GROUP BY user_id;

-- Conditional aggregation
SELECT 
    user_id,
    COUNT(*) as total_posts,
    COUNT(CASE WHEN status = 'published' THEN 1 END) as published_posts,
    COUNT(CASE WHEN status = 'draft' THEN 1 END) as draft_posts,
    AVG(CASE WHEN status = 'published' THEN CHAR_LENGTH(content) END) as avg_published_length
FROM posts
GROUP BY user_id;
```

### GROUP BY & HAVING
```sql
-- Basic grouping
SELECT 
    status,
    COUNT(*) as count,
    AVG(CHAR_LENGTH(content)) as avg_length
FROM posts
GROUP BY status;

-- Multiple column grouping
SELECT 
    user_id,
    status,
    YEAR(created_at) as year,
    COUNT(*) as posts_count
FROM posts
GROUP BY user_id, status, YEAR(created_at)
ORDER BY user_id, year DESC;

-- HAVING clause (filter after grouping)
SELECT 
    user_id,
    COUNT(*) as post_count,
    AVG(CHAR_LENGTH(content)) as avg_length
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 5 AND AVG(CHAR_LENGTH(content)) > 1000;

-- GROUP BY with ROLLUP (subtotals)
SELECT 
    COALESCE(status, 'TOTAL') as status,
    COALESCE(YEAR(created_at), 'ALL_YEARS') as year,
    COUNT(*) as count
FROM posts
GROUP BY status, YEAR(created_at) WITH ROLLUP;
```

---

## Subqueries

### Types of Subqueries
```sql
-- Scalar subquery (returns single value)
SELECT username, email,
    (SELECT COUNT(*) FROM posts WHERE posts.user_id = users.user_id) as post_count
FROM users;

-- Column subquery (returns single column)
SELECT * FROM posts
WHERE user_id IN (
    SELECT user_id FROM users WHERE created_at > '2023-01-01'
);

-- Row subquery (returns single row)
SELECT * FROM posts
WHERE (user_id, status) = (
    SELECT user_id, 'published' FROM users WHERE username = 'admin'
);

-- Table subquery (returns multiple rows/columns)
SELECT * FROM (
    SELECT user_id, COUNT(*) as post_count
    FROM posts
    GROUP BY user_id
) user_stats
WHERE post_count > 10;
```

### Advanced Subquery Patterns
```sql
-- EXISTS (check for existence)
SELECT u.username
FROM users u
WHERE EXISTS (
    SELECT 1 FROM posts p 
    WHERE p.user_id = u.user_id 
    AND p.status = 'published'
);

-- NOT EXISTS (anti-join pattern)
SELECT u.username
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM posts p WHERE p.user_id = u.user_id
);

-- Correlated subquery (subquery references outer query)
SELECT p.title, p.created_at
FROM posts p
WHERE p.created_at = (
    SELECT MAX(p2.created_at)
    FROM posts p2
    WHERE p2.user_id = p.user_id
);

-- ANY/ALL operators
SELECT * FROM posts
WHERE CHAR_LENGTH(content) > ALL (
    SELECT AVG(CHAR_LENGTH(content))
    FROM posts
    GROUP BY user_id
);

-- Subquery in FROM clause (derived table)
SELECT 
    user_stats.username,
    user_stats.post_count,
    CASE 
        WHEN user_stats.post_count > 20 THEN 'Prolific'
        WHEN user_stats.post_count > 5 THEN 'Active'
        ELSE 'Casual'
    END as user_type
FROM (
    SELECT 
        u.username,
        COUNT(p.post_id) as post_count
    FROM users u
    LEFT JOIN posts p ON u.user_id = p.user_id
    GROUP BY u.user_id, u.username
) user_stats;
```

---

## Window Functions

### Basic Window Function Syntax
```sql
SELECT 
    column1,
    column2,
    WINDOW_FUNCTION() OVER (
        [PARTITION BY column]
        [ORDER BY column]
        [ROWS/RANGE window_frame]
    ) as window_result
FROM table_name;
```

### Ranking Functions
```sql
-- ROW_NUMBER: Sequential numbering
SELECT 
    username,
    post_count,
    ROW_NUMBER() OVER (ORDER BY post_count DESC) as row_num
FROM (
    SELECT u.username, COUNT(p.post_id) as post_count
    FROM users u LEFT JOIN posts p ON u.user_id = p.user_id
    GROUP BY u.user_id, u.username
) user_stats;

-- RANK: Ranking with gaps for ties
SELECT 
    title,
    CHAR_LENGTH(content) as length,
    RANK() OVER (ORDER BY CHAR_LENGTH(content) DESC) as rank_with_gaps
FROM posts;

-- DENSE_RANK: Ranking without gaps
SELECT 
    title,
    CHAR_LENGTH(content) as length,
    DENSE_RANK() OVER (ORDER BY CHAR_LENGTH(content) DESC) as dense_rank
FROM posts;

-- NTILE: Divide into N buckets
SELECT 
    username,
    post_count,
    NTILE(4) OVER (ORDER BY post_count) as quartile
FROM user_post_counts;
```

### Analytical Functions
```sql
-- LAG/LEAD: Access previous/next row
SELECT 
    title,
    created_at,
    LAG(created_at, 1) OVER (PARTITION BY user_id ORDER BY created_at) as prev_post_date,
    LEAD(created_at, 1) OVER (PARTITION BY user_id ORDER BY created_at) as next_post_date
FROM posts;

-- FIRST_VALUE/LAST_VALUE: First/last value in window
SELECT 
    title,
    created_at,
    FIRST_VALUE(title) OVER (
        PARTITION BY user_id 
        ORDER BY created_at 
        ROWS UNBOUNDED PRECEDING
    ) as first_post_title,
    LAST_VALUE(title) OVER (
        PARTITION BY user_id 
        ORDER BY created_at 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as last_post_title
FROM posts;

-- Cumulative and moving aggregates
SELECT 
    title,
    created_at,
    CHAR_LENGTH(content) as length,
    SUM(CHAR_LENGTH(content)) OVER (
        PARTITION BY user_id 
        ORDER BY created_at 
        ROWS UNBOUNDED PRECEDING
    ) as cumulative_length,
    AVG(CHAR_LENGTH(content)) OVER (
        PARTITION BY user_id 
        ORDER BY created_at 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3_posts
FROM posts;
```

### Advanced Window Patterns
```sql
-- Percent of total
SELECT 
    status,
    COUNT(*) as count,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () as percent_of_total
FROM posts
GROUP BY status;

-- Running percentage
SELECT 
    user_id,
    title,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) as post_sequence,
    COUNT(*) OVER (PARTITION BY user_id) as total_posts,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) * 100.0 / 
    COUNT(*) OVER (PARTITION BY user_id) as percent_complete
FROM posts;

-- Detect streaks/islands
SELECT 
    user_id,
    DATE(created_at) as post_date,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) -
    ROW_NUMBER() OVER (PARTITION BY user_id, DATE(created_at) ORDER BY created_at) as streak_group
FROM posts
ORDER BY user_id, created_at;
```

---

## Common Table Expressions (CTEs)

### Basic CTE Syntax
```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name
WHERE another_condition;
```

### Simple CTE Examples
```sql
-- Basic CTE for readability
WITH active_users AS (
    SELECT user_id, username, email
    FROM users
    WHERE created_at > DATE_SUB(NOW(), INTERVAL 30 DAY)
),
user_post_counts AS (
    SELECT 
        au.user_id,
        au.username,
        COUNT(p.post_id) as post_count
    FROM active_users au
    LEFT JOIN posts p ON au.user_id = p.user_id
    GROUP BY au.user_id, au.username
)
SELECT *
FROM user_post_counts
WHERE post_count > 5;

-- Multiple CTEs
WITH monthly_stats AS (
    SELECT 
        YEAR(created_at) as year,
        MONTH(created_at) as month,
        COUNT(*) as post_count,
        COUNT(DISTINCT user_id) as unique_users
    FROM posts
    GROUP BY YEAR(created_at), MONTH(created_at)
),
growth_analysis AS (
    SELECT 
        year,
        month,
        post_count,
        unique_users,
        LAG(post_count, 1) OVER (ORDER BY year, month) as prev_month_posts,
        post_count - LAG(post_count, 1) OVER (ORDER BY year, month) as growth
    FROM monthly_stats
)
SELECT 
    year,
    month,
    post_count,
    growth,
    CASE 
        WHEN growth > 0 THEN 'Growing'
        WHEN growth < 0 THEN 'Declining'
        ELSE 'Stable'
    END as trend
FROM growth_analysis
WHERE prev_month_posts IS NOT NULL;
```

### Recursive CTEs
```sql
-- Hierarchical data traversal (comment threads)
WITH RECURSIVE comment_thread AS (
    -- Base case: root comments
    SELECT 
        comment_id,
        post_id,
        user_id,
        content,
        parent_comment_id,
        0 as depth,
        CAST(comment_id AS CHAR(1000)) as path
    FROM comments
    WHERE parent_comment_id IS NULL
    
    UNION ALL
    
    -- Recursive case: child comments
    SELECT 
        c.comment_id,
        c.post_id,
        c.user_id,
        c.content,
        c.parent_comment_id,
        ct.depth + 1,
        CONCAT(ct.path, ' -> ', c.comment_id)
    FROM comments c
    INNER JOIN comment_thread ct ON c.parent_comment_id = ct.comment_id
    WHERE ct.depth < 10  -- Prevent infinite recursion
)
SELECT 
    REPEAT('  ', depth) || content as indented_content,
    depth,
    path
FROM comment_thread
ORDER BY path;

-- Generate series (numbers 1-100)
WITH RECURSIVE number_series AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1
    FROM number_series
    WHERE n < 100
)
SELECT n FROM number_series;

-- Date series generation
WITH RECURSIVE date_series AS (
    SELECT DATE('2023-01-01') as date_val
    UNION ALL
    SELECT DATE_ADD(date_val, INTERVAL 1 DAY)
    FROM date_series
    WHERE date_val < '2023-12-31'
)
SELECT 
    ds.date_val,
    COALESCE(daily_posts.post_count, 0) as posts_that_day
FROM date_series ds
LEFT JOIN (
    SELECT DATE(created_at) as post_date, COUNT(*) as post_count
    FROM posts
    GROUP BY DATE(created_at)
) daily_posts ON ds.date_val = daily_posts.post_date;
```

---

## Advanced SQL Techniques

### PIVOT Operations (Manual)
```sql
-- Convert rows to columns
SELECT 
    user_id,
    SUM(CASE WHEN status = 'published' THEN 1 ELSE 0 END) as published,
    SUM(CASE WHEN status = 'draft' THEN 1 ELSE 0 END) as draft,
    SUM(CASE WHEN status = 'archived' THEN 1 ELSE 0 END) as archived
FROM posts
GROUP BY user_id;

-- Dynamic pivot with GROUP_CONCAT
SELECT 
    user_id,
    GROUP_CONCAT(
        CASE WHEN status = 'published' THEN title END 
        ORDER BY created_at DESC 
        SEPARATOR ' | '
    ) as published_titles
FROM posts
GROUP BY user_id;
```

### Advanced Date/Time Operations
```sql
-- Date arithmetic and functions
SELECT 
    title,
    created_at,
    DATEDIFF(NOW(), created_at) as days_old,
    TIMESTAMPDIFF(HOUR, created_at, NOW()) as hours_old,
    DATE_FORMAT(created_at, '%W, %M %e, %Y') as formatted_date,
    CASE 
        WHEN created_at > DATE_SUB(NOW(), INTERVAL 1 DAY) THEN 'Today'
        WHEN created_at > DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 'This Week'
        WHEN created_at > DATE_SUB(NOW(), INTERVAL 30 DAY) THEN 'This Month'
        ELSE 'Older'
    END as age_category
FROM posts;

-- Working with time zones
SELECT 
    title,
    created_at as utc_time,
    CONVERT_TZ(created_at, '+00:00', '+05:30') as ist_time,
    UNIX_TIMESTAMP(created_at) as unix_timestamp
FROM posts;

-- Generate time-based reports
SELECT 
    DATE_FORMAT(created_at, '%Y-%m') as month,
    DAYNAME(created_at) as day_of_week,
    HOUR(created_at) as hour_of_day,
    COUNT(*) as post_count
FROM posts
GROUP BY 
    DATE_FORMAT(created_at, '%Y-%m'),
    DAYNAME(created_at),
    HOUR(created_at)
ORDER BY month, FIELD(day_of_week, 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'), hour_of_day;
```

### JSON Operations (MySQL 5.7+)
```sql
-- Working with JSON data
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    profile_data JSON
);

INSERT INTO user_profiles VALUES 
(1, '{"name": "John", "age": 30, "skills": ["Python", "SQL", "Docker"], "address": {"city": "NYC", "country": "USA"}}'),
(2, '{"name": "Alice", "age": 25, "skills": ["Java", "Kubernetes"], "address": {"city": "SF", "country": "USA"}}');

-- JSON extraction
SELECT 
    user_id,
    JSON_EXTRACT(profile_data, '$.name') as name,
    JSON_EXTRACT(profile_data, '$.age') as age,
    JSON_EXTRACT(profile_data, '$.address.city') as city,
    JSON_LENGTH(profile_data, '$.skills') as skill_count
FROM user_profiles;

-- JSON array operations
SELECT user_id
FROM user_profiles
WHERE JSON_SEARCH(profile_data, 'one', 'Python', NULL, '$.skills[*]') IS NOT NULL;

-- JSON aggregation
SELECT 
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'user_id', user_id,
            'name', JSON_EXTRACT(profile_data, '$.name'),
            'skill_count', JSON_LENGTH(profile_data, '$.skills')
        )
    ) as users_summary
FROM user_profiles;
```

### Set Operations
```sql
-- UNION (combine results, remove duplicates)
SELECT user_id, 'post_author' as role FROM posts
UNION
SELECT user_id, 'commenter' as role FROM comments;

-- UNION ALL (combine results, keep duplicates)
SELECT user_id FROM posts
UNION ALL
SELECT user_id FROM comments;

-- INTERSECT (common records) - MySQL simulation
SELECT DISTINCT p.user_id
FROM posts p
INNER JOIN comments c ON p.user_id = c.user_id;

-- EXCEPT (difference) - MySQL simulation
SELECT DISTINCT p.user_id
FROM posts p
LEFT JOIN comments c ON p.user_id = c.user_id
WHERE c.user_id IS NULL;
```

---

## Performance & Optimization

### Index Strategies
```sql
-- Single column index
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_created_at ON posts(created_at);

-- Composite index (order matters!)
CREATE INDEX idx_posts_user_status_date ON posts(user_id, status, created_at);

-- Covering index (includes all needed columns)
CREATE INDEX idx_posts_covering ON posts(user_id, status) INCLUDE (title, created_at);

-- Partial index (MySQL 8.0+)
CREATE INDEX idx_published_posts ON posts(created_at) WHERE status = 'published';

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Full-text index
CREATE FULLTEXT INDEX idx_posts_content ON posts(title, content);

-- Check index usage
EXPLAIN SELECT * FROM posts WHERE status = 'published' AND user_id = 123;
```

### Query Optimization Techniques
```sql
-- Use EXISTS instead of IN for large subqueries
-- ❌ Slower
SELECT * FROM users 
WHERE user_id IN (SELECT user_id FROM posts WHERE status = 'published');

-- ✅ Faster
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.user_id AND p.status = 'published');

-- Use JOINs instead of subqueries when possible
-- ❌ Slower
SELECT u.*, 
    (SELECT COUNT(*) FROM posts WHERE user_id = u.user_id) as post_count
FROM users u;

-- ✅ Faster
SELECT u.*, COUNT(p.post_id) as post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id;

-- Avoid SELECT * in production
-- ❌ Bad
SELECT * FROM posts WHERE status = 'published';

-- ✅ Good
SELECT post_id, title, created_at FROM posts WHERE status = 'published';

-- Use LIMIT for large result sets
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 20;

-- Optimize OR conditions with UNION
-- ❌ Slower (can't use indexes effectively)
SELECT * FROM posts WHERE status = 'published' OR user_id = 123;

-- ✅ Faster (can use separate indexes)
SELECT * FROM posts WHERE status = 'published'
UNION
SELECT * FROM posts WHERE user_id = 123;
```

### Query Analysis Tools
```sql
-- EXPLAIN PLAN (shows execution strategy)
EXPLAIN FORMAT=JSON 
SELECT p.title, u.username
FROM posts p
JOIN users u ON p.user_id = u.user_id
WHERE p.status = 'published'
ORDER BY p.created_at DESC
LIMIT 10;

-- SHOW PROFILE (detailed timing)
SET profiling = 1;
SELECT COUNT(*) FROM posts WHERE status = 'published';
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;

-- Index usage analysis
SELECT 
    TABLE_NAME,
    INDEX_NAME,
    CARDINALITY,
    NULLABLE
FROM INFORMATION_SCHEMA.STATISTICS 
WHERE TABLE_SCHEMA = 'your_database';
```

### Database Design Best Practices
```sql
-- Normalization example
-- ❌ Denormalized (data duplication)
CREATE TABLE order_items_bad (
    item_id INT,
    order_id INT,
    product_name VARCHAR(100),
    product_category VARCHAR(50),
    product_price DECIMAL(10,2),
    quantity INT
);

-- ✅ Normalized (3NF)
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT,
    price DECIMAL(10,2),
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price_at_time DECIMAL(10,2),  -- Historical price
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

## Interview Questions

### Easy Level Questions
```sql
-- 1. Find the second highest salary
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Alternative with window function
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank_num
    FROM employees
) ranked
WHERE rank_num = 2;

-- 2. Find employees with no manager
SELECT employee_id, name
FROM employees
WHERE manager_id IS NULL;

-- 3. Count employees by department
SELECT 
    department,
    COUNT(*) as employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC;

-- 4. Find duplicate emails
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- 5. Delete duplicate records (keep one)
DELETE u1 FROM users u1
INNER JOIN users u2
WHERE u1.user_id > u2.user_id 
AND u1.email = u2.email;
```

### Medium Level Questions
```sql
-- 1. Find departments with average salary > company average
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > (
    SELECT AVG(salary) FROM employees
);

-- 2. Rank employees by salary within each department
SELECT 
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;

-- 3. Find customers who placed orders in consecutive months
WITH monthly_orders AS (
    SELECT 
        customer_id,
        YEAR(order_date) as year,
        MONTH(order_date) as month
    FROM orders
    GROUP BY customer_id, YEAR(order_date), MONTH(order_date)
),
with_prev_month AS (
    SELECT 
        customer_id,
        year,
        month,
        LAG(month) OVER (PARTITION BY customer_id ORDER BY year, month) as prev_month,
        LAG(year) OVER (PARTITION BY customer_id ORDER BY year, month) as prev_year
    FROM monthly_orders
)
SELECT DISTINCT customer_id
FROM with_prev_month
WHERE (month = prev_month + 1 AND year = prev_year)
   OR (month = 1 AND prev_month = 12 AND year = prev_year + 1);

-- 4. Calculate running total of sales
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY order_date ROWS UNBOUNDED PRECEDING) as running_total
FROM (
    SELECT 
        DATE(order_date) as order_date,
        SUM(amount) as daily_sales
    FROM orders
    GROUP BY DATE(order_date)
) daily_totals
ORDER BY order_date;

-- 5. Find the most popular product in each category
WITH product_sales AS (
    SELECT 
        p.category,
        p.product_name,
        COUNT(oi.product_id) as sales_count,
        RANK() OVER (PARTITION BY p.category ORDER BY COUNT(oi.product_id) DESC) as rank_num
    FROM products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    GROUP BY p.category, p.product_id, p.product_name
)
SELECT category, product_name, sales_count
FROM product_sales
WHERE rank_num = 1;
```

### Hard Level Questions
```sql
-- 1. Find users who made purchases on 3+ consecutive days
WITH daily_purchases AS (
    SELECT 
        user_id,
        DATE(purchase_date) as purchase_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY DATE(purchase_date)) as rn
    FROM purchases
    GROUP BY user_id, DATE(purchase_date)
),
streak_groups AS (
    SELECT 
        user_id,
        purchase_date,
        DATE_SUB(purchase_date, INTERVAL rn DAY) as streak_id
    FROM daily_purchases
),
streak_lengths AS (
    SELECT 
        user_id,
        streak_id,
        COUNT(*) as consecutive_days
    FROM streak_groups
    GROUP BY user_id, streak_id
)
SELECT DISTINCT user_id
FROM streak_lengths
WHERE consecutive_days >= 3;

-- 2. Calculate median salary by department
WITH ranked_salaries AS (
    SELECT 
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary) as row_num,
        COUNT(*) OVER (PARTITION BY department) as total_count
    FROM employees
)
SELECT 
    department,
    AVG(salary) as median_salary
FROM ranked_salaries
WHERE row_num IN (
    FLOOR((total_count + 1) / 2),
    CEIL((total_count + 1) / 2)
)
GROUP BY department;

-- 3. Find customers with increasing order values for 3+ consecutive orders
WITH order_sequence AS (
    SELECT 
        customer_id,
        order_id,
        order_amount,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) as order_seq,
        LAG(order_amount) OVER (PARTITION BY customer_id ORDER BY order_date) as prev_amount,
        LAG(order_amount, 2) OVER (PARTITION BY customer_id ORDER BY order_date) as prev_prev_amount
    FROM orders
),
increasing_sequences AS (
    SELECT 
        customer_id,
        order_id,
        CASE 
            WHEN order_amount > prev_amount AND prev_amount > prev_prev_amount 
            THEN 1 ELSE 0 
        END as is_increasing_third
    FROM order_sequence
    WHERE prev_prev_amount IS NOT NULL
)
SELECT DISTINCT customer_id
FROM increasing_sequences
WHERE is_increasing_third = 1;

-- 4. Hierarchical query - Employee reporting structure
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        0 as level,
        CAST(name AS CHAR(1000)) as hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Subordinates
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1,
        CONCAT(eh.hierarchy_path, ' -> ', e.name)
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
    WHERE eh.level < 10  -- Prevent infinite loops
)
SELECT 
    employee_id,
    REPEAT('  ', level) || name as indented_name,
    level,
    hierarchy_path
FROM employee_hierarchy
ORDER BY hierarchy_path;

-- 5. Advanced analytics - Customer cohort analysis
WITH first_purchase AS (
    SELECT 
        customer_id,
        MIN(DATE(order_date)) as first_purchase_date
    FROM orders
    GROUP BY customer_id
),
monthly_cohorts AS (
    SELECT 
        DATE_FORMAT(first_purchase_date, '%Y-%m') as cohort_month,
        customer_id
    FROM first_purchase
),
customer_activities AS (
    SELECT 
        mc.cohort_month,
        mc.customer_id,
        DATE_FORMAT(o.order_date, '%Y-%m') as activity_month,
        PERIOD_DIFF(
            DATE_FORMAT(o.order_date, '%Y%m'),
            DATE_FORMAT(STR_TO_DATE(mc.cohort_month, '%Y-%m'), '%Y%m')
        ) as months_since_first_purchase
    FROM monthly_cohorts mc
    LEFT JOIN orders o ON mc.customer_id = o.customer_id
),
cohort_analysis AS (
    SELECT 
        cohort_month,
        months_since_first_purchase,
        COUNT(DISTINCT customer_id) as active_customers
    FROM customer_activities
    WHERE activity_month IS NOT NULL
    GROUP BY cohort_month, months_since_first_purchase
),
cohort_sizes AS (
    SELECT 
        cohort_month,
        COUNT(DISTINCT customer_id) as cohort_size
    FROM monthly_cohorts
    GROUP BY cohort_month
)
SELECT 
    ca.cohort_month,
    ca.months_since_first_purchase,
    ca.active_customers,
    cs.cohort_size,
    ROUND(ca.active_customers * 100.0 / cs.cohort_size, 2) as retention_rate
FROM cohort_analysis ca
JOIN cohort_sizes cs ON ca.cohort_month = cs.cohort_month
ORDER BY ca.cohort_month, ca.months_since_first_purchase;
```

---

## Company-Specific SQL Patterns

### Big Tech Interview Patterns
```sql
-- 1. Facebook: Friend Recommendations
-- Find mutual friends for friend suggestions
WITH user_friends AS (
    SELECT user_id, friend_id FROM friendships
    UNION ALL
    SELECT friend_id, user_id FROM friendships
),
mutual_friends AS (
    SELECT 
        uf1.user_id as user1,
        uf2.user_id as user2,
        COUNT(*) as mutual_count
    FROM user_friends uf1
    JOIN user_friends uf2 ON uf1.friend_id = uf2.friend_id
    WHERE uf1.user_id != uf2.user_id
    AND NOT EXISTS (
        SELECT 1 FROM user_friends uf3 
        WHERE uf3.user_id = uf1.user_id AND uf3.friend_id = uf2.user_id
    )
    GROUP BY uf1.user_id, uf2.user_id
)
SELECT user1, user2, mutual_count
FROM mutual_friends
WHERE mutual_count >= 2
ORDER BY user1, mutual_count DESC;

-- 2. Amazon: Product Recommendations
-- Products frequently bought together
WITH order_products AS (
    SELECT order_id, product_id
    FROM order_items
),
product_pairs AS (
    SELECT 
        op1.product_id as product1,
        op2.product_id as product2,
        COUNT(*) as times_bought_together
    FROM order_products op1
    JOIN order_products op2 ON op1.order_id = op2.order_id
    WHERE op1.product_id < op2.product_id  -- Avoid duplicates
    GROUP BY op1.product_id, op2.product_id
)
SELECT 
    p1.product_name as product1_name,
    p2.product_name as product2_name,
    pp.times_bought_together
FROM product_pairs pp
JOIN products p1 ON pp.product1 = p1.product_id
JOIN products p2 ON pp.product2 = p2.product_id
WHERE pp.times_bought_together >= 10
ORDER BY pp.times_bought_together DESC;

-- 3. Netflix: Content Engagement Analysis
-- User viewing patterns and content performance
WITH viewing_sessions AS (
    SELECT 
        user_id,
        content_id,
        watch_duration,
        content_duration,
        CASE 
            WHEN watch_duration >= content_duration * 0.8 THEN 'completed'
            WHEN watch_duration >= content_duration * 0.3 THEN 'partial'
            ELSE 'abandoned'
        END as engagement_level
    FROM user_views
),
content_metrics AS (
    SELECT 
        content_id,
        COUNT(*) as total_views,
        COUNT(CASE WHEN engagement_level = 'completed' THEN 1 END) as completed_views,
        COUNT(CASE WHEN engagement_level = 'partial' THEN 1 END) as partial_views,
        COUNT(CASE WHEN engagement_level = 'abandoned' THEN 1 END) as abandoned_views,
        AVG(watch_duration) as avg_watch_duration
    FROM viewing_sessions
    GROUP BY content_id
)
SELECT 
    c.title,
    cm.total_views,
    ROUND(cm.completed_views * 100.0 / cm.total_views, 2) as completion_rate,
    ROUND(cm.avg_watch_duration / 60, 2) as avg_watch_minutes
FROM content_metrics cm
JOIN content c ON cm.content_id = c.content_id
WHERE cm.total_views >= 100
ORDER BY completion_rate DESC;

-- 4. Uber: Driver-Rider Matching Optimization
-- Find optimal driver assignments based on proximity and ratings
WITH available_drivers AS (
    SELECT 
        driver_id,
        latitude as driver_lat,
        longitude as driver_lng,
        rating,
        ST_POINT(longitude, latitude) as driver_location
    FROM drivers
    WHERE status = 'available'
),
ride_requests AS (
    SELECT 
        request_id,
        user_id,
        pickup_latitude,
        pickup_longitude,
        ST_POINT(pickup_longitude, pickup_latitude) as pickup_location,
        requested_at
    FROM ride_requests
    WHERE status = 'pending'
),
driver_distances AS (
    SELECT 
        rr.request_id,
        ad.driver_id,
        ad.rating,
        ST_DISTANCE_SPHERE(ad.driver_location, rr.pickup_location) as distance_meters
    FROM ride_requests rr
    CROSS JOIN available_drivers ad
    WHERE ST_DISTANCE_SPHERE(ad.driver_location, rr.pickup_location) <= 5000  -- Within 5km
),
ranked_matches AS (
    SELECT 
        request_id,
        driver_id,
        rating,
        distance_meters,
        ROW_NUMBER() OVER (
            PARTITION BY request_id 
            ORDER BY distance_meters ASC, rating DESC
        ) as match_rank
    FROM driver_distances
)
SELECT 
    request_id,
    driver_id,
    rating,
    ROUND(distance_meters) as distance_meters
FROM ranked_matches
WHERE match_rank = 1;
```

### Data Analysis Patterns
```sql
-- A/B Testing Analysis
WITH experiment_results AS (
    SELECT 
        user_id,
        variant,
        conversion_event,
        experiment_start_date,
        event_date
    FROM ab_test_data
    WHERE experiment_name = 'checkout_button_color'
),
conversion_stats AS (
    SELECT 
        variant,
        COUNT(DISTINCT user_id) as total_users,
        COUNT(CASE WHEN conversion_event = 'purchase' THEN 1 END) as conversions,
        COUNT(CASE WHEN conversion_event = 'purchase' THEN 1 END) * 100.0 / 
        COUNT(DISTINCT user_id) as conversion_rate
    FROM experiment_results
    GROUP BY variant
)
SELECT 
    variant,
    total_users,
    conversions,
    ROUND(conversion_rate, 2) as conversion_rate_percent,
    -- Statistical significance would require additional calculations
    CASE 
        WHEN variant = 'control' THEN 'baseline'
        ELSE CONCAT(
            ROUND(conversion_rate - (
                SELECT conversion_rate FROM conversion_stats WHERE variant = 'control'
            ), 2), 
            '% ', 
            CASE 
                WHEN conversion_rate > (SELECT conversion_rate FROM conversion_stats WHERE variant = 'control')
                THEN 'improvement'
                ELSE 'decline'
            END
        )
    END as vs_control
FROM conversion_stats
ORDER BY conversion_rate DESC;

-- Funnel Analysis
WITH funnel_events AS (
    SELECT 
        user_id,
        event_name,
        event_timestamp,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_timestamp) as event_sequence
    FROM user_events
    WHERE event_name IN ('page_view', 'add_to_cart', 'checkout_start', 'purchase')
    AND event_timestamp >= DATE_SUB(NOW(), INTERVAL 30 DAY)
),
funnel_steps AS (
    SELECT 
        'page_view' as step_name, 1 as step_order
    UNION ALL SELECT 'add_to_cart', 2
    UNION ALL SELECT 'checkout_start', 3
    UNION ALL SELECT 'purchase', 4
),
user_progression AS (
    SELECT 
        fs.step_name,
        fs.step_order,
        COUNT(DISTINCT fe.user_id) as users_reached
    FROM funnel_steps fs
    LEFT JOIN funnel_events fe ON fs.step_name = fe.event_name
    GROUP BY fs.step_name, fs.step_order
),
funnel_analysis AS (
    SELECT 
        step_name,
        users_reached,
        LAG(users_reached) OVER (ORDER BY step_order) as prev_step_users,
        CASE 
            WHEN LAG(users_reached) OVER (ORDER BY step_order) IS NOT NULL
            THEN ROUND(users_reached * 100.0 / LAG(users_reached) OVER (ORDER BY step_order), 2)
            ELSE 100.0
        END as conversion_rate,
        CASE 
            WHEN LAG(users_reached) OVER (ORDER BY step_order) IS NOT NULL
            THEN LAG(users_reached) OVER (ORDER BY step_order) - users_reached
            ELSE 0
        END as dropoff_count
    FROM user_progression
)
SELECT 
    step_name,
    users_reached,
    conversion_rate as step_conversion_rate,
    dropoff_count
FROM funnel_analysis
ORDER BY step_name;
```

---

## SQL Best Practices Summary

### DO's ✅
1. **Use meaningful table and column names**
2. **Always use explicit JOIN syntax** (avoid implicit joins in WHERE)
3. **Index frequently queried columns**
4. **Use parameterized queries** to prevent SQL injection
5. **Normalize your database design** (but denormalize for read-heavy workloads when needed)
6. **Use transactions for data consistency**
7. **Write readable, well-formatted SQL**
8. **Use EXPLAIN to analyze query performance**
9. **Validate data integrity with constraints**
10. **Use appropriate data types**

### DON'Ts ❌
1. **Don't use SELECT \*** in production code
2. **Don't ignore SQL injection vulnerabilities**
3. **Don't create indexes on every column**
4. **Don't use cursors when set-based operations work**
5. **Don't ignore query execution plans**
6. **Don't store calculated values unless necessary**
7. **Don't use reserved words as identifiers**
8. **Don't ignore database-specific optimizations**
9. **Don't forget to handle NULL values**
10. **Don't over-normalize (performance trade-offs)**

---

## Quick Reference Card

### Essential SQL Commands
```sql
-- Data Definition Language (DDL)
CREATE TABLE, ALTER TABLE, DROP TABLE
CREATE INDEX, DROP INDEX
CREATE VIEW, DROP VIEW

-- Data Manipulation Language (DML)  
SELECT, INSERT, UPDATE, DELETE

-- Data Control Language (DCL)
GRANT, REVOKE

-- Transaction Control
BEGIN, COMMIT, ROLLBACK, SAVEPOINT

-- Key Clauses (in order)
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
```

### Function Categories
```sql
-- Aggregate Functions
COUNT(), SUM(), AVG(), MIN(), MAX(), GROUP_CONCAT()

-- String Functions  
CONCAT(), SUBSTRING(), LENGTH(), UPPER(), LOWER(), TRIM()

-- Date Functions
NOW(), DATE(), YEAR(), MONTH(), DAY(), DATE_ADD(), DATEDIFF()

-- Mathematical Functions
ROUND(), CEIL(), FLOOR(), ABS(), MOD(), POWER()

-- Window Functions
ROW_NUMBER(), RANK(), DENSE_RANK(), LAG(), LEAD(), SUM() OVER()

-- Conditional Functions
CASE WHEN, IF(), COALESCE(), NULLIF()
```

---

*This cheatsheet covers the essential SQL concepts you'll need for technical interviews at product-based companies. Practice these patterns with real datasets and always consider performance implications in your solutions. Remember: understanding the 'why' behind each technique is more valuable than memorizing syntax!*