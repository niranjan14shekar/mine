# Java Interview - Medium Coding Questions

## Applied Concept-Based Programming Problems

### 1. Multi-threaded Producer-Consumer System
**Problem:** Implement a thread-safe producer-consumer system using BlockingQueue.

**Requirements:**
- Multiple producers and consumers
- Buffer with limited capacity
- Graceful shutdown mechanism

**Solution Framework:**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicBoolean;

class Producer implements Runnable {
    private final BlockingQueue<Integer> queue;
    private final AtomicBoolean running;
    private final String name;
    
    public Producer(BlockingQueue<Integer> queue, AtomicBoolean running, String name) {
        this.queue = queue;
        this.running = running;
        this.name = name;
    }
    
    @Override
    public void run() {
        int item = 0;
        while (running.get()) {
            try {
                queue.put(item++);
                System.out.println(name + " produced: " + (item-1));
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}

class Consumer implements Runnable {
    private final BlockingQueue<Integer> queue;
    private final AtomicBoolean running;
    private final String name;
    
    public Consumer(BlockingQueue<Integer> queue, AtomicBoolean running, String name) {
        this.queue = queue;
        this.running = running;
        this.name = name;
    }
    
    @Override
    public void run() {
        while (running.get() || !queue.isEmpty()) {
            try {
                Integer item = queue.poll(200, TimeUnit.MILLISECONDS);
                if (item != null) {
                    System.out.println(name + " consumed: " + item);
                    Thread.sleep(150);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}

public class ProducerConsumerDemo {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
        AtomicBoolean running = new AtomicBoolean(true);
        
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Start producers and consumers
        executor.submit(new Producer(queue, running, "Producer-1"));
        executor.submit(new Producer(queue, running, "Producer-2"));
        executor.submit(new Consumer(queue, running, "Consumer-1"));
        executor.submit(new Consumer(queue, running, "Consumer-2"));
        
        Thread.sleep(5000);
        running.set(false);
        
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

**Concepts Tested:** Multithreading, BlockingQueue, ExecutorService, AtomicBoolean

---

### 2. Custom HashMap Implementation
**Problem:** Implement a simplified HashMap with collision handling.

**Requirements:**
- Hash function with collision resolution
- Dynamic resizing
- Basic put/get/remove operations

**Solution Framework:**
```java
public class CustomHashMap<K, V> {
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private Node<K, V>[] buckets;
    private int size;
    private int capacity;
    private static final double LOAD_FACTOR = 0.75;
    
    @SuppressWarnings("unchecked")
    public CustomHashMap() {
        this.capacity = 16;
        this.buckets = new Node[capacity];
        this.size = 0;
    }
    
    private int hash(K key) {
        return key == null ? 0 : Math.abs(key.hashCode() % capacity);
    }
    
    public V put(K key, V value) {
        int index = hash(key);
        Node<K, V> head = buckets[index];
        
        // Check if key exists
        Node<K, V> current = head;
        while (current != null) {
            if (Objects.equals(current.key, key)) {
                V oldValue = current.value;
                current.value = value;
                return oldValue;
            }
            current = current.next;
        }
        
        // Add new node
        Node<K, V> newNode = new Node<>(key, value);
        newNode.next = head;
        buckets[index] = newNode;
        size++;
        
        // Resize if needed
        if (size >= capacity * LOAD_FACTOR) {
            resize();
        }
        
        return null;
    }
    
    public V get(K key) {
        int index = hash(key);
        Node<K, V> current = buckets[index];
        
        while (current != null) {
            if (Objects.equals(current.key, key)) {
                return current.value;
            }
            current = current.next;
        }
        return null;
    }
    
    @SuppressWarnings("unchecked")
    private void resize() {
        Node<K, V>[] oldBuckets = buckets;
        capacity *= 2;
        buckets = new Node[capacity];
        size = 0;
        
        for (Node<K, V> head : oldBuckets) {
            Node<K, V> current = head;
            while (current != null) {
                put(current.key, current.value);
                current = current.next;
            }
        }
    }
}
```

**Concepts Tested:** Data structures, Hash functions, Generics, Dynamic arrays

---

### 3. File Processing with Streams and Lambda
**Problem:** Process large CSV files efficiently using Java 8+ features.

**Requirements:**
- Parse CSV data
- Filter and transform records
- Aggregate calculations
- Error handling for malformed data

**Solution Framework:**
```java
import java.io.*;
import java.nio.file.*;
import java.util.*;
import java.util.stream.*;

class Employee {
    private String id, name, department;
    private double salary;
    private LocalDate hireDate;
    
    // Constructor, getters, setters
    public static Employee fromCsvLine(String line) {
        String[] parts = line.split(",");
        if (parts.length != 5) throw new IllegalArgumentException("Invalid CSV line");
        
        Employee emp = new Employee();
        emp.id = parts[0].trim();
        emp.name = parts[1].trim();
        emp.department = parts[2].trim();
        emp.salary = Double.parseDouble(parts[3].trim());
        emp.hireDate = LocalDate.parse(parts[4].trim());
        return emp;
    }
}

public class EmployeeProcessor {
    public static void processEmployeeFile(String filePath) {
        try (Stream<String> lines = Files.lines(Paths.get(filePath))) {
            Map<String, Double> departmentAvgSalary = lines
                .skip(1) // Skip header
                .map(line -> {
                    try {
                        return Employee.fromCsvLine(line);
                    } catch (Exception e) {
                        System.err.println("Error parsing line: " + line);
                        return null;
                    }
                })
                .filter(Objects::nonNull)
                .filter(emp -> emp.getSalary() > 0)
                .collect(Collectors.groupingBy(
                    Employee::getDepartment,
                    Collectors.averagingDouble(Employee::getSalary)
                ));
            
            departmentAvgSalary.entrySet().stream()
                .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
                .forEach(entry -> 
                    System.out.printf("%s: $%.2f%n", entry.getKey(), entry.getValue()));
                    
        } catch (IOException e) {
            System.err.println("Error reading file: " + e.getMessage());
        }
    }
    
    public static List<Employee> findTopPerformers(String filePath, int topN) {
        try (Stream<String> lines = Files.lines(Paths.get(filePath))) {
            return lines.skip(1)
                .map(Employee::fromCsvLine)
                .filter(emp -> emp.getHireDate().isBefore(LocalDate.now().minusYears(2)))
                .sorted(Comparator.comparing(Employee::getSalary).reversed())
                .limit(topN)
                .collect(Collectors.toList());
        } catch (IOException e) {
            return Collections.emptyList();
        }
    }
}
```

**Concepts Tested:** Stream API, Lambda expressions, File I/O, Error handling

---

### 4. Observer Pattern with Annotations
**Problem:** Implement event system with annotation-based listeners.

**Requirements:**
- Custom annotations for event handling
- Reflection-based event dispatching
- Multiple event types
- Asynchronous processing

**Solution Framework:**
```java
import java.lang.annotation.*;
import java.lang.reflect.*;
import java.util.*;
import java.util.concurrent.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface EventHandler {
    Class<? extends Event> value();
    boolean async() default false;
}

abstract class Event {
    private final long timestamp = System.currentTimeMillis();
    public long getTimestamp() { return timestamp; }
}

class UserLoginEvent extends Event {
    private final String username;
    public UserLoginEvent(String username) { this.username = username; }
    public String getUsername() { return username; }
}

class OrderCreatedEvent extends Event {
    private final String orderId;
    private final double amount;
    public OrderCreatedEvent(String orderId, double amount) {
        this.orderId = orderId;
        this.amount = amount;
    }
    // Getters...
}

class EventBus {
    private final Map<Class<? extends Event>, List<Method>> handlers = new HashMap<>();
    private final Map<Method, Object> instances = new HashMap<>();
    private final ExecutorService executor = Executors.newCachedThreadPool();
    
    public void register(Object listener) {
        Class<?> clazz = listener.getClass();
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(EventHandler.class)) {
                EventHandler annotation = method.getAnnotation(EventHandler.class);
                Class<? extends Event> eventType = annotation.value();
                
                handlers.computeIfAbsent(eventType, k -> new ArrayList<>()).add(method);
                instances.put(method, listener);
                method.setAccessible(true);
            }
        }
    }
    
    public void publish(Event event) {
        List<Method> eventHandlers = handlers.get(event.getClass());
        if (eventHandlers != null) {
            for (Method handler : eventHandlers) {
                EventHandler annotation = handler.getAnnotation(EventHandler.class);
                Object instance = instances.get(handler);
                
                if (annotation.async()) {
                    executor.submit(() -> invokeHandler(handler, instance, event));
                } else {
                    invokeHandler(handler, instance, event);
                }
            }
        }
    }
    
    private void invokeHandler(Method handler, Object instance, Event event) {
        try {
            handler.invoke(instance, event);
        } catch (Exception e) {
            System.err.println("Error invoking handler: " + e.getMessage());
        }
    }
}

// Usage example
class UserService {
    @EventHandler(UserLoginEvent.class)
    public void onUserLogin(UserLoginEvent event) {
        System.out.println("User logged in: " + event.getUsername());
    }
    
    @EventHandler(value = OrderCreatedEvent.class, async = true)
    public void onOrderCreated(OrderCreatedEvent event) {
        System.out.println("Processing order asynchronously: " + event.getOrderId());
    }
}
```

**Concepts Tested:** Annotations, Reflection, Observer pattern, Asynchronous processing

---

### 5. REST API Client with Retry Logic
**Problem:** Build robust HTTP client with retry mechanism and circuit breaker.

**Requirements:**
- Exponential backoff retry
- Circuit breaker pattern
- JSON serialization/deserialization
- Connection pooling

**Solution Framework:**
```java
import java.net.http.*;
import java.time.*;
import java.util.concurrent.*;
import com.fasterxml.jackson.databind.ObjectMapper;

class CircuitBreaker {
    private enum State { CLOSED, OPEN, HALF_OPEN }
    
    private State state = State.CLOSED;
    private int failureCount = 0;
    private Instant lastFailureTime = Instant.now();
    private final int failureThreshold;
    private final Duration timeout;
    
    public CircuitBreaker(int failureThreshold, Duration timeout) {
        this.failureThreshold = failureThreshold;
        this.timeout = timeout;
    }
    
    public boolean canExecute() {
        if (state == State.CLOSED) return true;
        if (state == State.OPEN && Instant.now().isAfter(lastFailureTime.plus(timeout))) {
            state = State.HALF_OPEN;
            return true;
        }
        return false;
    }
    
    public void onSuccess() {
        failureCount = 0;
        state = State.CLOSED;
    }
    
    public void onFailure() {
        failureCount++;
        lastFailureTime = Instant.now();
        if (failureCount >= failureThreshold) {
            state = State.OPEN;
        }
    }
}

class RestClient {
    private final HttpClient httpClient;
    private final ObjectMapper objectMapper;
    private final CircuitBreaker circuitBreaker;
    
    public RestClient() {
        this.httpClient = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(30))
            .build();
        this.objectMapper = new ObjectMapper();
        this.circuitBreaker = new CircuitBreaker(3, Duration.ofMinutes(1));
    }
    
    public <T> CompletableFuture<T> get(String url, Class<T> responseType) {
        return executeWithRetry(() -> {
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .GET()
                .build();
                
            return httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                .thenApply(response -> {
                    if (response.statusCode() >= 200 && response.statusCode() < 300) {
                        try {
                            return objectMapper.readValue(response.body(), responseType);
                        } catch (Exception e) {
                            throw new RuntimeException("JSON parsing failed", e);
                        }
                    } else {
                        throw new RuntimeException("HTTP " + response.statusCode());
                    }
                });
        }, 3);
    }
    
    private <T> CompletableFuture<T> executeWithRetry(
            Supplier<CompletableFuture<T>> operation, int maxRetries) {
        
        if (!circuitBreaker.canExecute()) {
            return CompletableFuture.failedFuture(
                new RuntimeException("Circuit breaker is open"));
        }
        
        return retryWithBackoff(operation, maxRetries, 0);
    }
    
    private <T> CompletableFuture<T> retryWithBackoff(
            Supplier<CompletableFuture<T>> operation, int maxRetries, int attempt) {
        
        return operation.get()
            .handle((result, throwable) -> {
                if (throwable == null) {
                    circuitBreaker.onSuccess();
                    return CompletableFuture.completedFuture(result);
                } else {
                    circuitBreaker.onFailure();
                    if (attempt < maxRetries) {
                        long delay = (long) Math.pow(2, attempt) * 1000; // Exponential backoff
                        return CompletableFuture.delayedExecutor(delay, TimeUnit.MILLISECONDS)
                            .execute(() -> retryWithBackoff(operation, maxRetries, attempt + 1));
                    } else {
                        return CompletableFuture.<T>failedFuture(throwable);
                    }
                }
            })
            .thenCompose(Function.identity());
    }
}
```

**Concepts Tested:** HTTP clients, CompletableFuture, Circuit breaker pattern, Retry mechanisms 