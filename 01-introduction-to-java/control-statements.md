# Unit 1: Introduction to Java

## Control Statements

Control statements dictate the order in which Java executes statements. Without control statements, Java runs code sequentially from top to bottom. Control statements let you skip lines, choose between branches, or exit early based on conditions.

---

### Core Concepts

#### 1. Decision-Making Statements
- **`if` Statement**: Evaluates a boolean expression. Runs the code block only if the expression evaluates to `true`.
- **`if-else` Statement**: Provides an alternative branch if the `if` condition evaluates to `false`.
- **`else-if` Ladder**: Chains multiple conditions together. Java tests each condition sequentially until it finds one that is `true`. Once a branch executes, Java skips all remaining `else-if` and `else` blocks.
- **`switch` Statement**: Compares a single variable against multiple discrete values (`case` labels). Ideal when checking one variable against fixed options like integers, characters, or strings.

#### 2. Branching & Exit Control
- **`break`**: Exits a `switch` block immediately. Without `break`, execution falls through into subsequent `case` blocks regardless of whether their labels match.
- **`return`**: Exits the current method immediately. In `main()`, calling `return` terminates the program.

---

### Common Pitfalls

1. **Using `=` instead of `==` in conditions**:
   - `if (score = 80)` assigns `80` to `score` and fails to compile in Java because `80` is an `int`, not a `boolean`. Always use `==` to test equality.
2. **Forgetting `break` in `switch` statements**:
   - Omitting `break` causes **fall-through execution**, where code in matching and subsequent cases runs unintentionally.
3. **Missing curly braces `{}`**:
   - Without braces, an `if` block controls only the single line immediately following it. Always use braces even for one-line bodies to avoid logic bugs when adding lines later.

---

## 1. Demo: Basic Control Flow

### `ControlFlowDemo.java`

```java
/**
 * Demonstrates basic decision-making using if-else ladders and switch statements.
 */
public class ControlFlowDemo {
    public static void main(String[] args) {
        int score = 85;

        // 1. If-Else Ladder
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

        // 2. Switch Statement
        System.out.println("\n=== 2. Switch Statement ===");
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

### Explanation

1. **Grade Evaluation with `if-else if-else`**:
   - `int score = 85;`: Creates an integer variable `score`.
   - `if (score >= 90)`: Java checks if 85 is greater than or equal to 90. It is false, so Java moves to the next `else if`.
   - `else if (score >= 80)`: Java checks if 85 is greater than or equal to 80. This is true. Java runs `System.out.println("Grade: B");` and skips the remaining `else if` (`score >= 70`) and `else` blocks completely.

2. **Day Selection with `switch`**:
   - `switch (dayOfWeek)`: Evaluates `dayOfWeek`, which equals `3`.
   - Java matches `case 3:`, prints `"Day: Wednesday"`, and hits `break;`.
   - `break;` immediately jumps execution past the end of the `switch` block. If `break` were missing, Java would also print `"Day: Thursday"`.

---

## 2. Real-World Program: ATM Cash Withdrawal System

### `AtmWithdrawalSystem.java`

```java
import java.util.Scanner;

/**
 * ATM Cash Withdrawal validation using multi-layered conditional logic.
 *
 * Example:
 *   Input PIN: 1234, Amount: $200
 *   Output: Transaction Successful. Remaining Balance: $800.00
 */
public class AtmWithdrawalSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

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

        // Control Statements for Transaction Validation
        if (withdrawAmount <= 0) {
            System.out.println("TRANSACTION FAILED: Withdrawal amount must be greater than zero.");
        } else if (withdrawAmount % 10 != 0) {
            System.out.println("TRANSACTION FAILED: Please enter an amount in multiples of 10.");
        } else if (withdrawAmount > DAILY_WITHDRAWAL_LIMIT) {
            System.out.println("TRANSACTION FAILED: Amount exceeds daily limit of $" + DAILY_WITHDRAWAL_LIMIT);
        } else if (withdrawAmount > accountBalance) {
            System.out.println("TRANSACTION FAILED: Insufficient account balance.");
        } else {
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

### Explanation

1. **Authentication Guard (`if (enteredPin != CORRECT_PIN)`)**:
   - Rejects wrong PINs immediately. Calling `return;` exits `main()`, stopping the user from reaching balance check code.

2. **Validation Chain**:
   - **`withdrawAmount <= 0`**: Prevents users from entering zero or negative withdrawal numbers.
   - **`withdrawAmount % 10 != 0`**: Uses the modulus operator (`%`) to check if the requested amount is a multiple of 10. If the remainder is non-zero (e.g. $25), the ATM refuses to dispense cash.
   - **`withdrawAmount > DAILY_WITHDRAWAL_LIMIT`**: Enforces security policies limiting daily withdrawals to $500.
   - **`withdrawAmount > accountBalance`**: Prevents accounts from going negative.

3. **Transaction Execution (`else` block)**:
   - Runs only when every validation check passes. It subtracts `withdrawAmount` from `accountBalance` and displays the updated balance.
