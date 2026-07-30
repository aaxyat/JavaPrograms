# Unit 4: Java IO

## Working with Files

Java provides file system manipulation capabilities through the legacy `java.io.File` class and the modern `java.nio.file` (NIO.2) API. These classes allow applications to inspect file metadata, create and delete files, manage nested directories, and traverse directory trees.

---

### Core Concepts

#### 1. The `java.io.File` Class
The `File` class represents an abstract pathname to a file or directory on disk. It does not open the file contents for reading or writing; instead, it manipulates file system metadata and structure.

#### 2. Key `File` API Methods

| Category | Method | Return Type | Description |
|----------|--------|-------------|-------------|
| **Creation & Deletion** | `createNewFile()` | `boolean` | Creates a new empty file if it does not already exist. |
| | `mkdir()` | `boolean` | Creates the directory named by this abstract pathname. |
| | `mkdirs()` | `boolean` | Creates the directory including any necessary missing parent directories. |
| | `delete()` | `boolean` | Deletes the file or empty directory immediately. |
| **Inspection** | `exists()` | `boolean` | Checks if the file or directory exists on disk. |
| | `isFile()` / `isDirectory()` | `boolean` | Verifies whether the pathname points to a regular file or directory. |
| | `canRead()` / `canWrite()` | `boolean` | Checks OS read and write permissions. |
| | `getName()` | `String` | Returns the name of the file or directory. |
| | `getAbsolutePath()` | `String` | Returns the absolute file system path string. |
| | `length()` | `long` | Returns file size in bytes (returns `0L` for directories or missing files). |
| | `lastModified()` | `long` | Returns timestamp of last modification in milliseconds since Unix epoch. |
| **Directory Navigation** | `list()` | `String[]` | Returns an array of file/directory name strings in a directory. |
| | `listFiles()` | `File[]` | Returns an array of `File` objects contained in a directory. |

#### 3. Legacy `File` vs. Modern `java.nio.file.Path` (NIO.2)

```java
// Legacy java.io.File
File legacyFile = new File("scratch/data.txt");
boolean exists = legacyFile.exists();

// Modern java.nio.file (NIO.2)
Path path = Paths.get("scratch/data.txt");
boolean nioExists = Files.exists(path);
```

While `java.io.File` remains widely used, `java.nio.file.Files` provides better exception handling, symbolic link support, and atomic file operations.

---

### Common Pitfalls

1. **Using `mkdir()` Instead of `mkdirs()` for Nested Directories**:
   - `file.mkdir()` fails and returns `false` if any parent directory in the path does not exist. Always use `file.mkdirs()`, which creates missing parent folders automatically.
