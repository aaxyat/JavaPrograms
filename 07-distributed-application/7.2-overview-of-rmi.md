# Unit 7: Distributed Application

## Overview of RMI

**Java Remote Method Invocation (RMI)** is an API in the `java.rmi` package that enables an object running in one Java Virtual Machine (JVM) to invoke methods on an object running in another JVM across a network.

---

### Core Concepts

#### 1. RMI vs. Socket Programming Comparison

```
  Low-Level Socket Communication             High-Level RMI Communication
  ┌────────────┐   Raw Byte Stream           ┌────────────┐   Native Object Method Call
  │ Client App │ ──────────────────────────> │ Client App │ ───────────────────────────>
  └────────────┘   (Manual String Parsing)   └────────────┘   (Automatic Marshalling)
```

| Metric | Socket Programming (`java.net`) | Java RMI (`java.rmi`) |
|--------|----------------------------------|-----------------------|
| **Abstraction Level** | Low-level transport layer (raw bytes/strings). | High-level application layer (object methods). |
| **Protocol** | Raw TCP / UDP streams. | JRMP (Java Remote Method Protocol) over TCP. |
| **Data Parsing** | Manual parsing of custom text/binary strings. | Automatic marshalling and unmarshalling of objects. |
| **Language Lock** | Language-neutral (C++, Python, Java can connect). | Java-to-Java specific. |
| **Method Calls** | Explicit socket reads and writes (`send`/`receive`). | Direct method syntax (`remoteObj.calculateTotal()`). |

#### 2. Key Packages and Classes in Java RMI

- **`java.rmi.Remote` Interface**:
  - A marker interface that all remote interfaces must extend. It identifies interfaces whose methods can be called from non-local JVMs.
- **`java.rmi.RemoteException`**:
  - A checked exception thrown by every method defined in a `Remote` interface. It signals communication failures, protocol errors, or server disconnects.
- **`java.rmi.server.UnicastRemoteObject`**:
  - The base class for remote object implementations. Extending `UnicastRemoteObject` exports the remote object so it listens for incoming RMI socket calls on an anonymous port.
- **`java.rmi.registry.LocateRegistry`**:
  - Utility class used to create or look up an RMI Registry instance on a specific port (default port `1099`).
- **`java.rmi.Naming`**:
  - Provides methods for storing and retrieving remote object references in the RMI Registry using URL strings:
    - `Naming.rebind("rmi://localhost:1099/ServiceName", remoteObj)`: Registers a service.
    - `Naming.lookup("rmi://localhost:1099/ServiceName")`: Retrieves a remote stub.

---

### Common Pitfalls

1. **Forgetting `throws RemoteException` on Remote Interface Methods**:
   - Every method declared inside an interface extending `java.rmi.Remote` **must** declare `throws RemoteException`. Failing to do so causes a compilation error.
2. **Attempting to Lookup Services Before Registry is Started**:
   - Calling `Naming.lookup()` before the server has created the RMI Registry or invoked `Naming.rebind()` throws a `NotBoundException` or `ConnectException`.
3. **Passing Non-Serializable Parameters to Remote Methods**:
   - Remote method arguments and return types are passed by value and must implement `java.io.Serializable` (unless the parameter is itself another `Remote` object, which is passed by remote reference).

---

## 1. Demo: RMI Remote Interface and Class Structure

### `RmiOverviewDemo.java`

```java
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.io.Serializable;

/**
 * Demonstrates defining an RMI Remote Interface and implementing a Remote Object.
 */

// 1. Define Remote Interface extending java.rmi.Remote
interface GreetingService extends Remote {
    // Remote method declaring throws RemoteException
    String getGreeting(String clientName) throws RemoteException;
}

// 2. Implement Remote Interface extending UnicastRemoteObject
class GreetingServiceImpl extends UnicastRemoteObject implements GreetingService {
    
    // Explicit constructor calling super() to export object
    public GreetingServiceImpl() throws RemoteException {
        super();
    }

    @Override
    public String getGreeting(String clientName) throws RemoteException {
        System.out.println("[RMI SERVER] Executing getGreeting() for client: " + clientName);
        return "Hello " + clientName + "! Welcome to Java RMI Remote Invocation.";
    }
}

public class RmiOverviewDemo {

    public static void main(String[] args) {
        System.out.println("=== 1. VERIFYING RMI REMOTE OBJECT SETUP ===");

        try {
            // Instantiate remote service
            GreetingService service = new GreetingServiceImpl();
            
            System.out.println("Remote Object Instantiated & Exported Successfully!");
            System.out.println("Interface extends Remote? : " + (service instanceof Remote));
            System.out.println("Class extends UnicastRemoteObject? : " + (service instanceof UnicastRemoteObject));

            // Test local invocation on remote implementation
            String response = service.getGreeting("Developer");
            System.out.println("Direct Test Response: " + response);

        } catch (RemoteException e) {
            System.err.println("RMI Setup Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **`GreetingService` Remote Interface**:
   - Extends `java.rmi.Remote`. The method `getGreeting()` declares `throws RemoteException`.

2. **`GreetingServiceImpl` Remote Object**:
   - Extends `UnicastRemoteObject` so the RMI runtime exports the object to listen on a network socket.

---

## 2. Practical Program: Remote Task Execution Service Interface

### `RemoteComputeServiceAuditor.java`

```java
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.io.Serializable;

/**
 * Enterprise remote task execution contract declaring Serializable task requests and results.
 */
class TaskRequest implements Serializable {
    private static final long serialVersionUID = 2026L;
    private String taskName;
    private int[] numbers;

    public TaskRequest(String taskName, int[] numbers) {
        this.taskName = taskName;
        this.numbers = numbers;
    }

    public String getTaskName() { return taskName; }
    public int[] getNumbers() { return numbers; }
}

class TaskResult implements Serializable {
    private static final long serialVersionUID = 2027L;
    private String taskName;
    private long computedSum;

    public TaskResult(String taskName, long computedSum) {
        this.taskName = taskName;
        this.computedSum = computedSum;
    }

    @Override
    public String toString() {
        return String.format("TaskResult [Task: %s | Sum: %d]", taskName, computedSum);
    }
}

// Remote Compute Engine Interface
interface ComputeEngineService extends Remote {
    TaskResult executeTask(TaskRequest request) throws RemoteException;
}

public class RemoteComputeServiceAuditor {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("   RMI COMPUTE SERVICE CONTRACT AUDITOR   ");
        System.out.println("==========================================");

        TaskRequest request = new TaskRequest("ARRAY_SUM", new int[]{10, 20, 30, 40, 50});

        System.out.println("Task Request Created : " + request.getTaskName());
        System.out.println("Payload Element Count: " + request.getNumbers().length);
        System.out.println("Is DTO Serializable? : " + (request instanceof Serializable));
        System.out.println("==========================================");
    }
}
```

### Explanation

1. **Serializable DTO Design**:
   - `TaskRequest` and `TaskResult` implement `Serializable` so arguments and return values pass seamlessly across RMI network boundaries.
