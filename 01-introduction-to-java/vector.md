# Unit 1: Introduction to Java

## Vector

The `Vector` class (`java.util.Vector`) implements a dynamic, growable array of objects. Standard Java arrays have a fixed size set at allocation; a `Vector` expands or shrinks automatically as items are added or removed.

---

### Core Concepts

#### 1. Dynamic Resizing & Capacity Increment
- **`size()`**: The actual count of items currently stored in the vector.
- **`capacity()`**: The total memory slots allocated before `Vector` must reallocate a larger internal array.
- **Expansion Logic**: When `size()` reaches `capacity()`, `Vector` creates a larger internal array and copies elements over. By default, it doubles its capacity unless a custom `capacityIncrement` was passed to the constructor (`new Vector<>(initialCapacity, capacityIncrement)`).

#### 2. Thread Synchronization
- All core methods in `Vector` (such as `add()`, `remove()`, and `get()`) use the `synchronized` modifier.
- This makes `Vector` safe to access concurrently across multiple threads without manual locking, though it introduces a slight performance overhead compared to un-synchronized structures like `ArrayList`.

#### 3. Traversal Methods
- **Enhanced For-Loop**: `for (String item : vector)`
- **Standard Index Loop**: `for (int i = 0; i < vector.size(); i++)`
- **Legacy `Enumeration`**: `Enumeration<String> e = vector.elements();`

---

### Common Pitfalls

1. **Confusing `size()` with `capacity()`**:
   - `size()` tells you how many elements are currently in the vector, while `capacity()` tells you how many slots are allocated in memory. Attempting to call `.get(5)` when `size()` is 3 throws an `ArrayIndexOutOfBoundsException` even if `capacity()` is 10.
2. **Not using generics (`Vector<T>`)**:
   - Declaring `Vector v = new Vector();` without generic type arguments treats elements as `Object`. You must cast items manually when retrieving them. Always specify the type: `Vector<String> v = new Vector<>();`.

---

## 1. Demo: Vector Operations and Capacity Growth

### `VectorBasicsDemo.java`

```java
import java.util.Vector;
import java.util.Enumeration;

/**
 * Demonstrates Vector instantiation, dynamic capacity growth, element access, and Enumeration traversal.
 */
public class VectorBasicsDemo {
    public static void main(String[] args) {
        
        // Initial capacity = 3, capacity increment = 2
        System.out.println("=== 1. Initializing Vector ===");
        Vector<String> vec = new Vector<>(3, 2);

        System.out.println("Initial Size    : " + vec.size());     // 0
        System.out.println("Initial Capacity: " + vec.capacity()); // 3

        System.out.println("\n=== 2. Adding Elements ===");
        vec.add("Alpha");
        vec.add("Beta");
        vec.add("Gamma");
        System.out.println("Size after 3 elements: " + vec.size() + " | Capacity: " + vec.capacity());

        // Adding 4th element triggers expansion by +2
        vec.add("Delta");
        System.out.println("Size after 4 elements: " + vec.size() + " | Capacity: " + vec.capacity());

        System.out.println("\n=== 3. Element Access ===");
        System.out.println("First Element: " + vec.firstElement());
        System.out.println("Last Element : " + vec.lastElement());
        System.out.println("Element at index 2: " + vec.get(2));

        System.out.println("\n=== 4. Enumeration Traversal ===");
        Enumeration<String> en = vec.elements();
        while (en.hasMoreElements()) {
            System.out.println("Vector Item: " + en.nextElement());
        }

        vec.remove("Beta");
        System.out.println("\nAfter removing 'Beta', Size: " + vec.size());
    }
}
```

### Detailed Code Walkthrough

1. **Custom Capacity Initialization (`new Vector<>(3, 2)`)**:
   - Sets starting capacity to `3` and capacity growth increment to `2`.
   - Before adding elements, `vec.size()` is `0` and `vec.capacity()` is `3`.

