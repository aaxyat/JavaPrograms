# Unit 1: Introduction to Java

## Subheading: Multi-threaded Programming

**Multi-threaded Programming** allows a Java program to execute multiple threads of execution concurrently within a single process. Threads share memory space while executing tasks independently, enabling high-performance applications, responsive user interfaces, and efficient CPU utilization.

---

### Detailed Conceptual Breakdown

#### 1. Thread Life Cycle States
A Java thread passes through 6 distinct states in its lifecycle, managed by the JVM and OS thread scheduler:

```
[ NEW ] ---> start() ---> [ RUNNABLE ] <---> [ BLOCKED / WAITING / TIMED_WAITING ]
                               |
                        run() completes
                               v
                         [ TERMINATED ]
```

1. **NEW**: A thread object has been instantiated (`new Thread()`), but `start()` has not yet been invoked.
2. **RUNNABLE**: The thread is ready or actively executing code inside the JVM (may be executing or waiting for CPU time slice allocation).
3. **BLOCKED**: The thread is waiting to acquire a monitor lock (`synchronized`) currently held by another thread.
4. **WAITING**: The thread is waiting indefinitely for another thread to perform a specific action (e.g., waiting after calling `Object.wait()` or `Thread.join()`).
5. **TIMED_WAITING**: The thread is waiting for a specified period (e.g., after calling `Thread.sleep(ms)`, `Object.wait(timeout)`, or `Thread.join(timeout)`).
6. **TERMINATED**: The thread has finished executing its `run()` method or exited due to an unhandled error.

---

#### 2. Thread Creation Mechanisms

Java provides two primary techniques to create threads:

| Technique | Description | Best Practice |
|-----------|-------------|---------------|
| **Extending `Thread` Class** | Inherit from `java.lang.Thread` and override `run()`. | Use only for simple standalone tasks; prevents extending any other class due to single inheritance. |
| **Implementing `Runnable` Interface** | Implement `java.lang.Runnable` interface and pass instance to `new Thread(runnable)`. | **Preferred Industry Standard**. Decouples task logic from thread execution mechanics and allows class inheritance. |

---

#### 3. Thread Synchronization & Race Conditions

When multiple threads access shared mutable data concurrently without synchronization, a **Race Condition** occurs, leading to corrupted data and unpredictable behavior.

- **Intrinsic Locks (Monitors)**: Every Java object has an implicit monitor lock.
- **`synchronized` Keyword**:
  - **Synchronized Method**: Locks the entire method on `this` object instance (or class object for static methods).
  - **Synchronized Block**: Locks only a critical section of code using a specified lock object (`synchronized(lockObj) { ... }`).
- **Inter-Thread Communication (`wait()`, `notify()`, `notifyAll()`)**:
  - `wait()`: Releases the object lock and suspends the current thread until notified.
  - `notify()`: Wakes up a single thread waiting on the object monitor.
  - `notifyAll()`: Wakes up all threads waiting on the object monitor.
  - *Must always be called from within a `synchronized` context.*

---

## 1. Demo Program: Thread Creation & Lifecycle Transitions

**Filename:** `MultithreadingBasicsDemo.java`

