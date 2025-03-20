# Hexagonal Architecture (Ports and Adapters)

An architectural pattern that isolates the core application logic from external concerns, making the application independent of frameworks, UIs, databases, and external services.

## Historical Context

**Origin**: Introduced by Alistair Cockburn in 2005, initially called "Ports and Adapters" pattern.

**Evolution**: Gained popularity in the context of Domain-Driven Design and Test-Driven Development communities. Increasingly relevant with the rise of microservices and the need for adaptable, maintainable systems.

## Core Concepts

1. **Hexagonal Structure**:
   - Core application logic at the center (domain)
   - External elements arranged around the core
   - No direct dependencies from core to external elements
   - Name derives from conceptual diagram, not literal implementation

2. **Ports**:
   - Interface definitions that specify how the application interacts with the outside world
   - Primary/Driving Ports: Interfaces through which external actors use the application
   - Secondary/Driven Ports: Interfaces through which the application uses external services

3. **Adapters**:
   - Implementations that connect external systems to the ports
   - Primary/Driving Adapters: Implement use cases defined by primary ports
   - Secondary/Driven Adapters: Implement interfaces defined by secondary ports

4. **Dependency Rule**:
   - Dependencies always point inward
   - Inner layers define interfaces that outer layers implement
   - Domain knows nothing about external systems

## Implementation

### Application Structure

1. **Domain Layer**:
   - Contains business entities and core logic
   - Completely isolated from external concerns
   - No dependencies on frameworks or infrastructure

2. **Application Layer**:
   - Implements use cases of the application
   - Defines port interfaces (both primary and secondary)
   - Orchestrates the flow of data between the domain and ports

3. **Adapters Layer**:
   - Primary Adapters: UI, API controllers, CLI interfaces
   - Secondary Adapters: Database repositories, external service clients
   - Framework-specific code isolated in these adapters

### Key Implementation Patterns

- **Dependency Injection**: Used to wire adapters to ports
- **Interface Segregation**: Small, client-specific interfaces for ports
- **Adapter Pattern**: Translates between external systems and the application
- **Command/Query Separation**: Often used with primary ports

## Example

### Java Implementation

```java
// DOMAIN LAYER
// Core domain entity
public class Product {
    private ProductId id;
    private String name;
    private Money price;
    private int stockQuantity;
    
    // Domain logic
    public void decreaseStock(int quantity) {
        if (quantity > stockQuantity) {
            throw new InsufficientStockException(id, stockQuantity, quantity);
        }
        stockQuantity -= quantity;
    }
    
    public boolean isInStock() {
        return stockQuantity > 0;
    }
    
    // Getters, constructor, etc.
}

// APPLICATION LAYER
// Primary port (driving port)
public interface ProductService {
    ProductDTO getProductById(ProductId id);
    void updateStockLevel(ProductId id, int newQuantity);
    List<ProductDTO> findAvailableProducts();
}

// Secondary port (driven port)
public interface ProductRepository {
    Optional<Product> findById(ProductId id);
    void save(Product product);
    List<Product> findByStockGreaterThan(int minimumStock);
}

// Application service implementing primary port
public class ProductServiceImpl implements ProductService {
    private final ProductRepository productRepository;
    
    public ProductServiceImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    @Override
    public ProductDTO getProductById(ProductId id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        return mapToDTO(product);
    }
    
    @Override
    public void updateStockLevel(ProductId id, int newQuantity) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        
        // Domain logic
        if (newQuantity > product.getStockQuantity()) {
            product.increaseStock(newQuantity - product.getStockQuantity());
        } else {
            product.decreaseStock(product.getStockQuantity() - newQuantity);
        }
        
        productRepository.save(product);
    }
    
    @Override
    public List<ProductDTO> findAvailableProducts() {
        return productRepository.findByStockGreaterThan(0).stream()
            .map(this::mapToDTO)
            .collect(Collectors.toList());
    }
    
    private ProductDTO mapToDTO(Product product) {
        return new ProductDTO(
            product.getId().getValue(),
            product.getName(),
            product.getPrice().getAmount(),
            product.getStockQuantity()
        );
    }
}

// ADAPTER LAYER
// Primary adapter (driving adapter)
@RestController
@RequestMapping("/api/products")
public class ProductController {
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable String id) {
        ProductDTO product = productService.getProductById(new ProductId(id));
        return ResponseEntity.ok(product);
    }
    
    @PutMapping("/{id}/stock")
    public ResponseEntity<Void> updateStock(
            @PathVariable String id, 
            @RequestBody StockUpdateRequest request) {
        productService.updateStockLevel(new ProductId(id), request.getQuantity());
        return ResponseEntity.noContent().build();
    }
    
    @GetMapping("/available")
    public ResponseEntity<List<ProductDTO>> getAvailableProducts() {
        List<ProductDTO> products = productService.findAvailableProducts();
        return ResponseEntity.ok(products);
    }
}

// Secondary adapter (driven adapter)
@Repository
public class JpaProductRepository implements ProductRepository {
    private final SpringDataProductRepository springRepo;
    
    public JpaProductRepository(SpringDataProductRepository springRepo) {
        this.springRepo = springRepo;
    }
    
    @Override
    public Optional<Product> findById(ProductId id) {
        return springRepo.findById(id.getValue())
            .map(this::mapToDomain);
    }
    
    @Override
    public void save(Product product) {
        ProductEntity entity = mapToEntity(product);
        springRepo.save(entity);
    }
    
    @Override
    public List<Product> findByStockGreaterThan(int minimumStock) {
        return springRepo.findByStockQuantityGreaterThan(minimumStock).stream()
            .map(this::mapToDomain)
            .collect(Collectors.toList());
    }
    
    private Product mapToDomain(ProductEntity entity) {
        // Map JPA entity to domain entity
        return new Product(
            new ProductId(entity.getId()),
            entity.getName(),
            new Money(entity.getPrice()),
            entity.getStockQuantity()
        );
    }
    
    private ProductEntity mapToEntity(Product product) {
        // Map domain entity to JPA entity
        ProductEntity entity = new ProductEntity();
        entity.setId(product.getId().getValue());
        entity.setName(product.getName());
        entity.setPrice(product.getPrice().getAmount());
        entity.setStockQuantity(product.getStockQuantity());
        return entity;
    }
}

// The actual Spring Data JPA repository
interface SpringDataProductRepository extends JpaRepository<ProductEntity, String> {
    List<ProductEntity> findByStockQuantityGreaterThan(int minimumStock);
}
```

## Advantages and Limitations

### Advantages
- **Testability**: Core application can be tested in isolation
- **Flexibility**: External components can be replaced without changing the core
- **Protection**: Domain logic isolated from external concerns
- **Maintainability**: Clear boundaries between concerns
- **Framework Independence**: Core business logic not tied to frameworks

### Limitations
- **Complexity**: More interfaces and classes than simpler architectures
- **Overhead**: Additional mapping between layers can add development time
- **Learning Curve**: Conceptually more difficult than layered architecture
- **Indirection**: Can make code flow harder to follow
- **Overkill**: Unnecessary for simple CRUD applications

## Related Concepts
- Clean Architecture
- Onion Architecture
- Ports and Adapters
- Domain-Driven Design
- Dependency Inversion Principle 