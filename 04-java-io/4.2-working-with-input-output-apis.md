# Unit 4: Java IO

## Working with Input/Output APIs

Java provides a rich hierarchy of I/O classes in the `java.io` package. These APIs allow applications to read and write files, buffer data for high-speed performance, process Java primitive data types in binary format, and format structured text reports.

---

### Core Concepts

#### 1. Java I/O API Class Taxonomy

```
                       java.io Package Architecture
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
      Byte Streams                                   Character Streams
 (Raw 8-bit Data APIs)                             (16-bit Text Data APIs)
     ├── FileInputStream                                ├── FileReader
     ├── FileOutputStream                               ├── FileWriter
     ├── BufferedInputStream                            ├── BufferedReader
     ├── BufferedOutputStream                           ├── BufferedWriter
     ├── DataInputStream                                └── PrintWriter
     └── DataOutputStream
```

#### 2. Class Roles & Capabilities

| Class | Category | Primary Purpose | Key Methods |
|-------|----------|-----------------|-------------|
| **`FileInputStream`** | Byte Stream | Reads raw bytes directly from a disk file. | `read()`, `available()` |
| **`FileOutputStream`** | Byte Stream | Writes raw bytes directly to a disk file. | `write()`, `flush()` |
| **`BufferedInputStream`** | Byte Stream | Adds an in-memory byte buffer (default 8KB) to minimize expensive disk access calls. | `read()` |
| **`BufferedOutputStream`** | Byte Stream | Accumulates byte writes in a buffer before flushing to disk. | `write()`, `flush()` |
| **`DataInputStream`** | Primitive Stream | Reads Java primitive data types (`int`, `double`, `boolean`) and UTF strings in binary format. | `readInt()`, `readDouble()`, `readUTF()` |
| **`DataOutputStream`** | Primitive Stream | Writes Java primitive data types in binary format. | `writeInt()`, `writeDouble()`, `writeUTF()` |
| **`FileReader` / `FileWriter`** | Character Stream | Reads and writes text files using default character encoding. | `read()`, `write()` |
| **`BufferedReader`** | Character Stream | Buffers text input and provides line-by-line reading. | `readLine()` |
| **`BufferedWriter`** | Character Stream | Buffers text output and provides line break helpers. | `write()`, `newLine()`, `flush()` |
| **`PrintWriter`** | Formatted Text | Formats text output using `print()`, `println()`, and `printf()`. | `println()`, `printf()` |

---

### Common Pitfalls

1. **Forgetting to Call `flush()` or Close Streams**:
   - Buffered output streams (`BufferedWriter`, `BufferedOutputStream`, `DataOutputStream`) hold data in memory until the buffer fills. If the program terminates before calling `flush()` or closing the stream, data remains unwritten on disk.
2. **Overwriting Files Accidentally**:
   - `new FileWriter("log.txt")` overwrites existing file content by default. Pass `true` to append data to the end of an existing file: `new FileWriter("log.txt", true)`.
3. **Misusing `DataInputStream.readUTF()`**:
   - `readUTF()` expects a specific binary length header created by `DataOutputStream.writeUTF()`. Calling `readUTF()` on a plain text file created by Notepad throws a `UTFDataFormatException` or `EOFException`.

---

## 1. Demo: Primitive Data Streams and Buffered Line I/O

### `IoApiBasicsDemo.java`

```java
import java.io.DataOutputStream;
import java.io.DataInputStream;
import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.BufferedWriter;
import java.io.BufferedReader;
import java.io.FileWriter;
import java.io.FileReader;
import java.io.File;
import java.io.IOException;

/**
 * Demonstrates binary primitive I/O (DataOutputStream/DataInputStream) and text line I/O (BufferedWriter/BufferedReader).
 */
public class IoApiBasicsDemo {

    public static void main(String[] args) {
        File binaryFile = new File("scratch/sensor_data.bin");
        File textFile = new File("scratch/sample_notes.txt");

        // Create directory if missing
        binaryFile.getParentFile().mkdirs();

        // 1. Binary Primitive I/O Demo
        System.out.println("=== 1. DataOutputStream & DataInputStream Demo ===");
        try (DataOutputStream dos = new DataOutputStream(new FileOutputStream(binaryFile))) {
            dos.writeInt(101);                  // Sensor ID
            dos.writeDouble(98.6);              // Temperature Reading
            dos.writeBoolean(true);             // Status Flag
            dos.writeUTF("Operational Normal"); // String Message
            System.out.println("Binary data successfully written to: " + binaryFile.getName());
        } catch (IOException e) {
            System.err.println("Error writing binary data: " + e.getMessage());
        }

        // Reading Binary Data back
        try (DataInputStream dis = new DataInputStream(new FileInputStream(binaryFile))) {
            int sensorId = dis.readInt();
            double temp = dis.readDouble();
            boolean status = dis.readBoolean();
            String message = dis.readUTF();

            System.out.println("Read Sensor ID   : " + sensorId);
            System.out.println("Read Temperature : " + temp + " °F");
            System.out.println("Read Status      : " + (status ? "Active" : "Inactive"));
            System.out.println("Read Message     : " + message);
        } catch (IOException e) {
            System.err.println("Error reading binary data: " + e.getMessage());
        }

        // 2. Buffered Character Text Line I/O Demo
        System.out.println("\n=== 2. BufferedWriter & BufferedReader Line I/O Demo ===");
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(textFile))) {
            writer.write("Line 1: System Initialization");
            writer.newLine();
            writer.write("Line 2: Executing Diagnostic Check");
            writer.newLine();
            writer.write("Line 3: Process Completed Successfully");
            System.out.println("Text lines written to: " + textFile.getName());
        } catch (IOException e) {
            System.err.println("Error writing text lines: " + e.getMessage());
        }

        // Reading Text Lines back using BufferedReader
        try (BufferedReader reader = new BufferedReader(new FileReader(textFile))) {
            String line;
            System.out.println("Reading text file line-by-line:");
            while ((line = reader.readLine()) != null) {
                System.out.println("  -> " + line);
            }
        } catch (IOException e) {
            System.err.println("Error reading text lines: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Binary Primitive I/O**:
   - `DataOutputStream` wraps `FileOutputStream`, writing structured binary data using `writeInt()`, `writeDouble()`, `writeBoolean()`, and `writeUTF()`.
   - `DataInputStream` reads primitive data back in the exact order it was written.

2. **Buffered Text Line I/O**:
   - `BufferedWriter` wraps `FileWriter` for buffered character writing, using `newLine()` for platform-independent line breaks.
   - `BufferedReader.readLine()` reads text line-by-line until returning `null` at EOF.

---

## 2. Practical Program: Server Log Analyzer & Filter

### `LogFileProcessor.java`

```java
import java.io.BufferedReader;
import java.io.PrintWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.File;
import java.io.IOException;

