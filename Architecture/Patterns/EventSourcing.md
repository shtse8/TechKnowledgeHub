# Event Sourcing

An architectural pattern where application state changes are captured as a sequence of events, which become the primary source of truth from which the current state can be derived.

## Historical Context

**Origin**: Emerged in the early 2000s, with the concept formalized by Martin Fowler and Greg Young around 2005-2006.

**Evolution**: Gained prominence alongside Domain-Driven Design and CQRS patterns. Increasingly adopted in complex domains where audit trails, temporal queries, and high scalability are required. Popularized by frameworks like Axon, EventStore, and Lagom.

## Core Concepts

1. **Events as Facts**:
   - Events represent something that has happened in the past
   - Events are immutable and never modified once stored
   - Each event contains all necessary data to understand what changed
   - Events are named in past tense (e.g., "OrderPlaced", "ProductShipped")

2. **Event Store**:
   - Specialized database for storing and retrieving event sequences
   - Provides mechanism to store events and retrieve them by aggregate ID
   - Often supports event versioning, optimistic concurrency, and subscriptions
   - May provide projection capabilities to derive current state

3. **Event Streams**:
   - Sequence of events for a particular entity or aggregate
   - Typically organized by aggregate ID
   - Represents the complete history of changes to an entity

4. **Projections**:
   - Derived data models built by processing event streams
   - Multiple read models for different use cases
   - Can be built, destroyed, and rebuilt from the event store
   - Often used for query optimization

## Implementation

### Components of Event Sourcing

1. **Aggregate**:
   - Entity or cluster of entities treated as a single unit
   - Handles commands and produces events
   - Reconstructs its state by replaying events

2. **Command Handlers**:
   - Validate commands and apply business rules
   - Load aggregate from event store
   - Call aggregate methods to process commands
   - Save generated events

3. **Event Store**:
   - Provides APIs to append and retrieve events
   - Ensures event ordering and atomicity
   - Often provides optimistic concurrency control

4. **Projections/Read Models**:
   - Subscribe to events and update denormalized views
   - Optimized for query performance
   - Can be rebuilt from event store if needed

5. **Snapshotting**:
   - Optimization to avoid replaying all events
   - Periodically capture aggregate state
   - Combines snapshot plus subsequent events for reconstruction

### Integration Patterns

- **Event Sourcing with CQRS**: Separate write model (events) from read models (projections)
- **Event Collaboration**: Services react to events from other services
- **Sagas/Process Managers**: Coordinate complex workflows using events

## Example

### Bank Account Management

