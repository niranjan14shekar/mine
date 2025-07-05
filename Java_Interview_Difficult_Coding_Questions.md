# Java Interview - Difficult Coding Questions

## Advanced Applied Concept-Based Programming Problems

### 1. Distributed Cache Implementation
**Problem:** Build a distributed cache system with consistency and partitioning.

**Requirements:**
- Consistent hashing for data distribution
- TTL support with automatic expiration
- Write-through and write-back strategies
- Network communication between nodes

**Solution Framework:**
```java
import java.util.concurrent.*;
import java.security.MessageDigest;
import java.util.*;

class ConsistentHash<T> {
    private final TreeMap<Long, T> ring = new TreeMap<>();
    private final int virtualNodes;
    
    public ConsistentHash(int virtualNodes) {
        this.virtualNodes = virtualNodes;
    }
    
    public void addNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            ring.put(hash(node.toString() + i), node);
        }
    }
    
    public T getNode(String key) {
        if (ring.isEmpty()) return null;
        Long hash = hash(key);
        Map.Entry<Long, T> entry = ring.ceilingEntry(hash);
        return entry != null ? entry.getValue() : ring.firstEntry().getValue();
    }
    
    private Long hash(String key) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(key.getBytes());
            long hash = 0;
            for (int i = 0; i < 8; i++) {
                hash <<= 8;
                hash |= ((int) digest[i]) & 0xFF;
            }
            return hash;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

class CacheEntry<V> {
    private final V value;
    private final long expirationTime;
    private volatile boolean dirty;
    
    public CacheEntry(V value, long ttlMs) {
        this.value = value;
        this.expirationTime = System.currentTimeMillis() + ttlMs;
        this.dirty = false;
    }
    
    public boolean isExpired() {
        return System.currentTimeMillis() > expirationTime;
    }
    
    // Getters and setters...
}

class DistributedCache<K, V> {
    private final ConcurrentHashMap<K, CacheEntry<V>> localCache = new ConcurrentHashMap<>();
    private final ConsistentHash<String> hashRing = new ConsistentHash<>(150);
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
    private final String nodeId;
    
    public DistributedCache(String nodeId, List<String> nodes) {
        this.nodeId = nodeId;
        nodes.forEach(hashRing::addNode);
        
        // Start TTL cleanup
        scheduler.scheduleAtFixedRate(this::cleanupExpired, 60, 60, TimeUnit.SECONDS);
    }
    
    public V get(K key) {
        String targetNode = hashRing.getNode(key.toString());
        
        if (nodeId.equals(targetNode)) {
            CacheEntry<V> entry = localCache.get(key);
            return (entry != null && !entry.isExpired()) ? entry.getValue() : null;
        } else {
            // Remote call to target node
            return getFromRemoteNode(targetNode, key);
        }
    }
    
    public void put(K key, V value, long ttlMs) {
        String targetNode = hashRing.getNode(key.toString());
        
        if (nodeId.equals(targetNode)) {
            localCache.put(key, new CacheEntry<>(value, ttlMs));
        } else {
            // Remote call to target node
            putToRemoteNode(targetNode, key, value, ttlMs);
        }
    }
    
    private void cleanupExpired() {
        localCache.entrySet().removeIf(entry -> entry.getValue().isExpired());
    }
    
    // Simulate network calls (implement with actual HTTP/RPC)
    private V getFromRemoteNode(String node, K key) { return null; }
    private void putToRemoteNode(String node, K key, V value, long ttl) {}
}
```

**Concepts Tested:** Distributed systems, Consistent hashing, Concurrent programming, Cache strategies

---

### 2. Custom Thread Pool with Work Stealing
**Problem:** Implement a work-stealing thread pool executor.

**Requirements:**
- Work-stealing deques for load balancing
- Dynamic thread management
- Task submission and completion tracking
- Graceful shutdown with timeout

**Solution Framework:**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.*;

class WorkStealingDeque<T> {
    private volatile T[] array;
    private volatile int top = 0;
    private volatile int bottom = 0;
    private final AtomicInteger capacity;
    
