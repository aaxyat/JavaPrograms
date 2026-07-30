# Unit 7: Distributed Application

## RMI Architecture

Java RMI relies on a **3-Layer Architecture** to handle remote object method calls: the **Stub/Skeleton Layer**, the **Remote Reference Layer (RRL)**, and the **Transport Layer**.

---

### Core Concepts

#### 1. The 3-Layer RMI Architecture

```
  Client JVM Host                                          Server JVM Host
  ┌──────────────────────────┐                             ┌──────────────────────────┐
  │   Client Application     │                             │  Remote Object Implementation
  └────────────┬─────────────┘                             └────────────▲─────────────┘
               │ Method Call (local syntax)                             │ Invoke Real Method
  ┌────────────▼─────────────┐                             ┌────────────┴─────────────┐
  │     Stub (Proxy)         │  Layer 1: Stub / Skeleton   │   Skeleton (Dispatcher)  │
  └────────────┬─────────────┘                             └────────────▲─────────────┘
               │ Marshalled Stream                                      │ Unmarshalled Stream
  ┌────────────▼─────────────┐                             ┌────────────┴─────────────┐
  │ Remote Reference Layer   │  Layer 2: Remote Reference  │ Remote Reference Layer   │
  └────────────┬─────────────┘                             └────────────▲─────────────┘
               │ JRMP Protocols                                         │ JRMP Protocols
  ┌────────────▼─────────────┐                             ┌────────────┴─────────────┐
  │     Transport Layer      │  Layer 3: Transport Layer   │     Transport Layer      │
  └────────────┬─────────────┘                             └────────────▲─────────────┘
               └────────────────── TCP/IP Socket Link ──────────────────┘
```

#### 2. Architectural Layers Breakdown

1. **Stub / Skeleton Layer**:
   - **Stub (Client Proxy)**: Lives in the client JVM. It acts as a gateway for remote objects, implementing the remote interface. When the client calls a method on the stub, the stub:
     - Initiates a connection to the remote JVM.
     - **Marshals** (serializes) method arguments into a byte stream.
     - Transmits the byte stream across the network.
     - Receives and **unmarshals** the return value.
   - **Skeleton (Server Dispatcher)**: Lives in the server JVM. It receives marshalled calls from the stub, unmarshals parameters, invokes the actual method on the server object, and marshals the result back to the stub. *(In modern Java 5+, dynamic proxy skeletons are generated automatically at runtime).*
2. **Remote Reference Layer (RRL)**:
   - Manages connection semantics. It determines whether a remote invocation is point-to-point (unicast) or replicated across a cluster.
3. **Transport Layer**:
   - Manages physical TCP/IP socket connections between client and server JVMs using **JRMP (Java Remote Method Protocol)**.

#### 3. Marshalling vs. Unmarshalling

- **Marshalling**: Converting object parameter graphs and method signatures into a serialized byte stream suitable for network socket transmission.
- **Unmarshalling**: Extracting serialized byte streams from network sockets and reconstructing them back into live Java object instances in memory.

#### 4. The RMI Registry (`rmiregistry`)
The **RMI Registry** is a bootstrap name lookup service running on port `1099`.
- **`rebind(name, obj)`**: Server binds a String name (e.g. `"CalculatorService"`) to a remote stub reference.
- **`lookup(name)`**: Client queries the registry by name to download the remote stub proxy reference.

---

### Common Pitfalls

1. **Class Not Found Exceptions During Unmarshalling**:
   - If a client sends a custom parameter object to the server, and the server JVM does not have the parameter's `.class` file on its classpath, the server throws an `UnmarshalException` wrapped around a `ClassNotFoundException`.
2. **Registry Port Conflicts (Port 1099)**:
   - Attempting to invoke `LocateRegistry.createRegistry(1099)` when another RMI server application is already running on port 1099 throws a `ExportException: Port already in use`.
3. **Mismatched Remote Interfaces**:
   - Modifying a method signature on the server remote interface without updating the client interface causes `NoSuchMethodError` or deserialization failures.

---

## 1. Demo: Simulated RMI Stub, Skeleton, and Registry Workflow

### `RmiArchitectureSimulator.java`

