# Model-View-Controller (MVC)

A software design pattern that separates an application into three interconnected components, promoting separation of concerns and code organization.

## Historical Context

**Origin**: Developed in the late 1970s by Trygve Reenskaug while working on Smalltalk at Xerox PARC.

**Evolution**: Initially called "Thing-Model-View-Editor," later simplified to Model-View-Controller. Gained widespread adoption during the rise of graphical user interfaces and web applications in the 1990s and 2000s.

## Core Concepts

MVC divides an application into three interconnected components:

1. **Model**: 
   - Manages data, logic, and rules of the application
   - Independent of the user interface
   - Directly manages the data, logic, and rules of the application

2. **View**:
   - Any representation of information (UI elements)
   - Multiple views can exist for a single model
   - Responsible for presenting data to the user

3. **Controller**:
   - Accepts input and converts it to commands for the model or view
   - Acts as an intermediary between Model and View
   - Handles user interaction and updates the Model accordingly

## Implementation

### Basic Flow
1. User interacts with the View (UI)
2. Controller receives input from View
3. Controller manipulates the Model
4. Model updates its state
5. View observes Model changes and updates itself

### Variations
- **Passive Model**: View and Controller both access the Model
- **Active Model**: Model notifies Views when data changes using Observer pattern
- **MVP (Model-View-Presenter)**: Presenter mediates between Model and View
- **MVVM (Model-View-ViewModel)**: ViewModel converts Model data for View consumption

## Examples

### Web Application MVC

```javascript
// Model
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  update(data) {
    this.name = data.name || this.name;
    this.email = data.email || this.email;
  }
}

// View
class UserView {
  constructor() {
    this.nameElement = document.getElementById('user-name');
    this.emailElement = document.getElementById('user-email');
  }
  
  render(user) {
    this.nameElement.textContent = user.name;
    this.emailElement.textContent = user.email;
  }
}

// Controller
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
  }
  
  updateUser(userData) {
    this.model.update(userData);
    this.view.render(this.model);
  }
}

// Usage
const user = new User('John Doe', 'john@example.com');
const view = new UserView();
const controller = new UserController(user, view);

// Initial render
view.render(user);

// User interaction handled by controller
document.getElementById('update-button').addEventListener('click', () => {
  controller.updateUser({
    name: document.getElementById('name-input').value
  });
});
```

## Advantages and Limitations

### Advantages
- **Separation of Concerns**: Each component has a distinct responsibility
- **Code Organization**: Logical structure makes large applications manageable
- **Parallel Development**: Teams can work on different components simultaneously
- **Testability**: Components can be tested independently
- **Reusability**: Views can be reused with different models

### Limitations
- **Complexity**: Can be overkill for simple applications
- **Tight Coupling**: Controllers often tightly coupled to Views
- **Learning Curve**: Developers new to MVC may find it hard to understand
- **Over-engineering**: May lead to excessive code for simple features

## Related Concepts
- Model-View-Presenter (MVP)
- Model-View-ViewModel (MVVM)
- Hexagonal Architecture
- Clean Architecture 