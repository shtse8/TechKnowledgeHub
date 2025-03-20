# TiDB

## Brief Definition
TiDB is an open-source, distributed SQL database that provides horizontal scalability, strong consistency, and high availability. It's designed to be MySQL compatible while offering the scalability of NoSQL databases.

## Historical Context
Created by PingCAP in 2015, TiDB was developed to address the scalability limitations of traditional MySQL databases. It was designed to provide a MySQL-compatible interface while offering distributed capabilities and strong consistency.

## Core Concepts
- MySQL compatibility
- Distributed architecture
- Strong consistency
- Horizontal scaling
- ACID compliance
- SQL support
- Automatic sharding
- Data replication
- Backup and restore
- Change data capture

## Implementation
```sql
-- Create database
CREATE DATABASE ecommerce;

-- Create table
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    profile JSON
);

-- Create index
CREATE INDEX idx_users_username ON users(username);

-- Create view
CREATE VIEW active_users AS
SELECT username, email
FROM users
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);
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

-- JSON operations
SELECT 
    username,
    JSON_EXTRACT(profile, '$.location') as location,
    JSON_EXTRACT(profile, '$.interests') as interests
FROM users
WHERE JSON_EXTRACT(profile, '$.location') = 'New York';

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
```

## Advantages and Limitations
Advantages:
- MySQL compatibility
- Distributed architecture
- Strong consistency
- Horizontal scaling
- ACID compliance
- Active community
- Good documentation
- Enterprise features

Limitations:
- Resource intensive
- Complex setup
- Learning curve
- Limited tooling
- Backup complexity
- Version compatibility
- Cost considerations
- Network latency

## Related Concepts
- Distributed Databases
- SQL
- ACID Properties
- Data Distribution
- Database Scaling
- Transaction Management
- Data Replication
- MySQL 