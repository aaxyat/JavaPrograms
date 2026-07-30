# Unit 1: Introduction to Java

## Subheading: Looping

Looping statements allow a program to execute a block of code repeatedly as long as a specified condition remains `true`. Loops eliminate redundant code when processing collections, performing calculations, or handling repetitive user interaction.

### Key Concepts
1. **`for` Loop**: Ideal when the exact number of iterations is known beforehand (definite loops). Uses initialization, condition, and update expressions in a single line.
2. **`while` Loop**: Best suited when the number of iterations depends on dynamic conditions (indefinite loops). Checks the condition *before* executing the loop body.
3. **`do-while` Loop**: Similar to `while`, but guarantees that the loop body executes *at least once* because the condition is evaluated at the end.
4. **Loop Control Statements**:
   - `break`: Immediately exits the loop regardless of the condition.
   - `continue`: Skips the remaining statements in the current iteration and jumps directly to the next iteration.

---

## 1. Demo Program: Looping Basics

**Filename:** `LoopingDemo.java`

### Source Code
```java
/**
 * Program: LoopingDemo.java
 * Teaches: Fundamental syntax and control flow of for, while, do-while, break, and continue.
 * Usage: Demonstrates counter-based iteration, condition-based iteration, and loop jumping.
 */
public class LoopingDemo {
    public static void main(String[] args) {
        
        // 1. For Loop Demo
        System.out.println("=== 1. Standard For Loop (Counting 1 to 5) ===");
        for (int i = 1; i <= 5; i++) {
            System.out.println("Iteration: " + i);
        }

        // 2. While Loop Demo with Continue
        System.out.println("\n=== 2. While Loop (Print Even Numbers up to 10) ===");
        int count = 0;
        while (count < 10) {
            count++;
            if (count % 2 != 0) {
                continue; // Skip odd numbers
            }
            System.out.println("Even number: " + count);
        }

        // 3. Do-While Loop Demo with Break
        System.out.println("\n=== 3. Do-While Loop (Exit on target value) ===");
        int num = 1;
        do {
            System.out.println("Processing value: " + num);
            if (num == 3) {
                System.out.println("Target value 3 reached! Breaking loop.");
                break;
            }
            num++;
        } while (num <= 5);
    }
}
```

### Code Explanation
1. **`for` Loop (`int i = 1; i <= 5; i++`)**:
   - Initializes `i` to `1`. Checks `i <= 5`. Runs `System.out.println`, then increments `i` by `1`. Stops when `i` reaches `6`.
2. **`while` Loop with `continue` (`count < 10`)**:
   - `count` is incremented on each iteration.
   - The condition `count % 2 != 0` identifies odd numbers and executes `continue`, skipping the print statement and jumping to the next check.
3. **`do-while` Loop with `break` (`num <= 5`)**:
   - Executes the block first. When `num == 3`, the `break` statement halts the loop immediately, skipping remaining iterations (`num = 4` and `num = 5`).

---

## 2. Real-World Program: Shopping Cart & Checkout System

**Filename:** `ShoppingCartCheckout.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: ShoppingCartCheckout.java
 * Teaches: Using sentinel-controlled while loops for dynamic user input, running totals, and batch item processing.
 * Example Input/Output:
 *   Input prices: 25.50, 40.00, -1 (to stop)
 *   Output: Total Items: 2, Subtotal: $65.50, Total with Tax (8%): $70.74
 */
public class ShoppingCartCheckout {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        double subtotal = 0.0;
        int itemCounter = 0;
        final double TAX_RATE = 0.08; // 8% Sales Tax

        System.out.println("==========================================");
        System.out.println("      SUPERMARKET CHECKOUT SYSTEM         ");
        System.out.println("==========================================");
        System.out.println("Enter item prices one by one.");
        System.out.println("Enter -1 when you are finished scanning items.\n");

        // Sentinel-Controlled While Loop
        while (true) {
            System.out.print("Enter item #" + (itemCounter + 1) + " price ($): ");
            double price = scanner.nextDouble();

            // Sentinel check (Exit condition)
            if (price == -1) {
                System.out.println("\nFinishing scanning items...");
                break;
            }

            // Input Validation using simple guard
            if (price < 0) {
                System.out.println("INVALID PRICE: Price cannot be negative! Item skipped.");
                continue;
            }

            // Accumulate running totals
            subtotal += price;
            itemCounter++;
            System.out.println("Added: $" + price + " | Current Subtotal: $" + subtotal);
        }

        // Output summary using loops & conditional summary logic
        System.out.println("\n==========================================");
        System.out.println("             RECEIPT SUMMARY              ");
        System.out.println("==========================================");

        if (itemCounter == 0) {
            System.out.println("No valid items scanned. Cart is empty!");
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

### Code Explanation
1. **Sentinel-Controlled Loop (`while (true)`)**:
   - The loop runs continuously to accept an arbitrary number of items until the user inputs the sentinel value `-1`.
2. **Sentinel Guard Clause (`price == -1`)**:
   - When `-1` is entered, `break` triggers, immediately halting item entry and proceeding to receipt calculation.
3. **Validation Guard (`price < 0`)**:
   - Prevents invalid negative entries. `continue` skips the accumulator statements so invalid inputs don't corrupt `subtotal` or `itemCounter`.
4. **Running Total Accumulation (`subtotal += price`)**:
   - In each valid iteration, `price` is added to `subtotal` and `itemCounter` is incremented by `1`.
