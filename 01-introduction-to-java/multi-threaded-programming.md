# Unit 1: Introduction to Java

## Multi-threaded Programming

**Multi-threaded Programming** allows a Java program to execute multiple threads of execution concurrently within a single process. Operating systems allocate CPU time slots across threads, enabling high-performance applications, smooth background operations, and responsive user interfaces.

---

### Core Concepts

#### 1. Detailed Thread Life Cycle States

```
[ NEW ] ---> start() ---> [ RUNNABLE ] <---> [ BLOCKED / WAITING / TIMED_WAITING ]
                               |
                        run() completes
                               v
                         [ TERMINATED ]
```

1. **NEW**: A thread object has been instantiated (`new Thread()`), but `.start()` has not been called yet.
2. **RUNNABLE**: Executing inside the JVM or waiting for a CPU time slot.
3. **BLOCKED**: Waiting to acquire an intrinsic monitor lock currently held by another thread.
4. **WAITING**: Paused indefinitely until another thread performs an action (via `Object.wait()` or `Thread.join()`).
5. **TIMED_WAITING**: Paused for a specified duration (via `Thread.sleep(ms)`, `Object.wait(ms)`, or `Thread.join(ms)`).
6. **TERMINATED**: Finished running its `run()` method or stopped due to an unhandled exception.

#### 2. Thread Creation Techniques

| Creation Method | Implementation Syntax | Trade-offs & Best Practices |
|-----------------|-----------------------|-----------------------------|
| **Extending `Thread` Class** | `class MyThread extends Thread` | Simple for small tests, but prevents extending any other class because Java does not support multiple class inheritance. |
| **Implementing `Runnable` Interface** | `class MyTask implements Runnable` | **Industry Standard**. Decouples task execution logic from thread management and allows extending another parent class. |

#### 3. Thread Synchronization & Race Conditions
- **Race Condition**: Occurs when two or more threads access shared mutable data concurrently, and at least one thread modifies it. Without coordination, operations overlap unpredictably, corrupting data.
- **Intrinsic Locks (Monitors)**: Every object in Java has an implicit monitor lock.
- **`synchronized` Keyword**:
  - **Synchronized Method**: Locks the entire method using the current instance (`this`).
  - **Synchronized Block**: Locks only a designated critical section of code (`synchronized(lock) { ... }`).
- **Inter-Thread Communication**:
  - `wait()`: Releases the object lock and pauses the current thread until another thread calls `notify()` or `notifyAll()`.
  - `notify()`: Wakes up a single thread waiting on the object lock.
  - `notifyAll()`: Wakes up all threads waiting on the object lock.
  - *Note: `wait()`, `notify()`, and `notifyAll()` must always be invoked inside a `synchronized` context.*

---

### Common Pitfalls

1. **Calling `.run()` instead of `.start()`**:
   - Calling `t.run()` executes the method synchronously on the current thread like a standard method call. Calling `t.start()` creates a new operating system thread to run `run()` asynchronously.
2. **Not synchronizing compound operations on shared data**:
   - Operations like `count++` are not atomic—they consist of three steps: read value, increment, write back. If two threads execute `count++` at the same time without `synchronized`, updates get overwritten.

---

## 1. Demo: Thread Creation and Lifecycle Transitions

### `MultithreadingBasicsDemo.java`

```java
/**
 * Demonstrates thread creation using Thread subclass and Runnable interface, Thread.sleep(), join(), and state tracking.
 */

class WorkerThread extends Thread {
    public WorkerThread(String threadName) {
        super(threadName);
    }

    @Override
    public void run() {
        System.out.println(" [" + getName() + "] Started execution. Initial State: " + getState());
        try {
            for (int i = 1; i <= 3; i++) {
                System.out.println(" [" + getName() + "] Processing step " + i);
                Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            System.out.println(" [" + getName() + "] Interrupted.");
        }
        System.out.println(" [" + getName() + "] Finished execution.");
    }
}

class TaskRunnable implements Runnable {
    @Override
    public void run() {
        String currentName = Thread.currentThread().getName();
        System.out.println(" [" + currentName + "] Runnable task executing...");
        try {
            Thread.sleep(800);
        } catch (InterruptedException e) {
            System.out.println(" [" + currentName + "] Task interrupted.");
        }
        System.out.println(" [" + currentName + "] Runnable task completed.");
    }
}

public class MultithreadingBasicsDemo {
    public static void main(String[] args) {
        
        System.out.println("==========================================");
        System.out.println("   THREAD CREATION & LIFECYCLE DEMO       ");
        System.out.println("==========================================");

        WorkerThread thread1 = new WorkerThread("Worker-Alpha");
        System.out.println("State of Worker-Alpha before start(): " + thread1.getState()); // NEW

        Thread thread2 = new Thread(new TaskRunnable(), "Task-Beta");

        System.out.println("\nStarting threads...");
        thread1.start();
        thread2.start();

        try {
            Thread.sleep(100);
            System.out.println("\n--- MID-EXECUTION STATE INSPECTION ---");
            System.out.println("State of Worker-Alpha: " + thread1.getState()); // TIMED_WAITING
            System.out.println("State of Task-Beta   : " + thread2.getState()); // TIMED_WAITING

            System.out.println("\nMain thread waiting for worker threads (join())...");
            thread1.join();
            thread2.join();

        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }

        System.out.println("\n--- FINAL STATE INSPECTION ---");
        System.out.println("State of Worker-Alpha after completion: " + thread1.getState()); // TERMINATED
        System.out.println("State of Task-Beta after completion   : " + thread2.getState()); // TERMINATED
        System.out.println("Main thread finished.");
    }
}
```