/**
 * Server access log analyzer reading entries via BufferedReader and exporting error logs using PrintWriter.
 */
public class LogFileProcessor {

    public static void processLogFile(File inputFile, File errorOutputFile) {
        System.out.println("==========================================");
        System.out.println("     SERVER ACCESS LOG FILE PROCESSOR     ");
        System.out.println("==========================================");

        int totalEntries = 0;
        int errorCount = 0;

        try (BufferedReader reader = new BufferedReader(new FileReader(inputFile));
             PrintWriter errorWriter = new PrintWriter(new FileWriter(errorOutputFile))) {

            // Write Header to Error Report
            errorWriter.println("==========================================");
            errorWriter.println("    FILTERED SERVER ERROR REPORT (4xx/5xx)");
            errorWriter.println("==========================================");

            String line;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.isEmpty() || line.startsWith("#")) continue; // Skip comments/empty lines

                totalEntries++;
                String[] parts = line.split("\\s+");

                if (parts.length >= 4) {
                    String timestamp = parts[0];
                    String ipAddress = parts[1];
                    String endpoint = parts[2];
                    int statusCode = Integer.parseInt(parts[3]);

                    // Filter HTTP Error Status Codes (4xx Client Error, 5xx Server Error)
                    if (statusCode >= 400) {
                        errorCount++;
                        errorWriter.printf("[%s] ERROR %d | IP: %s | Endpoint: %s%n",
                                timestamp, statusCode, ipAddress, endpoint);
                    }
                }
            }

            errorWriter.println("------------------------------------------");
            errorWriter.printf("Total Error Entries Filtered: %d / %d%n", errorCount, totalEntries);
            errorWriter.println("==========================================");

            System.out.println("Processing Complete!");
            System.out.println("Total Log Records Evaluated : " + totalEntries);
            System.out.println("HTTP Error Records Filtered  : " + errorCount);
            System.out.println("Error Report Generated At   : " + errorOutputFile.getAbsolutePath());

        } catch (IOException e) {
            System.err.println("File Processing Error: " + e.getMessage());
        } catch (NumberFormatException e) {
            System.err.println("Log Parsing Error: Malformed HTTP status code encountered.");
        }
    }

    public static void main(String[] args) {
        File sampleLogFile = new File("scratch/server_access.log");
        File errorReportFile = new File("scratch/error_report.txt");

        // Generate sample access log file for demonstration
        try (PrintWriter writer = new PrintWriter(new FileWriter(sampleLogFile))) {
            writer.println("# Server Log Started");
            writer.println("2026-07-30T10:00:01 192.168.1.10 /api/v1/login 200");
            writer.println("2026-07-30T10:00:05 192.168.1.15 /api/v1/user/profile 200");
            writer.println("2026-07-30T10:00:12 192.168.1.22 /api/v1/admin/dashboard 403");
            writer.println("2026-07-30T10:00:18 192.168.1.45 /images/avatar_missing.png 404");
            writer.println("2026-07-30T10:00:25 192.168.1.88 /api/v1/checkout 500");
        } catch (IOException e) {
            System.err.println("Sample log setup failed: " + e.getMessage());
        }

        processLogFile(sampleLogFile, errorReportFile);
    }
}
```

### Explanation

1. **Efficient Text Parsing with `BufferedReader`**:
   - `reader.readLine()` streams log file lines into memory one at a time, allowing processing of large log files without memory exhaustion.

2. **Formatted Export with `PrintWriter`**:
   - `PrintWriter` provides `println()` and `printf()` to output formatted error reports to disk.
