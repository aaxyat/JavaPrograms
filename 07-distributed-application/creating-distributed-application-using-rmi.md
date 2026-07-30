# Unit 7: Distributed Application

## Creating Distributed Application using RMI

Creating a complete **Java RMI Distributed Application** involves building an end-to-end multi-JVM architecture: defining a remote interface, implementing the service, starting the RMI Registry, binding the service on a server, and invoking remote methods from a client.

---

### Core Concepts

#### 1. The 5-Step RMI Application Development Lifecycle

```
  Step 1: Define Remote Interface         Step 2: Implement Remote Interface
  (Extends java.rmi.Remote)               (Extends UnicastRemoteObject)
  ┌──────────────────────────────┐        ┌──────────────────────────────┐
  │ public interface MyService   │        │ public class MyServiceImpl   │
  │   extends Remote { ... }     │        │   extends UnicastRemoteObject│
  └──────────────┬───────────────┘        └──────────────┬───────────────┘
                 │                                       │
                 └───────────────────┬───────────────────┘
                                     │
                                     ▼
  Step 3: Create Server & Bind Service    Step 4: Create Client & Lookup Stub
  (LocateRegistry.createRegistry / rebind) (Naming.lookup("rmi://..."))
  ┌──────────────────────────────┐        ┌──────────────────────────────┐
  │ LocateRegistry.createRegistry│        │ MyService stub = (MyService) │
  │ Naming.rebind("Service", obj)│        │   Naming.lookup("Service");  │
  └──────────────────────────────┘        └──────────────────────────────┘
```

| Step | Action | Key Java APIs |
|------|--------|---------------|
| **Step 1** | Define Remote Interface | `public interface Service extends Remote` |
| **Step 2** | Implement Remote Service Class | `public class ServiceImpl extends UnicastRemoteObject implements Service` |
| **Step 3** | Build RMI Server & Bind Service | `LocateRegistry.createRegistry(1099); Naming.rebind("ServiceName", service);` |
| **Step 4** | Build RMI Client & Lookup Stub | `Service stub = (Service) Naming.lookup("rmi://localhost:1099/ServiceName");` |
| **Step 5** | Launch Application | Run Server JVM first, then launch Client JVM. |

---

### Common Pitfalls

1. **`NotBoundException` during Client Lookup**:
   - Calling `Naming.lookup("rmi://localhost:1099/MyService")` throws a `NotBoundException` if the name string does not match the exact case-sensitive string passed to `Naming.rebind("MyService", obj)` on the server.
2. **`UnmarshalException` due to Non-Serializable Parameters**:
   - All object arguments passed into remote methods must implement `java.io.Serializable`. Passing non-serializable objects throws a `MarshalException`.
3. **Attempting to Instantiate Remote Interfaces Directly**:
   - Clients must retrieve remote stub proxies using `Naming.lookup()`. Calling `new MyService()` on the client throws a compilation error because `MyService` is an interface.

---

## 1. Demo System: Remote Math Compute Service

### `ComputeService.java` (Remote Interface)

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface ComputeService extends Remote {
    double calculateSquareRoot(double number) throws RemoteException;
    long calculateFactorial(int number) throws RemoteException;
}
```

### `ComputeServiceImpl.java` (Service Implementation)

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class ComputeServiceImpl extends UnicastRemoteObject implements ComputeService {

    public ComputeServiceImpl() throws RemoteException {
        super();
    }

    @Override
    public double calculateSquareRoot(double number) throws RemoteException {
        System.out.println("[SERVER] Calculating square root for: " + number);
        if (number < 0) throw new IllegalArgumentException("Cannot calculate square root of negative number");
        return Math.sqrt(number);
    }

    @Override
    public long calculateFactorial(int number) throws RemoteException {
        System.out.println("[SERVER] Calculating factorial for: " + number);
        if (number < 0 || number > 20) {
            throw new IllegalArgumentException("Number must be between 0 and 20");
        }
        long factorial = 1;
        for (int i = 1; i <= number; i++) {
            factorial *= i;
        }
        return factorial;
    }
}
```

### `ComputeServer.java` (RMI Server Application)

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.Naming;

public class ComputeServer {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     RMI COMPUTE SERVER INITIALIZATION    ");
        System.out.println("==========================================");

        try {
            // 1. Programprogrammatically start RMI Registry on port 1099
            LocateRegistry.createRegistry(1099);
            System.out.println("RMI Naming Registry started on port 1099.");

            // 2. Instantiate Remote Object Implementation
            ComputeService service = new ComputeServiceImpl();

            // 3. Bind Service in RMI Registry
            Naming.rebind("rmi://localhost:1099/ComputeService", service);
            System.out.println("ComputeService bound successfully to RMI Registry!");
            System.out.println("Server ready to accept remote client calls...");

        } catch (Exception e) {
            System.err.println("Server Startup Error: " + e.getMessage());
        }
    }
}
```

### `ComputeClient.java` (RMI Client Application)

```java
import java.rmi.Naming;

public class ComputeClient {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     RMI COMPUTE CLIENT APPLICATION       ");
        System.out.println("==========================================");

