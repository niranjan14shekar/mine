# Java Interview - Easy Coding Questions

## Applied Concept-Based Programming Problems

### 1. Student Grade Calculator
**Problem:** Create a Student class that calculates and categorizes grades.

**Requirements:**
- Calculate average of multiple subjects
- Determine grade category (A, B, C, D, F)
- Handle invalid inputs

**Expected Solution:**
```java
public class Student {
    private String name;
    private int[] marks;
    
    public Student(String name, int[] marks) {
        this.name = name;
        this.marks = marks.clone(); // Defensive copying
    }
    
    public double calculateAverage() {
        if (marks.length == 0) return 0;
        
        int sum = 0;
        for (int mark : marks) {
            if (mark < 0 || mark > 100) {
                throw new IllegalArgumentException("Invalid mark: " + mark);
            }
            sum += mark;
        }
        return (double) sum / marks.length;
    }
    
    public char getGrade() {
        double average = calculateAverage();
        if (average >= 90) return 'A';
        if (average >= 80) return 'B';
        if (average >= 70) return 'C';
        if (average >= 60) return 'D';
        return 'F';
    }
    
    public String getStudentInfo() {
        return String.format("%s - Average: %.2f, Grade: %c", 
                           name, calculateAverage(), getGrade());
    }
}

// Usage
public class TestStudent {
    public static void main(String[] args) {
        Student student = new Student("John Doe", new int[]{85, 92, 78, 96, 87});
        System.out.println(student.getStudentInfo());
    }
}
```

**Concepts Tested:**
- Class design and encapsulation
- Array handling
- Exception handling
- Data validation

---

### 2. Bank Account Management
**Problem:** Implement a simple bank account system with basic operations.

**Requirements:**
- Support deposit, withdrawal, and balance inquiry
- Prevent overdrafts
- Maintain transaction history
- Apply different account types (Savings, Checking)

**Expected Solution:**
```java
import java.util.*;

abstract class BankAccount {
    protected String accountNumber;
    protected double balance;
    protected List<String> transactionHistory;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
        this.transactionHistory = new ArrayList<>();
        addTransaction("Account opened with balance: " + initialBalance);
    }
    
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        balance += amount;
        addTransaction("Deposited: " + amount);
    }
    
    public abstract boolean withdraw(double amount);
    
    protected void addTransaction(String transaction) {
        String timestamp = new Date().toString();
        transactionHistory.add(timestamp + " - " + transaction);
    }
    
    public double getBalance() {
        return balance;
    }
    
    public void printStatement() {
        System.out.println("Account: " + accountNumber);
        System.out.println("Current Balance: " + balance);
        System.out.println("Transaction History:");
        for (String transaction : transactionHistory) {
            System.out.println(transaction);
        }
    }
}

class SavingsAccount extends BankAccount {
    private static final double MIN_BALANCE = 100.0;
    
    public SavingsAccount(String accountNumber, double initialBalance) {
        super(accountNumber, initialBalance);
    }
    
    @Override
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive");
            return false;
        }
        
        if (balance - amount < MIN_BALANCE) {
            System.out.println("Insufficient funds. Minimum balance required: " + MIN_BALANCE);
            return false;
        }
        
        balance -= amount;
        addTransaction("Withdrawn: " + amount);
        return true;
    }
}

class CheckingAccount extends BankAccount {
    private static final double OVERDRAFT_LIMIT = 500.0;
    
    public CheckingAccount(String accountNumber, double initialBalance) {
        super(accountNumber, initialBalance);
    }
    
    @Override
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive");
            return false;
        }
        
        if (balance - amount < -OVERDRAFT_LIMIT) {
            System.out.println("Overdraft limit exceeded. Limit: " + OVERDRAFT_LIMIT);
            return false;
        }
        
        balance -= amount;
        addTransaction("Withdrawn: " + amount);
        return true;
    }
}

// Usage
public class BankSystem {
    public static void main(String[] args) {
        SavingsAccount savings = new SavingsAccount("SAV001", 1000);
        CheckingAccount checking = new CheckingAccount("CHK001", 500);
        
        savings.deposit(200);
        savings.withdraw(150);
        
        checking.withdraw(800); // Uses overdraft
        
        savings.printStatement();
        checking.printStatement();
    }
}
```

