# Unit 6: Socket Programming

## Sample Programs

This section provides **4 complete, well-connected, real-world network applications**. Each application pairs client and server components to demonstrate end-to-end socket communications, followed by an in-depth code explanation.

---

## 1. Multi-User Real-Time TCP Chat Application

### `ChatServer.java`

```java
import java.net.ServerSocket;
import java.net.Socket;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;
import java.util.Set;
import java.util.HashSet;
import java.util.Collections;

public class ChatServer {

    private static final int PORT = 9000;
    private static final Set<PrintWriter> clientWriters = Collections.synchronizedSet(new HashSet<>());

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("   MULTI-USER REAL-TIME TCP CHAT SERVER   ");
        System.out.println("==========================================");
        System.out.println("Chat server listening on port " + PORT + "...");

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("New user connected: " + clientSocket.getRemoteSocketAddress());
                new Thread(new ClientHandler(clientSocket)).start();
            }
        } catch (IOException e) {
            System.err.println("Server Error: " + e.getMessage());
        }
    }

    private static class ClientHandler implements Runnable {
        private final Socket socket;
        private PrintWriter out;
        private String clientName;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
                out = new PrintWriter(socket.getOutputStream(), true);
                
                // Prompt for User Display Name
                out.println("SUBMITNAME");
                clientName = in.readLine();

                if (clientName == null || clientName.trim().isEmpty()) {
                    clientName = "User_" + socket.getPort();
                }

                clientWriters.add(out);
                broadcast("[SERVER] " + clientName + " has joined the chat room!");

                String message;
                while ((message = in.readLine()) != null) {
                    if ("QUIT".equalsIgnoreCase(message.trim())) {
                        break;
                    }
                    broadcast("[" + clientName + "]: " + message);
                }

            } catch (IOException e) {
                System.err.println("Communication error with " + clientName);
            } finally {
                if (out != null) {
                    clientWriters.remove(out);
                }
                broadcast("[SERVER] " + clientName + " has left the chat room.");
                try {
                    socket.close();
                } catch (IOException e) {
                    System.err.println("Socket close error: " + e.getMessage());
                }
            }
        }

        private void broadcast(String message) {
            System.out.println(message);
            synchronized (clientWriters) {
                for (PrintWriter writer : clientWriters) {
                    writer.println(message);
                }
            }
        }
    }
}
```

### `ChatClient.java`

```java
import java.net.Socket;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;

public class ChatClient {

    private static final String SERVER_HOST = "127.0.0.1";
    private static final int SERVER_PORT = 9000;

    public static void main(String[] args) {
        System.out.println("Connecting to Chat Server at " + SERVER_HOST + ":" + SERVER_PORT + "...");

        try (Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader consoleIn = new BufferedReader(new InputStreamReader(System.in))) {

            // Daemon thread listening for incoming broadcast messages
            Thread listener = new Thread(() -> {
                try {
                    String serverLine;
                    while ((serverLine = in.readLine()) != null) {
                        if ("SUBMITNAME".equals(serverLine)) {
                            System.out.print("Enter your display name: ");
                            String name = consoleIn.readLine();
                            out.println(name);
                        } else {
                            System.out.println(serverLine);
                        }
                    }
                } catch (IOException e) {
                    System.out.println("Disconnected from server.");
                }
            });
            listener.setDaemon(true);
            listener.start();

            // Main loop sending outgoing chat messages
            String userMsg;
            while ((userMsg = consoleIn.readLine()) != null) {
                out.println(userMsg);
                if ("QUIT".equalsIgnoreCase(userMsg.trim())) {
                    break;
                }
            }

        } catch (IOException e) {
            System.err.println("Client Error: " + e.getMessage());
        }
    }
}
```

### Explanation

- **Thread-Safe Broadcast Management**: `ChatServer` uses `Collections.synchronizedSet(new HashSet<>())` to maintain an active set of output streams (`PrintWriter`). When a user sends a message, `broadcast()` iterates through all connected client streams and transmits the text simultaneously.
- **Asynchronous Client I/O**: `ChatClient` runs a background daemon thread listening for incoming broadcast messages from `in.readLine()`, leaving the main thread free to capture console input from `System.in`.

---

## 2. Remote Binary File Transfer & Downloader System

### `FileServer.java`

