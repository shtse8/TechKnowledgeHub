# Domain-Driven Design (DDD)

A software development approach that focuses on modeling software to match a domain according to input from domain experts, emphasizing the core domain and domain logic over technical implementation details.

## Historical Context

**Origin**: Introduced by Eric Evans in his 2003 book "Domain-Driven Design: Tackling Complexity in the Heart of Software."

**Evolution**: Initially adopted in enterprise applications with complex business rules. Has significantly influenced modern architectural patterns like microservices, CQRS, and event sourcing. Continues to evolve with strategic design principles for large-scale systems.

## Core Concepts

### Strategic Design

1. **Bounded Context**:
   - Explicit boundary within which a domain model exists
   - Each context has its own ubiquitous language
   - Models don't need to be consistent across contexts
   - Different teams can work on different bounded contexts

2. **Ubiquitous Language**:
   - Common language shared by developers and domain experts
   - Used consistently in code, documentation, and communication
   - Reduces translation between technical and business terminology
   - Evolves as understanding of the domain improves

3. **Context Mapping**:
   - Explicit relationships between bounded contexts
   - Patterns include: Partnership, Shared Kernel, Customer-Supplier, Conformist, Anti-corruption Layer
   - Defines integration strategies between different contexts

### Tactical Design

1. **Entities**:
   - Objects defined by their identity rather than attributes
   - Have continuity through time and state changes
   - Often represent business objects with unique identifiers

2. **Value Objects**:
   - Immutable objects defined by their attributes
   - No identity concept - two value objects with same attributes are equal
   - Used to represent concepts like money, dates, addresses

3. **Aggregates**:
   - Cluster of domain objects treated as a single unit
   - Has a single entity as its root (aggregate root)
   - Enforces invariants and consistency rules
   - Transactional boundary for changes

4. **Domain Events**:
   - Represent significant occurrences within the domain
   - Capture state changes and business-significant moments
   - Allow for loose coupling between aggregates and bounded contexts

5. **Repositories**:
   - Abstraction for persistence of aggregates
   - Provides collection-like interface to access domain objects
   - Hides data access infrastructure details

6. **Domain Services**:
   - Contains domain logic that doesn't naturally fit in entities or value objects
   - Stateless operations that involve multiple domain objects
   - Named using domain terminology

## Implementation

### DDD Implementation Patterns

1. **Layered Architecture**:
   - User Interface / Presentation Layer
   - Application Layer (orchestration, use cases)
   - Domain Layer (entities, value objects, domain services)
   - Infrastructure Layer (persistence, messaging, external systems)

2. **Hexagonal Architecture / Ports and Adapters**:
   - Domain-centric design with adapters for external systems
   - Allows domain model to be independent of external concerns

3. **Event Sourcing with DDD**:
   - Store domain events as the source of truth
   - Rebuild aggregate state by replaying events
   - Natural fit with domain events concept

### Analyzing Domain Complexity

- **Core Domain**: Critical differentiating business areas - invest most effort here
- **Supporting Domains**: Important but not differentiating - consider off-the-shelf solutions
- **Generic Domains**: Not specific to the business - use existing solutions

## Example

### E-commerce Order Management

```java
// Value Object
public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    // More operations, equals, hashCode, etc.
}

// Entity
public class Order {
    private OrderId id;
    private CustomerId customerId;
    private Set<OrderItem> items;
    private OrderStatus status;
    private DeliveryAddress deliveryAddress;
    
    // Constructor for new orders
    public Order(OrderId id, CustomerId customerId, DeliveryAddress address) {
        this.id = id;
        this.customerId = customerId;
        this.deliveryAddress = address;
        this.items = new HashSet<>();
        this.status = OrderStatus.DRAFT;
    }
    
    // Domain logic
    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new OrderAlreadyConfirmedException();
        }
        
        var existingItem = findItemByProductId(product.getId());
        if (existingItem.isPresent()) {
            existingItem.get().increaseQuantity(quantity);
        } else {
            items.add(new OrderItem(product.getId(), product.getName(), 
                product.getPrice(), quantity));
        }
    }
    
    public void confirm() {
        if (items.isEmpty()) {
            throw new EmptyOrderException();
        }
        
        if (status != OrderStatus.DRAFT) {
            throw new OrderAlreadyConfirmedException();
        }
        
        status = OrderStatus.CONFIRMED;
        DomainEvents.publish(new OrderConfirmedEvent(this.id, this.customerId));
    }
    
    public Money calculateTotal() {
        return items.stream()
            .map(OrderItem::getTotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    // Other methods and business logic
}

// Repository Interface
public interface OrderRepository {
    void save(Order order);
    Optional<Order> findById(OrderId id);
    List<Order> findByCustomerId(CustomerId customerId);
}

// Domain Service
public class OrderPriceCalculator {
    private final TaxRateService taxRateService;
    
    public OrderPriceCalculator(TaxRateService taxRateService) {
        this.taxRateService = taxRateService;
    }
    
    public OrderPricing calculatePricing(Order order) {
        Money subtotal = order.calculateTotal();
        
        // Calculate tax based on delivery address and items
        TaxRate taxRate = taxRateService.getTaxRateFor(
            order.getDeliveryAddress().getCountry(),
            order.getItems().stream()
                .map(OrderItem::getProductId)
                .collect(Collectors.toSet())
        );
        
        Money tax = subtotal.multiply(taxRate.getValue());
        Money total = subtotal.add(tax);
        
        return new OrderPricing(subtotal, tax, total);
    }
}

// Application Service
public class OrderApplicationService {
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final OrderPriceCalculator priceCalculator;
    
    // Constructor with dependencies
    
    @Transactional
    public OrderId createOrder(CustomerId customerId, DeliveryAddress address) {
        OrderId orderId = orderIdGenerator.nextId();
        Order order = new Order(orderId, customerId, address);
        orderRepository.save(order);
        return orderId;
    }
    
    @Transactional
    public void addProductToOrder(OrderId orderId, ProductId productId, int quantity) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
            
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
            
        order.addItem(product, quantity);
        orderRepository.save(order);
    }
    
    @Transactional
    public OrderPricing confirmOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
            
        order.confirm();
        orderRepository.save(order);
        
        return priceCalculator.calculatePricing(order);
    }
}
```

## Advantages and Limitations

### Advantages
- **Business Alignment**: Software closely models the business domain
- **Knowledge Sharing**: Ubiquitous language bridges technical-business gap
- **Complexity Management**: Bounded contexts divide complex domains
- **Flexibility**: Allows for different models in different contexts
- **Maintainability**: Clear boundaries and responsibilities

### Limitations
- **Learning Curve**: Substantial investment to learn principles and patterns
- **Time Investment**: Requires significant upfront analysis and modeling
- **Overhead**: Can be excessive for simple CRUD applications
- **Evolving Understanding**: Model must evolve as domain understanding changes
- **Team Skills**: Requires experienced developers and domain experts

## Related Concepts
- Command Query Responsibility Segregation (CQRS)
- Event Sourcing
- Microservices Architecture
- Hexagonal Architecture
- Ubiquitous Language 