**Concepts Tested:**
- Inheritance and abstract classes
- Polymorphism
- Collections (ArrayList)
- Access modifiers
- Business logic implementation

---

### 3. Shopping Cart System
**Problem:** Create a shopping cart that calculates totals with discounts.

**Requirements:**
- Add/remove items
- Calculate subtotal, tax, and total
- Apply discount codes
- Generate itemized receipt

**Expected Solution:**
```java
import java.util.*;

class Item {
    private String name;
    private double price;
    private int quantity;
    
    public Item(String name, double price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }
    
    public double getTotal() {
        return price * quantity;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }
    
    @Override
    public String toString() {
        return String.format("%s - $%.2f x %d = $%.2f", 
                           name, price, quantity, getTotal());
    }
}

class ShoppingCart {
    private Map<String, Item> items;
    private static final double TAX_RATE = 0.08;
    
    public ShoppingCart() {
        items = new HashMap<>();
    }
    
    public void addItem(String name, double price, int quantity) {
        if (items.containsKey(name)) {
            Item existingItem = items.get(name);
            existingItem.setQuantity(existingItem.getQuantity() + quantity);
        } else {
            items.put(name, new Item(name, price, quantity));
        }
    }
    
    public boolean removeItem(String name) {
        return items.remove(name) != null;
    }
    
    public void updateQuantity(String name, int quantity) {
        if (items.containsKey(name)) {
            if (quantity <= 0) {
                removeItem(name);
            } else {
                items.get(name).setQuantity(quantity);
            }
        }
    }
    
    public double getSubtotal() {
        return items.values().stream()
                   .mapToDouble(Item::getTotal)
                   .sum();
    }
    
    public double getTax() {
        return getSubtotal() * TAX_RATE;
    }
    
    public double getTotal() {
        return getSubtotal() + getTax();
    }
    
    public double applyDiscount(String discountCode) {
        double discount = 0;
        switch (discountCode.toUpperCase()) {
            case "SAVE10":
                discount = getSubtotal() * 0.10;
                break;
            case "SAVE20":
                discount = getSubtotal() * 0.20;
                break;
            case "NEWCUSTOMER":
                discount = Math.min(50, getSubtotal() * 0.15);
                break;
        }
        return discount;
    }
    
    public void printReceipt(String discountCode) {
        System.out.println("===== RECEIPT =====");
        for (Item item : items.values()) {
            System.out.println(item);
        }
        
        double subtotal = getSubtotal();
        double discount = applyDiscount(discountCode);
        double tax = (subtotal - discount) * TAX_RATE;
        double total = subtotal - discount + tax;
        
        System.out.println("-------------------");
        System.out.printf("Subtotal: $%.2f%n", subtotal);
        if (discount > 0) {
            System.out.printf("Discount (%s): -$%.2f%n", discountCode, discount);
        }
        System.out.printf("Tax: $%.2f%n", tax);
        System.out.printf("TOTAL: $%.2f%n", total);
        System.out.println("==================");
    }
}

// Usage
public class ShoppingCartDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        cart.addItem("Laptop", 999.99, 1);
        cart.addItem("Mouse", 25.50, 2);
        cart.addItem("Keyboard", 75.00, 1);
        
        cart.printReceipt("SAVE10");
    }
}
```

**Concepts Tested:**
- Object composition
- HashMap usage
- Stream API basics
- String manipulation
- Method overloading

---

### 4. Employee Payroll System
**Problem:** Design a payroll system for different employee types.

**Requirements:**
- Handle full-time and part-time employees
- Calculate different pay structures
- Generate pay stubs
- Track overtime