    @SuppressWarnings("unchecked")
    public WorkStealingDeque(int initialCapacity) {
        this.array = (T[]) new Object[initialCapacity];
        this.capacity = new AtomicInteger(initialCapacity);
    }
    
    public void pushBottom(T task) {
        int b = bottom;
        int t = top;
        T[] a = array;
        
        if (b - t >= a.length - 1) {
            resize();
            a = array;
        }
        
        a[b % a.length] = task;
        bottom = b + 1;
    }
    
    public T popBottom() {
        int b = bottom - 1;
        T[] a = array;
        bottom = b;
        
        int t = top;
        if (b < t) {
            bottom = t;
            return null;
        }
        
        T task = a[b % a.length];
        if (b > t) {
            return task;
        }
        
        // Compete with steal
        if (!compareAndSwapTop(t, t + 1)) {
            task = null;
        }
        bottom = t + 1;
        return task;
    }
    
    public T steal() {
        int t = top;
        int b = bottom;
        
        if (t >= b) return null;
        
        T[] a = array;
        T task = a[t % a.length];
        
        if (!compareAndSwapTop(t, t + 1)) {
            return null;
        }
        
        return task;
    }
    
    private synchronized void resize() {
        T[] oldArray = array;
        int oldCapacity = oldArray.length;
        int newCapacity = oldCapacity * 2;
        
        @SuppressWarnings("unchecked")
        T[] newArray = (T[]) new Object[newCapacity];
        
        for (int i = top; i < bottom; i++) {
            newArray[i % newCapacity] = oldArray[i % oldCapacity];
        }
        
        array = newArray;
        capacity.set(newCapacity);
    }
    
    private boolean compareAndSwapTop(int expected, int update) {
        // Simplified CAS operation
        synchronized (this) {
            if (top == expected) {
                top = update;
                return true;
            }
            return false;
        }
    }
}

class WorkStealingThread extends Thread {
    private final WorkStealingDeque<Runnable> deque;
    private final List<WorkStealingThread> allThreads;
    private final AtomicBoolean shutdown;
    private final Random random = new Random();
    
    public WorkStealingThread(String name, List<WorkStealingThread> allThreads, AtomicBoolean shutdown) {
        super(name);
        this.deque = new WorkStealingDeque<>(1024);
        this.allThreads = allThreads;
        this.shutdown = shutdown;
    }
    
    public void submitTask(Runnable task) {
        deque.pushBottom(task);
    }
    
    @Override
    public void run() {
        while (!shutdown.get() || !isEmpty()) {
            Runnable task = getTask();
            if (task != null) {
                try {
                    task.run();
                } catch (Exception e) {
                    System.err.println("Task execution failed: " + e.getMessage());
                }
            } else {
                Thread.yield();
            }
        }
    }
    
    private Runnable getTask() {
        // Try local deque first
        Runnable task = deque.popBottom();
        if (task != null) return task;
        
        // Try to steal from other threads
        for (int i = 0; i < allThreads.size() * 2; i++) {
            WorkStealingThread victim = allThreads.get(random.nextInt(allThreads.size()));
            if (victim != this) {
                task = victim.deque.steal();
                if (task != null) return task;
            }
        }
        
        return null;
    }
    
    private boolean isEmpty() {
        return deque.popBottom() == null;
    }
}

public class WorkStealingExecutor implements ExecutorService {
    private final List<WorkStealingThread> workers;
    private final AtomicBoolean shutdown = new AtomicBoolean(false);
    private final AtomicInteger taskCounter = new AtomicInteger(0);
    
    public WorkStealingExecutor(int parallelism) {
        workers = new ArrayList<>(parallelism);
        
        for (int i = 0; i < parallelism; i++) {
            WorkStealingThread worker = new WorkStealingThread(
                "WorkStealer-" + i, workers, shutdown);
            workers.add(worker);
            worker.start();
        }
    }
    
    @Override
    public void execute(Runnable command) {
        if (shutdown.get()) {
            throw new RejectedExecutionException("Executor is shutdown");
        }
        
        // Submit to least loaded thread (simplified strategy)
        int index = taskCounter.getAndIncrement() % workers.size();
        workers.get(index).submitTask(command);
    }
    