```java
import java.net.ServerSocket;
import java.net.Socket;
import java.io.File;
import java.io.FileInputStream;
import java.io.BufferedInputStream;
import java.io.OutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;

public class FileServer {

    private static final int PORT = 9001;
    private static final String STORAGE_DIR = "scratch/server_files";

    public static void main(String[] args) {
        File folder = new File(STORAGE_DIR);
        folder.mkdirs();

        System.out.println("==========================================");
        System.out.println("    REMOTE FILE TRANSFER SERVER           ");
        System.out.println("==========================================");
        System.out.println("File Server listening on port " + PORT + "...");

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket clientSocket = serverSocket.accept();
                new Thread(() -> handleFileRequest(clientSocket)).start();
            }
        } catch (IOException e) {
            System.err.println("File Server Error: " + e.getMessage());
        }
    }

    private static void handleFileRequest(Socket socket) {
        try (DataInputStream dis = new DataInputStream(socket.getInputStream());
             DataOutputStream dos = new DataOutputStream(socket.getOutputStream())) {

            String requestedFileName = dis.readUTF();
            File requestedFile = new File(STORAGE_DIR, requestedFileName);

            if (!requestedFile.exists() || !requestedFile.isFile()) {
                dos.writeBoolean(false); // Signal File Not Found
                dos.writeUTF("ERROR: File not found on server.");
                return;
            }

            dos.writeBoolean(true); // Signal File Found
            dos.writeLong(requestedFile.length()); // Send File Size in Bytes

            // Stream File Bytes across socket
            byte[] buffer = new byte[4096];
            try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(requestedFile))) {
                int bytesRead;
                while ((bytesRead = bis.read(buffer)) != -1) {
                    dos.write(buffer, 0, bytesRead);
                }
                dos.flush();
            }
            System.out.println("Successfully transferred file '" + requestedFileName + "' (" + requestedFile.length() + " bytes).");

        } catch (IOException e) {
            System.err.println("File Transfer Error: " + e.getMessage());
        } finally {
            try {
                socket.close();
            } catch (IOException ignored) {}
        }
    }
}
```

### `FileClient.java`

```java
import java.net.Socket;
import java.io.File;
import java.io.FileOutputStream;
import java.io.BufferedOutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;

public class FileClient {

    private static final String SERVER_HOST = "127.0.0.1";
    private static final int SERVER_PORT = 9001;

    public static void downloadFile(String fileName, String destinationPath) {
        System.out.println("Requesting file '" + fileName + "' from " + SERVER_HOST + ":" + SERVER_PORT + "...");

        try (Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
             DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
             DataInputStream dis = new DataInputStream(socket.getInputStream())) {

            dos.writeUTF(fileName);

            boolean fileExists = dis.readBoolean();
            if (!fileExists) {
                String errorMsg = dis.readUTF();
                System.err.println("Download Failed: " + errorMsg);
                return;
            }

            long fileSize = dis.readLong();
            System.out.println("File Found! Size: " + fileSize + " bytes. Downloading...");

            File outputFile = new File(destinationPath);
            outputFile.getParentFile().mkdirs();

            byte[] buffer = new byte[4096];
            long totalRead = 0;
            try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outputFile))) {
                int bytesRead;
                while (totalRead < fileSize && (bytesRead = dis.read(buffer, 0, (int) Math.min(buffer.length, fileSize - totalRead))) != -1) {
                    bos.write(buffer, 0, bytesRead);
                    totalRead += bytesRead;
                }
                bos.flush();
            }

            System.out.println("Download Complete! Saved to: " + outputFile.getAbsolutePath());

        } catch (IOException e) {
            System.err.println("Client Download Error: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        downloadFile("sample.pdf", "scratch/downloads/sample.pdf");
    }
}
```

### Explanation

- **Binary Stream Chunking**: `FileServer` reads disk files using 4KB buffer chunks (`byte[4096]`) and writes them directly to `DataOutputStream`.
- **File Length Protocol Handshake**: The server transmits a boolean status flag followed by file size in bytes (`dos.writeLong(fileSize)`), allowing `FileClient` to track exact download progress and terminate stream reading cleanly.

---

## 3. UDP Network Time Protocol (NTP) & Heartbeat System

### `UdpHeartbeatServer.java`

```java
import java.net.DatagramSocket;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.io.IOException;
import java.util.Date;

public class UdpHeartbeatServer {

    private static final int PORT = 9002;

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     UDP NTP & HEARTBEAT RESPONSE SERVER  ");
        System.out.println("==========================================");
        System.out.println("UDP Server listening on port " + PORT + "...");

        try (DatagramSocket socket = new DatagramSocket(PORT)) {
            byte[] receiveBuffer = new byte[1024];

            while (true) {
                DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);
                socket.receive(receivePacket);

                String requestText = new String(receivePacket.getData(), 0, receivePacket.getLength()).trim();
                InetAddress clientIp = receivePacket.getAddress();
                int clientPort = receivePacket.getPort();

                System.out.printf("Received Ping [%s] from %s:%d%n", requestText, clientIp.getHostAddress(), clientPort);

                // Construct Response Payload with Server Timestamp
                String timestampResponse = "PONG | Server Time: " + new Date().toString();
                byte[] sendBuffer = timestampResponse.getBytes();

                DatagramPacket sendPacket = new DatagramPacket(sendBuffer, sendBuffer.length, clientIp, clientPort);
                socket.send(sendPacket);
            }
        } catch (IOException e) {
            System.err.println("UDP Server Error: " + e.getMessage());
        }
    }
}
```