2. **Hardcoding Path Delimiters (`\` vs `/`)**:
   - Hardcoding Windows backslashes (`C:\\folder\\file.txt`) breaks code compatibility when ported to Linux or macOS. Use forward slashes (`/`) or `File.separator`.
3. **Assuming `delete()` Throws Exceptions on Failure**:
   - `File.delete()` returns `false` when deletion fails (for example, if the file is locked or a directory is not empty) without throwing an exception. Always check the boolean return value.

---

## 1. Demo: File Metadata Inspection and Directory Operations

### `FileOperationsDemo.java`

```java
import java.io.File;
import java.io.IOException;
import java.util.Date;

/**
 * Demonstrates file creation, metadata inspection, directory tree setup, and file deletion.
 */
public class FileOperationsDemo {

    public static void main(String[] args) {
        
        // 1. Setup Directory Path
        File targetDir = new File("scratch/demo_directory/subfolder");
        
        // Create nested directories using mkdirs()
        if (targetDir.mkdirs()) {
            System.out.println("Created directory path: " + targetDir.getPath());
        } else {
            System.out.println("Directory path already exists.");
        }

        // 2. Create Target File
        File sampleFile = new File(targetDir, "sample_doc.txt");

        try {
            if (sampleFile.createNewFile()) {
                System.out.println("Created new file: " + sampleFile.getName());
            } else {
                System.out.println("File already exists: " + sampleFile.getName());
            }
        } catch (IOException e) {
            System.err.println("File creation error: " + e.getMessage());
        }

        // 3. Inspect File Metadata Attributes
        System.out.println("\n=== FILE METADATA INSPECTION ===");
        System.out.println("File Name          : " + sampleFile.getName());
        System.out.println("Absolute Path      : " + sampleFile.getAbsolutePath());
        System.out.println("Is Regular File?   : " + sampleFile.isFile());
        System.out.println("Is Directory?      : " + sampleFile.isDirectory());
        System.out.println("Readable Permission: " + sampleFile.canRead());
        System.out.println("Writable Permission: " + sampleFile.canWrite());
        System.out.println("File Size (bytes)  : " + sampleFile.length());
        System.out.println("Last Modified Date : " + new Date(sampleFile.lastModified()));

        // 4. List Directory Contents
        File parentFolder = new File("scratch/demo_directory");
        System.out.println("\n=== LISTING DIRECTORY CONTENTS (" + parentFolder.getName() + ") ===");
        
        File[] fileList = parentFolder.listFiles();
        if (fileList != null) {
            for (File item : fileList) {
                String typeStr = item.isDirectory() ? "[DIR] " : "[FILE]";
                System.out.printf("%s %-20s (Size: %d bytes)%n", typeStr, item.getName(), item.length());
            }
        }

        // 5. Delete File
        System.out.println("\n=== CLEANUP OPERATION ===");
        if (sampleFile.delete()) {
            System.out.println("Successfully deleted file: " + sampleFile.getName());
        } else {
            System.err.println("Failed to delete file: " + sampleFile.getName());
        }
    }
}
```

### Explanation

1. **Nested Directory Creation (`mkdirs()`)**:
   - `targetDir.mkdirs()` creates both `demo_directory` and its child `subfolder` in a single operation.

2. **File Metadata Accessors**:
   - `getAbsolutePath()`, `length()`, `canRead()`, and `lastModified()` inspect file attributes without opening input streams.

3. **Directory Traversal (`listFiles()`)**:
   - `parentFolder.listFiles()` returns `File[]` array objects, allowing the application to inspect child subdirectories and files.

---

## 2. Practical Program: Recursive Directory Inspector and File Maintenance Tool

### `DirectorySizeAndCleanUpTool.java`

```java
import java.io.File;
import java.io.PrintWriter;
import java.io.IOException;
import java.util.Date;

/**
 * Recursive directory scanner calculating total size, counting file types, and generating inspection reports.
 */
public class DirectorySizeAndCleanUpTool {

    private static long totalByteSize = 0;
    private static int totalFileCount = 0;
    private static int totalDirectoryCount = 0;

    public static void scanDirectory(File directory) {
        if (directory == null || !directory.exists()) return;

        File[] files = directory.listFiles();
        if (files == null) return;

        for (File file : files) {
            if (file.isDirectory()) {
                totalDirectoryCount++;
                scanDirectory(file); // Recursive call for subdirectories
            } else if (file.isFile()) {
                totalFileCount++;
                totalByteSize += file.length();
                
                System.out.printf("FILE: %-35s | Size: %8d bytes | Modified: %s%n",
                        file.getName(), file.length(), new Date(file.lastModified()));
            }
        }
    }

    public static void main(String[] args) {
        File rootFolder = new File("scratch/maintenance_test_dir");

        // Setup test directory structure with mock files
        setupMockDirectoryTree(rootFolder);

        System.out.println("==========================================");
        System.out.println("   RECURSIVE DIRECTORY INSPECTOR REPORT   ");
        System.out.println("==========================================");

        scanDirectory(rootFolder);

        System.out.println("------------------------------------------");
        System.out.println("Total Directories Scanned : " + totalDirectoryCount);
        System.out.println("Total Files Scanned       : " + totalFileCount);
        System.out.printf("Total Storage Used        : %.2f KB (%d bytes)%n",
                totalByteSize / 1024.0, totalByteSize);
        System.out.println("==========================================");
    }

    private static void setupMockDirectoryTree(File root) {
        File logsDir = new File(root, "logs");
        File tempDir = new File(root, "temp");

        logsDir.mkdirs();
        tempDir.mkdirs();

        createMockFile(new File(root, "app_config.json"), "{\"status\": \"active\"}");
        createMockFile(new File(logsDir, "system_audit.log"), "2026-07-30 [INFO] System running cleanly.");
        createMockFile(new File(tempDir, "cache_01.tmp"), "TEMP_CACHE_PAYLOAD_DATA_12345");
    }

    private static void createMockFile(File targetFile, String content) {
        try (PrintWriter writer = new PrintWriter(targetFile)) {
            writer.print(content);
        } catch (IOException e) {
            System.err.println("Mock file creation failed: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Recursive Traversal Algorithm**:
   - `scanDirectory()` uses recursion: if a child is a directory (`file.isDirectory()`), it calls itself to inspect subfolders. If a child is a file, it aggregates file size (`file.length()`) and count.

2. **Storage Size Aggregation**:
   - Accumulates total bytes into `totalByteSize` and converts bytes to kilobytes (`totalByteSize / 1024.0`) for display.