### Source Code
```java
/**
 * Program: MultithreadingBasicsDemo.java
 * Teaches: Thread creation via Thread class and Runnable interface, Thread.sleep(), join(), and inspecting Thread states.
 * Usage: Demonstrates concurrent execution flow, thread joining, and state monitoring.
 */

// Approach 1: Extending Thread Class
class WorkerThread extends Thread {
    public WorkerThread(String threadName) {
        super(threadName); // Set thread name
    }

    @Override
    public void run() {
        System.out.println(" [" + getName() + "] Started execution. Initial State: " + getState());
        try {
            for (int i = 1; i <= 3; i++) {
                System.out.println(" [" + getName() + "] Processing step " + i);
                Thread.sleep(500); // Transitions thread into TIMED_WAITING state for 500ms
            }
        } catch (InterruptedException e) {
            System.out.println(" [" + getName() + "] Interrupted!");
        }
        System.out.println(" [" + getName() + "] Finished execution.");
    }
}

// Approach 2: Implementing Runnable Interface (Preferred)
class TaskRunnable implements Runnable {
    @Override
    public void run() {
        String currentName = Thread.currentThread().getName();
        System.out.println(" [" + currentName + "] Runnable task executing...");
        try {
            Thread.sleep(800); // Simulate background work
        } catch (InterruptedException e) {
            System.out.println(" [" + currentName + "] Task interrupted!");
        }
        System.out.println(" [" + currentName + "] Runnable task completed.");
    }
}

public class MultithreadingBasicsDemo {
    public static void main(String[] args) {
        
        System.out.println("==========================================");
        System.out.println("   THREAD CREATION & LIFECYCLE DEMO       ");
        System.out.println("==========================================");

        // 1. Thread Creation via Thread Subclass
        WorkerThread thread1 = new WorkerThread("Worker-Alpha");

        // Inspect state before start()
        System.out.println("State of Worker-Alpha before start(): " + thread1.getState()); // NEW

        // 2. Thread Creation via Runnable Interface
        Thread thread2 = new Thread(new TaskRunnable(), "Task-Beta");

        // Starting Threads (Moves states from NEW to RUNNABLE)
        System.out.println("\nStarting threads...");
        thread1.start();
        thread2.start();

        // 3. Inspecting States while Threads run
        try {
            Thread.sleep(100); // Main thread sleeps briefly to let worker threads transition
            System.out.println("\n--- MID-EXECUTION STATE INSPECTION ---");
            System.out.println("State of Worker-Alpha: " + thread1.getState()); // Expected: TIMED_WAITING
            System.out.println("State of Task-Beta   : " + thread2.getState()); // Expected: TIMED_WAITING

            // 4. Thread Joining (Main thread waits for worker threads to complete)
            System.out.println("\nMain thread waiting for worker threads to complete (join())...");
            thread1.join(); // Blocks main thread until thread1 finishes
            thread2.join(); // Blocks main thread until thread2 finishes

        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted!");
        }

        // Final State Check
        System.out.println("\n--- FINAL STATE INSPECTION ---");
        System.out.println("State of Worker-Alpha after completion: " + thread1.getState()); // TERMINATED
        System.out.println("State of Task-Beta after completion   : " + thread2.getState()); // TERMINATED
        System.out.println("Main thread execution finished.");
    }
}
```

### In-Depth Code Explanation

1. **Thread Instantiation (`new WorkerThread("Worker-Alpha")`)**:
   - Creating `thread1` allocates memory for the thread object. At this stage, `thread1.getState()` returns `NEW` because the underlying operating system thread has not been created yet.
2. **Starting Execution (`thread1.start()`)**:
   - Calling `.start()` notifies the JVM to create a native OS thread and schedule its `.run()` method for execution. Calling `.run()` directly would execute on the current thread instead of spawning a new thread!
3. **Timed Waiting State (`Thread.sleep(500)`)**:
   - Inside the loop, `Thread.sleep(500)` forces the worker thread to pause execution for 500 milliseconds. During this pause, the thread transitions into the `TIMED_WAITING` state, relinquishing CPU execution time.
4. **Synchronization Control (`thread1.join()`)**:
   - The `main` thread invokes `thread1.join()`. This causes the `main` thread to pause its own execution until `thread1` completely finishes its `.run()` method and enters the `TERMINATED` state.

---

## 2. Real-World Program: Concurrent Ticket Booking System