### `UdpHeartbeatClient.java`

```java
import java.net.DatagramSocket;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.io.IOException;

public class UdpHeartbeatClient {

    private static final String SERVER_HOST = "127.0.0.1";
    private static final int SERVER_PORT = 9002;

    public static void main(String[] args) {
        System.out.println("Sending Heartbeat Pings to " + SERVER_HOST + ":" + SERVER_PORT + "...");

        try (DatagramSocket socket = new DatagramSocket()) {
            socket.setSoTimeout(2000); // 2-second timeout

            InetAddress serverIp = InetAddress.getByName(SERVER_HOST);

            for (int sequence = 1; sequence <= 3; sequence++) {
                String pingMessage = "PING_SEQ_" + sequence;
                byte[] sendBuffer = pingMessage.getBytes();

                DatagramPacket sendPacket = new DatagramPacket(sendBuffer, sendBuffer.length, serverIp, SERVER_PORT);

                long startTime = System.currentTimeMillis();
                socket.send(sendPacket);

                byte[] receiveBuffer = new byte[1024];
                DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);

                try {
                    socket.receive(receivePacket);
                    long rtt = System.currentTimeMillis() - startTime;
                    String response = new String(receivePacket.getData(), 0, receivePacket.getLength());

                    System.out.printf("Reply from Server: %s | Round-Trip Time (RTT): %d ms%n", response, rtt);
                } catch (IOException e) {
                    System.err.println("Ping Sequence " + sequence + " timed out (Packet Lost).");
                }

                Thread.sleep(1000); // Wait 1 second between pings
            }

        } catch (Exception e) {
            System.err.println("UDP Heartbeat Error: " + e.getMessage());
        }
    }
}
```

### Explanation

- **Connectionless Latency Measurement**: `UdpHeartbeatClient` transmits UDP ping packets without establishing a connection. It measures Round-Trip Time (RTT) latency by calculating the elapsed time between `socket.send()` and `socket.receive()`.

---

## 4. Multi-Threaded HTTP Web Server

### `MiniHttpWebServer.java`

```java
import java.net.ServerSocket;
import java.net.Socket;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.IOException;
import java.util.Date;

public class MiniHttpWebServer {

    private static final int PORT = 8085;

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     MINIATURE HTTP / 1.1 WEB SERVER      ");
        System.out.println("==========================================");
        System.out.println("HTTP Server active! Point your web browser to: http://localhost:" + PORT);

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket clientSocket = serverSocket.accept();
                new Thread(() -> handleHttpRequest(clientSocket)).start();
            }
        } catch (IOException e) {
            System.err.println("HTTP Server Error: " + e.getMessage());
        }
    }

    private static void handleHttpRequest(Socket socket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {

            String requestLine = in.readLine();
            if (requestLine == null || requestLine.isEmpty()) return;

            System.out.println("[HTTP REQUEST] " + requestLine);
            String[] requestTokens = requestLine.split("\\s+");
            String method = requestTokens[0];
            String path = requestTokens.length > 1 ? requestTokens[1] : "/";

            if ("GET".equalsIgnoreCase(method)) {
                if ("/".equals(path) || "/index.html".equals(path)) {
                    sendHttpResponse(out, 200, "OK", "text/html",
                            "<html><body><h1>Welcome to Java HTTP Web Server!</h1><p>Server Date: " + new Date() + "</p></body></html>");
                } else {
                    sendHttpResponse(out, 404, "Not Found", "text/html",
                            "<html><body><h1>404 Not Found</h1><p>The requested URL " + path + " was not found.</p></body></html>");
                }
            } else {
                sendHttpResponse(out, 501, "Not Implemented", "text/html", "<h1>501 Method Not Implemented</h1>");
            }

        } catch (IOException e) {
            System.err.println("HTTP Request Error: " + e.getMessage());
        } finally {
            try {
                socket.close();
            } catch (IOException ignored) {}
        }
    }

    private static void sendHttpResponse(PrintWriter out, int statusCode, String statusText, String contentType, String htmlBody) {
        out.println("HTTP/1.1 " + statusCode + " " + statusText);
        out.println("Content-Type: " + contentType + "; charset=UTF-8");
        out.println("Content-Length: " + htmlBody.getBytes().length);
        out.println("Connection: close");
        out.println(); // Blank line separating headers from HTTP body
        out.println(htmlBody);
    }
}
```

### Explanation

- **HTTP/1.1 Protocol Handling**: `MiniHttpWebServer` listens on TCP port `8085`. It parses raw HTTP request header lines (`GET / HTTP/1.1`), returns standard HTTP response headers (`HTTP/1.1 200 OK`, `Content-Type`, `Content-Length`), and serves dynamic HTML content to any web browser.
