# Java Interview Theory Questions

## Table of Contents
- [Easy Questions (1-10)](#easy-questions)
- [Medium Questions (11-20)](#medium-questions)
- [Difficult Questions (21-30)](#difficult-questions)

---

## Easy Questions

### 1. What is Java and what are its main features?
**Answer:** Java is a high-level, object-oriented programming language developed by Sun Microsystems (now Oracle). Its main features include:
- **Platform Independence**: "Write Once, Run Anywhere" (WORA) capability through JVM
- **Object-Oriented**: Supports encapsulation, inheritance, and polymorphism
- **Simple and Familiar**: C/C++ like syntax but removes complex features like pointers
- **Robust**: Strong memory management and exception handling
- **Secure**: Built-in security features and sandbox execution
- **Multithreaded**: Built-in support for concurrent programming
- **Automatic Memory Management**: Garbage collection handles memory deallocation

### 2. What is the difference between JDK, JRE, and JVM?
**Answer:**
- **JVM (Java Virtual Machine)**: Runtime environment that executes Java bytecode. Platform-specific but provides platform independence for Java programs.
- **JRE (Java Runtime Environment)**: Includes JVM + standard libraries + supporting files needed to run Java applications. For end users.
- **JDK (Java Development Kit)**: Includes JRE + development tools (compiler, debugger, etc.). For developers.

Relationship: JDK ⊃ JRE ⊃ JVM

### 3. What are the different data types in Java?
**Answer:**
**Primitive Data Types (8):**
- **byte**: 8-bit signed integer (-128 to 127)
- **short**: 16-bit signed integer (-32,768 to 32,767)
- **int**: 32-bit signed integer (-2³¹ to 2³¹-1)
- **long**: 64-bit signed integer (-2⁶³ to 2⁶³-1)
- **float**: 32-bit IEEE 754 floating point
- **double**: 64-bit IEEE 754 floating point
- **boolean**: true or false
- **char**: 16-bit Unicode character

**Non-primitive Data Types:**
- Classes, Interfaces, Arrays, Strings

### 4. What is the difference between == and equals() method?
**Answer:**
- **==**: Compares references (memory addresses) for objects, values for primitives
- **equals()**: Compares the actual content/values of objects (when properly overridden)

```java
String s1 = new String("Hello");
String s2 = new String("Hello");
System.out.println(s1 == s2);        // false (different references)
System.out.println(s1.equals(s2));   // true (same content)
```

### 5. What is method overloading?
**Answer:** Method overloading allows multiple methods with the same name but different parameters (different type, number, or order of parameters) in the same class. Compiler determines which method to call based on method signature.

```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}
```

### 6. What are access modifiers in Java?
**Answer:**
- **public**: Accessible from anywhere
- **private**: Accessible only within the same class
- **protected**: Accessible within same package and subclasses
- **default (package-private)**: Accessible only within the same package

| Modifier | Same Class | Same Package | Subclass | Other Package |
|----------|------------|--------------|----------|---------------|
| public   | ✓          | ✓            | ✓        | ✓             |
| protected| ✓          | ✓            | ✓        | ✗             |
| default  | ✓          | ✓            | ✗        | ✗             |
| private  | ✓          | ✗            | ✗        | ✗             |

### 7. What is the difference between String, StringBuffer, and StringBuilder?
**Answer:**
- **String**: Immutable, thread-safe. Each modification creates a new object.
- **StringBuffer**: Mutable, thread-safe (synchronized methods). Good for multi-threaded environments.
- **StringBuilder**: Mutable, not thread-safe. Faster than StringBuffer for single-threaded applications.

```java
String s = "Hello";
s += " World";  // Creates new String object

StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies existing object
```

### 8. What is inheritance in Java?
**Answer:** Inheritance is a mechanism where a new class (subclass/child) inherits properties and methods from an existing class (superclass/parent). It promotes code reusability and establishes IS-A relationship.

```java
class Animal {
    void eat() { System.out.println("Animal eats"); }
}

class Dog extends Animal {
    void bark() { System.out.println("Dog barks"); }
}
```

Java supports single inheritance (one parent class) but multiple interface implementation.

### 9. What is polymorphism?
**Answer:** Polymorphism means "one interface, multiple implementations." It allows objects of different classes to be treated as objects of a common base class.

**Types:**
- **Compile-time (Static)**: Method overloading
- **Runtime (Dynamic)**: Method overriding

```java
Animal animal = new Dog();  // Polymorphic assignment
animal.eat();               // Calls Dog's version if overridden
```

### 10. What is encapsulation?
**Answer:** Encapsulation is the bundling of data (variables) and methods that operate on that data within a single unit (class), and restricting direct access to object's internal state using access modifiers.

```java
public class Student {
    private String name;  // Private data
    
    public String getName() { return name; }        // Public getter
    public void setName(String name) { this.name = name; }  // Public setter
}
```

---

## Medium Questions

### 11. What is the difference between abstract class and interface?
**Answer:**

| Abstract Class | Interface |
|----------------|-----------|
| Can have concrete and abstract methods | All methods are abstract (until Java 8) |
| Can have instance variables | Only constants (public static final) |
| Can have constructors | Cannot have constructors |
| Supports single inheritance | Supports multiple inheritance |
| Can have access modifiers | Methods are public by default |
| Use 'extends' keyword | Use 'implements' keyword |

**Java 8+ Interfaces can have:**
- Default methods (with implementation)
- Static methods
- Private methods (Java 9+)

### 12. What is exception handling in Java?
**Answer:** Exception handling is a mechanism to handle runtime errors gracefully using try-catch-finally blocks.

**Exception Hierarchy:**
```
Throwable
├── Error (System-level errors)
└── Exception
    ├── RuntimeException (Unchecked)
    └── Other Exceptions (Checked)
```

**Types:**
- **Checked Exceptions**: Must be handled (IOException, SQLException)
- **Unchecked Exceptions**: Runtime exceptions (NullPointerException, ArrayIndexOutOfBoundsException)

```java
try {
    // Risky code
} catch (SpecificException e) {
    // Handle specific exception
} catch (Exception e) {
    // Handle general exception
} finally {
    // Always executes
}
```

### 13. What is the difference between final, finally, and finalize?
**Answer:**
- **final**: Keyword used to restrict inheritance, method overriding, or variable reassignment
  ```java
  final class FinalClass {}  // Cannot be extended
  final void method() {}     // Cannot be overridden
  final int x = 10;         // Cannot be reassigned
  ```

- **finally**: Block that always executes regardless of exception occurrence
  ```java
  try { /* code */ } finally { /* cleanup code */ }
  ```

- **finalize()**: Method called by garbage collector before object destruction (deprecated in Java 9+)
  ```java
  protected void finalize() throws Throwable {
      // Cleanup code
  }
  ```

### 14. What are collections in Java?
**Answer:** Collections Framework provides a unified architecture for storing and manipulating groups of objects.

**Core Interfaces:**
- **Collection**: Root interface
  - **List**: Ordered collection (ArrayList, LinkedList, Vector)
  - **Set**: No duplicates (HashSet, TreeSet, LinkedHashSet)
  - **Queue**: FIFO operations (PriorityQueue, ArrayDeque)
- **Map**: Key-value pairs (HashMap, TreeMap, LinkedHashMap)

**Hierarchy:**
```
Collection
├── List (ArrayList, LinkedList)
├── Set (HashSet, TreeSet)
└── Queue (PriorityQueue)

Map (HashMap, TreeMap) - separate hierarchy
```

### 15. What is multithreading and how do you create threads in Java?
**Answer:** Multithreading allows concurrent execution of multiple threads to maximize CPU utilization.

**Creating Threads:**
1. **Extending Thread class:**
```java
class MyThread extends Thread {
    public void run() { /* thread logic */ }
}
MyThread t = new MyThread();
t.start();
```

2. **Implementing Runnable interface:**
```java
class MyTask implements Runnable {
    public void run() { /* thread logic */ }
}
Thread t = new Thread(new MyTask());
t.start();
```

**Thread States:** NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED

### 16. What is synchronization and why is it needed?
**Answer:** Synchronization ensures thread safety by controlling access to shared resources, preventing race conditions and data inconsistency.

**Types:**
1. **Method Synchronization:**
```java
synchronized void method() { /* synchronized code */ }
```

2. **Block Synchronization:**
```java
synchronized(object) { /* synchronized code */ }
```

**Why needed:**
- Prevents race conditions
- Ensures data consistency
- Provides thread safety in multi-threaded environments

### 17. What is garbage collection in Java?
**Answer:** Garbage Collection (GC) is automatic memory management that removes unreferenced objects from heap memory.

**GC Process:**
1. **Mark**: Identify which objects are still referenced
2. **Sweep**: Remove unreferenced objects
3. **Compact**: Defragment memory

**Types of GC:**
- **Serial GC**: Single-threaded
- **Parallel GC**: Multi-threaded
- **G1 GC**: Low-latency collector
- **ZGC/Shenandoah**: Ultra-low latency collectors

**Memory Areas:**
- **Young Generation**: Eden + Survivor spaces
- **Old Generation**: Long-lived objects
- **Metaspace**: Class metadata (Java 8+)

### 18. What is the difference between ArrayList and LinkedList?
**Answer:**

| ArrayList | LinkedList |
|-----------|------------|
| Dynamic array implementation | Doubly linked list implementation |
| Random access: O(1) | Sequential access: O(n) |
| Insertion/deletion at middle: O(n) | Insertion/deletion at middle: O(1) |
| Memory efficient | Extra memory for pointers |
| Better for frequent access | Better for frequent insertions/deletions |
| Implements List, RandomAccess | Implements List, Deque |

**Use Cases:**
- **ArrayList**: When frequent random access is needed
- **LinkedList**: When frequent insertions/deletions in middle

### 19. What is method overriding and what are its rules?
**Answer:** Method overriding allows a subclass to provide a specific implementation of a method already defined in its parent class.

**Rules:**
1. Same method signature (name, parameters)
2. IS-A relationship required (inheritance)
3. Cannot override static, final, or private methods
4. Return type must be same or covariant
5. Access modifier cannot be more restrictive
6. Cannot throw broader checked exceptions

```java
class Parent {
    protected void display() { System.out.println("Parent"); }
}

class Child extends Parent {
    @Override
    public void display() { System.out.println("Child"); }  // Valid override
}
```

### 20. What is the difference between static and non-static methods?
**Answer:**

| Static Methods | Non-Static Methods |
|----------------|-------------------|
| Belong to class | Belong to instance |
| Called using class name | Called using object reference |
| Cannot access instance variables | Can access all variables |
| Cannot be overridden | Can be overridden |
| Loaded at class loading time | Created with object |
| Cannot use 'this' or 'super' | Can use 'this' and 'super' |

```java
class Example {
    static int staticVar = 10;
    int instanceVar = 20;
    
    static void staticMethod() {
        System.out.println(staticVar);    // OK
        // System.out.println(instanceVar); // Error
    }
    
    void instanceMethod() {
        System.out.println(staticVar);    // OK
        System.out.println(instanceVar);  // OK
    }
}
```

---

## Difficult Questions

### 21. Explain the Java Memory Model and memory areas.
**Answer:** The Java Memory Model defines how threads interact through memory and what behaviors are allowed in concurrent execution.

**Memory Areas:**
1. **Heap Memory:**
   - **Young Generation:**
     - Eden Space: New objects
     - Survivor Spaces (S0, S1): Objects surviving first GC
   - **Old Generation (Tenured)**: Long-lived objects
   - **Metaspace**: Class metadata (Java 8+, replaced PermGen)

2. **Non-Heap Memory:**
   - **Method Area**: Class-level data, constant pool
   - **PC Registers**: Current executing instruction per thread
   - **Native Method Stacks**: Native method calls
   - **JVM Stacks**: Method call frames, local variables

**Memory Visibility:**
- Each thread has local memory (cache)
- Shared variables stored in main memory
- `volatile` ensures visibility across threads
- `synchronized` provides both atomicity and visibility

### 22. What are design patterns? Explain Singleton, Factory, and Observer patterns.
**Answer:** Design patterns are reusable solutions to common software design problems.

**1. Singleton Pattern:** Ensures only one instance of a class exists.
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**2. Factory Pattern:** Creates objects without specifying exact classes.
```java
interface Animal {
    void makeSound();
}

class AnimalFactory {
    public static Animal createAnimal(String type) {
        switch (type) {
            case "dog": return new Dog();
            case "cat": return new Cat();
            default: throw new IllegalArgumentException();
        }
    }
}
```

**3. Observer Pattern:** Defines one-to-many dependency between objects.
```java
interface Observer {
    void update(String message);
}

class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void addObserver(Observer observer) { observers.add(observer); }
    public void notifyObservers(String message) {
        observers.forEach(observer -> observer.update(message));
    }
}
```

### 23. What is reflection in Java and when is it used?
**Answer:** Reflection allows inspection and manipulation of classes, methods, and fields at runtime.

**Key Classes:**
- `Class<?>`: Represents a class
- `Method`: Represents a method
- `Field`: Represents a field
- `Constructor`: Represents a constructor

```java
Class<?> clazz = Class.forName("com.example.MyClass");
Object instance = clazz.getDeclaredConstructor().newInstance();

Method method = clazz.getDeclaredMethod("methodName", String.class);
method.setAccessible(true);  // Access private methods
Object result = method.invoke(instance, "parameter");

Field field = clazz.getDeclaredField("fieldName");
field.setAccessible(true);
field.set(instance, "new value");
```

**Use Cases:**
- Frameworks (Spring, Hibernate)
- Testing frameworks
- Serialization/Deserialization
- Dynamic proxy creation
- IDE features (code completion)

**Drawbacks:**
- Performance overhead
- Security restrictions
- Code maintainability issues

### 24. Explain Java 8 features: Lambda expressions, Streams, and Optional.
**Answer:**

**Lambda Expressions:** Anonymous functions for functional programming.
```java
// Before Java 8
Comparator<String> comparator = new Comparator<String>() {
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
};

// Java 8
Comparator<String> comparator = (a, b) -> a.compareTo(b);
```

**Streams API:** Functional-style operations on collections.
```java
List<String> names = Arrays.asList("John", "Jane", "Jack");
List<String> result = names.stream()
    .filter(name -> name.startsWith("J"))
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

**Optional:** Container object to avoid null pointer exceptions.
```java
Optional<String> optional = Optional.ofNullable(getString());
optional.ifPresent(System.out::println);
String value = optional.orElse("default");
String value2 = optional.orElseThrow(() -> new RuntimeException("Not found"));
```

### 25. What is the difference between fail-fast and fail-safe iterators?
**Answer:**

**Fail-Fast Iterators:**
- Throw `ConcurrentModificationException` if collection is modified during iteration
- Used by: ArrayList, HashMap, HashSet
- Check `modCount` against expected count

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    list.add("D");  // Throws ConcurrentModificationException
}
```

**Fail-Safe Iterators:**
- Work on copy of collection, no exception thrown
- Used by: ConcurrentHashMap, CopyOnWriteArrayList
- May not reflect recent modifications

```java
ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
map.put("1", "A");
for (String key : map.keySet()) {
    map.put("2", "B");  // No exception, but may not be visible in current iteration
}
```

### 26. Explain different types of ClassLoaders in Java.
**Answer:** ClassLoaders are responsible for loading Java classes into JVM at runtime.

**Hierarchy:**
1. **Bootstrap ClassLoader:**
   - Written in native code (C/C++)
   - Loads core Java classes from `rt.jar`
   - Parent of all other classloaders

2. **Extension ClassLoader:**
   - Loads classes from `jre/lib/ext` directory
   - Child of Bootstrap ClassLoader

3. **Application/System ClassLoader:**
   - Loads classes from CLASSPATH
   - Child of Extension ClassLoader

4. **Custom ClassLoaders:**
   - User-defined classloaders
   - Can load classes from databases, networks, etc.

**Delegation Model:**
- Child first checks with parent
- If parent can't load, child attempts to load
- Prevents core classes from being overridden

```java
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        return defineClass(name, classData, 0, classData.length);
    }
}
```

### 27. What is the difference between HashMap and ConcurrentHashMap?
**Answer:**

| HashMap | ConcurrentHashMap |
|---------|-------------------|
| Not thread-safe | Thread-safe |
| Allows null keys/values | No null keys/values |
| Fail-fast iterator | Fail-safe iterator |
| Single lock (when synchronized) | Segment-based locking (Java 7), Node-based locking (Java 8+) |
| Better performance in single-threaded | Better performance in multi-threaded |

**ConcurrentHashMap Implementation:**
- **Java 7**: Segment-based locking (16 segments by default)
- **Java 8+**: Node-based locking with CAS operations

```java
// HashMap - not thread-safe
Map<String, String> hashMap = new HashMap<>();

// ConcurrentHashMap - thread-safe
Map<String, String> concurrentMap = new ConcurrentHashMap<>();

// Safe concurrent operations
concurrentMap.putIfAbsent("key", "value");
concurrentMap.compute("key", (k, v) -> v == null ? "new" : v + "_updated");
```

### 28. Explain JVM architecture and compilation process.
**Answer:**

**JVM Architecture:**
1. **Class Loader Subsystem:**
   - Loading: Finds and loads bytecode
   - Linking: Verification, Preparation, Resolution
   - Initialization: Static variable initialization

2. **Runtime Data Areas:**
   - Method Area: Class metadata, constant pool
   - Heap: Object instances
   - Stack: Method frames, local variables
   - PC Registers: Current instruction pointers
   - Native Method Stacks: Native method calls

3. **Execution Engine:**
   - Interpreter: Executes bytecode line by line
   - JIT Compiler: Compiles frequently used code to native
   - Garbage Collector: Manages memory

**Compilation Process:**
1. **Source Code (.java)** → **Java Compiler (javac)** → **Bytecode (.class)**
2. **Bytecode** → **JVM** → **Interpreter/JIT** → **Native Machine Code**

**JIT Compilation:**
- Profiles code execution
- Compiles hot spots to native code
- Optimizations: method inlining, dead code elimination, loop optimization

### 29. What are annotations in Java and how do you create custom annotations?
**Answer:** Annotations provide metadata about code without affecting its execution.

**Built-in Annotations:**
- `@Override`: Method overriding
- `@Deprecated`: Marks deprecated elements
- `@SuppressWarnings`: Suppresses compiler warnings
- `@FunctionalInterface`: Marks functional interfaces

**Meta-Annotations:**
- `@Retention`: When annotation is available (SOURCE, CLASS, RUNTIME)
- `@Target`: Where annotation can be applied (TYPE, METHOD, FIELD, etc.)
- `@Inherited`: Annotation is inherited by subclasses
- `@Documented`: Include in JavaDoc

**Creating Custom Annotations:**
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface MyAnnotation {
    String value() default "";
    int priority() default 0;
    String[] tags() default {};
}

// Usage
@MyAnnotation(value = "test", priority = 1, tags = {"important", "review"})
public void myMethod() {
    // method implementation
}

// Processing annotations
Method method = MyClass.class.getMethod("myMethod");
if (method.isAnnotationPresent(MyAnnotation.class)) {
    MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
    System.out.println(annotation.value());
}
```

### 30. Explain microservices architecture and how Java supports it.
**Answer:** Microservices architecture decomposes applications into small, independent services that communicate over well-defined APIs.

**Characteristics:**
- **Decentralized**: Independent deployment and scaling
- **Technology Agnostic**: Each service can use different tech stack
- **Fault Isolation**: Failure in one service doesn't affect others
- **Business Capability Focused**: Services organized around business functions

**Java Support for Microservices:**

1. **Spring Boot:**
```java
@SpringBootApplication
@RestController
public class UserService {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userRepository.findById(id);
    }
}
```

2. **Spring Cloud:**
   - **Service Discovery**: Eureka, Consul
   - **Load Balancing**: Ribbon
   - **Circuit Breaker**: Hystrix
   - **API Gateway**: Zuul, Spring Cloud Gateway

3. **Communication Patterns:**
   - **Synchronous**: REST APIs, gRPC
   - **Asynchronous**: Message queues (RabbitMQ, Kafka)

4. **Containerization:**
```dockerfile
FROM openjdk:11-jre-slim
COPY target/user-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

**Benefits:**
- Independent scaling and deployment
- Technology diversity
- Fault isolation
- Team autonomy

**Challenges:**
- Distributed system complexity
- Network latency
- Data consistency
- Service coordination 