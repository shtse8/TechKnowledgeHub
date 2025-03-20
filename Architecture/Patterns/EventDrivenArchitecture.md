# Event-Driven Architecture (EDA)

A software architecture pattern that promotes the production, detection, consumption, and reaction to events through loosely coupled components.

## Historical Context

**Origin**: Emerged in the late 1990s as distributed systems became more prevalent, although the core concept of event handling has existed since the early days of computing.

**Evolution**: Gained significant adoption in the 2000s with the rise of enterprise messaging systems. Further popularized in the 2010s with the growth of microservices, IoT, and real-time data processing needs.

## Core Concepts

1. **Events**:
   - Self-contained units representing something that happened
   - Typically immutable facts about past occurrences
   - Contains event metadata, type, and payload

2. **Event Producers**:
   - Components that detect or create events
   - Publish events without knowledge of consumers
   - No expectation of how events will be processed

3. **Event Consumers**:
   - Components that react to events
   - Subscribe to specific event types
   - Act upon received events independently

4. **Event Channels**:
   - Medium through which events flow (messaging queues, brokers, buses)
   - Decouples producers from consumers
   - Handles event routing, filtering, and delivery

## Implementation

### Event Flow Patterns

1. **Publish-Subscribe (Pub/Sub)**:
   - Publishers emit events to topics/channels
   - Subscribers receive events from topics they've subscribed to
   - Many-to-many relationship between publishers and subscribers

2. **Event Streaming**:
   - Events stored in an ordered, replayable log
   - Multiple consumers can process events at their own pace
   - Historical events can be replayed

3. **Event Sourcing**:
   - State changes captured as a sequence of events
   - Current state reconstructed by replaying events
   - Complete audit trail of all changes

### Technical Components

- **Event Broker**: Apache Kafka, RabbitMQ, Amazon SNS/SQS, Google Pub/Sub
- **Stream Processing**: Apache Flink, Kafka Streams, Spark Streaming
- **API Gateways**: Handle HTTP events and convert to internal format
- **Event Stores**: Specialized databases for storing event streams

## Example

### E-commerce Order Processing

```javascript
// Order Service (Event Producer)
function placeOrder(orderData) {
  // Process and validate order
  const order = createOrder(orderData);
  saveToDatabase(order);
  
  // Publish order placed event
  eventBus.publish('order.placed', {
    orderId: order.id,
    customerId: order.customerId,
    totalAmount: order.totalAmount,
    items: order.items,
    timestamp: new Date().toISOString()
  });
  
  return order;
}

// Inventory Service (Event Consumer)
eventBus.subscribe('order.placed', async (event) => {
  // Update inventory levels
  for (const item of event.items) {
    await reduceStock(item.productId, item.quantity);
    
    // If stock drops below threshold, publish another event
    const newStockLevel = await getStockLevel(item.productId);
    if (newStockLevel < RESTOCK_THRESHOLD) {
      eventBus.publish('inventory.low', {
        productId: item.productId,
        currentLevel: newStockLevel,
        timestamp: new Date().toISOString()
      });
    }
  }
});

// Notification Service (Event Consumer)
eventBus.subscribe('order.placed', async (event) => {
  const customer = await getCustomer(event.customerId);
  sendOrderConfirmation(customer.email, {
    orderId: event.orderId,
    amount: event.totalAmount,
    date: event.timestamp
  });
});

// Procurement Service (Event Consumer)
eventBus.subscribe('inventory.low', async (event) => {
  createPurchaseOrder(event.productId, RESTOCK_QUANTITY);
});
```

## Advantages and Limitations

### Advantages
- **Loose Coupling**: Components interact without direct dependencies
- **Scalability**: Components can scale independently
- **Resilience**: Failure in one component doesn't affect others
- **Flexibility**: Easy to add new consumers without changing producers
- **Responsiveness**: Real-time processing of events

### Limitations
- **Eventual Consistency**: Data consistency challenges across components
- **Complexity**: Debugging and tracking event flows can be difficult
- **Ordering**: Maintaining event order can be challenging at scale
- **Idempotency**: Consumers must handle duplicate events gracefully
- **Versioning**: Managing event schema changes requires careful planning

## Related Concepts
- Command Query Responsibility Segregation (CQRS)
- Event Sourcing
- Reactive Systems
- Message-Driven Architecture
- Streaming Architecture 