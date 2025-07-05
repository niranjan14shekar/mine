# Java Interview - Oral/Conceptual Coding Questions

## Applied Coding Questions for Verbal Interview Discussion

### Easy Level (Conceptual Understanding)

#### 1. Object Comparison Based on Field
**Question:** "You have a Student class with fields like name, age, and grade. How would you compare two Student objects based on their grade field? Explain the different approaches."

**Expected Answer Discussion Points:**
- Implement Comparable interface with compareTo() method
- Use Comparator interface for external comparison
- Override equals() and hashCode() for equality comparison
- Difference between natural ordering (Comparable) vs custom ordering (Comparator)
- Example: `Collections.sort(students, Comparator.comparing(Student::getGrade))`

#### 2. Finding Duplicate Objects in a List
**Question:** "Given a List of Employee objects, how would you find and remove duplicates based on employee ID? Walk me through your approach."

**Expected Answer Discussion Points:**
- Override equals() and hashCode() in Employee class based on ID
- Use HashSet to filter duplicates: `new ArrayList<>(new HashSet<>(employees))`
- Use Stream API: `employees.stream().distinct().collect(toList())`
- Use Map to track seen IDs and filter
- Discuss time complexity: O(n) vs O(n²)

#### 3. Grouping Objects by Property
**Question:** "You have a list of Order objects with customer names. How would you group all orders by customer name? Explain the approach."

**Expected Answer Discussion Points:**
- Use Stream API: `orders.stream().collect(groupingBy(Order::getCustomerName))`
- Manual approach with HashMap<String, List<Order>>
- Iterate through orders and populate map
- Result is Map<String, List<Order>>
- Mention Collectors.groupingBy() variations

#### 4. Sorting Objects by Multiple Criteria
**Question:** "How would you sort a list of Book objects first by author name, then by publication year? Describe your approach."

**Expected Answer Discussion Points:**
- Use Comparator.comparing() with thenComparing()
- `books.sort(Comparator.comparing(Book::getAuthor).thenComparing(Book::getYear))`
- For reverse order: `thenComparing(Book::getYear, Comparator.reverseOrder())`
- Custom Comparator implementation
- Handling null values with nullsFirst()/nullsLast()

#### 5. Converting Object List to Map
**Question:** "You have a List<Product> and want to create a Map where product ID is the key and product name is the value. How would you do this?"

**Expected Answer Discussion Points:**
- Stream API: `products.stream().collect(toMap(Product::getId, Product::getName))`
- Handle duplicate keys with merge function
- Manual iteration with HashMap
- Discuss what happens with duplicate keys
- Alternative: collect to Map<ID, Product> for full object access

---

### Medium Level (Problem-Solving Approaches)

#### 6. Finding Common Elements Between Collections
**Question:** "Given two lists of Student objects, how would you find students who appear in both lists? Assume students are same if they have same student ID."

**Expected Answer Discussion Points:**
- Override equals() and hashCode() based on student ID
- Use retainAll() method: `list1.retainAll(list2)`
- Convert one list to Set and filter the other
- Stream approach: `list1.stream().filter(list2::contains)`
- Discuss performance implications of different approaches

#### 7. Caching Strategy for Expensive Operations
**Question:** "You have a method that calculates complex statistics for a large dataset. The calculation is expensive but data doesn't change often. How would you implement caching?"

**Expected Answer Discussion Points:**
- Use ConcurrentHashMap for thread-safe caching
- Implement cache with expiration using timestamps
- Consider using existing solutions like Caffeine or Guava Cache
- Discuss cache eviction policies (LRU, TTL)
- Weak references for memory management
- Cache-aside vs write-through patterns

#### 8. Handling Null Values in Object Processing
**Question:** "When processing a list of Customer objects where some fields might be null (like email or phone), how would you safely handle operations like filtering and sorting?"

**Expected Answer Discussion Points:**
- Use Optional class for null-safe operations
- Stream filtering with null checks: `filter(c -> c.getEmail() != null)`
- Comparator with nullsFirst()/nullsLast()
- Defensive programming with null checks
- Default values vs Optional.orElse()
- Stream.ofNullable() for individual objects

#### 9. Converting Between Object Types
**Question:** "You need to convert a List<EmployeeEntity> (from database) to List<EmployeeDTO> (for API response). The DTO has fewer fields. How would you approach this transformation efficiently?"

**Expected Answer Discussion Points:**
- Stream map operation: `entities.stream().map(this::convertToDTO).collect(toList())`
- Builder pattern for DTO creation
- Use mapping libraries like MapStruct or ModelMapper
- Manual conversion method with null safety
- Consider performance for large datasets
- Batch processing for very large lists

#### 10. Implementing Observer Pattern for Object Changes
**Question:** "You have a ShoppingCart object and want to notify multiple listeners (UI, logging, analytics) when items are added/removed. How would you design this?"

**Expected Answer Discussion Points:**
- Implement Observer pattern with List<CartListener>
- Event-driven architecture with custom events
- Use PropertyChangeListener for specific field changes
- Consider using existing event libraries
- Discuss synchronous vs asynchronous notifications
- Thread safety for multiple listeners