```java
import java.io.Serializable;
import java.util.HashMap;
import java.util.Map;

/**
 * Architectural simulation of RMI Stub marshalling, Registry lookup, and Skeleton unmarshalling.
 */

// Shared Parameter Payload
class CalculationPayload implements Serializable {
    private static final long serialVersionUID = 1L;
    private double num1;
    private double num2;

    public CalculationPayload(double num1, double num2) {
        this.num1 = num1;
        this.num2 = num2;
    }

    public double getNum1() { return num1; }
    public double getNum2() { return num2; }
}

// 1. Simulated RMI Registry
class SimulatedRmiRegistry {
    private static final Map<String, Object> registry = new HashMap<>();

    public static void rebind(String name, Object stub) {
        registry.put(name, stub);
        System.out.println("[REGISTRY] Bound service '" + name + "' to RMI Naming Registry.");
    }

    public static Object lookup(String name) {
        System.out.println("[REGISTRY] Client looking up service '" + name + "'...");
        return registry.get(name);
    }
}

// 2. Simulated Server Skeleton (Dispatcher)
class CalculatorSkeleton {
    public double dispatch(String methodName, CalculationPayload payload) {
        System.out.println("  [SKELETON] Unmarshalled payload: num1=" + payload.getNum1() + ", num2=" + payload.getNum2());
        
        // Execute actual method logic on server object
        if ("add".equalsIgnoreCase(methodName)) {
            double result = payload.getNum1() + payload.getNum2();
            System.out.println("  [SERVER OBJ] Computed Addition Result = " + result);
            return result;
        }
        return 0.0;
    }
}

// 3. Simulated Client Stub (Proxy)
class CalculatorStub {
    private final CalculatorSkeleton serverSkeleton;

    public CalculatorStub(CalculatorSkeleton serverSkeleton) {
        this.serverSkeleton = serverSkeleton;
    }

    public double add(double a, double b) {
        System.out.println("[STUB] Marshalling parameters into CalculationPayload DTO...");
        CalculationPayload payload = new CalculationPayload(a, b);

        System.out.println("[STUB] Transmitting marshalled payload across RRL & Transport Layer...");
        return serverSkeleton.dispatch("add", payload);
    }
}

public class RmiArchitectureSimulator {

    public static void main(String[] args) {
        System.out.println("=== RMI 3-LAYER ARCHITECTURE SIMULATION ===");

        // 1. Server Setup: Instantiate Skeleton and Register Stub
        CalculatorSkeleton skeleton = new CalculatorSkeleton();
        CalculatorStub stub = new CalculatorStub(skeleton);
        SimulatedRmiRegistry.rebind("rmi://localhost:1099/CalculatorService", stub);

        // 2. Client Setup: Lookup Stub from Registry
        CalculatorStub clientStub = (CalculatorStub) SimulatedRmiRegistry.lookup("rmi://localhost:1099/CalculatorService");

        // 3. Client Invokes Method on Stub Proxy
        System.out.println("\n--- Invoking Remote Method: clientStub.add(45.5, 54.5) ---");
        double sum = clientStub.add(45.5, 54.5);
        System.out.println("\nClient Received Final Result: " + sum);
    }
}
```

### Explanation

1. **Stub Parameter Marshalling**:
   - `CalculatorStub.add()` packages arguments into a `CalculationPayload` DTO, simulating how RMI stubs serialize method parameters.

2. **Skeleton Unmarshalling & Dispatch**:
   - `CalculatorSkeleton.dispatch()` unmarshals the payload, executes server logic, and returns the result back to the stub.

---

## 2. Practical Program: RMI Registry Service Inspector

### `RmiRegistryInspectorApp.java`

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;

/**
 * Inspection utility querying RMI registries on specific ports and listing bound remote service names.
 */
public class RmiRegistryInspectorApp {

    public static void inspectRmiRegistry(String host, int port) {
        System.out.println("==========================================");
        System.out.println("    RMI NAMING REGISTRY SERVICE INSPECTOR ");
        System.out.println("==========================================");
        System.out.printf("Connecting to RMI Registry at %s:%d...%n", host, port);

        try {
            // Programmatically start or locate RMI registry
            Registry registry = LocateRegistry.getRegistry(host, port);
            
            String[] boundServices = registry.list();

            System.out.println("Connection Successful!");
            System.out.println("Total Services Bound in Registry: " + boundServices.length);
            System.out.println("------------------------------------------");

            for (int i = 0; i < boundServices.length; i++) {
                System.out.printf("  Service #%d: %s%n", (i + 1), boundServices[i]);
            }
            if (boundServices.length == 0) {
                System.out.println("  (No active services bound in registry)");
            }
            System.out.println("==========================================");

        } catch (RemoteException e) {
            System.err.println("Registry Connection Error: Unable to locate RMI Registry at " + host + ":" + port);
            System.err.println("Details: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        inspectRmiRegistry("127.0.0.1", 1099);
    }
}
```

### Explanation

1. **Registry Lookup (`LocateRegistry.getRegistry`)**:
   - `LocateRegistry.getRegistry(host, port)` obtains a reference to an RMI Registry listening on port `1099`.

2. **Enumerating Services (`registry.list()`)**:
   - `registry.list()` returns a `String[]` array of all bound remote service names.
