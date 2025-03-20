# Monolithic Architecture

An architectural pattern where all components of an application are combined into a single program deployed as a unified unit, with all functionality tightly integrated within the same codebase and process space.

## Historical Context

**Origin**: The default approach to software development since the early days of computing, predating modern architectural concepts.

**Evolution**: Dominated enterprise application development through the 1990s and 2000s. While newer architectural styles like microservices have emerged, monolithic architecture remains common for many applications due to its simplicity and straightforward development model.

## Core Concepts

1. **Single Deployment Unit**:
   - Entire application packaged and deployed as one unit
   - All components run within the same process
   - Shared resources and memory space
   - Typically a single codebase

2. **Tight Coupling**:
   - Components interact through function/method calls
   - Shared libraries and dependencies
   - Direct access to other components' data structures
   - Strong compile-time dependencies

3. **Shared Database**:
   - Single database for the entire application
   - Common data model across all components
   - Centralized transaction management
   - Strong schema consistency guarantees

4. **Vertical Scaling**:
   - Scaled by deploying more powerful hardware
   - Adding CPU, memory, or disk to handle increased load
   - All components scale together as a unit
   - Limited horizontal scalability options

## Implementation

### Common Structural Patterns

1. **Layered Architecture**:
   - Presentation layer (UI/API)
   - Business logic layer
   - Data access layer
   - Clear separation between responsibilities
   - Example: Traditional Spring/Hibernate Java applications

2. **Modular Monolith**:
   - Internal modules with defined interfaces
   - Stronger boundaries between components
   - Minimized dependencies between modules
   - Often implemented with package/namespace controls

3. **MVC/MVVM Patterns**:
   - Separation of concerns within the monolith
   - Organized code structure based on responsibility
   - Clear patterns for presentation logic
   - Example: Ruby on Rails, Django, Laravel applications

### Technology Stacks

Monolithic architectures are implemented in various technology stacks:

- **Java**: Spring Boot, Jakarta EE
- **C#/.NET**: ASP.NET Core, .NET Framework
- **PHP**: Laravel, Symfony
- **Ruby**: Ruby on Rails
- **Python**: Django, Flask
- **JavaScript**: Express.js with server-rendered views

## Example

### E-commerce Application (Java Spring Boot)

```java
// Application Entry Point
@SpringBootApplication
public class EcommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(EcommerceApplication.class, args);
    }
}

// Product Domain Model
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String description;
    private BigDecimal price;
    private int stockQuantity;
    
    // Getters, setters, constructors
}

// Order Domain Model
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
    
    private OrderStatus status;
    private LocalDateTime orderDate;
    private BigDecimal totalAmount;
    
    // Methods for order management
    public void addItem(Product product, int quantity) {
        OrderItem item = new OrderItem(this, product, quantity, product.getPrice());
        items.add(item);
        recalculateTotal();
    }
    
    public void recalculateTotal() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    public void placeOrder() {
        this.orderDate = LocalDateTime.now();
        this.status = OrderStatus.PLACED;
    }
    
    // Getters, setters, constructors
}

// Data Access Layer
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContaining(String name);
    
    @Query("SELECT p FROM Product p WHERE p.stockQuantity > 0")
    List<Product> findAvailableProducts();
}

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerId(Long customerId);
    List<Order> findByStatusAndOrderDateBetween(OrderStatus status, 
                                              LocalDateTime start, 
                                              LocalDateTime end);
}

// Service Layer
@Service
@Transactional
public class OrderService {
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    private final PaymentService paymentService;
    
    // Constructor injection
    public OrderService(OrderRepository orderRepository,
                       ProductRepository productRepository,
                       InventoryService inventoryService,
                       NotificationService notificationService,
                       PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.productRepository = productRepository;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
        this.paymentService = paymentService;
    }
    
    public Order createOrder(OrderRequest orderRequest) {
        // Create a new order
        Customer customer = getCustomer(orderRequest.getCustomerId());
        Order order = new Order(customer);
        
        // Add items to order
        for (OrderItemRequest itemRequest : orderRequest.getItems()) {
            Product product = productRepository.findById(itemRequest.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(itemRequest.getProductId()));
                
            // Check stock availability
            if (product.getStockQuantity() < itemRequest.getQuantity()) {
                throw new InsufficientStockException(product.getId());
            }
            
            order.addItem(product, itemRequest.getQuantity());
        }
        
        return orderRepository.save(order);
    }
    
    @Transactional
    public Order placeOrder(Long orderId, PaymentDetails paymentDetails) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
            
        // Process payment
        PaymentResult paymentResult = paymentService.processPayment(
            order.getCustomer(), order.getTotalAmount(), paymentDetails);
            
        if (!paymentResult.isSuccessful()) {
            throw new PaymentFailedException(paymentResult.getErrorMessage());
        }
        
        // Update inventory
        for (OrderItem item : order.getItems()) {
            inventoryService.reduceStock(item.getProduct().getId(), item.getQuantity());
        }
        
        // Update order status
        order.placeOrder();
        orderRepository.save(order);
        
        // Send confirmation
        notificationService.sendOrderConfirmation(order);
        
        return order;
    }
    
    // Other business methods
}

// Controller Layer
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<OrderDTO> createOrder(@RequestBody OrderRequest orderRequest) {
        Order order = orderService.createOrder(orderRequest);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(OrderMapper.toDTO(order));
    }
    
    @PostMapping("/{orderId}/payment")
    public ResponseEntity<OrderDTO> processPayment(
            @PathVariable Long orderId,
            @RequestBody PaymentRequest paymentRequest) {
        Order order = orderService.placeOrder(orderId, paymentRequest.getPaymentDetails());
        return ResponseEntity.ok(OrderMapper.toDTO(order));
    }
    
    @GetMapping("/{orderId}")
    public ResponseEntity<OrderDTO> getOrder(@PathVariable Long orderId) {
        Order order = orderService.getOrder(orderId);
        return ResponseEntity.ok(OrderMapper.toDTO(order));
    }
}

// Configuration (application.properties)
/*
spring.datasource.url=jdbc:postgresql://localhost:5432/ecommerce
spring.datasource.username=app_user
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Email service configuration
notification.email.from=orders@example.com
notification.email.enabled=true

# Payment gateway configuration
payment.gateway.url=https://payment.example.com/api
payment.gateway.apiKey=${PAYMENT_API_KEY}

# Server configuration
server.port=8080
*/
```

