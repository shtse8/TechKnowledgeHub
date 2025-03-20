# CouchDB

## Brief Definition
Apache CouchDB is a document-oriented NoSQL database that uses JSON for documents, JavaScript for MapReduce queries, and regular HTTP for its API. It's designed for reliability, scalability, and offline-first applications.

## Historical Context
Created by Damien Katz in 2005, CouchDB was developed to provide a database that could work offline and sync when back online. It became an Apache project in 2008 and has evolved into a robust document database with strong replication capabilities.

## Core Concepts
- Document-oriented
- JSON storage
- MapReduce views
- HTTP API
- Multi-version concurrency control
- Offline-first
- Replication
- Conflict resolution
- Authentication
- CORS support

## Implementation
```javascript
// Create document
const doc = {
    _id: 'user_1',
    type: 'user',
    username: 'johndoe',
    email: 'john@example.com',
    created_at: new Date().toISOString(),
    profile: {
        location: 'New York',
        interests: ['technology', 'music']
    }
};

// Create view
const view = {
    views: {
        by_username: {
            map: function(doc) {
                if (doc.type === 'user') {
                    emit(doc.username, doc);
                }
            }
        },
        by_location: {
            map: function(doc) {
                if (doc.type === 'user' && doc.profile && doc.profile.location) {
                    emit(doc.profile.location, doc);
                }
            }
        }
    }
};

// Create design document
const designDoc = {
    _id: '_design/users',
    ...view
};
```

## Examples
```javascript
// Query with view
const query = {
    key: 'johndoe',
    include_docs: true
};

// Complex view with reduce
const complexView = {
    views: {
        user_stats: {
            map: function(doc) {
                if (doc.type === 'user') {
                    emit(doc.profile.location, 1);
                }
            },
            reduce: function(keys, values, rereduce) {
                return sum(values);
            }
        }
    }
};

// Replication
const replication = {
    source: 'http://localhost:5984/users',
    target: 'http://remote-server:5984/users_backup',
    continuous: true,
    filter: function(doc) {
        return doc.type === 'user';
    }
};
```

## Advantages and Limitations
Advantages:
- Offline-first
- HTTP API
- Built-in replication
- Conflict resolution
- JSON native
- RESTful
- Easy to use
- Cross-platform

Limitations:
- Limited querying
- No joins
- Resource intensive
- Complex views
- Limited indexing
- No transactions
- Learning curve
- Version conflicts

## Related Concepts
- Document Databases
- NoSQL
- Data Replication
- Offline Storage
- HTTP APIs
- JSON
- MapReduce
- Conflict Resolution 