### Detailed Code Walkthrough

1. **Instantiation (`new WorkerThread(...)`)**:
   - `thread1.getState()` returns `NEW` because `.start()` has not been invoked yet.

2. **Starting Execution (`.start()`)**:
   - `thread1.start()` requests the JVM to allocate a new operating system thread and call `WorkerThread.run()`.

3. **Timed Waiting (`Thread.sleep(500)`)**:
   - Pauses execution for 500ms, shifting `thread1` into the `TIMED_WAITING` state and freeing the CPU for other threads.

4. **Thread Joining (`.join()`)**:
   - Calling `thread1.join()` pauses the `main` thread until `thread1` completes its `run()` method and enters `TERMINATED`.

---

## 2. Real-World Program: Concurrent Ticket Booking System

### `TicketBookingSystem.java`

```java
import java.util.Scanner;

/**
 * Ticket booking system using synchronized methods to prevent seat overbooking.
 *
 * Example:
 *   Alice (2 seats) and Bob (2 seats) request tickets when 3 seats are left.
 *   Output: Alice receives 2 seats. Bob is rejected or waits for restock.
 */

class CinemaHall {
    private int availableSeats;

    public CinemaHall(int initialSeats) {
        this.availableSeats = initialSeats;
    }

    public synchronized boolean bookSeats(String customerName, int requestedSeats) {
        System.out.println("\n>>> [" + customerName + "] Attempting to book " + requestedSeats + " seat(s). Available: " + availableSeats);

        if (requestedSeats <= availableSeats) {
            System.out.println("    [" + customerName + "] Seats available. Processing booking...");

            try {
                Thread.sleep(400);
            } catch (InterruptedException e) {
                System.out.println("    [" + customerName + "] Booking interrupted.");
            }

            availableSeats -= requestedSeats;
            System.out.println("SUCCESS: [" + customerName + "] Booked " + requestedSeats + " seat(s). Remaining: " + availableSeats);
            return true;

        } else {
            System.out.println("FAILURE: [" + customerName + "] Insufficient seats. Requested: " + requestedSeats + ", Available: " + availableSeats);
            return false;
        }
    }

    public synchronized void restockSeats(int newSeats) {
        System.out.println("\n==========================================");
        System.out.println(" ADMIN: Restocking " + newSeats + " seats into Cinema Hall.");
        this.availableSeats += newSeats;
        System.out.println(" ADMIN: New Available Seats: " + availableSeats);
        System.out.println("==========================================");

        notifyAll();
    }

    public synchronized int getAvailableSeats() {
        return availableSeats;
    }
}

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

        Thread user1 = new Thread(new BookingCustomerTask(hall, "Alice", 2), "Thread-Alice");
        Thread user2 = new Thread(new BookingCustomerTask(hall, "Bob", 2), "Thread-Bob");
        Thread user3 = new Thread(new BookingCustomerTask(hall, "Charlie", 1), "Thread-Charlie");

        System.out.println("\nLaunching concurrent requests: Alice (2), Bob (2), Charlie (1)...");

        user1.start();
        user2.start();
        user3.start();

        try {
            user1.join();
            user2.join();
            user3.join();
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }

        System.out.println("\nTriggering Admin Seat Restock...");
        hall.restockSeats(5);

        System.out.println("\nFinal Available Seats: " + hall.getAvailableSeats());
        System.out.println("Ticket Booking System Session Ended.");

        scanner.close();
    }
}
```

### Detailed Synchronization Walkthrough

1. **Race Condition Prevention (`synchronized boolean bookSeats(...)`)**:
   - If `Alice` and `Bob` check `availableSeats` (currently `3`) at the exact same microsecond without `synchronized`, both threads see `3 >= 2` as `true` and both proceed to deduct 2 seats (`3 - 2 - 2 = -1`), corrupting system state with an overbooking bug.
   - Adding `synchronized` locks the `CinemaHall` instance monitor. When `Thread-Alice` enters `bookSeats()`, `Thread-Bob` is placed into the `BLOCKED` state until Alice finishes and releases the lock.

2. **Critical Section**:
   - `availableSeats -= requestedSeats;` is the critical section. Locking ensures reading, checking, and modifying `availableSeats` happens atomically.

3. **Signaling (`notifyAll()`)**:
   - `restockSeats()` updates available seats and calls `notifyAll()`, waking up any threads waiting on the `CinemaHall` monitor.
