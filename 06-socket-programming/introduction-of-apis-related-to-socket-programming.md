# Unit 6: Socket Programming

## Introduction of APIs Related to Socket Programming

Java provides networking capabilities through the `java.net` package. This section introduces the core classes and interfaces used to establish network connections, construct datagram packets, and parse URLs.

---

### Core Concepts

#### 1. Core `java.net` Class Hierarchy

```
                                java.net Package
                                       │
      ┌────────────────────────────────┼────────────────────────────────┐
      ▼                                ▼                                ▼
IP & Addresses                  TCP Networking                    UDP Networking
 (InetAddress)                   ├── Socket                        ├── DatagramSocket
                                 └── ServerSocket                  └── DatagramPacket
```

#### 2. Networking API Class Reference

| Class Name | Transport Layer | Primary Role & Capabilities |
|------------|-----------------|-----------------------------|
| **`InetAddress`** | Network (IP Layer) | Represents IPv4 or IPv6 IP addresses. Converts hostnames to IP addresses. |
| **`Socket`** | TCP (Client Endpoint) | Client-side TCP socket. Connects to server IP/port and provides `InputStream` / `OutputStream` handles. |
| **`ServerSocket`** | TCP (Server Endpoint) | Server-side TCP socket. Binds to a port and listens for incoming client connections via `accept()`. |
| **`DatagramSocket`** | UDP (Packet Endpoint) | Sends and receives UDP datagram packets over network sockets (`send()`, `receive()`). |
| **`DatagramPacket`** | UDP (Packet Payload) | Container wrapping byte payload data, payload length, target IP address, and target port number. |
| **`URL` / `URLConnection`**| Application (HTTP/HTTPS) | Higher-level abstraction for reading data directly from web URLs over HTTP. |

#### 3. Stream Integration for Sockets
Because `Socket.getInputStream()` and `Socket.getOutputStream()` return raw byte streams, they are typically wrapped inside character buffers for text communication:

```java
// Wrapping Socket Output Stream for Line Text Writing
PrintWriter out = new PrintWriter(socket.getOutputStream(), true); // Auto-flush enabled

// Wrapping Socket Input Stream for Line Text Reading
BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
```

---

### Common Pitfalls

1. **Forgetting Auto-Flush on `PrintWriter`**:
   - `new PrintWriter(socket.getOutputStream())` buffers text output in memory without sending it across the network socket. Always pass `true` to enable auto-flushing: `new PrintWriter(out, true)`.
2. **Blocking Operations Hanging Threads**:
   - `ServerSocket.accept()` and `DatagramSocket.receive()` block thread execution indefinitely until data arrives. Use `.setSoTimeout(milliseconds)` to set read timeouts and prevent thread deadlocks.
3. **Mismatched Datagram Buffer Capacities**:
   - When receiving UDP packets with `DatagramSocket.receive(packet)`, if the incoming packet payload exceeds the buffer size allocated to `DatagramPacket`, excess bytes are truncated silently. Allocate sufficiently large byte array buffers (e.g. 1024 bytes).

---

## 1. Demo: InetAddress and URL Networking APIs

### `InetAddressApiDemo.java`

```java
import java.net.InetAddress;
import java.net.URL;
import java.net.URLConnection;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

/**
 * Demonstrates using InetAddress methods and reading raw data from a web URL using URLConnection.
 */
public class InetAddressApiDemo {

    public static void main(String[] args) {
        
        // 1. InetAddress API Operations
        System.out.println("=== 1. INETADDRESS API METADATA ===");
        try {
            InetAddress address = InetAddress.getByName("www.google.com");
            System.out.println("Host Name        : " + address.getHostName());
            System.out.println("IP Address       : " + address.getHostAddress());
            System.out.println("Is Multicast?    : " + address.isMulticastAddress());

            // Retrieve all IP addresses associated with a domain
            InetAddress[] allAddresses = InetAddress.getAllByName("www.google.com");
            System.out.println("Total IP Endpoints Found: " + allAddresses.length);
            for (InetAddress ip : allAddresses) {
                System.out.println("  -> " + ip.getHostAddress());
            }
        } catch (IOException e) {
            System.err.println("DNS resolution failed: " + e.getMessage());
        }

        // 2. URL and URLConnection API Operations
        System.out.println("\n=== 2. URL CONNECTION API ===");
        try {
            URL url = new URL("https://httpbin.org/get");
            URLConnection connection = url.openConnection();
            connection.setConnectTimeout(3000); // 3-second connection timeout

            System.out.println("Content Type     : " + connection.getContentType());
            System.out.println("Content Length   : " + connection.getContentLength());

            // Read first 3 lines of web response
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
                System.out.println("Reading HTTP Response Payload Header:");
                for (int i = 0; i < 3; i++) {
                    String line = reader.readLine();
                    if (line != null) System.out.println("  " + line);
                }
            }
        } catch (IOException e) {
            System.err.println("URL connection error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **`InetAddress.getAllByName()`**:
   - Fetches all IP addresses associated with a domain name, demonstrating how load balancers expose multiple IP endpoints.

2. **`URLConnection` Stream Processing**:
   - `url.openConnection()` creates an HTTP connection. `connection.getInputStream()` reads raw HTTP payload text into a `BufferedReader`.

---

## 2. Practical Program: Network Adapter Interface Inspector

### `NetworkInterfaceAuditor.java`

```java
import java.net.NetworkInterface;
import java.net.InetAddress;
import java.net.SocketException;
import java.util.Enumeration;

/**
 * Diagnostic networking tool auditing all active physical and virtual network interface cards (NICs) on the machine.
 */
public class NetworkInterfaceAuditor {

    public static void auditNetworkInterfaces() {
        System.out.println("==========================================");
        System.out.println("   NETWORK INTERFACE CARD (NIC) AUDITOR   ");
        System.out.println("==========================================");

        try {
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            
            while (interfaces.hasMoreElements()) {
                NetworkInterface ni = interfaces.nextElement();
                
                System.out.println("Interface Display Name: " + ni.getDisplayName());
                System.out.println("  Interface Name       : " + ni.getName());
                System.out.println("  Is Up and Running?   : " + ni.isUp());
                System.out.println("  Is Loopback Interface: " + ni.isLoopback());
                System.out.println("  Supports Multicast?  : " + ni.supportsMulticast());

                // Inspect Hardware MAC Address
                byte[] mac = ni.getHardwareAddress();
                if (mac != null) {
                    StringBuilder macBuilder = new StringBuilder();
                    for (int i = 0; i < mac.length; i++) {
                        macBuilder.append(String.format("%02X%s", mac[i], (i < mac.length - 1) ? "-" : ""));
                    }
                    System.out.println("  Hardware MAC Address : " + macBuilder);
                }

                // Enumerate IP Addresses assigned to this NIC
                Enumeration<InetAddress> addresses = ni.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress addr = addresses.nextElement();
                    System.out.println("  Assigned IP Address  : " + addr.getHostAddress());
                }
                System.out.println("------------------------------------------");
            }
        } catch (SocketException e) {
            System.err.println("Unable to inspect network interfaces: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        auditNetworkInterfaces();
    }
}
```

### Explanation

1. **`NetworkInterface` Inspection**:
   - Enumerates physical Ethernet, Wi-Fi, and virtual network adapters.

2. **MAC Address Formatting**:
   - `ni.getHardwareAddress()` extracts raw MAC bytes, formatting them into standard hexadecimal colon/hyphen pairs (`00-1A-2B-3C-4D-5E`).
