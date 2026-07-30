# Unit 6: Socket Programming

## Server Side Programming [TCP and UDP]

Server-side socket programming involves creating server endpoints that listen on network ports, accept incoming client requests, process data payloads, and send responses back to clients. Java handles server-side communication using **`ServerSocket`** for TCP and **`DatagramSocket`** for UDP.

---

### Core Concepts

#### 1. TCP Server Lifecycle (`ServerSocket`)

```
                          TCP Server Lifecycle
  ┌─────────────────────────────────────────────────────────────────┐
  │ 1. Create ServerSocket listening on port (e.g. 8080)            │
  │    ServerSocket serverSocket = new ServerSocket(8080);          │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 2. Listen & Accept Client Connection (Blocks until client joins)│
  │    Socket clientSocket = serverSocket.accept();                 │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 3. Obtain Input/Output Streams for Communication                │
  │    BufferedReader in = new BufferedReader(...);                 │
  │    PrintWriter out = new PrintWriter(...);                      │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 4. Read Client Request & Write Server Response                  │
  │    String message = in.readLine();                              │
  │    out.println("Echo: " + message);                             │
  └────────────────────────────────┬────────────────────────────────┘
                                   │
  ┌────────────────────────────────▼────────────────────────────────┐
  │ 5. Close Client Socket or delegate to Worker Thread             │
  │    clientSocket.close();                                        │
  └─────────────────────────────────────────────────────────────────┘
```

#### 2. UDP Server Lifecycle (`DatagramSocket`)
UDP servers do not maintain persistent client connections. A single `DatagramSocket` listens for incoming data packets:
1. **Instantiate DatagramSocket**: `DatagramSocket serverSocket = new DatagramSocket(port)`.
2. **Allocate Packet Buffer**: Prepare a byte array buffer and wrap it in a `DatagramPacket`:
   ```java
   byte[] buffer = new byte[1024];
   DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
   ```
3. **Receive Packet**: `serverSocket.receive(receivePacket)` blocks execution until a datagram packet arrives.
4. **Extract Sender Info**: Retrieve client IP address and port from the received packet:
   ```java
   InetAddress clientIp = receivePacket.getAddress();
   int clientPort = receivePacket.getPort();
   ```
5. **Send Response Packet**: Construct a response `DatagramPacket` with the client's IP and port, and invoke `serverSocket.send(responsePacket)`.

---

### Common Pitfalls

1. **Single-Threaded TCP Server Bottlenecks**:
   - A single-threaded TCP server serving one client inside a sequential loop cannot accept new connections until the active client disconnects. Always use a thread pool (`ExecutorService`) to handle each accepted `Socket` in a separate thread.
2. **Forgetting `out.flush()` on Network Sockets**:
   - Buffered output streams or `PrintWriter` instances without auto-flush enabled buffer text in memory without transmitting it across the TCP connection.
3. **UDP Packet Loss & Overflows**:
   - UDP does not guarantee packet arrival or buffer management. If client datagram payloads exceed the server buffer length, data is truncated.

---

## 1. Demo: Dual TCP and UDP Server Implementations

### `TcpAndUdpServerDemo.java`

```java
import java.net.ServerSocket;
import java.net.Socket;
import java.net.DatagramSocket;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;

/**
 * Demonstrates basic TCP server (ServerSocket) and UDP server (DatagramSocket) handling.
 */
public class TcpAndUdpServerDemo {

    public static void runTcpServer(int port) {
        System.out.println("=== STARTING TCP ECHO SERVER ON PORT " + port + " ===");
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("TCP Server listening... Waiting for client connection.");
            
            try (Socket clientSocket = serverSocket.accept();
                 BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                 PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

                System.out.println("TCP Client Connected: " + clientSocket.getRemoteSocketAddress());
                String clientMessage = in.readLine();
                System.out.println("Received TCP Request: " + clientMessage);

                // Send Echo Response
                out.println("TCP SERVER ECHO: " + clientMessage);
                System.out.println("Sent TCP Echo Response.");
            }
        } catch (IOException e) {
            System.err.println("TCP Server Error: " + e.getMessage());
        }
    }

    public static void runUdpServer(int port) {
        System.out.println("\n=== STARTING UDP ECHO SERVER ON PORT " + port + " ===");
        try (DatagramSocket udpSocket = new DatagramSocket(port)) {
            byte[] receiveBuffer = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);

            System.out.println("UDP Server listening... Waiting for datagram packet.");
            udpSocket.receive(receivePacket); // Blocks until datagram arrives

            String message = new String(receivePacket.getData(), 0, receivePacket.getLength());
            InetAddress clientIp = receivePacket.getAddress();
            int clientPort = receivePacket.getPort();

            System.out.println("Received UDP Packet from " + clientIp.getHostAddress() + ":" + clientPort);
            System.out.println("Payload: " + message);

            // Construct and send response datagram
            String responseStr = "UDP SERVER ECHO: " + message;
            byte[] sendBuffer = responseStr.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(sendBuffer, sendBuffer.length, clientIp, clientPort);

            udpSocket.send(sendPacket);
            System.out.println("Sent UDP Echo Response.");

        } catch (IOException e) {
            System.err.println("UDP Server Error: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        // Run TCP and UDP servers sequentially for demonstration
        runTcpServer(9090);
        runUdpServer(9091);
    }
}
```

