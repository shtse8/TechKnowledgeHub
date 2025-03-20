# MySQL

## Brief Definition
MySQL is an open-source relational database management system that uses SQL (Structured Query Language) for managing and manipulating data. It's known for its reliability, ease of use, and wide platform support, making it one of the most popular databases for web applications.

## Historical Context
Created by Michael Widenius and David Axmark in 1995, MySQL was developed to provide a fast, reliable, and easy-to-use database system. It was acquired by Oracle Corporation in 2010 and remains one of the most widely used databases in the world.

## Core Concepts
- ACID compliance
- InnoDB storage engine
- Replication
- Partitioning
- Stored procedures
- Triggers
- Views
- Indexing
- Transactions
- User management

## Implementation
```sql
-- Database and table creation
CREATE DATABASE ecommerce;
USE ecommerce;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
);

-- Stored procedure
DELIMITER //
CREATE PROCEDURE create_user(
    IN p_username VARCHAR(50),
    IN p_email VARCHAR(255)
)
BEGIN
    INSERT INTO users (username, email)
    VALUES (p_username, p_email);
END //
DELIMITER ;

-- Trigger
DELIMITER //
CREATE TRIGGER before_user_update
BEFORE UPDATE ON users
FOR EACH ROW
BEGIN
    SET NEW.updated_at = CURRENT_TIMESTAMP;
END //
DELIMITER ;
```

## Examples
```sql
-- Complex query with joins
SELECT 
    u.username,
    COUNT(o.id) as total_orders,
    SUM(o.total_amount) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY u.id, u.username
HAVING total_orders > 0
ORDER BY total_spent DESC;

-- Window functions
SELECT 
    user_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY user_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM orders;

-- Full-text search
SELECT 
    title,
    content,
    MATCH(title, content) AGAINST ('database optimization' IN BOOLEAN MODE) as relevance
FROM articles
WHERE MATCH(title, content) AGAINST ('database optimization' IN BOOLEAN MODE)
ORDER BY relevance DESC;
```

## Advantages and Limitations
Advantages:
- Easy to use
- Wide platform support
- Active community
- Good documentation
- Cost-effective
- Scalable
- Reliable
- Popular hosting support

Limitations:
- Limited ACID compliance
- Less advanced features
- Limited JSON support
- Complex partitioning
- Limited full-text search
- Version fragmentation
- Less extensible
- Limited spatial data

## Related Concepts
- Relational Databases
- SQL
- Database Design
- Data Modeling
- ACID Properties
- Database Indexing
- Database Triggers
- Database Views 