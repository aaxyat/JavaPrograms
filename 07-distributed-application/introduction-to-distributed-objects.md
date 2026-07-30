# Unit 7: Distributed Application

## Introduction to Distributed Objects

A **Distributed Application** is a software system composed of multiple distinct components executing on separate computing nodes across a network. These components coordinate actions and exchange data by calling methods on **Distributed Objects** as if they were local objects in the same JVM memory space.

---

### Core Concepts

#### 1. Monolithic vs. Distributed Object Architectures

```
  Monolithic Architecture                       Distributed Object Architecture
  (Single JVM / Shared Memory)                  (Multi-JVM / Network Bound)
  ┌──────────────────────────┐                  ┌────────────────┐           ┌────────────────┐
  │         JVM              │                  │ Client JVM     │           │ Server JVM     │
  │  ┌────────────────────┐  │                  │ ┌────────────┐ │           │ ┌────────────┐ │
  │  │  Caller Object     │  │                  │ │ Caller Obj │ │           │ │ Remote Obj │ │
  │  └─────────┬──────────┘  │                  │ └─────┬──────┘ │ Network   │ └─────▲──────┘ │
  │            │ Local Call  │                  │       │ Remote │ Protocol  │       │ Direct │
  │  ┌─────────▼──────────┐  │                  │ ┌─────▼──────┐ │ (RMI/TCP) │ ┌─────┴──────┐ │
  │  │ Target Object      │  │                  │ │ Local Stub │ ┼───────────┼─┤ Skeleton   │ │
  │  └────────────────────┘  │                  │ └────────────┘ │           │ └────────────┘ │
  └──────────────────────────┘                  └────────────────┘           └────────────────┘
```

#### 2. Key Concepts in Distributed Computing

- **Remote Method Invocation**: Invoking a method on an object located in a different Java Virtual Machine (JVM), potentially on another machine across the globe.
- **Marshalling**: The process of packaging method parameters and object states into a serialized byte stream suitable for network transmission.
- **Unmarshalling**: Reconstructing serialized byte streams back into live objects and primitive parameters on the receiving JVM.
- **Remote Proxy (Stub)**: A local client-side object implementing the same interface as the remote object. It intercepts local method calls and forwards them across the network to the actual remote instance.

#### 3. Core Challenges of Distributed Systems

| Challenge | Description | Mitigation Strategy |
|-----------|-------------|---------------------|
| **Network Latency** | Remote method calls take milliseconds over networks vs nanoseconds in RAM. | Minimize round-trip calls; use coarse-grained remote methods. |
| **Partial Failure** | Network links or remote server nodes can crash independently. | Use timeouts, retries, and checked remote exceptions (`RemoteException`). |
| **State Synchronization** | Objects on different nodes must maintain consistent data states. | Implement distributed transaction boundaries and immutable state transfers. |
| **Security & Authentication** | Remote endpoints are exposed to network attacks. | Use SSL/TLS socket encryption and authentication tokens. |

---

### Common Pitfalls

1. **Treating Remote Calls Like Local Calls**:
   - Calling remote methods inside tight loops causes extreme latency degradation due to repeated network round-trips. Batch data into single remote method invocations.
2. **Passing Non-Serializable Parameters**:
   - Parameters passed to remote methods must either be primitive types or implement `java.io.Serializable`. Passing non-serializable objects throws a `MarshalException`.
3. **Ignoring Network Exception Handling**:
   - Remote calls can fail due to dropped packets or offline servers. Failing to catch checked network exceptions crashes client applications.

---

## 1. Demo: Local Object vs. Simulated Distributed Proxy

### `DistributedObjectConceptDemo.java`

