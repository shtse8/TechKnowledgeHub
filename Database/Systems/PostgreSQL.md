# PostgreSQL

## Brief Definition
PostgreSQL is a powerful, open-source object-relational database system that extends the SQL language with features designed to store and scale complex data workloads. It's known for its reliability, data integrity, and extensibility.

## Historical Context
PostgreSQL was developed at the University of California, Berkeley, starting in 1986. It evolved from the Ingres project and was initially called Postgres. The first version was released in 1996, and it has since become one of the most advanced open-source databases.

## Core Concepts
- ACID compliance
- Complex queries
- Foreign keys
- Views
- Stored procedures
- Triggers
- Extensions
- Full-text search
- JSON support
- Materialized views

## Implementation
```sql
-- Table creation with constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    profile JSONB
);

-- Index creation
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);

-- Trigger function
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Trigger creation
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

## Examples
```sql
-- Complex query with window functions
SELECT 
    user_id,
    username,
    COUNT(*) OVER (PARTITION BY user_id) as total_posts,
    ROW_NUMBER() OVER (ORDER BY created_at DESC) as post_rank
FROM posts
WHERE created_at >= NOW() - INTERVAL '7 days';

-- JSON operations
SELECT 
    username,
    profile->>'location' as location,
    profile->>'interests' as interests
FROM users
WHERE profile->>'location' = 'New York';

-- Full-text search
SELECT 
    title,
    content,
    ts_rank_cd(to_tsvector('english', content), query) as rank
FROM articles, to_tsquery('english', 'postgresql & database') query
WHERE to_tsvector('english', content) @@ query
ORDER BY rank DESC;
```

## Advantages and Limitations
Advantages:
- ACID compliance
- Complex queries
- Extensible
- JSON support
- Full-text search
- Spatial data
- Active community
- Enterprise features

Limitations:
- Complex setup
- Resource intensive
- Learning curve
- Limited horizontal scaling
- Backup complexity
- Version upgrades
- Configuration complexity
- Memory requirements

## Related Concepts
- Relational Databases
- SQL
- Database Design
- Data Modeling
- ACID Properties
- Database Indexing
- Database Triggers
- Database Views 