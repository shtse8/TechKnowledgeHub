# DynamoDB

## Brief Definition
Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. It's designed for applications requiring consistent, single-digit millisecond latency at any scale.

## Historical Context
Released by Amazon Web Services in 2012, DynamoDB was built on the principles of Amazon's Dynamo paper and is designed to address the scalability and performance challenges of traditional databases. It's a core component of AWS's database services portfolio.

## Core Concepts
- Key-value store
- DynamoDB Streams
- Global tables
- Point-in-time recovery
- On-demand capacity
- Provisioned capacity
- Secondary indexes
- Atomic counters
- Conditional updates
- TTL (Time To Live)

## Implementation
```python
# Python example using boto3
import boto3

# Create DynamoDB client
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# Create item
response = table.put_item(
    Item={
        'user_id': 'u1',
        'username': 'johndoe',
        'email': 'john@example.com',
        'created_at': '2024-03-20T10:00:00Z',
        'profile': {
            'location': 'New York',
            'interests': ['technology', 'music']
        }
    }
)

# Create secondary index
response = dynamodb.create_table(
    TableName='Users',
    KeySchema=[
        {'AttributeName': 'user_id', 'KeyType': 'HASH'},
        {'AttributeName': 'username', 'KeyType': 'RANGE'}
    ],
    AttributeDefinitions=[
        {'AttributeName': 'user_id', 'AttributeType': 'S'},
        {'AttributeName': 'username', 'AttributeType': 'S'}
    ],
    ProvisionedThroughput={
        'ReadCapacityUnits': 5,
        'WriteCapacityUnits': 5
    }
)
```

## Examples
```python
# Query with secondary index
response = table.query(
    IndexName='UsernameIndex',
    KeyConditionExpression='username = :username',
    ExpressionAttributeValues={
        ':username': 'johndoe'
    }
)

# Update with condition
response = table.update_item(
    Key={'user_id': 'u1'},
    UpdateExpression='SET profile.location = :location',
    ConditionExpression='attribute_exists(user_id)',
    ExpressionAttributeValues={
        ':location': 'San Francisco'
    }
)

# Batch operations
with table.batch_writer() as batch:
    batch.put_item(Item={'user_id': 'u1', 'username': 'user1'})
    batch.put_item(Item={'user_id': 'u2', 'username': 'user2'})
    batch.delete_item(Key={'user_id': 'u3'})

# Scan with filter
response = table.scan(
    FilterExpression='contains(profile.interests, :interest)',
    ExpressionAttributeValues={
        ':interest': 'technology'
    }
)
```

## Advantages and Limitations
Advantages:
- Fully managed
- Serverless
- Automatic scaling
- Consistent performance
- Global tables
- Point-in-time recovery
- Built-in security
- AWS integration

Limitations:
- Cost considerations
- Limited query flexibility
- No SQL support
- Complex data modeling
- Limited transactions
- Cold starts
- Learning curve
- Vendor lock-in

## Related Concepts
- NoSQL Databases
- Cloud Databases
- Database Scaling
- Data Modeling
- Key-Value Stores
- Database Security
- Database Backup
- Cloud Services 