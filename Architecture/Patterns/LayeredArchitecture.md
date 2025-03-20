# Layered Architecture

A design pattern that organizes software into horizontal layers, each with a specific role and responsibility, promoting separation of concerns and providing a clear structure for organizing code.

## Historical Context

**Origin**: Emerged in the 1970s with the development of early networking protocols, particularly the OSI model (1984), which conceptualized network communication in seven distinct layers.

**Evolution**: Became a fundamental architectural approach in enterprise software development during the 1990s and 2000s, influencing frameworks like Java EE and .NET. Has remained a standard architectural pattern for organizing complex applications.

## Core Concepts

1. **Layer Separation**:
   - Code organized into horizontal layers
   - Each layer has a specific responsibility
   - Layers communicate through well-defined interfaces

2. **Layer Dependencies**:
   - Higher layers depend on lower layers (downward dependencies)
   - Lower layers should not depend on higher layers
   - Dependencies are typically unidirectional (top-to-bottom)

3. **Abstraction Levels**:
   - Each layer represents a different level of abstraction
   - Higher layers deal with user interaction and business rules
   - Lower layers handle technical and infrastructure concerns

4. **Layer Isolation**:
   - Changes in one layer should not affect other layers
   - Implementation details of a layer are hidden from other layers
   - Each layer can be replaced or modified independently

## Implementation

### Common Layers

1. **Presentation Layer**:
   - Handles user interface and user interaction
   - Formats data for display and interprets user input
   - Usually implemented as web pages, mobile UI, or desktop UI

2. **Application Layer** (Service Layer):
   - Orchestrates business operations and workflows
   - Implements application-specific logic
   - Coordinates interactions between different domain objects

3. **Domain Layer** (Business Layer):
   - Contains business entities and business rules
   - Implements core business logic
   - Independent of presentation and persistence concerns

4. **Data Access Layer**:
   - Provides access to external data sources
   - Abstracts database interactions
   - Handles data persistence and retrieval

5. **Infrastructure Layer** (optional):
   - Provides technical capabilities to other layers
   - Handles cross-cutting concerns (logging, security, etc.)
   - Implements interfaces with external systems

### Communication Between Layers

Typically follows one of these patterns:
- **Strict Layering**: Each layer can only access the layer immediately below it
- **Relaxed Layering**: A layer can access any layer below it
- **Request-Response**: Higher layers send requests, lower layers return responses

## Example

### Java Enterprise Application

```java
// Presentation Layer
@Controller
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/users/{id}")
    public ModelAndView getUser(@PathVariable Long id) {
        UserDTO user = userService.findById(id);
        ModelAndView modelAndView = new ModelAndView("user-details");
        modelAndView.addObject("user", user);
        return modelAndView;
    }
}

// Application Layer
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        return convertToDTO(user);
    }
    
    private UserDTO convertToDTO(User user) {
        // Map domain object to DTO
        return new UserDTO(user.getId(), user.getUsername(), user.getEmail());
    }
}

// Domain Layer
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    private String passwordHash;
    
    // Methods that implement business rules
    public boolean isPasswordValid(String password) {
        return BCrypt.checkpw(password, this.passwordHash);
    }
    
    // Getters, setters, and other domain logic
}

// Data Access Layer
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByEmail(String email);
}
```

## Advantages and Limitations

### Advantages
- **Separation of Concerns**: Each layer has a single responsibility
- **Maintainability**: Easier to understand, maintain, and modify
- **Testability**: Layers can be tested in isolation
- **Flexibility**: Individual layers can be replaced or modified
- **Standardization**: Team members have a clear understanding of code organization

### Limitations
- **Performance Overhead**: Communication between layers can add overhead
- **Rigidity**: Can lead to unnecessary complexity for simple applications
- **Over-abstraction**: May result in excessive abstraction and boilerplate code
- **Layer Violations**: Developers may bypass layers for convenience
- **Monolithic Tendency**: Can still result in a large, tightly coupled codebase

## Related Concepts
- N-Tier Architecture
- Onion Architecture
- Clean Architecture
- Hexagonal Architecture (Ports and Adapters)
- Model-View-Controller (MVC) 