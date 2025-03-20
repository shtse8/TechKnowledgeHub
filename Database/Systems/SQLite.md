# SQLite

## Brief Definition
SQLite is a self-contained, serverless, zero-configuration, transactional SQL database engine. It's designed to be embedded directly into applications, making it ideal for local storage, mobile apps, and small to medium-sized applications.

## Historical Context
Created by D. Richard Hipp in 2000, SQLite was designed to be a simple, reliable, and efficient database that could be embedded directly into applications. It has become one of the most widely deployed database engines, used in countless applications and devices.

## Core Concepts
- Serverless architecture
- Single file database
- ACID compliance
- SQL support
- Transaction support
- Indexing
- Triggers
- Views
- Foreign keys
- Virtual tables

## Implementation
```sql
-- Create database (automatically created on first connection)
-- Create table with constraints
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    profile JSON
);

-- Create index
CREATE INDEX idx_users_username ON users(username);

-- Create trigger
CREATE TRIGGER update_timestamp 
AFTER UPDATE ON users
BEGIN
    UPDATE users 
    SET updated_at = CURRENT_TIMESTAMP 
    WHERE id = NEW.id;
END;
```

## Examples
```sql
-- Insert data
INSERT INTO users (username, email, profile)
VALUES (
    'johndoe',
    'john@example.com',
    json('{"location": "New York", "interests": ["technology", "music"]}')
);

-- Complex query with joins
SELECT 
    u.username,
    COUNT(o.id) as total_orders,
    SUM(o.total_amount) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= date('now', '-30 days')
GROUP BY u.id, u.username
HAVING total_orders > 0
ORDER BY total_spent DESC;

-- JSON operations
SELECT 
    username,
    json_extract(profile, '$.location') as location,
    json_extract(profile, '$.interests') as interests
FROM users
WHERE json_extract(profile, '$.location') = 'New York';
```

## Advantages and Limitations
Advantages:
- Zero configuration
- Serverless
- Self-contained
- ACID compliant
- Cross-platform
- Lightweight
- Easy to use
- Wide adoption

Limitations:
- Limited concurrency
- No user management
- Limited scalability
- No network access
- Limited backup options
- No built-in security
- Limited features
- File size limits

## Related Concepts
- Embedded Databases
- SQL
- Database Design
- Data Modeling
- ACID Properties
- Database Indexing
- Database Triggers
- Database Views 