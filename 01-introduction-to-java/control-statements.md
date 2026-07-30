# Unit 1: Introduction to Java

## Subheading: Control Statements

Control statements in Java direct the flow of program execution based on conditional logic and decisions. They allow a program to select execution paths, validate user inputs, and process business logic dynamically.

### Key Concepts
1. **Decision-Making Statements**: `if`, `if-else`, `else-if` ladder, and `switch` statements evaluate boolean conditions to choose execution branches.
2. **Branching & Exit Control**: Early return statements and `break` controls prevent execution of unauthorized or invalid logic blocks.

---

## 1. Demo Program: Control Flow Basics

**Filename:** `ControlFlowDemo.java`

### Source Code
```java
/**
 * Program: ControlFlowDemo.java
 * Teaches: Basic decision-making using if-else and switch statements.
 * Usage: Demonstrates grade calculation and day selection.
 */
public class ControlFlowDemo {
    public static void main(String[] args) {
        int score = 85;

        // 1. If-Else Ladder Demo
        System.out.println("=== 1. If-Else Grade Evaluation ===");
        if (score >= 90) {
            System.out.println("Grade: A");
        } else if (score >= 80) {
            System.out.println("Grade: B");
        } else if (score >= 70) {
            System.out.println("Grade: C");
        } else {
            System.out.println("Grade: F");
        }

        // 2. Switch Statement Demo
        System.out.println("\n=== 2. Switch Statement Demo ===");
        int dayOfWeek = 3;
        switch (dayOfWeek) {
            case 1:
                System.out.println("Day: Monday");
                break;
            case 2:
                System.out.println("Day: Tuesday");
                break;
            case 3:
                System.out.println("Day: Wednesday");
                break;
            case 4:
                System.out.println("Day: Thursday");
                break;
            case 5:
                System.out.println("Day: Friday");
                break;
            default:
                System.out.println("Day: Weekend");
                break;
        }
    }
}
```

### Code Explanation
1. **If-Else Ladder (`score >= 80`)**:
   - The program checks conditions sequentially from top to bottom.
   - Since `score = 85`, the expression `score >= 90` evaluates to `false`. Next, `score >= 80` evaluates to `true`, executing its block and printing `Grade: B`. Remaining branches are skipped.
2. **Switch Statement (`dayOfWeek = 3`)**:
   - The `switch` expression evaluates the integer value of `dayOfWeek`.
   - It jumps directly to `case 3:` and executes `System.out.println("Day: Wednesday")`.
   - The `break` statement terminates the switch block, preventing execution from falling through to subsequent cases.

---

## 2. Real-World Program: ATM Cash Withdrawal System

**Filename:** `AtmWithdrawalSystem.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: AtmWithdrawalSystem.java
 * Teaches: Real-world control statements with multi-layered conditional logic (PIN verification, balance checks, transaction limits).
 * Example Input/Output:
 *   Input PIN: 1234, Withdraw: $200
 *   Output: Transaction Successful. Remaining Balance: $800
 */
public class AtmWithdrawalSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Account Details
        final int CORRECT_PIN = 1234;
        double accountBalance = 1000.00;
        final double DAILY_WITHDRAWAL_LIMIT = 500.00;

        System.out.println("==========================================");
        System.out.println("       WELCOME TO JAVA SECURE ATM         ");
        System.out.println("==========================================");

        // Step 1: PIN Authentication
        System.out.print("Enter your 4-digit PIN: ");
        int enteredPin = scanner.nextInt();

        if (enteredPin != CORRECT_PIN) {
            System.out.println("ERROR: Incorrect PIN. Access Denied.");
            scanner.close();
            return;
        }

        System.out.println("Authentication Successful!\n");

        // Step 2: Amount Input & Validation
        System.out.println("Current Balance: $" + accountBalance);
        System.out.print("Enter amount to withdraw: $");
        double withdrawAmount = scanner.nextDouble();

        // Control Statements for Transaction Processing
        if (withdrawAmount <= 0) {
            System.out.println("TRANSACTION FAILED: Withdrawal amount must be greater than zero.");
        } else if (withdrawAmount % 10 != 0) {
            System.out.println("TRANSACTION FAILED: Please enter an amount in multiples of 10.");
        } else if (withdrawAmount > DAILY_WITHDRAWAL_LIMIT) {
            System.out.println("TRANSACTION FAILED: Amount exceeds daily limit of $" + DAILY_WITHDRAWAL_LIMIT);
        } else if (withdrawAmount > accountBalance) {
            System.out.println("TRANSACTION FAILED: Insufficient account balance.");
        } else {
            // Process Transaction
            accountBalance -= withdrawAmount;
            System.out.println("\n--- TRANSACTION SUCCESSFUL ---");
            System.out.println("Dispensed: $" + withdrawAmount);
            System.out.println("Remaining Balance: $" + accountBalance);
        }

        System.out.println("Thank you for using Java Secure ATM!");
        scanner.close();
    }
}
```

### Code Explanation
1. **Authentication Gate (`enteredPin != CORRECT_PIN`)**:
   - Uses an early `if` condition to reject unauthorized PINs immediately, exiting the program before sensitive operations occur.
2. **Multi-Condition Validation Chain (`if-else if-else`)**:
   - **Positive Amount Check (`withdrawAmount <= 0`)**: Rejects zero or negative withdrawal requests.
   - **Denomination Check (`withdrawAmount % 10 != 0`)**: Uses modulus (`%`) to enforce bill denomination rules ($10 multiples).
   - **Limit Check (`withdrawAmount > DAILY_WITHDRAWAL_LIMIT`)**: Ensures compliance with security limits ($500 limit).
   - **Balance Check (`withdrawAmount > accountBalance`)**: Verifies sufficient funds prior to account deduction.
3. **Transaction Execution (`else` branch)**:
   - Runs strictly when all previous guard clauses evaluate to `false`, deducting the requested amount and updating the balance.
