# Redis

## Brief Definition
Redis (Remote Dictionary Server) is an open-source, in-memory data structure store that can be used as a database, cache, message broker, and queue. It supports various data structures such as strings, hashes, lists, sets, and sorted sets.

## Historical Context
Created by Salvatore Sanfilippo in 2009, Redis was initially developed to solve the scalability issues of his real-time analytics system. It has since become one of the most popular in-memory data stores, known for its speed and versatility.

## Core Concepts
- In-memory storage
- Data structures
- Persistence options
- Pub/Sub messaging
- Atomic operations
- Lua scripting
- Master-slave replication
- Sentinel for high availability
- Cluster mode
- Key expiration

## Implementation
```python
# Python example using redis-py
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# String operations
r.set('user:1:name', 'John Doe')
r.set('user:1:email', 'john@example.com')

# Hash operations
r.hset('user:1', mapping={
    'name': 'John Doe',
    'email': 'john@example.com',
    'age': '30'
})

# List operations
r.lpush('recent_users', 'user:1')
r.lpush('recent_users', 'user:2')

# Set operations
r.sadd('online_users', 'user:1')
r.sadd('online_users', 'user:2')

# Sorted set operations
r.zadd('leaderboard', {'player1': 100, 'player2': 200})
```

## Examples
```python
# Complex data structure example
# User session management
def create_user_session(user_id, session_data):
    # Store session data in hash
    r.hmset(f'session:{user_id}', session_data)
    # Set expiration (30 minutes)
    r.expire(f'session:{user_id}', 1800)
    # Add to active sessions set
    r.sadd('active_sessions', user_id)

# Rate limiting example
def check_rate_limit(user_id, limit=100, window=3600):
    key = f'rate_limit:{user_id}'
    current = r.incr(key)
    if current == 1:
        r.expire(key, window)
    return current <= limit

# Leaderboard implementation
def update_leaderboard(player_id, score):
    r.zadd('game_leaderboard', {player_id: score})
    # Get top 10 players
    return r.zrevrange('game_leaderboard', 0, 9, withscores=True)
```

## Advantages and Limitations
Advantages:
- Extremely fast performance
- Rich data structures
- Atomic operations
- Built-in pub/sub
- Persistence options
- Master-slave replication
- Active community
- Wide language support

Limitations:
- Memory bound
- No built-in security
- Limited query capabilities
- No built-in indexing
- Single-threaded
- Memory management complexity
- Backup/restore considerations
- No built-in authentication

## Related Concepts
- In-Memory Databases
- Caching
- Message Queues
- Data Structures
- Distributed Systems
- Key-Value Stores
- Pub/Sub Systems
- Session Management 