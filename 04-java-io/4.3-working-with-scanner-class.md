# Unit 4: Java IO

## Working with Scanner Class

The `java.util.Scanner` class simplifies parsing text, formatted strings, and disk files into primitive data types (`int`, `double`, `boolean`) and tokens using regular expression delimiters.

---

### Core Concepts

#### 1. Scanner Input Data Sources
A `Scanner` instance can be constructed to parse data from multiple input sources:

```java
// 1. Keyboard Console Input
Scanner consoleScanner = new Scanner(System.in);

// 2. Disk File Input
Scanner fileScanner = new Scanner(new File("data.txt"));

// 3. String Memory Buffer Input
Scanner stringScanner = new Scanner("Alice 25 3.85");
```

#### 2. Core Scanner Methods

| Method Group | Method Name | Description |
|--------------|-------------|-------------|
| **Token Reading** | `next()` | Reads and returns the next whitespace-delimited word token. |
| **Line Reading** | `nextLine()` | Reads and returns the remaining text on the current line including spaces. |
| **Primitive Parsing** | `nextInt()`, `nextDouble()`, `nextBoolean()` | Parses the next token as an `int`, `double`, or `boolean`. |
| **Look-Ahead Testing**| `hasNext()`, `hasNextInt()`, `hasNextLine()` | Returns `true` if another matching token is available without advancing the cursor. |
| **Custom Delimiters**| `useDelimiter(Pattern/String)` | Changes token separation from default whitespace to custom patterns (e.g. commas for CSV). |

---

### Common Pitfalls

1. **The `nextInt()` Followed by `nextLine()` Buffer Trap**:
   - `nextInt()` reads only the numeric digits and leaves the trailing newline character (`\n`) in the input buffer. When a subsequent `nextLine()` is called, it immediately consumes that leftover `\n` and returns an empty string without pausing for user input.
   - **Fix**: Consume the leftover newline by inserting an extra `scanner.nextLine()` call:
     ```java
     int age = scanner.nextInt();
     scanner.nextLine(); // Consume leftover newline character
     String name = scanner.nextLine();
     ```
2. **`InputMismatchException` Crashes**:
   - If a user enters text (e.g. `"twenty"`) when `nextInt()` is called, Java throws an `InputMismatchException`. Protect calls using look-ahead methods (`hasNextInt()`) or `try-catch` blocks.
3. **Leaving File Scanners Open**:
   - Passing a `File` to `new Scanner(file)` opens an OS file handle. Always wrap file scanners inside a try-with-resources statement to guarantee automatic file release.

---

## 1. Demo: Scanner Parsing and Buffer Trap Solutions

### `ScannerBasicsDemo.java`

```java
import java.util.Scanner;
import java.util.InputMismatchException;

/**
 * Demonstrates primitive parsing, solving the nextInt()/nextLine() buffer trap, and custom comma delimiter parsing.
 */
public class ScannerBasicsDemo {

    public static void main(String[] args) {
        
        // 1. Console Input & Buffer Trap Resolution
        Scanner scanner = new Scanner(System.in);

        System.out.println("=== 1. Interactive Input & Newline Buffer Solution ===");
        System.out.print("Enter your Age (integer): ");
        
        int age = 0;
        if (scanner.hasNextInt()) {
            age = scanner.nextInt();
            scanner.nextLine(); // CONSUME LEFTOVER NEWLINE (\n)
        } else {
            System.out.println("Invalid number entered. Defaulting age to 18.");
            scanner.nextLine(); // Clear invalid token
            age = 18;
        }

        System.out.print("Enter your Full Name: ");
        String fullName = scanner.nextLine(); // Reads full name correctly

        System.out.println("Captured Name: " + fullName + " | Age: " + age);

        // 2. Custom Delimiter String Scanning (CSV Parsing)
        System.out.println("\n=== 2. Custom Comma Delimiter Scanning ===");
        String csvData = "101,Sarah Connor,89.5,Active";

        try (Scanner csvScanner = new Scanner(csvData)) {
            csvScanner.useDelimiter(","); // Set comma as token delimiter

            int id = csvScanner.nextInt();
            String name = csvScanner.next();
            double score = csvScanner.nextDouble();
            String status = csvScanner.next();

            System.out.println("Parsed Student ID    : " + id);
            System.out.println("Parsed Name          : " + name);
            System.out.println("Parsed Score         : " + score);
            System.out.println("Parsed Status        : " + status);
        }

        scanner.close();
    }
}
```

