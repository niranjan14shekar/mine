# Java Oral Coding Questions - Design Patterns & Architecture

## 10 Questions on Design Patterns and Architecture

### 1. Implementing Observer Pattern for Object Changes
**Question:** "You have a ShoppingCart object and want to notify multiple listeners (UI, logging, analytics) when items are added/removed. How would you design this?"

**Expected Answer Discussion Points:**
- Observer pattern with List<CartListener> interface
- Event-driven architecture with custom events
- PropertyChangeListener for specific field changes
- Synchronous vs asynchronous notifications
- Thread safety considerations for multiple listeners
- Weak references to prevent memory leaks

### 2. Factory Pattern for Complex Object Creation
**Question:** "You need to create different types of Payment objects (CreditCard, PayPal, BankTransfer) based on user selection. Each has different validation rules. How would you design the factory?"

**Expected Answer Discussion Points:**
- Abstract factory pattern with payment type enum
- Strategy pattern for validation and processing
- Builder pattern for complex payment objects
- Configuration-driven factory selection
- Dependency injection for payment processors
- Plugin architecture for extensibility

### 3. Singleton Pattern with Thread Safety
**Question:** "Design a ConfigurationManager that should have only one instance throughout the application and be thread-safe. Walk me through different implementation approaches."

**Expected Answer Discussion Points:**
- Double-checked locking pattern
- Enum-based singleton (thread-safe by default)
- Static inner class approach (lazy initialization)
- Volatile keyword importance
- Early vs lazy initialization trade-offs
- Testing challenges with singletons

### 4. Strategy Pattern for Dynamic Behavior
**Question:** "You have a PriceCalculator that needs to apply different discount strategies (percentage, fixed amount, buy-one-get-one). How would you implement this flexibly?"

**Expected Answer Discussion Points:**
- Strategy interface with different implementations
- Context class that uses strategy
- Runtime strategy selection
- Combining multiple strategies
- Configuration-driven strategy selection
- Lambda expressions for simple strategies

### 5. Decorator Pattern for Feature Enhancement
**Question:** "You have a basic Coffee class and want to add features like milk, sugar, extra shots dynamically. How would you design this to avoid class explosion?"

**Expected Answer Discussion Points:**
- Decorator pattern with component interface
- Chain of decorators for multiple features
- Order dependency considerations
- Cost calculation through decoration chain
- Alternative: Builder pattern with feature flags
- Performance implications of multiple wrappers

### 6. Command Pattern for Undo Operations
**Question:** "Design a text editor where users can perform operations (insert, delete, format) with undo/redo functionality. How would you implement this?"

**Expected Answer Discussion Points:**
- Command interface with execute() and undo() methods
- Command history stack for undo operations
- Separate redo stack or bidirectional commands
- Macro commands for composite operations
- Memory management for large command histories
- State preservation vs operation reversal

### 7. Template Method Pattern for Processing Pipeline
**Question:** "You have different types of data import processes (CSV, XML, JSON) that follow the same steps: validate, parse, transform, save. How would you avoid code duplication?"

**Expected Answer Discussion Points:**
- Template method pattern with abstract base class
- Hook methods for customization points
- Protected methods for common functionality
- Alternative: functional approach with strategy
- Error handling in template methods
- Extensibility for new data formats

### 8. Adapter Pattern for External Integration
**Question:** "You need to integrate with multiple payment gateways that have different APIs. How would you create a unified interface for your application?"

**Expected Answer Discussion Points:**
- Adapter pattern to wrap external APIs
- Common interface for payment operations
- Error handling and translation between systems
- Configuration for different adapters
- Testing strategies for external dependencies
- Fallback mechanisms for failed adapters

### 9. Composite Pattern for Hierarchical Structures
**Question:** "Model a file system where you can have files and directories, and you want to calculate total size for any level. How would you design this?"

**Expected Answer Discussion Points:**
- Composite pattern with File/Directory components
- Recursive size calculation
- Tree traversal strategies (DFS, BFS)
- Visitor pattern for different operations
- Parent-child relationship management
- Performance for deep hierarchies

### 10. Builder Pattern for Complex Object Construction
**Question:** "Create a DatabaseConnection class that has many optional parameters (timeout, pool size, SSL settings, etc.). How would you make object creation clean and readable?"

**Expected Answer Discussion Points:**
- Builder pattern with fluent interface
- Static nested Builder class
- Validation in build() method
- Immutable objects from builder
- Method chaining for readability
- Default values and required parameters 