# Unit 4: Java IO

## Introduction to IO in Java

Java I/O (Input/Output) handles data transfer between a Java application and external data sources or destinations—including disk files, network sockets, memory buffers, and system console terminals.

---

### Core Concepts

#### 1. The Stream Abstraction
In Java, **I/O operations are modeled as Streams**. A stream represents a continuous flow of data:

```
  Data Source                        Java Application                       Data Destination
 (File / Keyboard) ──[ InputStream / Reader ]──> (Program Memory) ──[ OutputStream / Writer ]──> (File / Screen)
```

- **Input Stream**: Reads data sequentially into the Java program.
- **Output Stream**: Writes data sequentially out of the Java program.

#### 2. Byte Streams vs. Character Streams

Java divides its I/O framework (`java.io`) into two distinct hierarchies based on data types:

| Metric | Byte Streams | Character Streams |
|--------|--------------|-------------------|
| **Data Unit** | 8-bit raw bytes (`byte`). | 16-bit Unicode characters (`char`). |
| **Base Abstract Classes** | `InputStream` and `OutputStream` | `Reader` and `Writer` |
| **Primary Use Cases** | Binary data (images, audio, PDF files, executable streams). | Text documents, source code, JSON, XML files. |
| **Encoding Handling** | Raw binary bytes without automatic character set translation. | Automatically translates between binary bytes and Unicode charsets (UTF-8, UTF-16). |

#### 3. Standard Console Streams (`System`)
Java provides three pre-defined stream objects in the `System` class:
- **`System.in`**: Standard Input Stream (`InputStream`) connected to keyboard input.
- **`System.out`**: Standard Output Stream (`PrintStream`) connected to console screen output.
- **`System.err`**: Standard Error Stream (`PrintStream`) connected to console error output.

---

### Common Pitfalls

1. **Using Byte Streams for Unicode Text Files**:
   - Using `FileInputStream` to read multibyte UTF-8 characters processes data byte-by-byte. If a UTF-8 character consists of 2 or 3 bytes, splitting it across byte reads corrupts text characters. Use `FileReader` or `BufferedReader` for text.
2. **Forgetting to Close Streams**:
   - Leaving open file streams consumes operating system file handles and causes file locks. Always use the **try-with-resources** statement (`try (InputStream is = ...)`) to guarantee automatic stream closure even when exceptions occur.
3. **Misinterpreting the EOF Return Value (`-1`)**:
   - The `read()` method returns an `int` rather than a `byte` or `char`. It returns values from `0` to `255` for valid byte data, and returns `-1` to signal the End-Of-File (EOF). Comparing `(byte) is.read() == -1` causes logic bugs because byte `-1` (`0xFF`) matches EOF prematurely.

---

## 1. Demo: Byte Stream vs. Character Stream Basics

### `IoBasicsDemo.java`

```java
import java.io.ByteArrayInputStream;
import java.io.StringReader;
import java.io.InputStream;
import java.io.Reader;
import java.io.IOException;

/**
 * Demonstrates reading raw bytes with InputStream vs reading characters with Reader.
 */
public class IoBasicsDemo {

    public static void main(String[] args) {
        
        // 1. Byte Stream Reading (ByteArrayInputStream)
        System.out.println("=== 1. Byte Stream Processing (InputStream) ===");
        byte[] binaryData = {74, 97, 118, 97, 32, 73, 47, 79}; // ASCII values for "Java I/O"

        try (InputStream byteStream = new ByteArrayInputStream(binaryData)) {
            int byteValue;
            System.out.print("Reading Raw Bytes: ");
            while ((byteValue = byteStream.read()) != -1) {
                System.out.print(byteValue + " ");
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("Byte stream error: " + e.getMessage());
        }

        // 2. Character Stream Reading (StringReader)
        System.out.println("\n=== 2. Character Stream Processing (Reader) ===");
        String textContent = "Java Unicode: \u0915\u0925\u093E"; // Text with Unicode characters

        try (Reader charStream = new StringReader(textContent)) {
            int charValue;
            System.out.print("Reading Characters: ");
            while ((charValue = charStream.read()) != -1) {
                System.out.print((char) charValue);
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("Character stream error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Byte Stream Reading (`ByteArrayInputStream`)**:
   - `ByteArrayInputStream` wraps an array of 8-bit bytes.
   - `byteStream.read()` reads one byte at a time, returning an `int` value until `-1` indicates end of stream.

2. **Character Stream Reading (`StringReader`)**:
   - `StringReader` processes 16-bit Unicode characters directly.
   - Casting `(char) charValue` prints multi-byte Unicode characters accurately without corrupting character encodings.

3. **Try-With-Resources Syntax**:
   - Enclosing stream declarations inside `try (InputStream byteStream = ...)` ensures that `.close()` runs automatically when exiting the block.

---

## 2. Practical Program: Stream Type and Encoding Auditor

### `DataStreamAuditor.java`

```java
import java.io.InputStream;
import java.io.ByteArrayInputStream;
import java.io.Reader;
import java.io.InputStreamReader;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * Practical I/O auditor comparing byte counts vs Unicode character counts across stream decoders.
 */
public class DataStreamAuditor {

    public static void auditStreamData(byte[] rawInputBytes) {
        System.out.println("==========================================");
        System.out.println("      JAVA I/O DATA STREAM AUDIT REPORT    ");
        System.out.println("==========================================");

        int totalBytesRead = 0;
        int totalCharsRead = 0;

        // 1. Audit Raw Byte Stream
        try (InputStream inputStream = new ByteArrayInputStream(rawInputBytes)) {
            while (inputStream.read() != -1) {
                totalBytesRead++;
            }
        } catch (IOException e) {
            System.err.println("Error reading input byte stream: " + e.getMessage());
        }

        // 2. Audit Character Stream via InputStreamReader Bridge (UTF-8)
        try (Reader reader = new InputStreamReader(new ByteArrayInputStream(rawInputBytes), StandardCharsets.UTF_8)) {
            while (reader.read() != -1) {
                totalCharsRead++;
            }
        } catch (IOException e) {
            System.err.println("Error reading character stream: " + e.getMessage());
        }

        System.out.println("Raw Byte Count Read (8-bit bytes) : " + totalBytesRead);
        System.out.println("Decoded Character Count (16-bit)  : " + totalCharsRead);
        System.out.println("Byte-to-Char Expansion Ratio      : " + String.format("%.2f", (double) totalBytesRead / totalCharsRead));
        System.out.println("==========================================");
    }

    public static void main(String[] args) {
        // Multi-byte UTF-8 encoded text ("Hello Java!")
        String sampleText = "Hello Java! \u092D\u093E\u0937\u093E"; // Contains Devanagari Unicode characters
        byte[] utf8Bytes = sampleText.getBytes(StandardCharsets.UTF_8);

        auditStreamData(utf8Bytes);
    }
}
```

### Explanation

1. **`InputStreamReader` Bridge**:
   - `InputStreamReader` acts as a bridge from byte streams to character streams, reading bytes and decoding them into characters using a specified charset (`StandardCharsets.UTF-8`).

2. **Comparing Bytes vs Characters**:
   - Non-ASCII Unicode characters require 3 bytes in UTF-8 encoding. `totalBytesRead` reflects raw disk/network payload size, while `totalCharsRead` reflects actual text length.
