# Elasticsearch

## Brief Definition
Elasticsearch is a distributed search and analytics engine built on Apache Lucene. It's designed for real-time search, full-text search, and analytics, making it ideal for log management, search applications, and data analysis.

## Historical Context
Created by Shay Banon in 2010, Elasticsearch was developed as a distributed version of Lucene. It became part of the Elastic Stack (ELK) and has grown into one of the most popular search engines for enterprise applications.

## Core Concepts
- Distributed by nature
- Document-oriented
- Full-text search
- Real-time search
- Inverted index
- Sharding and replication
- RESTful API
- Query DSL
- Aggregations
- Mapping and analysis

## Implementation
```json
// Index creation with mapping
PUT /products
{
    "mappings": {
        "properties": {
            "name": { "type": "text" },
            "price": { "type": "float" },
            "description": { "type": "text" },
            "tags": { "type": "keyword" },
            "created_at": { "type": "date" }
        }
    },
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}

// Document indexing
POST /products/_doc
{
    "name": "Smartphone X",
    "price": 999.99,
    "description": "Latest smartphone with advanced features",
    "tags": ["electronics", "mobile", "gadgets"],
    "created_at": "2024-03-20T10:00:00Z"
}
```

## Examples
```json
// Full-text search
GET /products/_search
{
    "query": {
        "multi_match": {
            "query": "smartphone features",
            "fields": ["name^2", "description"]
        }
    },
    "aggs": {
        "price_ranges": {
            "range": {
                "field": "price",
                "ranges": [
                    { "to": 500 },
                    { "from": 500, "to": 1000 },
                    { "from": 1000 }
                ]
            }
        }
    }
}

// Complex aggregation
GET /products/_search
{
    "size": 0,
    "aggs": {
        "tags": {
            "terms": { "field": "tags" },
            "aggs": {
                "avg_price": { "avg": { "field": "price" } }
            }
        }
    }
}
```

## Advantages and Limitations
Advantages:
- Distributed by nature
- Real-time search
- Full-text search
- Rich query DSL
- Powerful aggregations
- Scalable
- Active community

Limitations:
- Resource intensive
- Complex configuration
- No ACID compliance
- Limited transaction support
- Steeper learning curve
- Memory requirements
- Complex backup/restore

## Related Concepts
- Full-Text Search
- Log Management
- Data Analytics
- Information Retrieval
- Distributed Systems
- Search Engines
- Data Indexing 