2. **Triggering Dynamic Reallocation**:
   - Adding 3 items (`Alpha`, `Beta`, `Gamma`) fills all 3 allocated slots (`size = 3`, `capacity = 3`).
   - Adding the 4th item (`Delta`) causes `Vector` to expand by its custom increment of `2`. `capacity()` increases from `3` to `5`.

3. **Retrieval & Enumeration**:
   - `firstElement()` and `lastElement()` access boundary elements directly.
   - `vec.elements()` returns an `Enumeration`. `en.hasMoreElements()` tests if elements remain, and `en.nextElement()` fetches each element sequentially.

---

## 2. Real-World Program: Office Print Job Spooler

### `PrintJobQueueManager.java`

```java
import java.util.Vector;
import java.util.Scanner;

/**
 * Print job queue manager using Vector for document queuing and FIFO processing.
 *
 * Example:
 *   Add jobs: "Report.pdf", "Invoice.docx"
 *   Output: Processing Job: "Report.pdf", 1 job remaining.
 */
public class PrintJobQueueManager {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Vector<String> printQueue = new Vector<>();

        System.out.println("==========================================");
        System.out.println("   OFFICE PRINT JOB SPOOLER SYSTEM        ");
        System.out.println("==========================================");

        boolean running = true;

        while (running) {
            System.out.println("\n--- PRINT QUEUE MENU ---");
            System.out.println("1. Submit New Print Job");
            System.out.println("2. Process Next Print Job");
            System.out.println("3. View Pending Print Queue");
            System.out.println("4. Cancel Specific Job");
            System.out.println("5. Exit Spooler");
            System.out.print("Select Option (1-5): ");

            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1:
                    System.out.print("Enter document name (e.g. AnnualReport.pdf): ");
                    String docName = scanner.nextLine().trim();
                    if (!docName.isEmpty()) {
                        printQueue.add(docName);
                        System.out.println("SUCCESS: '" + docName + "' added to queue (Position #" + printQueue.size() + ")");
                    } else {
                        System.out.println("ERROR: Document name cannot be empty.");
                    }
                    break;

                case 2:
                    if (printQueue.isEmpty()) {
                        System.out.println("NOTICE: Print queue is currently empty.");
                    } else {
                        String jobToPrint = printQueue.remove(0);
                        System.out.println("PRINTING: Dispatching document '" + jobToPrint + "' to printer...");
                        System.out.println("Remaining Jobs in Queue: " + printQueue.size());
                    }
                    break;

                case 3:
                    System.out.println("\n--- CURRENT PENDING JOBS (" + printQueue.size() + ") ---");
                    if (printQueue.isEmpty()) {
                        System.out.println("No pending print jobs.");
                    } else {
                        for (int i = 0; i < printQueue.size(); i++) {
                            System.out.println("  Job #" + (i + 1) + ": " + printQueue.get(i));
                        }
                    }
                    break;

                case 4:
                    System.out.print("Enter document name to cancel: ");
                    String cancelDoc = scanner.nextLine().trim();
                    if (printQueue.remove(cancelDoc)) {
                        System.out.println("SUCCESS: Cancelled print job '" + cancelDoc + "'.");
                    } else {
                        System.out.println("ERROR: Job '" + cancelDoc + "' not found in queue.");
                    }
                    break;

                case 5:
                    System.out.println("Shutting down Print Spooler...");
                    running = false;
                    break;

                default:
                    System.out.println("INVALID OPTION: Please enter a number between 1 and 5.");
                    break;
            }
        }

        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **FIFO Queue Management**:
   - `printQueue.add(docName)` appends new print jobs to the end of the vector.
   - `printQueue.remove(0)` processes the oldest document at index `0` (First-In-First-Out queue behavior) and automatically shifts subsequent items left.

2. **Searching & Removing Elements**:
   - `printQueue.remove(cancelDoc)` searches the vector for a string matching `cancelDoc`. If found, it removes the element and returns `true`. If not found, it returns `false`.