    @Override
    public void shutdown() {
        shutdown.set(true);
    }
    
    @Override
    public boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException {
        long deadline = System.currentTimeMillis() + unit.toMillis(timeout);
        
        for (WorkStealingThread worker : workers) {
            long remaining = deadline - System.currentTimeMillis();
            if (remaining <= 0) return false;
            worker.join(remaining);
        }
        
        return workers.stream().noneMatch(Thread::isAlive);
    }
    
    // Other ExecutorService methods...
    @Override public List<Runnable> shutdownNow() { return Collections.emptyList(); }
    @Override public boolean isShutdown() { return shutdown.get(); }
    @Override public boolean isTerminated() { return isShutdown() && workers.stream().noneMatch(Thread::isAlive); }
    // ... submit methods
}
```

**Concepts Tested:** Advanced concurrency, Lock-free data structures, Work-stealing algorithms

---

### 3. Custom Serialization Framework
**Problem:** Build a reflection-based serialization framework with versioning.

**Requirements:**
- Schema evolution support
- Custom serializers for complex types
- Binary format with compression
- Circular reference detection

**Solution Framework:**
```java
import java.lang.annotation.*;
import java.lang.reflect.*;
import java.io.*;
import java.util.*;
import java.util.zip.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface SerialField {
    String name() default "";
    int version() default 1;
    boolean ignore() default false;
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Serializable {
    int version() default 1;
}

interface CustomSerializer<T> {
    void serialize(T object, DataOutputStream out) throws IOException;
    T deserialize(DataInputStream in) throws IOException;
}

class SerializationContext {
    private final Map<Object, Integer> objectIds = new IdentityHashMap<>();
    private final List<Object> objects = new ArrayList<>();
    private int nextId = 0;
    
    public int getObjectId(Object obj) {
        return objectIds.computeIfAbsent(obj, k -> {
            objects.add(obj);
            return nextId++;
        });
    }
    
    public Object getObject(int id) {
        return objects.get(id);
    }
    
    public boolean isFirstOccurrence(Object obj) {
        return !objectIds.containsKey(obj);
    }
}

public class SerializationFramework {
    private final Map<Class<?>, CustomSerializer<?>> serializers = new HashMap<>();
    
    public <T> void registerSerializer(Class<T> type, CustomSerializer<T> serializer) {
        serializers.put(type, serializer);
    }
    
    public byte[] serialize(Object obj) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (GZIPOutputStream gzos = new GZIPOutputStream(baos);
             DataOutputStream dos = new DataOutputStream(gzos)) {
            
            SerializationContext context = new SerializationContext();
            writeObject(obj, dos, context);
        }
        return baos.toByteArray();
    }
    
    @SuppressWarnings("unchecked")
    private void writeObject(Object obj, DataOutputStream out, SerializationContext context) 
            throws IOException {
        
        if (obj == null) {
            out.writeByte(0); // NULL marker
            return;
        }
        
        Class<?> clazz = obj.getClass();
        
        // Handle circular references
        if (!context.isFirstOccurrence(obj)) {
            out.writeByte(2); // REFERENCE marker
            out.writeInt(context.getObjectId(obj));
            return;
        }
        
        int objectId = context.getObjectId(obj);
        out.writeByte(1); // OBJECT marker
        out.writeInt(objectId);
        out.writeUTF(clazz.getName());
        
        // Use custom serializer if available
        CustomSerializer<Object> customSerializer = 
            (CustomSerializer<Object>) serializers.get(clazz);
        
        if (customSerializer != null) {
            out.writeBoolean(true); // Has custom serializer
            customSerializer.serialize(obj, out);
            return;
        }
        
        out.writeBoolean(false); // No custom serializer
        
        // Reflection-based serialization
        Serializable serializableAnnotation = clazz.getAnnotation(Serializable.class);
        int classVersion = serializableAnnotation != null ? 
            serializableAnnotation.version() : 1;
        out.writeInt(classVersion);
        
        Field[] fields = getAllFields(clazz);
        List<Field> serializableFields = new ArrayList<>();
        
        for (Field field : fields) {
            SerialField annotation = field.getAnnotation(SerialField.class);
            if (annotation == null || !annotation.ignore()) {
                serializableFields.add(field);
            }
        }
        
        out.writeInt(serializableFields.size());
        
        for (Field field : serializableFields) {
            field.setAccessible(true);
            SerialField annotation = field.getAnnotation(SerialField.class);
            
            String fieldName = annotation != null && !annotation.name().isEmpty() ? 
                annotation.name() : field.getName();
            int fieldVersion = annotation != null ? annotation.version() : 1;
            
            out.writeUTF(fieldName);
            out.writeInt(fieldVersion);
            
            try {
                Object fieldValue = field.get(obj);
                writeObject(fieldValue, out, context);
            } catch (IllegalAccessException e) {
                throw new IOException("Cannot access field: " + field.getName(), e);
            }
        }
    }
    
    public Object deserialize(byte[] data) throws IOException, ClassNotFoundException {
        ByteArrayInputStream bais = new ByteArrayInputStream(data);
        try (GZIPInputStream gzis = new GZIPInputStream(bais);
             DataInputStream dis = new DataInputStream(gzis)) {
            
            SerializationContext context = new SerializationContext();
            return readObject(dis, context);
        }
    }
    
    @SuppressWarnings("unchecked")
    private Object readObject(DataInputStream in, SerializationContext context) 
            throws IOException, ClassNotFoundException {
        
        byte marker = in.readByte();
        
        switch (marker) {
            case 0: // NULL
                return null;
                
            case 2: // REFERENCE
                int refId = in.readInt();
                return context.getObject(refId);
                
            case 1: // OBJECT
                int objectId = in.readInt();
                String className = in.readUTF();
                Class<?> clazz = Class.forName(className);
                
                boolean hasCustomSerializer = in.readBoolean();
                
                if (hasCustomSerializer) {
                    CustomSerializer<Object> serializer = 
                        (CustomSerializer<Object>) serializers.get(clazz);
                    if (serializer == null) {
                        throw new IOException("No serializer found for: " + className);
                    }
                    Object obj = serializer.deserialize(in);
                    context.objects.add(objectId, obj);
                    return obj;
                }
                
                // Reflection-based deserialization
                int classVersion = in.readInt();
                
                try {
                    Object obj = clazz.getDeclaredConstructor().newInstance();
                    context.objects.add(objectId, obj);
                    
                    int fieldCount = in.readInt();
                    Map<String, Field> fieldMap = getFieldMap(clazz);
                    
                    for (int i = 0; i < fieldCount; i++) {
                        String fieldName = in.readUTF();
                        int fieldVersion = in.readInt();
                        Object fieldValue = readObject(in, context);
                        
                        Field field = fieldMap.get(fieldName);
                        if (field != null) {
                            field.setAccessible(true);
                            field.set(obj, fieldValue);
                        }
                        // Skip unknown fields for forward compatibility
                    }
                    
                    return obj;
                    
                } catch (Exception e) {
                    throw new IOException("Cannot deserialize object", e);
                }
                
            default:
                throw new IOException("Unknown marker: " + marker);
        }
    }
    
    private Field[] getAllFields(Class<?> clazz) {
        List<Field> fields = new ArrayList<>();
        while (clazz != null) {
            fields.addAll(Arrays.asList(clazz.getDeclaredFields()));
            clazz = clazz.getSuperclass();
        }
        return fields.toArray(new Field[0]);
    }
    
    private Map<String, Field> getFieldMap(Class<?> clazz) {
        Map<String, Field> fieldMap = new HashMap<>();
        for (Field field : getAllFields(clazz)) {
            SerialField annotation = field.getAnnotation(SerialField.class);
            String fieldName = annotation != null && !annotation.name().isEmpty() ? 
                annotation.name() : field.getName();
            fieldMap.put(fieldName, field);
        }
        return fieldMap;
    }
}
```

**Concepts Tested:** Reflection, Annotations, Serialization, Binary protocols, Schema evolution

---

### 4. Reactive Stream Processor
**Problem:** Implement a reactive streams processor with backpressure.

**Requirements:**
- Publisher/Subscriber pattern
- Backpressure handling
- Error propagation
- Hot and cold streams

**Solution Framework:**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.function.*;

interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}

interface Subscriber<T> {
    void onSubscribe(Subscription subscription);
    void onNext(T item);
    void onError(Throwable throwable);
    void onComplete();
}

interface Subscription {
    void request(long n);
    void cancel();
}

interface Processor<T, R> extends Publisher<R>, Subscriber<T> {
}

class FlowableProcessor<T, R> implements Processor<T, R> {
    private final Function<T, R> mapper;
    private final Predicate<T> filter;
    private final AtomicReference<Subscriber<? super R>> subscriber = new AtomicReference<>();
    private final AtomicReference<Subscription> upstream = new AtomicReference<>();
    private final AtomicLong requested = new AtomicLong();
    private final AtomicBoolean cancelled = new AtomicBoolean();
    
    public FlowableProcessor(Function<T, R> mapper, Predicate<T> filter) {
        this.mapper = mapper;
        this.filter = filter;
    }
    
    @Override
    public void subscribe(Subscriber<? super R> s) {
        if (subscriber.compareAndSet(null, s)) {
            s.onSubscribe(new Subscription() {
                @Override
                public void request(long n) {
                    if (n <= 0) {
                        s.onError(new IllegalArgumentException("Request must be positive"));
                        return;
                    }
                    
                    long current = requested.get();
                    long updated = current + n;
                    if (updated < 0) updated = Long.MAX_VALUE; // Overflow protection
                    
                    if (requested.compareAndSet(current, updated)) {
                        Subscription up = upstream.get();
                        if (up != null) {
                            up.request(n);
                        }
                    }
                }
                
                @Override
                public void cancel() {
                    cancelled.set(true);
                    Subscription up = upstream.get();
                    if (up != null) {
                        up.cancel();
                    }
                }
            });
        } else {
            s.onError(new IllegalStateException("Only one subscriber allowed"));
        }
    }
    
    @Override
    public void onSubscribe(Subscription subscription) {
        if (upstream.compareAndSet(null, subscription)) {
            long n = requested.get();
            if (n > 0) {
                subscription.request(n);
            }
        } else {
            subscription.cancel();
        }
    }
    
    @Override
    public void onNext(T item) {
        if (cancelled.get()) return;
        
        Subscriber<? super R> s = subscriber.get();
        if (s == null) return;
        
        try {
            if (filter.test(item)) {
                R mapped = mapper.apply(item);
                
                long current = requested.get();
                while (current > 0) {
                    if (requested.compareAndSet(current, current - 1)) {
                        s.onNext(mapped);
                        break;
                    }
                    current = requested.get();
                }
            }
        } catch (Exception e) {
            onError(e);
        }
    }
    
    @Override
    public void onError(Throwable throwable) {
        if (cancelled.get()) return;
        
        Subscriber<? super R> s = subscriber.get();
        if (s != null) {
            s.onError(throwable);
        }
    }
    
    @Override
    public void onComplete() {
        if (cancelled.get()) return;
        
        Subscriber<? super R> s = subscriber.get();
        if (s != null) {
            s.onComplete();
        }
    }
}

class ReactiveStream<T> implements Publisher<T> {
    private final List<T> items;
    private final boolean hot;
    
    public ReactiveStream(List<T> items, boolean hot) {
        this.items = new ArrayList<>(items);
        this.hot = hot;
    }
    
    @Override
    public void subscribe(Subscriber<? super T> subscriber) {
        if (hot) {
            // Hot stream - emit items immediately
            subscribeHot(subscriber);
        } else {
            // Cold stream - emit items on demand
            subscribeCold(subscriber);
        }
    }
    
    private void subscribeCold(Subscriber<? super T> subscriber) {
        subscriber.onSubscribe(new Subscription() {
            private final AtomicInteger index = new AtomicInteger(0);
            private final AtomicBoolean cancelled = new AtomicBoolean(false);
            
            @Override
            public void request(long n) {
                if (cancelled.get()) return;
                
                for (long i = 0; i < n && !cancelled.get(); i++) {
                    int idx = index.getAndIncrement();
                    if (idx < items.size()) {
                        subscriber.onNext(items.get(idx));
                    } else {
                        subscriber.onComplete();
                        break;
                    }
                }
            }
            
            @Override
            public void cancel() {
                cancelled.set(true);
            }
        });
    }
    
    private void subscribeHot(Subscriber<? super T> subscriber) {
        // Simplified hot stream implementation
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        subscriber.onSubscribe(new Subscription() {
            private final AtomicBoolean cancelled = new AtomicBoolean(false);
            
            @Override
            public void request(long n) {
                executor.submit(() -> {
                    for (T item : items) {
                        if (cancelled.get()) break;
                        subscriber.onNext(item);
                        try {
                            Thread.sleep(100); // Simulate real-time emission
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            break;
                        }
                    }
                    if (!cancelled.get()) {
                        subscriber.onComplete();
                    }
                });
            }
            
            @Override
            public void cancel() {
                cancelled.set(true);
                executor.shutdown();
            }
        });
    }
    
    public <R> ReactiveStream<R> map(Function<T, R> mapper) {
        FlowableProcessor<T, R> processor = new FlowableProcessor<>(mapper, t -> true);
        return new ReactiveStream<R>(Collections.emptyList(), false) {
            @Override
            public void subscribe(Subscriber<? super R> subscriber) {
                processor.subscribe(subscriber);
                ReactiveStream.this.subscribe(processor);
            }
        };
    }
    
    public ReactiveStream<T> filter(Predicate<T> predicate) {
        FlowableProcessor<T, T> processor = new FlowableProcessor<>(Function.identity(), predicate);
        return new ReactiveStream<T>(Collections.emptyList(), false) {
            @Override
            public void subscribe(Subscriber<? super T> subscriber) {
                processor.subscribe(subscriber);
                ReactiveStream.this.subscribe(processor);
            }
        };
    }
}
```

