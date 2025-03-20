# MongoDB

## Brief Definition
MongoDB is a document-oriented NoSQL database that stores data in flexible, JSON-like documents. It's designed for scalability, flexibility, and high performance, making it ideal for modern applications with rapidly changing data structures.

## Historical Context
Created by 10gen (now MongoDB Inc.) in 2007, MongoDB was developed to address the limitations of traditional relational databases in handling modern web applications' data requirements. It has become one of the most popular NoSQL databases.

## Core Concepts
- Document-oriented
- BSON format
- Collections and documents
- Indexing
- Aggregation pipeline
- Replication
- Sharding
- Change streams
- GridFS
- MongoDB Atlas

## Implementation
```javascript
// MongoDB connection
const { MongoClient } = require('mongodb');
const uri = "mongodb://localhost:27017";
const client = new MongoClient(uri);

// Document structure
const userDocument = {
    _id: ObjectId(),
    username: "johndoe",
    email: "john@example.com",
    profile: {
        firstName: "John",
        lastName: "Doe",
        age: 30,
        interests: ["technology", "music", "travel"]
    },
    createdAt: new Date(),
    lastLogin: new Date()
};

// Insert document
async function insertUser(user) {
    try {
        await client.connect();
        const database = client.db("myDatabase");
        const users = database.collection("users");
        const result = await users.insertOne(user);
        return result;
    } finally {
        await client.close();
    }
}
```

## Examples
```javascript
// Complex queries and aggregations
// Find users with specific criteria
const query = {
    "profile.age": { $gt: 25 },
    "profile.interests": "technology"
};

// Aggregation pipeline
const pipeline = [
    { $match: { "profile.age": { $gt: 25 } } },
    { $group: {
        _id: "$profile.interests",
        count: { $sum: 1 },
        avgAge: { $avg: "$profile.age" }
    }},
    { $sort: { count: -1 } }
];

// Change streams
const pipeline = [
    { $match: { "operationType": "insert" } }
];

const changeStream = collection.watch(pipeline);
changeStream.on("change", change => {
    console.log("New document:", change);
});
```

## Advantages and Limitations
Advantages:
- Flexible schema
- Horizontal scaling
- Rich query language
- Powerful aggregation
- Document-oriented
- High performance
- Active community
- Cloud-native support

Limitations:
- No ACID compliance
- Memory usage
- Complex transactions
- Limited joins
- Data consistency
- Backup complexity
- Learning curve
- Resource requirements

## Related Concepts
- NoSQL Databases
- Document Stores
- Data Modeling
- Distributed Systems
- Database Scaling
- Data Aggregation
- Indexing
- Replication 