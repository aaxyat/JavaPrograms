# Unit 6: Socket Programming

## Client Side Programming [TCP and UDP]

Client-side socket programming involves initiating network requests to server applications using **`Socket`** for TCP connections or **`DatagramSocket`** for UDP datagram transmissions.

---

### Core Concepts

#### 1. TCP Client Architecture (`Socket`)

```
                          TCP Client Lifecycle
  ┌─────────────────────────────────────────────────────────────────┐
  │ 1. Instantiate Socket & Connect to Server IP and Port           │
  │    Socket socket = new Socket("127.0.0.1", 8080);               │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 2. Obtain Input/Output Streams                                  │
  │    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
  │    BufferedReader in = new BufferedReader(...);                 │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 3. Send Requests & Read Server Responses                        │
  │    out.println("Hello Server");                                 │
  │    String response = in.readLine();                             │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 4. Close Socket & Streams                                       │
  │    socket.close();                                              │
  └─────────────────────────────────────────────────────────────────┘
```

#### 2. UDP Client Architecture (`DatagramSocket`)
UDP clients do not establish a persistent connection before transmitting data:
1. **Instantiate DatagramSocket**: `DatagramSocket clientSocket = new DatagramSocket()`. (The OS automatically binds to an available dynamic ephemeral port).
2. **Prepare Datagram Packet**: Convert data into bytes and specify target IP and server port:
   ```java
   byte[] sendData = "Hello UDP Server".getBytes();
   InetAddress serverIp = InetAddress.getByName("127.0.0.1");
   DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, serverIp, 9091);
   ```
3. **Transmit Packet**: Call `clientSocket.send(sendPacket)`.
4. **Receive Response (Optional)**: Allocate a buffer packet and call `clientSocket.receive(responsePacket)`.

---

### Common Pitfalls

1. **`ConnectException: Connection refused`**:
   - Thrown when attempting to connect to a target port where no server socket is currently listening, or if a firewall blocks the port. Always verify the server service is active before connecting clients.
2. **`UnknownHostException`**:
   - Occurs when the DNS resolver cannot resolve the specified server hostname string. Verify hostname spelling or use an IP address.
3. **Forgetting Auto-Flush on Client Output Streams**:
   - Using `new PrintWriter(socket.getOutputStream())` without auto-flush buffers request text without transmitting it, causing both client and server to hang waiting for data.

---

## 1. Demo: Dual TCP and UDP Client Implementations

### `TcpAndUdpClientDemo.java`

```java
import java.net.Socket;
import java.net.DatagramSocket;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;

/**
 * Demonstrates basic TCP client (Socket) and UDP client (DatagramSocket) communication.
 */
public class TcpAndUdpClientDemo {

    public static void runTcpClient(String host, int port) {
        System.out.println("=== 1. CONNECTING TCP CLIENT TO " + host + ":" + port + " ===");
        try (Socket socket = new Socket(host, port);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true); // Auto-flush enabled
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {

            System.out.println("Connected to TCP Server successfully.");
            
            // Send Request Message
            String requestMessage = "Hello from TCP Client!";
            System.out.println("Sending TCP Request: " + requestMessage);
            out.println(requestMessage);

            // Read Server Response
            String response = in.readLine();
            System.out.println("Received Server Response: " + response);

        } catch (IOException e) {
            System.err.println("TCP Client Error: " + e.getMessage());
        }
    }

    public static void runUdpClient(String host, int port) {
        System.out.println("\n=== 2. TRANSMITTING UDP DATAGRAM TO " + host + ":" + port + " ===");
        try (DatagramSocket udpSocket = new DatagramSocket()) {
            udpSocket.setSoTimeout(3000); // Set 3-second receive timeout

            InetAddress serverIp = InetAddress.getByName(host);
            String payload = "Hello from UDP Client!";
            byte[] sendBuffer = payload.getBytes();

            // Construct UDP Datagram Packet with target destination IP and Port
            DatagramPacket sendPacket = new DatagramPacket(sendBuffer, sendBuffer.length, serverIp, port);
            System.out.println("Sending UDP Datagram Packet...");
            udpSocket.send(sendPacket);

            // Prepare packet to receive UDP response
            byte[] receiveBuffer = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);

            udpSocket.receive(receivePacket);
            String response = new String(receivePacket.getData(), 0, receivePacket.getLength());
            System.out.println("Received UDP Server Response: " + response);

        } catch (IOException e) {
            System.err.println("UDP Client Error: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        // Run clients against local demo ports
        runTcpClient("127.0.0.1", 9090);
        runUdpClient("127.0.0.1", 9091);
    }
}
```

### Explanation

1. **TCP Connection & Auto-Flush (`PrintWriter(out, true)`)**:
   - Opens a TCP socket to `127.0.0.1:9090` and uses auto-flushing `PrintWriter` to transmit request strings.

2. **UDP Datagram Construction & Timeout (`setSoTimeout(3000)`)**:
   - `DatagramPacket` encapsulates byte payload data alongside destination IP and port (`9091`). `setSoTimeout(3000)` ensures the client will not block forever if the UDP packet is lost.

---

## 2. Practical Program: Interactive Network Service Client

### `InteractiveChatClient.java`

```java
import java.net.Socket;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;

/**
 * Interactive TCP client connecting to multi-threaded servers with live server response listening thread.
 */
public class InteractiveChatClient {

    private static final String SERVER_HOST = "127.0.0.1";
    private static final int SERVER_PORT = 8080;

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("      INTERACTIVE NETWORK CHAT CLIENT     ");
        System.out.println("==========================================");
        System.out.println("Connecting to Server " + SERVER_HOST + ":" + SERVER_PORT + "...");

        try (Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
             BufferedReader serverIn = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter serverOut = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader consoleIn = new BufferedReader(new InputStreamReader(System.in))) {

            System.out.println("Connected Successfully!");

            // Background Thread reading incoming Server Messages
            Thread listenerThread = new Thread(() -> {
                try {
                    String serverMsg;
                    while ((serverMsg = serverIn.readLine()) != null) {
                        System.out.println("\n[SERVER] " + serverMsg);
                        System.out.print("Enter Message > ");
                    }
                } catch (IOException e) {
                    System.out.println("\nServer connection closed.");
                }
            });
            listenerThread.setDaemon(true);
            listenerThread.start();

            // Main Thread reading User Console Input
            String userInput;
            System.out.print("Enter Message > ");
            while ((userInput = consoleIn.readLine()) != null) {
                serverOut.println(userInput);
                if ("QUIT".equalsIgnoreCase(userInput.trim())) {
                    System.out.println("Disconnecting from server...");
                    break;
                }
            }

        } catch (IOException e) {
            System.err.println("Client Communication Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Asynchronous Listener Thread**:
   - Spawns a background daemon thread (`listenerThread`) to continuously read incoming server messages from `serverIn.readLine()`, preventing console input reading from blocking server response updates.
