# Command Query Responsibility Segregation (CQRS)

An architectural pattern that separates read operations (queries) from write operations (commands), allowing each to be optimized independently.

## Historical Context

**Origin**: Introduced by Greg Young around 2010, building on concepts from Command-Query Separation (CQS) principle defined by Bertrand Meyer in the 1980s.

**Evolution**: Gained traction in complex domain-driven applications where performance, scalability, and complexity management were key concerns. Often implemented alongside Event Sourcing in modern distributed systems.

## Core Concepts

1. **Command-Query Separation**:
   - **Commands**: Change the state of the system but do not return data
   - **Queries**: Return data but do not change the system state
   - Operations are classified as either commands or queries, never both

2. **Model Segregation**:
   - **Write Model**: Optimized for data modification operations
   - **Read Model**: Optimized for data retrieval operations
   - Different models used for different purposes

3. **Data Flow**:
   - Commands flow through validation, business logic, and persistence
   - Queries retrieve data from read-optimized stores or projections
   - Synchronization mechanism updates read models from write models

4. **Responsibility Division**:
   - Different components handle commands and queries
   - Each side can be scaled, optimized, and evolved independently
   - Clear separation of business logic from query requirements

## Implementation

### Components

1. **Command Side**:
   - **Command Objects**: Immutable objects representing user intent
   - **Command Handlers**: Process commands and apply business rules
   - **Aggregate Models**: Domain entities that enforce consistency rules
   - **Event Store/Write Database**: Persists state changes

2. **Query Side**:
   - **Query Objects**: Request for specific data
   - **Query Handlers**: Retrieve and format data
   - **Read Models/Projections**: Data structures optimized for queries
   - **Read Database**: Stores denormalized views of data

3. **Synchronization Mechanisms**:
   - **Eventual Consistency**: Read model updated asynchronously
   - **Event Handlers**: Listen for domain events and update read models
   - **Background Processes**: Rebuild read models from write data

### Variations

- **Simple CQRS**: Same database but different models for reads and writes
- **Full CQRS**: Completely separate databases for reads and writes
- **CQRS with Event Sourcing**: Events as the primary source of truth with read projections

## Example

### Online Shopping Platform

```csharp
// Command Side
public class AddToCartCommand
{
    public Guid CartId { get; }
    public Guid ProductId { get; }
    public int Quantity { get; }
    
    public AddToCartCommand(Guid cartId, Guid productId, int quantity)
    {
        CartId = cartId;
        ProductId = productId;
        Quantity = quantity;
    }
}

public class AddToCartCommandHandler
{
    private readonly ICartRepository _cartRepository;
    private readonly IProductRepository _productRepository;
    private readonly IEventBus _eventBus;
    
    // Constructor with dependencies...
    
    public async Task Handle(AddToCartCommand command)
    {
        // Retrieve aggregate
        var cart = await _cartRepository.GetByIdAsync(command.CartId);
        var product = await _productRepository.GetByIdAsync(command.ProductId);
        
        // Execute business logic
        cart.AddItem(product, command.Quantity);
        
        // Save changes
        await _cartRepository.SaveAsync(cart);
        
        // Publish event for read side
        _eventBus.Publish(new ItemAddedToCartEvent(
            command.CartId, 
            command.ProductId,
            product.Name,
            product.Price,
            command.Quantity
        ));
    }
}

// Query Side
public class GetCartSummaryQuery
{
    public Guid CartId { get; }
    
    public GetCartSummaryQuery(Guid cartId)
    {
        CartId = cartId;
    }
}

public class CartSummaryDto
{
    public Guid CartId { get; set; }
    public List<CartItemDto> Items { get; set; }
    public decimal TotalAmount { get; set; }
    public int ItemCount { get; set; }
}

public class GetCartSummaryQueryHandler
{
    private readonly IReadDbContext _readDb;
    
    public GetCartSummaryQueryHandler(IReadDbContext readDb)
    {
        _readDb = readDb;
    }
    
    public async Task<CartSummaryDto> Handle(GetCartSummaryQuery query)
    {
        // Direct query to optimized read model
        return await _readDb.CartSummaries
            .AsNoTracking()
            .Where(c => c.CartId == query.CartId)
            .FirstOrDefaultAsync();
    }
}

// Event Handler for Synchronization
public class CartEventHandler
{
    private readonly IReadDbContext _readDb;
    
    public CartEventHandler(IReadDbContext readDb)
    {
        _readDb = readDb;
    }
    
    public async Task Handle(ItemAddedToCartEvent @event)
    {
        var summary = await _readDb.CartSummaries
            .FirstOrDefaultAsync(c => c.CartId == @event.CartId);
            
        if (summary == null)
        {
            summary = new CartSummaryDto
            {
                CartId = @event.CartId,
                Items = new List<CartItemDto>(),
                TotalAmount = 0,
                ItemCount = 0
            };
            _readDb.CartSummaries.Add(summary);
        }
        
        var existingItem = summary.Items
            .FirstOrDefault(i => i.ProductId == @event.ProductId);
            
        if (existingItem != null)
        {
            existingItem.Quantity += @event.Quantity;
        }
        else
        {
            summary.Items.Add(new CartItemDto
            {
                ProductId = @event.ProductId,
                ProductName = @event.ProductName,
                Price = @event.Price,
                Quantity = @event.Quantity
            });
        }
        
        summary.TotalAmount += @event.Price * @event.Quantity;
        summary.ItemCount += @event.Quantity;
        
        await _readDb.SaveChangesAsync();
    }
}
```

## Advantages and Limitations

### Advantages
- **Scalability**: Read and write sides can be scaled independently
- **Performance**: Models can be optimized for specific use cases
- **Complexity Management**: Separates complex domain logic from queries
- **Specialized Persistence**: Different storage types for different needs
- **Task-Based UI**: Aligns well with user intent and actions

### Limitations
- **Increased Complexity**: More components, synchronization challenges
- **Eventual Consistency**: Read model may lag behind write model
- **Development Overhead**: Duplicate models and synchronization code
- **Learning Curve**: Differs from traditional CRUD approaches
- **Overkill**: Unnecessary for simple applications with modest scale requirements

## Related Concepts
- Event Sourcing
- Domain-Driven Design
- Microservices Architecture
- Event-Driven Architecture
- Materialized Views 