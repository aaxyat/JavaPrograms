# Unit 1: Introduction to Java

## Subheading: Vector

The `Vector` class in Java (`java.util.Vector`) implements a growable, dynamic array of objects. Unlike standard arrays with fixed size, a `Vector` expands automatically as elements are added. 

### Key Concepts
1. **Dynamic Resizing**: `Vector` grows its capacity automatically when the current storage limit is reached (doubles capacity by default unless a `capacityIncrement` is specified).
2. **Synchronization (Thread-Safety)**: All legacy operations in `Vector` are synchronized, making it safe for concurrent multi-threaded access without external locking.
3. **Size vs Capacity**:
   - `size()`: Number of actual elements currently stored in the vector.
   - `capacity()`: Total number of element slots allocated in memory before resizing is required.
4. **Traversal**: Can be traversed using standard loops, enhanced for-loops, iterators, or legacy `Enumeration` (`elements()`).

---

## 1. Demo Program: Vector Operations & Capacity Dynamics

**Filename:** `VectorBasicsDemo.java`

### Source Code
```java
import java.util.Vector;
import java.util.Enumeration;

/**
 * Program: VectorBasicsDemo.java
 * Teaches: Vector instantiation, dynamic resizing, size vs capacity, element access, and Enumeration traversal.
 * Usage: Demonstrates adding/removing elements and tracking vector capacity adjustments.
 */
public class VectorBasicsDemo {
    public static void main(String[] args) {
        
        // 1. Instantiating Vector with Initial Capacity = 3, Capacity Increment = 2
        System.out.println("=== 1. Initializing Vector ===");
        Vector<String> vec = new Vector<>(3, 2);

        System.out.println("Initial Size    : " + vec.size());     // 0
        System.out.println("Initial Capacity: " + vec.capacity()); // 3

        // 2. Adding Elements & Triggering Capacity Expansion
        System.out.println("\n=== 2. Adding Elements ===");
        vec.add("Alpha");
        vec.add("Beta");
        vec.add("Gamma");
        System.out.println("Size after 3 elements: " + vec.size() + " | Capacity: " + vec.capacity());

        // Exceeding capacity trigger expansion (+2)
        vec.add("Delta");
        System.out.println("Size after 4 elements: " + vec.size() + " | Capacity: " + vec.capacity());

        // 3. Element Access & Modification
        System.out.println("\n=== 3. Element Access ===");
        System.out.println("First Element: " + vec.firstElement());
        System.out.println("Last Element : " + vec.lastElement());
        System.out.println("Element at index 2: " + vec.get(2));

        // 4. Legacy Traversal using Enumeration
        System.out.println("\n=== 4. Traversing via Enumeration ===");
        Enumeration<String> en = vec.elements();
        while (en.hasMoreElements()) {
            System.out.println("Vector Item: " + en.nextElement());
        }

        // 5. Element Removal
        vec.remove("Beta");
        System.out.println("\nAfter removing 'Beta', Size: " + vec.size());
    }
}
```

### Code Explanation
1. **Capacity Tracking (`size()` vs `capacity()`)**:
   - Initialized with capacity `3` and increment `2`. When the 4th element `"Delta"` is added, the vector automatically increases its capacity to `5` (`3 + 2`).
2. **First / Last / Index Access (`firstElement()`, `lastElement()`, `get(2)`)**:
   - Provides convenience methods to retrieve elements at specific structural positions.
3. **Enumeration Traversal (`vec.elements()`)**:
   - Demonstrates the legacy `Enumeration` interface (`hasMoreElements()`, `nextElement()`) designed specifically for `Vector`.

---

## 2. Real-World Program: Office Print Job Spooler System

**Filename:** `PrintJobQueueManager.java`

### Source Code
```java
import java.util.Vector;
import java.util.Scanner;

/**
 * Program: PrintJobQueueManager.java
 * Teaches: Building a thread-safe print job spooler using Vector for concurrent document queuing and processing.
 * Example Input/Output:
 *   Add jobs: "Report.pdf", "Invoice.docx"
 *   Output: Processing Job: "Report.pdf", 1 job remaining in queue.
 */
public class PrintJobQueueManager {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Synchronized Vector holding pending print document tasks
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
            scanner.nextLine(); // Consume newline

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
                        // Process and remove the first document (FIFO order)
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

### Code Explanation
1. **Thread-Safe Task Queue (`Vector<String> printQueue`)**:
   - `Vector` provides built-in thread safety for concurrent document submissions and print processing.
2. **First-In-First-Out (FIFO) Processing (`printQueue.remove(0)`)**:
   - Removes the job at index `0`, simulating a queue where the oldest submitted document gets printed first.
3. **Dynamic Queue Inspection & Cancellation (`remove(cancelDoc)`)**:
   - Searches and removes specific document requests dynamically, updating queue order automatically.
