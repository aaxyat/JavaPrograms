# Unit 6: Socket Programming

## Overview of Socket Programming

**Socket Programming** enables two applications running on different host machines (or different processes on the same machine) to communicate over a computer network. A **Socket** represents one endpoint of a two-way network communication link.

---

### Core Concepts

#### 1. Network Sockets & Port Numbers

```
  Client Machine (192.168.1.50)                          Server Machine (192.168.1.100)
  ┌───────────────────────────┐                          ┌───────────────────────────┐
  │ Application Process       │                          │ Server Process (Web/App)  │
  │ ┌──────────────────────┐  │                          │ ┌──────────────────────┐  │
  │ │ Client Socket        │  │   TCP / UDP Network Link │ │ Server Socket        │  │
  │ │ (Ephemeral Port 52140)│ ──┼──────────────────────────┼─┤ (Listening Port 8080)│  │
  │ └──────────────────────┘  │                          │ └──────────────────────┘  │
  └───────────────────────────┘                          └───────────────────────────┘
```

- **IP Address**: Uniquely identifies a host device on a network (e.g., `192.168.1.100` or `127.0.0.1` for localhost).
- **Port Number**: Identifies a specific software process on the host. Port numbers range from `0` to `65535`:
  - **Well-Known Ports (`0` - `1023`)**: Reserved for system services (HTTP `80`, HTTPS `443`, SSH `22`, FTP `21`).
  - **Registered Ports (`1024` - `49151`)**: Assigned to user processes or applications (MySQL `3306`, PostgreSQL `5432`).
  - **Dynamic / Ephemeral Ports (`49152` - `65535`)**: Assigned dynamically by the OS to client sockets during outgoing connections.

#### 2. TCP vs. UDP Protocol Comparison

| Metric | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|--------|-------------------------------------|------------------------------|
| **Connection Type** | Connection-oriented (requires 3-way handshake). | Connectionless (sends datagrams without handshake). |
| **Reliability** | **Guaranteed delivery**. Retransmits lost packets. | **Unreliable**. Packets may be lost or duplicated. |
| **Data Order** | **Ordered**. Packets arrive in exact sequence. | **Unordered**. Packets may arrive out of sequence. |
| **Data Format** | Continuous Byte Stream. | Discrete Datagram Packets. |
| **Header Size** | 20 bytes (Heavyweight). | 8 bytes (Lightweight). |
| **Use Cases** | Web pages (HTTP), File transfers (FTP), Email (SMTP). | Real-time video streaming, VoIP, Online gaming, DNS. |

---

### Common Pitfalls

1. **Attempting to Bind to Reserved Ports (< 1024)**:
   - Operating systems block non-root applications from opening listening sockets on ports under `1024`. Use ports higher than `1024` (such as `8080` or `9090`) for application development.
2. **Confusing TCP Byte Streams with UDP Datagram Packets**:
   - TCP provides a continuous stream of bytes where message boundaries are not preserved automatically. UDP sends distinct packets with fixed size boundaries.
3. **Assuming `127.0.0.1` Connects Across Physical Networks**:
   - `127.0.0.1` (localhost) routes traffic strictly internally inside the local machine. To connect across a physical network, bind sockets to actual network IP interfaces (`0.0.0.0` or local network IP).

---

## 1. Demo: IP Address and Hostname Resolution

### `NetworkAddressResolverDemo.java`

```java
import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * Demonstrates resolving local host IP addresses and performing domain name resolution (DNS).
 */
public class NetworkAddressResolverDemo {

    public static void main(String[] args) {
        
        System.out.println("=== 1. LOCAL HOST ADDRESS RESOLUTION ===");
        try {
            // Get Local Host Machine IP
            InetAddress localHost = InetAddress.getLocalHost();
            System.out.println("Local Host Name    : " + localHost.getHostName());
            System.out.println("Local IP Address   : " + localHost.getHostAddress());
            System.out.println("Loopback Address   : " + InetAddress.getLoopbackAddress().getHostAddress());
        } catch (UnknownHostException e) {
            System.err.println("Unable to determine local host: " + e.getMessage());
        }

        System.out.println("\n=== 2. DOMAIN NAME RESOLUTION (DNS) ===");
        String domain = "example.com";
        try {
            InetAddress domainAddress = InetAddress.getByName(domain);
            System.out.println("Domain Name        : " + domainAddress.getHostName());
            System.out.println("Resolved IP Address: " + domainAddress.getHostAddress());
            System.out.println("Reachable (2 sec)? : " + domainAddress.isReachable(2000));
        } catch (UnknownHostException e) {
            System.err.println("Unknown Host: " + domain);
        } catch (Exception e) {
            System.err.println("Network error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Local Host Inspection (`InetAddress.getLocalHost()`)**:
   - Inspects the client host name and assigned IP address.

2. **DNS Resolution (`InetAddress.getByName()`)**:
   - Queries DNS servers to resolve hostnames (`example.com`) to IP addresses. `isReachable(2000)` sends an ICMP ping packet with a 2-second timeout.

---

## 2. Practical Program: Local TCP Port Availability Scanner

### `PortScannerAuditor.java`

```java
import java.net.Socket;
import java.io.IOException;

/**
 * Port scanner checking availability across common TCP service ports on the local machine.
 */
public class PortScannerAuditor {

    public static void scanLocalPorts(String host, int startPort, int endPort) {
        System.out.println("==========================================");
        System.out.println("      LOCAL TCP PORT AVAILABILITY SCAN     ");
        System.out.println("==========================================");
        System.out.printf("Scanning Host: %s (Ports %d to %d)%n", host, startPort, endPort);
        System.out.println("------------------------------------------");

        int openCount = 0;
        for (int port = startPort; port <= endPort; port++) {
            try (Socket socket = new Socket(host, port)) {
                System.out.printf("Port %-5d : OPEN  (Active Service Running)%n", port);
                openCount++;
            } catch (IOException e) {
                // Connection failed -> Port is closed or blocked by firewall
            }
        }

        System.out.println("------------------------------------------");
        System.out.println("Scan Complete! Total Open Ports Found: " + openCount);
        System.out.println("==========================================");
    }

    public static void main(String[] args) {
        // Scan common development ports on localhost
        scanLocalPorts("127.0.0.1", 80, 8090);
    }
}
```

### Explanation

1. **Port Connection Testing (`new Socket(host, port)`)**:
   - Attempts to open a TCP connection to each port. If successful, the port is open and listening; if an `IOException` is thrown, the port is closed.
