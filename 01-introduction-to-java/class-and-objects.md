# Unit 1: Introduction to Java

## Class and Objects

**Object-Oriented Programming (OOP)** structures software around objects rather than functions or isolated procedures. In Java, **Classes** and **Objects** form the foundation of OOP design.

---

### Core Concepts

#### 1. What is a Class?
- A **Class** is a user-defined blueprint or template. It defines:
  - **State (Fields / Instance Variables)**: Data attributes that objects of this class hold.
  - **Behavior (Methods)**: Functions that perform actions on the class fields.
- Writing a class creates a new custom data type in Java.

#### 2. What is an Object?
- An **Object** is a specific instance of a class stored in memory.
- While a class defines the blueprint (e.g. `Student`), an object represents an actual entity built from that blueprint (e.g. `student1` with name `"Alice"` and GPA `3.85`).

#### 3. Instantiation (`new` Keyword)
- **`new`**: Allocates memory on the heap for a new object instance and invokes a constructor to initialize its fields.
  ```java
  Student s = new Student();
  ```

#### 4. Constructors & Overloading
- A **Constructor** is a special method called automatically when an object is created using `new`.
- Characteristics: Shares the **exact same name as the class** and has **no return type** (not even `void`).
- **Default Constructor**: Takes no parameters. Set up automatically by Java if no constructor is written.
- **Parameterized Constructor**: Accepts arguments to initialize custom instance state.
- **Constructor Overloading**: Defining multiple constructors within the same class, differing in parameter lists.

#### 5. The `this` Keyword
- `this` refers to the **current object instance** executing the code.
- Primary use case: Resolves naming collisions when constructor parameters share the exact same names as instance variables:
  ```java
  public Student(String name) {
      this.name = name; // 'this.name' is the field, 'name' is the parameter
  }
  ```

---

### Common Pitfalls

1. **Forgetting `new` when creating objects**:
   - `Student s;` only creates an uninitialized reference variable pointing to `null`. Trying to call `s.displayInfo()` causes a `NullPointerException`. You must instantiate it: `s = new Student();`.
2. **Adding a return type to a constructor**:
   - Writing `public void Student()` turns the constructor into a regular method. Java will no longer invoke it during `new Student()`.
3. **Losing default constructors after writing a custom constructor**:
   - Once you write any parameterized constructor, Java stops providing the default zero-argument constructor automatically. If you still need `new Student()`, you must explicitly write a zero-argument constructor in your class.

---

## 1. Demo: Class Definition and Object Instantiation

### `ClassAndObjectDemo.java`

```java
/**
 * Demonstrates class creation, fields, constructor overloading, and method calls on objects.
 */

class Student {
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

    public void displayInfo() {
        System.out.println("Student Name: " + name);
        System.out.println("Roll Number : " + rollNumber);
        System.out.println("GPA         : " + gpa);
        System.out.println("-----------------------------");
    }
}

public class ClassAndObjectDemo {
    public static void main(String[] args) {
        
        System.out.println("=== 1. Object Created via Default Constructor ===");
        Student student1 = new Student();
        student1.displayInfo();

        System.out.println("=== 2. Objects Created via Parameterized Constructor ===");
        Student student2 = new Student("Alice Johnson", 101, 3.85);
        Student student3 = new Student("Bob Smith", 102, 3.60);

        student2.displayInfo();
        student3.displayInfo();
    }
}
```

### Explanation

1. **Class Structure (`class Student`)**:
   - Declares instance variables `name`, `rollNumber`, and `gpa`. Every instance of `Student` created will store its own copy of these fields.

2. **Constructor Overloading**:
   - `public Student()`: Zero-argument constructor assigning default fallback values (`"Unknown"`, `0`, `0.0`).
   - `public Student(String name, int rollNumber, double gpa)`: Parameterized constructor. Uses `this.name = name;` to assign the incoming argument `name` to the instance variable `this.name`.

3. **Instantiating Objects in `main`**:
   - `Student student1 = new Student();`: Calls default constructor. `displayInfo()` outputs `"Unknown"`.
   - `Student student2 = new Student("Alice Johnson", 101, 3.85);`: Allocates memory for `student2` and initializes fields with Alice's details.

---

## 2. Real-World Program: Bank Account Manager

### `BankAccountManager.java`

```java
import java.util.Scanner;

/**
 * Bank account model featuring encapsulated fields, deposits, withdrawals, and balance reports.
 *
 * Example:
 *   Create Account: "Sarah", Balance: $500.00
 *   Withdraw: $150.00 -> New Balance: $350.00
 */

class BankAccount {
    private String accountNumber;
    private String accountHolderName;
    private double balance;

    public BankAccount(String accountNumber, String accountHolderName, double initialDeposit) {
        this.accountNumber = accountNumber;
        this.accountHolderName = accountHolderName;
        this.balance = (initialDeposit >= 0) ? initialDeposit : 0.0;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.printf("SUCCESS: Deposited $%.2f. New Balance: $%.2f%n", amount, balance);
        } else {
            System.out.println("ERROR: Deposit amount must be positive.");
        }
    }

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

        System.out.print("Enter Account Number (e.g. ACC-8849): ");
        String accNum = scanner.nextLine().trim();

        System.out.print("Enter Account Holder Name: ");
        String holderName = scanner.nextLine().trim();

        System.out.print("Enter Initial Deposit Amount ($): ");
        double initDeposit = scanner.nextDouble();

        BankAccount userAccount = new BankAccount(accNum, holderName, initDeposit);
        userAccount.printAccountSummary();

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

### Explanation

1. **Encapsulation with `private` Modifiers**:
   - `private double balance;`: Prevents code outside `BankAccount` from modifying balance directly (e.g. `acc.balance = -999;`). External code must use controlled methods like `deposit()` or `withdraw()`.

2. **Constructor Input Guarding**:
   - `(initialDeposit >= 0) ? initialDeposit : 0.0`: Ternary operator checking if initial deposit is non-negative before setting starting balance.

3. **Method Logic**:
   - `withdraw(double amount)`: Checks two guard conditions (`amount <= 0` and `amount > balance`) before subtracting funds from `balance`.
