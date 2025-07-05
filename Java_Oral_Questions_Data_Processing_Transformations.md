# Java Oral Coding Questions - Data Processing & Transformations

## 10 Questions on Data Processing and Transformations

### 1. Converting Between Object Types
**Question:** "You need to convert a List<EmployeeEntity> from database to List<EmployeeDTO> for API response. How would you approach this transformation efficiently?"

**Expected Answer Discussion Points:**
- Stream map operation with conversion method
- Builder pattern for DTO creation
- Mapping libraries like MapStruct or ModelMapper
- Manual conversion with null safety
- Performance considerations for large datasets
- Batch processing strategies

### 2. Handling Null Values in Object Processing
**Question:** "When processing Customer objects where some fields might be null (email, phone), how would you safely filter and sort them?"

**Expected Answer Discussion Points:**
- Use Optional class for null-safe operations
- Stream filtering with null checks
- Comparator with nullsFirst()/nullsLast()
- Defensive programming practices
- Default values vs Optional.orElse()
- Stream.ofNullable() usage

### 3. Aggregating Data from Object Collections
**Question:** "You have a list of Sale objects with amount and category. How would you calculate total sales per category and find the highest-selling category?"

**Expected Answer Discussion Points:**
- Collectors.groupingBy() with downstream collectors
- Collectors.summingDouble() for totals
- Map.Entry.comparingByValue() for finding maximum
- Stream.collect() with custom collectors
- Alternative manual aggregation approaches

### 4. Flattening Complex Object Structures
**Question:** "You have a list of Order objects, each containing a list of OrderItem objects. How would you get all items that cost more than $50 across all orders?"

**Expected Answer Discussion Points:**
- flatMap() for flattening nested collections
- Stream chaining with multiple operations
- Filter operations on nested properties
- Performance implications of nested iterations
- Memory efficiency considerations

### 5. Conditional Object Processing
**Question:** "Process a list of Invoice objects differently based on their status: 'PAID' invoices go to archive, 'OVERDUE' to collections, 'PENDING' stay in main list. How would you implement this?"

**Expected Answer Discussion Points:**
- Collectors.partitioningBy() for binary splitting
- Collectors.groupingBy() for multiple categories
- Multiple stream operations for each category
- Switch expressions (Java 14+) in stream operations
- Efficient single-pass processing

### 6. Data Validation and Error Handling
**Question:** "You're processing user registration data where some fields might be invalid. How would you validate objects and collect both valid records and validation errors?"

**Expected Answer Discussion Points:**
- Validation with predicate functions
- Collecting valid vs invalid records separately
- Custom collectors for complex validation
- Either/Result types for error handling
- Accumulating validation errors per object

### 7. Temporal Data Processing
**Question:** "You have a list of Transaction objects with timestamps. How would you group them by day and calculate daily totals for the last 30 days?"

**Expected Answer Discussion Points:**
- LocalDate grouping with date extraction
- Filtering by date ranges
- Collectors.groupingBy() with date keys
- Handling missing days in results
- Time zone considerations

### 8. Object Transformation with External Data
**Question:** "You have Employee objects with department IDs. You also have a Map<DepartmentId, DepartmentName>. How would you create Employee objects enriched with department names?"

**Expected Answer Discussion Points:**
- Stream map with external lookup
- Handling missing department mappings
- Optional usage for missing values
- Performance implications of lookups
- Alternative: joining collections beforehand

### 9. Streaming Large Datasets
**Question:** "You need to process 1 million Product objects from a file to find products below reorder level. How would you handle this efficiently?"

**Expected Answer Discussion Points:**
- Stream processing with Files.lines()
- Lazy evaluation benefits
- Memory-efficient filtering
- Parallel streams for CPU-intensive operations
- try-with-resources for proper cleanup

### 10. Custom Reduction Operations
**Question:** "From a list of Student objects, calculate the weighted average grade where each course has different credit hours. How would you implement this?"

**Expected Answer Discussion Points:**
- Stream.reduce() with custom accumulator
- Collectors.reducing() for complex calculations
- Custom collector implementation
- Accumulating both numerator and denominator
- Handling division by zero cases 