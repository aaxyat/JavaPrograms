# Unit 1: Introduction to Java

## Subheading: Class and Objects

Java is an Object-Oriented Programming (OOP) language built around **Classes** and **Objects**. Object-oriented design models real-world entities into modular, reusable software components.

### Key Concepts
1. **Class**: A user-defined blueprint or template that specifies the attributes (state/data fields) and methods (behavior/functions) common to all objects of that type.
2. **Object**: An instance of a class allocated in heap memory. Objects encapsulate state and interact via methods.
3. **Instantiation (`new` keyword)**: The process of creating an object instance in memory (e.g., `Car myCar = new Car();`).
4. **Constructors**: Special methods invoked automatically during object creation to initialize object attributes.
   - **Default Constructor**: Provided by Java automatically if no explicit constructor is defined.
   - **Parameterized Constructor**: Accepts arguments to initialize custom instance state.
5. **The `this` Keyword**: A reference variable pointing to the current object instance within a class method or constructor.

---

## 1. Demo Program: Basic Class & Object Instantiation

**Filename:** `ClassAndObjectDemo.java`

### Source Code
```java
/**
 * Program: ClassAndObjectDemo.java
 * Teaches: Defining classes, instance variables, parameterized constructors, methods, and creating multiple objects.
 * Usage: Demonstrates object creation, attribute initialization, and method calls.
 */

// Class Definition (Blueprint)
class Student {
    // Instance Variables (Attributes)
    String name;
    int rollNumber;
    double gpa;

    // Default Constructor
    public Student() {
        this.name = "Unknown";
        this.rollNumber = 0;
        this.gpa = 0.0;
    }

    // Parameterized Constructor
    public Student(String name, int rollNumber, double gpa) {
        this.name = name;
        this.rollNumber = rollNumber;
        this.gpa = gpa;
    }

    // Method to display student information
    public void displayInfo() {
        System.out.println("Student Name: " + name);
        System.out.println("Roll Number : " + rollNumber);
        System.out.println("GPA         : " + gpa);
        System.out.println("-----------------------------");
    }
}

public class ClassAndObjectDemo {
    public static void main(String[] args) {
        
        System.out.println("=== 1. Creating Object via Default Constructor ===");
        Student student1 = new Student();
        student1.displayInfo();

        System.out.println("=== 2. Creating Objects via Parameterized Constructor ===");
        Student student2 = new Student("Alice Johnson", 101, 3.85);
        Student student3 = new Student("Bob Smith", 102, 3.60);

        // Invoking object behaviors (methods)
        student2.displayInfo();
        student3.displayInfo();
    }
}
```

### Code Explanation
1. **Class Declaration (`class Student`)**:
   - Outlines the attributes (`name`, `rollNumber`, `gpa`) and behavioral methods (`displayInfo()`) for all student entities.
2. **Constructor Overloading**:
   - `Student()` sets default fallback values.
   - `Student(name, rollNumber, gpa)` uses `this` to resolve naming conflicts between parameters and instance variables.
3. **Object Instantiation & Method Access (`student2.displayInfo()`)**:
   - `new Student("Alice Johnson", 101, 3.85)` allocates memory for `student2` on the heap and executes constructor initialization.
   - Invoking `.displayInfo()` prints the distinct attribute state of each individual object.

---

## 2. Real-World Program: Bank Account Management System

**Filename:** `BankAccountManager.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: BankAccountManager.java
 * Teaches: Real-world class modeling, state encapsulation, deposit/withdrawal methods, and account reporting.
 * Example Input/Output:
 *   Create Account for "Sarah Jenkins", Deposit: $500, Withdraw: $150
 *   Output: Transaction Successful. Current Balance: $350.00
 */

// BankAccount Class
class BankAccount {
    private String accountNumber;
    private String accountHolderName;
    private double balance;

    // Parameterized Constructor
    public BankAccount(String accountNumber, String accountHolderName, double initialDeposit) {
        this.accountNumber = accountNumber;
        this.accountHolderName = accountHolderName;
        this.balance = (initialDeposit >= 0) ? initialDeposit : 0.0;
    }

    // Deposit Method
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.printf("SUCCESS: Deposited $%.2f. New Balance: $%.2f%n", amount, balance);
        } else {
            System.out.println("ERROR: Deposit amount must be positive.");
        }
    }

    // Withdrawal Method
    public void withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("ERROR: Withdrawal amount must be positive.");
        } else if (amount > balance) {
            System.out.println("TRANSACTION FAILED: Insufficient funds. Available: $" + balance);
        } else {
            balance -= amount;
            System.out.printf("SUCCESS: Withdrew $%.2f. Remaining Balance: $%.2f%n", amount, balance);
        }
    }

    // Display Account Summary
    public void printAccountSummary() {
        System.out.println("\n------------------------------------------");
        System.out.println("         ACCOUNT SUMMARY REPORT           ");
        System.out.println("------------------------------------------");
        System.out.println("Account Number : " + accountNumber);
        System.out.println("Account Holder : " + accountHolderName);
        System.out.printf("Current Balance: $%.2f%n", balance);
        System.out.println("------------------------------------------");
    }
}

public class BankAccountManager {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("     BANK ACCOUNT CREATION & MANAGEMENT   ");
        System.out.println("==========================================");

        // Prompting Account Details
        System.out.print("Enter Account Number (e.g. ACC-8849): ");
        String accNum = scanner.nextLine().trim();

        System.out.print("Enter Account Holder Name: ");
        String holderName = scanner.nextLine().trim();

        System.out.print("Enter Initial Deposit Amount ($): ");
        double initDeposit = scanner.nextDouble();

        // Instantiating BankAccount Object
        BankAccount userAccount = new BankAccount(accNum, holderName, initDeposit);
        userAccount.printAccountSummary();

        // Interactive Operations
        boolean managing = true;
        while (managing) {
            System.out.println("\nSelect Action: 1. Deposit | 2. Withdraw | 3. View Summary | 4. Exit");
            System.out.print("Choice: ");
            int choice = scanner.nextInt();

            switch (choice) {
                case 1:
                    System.out.print("Enter deposit amount ($): ");
                    double depAmt = scanner.nextDouble();
                    userAccount.deposit(depAmt);
                    break;
                case 2:
                    System.out.print("Enter withdrawal amount ($): ");
                    double withAmt = scanner.nextDouble();
                    userAccount.withdraw(withAmt);
                    break;
                case 3:
                    userAccount.printAccountSummary();
                    break;
                case 4:
                    System.out.println("Exiting account session. Goodbye!");
                    managing = false;
                    break;
                default:
                    System.out.println("Invalid option selected.");
                    break;
            }
        }

        scanner.close();
    }
}
```

### Code Explanation
1. **Class Encapsulation (`BankAccount`)**:
   - Bundles account data (`accountNumber`, `accountHolderName`, `balance`) together with behavior methods (`deposit()`, `withdraw()`, `printAccountSummary()`).
2. **State Validation in Constructor**:
   - Ensures `initialDeposit` cannot set a negative starting balance (`(initialDeposit >= 0) ? initialDeposit : 0.0`).
3. **Behavior Execution (`userAccount.deposit()`, `userAccount.withdraw()`)**:
   - Modifies object internal state safely with guard clauses preventing invalid negative transactions or overdrafts.
