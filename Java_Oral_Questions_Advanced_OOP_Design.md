# Java Oral Coding Questions - Advanced OOP Design

## 10 Questions on Advanced Object-Oriented Design

### 1. Implementing Custom Equals and HashCode
**Question:** "You have a Person class with firstName, lastName, and dateOfBirth. Two persons are equal if all three fields match. How would you implement equals() and hashCode() correctly?"

**Expected Answer Discussion Points:**
- Override both equals() and hashCode() together
- Null safety with Objects.equals() and Objects.hash()
- Symmetric, reflexive, and transitive properties
- HashCode contract and performance considerations
- Using same fields in both methods
- Immutable objects benefits for hashing

### 2. Designing Immutable Objects with Builder
**Question:** "Create an immutable Address class with many optional fields (street, city, state, country, zipCode). How would you handle object creation and validation?"

**Expected Answer Discussion Points:**
- Final fields and no setters
- Builder pattern for optional parameters
- Defensive copying for mutable fields
- Validation in builder's build() method
- Static nested Builder class design
- Thread safety benefits of immutability

### 3. Inheritance vs Composition Design Decision
**Question:** "You're modeling vehicles where cars have engines and boats have engines too, but with different behaviors. How would you structure this to avoid deep inheritance hierarchies?"

**Expected Answer Discussion Points:**
- Favor composition over inheritance principle
- Engine as a composed component
- Interface segregation for different vehicle behaviors
- Strategy pattern for engine-specific behavior
- Avoiding fragile base class problems
- Multiple inheritance simulation with interfaces

### 4. Generic Type Design with Wildcards
**Question:** "Design a container class that can hold collections of any type but allows type-safe operations. How would you handle adding and retrieving elements?"

**Expected Answer Discussion Points:**
- Generic type parameters with bounds
- Wildcard usage (? extends, ? super)
- PECS principle (Producer Extends, Consumer Super)
- Type erasure implications
- Raw types vs parameterized types
- Bridge methods and type safety

### 5. Interface Design with Default Methods
**Question:** "You have a PaymentProcessor interface used by many implementations. You need to add logging functionality to all implementations without breaking existing code. How would you approach this?"

**Expected Answer Discussion Points:**
- Default methods in interfaces (Java 8+)
- Interface evolution without breaking changes
- Multiple inheritance of behavior
- Diamond problem resolution
- When to use default vs abstract methods
- Decorator pattern as alternative

### 6. Enum Design for Complex State Management
**Question:** "Model an order status system where orders can transition between states (PENDING → CONFIRMED → SHIPPED → DELIVERED) with validation rules. How would you implement this?"

**Expected Answer Discussion Points:**
- Enum with behavior methods
- State transition validation
- Abstract methods in enums
- Enum constructor for additional data
- Strategy pattern with enums
- Finite state machine implementation

### 7. Exception Hierarchy Design
**Question:** "Design a custom exception hierarchy for a banking system that handles different types of errors (insufficient funds, invalid account, network issues). How would you structure this?"

**Expected Answer Discussion Points:**
- Checked vs unchecked exception decisions
- Exception hierarchy with meaningful inheritance
- Error codes and additional context
- Exception chaining and cause preservation
- Recovery strategies for different exception types
- Performance implications of exception handling

### 8. Fluent Interface Design
**Question:** "Create a QueryBuilder class that allows method chaining like query.select('name').from('users').where('age > 18').orderBy('name'). How would you implement this?"

**Expected Answer Discussion Points:**
- Method chaining by returning 'this'
- Builder pattern with fluent interface
- Immutable builder vs mutable builder
- Type-safe builder with compile-time validation
- Method ordering constraints
- Performance of object creation

### 9. Visitor Pattern for Object Processing
**Question:** "You have a document structure with different element types (paragraphs, images, tables). You need to implement various operations (rendering, word counting, spell checking) without modifying the element classes. How would you design this?"

**Expected Answer Discussion Points:**
- Visitor pattern with double dispatch
- Element interface with accept() method
- Visitor interface with visit() methods for each type
- Adding new operations without modifying elements
- Type safety and compile-time checking
- Performance implications of double dispatch

### 10. Dynamic Proxy for Cross-Cutting Concerns
**Question:** "You want to add logging, timing, and security checks to service methods without modifying the service classes directly. How would you implement this interceptor functionality?"

**Expected Answer Discussion Points:**
- Dynamic proxy with InvocationHandler
- Interceptor pattern implementation
- Aspect-oriented programming concepts
- Proxy vs decorator pattern differences
- Performance overhead of dynamic proxies
- Alternative: bytecode manipulation or annotations 