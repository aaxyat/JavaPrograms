# Unit 1: Introduction to Java

## Subheading: Exception Handling

**Exception Handling** in Java provides a robust mechanism to manage runtime errors, ensuring that the normal flow of application execution is maintained rather than abruptly crashing.

### Key Concepts
1. **`try` Block**: Encloses code that might throw an exception.
2. **`catch` Block**: Catches and handles specific exceptions thrown by the corresponding `try` block.
3. **`finally` Block**: Executes cleanup code regardless of whether an exception occurred or was caught.
4. **`throw` Keyword**: Explicitly triggers/throws an exception instance (e.g., `throw new IllegalArgumentException("Invalid value");`).
5. **`throws` Keyword**: Declares in a method signature that the method may propagate specified checked exceptions upstream.
6. **User-Defined (Custom) Exceptions**: Custom exception classes created by extending `Exception` (checked) or `RuntimeException` (unchecked).

---

## 1. Demo Program: Built-in & Custom Exception Handling

**Filename:** `ExceptionHandlingDemo.java`

### Source Code
```java
/**
 * Program: ExceptionHandlingDemo.java
 * Teaches: try-catch-finally blocks, throw, throws, and creating user-defined exceptions.
 * Usage: Demonstrates catching ArithmeticException and throwing custom InvalidAgeException.
 */

// User-Defined (Custom) Exception Class
class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message); // Pass message to Superclass Exception
    }
}

public class ExceptionHandlingDemo {

    // Method declaring potential exception using 'throws'
    public static void validateVoterAge(int age) throws InvalidAgeException {
        if (age < 18) {
            // Explicitly triggering exception using 'throw'
            throw new InvalidAgeException("Age " + age + " is below legal voting limit of 18!");
        }
        System.out.println("Voter eligibility verified for age: " + age);
    }

    public static void main(String[] args) {
        
        // 1. Built-in ArithmeticException Handling (try-catch-finally)
        System.out.println("=== 1. Handling Built-in ArithmeticException ===");
        try {
            int numerator = 50;
            int denominator = 0;
            int result = numerator / denominator; // Division by zero!
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            System.out.println("CAUGHT EXCEPTION: Cannot divide by zero! Details: " + e.getMessage());
        } finally {
            System.out.println("FINALLY BLOCK: Cleanup completed for Division Demo.");
        }

        // 2. Custom User-Defined Exception Handling
        System.out.println("\n=== 2. Handling User-Defined Custom Exception ===");
        int[] testAges = {22, 15};

        for (int age : testAges) {
            try {
                System.out.println("Checking age: " + age);
                validateVoterAge(age);
            } catch (InvalidAgeException e) {
                System.out.println("CUSTOM EXCEPTION CAUGHT: " + e.getMessage());
            } finally {
                System.out.println("Validation check finished for age: " + age + "\n");
            }
        }
    }
}
```

### Code Explanation
1. **User-Defined Exception (`class InvalidAgeException extends Exception`)**:
   - Extends Java's `Exception` class to represent custom domain validation errors.
2. **Explicit Exception Throwing (`throw new InvalidAgeException(...)`)**:
   - `validateVoterAge` checks if `age < 18` and raises `InvalidAgeException` using `throw`.
3. **Try-Catch-Finally Flow**:
   - **`try`**: Encapsulates code where division or age checking occurs.
   - **`catch`**: Catches specific exception types (`ArithmeticException` or `InvalidAgeException`).
   - **`finally`**: Runs guaranteed cleanup statements unconditionally.

---

## 2. Real-World Program: Online Banking Funds Transfer System

**Filename:** `OnlineBankingTransferSystem.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: OnlineBankingTransferSystem.java
 * Teaches: Building a fault-tolerant financial transfer system using custom exceptions (InsufficientFundsException, InvalidAccountException).
 * Example Input/Output:
 *   Transfer $1500 from Account with $1000 balance -> Throws InsufficientFundsException: Requested $1500.00 exceeds balance $1000.00
 */

// Custom Exception 1: Insufficient Funds
class InsufficientFundsException extends Exception {
    private double currentBalance;
    private double requestedAmount;

    public InsufficientFundsException(double currentBalance, double requestedAmount) {
        super("TRANSFER REJECTED: Requested $" + requestedAmount + " exceeds available balance of $" + currentBalance);
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }

    public double getDeficit() {
        return requestedAmount - currentBalance;
    }
}

// Custom Exception 2: Invalid Account Number
class InvalidAccountException extends Exception {
    public InvalidAccountException(String message) {
        super(message);
    }
}

public class OnlineBankingTransferSystem {

    // Simulates transferring money between accounts
    public static void executeTransfer(String sourceAcc, String destAcc, double amount, double currentBalance)
            throws InvalidAccountException, InsufficientFundsException {

        // Validate Source Account Length
        if (sourceAcc == null || sourceAcc.length() != 8) {
            throw new InvalidAccountException("INVALID ACCOUNT: Source account '" + sourceAcc + "' must be 8 digits long.");
        }

        // Validate Destination Account Length
        if (destAcc == null || destAcc.length() != 8) {
            throw new InvalidAccountException("INVALID ACCOUNT: Destination account '" + destAcc + "' must be 8 digits long.");
        }

        // Validate Transfer Amount against Balance
        if (amount > currentBalance) {
            throw new InsufficientFundsException(currentBalance, amount);
        }

        // If all checks pass, complete transaction
        System.out.printf("TRANSACTION SUCCESSFUL: Transferred $%.2f from [%s] to [%s]%n", amount, sourceAcc, destAcc);
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        double senderBalance = 1000.00;

        System.out.println("==========================================");
        System.out.println("      JAVA SECURE BANK TRANSFER PORTAL     ");
        System.out.println("==========================================");
        System.out.printf("Sender Account Balance: $%.2f%n%n", senderBalance);

        try {
            System.out.print("Enter 8-digit Sender Account #: ");
            String senderAcc = scanner.nextLine().trim();

            System.out.print("Enter 8-digit Recipient Account #: ");
            String recipientAcc = scanner.nextLine().trim();

            System.out.print("Enter Transfer Amount ($): ");
            double transferAmount = scanner.nextDouble();

            // Attempting transaction that might throw custom exceptions
            executeTransfer(senderAcc, recipientAcc, transferAmount, senderBalance);

        } catch (InvalidAccountException e) {
            System.out.println("\n[ACCOUNT ERROR] " + e.getMessage());
        } catch (InsufficientFundsException e) {
            System.out.println("\n[FUNDS ERROR] " + e.getMessage());
            System.out.printf("Shortfall Deficit: $%.2f%n", e.getDeficit());
        } catch (Exception e) {
            System.out.println("\n[UNEXPECTED ERROR] System encountered error: " + e.getMessage());
        } finally {
            System.out.println("\n------------------------------------------");
            System.out.println("Banking Transaction Session Safely Closed.");
            System.out.println("------------------------------------------");
            scanner.close();
        }
    }
}
```

### Code Explanation
1. **Domain-Specific Custom Exceptions**:
   - `InsufficientFundsException` tracks financial metadata (`currentBalance`, `requestedAmount`, `getDeficit()`).
   - `InvalidAccountException` handles account format verification failures.
2. **Multi-Catch Hierarchy**:
   - Catch blocks are ordered from specific custom exceptions (`InvalidAccountException`, `InsufficientFundsException`) down to generic `Exception` fallback.
3. **Transaction Safety**:
   - Guard clauses throw exceptions before mutating any balance, preventing partial or corrupted transactions.