### Project Structure

```
ecommerce-monolith/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── ecommerce/
│   │   │               ├── EcommerceApplication.java
│   │   │               ├── config/
│   │   │               │   ├── SecurityConfig.java
│   │   │               │   └── WebConfig.java
│   │   │               ├── controller/
│   │   │               │   ├── ProductController.java
│   │   │               │   ├── OrderController.java
│   │   │               │   ├── CustomerController.java
│   │   │               │   └── AdminController.java
│   │   │               ├── dto/
│   │   │               │   ├── ProductDTO.java
│   │   │               │   ├── OrderDTO.java
│   │   │               │   └── CustomerDTO.java
│   │   │               ├── model/
│   │   │               │   ├── Product.java
│   │   │               │   ├── Order.java
│   │   │               │   ├── OrderItem.java
│   │   │               │   ├── Customer.java
│   │   │               │   └── Address.java
│   │   │               ├── repository/
│   │   │               │   ├── ProductRepository.java
│   │   │               │   ├── OrderRepository.java
│   │   │               │   └── CustomerRepository.java
│   │   │               ├── service/
│   │   │               │   ├── ProductService.java
│   │   │               │   ├── OrderService.java
│   │   │               │   ├── InventoryService.java
│   │   │               │   ├── CustomerService.java
│   │   │               │   ├── PaymentService.java
│   │   │               │   └── NotificationService.java
│   │   │               └── exception/
│   │   │                   ├── NotFoundException.java
│   │   │                   ├── InsufficientStockException.java
│   │   │                   └── PaymentFailedException.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── templates/
│   │       └── static/
│   └── test/
├── pom.xml
└── README.md
```

## Advantages and Limitations

### Advantages
- **Simplicity**: Easier to develop, test, and deploy initially
- **Development Speed**: Faster development cycles in early stages
- **Strong Consistency**: Easier to maintain data integrity with shared database
- **Simplified Debugging**: All code runs in the same process
- **Less Operational Complexity**: Single application to deploy and monitor

### Limitations
- **Scalability Challenges**: Difficult to scale specific components independently
- **Technology Lock-in**: Entire application must use the same stack
- **Development Bottlenecks**: Large teams face coordination challenges
- **Deployment Risk**: Any change requires redeploying the entire application
- **Long-term Maintainability**: Can become complex and unwieldy over time

## Related Concepts
- Layered Architecture
- Modular Monolith
- Microservices Architecture
- Service-Oriented Architecture
- Decomposition Patterns 