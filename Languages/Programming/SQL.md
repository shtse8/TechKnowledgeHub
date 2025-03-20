# SQL (Structured Query Language)

## Brief Definition
SQL is a domain-specific language designed for managing and manipulating relational databases. It's the standard language for relational database management systems (RDBMS) and is used for storing, retrieving, and managing data in structured formats.

## Historical Context
Developed by IBM researchers in the 1970s, SQL was originally called SEQUEL (Structured English Query Language). It became a standard in 1986 when ANSI and ISO adopted it. SQL has become the de facto standard for database operations.

## Core Concepts
- Data Definition Language (DDL)
- Data Manipulation Language (DML)
- Data Control Language (DCL)
- Transaction Control Language (TCL)
- Relational algebra
- Normalization
- Indexing
- Constraints
- Views
- Stored procedures

## Implementation
```sql
-- Table creation
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO users (id, name, email)
VALUES (1, 'John Doe', 'john@example.com');

-- Select with joins
SELECT u.name, o.order_id
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';
```

## Examples
```sql
-- Complex query with subquery
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);

-- Window functions
SELECT 
    department,
    name,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- Transaction
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

## Advantages and Limitations
Advantages:
- Standardized language
- Declarative syntax
- Powerful data manipulation
- ACID compliance
- Scalable
- Widely supported
- Rich ecosystem

Limitations:
- Not Turing complete
- Limited procedural capabilities
- Performance depends on implementation
- Different dialects between vendors
- Complex queries can be hard to optimize
- Limited support for unstructured data

## Related Concepts
- Relational Databases
- Database Design
- Data Modeling
- Query Optimization
- Database Administration
- Data Warehousing
- Business Intelligence 