**Expected Solution:**
```java
import java.util.*;

abstract class Employee {
    protected String empId;
    protected String name;
    protected Date hireDate;
    
    public Employee(String empId, String name) {
        this.empId = empId;
        this.name = name;
        this.hireDate = new Date();
    }
    
    public abstract double calculatePay();
    public abstract String getEmployeeType();
    
    public void generatePayStub() {
        System.out.println("===== PAY STUB =====");
        System.out.println("Employee ID: " + empId);
        System.out.println("Name: " + name);
        System.out.println("Type: " + getEmployeeType());
        System.out.println("Pay: $" + String.format("%.2f", calculatePay()));
        System.out.println("==================");
    }
    
    // Getters
    public String getEmpId() { return empId; }
    public String getName() { return name; }
}

class FullTimeEmployee extends Employee {
    private double annualSalary;
    
    public FullTimeEmployee(String empId, String name, double annualSalary) {
        super(empId, name);
        this.annualSalary = annualSalary;
    }
    
    @Override
    public double calculatePay() {
        return annualSalary / 12; // Monthly pay
    }
    
    @Override
    public String getEmployeeType() {
        return "Full-Time";
    }
}

class PartTimeEmployee extends Employee {
    private double hourlyRate;
    private int hoursWorked;
    private static final int STANDARD_HOURS = 40;
    private static final double OVERTIME_MULTIPLIER = 1.5;
    
    public PartTimeEmployee(String empId, String name, double hourlyRate) {
        super(empId, name);
        this.hourlyRate = hourlyRate;
        this.hoursWorked = 0;
    }
    
    public void setHoursWorked(int hours) {
        this.hoursWorked = hours;
    }
    
    @Override
    public double calculatePay() {
        if (hoursWorked <= STANDARD_HOURS) {
            return hourlyRate * hoursWorked;
        } else {
            int overtimeHours = hoursWorked - STANDARD_HOURS;
            return (hourlyRate * STANDARD_HOURS) + 
                   (hourlyRate * OVERTIME_MULTIPLIER * overtimeHours);
        }
    }
    
    @Override
    public String getEmployeeType() {
        return "Part-Time";
    }
    
    @Override
    public void generatePayStub() {
        System.out.println("===== PAY STUB =====");
        System.out.println("Employee ID: " + empId);
        System.out.println("Name: " + name);
        System.out.println("Type: " + getEmployeeType());
        System.out.println("Hours Worked: " + hoursWorked);
        System.out.println("Hourly Rate: $" + hourlyRate);
        if (hoursWorked > STANDARD_HOURS) {
            System.out.println("Overtime Hours: " + (hoursWorked - STANDARD_HOURS));
        }
        System.out.println("Total Pay: $" + String.format("%.2f", calculatePay()));
        System.out.println("==================");
    }
}

class PayrollSystem {
    private List<Employee> employees;
    
    public PayrollSystem() {
        employees = new ArrayList<>();
    }
    
    public void addEmployee(Employee employee) {
        employees.add(employee);
    }
    
    public Employee findEmployee(String empId) {
        return employees.stream()
                       .filter(emp -> emp.getEmpId().equals(empId))
                       .findFirst()
                       .orElse(null);
    }
    
    public void processPayroll() {
        System.out.println("Processing Payroll for all employees:");
        for (Employee emp : employees) {
            emp.generatePayStub();
            System.out.println();
        }
    }
    
    public double getTotalPayroll() {
        return employees.stream()
                       .mapToDouble(Employee::calculatePay)
                       .sum();
    }
}

// Usage
public class PayrollDemo {
    public static void main(String[] args) {
        PayrollSystem payroll = new PayrollSystem();
        
        FullTimeEmployee fullTime = new FullTimeEmployee("FT001", "Alice Johnson", 60000);
        PartTimeEmployee partTime = new PartTimeEmployee("PT001", "Bob Smith", 15.0);
        partTime.setHoursWorked(45); // Including overtime
        
        payroll.addEmployee(fullTime);
        payroll.addEmployee(partTime);
        
        payroll.processPayroll();
        System.out.println("Total Payroll: $" + payroll.getTotalPayroll());
    }
}
```

**Concepts Tested:**
- Abstract classes and inheritance
- Method overriding
- Polymorphism
- List operations
- Stream API
- Business logic

---

### 5. Library Book Management
**Problem:** Create a library system to manage books and borrowers.

**Requirements:**
- Track book availability
- Handle borrowing and returning
- Search books by title/author
- Calculate late fees