---

### Difficult Level (Advanced Problem-Solving)

#### 11. Designing Thread-Safe Object Pool
**Question:** "Design an object pool for expensive-to-create database connections. The pool should be thread-safe, have max capacity, and reuse connections efficiently. Walk me through your design."

**Expected Answer Discussion Points:**
- Use BlockingQueue for thread-safe pool management
- acquire() and release() methods with timeout
- Connection validation before reuse
- Max pool size and connection lifecycle management
- Handle connection leaks with weak references
- Consider existing solutions like HikariCP
- Monitoring and metrics for pool health

#### 12. Implementing Custom Equals and HashCode
**Question:** "You have a Person class with firstName, lastName, and dateOfBirth. Two persons are equal if all three fields match. How would you implement equals() and hashCode()? What are the important considerations?"

**Expected Answer Discussion Points:**
- Override both equals() and hashCode() together
- Check for null values and handle appropriately
- Use Objects.equals() for null-safe comparison
- HashCode should use same fields as equals()
- Symmetric, reflexive, and transitive properties
- Consider performance and hash distribution
- Immutable objects are safer for hashing

#### 13. Memory-Efficient Processing of Large Object Collections
**Question:** "You need to process 10 million Customer records to generate a report, but you can't load all objects into memory at once. How would you approach this?"

**Expected Answer Discussion Points:**
- Stream processing with pagination/batching
- Database cursor-based iteration
- Parallel processing with limited concurrency
- Use try-with-resources for proper cleanup
- Consider memory mapping for file-based data
- JVM tuning for large heap vs frequent GC
- Monitoring memory usage during processing

#### 14. Implementing Custom Serialization for Complex Objects
**Question:** "You have a complex object graph with circular references that needs to be serialized to JSON for an API. How would you handle this without getting infinite loops?"

**Expected Answer Discussion Points:**
- Use @JsonIgnore on back-references
- Implement @JsonIdentityInfo for reference handling
- Custom serializer with visited object tracking
- DTOs without circular references
- Lazy loading strategies
- Consider Jackson's @JsonManagedReference/@JsonBackReference
- Graph traversal with cycle detection

#### 15. Designing Event Sourcing for Object State Changes
**Question:** "Instead of storing current object state, you want to store all events that led to the current state. How would you design this for a BankAccount object with deposit/withdrawal operations?"

**Expected Answer Discussion Points:**
- Event classes (DepositEvent, WithdrawEvent) with immutable data
- Event store as append-only log
- Replay events to rebuild current state
- Snapshots for performance optimization
- Command vs Event separation
- Handling concurrent events
- Event versioning for schema evolution
- Consider frameworks like Axon or EventStore

---

### Advanced Object-Oriented Design Questions

#### 16. Factory Pattern for Complex Object Creation
**Question:** "You need to create different types of Payment objects (CreditCard, PayPal, BankTransfer) based on configuration. Each has different validation rules and processing logic. How would you design the factory?"

**Expected Answer Discussion Points:**
- Abstract factory pattern with payment type enum
- Strategy pattern for validation and processing
- Builder pattern for complex payment objects
- Configuration-driven factory selection
- Dependency injection for payment processors
- Plugin architecture for extensibility

#### 17. Implementing Immutable Objects with Builder
**Question:** "Design an immutable User class with many optional fields (name, email, phone, address, preferences). How would you make object creation convenient while maintaining immutability?"

**Expected Answer Discussion Points:**
- Builder pattern for optional parameters
- Final fields and no setters
- Defensive copying for mutable fields (List, Date)
- Static nested Builder class
- Validation in builder's build() method
- Consider using Lombok or records (Java 14+)
- Thread safety benefits of immutability

#### 18. Composite Pattern for Hierarchical Data
**Question:** "You need to represent a company's organizational structure where employees can have subordinates, and you want to calculate total salary for any level of the hierarchy. How would you model this?"

**Expected Answer Discussion Points:**
- Composite pattern with Component interface
- Employee as leaf, Manager as composite
- Recursive salary calculation
- Tree traversal strategies (DFS, BFS)
- Visitor pattern for different operations
- Consider performance for deep hierarchies
- Parent-child relationship management

#### 19. Decorator Pattern for Dynamic Object Enhancement
**Question:** "You have a basic Message class and want to add features like encryption, compression, and logging dynamically at runtime. How would you design this?"

**Expected Answer Discussion Points:**
- Decorator pattern with Message interface
- Chain of decorators for multiple features
- Order dependency of decorators
- Performance implications of wrapping
- Alternative: Strategy pattern with feature flags
- Proxy pattern for cross-cutting concerns
- AOP (Aspect-Oriented Programming) alternatives

#### 20. Command Pattern with Undo/Redo Functionality
**Question:** "Design a text editor where users can perform operations (insert, delete, format) and have undo/redo capabilities. How would you implement this?"

**Expected Answer Discussion Points:**
- Command pattern with execute() and undo() methods
- Command history stack for undo operations
- Separate redo stack or bidirectional commands
- Macro commands for composite operations
- Memory management for large command histories
- State preservation vs operation reversal
- Consider memento pattern for state snapshots 