```typescript
// DOMAIN EVENTS
interface AccountEvent {
  accountId: string;
  timestamp: Date;
  version: number;
}

class AccountCreatedEvent implements AccountEvent {
  constructor(
    public accountId: string,
    public ownerName: string,
    public timestamp: Date,
    public version: number
  ) {}
}

class MoneyDepositedEvent implements AccountEvent {
  constructor(
    public accountId: string,
    public amount: number,
    public balance: number,
    public timestamp: Date,
    public version: number
  ) {}
}

class MoneyWithdrawnEvent implements AccountEvent {
  constructor(
    public accountId: string,
    public amount: number,
    public balance: number,
    public timestamp: Date,
    public version: number
  ) {}
}

class AccountOverdrawnEvent implements AccountEvent {
  constructor(
    public accountId: string,
    public balance: number,
    public timestamp: Date,
    public version: number
  ) {}
}

// AGGREGATE
class BankAccount {
  private id: string;
  private ownerName: string;
  private balance: number = 0;
  private overdrawn: boolean = false;
  private version: number = 0;
  private uncommittedEvents: AccountEvent[] = [];
  
  // Factory method to create a new account
  static create(id: string, ownerName: string): BankAccount {
    const account = new BankAccount();
    
    // Create and apply the event
    const event = new AccountCreatedEvent(
      id, 
      ownerName, 
      new Date(), 
      0
    );
    
    account.apply(event);
    account.uncommittedEvents.push(event);
    
    return account;
  }
  
  // Reconstitute account from events
  static fromEvents(events: AccountEvent[]): BankAccount {
    const account = new BankAccount();
    
    for (const event of events) {
      account.apply(event);
    }
    
    return account;
  }
  
  // Apply a single event to update state
  private apply(event: AccountEvent): void {
    if (event instanceof AccountCreatedEvent) {
      this.id = event.accountId;
      this.ownerName = event.ownerName;
      this.version = event.version;
    } 
    else if (event instanceof MoneyDepositedEvent) {
      this.balance = event.balance;
      this.overdrawn = this.balance < 0;
      this.version = event.version;
    } 
    else if (event instanceof MoneyWithdrawnEvent) {
      this.balance = event.balance;
      this.overdrawn = this.balance < 0;
      this.version = event.version;
    }
    else if (event instanceof AccountOverdrawnEvent) {
      this.overdrawn = true;
      this.version = event.version;
    }
  }
  
  // Business methods that generate events
  deposit(amount: number): void {
    if (amount <= 0) {
      throw new Error("Deposit amount must be positive");
    }
    
    const newBalance = this.balance + amount;
    
    const event = new MoneyDepositedEvent(
      this.id,
      amount,
      newBalance,
      new Date(),
      this.version + 1
    );
    
    this.apply(event);
    this.uncommittedEvents.push(event);
  }
  
  withdraw(amount: number): void {
    if (amount <= 0) {
      throw new Error("Withdrawal amount must be positive");
    }
    
    const newBalance = this.balance - amount;
    
    const withdrawalEvent = new MoneyWithdrawnEvent(
      this.id,
      amount,
      newBalance,
      new Date(),
      this.version + 1
    );
    
    this.apply(withdrawalEvent);
    this.uncommittedEvents.push(withdrawalEvent);
    
    // Check for overdraft and generate additional event if needed
    if (newBalance < 0 && !this.overdrawn) {
      const overdrawnEvent = new AccountOverdrawnEvent(
        this.id,
        newBalance,
        new Date(),
        this.version + 1
      );
      
      this.apply(overdrawnEvent);
      this.uncommittedEvents.push(overdrawnEvent);
    }
  }
  
  // Get and clear uncommitted events
  getUncommittedEvents(): AccountEvent[] {
    return [...this.uncommittedEvents];
  }
  
  clearUncommittedEvents(): void {
    this.uncommittedEvents = [];
  }
  
  // Getters for account properties
  getId(): string { return this.id; }
  getOwnerName(): string { return this.ownerName; }
  getBalance(): number { return this.balance; }
  isOverdrawn(): boolean { return this.overdrawn; }
  getVersion(): number { return this.version; }
}

// EVENT STORE (simplified)
class EventStore {
  private events: { [key: string]: AccountEvent[] } = {};
  
  // Save events for an aggregate
  async saveEvents(aggregateId: string, events: AccountEvent[], expectedVersion: number): Promise<void> {
    const existingStream = this.events[aggregateId] || [];
    
    // Optimistic concurrency check
    if (existingStream.length > 0 && existingStream[existingStream.length - 1].version !== expectedVersion) {
      throw new Error("Concurrency conflict - aggregate was modified");
    }
    
    this.events[aggregateId] = [...existingStream, ...events];
  }
  
  // Get events for an aggregate
  async getEventsForAggregate(aggregateId: string): Promise<AccountEvent[]> {
    return this.events[aggregateId] || [];
  }
}

// APPLICATION SERVICE
class BankAccountService {
  constructor(private eventStore: EventStore) {}
  
  async createAccount(accountId: string, ownerName: string): Promise<void> {
    const account = BankAccount.create(accountId, ownerName);
    
    await this.eventStore.saveEvents(
      accountId, 
      account.getUncommittedEvents(), 
      -1
    );
    
    account.clearUncommittedEvents();
  }
  
  async deposit(accountId: string, amount: number): Promise<void> {
    // Load events and reconstruct the account
    const events = await this.eventStore.getEventsForAggregate(accountId);
    if (events.length === 0) {
      throw new Error("Account not found");
    }
    
    const account = BankAccount.fromEvents(events);
    const currentVersion = account.getVersion();
    
    // Execute business logic
    account.deposit(amount);
    
    // Save new events
    await this.eventStore.saveEvents(
      accountId,
      account.getUncommittedEvents(),
      currentVersion
    );
    
    account.clearUncommittedEvents();
  }
  
  async withdraw(accountId: string, amount: number): Promise<void> {
    // Load events and reconstruct the account
    const events = await this.eventStore.getEventsForAggregate(accountId);
    if (events.length === 0) {
      throw new Error("Account not found");
    }
    
    const account = BankAccount.fromEvents(events);
    const currentVersion = account.getVersion();
    
    // Execute business logic
    account.withdraw(amount);
    
    // Save new events
    await this.eventStore.saveEvents(
      accountId,
      account.getUncommittedEvents(),
      currentVersion
    );
    
    account.clearUncommittedEvents();
  }
  
  // Query method that uses a projection
  async getAccountBalance(accountId: string): Promise<number> {
    const events = await this.eventStore.getEventsForAggregate(accountId);
    if (events.length === 0) {
      throw new Error("Account not found");
    }
    
    const account = BankAccount.fromEvents(events);
    return account.getBalance();
  }
}
```

## Advantages and Limitations

### Advantages
- **Complete Audit Trail**: Every change is captured as an immutable event
- **Temporal Queries**: Can determine state at any point in time
- **Business Insights**: Rich history enables advanced analytics
- **Debugging**: Easier to diagnose issues by replaying events
- **Evolvability**: New projections can be created for new requirements

### Limitations
- **Complexity**: More complex than CRUD-based architectures
- **Learning Curve**: Requires shift in development thinking
- **Performance**: Reconstructing state from events can be expensive
- **Event Schema Evolution**: Careful management of event versioning needed
- **Eventual Consistency**: Projections may lag behind event store

## Related Concepts
- Command Query Responsibility Segregation (CQRS)
- Domain-Driven Design (DDD)
- Message-Driven Architecture
- Stream Processing
- Immutable Data Patterns 