**Expected Solution:**
```java
import java.time.*;
import java.time.temporal.ChronoUnit;
import java.util.*;

class Book {
    private String isbn;
    private String title;
    private String author;
    private boolean isAvailable;
    private LocalDate dueDate;
    private String borrowerId;
    
    public Book(String isbn, String title, String author) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.isAvailable = true;
    }
    
    public boolean borrow(String borrowerId, int borrowDays) {
        if (!isAvailable) {
            return false;
        }
        this.isAvailable = false;
        this.borrowerId = borrowerId;
        this.dueDate = LocalDate.now().plusDays(borrowDays);
        return true;
    }
    
    public double returnBook() {
        if (isAvailable) {
            return 0; // Book wasn't borrowed
        }
        
        isAvailable = true;
        double lateFee = calculateLateFee();
        borrowerId = null;
        dueDate = null;
        return lateFee;
    }
    
    private double calculateLateFee() {
        if (dueDate == null || LocalDate.now().isBefore(dueDate)) {
            return 0;
        }
        long daysLate = ChronoUnit.DAYS.between(dueDate, LocalDate.now());
        return daysLate * 0.50; // $0.50 per day late
    }
    
    public boolean isOverdue() {
        return !isAvailable && dueDate != null && LocalDate.now().isAfter(dueDate);
    }
    
    // Getters
    public String getIsbn() { return isbn; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public boolean isAvailable() { return isAvailable; }
    public LocalDate getDueDate() { return dueDate; }
    public String getBorrowerId() { return borrowerId; }
    
    @Override
    public String toString() {
        return String.format("%s by %s (ISBN: %s) - %s", 
                           title, author, isbn, 
                           isAvailable ? "Available" : "Borrowed until " + dueDate);
    }
}

class Library {
    private Map<String, Book> books;
    private static final int DEFAULT_BORROW_DAYS = 14;
    
    public Library() {
        books = new HashMap<>();
    }
    
    public void addBook(Book book) {
        books.put(book.getIsbn(), book);
    }
    
    public boolean borrowBook(String isbn, String borrowerId) {
        Book book = books.get(isbn);
        if (book == null) {
            System.out.println("Book not found");
            return false;
        }
        
        if (book.borrow(borrowerId, DEFAULT_BORROW_DAYS)) {
            System.out.println("Book borrowed successfully. Due date: " + book.getDueDate());
            return true;
        } else {
            System.out.println("Book is not available");
            return false;
        }
    }
    
    public double returnBook(String isbn) {
        Book book = books.get(isbn);
        if (book == null) {
            System.out.println("Book not found");
            return 0;
        }
        
        double lateFee = book.returnBook();
        if (lateFee > 0) {
            System.out.println("Book returned with late fee: $" + String.format("%.2f", lateFee));
        } else {
            System.out.println("Book returned on time");
        }
        return lateFee;
    }
    
    public List<Book> searchByTitle(String title) {
        return books.values().stream()
                   .filter(book -> book.getTitle().toLowerCase().contains(title.toLowerCase()))
                   .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
    
    public List<Book> searchByAuthor(String author) {
        return books.values().stream()
                   .filter(book -> book.getAuthor().toLowerCase().contains(author.toLowerCase()))
                   .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
    
    public List<Book> getOverdueBooks() {
        return books.values().stream()
                   .filter(Book::isOverdue)
                   .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
    
    public void printLibraryStatus() {
        System.out.println("===== LIBRARY STATUS =====");
        for (Book book : books.values()) {
            System.out.println(book);
        }
        
        List<Book> overdueBooks = getOverdueBooks();
        if (!overdueBooks.isEmpty()) {
            System.out.println("\nOVERDUE BOOKS:");
            for (Book book : overdueBooks) {
                System.out.println(book + " (Borrower: " + book.getBorrowerId() + ")");
            }
        }
        System.out.println("==========================");
    }
}

// Usage
public class LibraryDemo {
    public static void main(String[] args) {
        Library library = new Library();
        
        // Add books
        library.addBook(new Book("978-0-134-68552-4", "Java: The Complete Reference", "Herbert Schildt"));
        library.addBook(new Book("978-0-321-35668-0", "Effective Java", "Joshua Bloch"));
        library.addBook(new Book("978-0-596-00726-8", "Head First Design Patterns", "Eric Freeman"));
        
        // Borrow books
        library.borrowBook("978-0-134-68552-4", "USER001");
        library.borrowBook("978-0-321-35668-0", "USER002");
        
        // Search functionality
        System.out.println("Java books:");
        List<Book> javaBooks = library.searchByTitle("Java");
        javaBooks.forEach(System.out::println);
        
        // Return a book
        library.returnBook("978-0-134-68552-4");
        
        // Print status
        library.printLibraryStatus();
    }
}
```

**Concepts Tested:**
- Date and time handling (LocalDate)
- Stream API and collectors
- HashMap operations
- Business logic with calculations
- Search and filter operations 