```java
import java.io.Serializable;

/**
 * Demonstrates the structural difference between direct local invocation and simulated remote proxy invocation.
 */

// Shared Data Transfer Object (DTO)
class DataPayload implements Serializable {
    private static final long serialVersionUID = 1L;
    private String content;

    public DataPayload(String content) {
        this.content = content;
    }

    public String getContent() { return content; }
}

// Service Interface
interface ServiceContract {
    String processData(DataPayload payload);
}

// Real Server-Side Service Implementation
class ActualService implements ServiceContract {
    @Override
    public String processData(DataPayload payload) {
        return "SERVER PROCESSED: " + payload.getContent().toUpperCase();
    }
}

// Client-Side Simulated Network Proxy (Stub)
class NetworkServiceProxy implements ServiceContract {
    private final ActualService remoteInstance;

    public NetworkServiceProxy(ActualService remoteInstance) {
        this.remoteInstance = remoteInstance;
    }

    @Override
    public String processData(DataPayload payload) {
        // Simulate Marshalling & Network Latency Overhead
        System.out.println("[PROXY] Marshalling parameters for network transfer...");
        long startTime = System.currentTimeMillis();

        try {
            Thread.sleep(100); // Simulate 100ms network delay
        } catch (InterruptedException ignored) {}

        // Delegate to remote instance (Simulating remote invocation)
        String result = remoteInstance.processData(payload);

        long latency = System.currentTimeMillis() - startTime;
        System.out.println("[PROXY] Unmarshalled response from remote host (Latency: " + latency + " ms)");
        return result;
    }
}

public class DistributedObjectConceptDemo {

    public static void main(String[] args) {
        System.out.println("=== DISTRIBUTED OBJECT PROXY SIMULATION ===");

        ActualService serverObject = new ActualService();
        ServiceContract clientProxy = new NetworkServiceProxy(serverObject);

        DataPayload payload = new DataPayload("hello distributed world");

        // Client invokes method on local proxy interface
        String response = clientProxy.processData(payload);
        System.out.println("Client Received Response: " + response);
    }
}
```

### Explanation

1. **Service Contract Interface**:
   - Both the real server implementation (`ActualService`) and the client proxy (`NetworkServiceProxy`) implement `ServiceContract`.

2. **Simulated Proxy Overhead**:
   - `NetworkServiceProxy` intercepts calls, marshals serializable `DataPayload` objects, introduces simulated network latency, and forwards requests to the remote object.

---

## 2. Practical Program: Distributed Node Latency & Health Auditor

### `DistributedNodeHealthMonitor.java`

```java
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

/**
 * Diagnostic tool auditing distributed computing node readiness and measuring simulated network RTT latencies.
 */
class NodeStatus implements Serializable {
    private static final long serialVersionUID = 101L;

    private String nodeId;
    private String ipAddress;
    private boolean isOnline;
    private long latencyMs;

    public NodeStatus(String nodeId, String ipAddress, boolean isOnline, long latencyMs) {
        this.nodeId = nodeId;
        this.ipAddress = ipAddress;
        this.isOnline = isOnline;
        this.latencyMs = latencyMs;
    }

    public String getNodeId() { return nodeId; }
    public String getIpAddress() { return ipAddress; }
    public boolean isOnline() { return isOnline; }
    public long getLatencyMs() { return latencyMs; }
}

public class DistributedNodeHealthMonitor {

    public static void auditClusterHealth(List<NodeStatus> nodes) {
        System.out.println("==========================================");
        System.out.println("    DISTRIBUTED CLUSTER HEALTH AUDIT      ");
        System.out.println("==========================================");

        int activeNodes = 0;
        long totalLatency = 0;

        for (NodeStatus node : nodes) {
            String statusStr = node.isOnline() ? "ONLINE " : "OFFLINE";
            System.out.printf("Node ID: %-10s | IP: %-15s | Status: %s | Latency: %d ms%n",
                    node.getNodeId(), node.getIpAddress(), statusStr, node.getLatencyMs());

            if (node.isOnline()) {
                activeNodes++;
                totalLatency += node.getLatencyMs();
            }
        }

        System.out.println("------------------------------------------");
        System.out.println("Total Nodes Registered : " + nodes.size());
        System.out.println("Active Online Nodes    : " + activeNodes);
        if (activeNodes > 0) {
            System.out.printf("Average Cluster Latency: %.2f ms%n", (double) totalLatency / activeNodes);
        }
        System.out.println("==========================================");
    }

    public static void main(String[] args) {
        List<NodeStatus> cluster = new ArrayList<>();
        cluster.add(new NodeStatus("NODE-US-EAST", "10.0.1.15", true, 45));
        cluster.add(new NodeStatus("NODE-EU-WEST", "10.0.2.22", true, 120));
        cluster.add(new NodeStatus("NODE-AP-SOUTH", "10.0.3.88", false, 0));

        auditClusterHealth(cluster);
    }
}
```

### Explanation

1. **Serializable Data Transfer Objects (DTO)**:
   - `NodeStatus` implements `Serializable` so health metrics can be transferred across distributed network nodes.

2. **Cluster Health Aggregation**:
   - Iterates through node status reports to calculate average distributed network latency.