        try {
            // 1. Lookup Remote Service Stub from RMI Registry
            String registryUrl = "rmi://localhost:1099/ComputeService";
            System.out.println("Looking up service at: " + registryUrl);
            ComputeService comp = (ComputeService) Naming.lookup(registryUrl);

            System.out.println("Remote Stub acquired successfully!\n");

            // 2. Execute Remote Method Calls
            double sqrtResult = comp.calculateSquareRoot(144.0);
            System.out.println("Remote Call -> calculateSquareRoot(144.0) = " + sqrtResult);

            long factResult = comp.calculateFactorial(10);
            System.out.println("Remote Call -> calculateFactorial(10)     = " + factResult);

            System.out.println("==========================================");

        } catch (Exception e) {
            System.err.println("Client RMI Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Remote Interface & Exception Declaration**:
   - `ComputeService` extends `Remote`, declaring `throws RemoteException` on all methods.

2. **Server Registration (`LocateRegistry.createRegistry`)**:
   - `ComputeServer` creates an in-process RMI registry on port `1099` via `LocateRegistry.createRegistry(1099)` and binds `ComputeServiceImpl` via `Naming.rebind()`.

3. **Client Stub Acquisition (`Naming.lookup`)**:
   - `ComputeClient` calls `Naming.lookup()` to download the `ComputeService` stub proxy and executes `calculateSquareRoot()` and `calculateFactorial()` across JVM boundaries.

---

## 2. Practical System: Enterprise Distributed Banking Portal

### `BankService.java` (Remote Interface)

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface BankService extends Remote {
    double checkBalance(String accountNumber) throws RemoteException;
    boolean deposit(String accountNumber, double amount) throws RemoteException;
    boolean withdraw(String accountNumber, double amount) throws RemoteException;
}
```

### `BankServiceImpl.java` (Service Implementation)

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class BankServiceImpl extends UnicastRemoteObject implements BankService {

    private final Map<String, Double> accountBalances = new ConcurrentHashMap<>();

    public BankServiceImpl() throws RemoteException {
        super();
        // Seed account data
        accountBalances.put("ACC-1001", 5000.00);
        accountBalances.put("ACC-1002", 1200.50);
    }

    @Override
    public synchronized double checkBalance(String accountNumber) throws RemoteException {
        System.out.println("[BANK SERVER] Balance check requested for: " + accountNumber);
        return accountBalances.getOrDefault(accountNumber, 0.0);
    }

    @Override
    public synchronized boolean deposit(String accountNumber, double amount) throws RemoteException {
        System.out.printf("[BANK SERVER] Deposit $%.2f into %s%n", amount, accountNumber);
        if (amount <= 0 || !accountBalances.containsKey(accountNumber)) return false;

        double current = accountBalances.get(accountNumber);
        accountBalances.put(accountNumber, current + amount);
        return true;
    }

    @Override
    public synchronized boolean withdraw(String accountNumber, double amount) throws RemoteException {
        System.out.printf("[BANK SERVER] Withdrawal $%.2f requested from %s%n", amount, accountNumber);
        if (amount <= 0 || !accountBalances.containsKey(accountNumber)) return false;

        double current = accountBalances.get(accountNumber);
        if (current >= amount) {
            accountBalances.put(accountNumber, current - amount);
            return true;
        }
        return false;
    }
}
```

### `BankServer.java` (RMI Server Application)

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.Naming;

public class BankServer {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("   ENTERPRISE RMI BANKING SERVER SYSTEM   ");
        System.out.println("==========================================");

        try {
            LocateRegistry.createRegistry(1099);
            BankService bankService = new BankServiceImpl();
            Naming.rebind("rmi://localhost:1099/BankService", bankService);

            System.out.println("BankService bound to RMI Registry on port 1099.");
            System.out.println("Banking Server online and listening for remote transactions...");

        } catch (Exception e) {
            System.err.println("Banking Server Error: " + e.getMessage());
        }
    }
}
```

### `BankClient.java` (RMI Client Application)

```java
import java.rmi.Naming;

public class BankClient {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     ENTERPRISE RMI BANKING CLIENT        ");
        System.out.println("==========================================");

        try {
            BankService bank = (BankService) Naming.lookup("rmi://localhost:1099/BankService");
            String accNum = "ACC-1001";

            // 1. Initial Balance Check
            double initialBalance = bank.checkBalance(accNum);
            System.out.printf("Account %s Initial Balance: $%.2f%n", accNum, initialBalance);

            // 2. Execute Remote Deposit
            boolean depositOk = bank.deposit(accNum, 1500.00);
            System.out.println("Deposit $1,500.00 Success? : " + depositOk);

            // 3. Execute Remote Withdrawal
            boolean withdrawOk = bank.withdraw(accNum, 2000.00);
            System.out.println("Withdrawal $2,000.00 Success? : " + withdrawOk);

            // 4. Final Balance Check
            double finalBalance = bank.checkBalance(accNum);
            System.out.printf("Account %s Final Balance  : $%.2f%n", accNum, finalBalance);

            System.out.println("==========================================");

        } catch (Exception e) {
            System.err.println("Banking Client Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Thread-Safe Shared State (`ConcurrentHashMap`)**:
   - `BankServiceImpl` uses a `ConcurrentHashMap` and `synchronized` methods to maintain account balances safely across concurrent client RMI invocations.

2. **Distributed Transaction Methods**:
   - `BankClient` executes remote banking operations (`checkBalance()`, `deposit()`, `withdraw()`) over RMI, updating persistent server state.