### Explanation

1. **TCP `ServerSocket.accept()`**:
   - `serverSocket.accept()` listens on port `9090` and blocks execution until a client connects, returning a dedicated `Socket` object for client communication.

2. **UDP `DatagramSocket.receive()`**:
   - `udpSocket.receive(receivePacket)` listens on port `9091` for datagram packets, extracting the sender's IP address and port number (`receivePacket.getAddress()` and `receivePacket.getPort()`) to route responses.

---

## 2. Practical Program: Production Multi-Threaded TCP Server

### `MultiThreadedEchoServer.java`

```java
import java.net.ServerSocket;
import java.net.Socket;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Multi-threaded TCP server using an ExecutorService thread pool to handle concurrent client connections.
 */
public class MultiThreadedEchoServer {

    private static final int PORT = 8080;
    private static final int THREAD_POOL_SIZE = 10;

    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_POOL_SIZE);

        System.out.println("==========================================");
        System.out.println("  MULTI-THREADED TCP ENTERPRISE SERVER    ");
        System.out.println("==========================================");
        System.out.println("Server listening on port " + PORT + " with thread pool size " + THREAD_POOL_SIZE);

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket clientSocket = serverSocket.accept(); // Accept incoming client connection
                System.out.println("\n[NEW CONNECTION] Client connected from: " + clientSocket.getRemoteSocketAddress());

                // Submit client handling task to thread pool
                threadPool.execute(new ClientHandler(clientSocket));
            }
        } catch (IOException e) {
            System.err.println("Server Exception: " + e.getMessage());
        } finally {
            threadPool.shutdown();
        }
    }

    // Client Handler Task Runnable
    private static class ClientHandler implements Runnable {
        private final Socket socket;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            String clientIp = socket.getRemoteSocketAddress().toString();

            try (BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                 PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {

                out.println("WELCOME! Connected to Enterprise Echo Server. Type 'QUIT' to exit.");

                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    System.out.printf("[%s] Sent: %s%n", clientIp, inputLine);

                    if ("QUIT".equalsIgnoreCase(inputLine.trim())) {
                        out.println("Goodbye! Connection closed.");
                        break;
                    }

                    // Process and echo response
                    out.println("SERVER ECHO: " + inputLine.toUpperCase());
                }

            } catch (IOException e) {
                System.err.println("Error handling client [" + clientIp + "]: " + e.getMessage());
            } finally {
                try {
                    socket.close();
                    System.out.println("[DISCONNECTED] Client socket closed: " + clientIp);
                } catch (IOException e) {
                    System.err.println("Failed to close socket: " + e.getMessage());
                }
            }
        }
    }
}
```

### Explanation

1. **Thread Pool Delegation (`ExecutorService`)**:
   - `Executors.newFixedThreadPool(10)` creates a pool of 10 worker threads. When `serverSocket.accept()` returns a new `clientSocket`, it delegates client communication to a `ClientHandler` runnable, keeping the main thread free to accept additional client connections.

2. **Session Lifecycle Management**:
   - The worker thread loops over `in.readLine()` until the client sends `"QUIT"` or disconnects, cleanly closing the client socket in a `finally` block.