**Concepts Tested:** Reactive programming, Backpressure, Asynchronous processing, Functional programming

---

### 5. Dynamic Class Loading and Bytecode Manipulation
**Problem:** Build a plugin system with hot-swapping capabilities.

**Requirements:**
- Custom class loaders for isolation
- Bytecode modification at runtime
- Interface injection for plugins
- Memory leak prevention

**Solution Framework:**
```java
import java.lang.instrument.*;
import java.security.ProtectionDomain;
import java.util.concurrent.ConcurrentHashMap;
import javassist.*;

interface Plugin {
    String getName();
    void execute(Map<String, Object> context);
    void destroy();
}

class PluginClassLoader extends ClassLoader {
    private final Map<String, Class<?>> loadedClasses = new ConcurrentHashMap<>();
    private final String pluginPath;
    
    public PluginClassLoader(String pluginPath, ClassLoader parent) {
        super(parent);
        this.pluginPath = pluginPath;
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class<?> cached = loadedClasses.get(name);
        if (cached != null) return cached;
        
        try {
            byte[] classData = loadClassData(name);
            if (classData != null) {
                // Apply bytecode transformations
                classData = transformClass(name, classData);
                
                Class<?> clazz = defineClass(name, classData, 0, classData.length);
                loadedClasses.put(name, clazz);
                return clazz;
            }
        } catch (Exception e) {
            throw new ClassNotFoundException("Could not load class: " + name, e);
        }
        
        return super.findClass(name);
    }
    
    private byte[] loadClassData(String className) {
        // Load from plugin path
        String fileName = className.replace('.', '/') + ".class";
        try (InputStream is = getResourceAsStream(fileName)) {
            if (is == null) return null;
            
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[4096];
            int read;
            while ((read = is.read(buffer)) != -1) {
                baos.write(buffer, 0, read);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            return null;
        }
    }
    
    private byte[] transformClass(String className, byte[] classData) {
        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass ctClass = pool.makeClass(new ByteArrayInputStream(classData));
            
            // Add monitoring/logging capabilities
            if (implementsInterface(ctClass, Plugin.class.getName())) {
                addExecutionTracking(ctClass);
            }
            
            return ctClass.toBytecode();
        } catch (Exception e) {
            System.err.println("Failed to transform class: " + className);
            return classData; // Return original if transformation fails
        }
    }
    
    private boolean implementsInterface(CtClass ctClass, String interfaceName) {
        try {
            CtClass[] interfaces = ctClass.getInterfaces();
            for (CtClass iface : interfaces) {
                if (iface.getName().equals(interfaceName)) {
                    return true;
                }
            }
            return false;
        } catch (NotFoundException e) {
            return false;
        }
    }
    
    private void addExecutionTracking(CtClass ctClass) throws CannotCompileException {
        CtMethod[] methods = ctClass.getDeclaredMethods();
        for (CtMethod method : methods) {
            if (method.getName().equals("execute")) {
                method.insertBefore(
                    "System.out.println(\"Plugin \" + getName() + \" executing at \" + " +
                    "System.currentTimeMillis());"
                );
                method.insertAfter(
                    "System.out.println(\"Plugin \" + getName() + \" completed\");"
                );
            }
        }
    }
    
    public void unload() {
        loadedClasses.clear();
        // Additional cleanup...
    }
}

class PluginManager {
    private final Map<String, PluginInfo> plugins = new ConcurrentHashMap<>();
    private final ExecutorService pluginExecutor = Executors.newCachedThreadPool();
    
    private static class PluginInfo {
        final Plugin instance;
        final PluginClassLoader classLoader;
        final long loadTime;
        
        PluginInfo(Plugin instance, PluginClassLoader classLoader) {
            this.instance = instance;
            this.classLoader = classLoader;
            this.loadTime = System.currentTimeMillis();
        }
    }
    
    public void loadPlugin(String pluginPath, String className) {
        try {
            PluginClassLoader classLoader = new PluginClassLoader(pluginPath, 
                Thread.currentThread().getContextClassLoader());
            
            Class<?> pluginClass = classLoader.loadClass(className);
            
            if (!Plugin.class.isAssignableFrom(pluginClass)) {
                throw new IllegalArgumentException("Class does not implement Plugin interface");
            }
            
            Plugin plugin = (Plugin) pluginClass.getDeclaredConstructor().newInstance();
            
            PluginInfo info = new PluginInfo(plugin, classLoader);
            plugins.put(plugin.getName(), info);
            
            System.out.println("Loaded plugin: " + plugin.getName());
            
        } catch (Exception e) {
            System.err.println("Failed to load plugin: " + e.getMessage());
        }
    }
    
    public void executePlugin(String name, Map<String, Object> context) {
        PluginInfo info = plugins.get(name);
        if (info == null) {
            throw new IllegalArgumentException("Plugin not found: " + name);
        }
        
        pluginExecutor.submit(() -> {
            try {
                // Set context class loader to plugin's class loader
                ClassLoader oldLoader = Thread.currentThread().getContextClassLoader();
                Thread.currentThread().setContextClassLoader(info.classLoader);
                
                try {
                    info.instance.execute(context);
                } finally {
                    Thread.currentThread().setContextClassLoader(oldLoader);
                }
            } catch (Exception e) {
                System.err.println("Plugin execution failed: " + e.getMessage());
            }
        });
    }
    
    public void unloadPlugin(String name) {
        PluginInfo info = plugins.remove(name);
        if (info != null) {
            try {
                info.instance.destroy();
                info.classLoader.unload();
                
                // Force garbage collection to clean up plugin classes
                System.gc();
                
                System.out.println("Unloaded plugin: " + name);
            } catch (Exception e) {
                System.err.println("Error unloading plugin: " + e.getMessage());
            }
        }
    }
    
    public void hotSwapPlugin(String name, String pluginPath, String className) {
        unloadPlugin(name);
        loadPlugin(pluginPath, className);
    }
    
    public void shutdown() {
        plugins.keySet().forEach(this::unloadPlugin);
        pluginExecutor.shutdown();
    }
}
```

**Concepts Tested:** ClassLoaders, Bytecode manipulation, Plugin architecture, Memory management 