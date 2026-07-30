# Unit 1: Introduction to Java

## Looping Statements

Looping statements repeat a block of code while a specified condition evaluates to `true`. Loops eliminate repetitive code when processing arrays, performing mathematical iterations, or accepting user input until an exit command is given.

---

### Core Concepts

#### 1. `for` Loop (Definite Iteration)
- Best when you know the exact number of times a loop should run before starting.
- Syntactically combines initialization, loop condition, and variable update into a single line:
  ```java
  for (initialization; condition; update) {
      // Loop body
  }
  ```
- **Execution Flow**: Runs initialization once → checks condition → runs body if true → executes update → checks condition again.

#### 2. `while` Loop (Pre-test Indefinite Iteration)
- Best when the number of iterations depends on runtime events (such as user input).
- Evaluates the condition **before** executing the loop body. If the condition starts as `false`, the body never runs.

#### 3. `do-while` Loop (Post-test Indefinite Iteration)
- Evaluates the condition **after** executing the body.
- Guarantees the body executes **at least once**, making it suitable for menu displays where you want to show choices before checking if the user wants to quit.

#### 4. Loop Control Statements
- **`break`**: Exits the loop immediately, jumping control to the line after the loop's closing brace.
- **`continue`**: Skips the remaining code inside the loop body for the current iteration and jumps directly to the next condition test / update step.

---

### Common Pitfalls

1. **Infinite Loops**:
   - Forgetting to update counter variables inside a `while` loop (e.g. omitting `count++`) causes the condition to remain `true` forever, freezing the program.
2. **Off-by-One Errors**:
   - Writing `i <= 5` when counting indices for a 5-element array (indices 0 through 4) causes an out-of-bounds error.
3. **Placing a semicolon right after loop headers**:
   - Writing `for (int i=0; i<5; i++);` defines an empty loop body. The code block beneath it runs only once *after* the empty loop finishes counting.

---

## 1. Demo: Basic Looping Structures

### `LoopingDemo.java`

```java
/**
 * Demonstrates syntax for for, while, do-while, break, and continue.
 */
public class LoopingDemo {
    public static void main(String[] args) {
        
        // 1. For Loop
        System.out.println("=== 1. Standard For Loop (Counting 1 to 5) ===");
        for (int i = 1; i <= 5; i++) {
            System.out.println("Iteration: " + i);
        }

        // 2. While Loop with Continue
        System.out.println("\n=== 2. While Loop (Even Numbers up to 10) ===");
        int count = 0;
        while (count < 10) {
            count++;
            if (count % 2 != 0) {
                continue; // Skip odd numbers
            }
            System.out.println("Even number: " + count);
        }

        // 3. Do-While Loop with Break
        System.out.println("\n=== 3. Do-While Loop (Exit on Target) ===");
        int num = 1;
        do {
            System.out.println("Processing value: " + num);
            if (num == 3) {
                System.out.println("Target 3 reached. Exiting loop.");
                break;
            }
            num++;
        } while (num <= 5);
    }
}
```

### Detailed Code Walkthrough

1. **`for` Loop Counting**:
   - `int i = 1`: Declares loop variable `i` set to 1.
   - `i <= 5`: Checks if `i` is 5 or less.
   - `i++`: Increments `i` by 1 after each loop body run. Prints iterations 1 through 5, then terminates when `i` becomes 6.

2. **`while` Loop with `continue`**:
   - `int count = 0;`: Starts counter at 0.
   - `while (count < 10)`: Checks condition before entering loop.
   - `count++;`: Increments counter right at the top of the loop body.
   - `if (count % 2 != 0)`: Checks if the number is odd. If true, `continue;` skips `System.out.println(...)` and jumps to the `while (count < 10)` condition check. Only even numbers (2, 4, 6, 8, 10) are printed.

3. **`do-while` Loop with `break`**:
   - Runs `System.out.println("Processing value: " + num);` first.
   - When `num` reaches `3`, `if (num == 3)` evaluates to true.
   - `break;` terminates the loop immediately, skipping `num++` and bypassing subsequent checks for 4 and 5.

---

## 2. Real-World Program: Supermarket Checkout System

### `ShoppingCartCheckout.java`

```java
import java.util.Scanner;

/**
 * Sentinel-controlled checkout loop accumulating prices until the user inputs -1.
 *
 * Example:
 *   Input: 25.50, 40.00, -1
 *   Output: Total Items: 2, Subtotal: $65.50, Tax (8%): $5.24, Grand Total: $70.74
 */
public class ShoppingCartCheckout {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        double subtotal = 0.0;
        int itemCounter = 0;
        final double TAX_RATE = 0.08;

        System.out.println("==========================================");
        System.out.println("      SUPERMARKET CHECKOUT SYSTEM         ");
        System.out.println("==========================================");
        System.out.println("Enter item prices one by one.");
        System.out.println("Enter -1 to complete checkout.\n");

        while (true) {
            System.out.print("Enter item #" + (itemCounter + 1) + " price ($): ");
            double price = scanner.nextDouble();

            if (price == -1) {
                System.out.println("\nFinishing scanning items...");
                break;
            }

            if (price < 0) {
                System.out.println("INVALID PRICE: Price cannot be negative. Item skipped.");
                continue;
            }

            subtotal += price;
            itemCounter++;
            System.out.println("Added: $" + price + " | Current Subtotal: $" + subtotal);
        }

        System.out.println("\n==========================================");
        System.out.println("             RECEIPT SUMMARY              ");
        System.out.println("==========================================");

        if (itemCounter == 0) {
            System.out.println("No valid items scanned. Cart is empty.");
        } else {
            double taxAmount = subtotal * TAX_RATE;
            double grandTotal = subtotal + taxAmount;

            System.out.println("Total Items Scanned : " + itemCounter);
            System.out.printf("Subtotal            : $%.2f%n", subtotal);
            System.out.printf("Sales Tax (8%%)       : $%.2f%n", taxAmount);
            System.out.printf("Grand Total         : $%.2f%n", grandTotal);
        }

        System.out.println("==========================================");
        System.out.println("Thank you for shopping with us!");
        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **Sentinel Loop Setup (`while (true)`)**:
   - `while (true)` creates an intentional indefinite loop that runs until explicitly stopped by a `break` statement inside.
   - `-1` serves as a **sentinel value**—a special input value used to signal the end of data entry.

2. **Input Validation and Accumulation**:
   - `if (price == -1)`: Checks if the user typed the sentinel value. If so, `break;` exits the loop.
   - `if (price < 0)`: Checks for negative numbers. If typed by mistake, `continue;` skips `subtotal += price` and `itemCounter++`, asking for the next item price without corrupting checkout calculations.
   - `subtotal += price;`: Shorthand for `subtotal = subtotal + price;`. Adds each valid item cost to the running subtotal.

3. **Receipt Calculation**:
   - Computes `taxAmount = subtotal * 0.08` and `grandTotal = subtotal + taxAmount`.
   - Uses `System.out.printf("... $%.2f%n", ...)` to round dollar amounts to 2 decimal places.
