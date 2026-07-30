# Unit 1: Introduction to Java

## Exception Handling

**Exception Handling** manages runtime errors so a program can recover or clean up gracefully instead of crashing abruptly when an unexpected condition occurs.

---

### Core Concepts

#### 1. The `try-catch-finally` Architecture
- **`try` Block**: Wraps code that might throw an exception.
- **`catch` Block**: Intercepts specific exception types thrown from the `try` block and executes recovery code.
- **`finally` Block**: Executes cleanup statements (such as closing database connections or scanner streams) **unconditionally**, whether an exception occurred, was caught, or was not thrown at all.

#### 2. The `throw` vs. `throws` Keywords
- **`throw` (Action)**: Used inside a method body to explicitly throw an exception instance:
  ```java
  throw new IllegalArgumentException("Age cannot be negative");
  ```
- **`throws` (Declaration)**: Used in a method signature to declare that the method might propagate checked exceptions to its caller:
  ```java
  public void readFile(String path) throws IOException { ... }
  ```

#### 3. User-Defined (Custom) Exceptions
- You can create custom domain-specific exceptions by extending Java's built-in exception classes:
  - **Checked Exceptions**: Extend `java.lang.Exception`. The compiler forces callers to handle or declare them.
  - **Unchecked Exceptions**: Extend `java.lang.RuntimeException`. The compiler does not enforce explicit catch/throws syntax.

---

### Common Pitfalls

1. **Catching generic `Exception` everywhere without specific handling**:
   - Writing `catch (Exception e) {}` swallows all errors silently (including logic bugs), making debugging extremely difficult. Catch specific exception types first (`ArithmeticException`, `IOException`).
2. **Order of multi-catch blocks**:
   - Catching a parent class (`Exception`) before a subclass (`InvalidAgeException`) causes a compilation error because the subclass catch block becomes unreachable. Always list specific exceptions before general ones.
3. **Omitting `finally` for resource cleanup**:
   - Forgetting to close streams in `finally` (or using try-with-resources) can leave files locked or leak system resources if an exception halts normal execution before `.close()` is reached.

---

## 1. Demo: Built-in and Custom Exceptions

### `ExceptionHandlingDemo.java`

```java
/**
 * Demonstrates try-catch-finally blocks, throw, throws, and user-defined exception handling.
 */

class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}

public class ExceptionHandlingDemo {

    public static void validateVoterAge(int age) throws InvalidAgeException {
        if (age < 18) {
            throw new InvalidAgeException("Age " + age + " is below the voting limit of 18.");
        }
        System.out.println("Voter eligibility verified for age: " + age);
    }

    public static void main(String[] args) {
        
        // 1. Built-in ArithmeticException
        System.out.println("=== 1. Handling Built-in ArithmeticException ===");
        try {
            int numerator = 50;
            int denominator = 0;
            int result = numerator / denominator;
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            System.out.println("CAUGHT EXCEPTION: Cannot divide by zero. Message: " + e.getMessage());
        } finally {
            System.out.println("FINALLY BLOCK: Cleanup completed.");
        }

        // 2. Custom Exception
        System.out.println("\n=== 2. Handling Custom InvalidAgeException ===");
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

### Explanation

1. **Defining Custom Exception (`class InvalidAgeException extends Exception`)**:
   - Inherits from `Exception`. Passing `message` to `super(message)` stores the error description inside the exception object.

2. **Triggering Exception (`throw new InvalidAgeException(...)`)**:
   - `validateVoterAge` checks `if (age < 18)`. If true, it instantiates `InvalidAgeException` and triggers it using `throw`.

3. **Handling Flow in `main()`**:
   - For `age = 22`: `validateVoterAge` prints eligibility cleanly.
   - For `age = 15`: `validateVoterAge` throws `InvalidAgeException`. Control jumps to `catch (InvalidAgeException e)`, printing the custom error message.
   - In both cases, the `finally` block runs right before the next iteration.

---

## 2. Real-World Program: Online Banking Funds Transfer

### `OnlineBankingTransferSystem.java`

```java
import java.util.Scanner;

/**
 * Bank transfer application using custom InsufficientFundsException and InvalidAccountException classes.
 *
 * Example:
 *   Transfer $1500 with $1000 balance -> Catches InsufficientFundsException.
 */

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

class InvalidAccountException extends Exception {
    public InvalidAccountException(String message) {
        super(message);
    }
}

public class OnlineBankingTransferSystem {

    public static void executeTransfer(String sourceAcc, String destAcc, double amount, double currentBalance)
            throws InvalidAccountException, InsufficientFundsException {

        if (sourceAcc == null || sourceAcc.length() != 8) {
            throw new InvalidAccountException("INVALID ACCOUNT: Source account '" + sourceAcc + "' must be 8 digits.");
        }

        if (destAcc == null || destAcc.length() != 8) {
            throw new InvalidAccountException("INVALID ACCOUNT: Destination account '" + destAcc + "' must be 8 digits.");
        }

        if (amount > currentBalance) {
            throw new InsufficientFundsException(currentBalance, amount);
        }

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
            System.out.println("Banking Transaction Session Closed.");
            System.out.println("------------------------------------------");
            scanner.close();
        }
    }
}
```

### Explanation

1. **Custom Metadata in Exceptions**:
   - `InsufficientFundsException` stores `currentBalance` and `requestedAmount`, providing a helper method `getDeficit()` so caller catch blocks can calculate exact shortfalls.

2. **Ordered Exception Handlers**:
   - Catch blocks first handle specific domain errors (`InvalidAccountException` and `InsufficientFundsException`), providing tailored error messaging, before falling back to a general `Exception` block for unexpected system errors.