### Explanation

1. **Solving the Newline Buffer Trap (`scanner.nextLine()`)**:
   - `scanner.nextInt()` reads the integer value. Calling `scanner.nextLine()` immediately afterward consumes the leftover newline character (`\n`), allowing the next `scanner.nextLine()` to capture the user's full name properly.

2. **Input Validation (`hasNextInt()`)**:
   - `scanner.hasNextInt()` tests if the input token is an integer before attempting to parse, avoiding `InputMismatchException` crashes.

3. **Custom Delimiters (`useDelimiter(",")`)**:
   - `csvScanner.useDelimiter(",")` splits the string `"101,Sarah Connor,89.5,Active"` at commas, extracting each token into matching primitive data types.

---

## 2. Practical Program: CSV Student Grade File Importer

### `CsvStudentDataImporter.java`

```java
import java.util.Scanner;
import java.io.File;
import java.io.PrintWriter;
import java.io.IOException;

/**
 * CSV file importer using Scanner to parse structured student records from disk.
 */
public class CsvStudentDataImporter {

    public static void importAndReportStudentData(File csvFile) {
        System.out.println("==========================================");
        System.out.println("      CSV STUDENT DATA IMPORT REPORT      ");
        System.out.println("==========================================");

        int recordCount = 0;
        double totalGpaSum = 0.0;

        try (Scanner fileScanner = new Scanner(csvFile)) {
            while (fileScanner.hasNextLine()) {
                String line = fileScanner.nextLine().trim();
                
                // Skip empty lines or header comments
                if (line.isEmpty() || line.startsWith("#")) continue;

                // Parse line using a line-level Scanner with comma delimiter
                try (Scanner lineScanner = new Scanner(line)) {
                    lineScanner.useDelimiter(",");

                    if (lineScanner.hasNextInt()) {
                        int studentId = lineScanner.nextInt();
                        String name = lineScanner.next();
                        String major = lineScanner.next();
                        double gpa = lineScanner.nextDouble();

                        recordCount++;
                        totalGpaSum += gpa;

                        System.out.printf("Record #%d -> ID: %d | Name: %-15s | Major: %-12s | GPA: %.2f%n",
                                recordCount, studentId, name, major, gpa);
                    }
                }
            }

            System.out.println("------------------------------------------");
            if (recordCount > 0) {
                double classAverageGpa = totalGpaSum / recordCount;
                System.out.println("Total Valid Records Imported : " + recordCount);
                System.out.printf("Class Average GPA            : %.2f%n", classAverageGpa);
            } else {
                System.out.println("No valid student records found in file.");
            }
            System.out.println("==========================================");

        } catch (IOException e) {
            System.err.println("File Import Error: Unable to read file " + csvFile.getName() + " - " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        File sampleCsvFile = new File("scratch/students.csv");

        // Generate sample CSV file for demonstration
        try (PrintWriter writer = new PrintWriter(sampleCsvFile)) {
            writer.println("# Student Records CSV File");
            writer.println("101,Alice Smith,Computer Sci,3.85");
            writer.println("102,Bob Jones,Electrical Eng,3.40");
            writer.println("103,Charlie Brown,Mathematics,3.92");
            writer.println("104,Diana Prince,Physics,3.65");
        } catch (IOException e) {
            System.err.println("Failed to setup sample CSV: " + e.getMessage());
        }

        importAndReportStudentData(sampleCsvFile);
    }
}
```

### Explanation

1. **Nested Scanner Architecture**:
   - `fileScanner` reads the CSV file line-by-line using `hasNextLine()` and `nextLine()`.
   - A secondary `lineScanner` parses tokens within each individual line using comma delimiters (`lineScanner.useDelimiter(",")`).

2. **Safe Token Parsing**:
   - `lineScanner.hasNextInt()` verifies the line starts with a valid numeric ID before reading `studentId`, `name`, `major`, and `gpa`.
