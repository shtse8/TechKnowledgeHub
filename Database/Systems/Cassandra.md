# Cassandra

## Brief Definition
Apache Cassandra is a distributed NoSQL database designed to handle large amounts of data across many commodity servers, providing high availability with no single point of failure. It's known for its linear scalability and fault tolerance.

## Historical Context
Created at Facebook in 2008, Cassandra was developed to power the inbox search feature. It was later open-sourced and became an Apache project in 2009. The database was named after the Greek mythological figure Cassandra, who had the gift of prophecy but was cursed to never be believed.

## Core Concepts
- Distributed architecture
- Peer-to-peer model
- Column-family store
- Tunable consistency
- Gossip protocol
- Virtual nodes
- Snitches
- Partitioners
- Compaction
- CQL (Cassandra Query Language)

## Implementation
```sql
-- Keyspace creation
CREATE KEYSPACE ecommerce
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- Table creation
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    username text,
    email text,
    created_at timestamp,
    profile map<text, text>
);

-- Secondary index
CREATE INDEX idx_users_email ON users(email);

-- Materialized view
CREATE MATERIALIZED VIEW user_by_email AS
SELECT *
FROM users
WHERE email IS NOT NULL
PRIMARY KEY (email, user_id);
```

## Examples
```sql
-- Data insertion
INSERT INTO users (user_id, username, email, created_at, profile)
VALUES (
    uuid(),
    'johndoe',
    'john@example.com',
    toTimestamp(now()),
    {'location': 'New York', 'interests': 'technology'}
);

-- Complex query with clustering
SELECT *
FROM orders
WHERE user_id = 123e4567-e89b-12d3-a456-426614174000
AND order_date >= '2024-01-01'
AND order_date <= '2024-03-20';

-- Batch operations
BEGIN BATCH
    INSERT INTO users (user_id, username, email) VALUES (uuid(), 'user1', 'user1@example.com');
    INSERT INTO users (user_id, username, email) VALUES (uuid(), 'user2', 'user2@example.com');
    UPDATE users SET profile = {'status': 'active'} WHERE user_id = uuid();
APPLY BATCH;
```

## Advantages and Limitations
Advantages:
- Linear scalability
- High availability
- No single point of failure
- Write optimization
- Geographic distribution
- Flexible schema
- Active community
- Enterprise features

Limitations:
- Complex data modeling
- Limited query capabilities
- No ACID compliance
- Resource intensive
- Complex maintenance
- Limited joins
- Learning curve
- Backup complexity

## Related Concepts
- Distributed Systems
- NoSQL Databases
- Data Modeling
- Database Scaling
- Data Replication
- Consistency Models
- Database Clustering
- Query Languages 