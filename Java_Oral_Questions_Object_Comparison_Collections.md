# Java Oral Coding Questions - Object Comparison & Collections

## 10 Questions on Object Comparison and Collections Handling

### 1. Object Comparison Based on Field
**Question:** "You have a Student class with fields like name, age, and grade. How would you compare two Student objects based on their grade field? Explain the different approaches."

**Expected Answer Discussion Points:**
- Implement Comparable interface with compareTo() method
- Use Comparator interface for external comparison
- Override equals() and hashCode() for equality comparison
- Difference between natural ordering vs custom ordering
- Example: `Collections.sort(students, Comparator.comparing(Student::getGrade))`

### 2. Integer Caching and Object Comparison Gotcha
**Question:** "You're comparing Integer objects using == and getting unexpected results. For values like 100, it works fine, but for values like 200, it fails. Can you explain this behavior and how to fix it?"

**Expected Answer Discussion Points:**
- Java caches Integer objects from -128 to 127 (Integer cache pool)
- Within this range, == comparison works because same cached objects are reused
- Outside this range, new Integer objects are created, so == fails even with same values
- Always use equals() method for Integer value comparison, never ==
- This applies to other wrapper classes too (Byte, Short, Long with smaller ranges)
- Autoboxing can make this behavior subtle and hard to detect
- Example scenario: `Integer a = 100; Integer b = 100;` (a == b is true) vs `Integer c = 200; Integer d = 200;` (c == d is false)

### 3. Finding Duplicate Objects in a List
**Question:** "Given a List of Employee objects, how would you find and remove duplicates based on employee ID? Walk me through your approach."

**Expected Answer Discussion Points:**
- Override equals() and hashCode() in Employee class based on ID
- Use HashSet to filter duplicates
- Use Stream API distinct() method
- Manual approach with Map to track seen IDs
- Discuss time complexity differences

### 4. Grouping Objects by Property
**Question:** "You have a list of Order objects with customer names. How would you group all orders by customer name?"

**Expected Answer Discussion Points:**
- Stream API with Collectors.groupingBy()
- Manual approach with HashMap<String, List<Order>>
- Result structure as Map<String, List<Order>>
- Variations like counting, summing grouped values
- Performance considerations

### 5. Sorting Objects by Multiple Criteria
**Question:** "How would you sort a list of Book objects first by author name, then by publication year?"

**Expected Answer Discussion Points:**
- Comparator.comparing() with thenComparing()
- Custom Comparator implementation
- Handling null values with nullsFirst()/nullsLast()
- Reverse ordering options
- Performance implications of complex sorting

### 6. Converting Object List to Map
**Question:** "You have a List<Product> and want to create a Map where product ID is the key. How would you handle this conversion?"

**Expected Answer Discussion Points:**
- Stream.collect(toMap()) with key and value mappers
- Handling duplicate keys with merge functions
- Different value options (full object vs specific field)
- Manual iteration approach
- Error handling for null values

### 7. Finding Common Elements Between Collections
**Question:** "Given two lists of Student objects, how would you find students who appear in both lists?"

**Expected Answer Discussion Points:**
- Override equals() and hashCode() properly
- Use retainAll() method for intersection
- Stream filter with contains() check
- Convert to Set for better performance
- Discuss algorithmic complexity

### 8. Filtering and Transforming Collections
**Question:** "You have a list of Order objects. How would you get all orders from last month with total amount greater than $100, then extract just the customer names?"

**Expected Answer Discussion Points:**
- Stream pipeline with filter operations
- Date comparison techniques
- Map transformation to extract customer names
- Collecting results to appropriate collection
- Performance considerations for large datasets

### 9. Partitioning Collections Based on Criteria
**Question:** "How would you split a list of Student objects into two groups: those who passed (grade >= 60) and those who failed?"

**Expected Answer Discussion Points:**
- Collectors.partitioningBy() method
- Result as Map<Boolean, List<Student>>
- Manual approach with two separate lists
- Stream filtering for each partition
- Use cases for partitioning vs grouping

### 10. Finding Maximum/Minimum Objects
**Question:** "From a list of Employee objects, how would you find the employee with the highest salary? What if multiple employees have the same highest salary?"

**Expected Answer Discussion Points:**
- Stream.max() with Comparator
- Optional handling for empty collections
- Finding all maximum values (not just first)
- Custom reduction operations
- Handling edge cases and null values

### 11. Nested Object Collections
**Question:** "You have a list of Department objects, each containing a list of Employee objects. How would you get all employees across all departments who earn more than $50,000?"

**Expected Answer Discussion Points:**
- flatMap() to flatten nested collections
- Stream chaining for complex operations
- Filter operations on nested properties
- Performance implications of nested iterations
- Alternative approaches with traditional loops 