**Filename:** `TicketBookingSystem.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: TicketBookingSystem.java
 * Teaches: Synchronized methods/blocks, monitor lock acquisition, preventing race conditions, and wait/notify inter-thread communication.
 * Example Input/Output:
 *   2 Users (User-A requesting 2 seats, User-B requesting 2 seats) on a system with 3 remaining seats.
 *   Output: User-A successfully books 2 seats (1 remaining). User-B request fails or waits for restocking.
 */

// Shared Resource Class representing a Cinema Theater Hall
class CinemaHall {
    private int availableSeats;

    public CinemaHall(int initialSeats) {
        this.availableSeats = initialSeats;
    }

    // Synchronized Method: Ensures only ONE thread can book seats at a time
    public synchronized boolean bookSeats(String customerName, int requestedSeats) {
        System.out.println("\n>>> [" + customerName + "] Attempting to book " + requestedSeats + " seat(s). Available: " + availableSeats);

        // Check seat availability inside critical section
        if (requestedSeats <= availableSeats) {
            System.out.println("    [" + customerName + "] Seats available! Processing booking...");

            try {
                // Simulate processing/payment delay
                Thread.sleep(400);
            } catch (InterruptedException e) {
                System.out.println("    [" + customerName + "] Booking process interrupted.");
            }

            availableSeats -= requestedSeats;
            System.out.println("SUCCESS: [" + customerName + "] Booked " + requestedSeats + " seat(s). Remaining Seats: " + availableSeats);
            return true;

        } else {
            System.out.println("FAILURE: [" + customerName + "] Insufficient seats! Requested: " + requestedSeats + ", Available: " + availableSeats);
            return false;
        }
    }

    // Synchronized Method for Restocking Seats + Inter-thread Notification
    public synchronized void restockSeats(int newSeats) {
        System.out.println("\n==========================================");
        System.out.println(" ADMIN: Restocking " + newSeats + " additional seats into Cinema Hall.");
        this.availableSeats += newSeats;
        System.out.println(" ADMIN: New Available Seats Count: " + availableSeats);
        System.out.println("==========================================");

        // Notify any waiting threads that new seats are available
        notifyAll();
    }

    public synchronized int getAvailableSeats() {
        return availableSeats;
    }
}

// Runnable Customer Task attempting to book seats concurrently
class BookingCustomerTask implements Runnable {
    private CinemaHall hall;
    private String customerName;
    private int seatsToBook;

    public BookingCustomerTask(CinemaHall hall, String customerName, int seatsToBook) {
        this.hall = hall;
        this.customerName = customerName;
        this.seatsToBook = seatsToBook;
    }

    @Override
    public void run() {
        hall.bookSeats(customerName, seatsToBook);
    }
}

public class TicketBookingSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("     REAL-TIME TICKET BOOKING SYSTEM      ");
        System.out.println("==========================================");

        int initialSeats = 3;
        CinemaHall hall = new CinemaHall(initialSeats);
        System.out.println("Theater Initialized with " + initialSeats + " available seats.");

        // Creating multiple customer threads attempting to book simultaneously
        Thread user1 = new Thread(new BookingCustomerTask(hall, "Alice", 2), "Thread-Alice");
        Thread user2 = new Thread(new BookingCustomerTask(hall, "Bob", 2), "Thread-Bob");
        Thread user3 = new Thread(new BookingCustomerTask(hall, "Charlie", 1), "Thread-Charlie");

        System.out.println("\nLaunching concurrent booking requests from Alice (2 seats), Bob (2 seats), Charlie (1 seat)...");

        // Concurrent Execution Launch
        user1.start();
        user2.start();
        user3.start();

        // Wait for all user booking threads to complete
        try {
            user1.join();
            user2.join();
            user3.join();
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }

        // Admin Restocking Simulation
        System.out.println("\nTriggering Admin Seat Restock...");
        hall.restockSeats(5);

        System.out.println("\nFinal Available Seats: " + hall.getAvailableSeats());
        System.out.println("Ticket Booking System Session Ended.");

        scanner.close();
    }
}
```

### In-Depth Code & Synchronization Breakdown

#### 1. Why Synchronization is Essential (`synchronized void bookSeats(...)`)
- Without `synchronized`, if `Alice` and `Bob` check `availableSeats` (currently `3`) at the exact same microsecond:
  1. Both see `3` available seats and determine they can each book `2` seats.
  2. Both deduct `2` seats (`3 - 2 = 1` then `1 - 2 = -1`), leaving `availableSeats` corrupted at `-1` (Overbooking bug).
- Adding `synchronized` forces `bookSeats()` to acquire the **intrinsic monitor lock** of the `CinemaHall` object. Only **one thread** can hold this lock at any given time. If `Thread-Alice` enters `bookSeats()`, `Thread-Bob` is placed into the `BLOCKED` state until Alice finishes and releases the lock.

#### 2. Critical Section Guarding
- The statement `availableSeats -= requestedSeats;` inside `bookSeats()` is the **critical section**. Guarding this line ensures atomic execution—meaning read, check, and update steps happen as an indivisible unit.

#### 3. Inter-Thread Communication (`notifyAll()`)
- When `hall.restockSeats(5)` is invoked, it updates the available seat count and executes `notifyAll()`.
- If any customer thread was waiting on the `hall` object monitor (e.g. paused via `wait()`), `notifyAll()` wakes up all waiting threads so they can re-evaluate the seat condition.

#### 4. Thread Join & Termination Sequence
- `user1.join()`, `user2.join()`, and `user3.join()` guarantee that the main administrative thread will not proceed to print summary reports until all customer booking attempts have finished processing.
