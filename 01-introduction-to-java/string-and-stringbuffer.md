# Unit 1: Introduction to Java

## Subheading: String and StringBuffer

In Java, text is represented primarily using the `String` and `StringBuffer` classes. Understanding the difference between immutable strings and mutable string buffers is essential for efficient memory management and performance.

### Key Concepts
1. **`String` Immutability**: `String` objects are immutable—once created, their values cannot be changed. String modifications create new objects in memory (String Constant Pool).
2. **`StringBuffer` Mutability**: `StringBuffer` represents a mutable sequence of characters. It modifies the underlying character array directly without creating new object instances, making it thread-safe and memory-efficient for heavy modifications.
3. **Comparison (`equals()` vs `==`)**:
   - `==` compares object references (memory address memory location).
   - `.equals()` compares actual character content sequence.
4. **Key Methods**:
   - **`String`**: `.length()`, `.charAt()`, `.substring()`, `.indexOf()`, `.trim()`, `.toLowerCase()`, `.equals()`
   - **`StringBuffer`**: `.append()`, `.insert()`, `.replace()`, `.delete()`, `.reverse()`

---

## 1. Demo Program: String Immutability vs StringBuffer Mutability

**Filename:** `StringVsStringBufferDemo.java`

### Source Code
```java
/**
 * Program: StringVsStringBufferDemo.java
 * Teaches: Immutability of String vs Mutability of StringBuffer, memory pool behaviors, and core method manipulation.
 * Usage: Demonstrates string operations, equality checks, and thread-safe buffer modifications.
 */
public class StringVsStringBufferDemo {
    public static void main(String[] args) {
        
        // 1. String Immutability Demo
        System.out.println("=== 1. String Immutability ===");
        String str1 = "Hello";
        String str2 = str1.concat(" World");
        
        System.out.println("Original str1: " + str1); // Output: Hello (Unchanged)
        System.out.println("New str2     : " + str2); // Output: Hello World

        // 2. Reference vs Value Comparison (String Constant Pool)
        System.out.println("\n=== 2. String Comparison (== vs .equals()) ===");
        String s1 = "Java";
        String s2 = "Java";
        String s3 = new String("Java");

        System.out.println("s1 == s2      : " + (s1 == s2));      // true (Same Literal Pool reference)
        System.out.println("s1 == s3      : " + (s1 == s3));      // false (Different heap memory reference)
        System.out.println("s1.equals(s3) : " + s1.equals(s3));  // true (Same text content)

        // 3. StringBuffer Mutability Demo
        System.out.println("\n=== 3. StringBuffer Operations ===");
        StringBuffer buffer = new StringBuffer("Java");
        
        buffer.append(" Programming");
        System.out.println("After append : " + buffer);

        buffer.insert(4, " Language");
        System.out.println("After insert : " + buffer);

        buffer.reverse();
        System.out.println("After reverse: " + buffer);
    }
}
```

### Code Explanation
1. **String Immutability (`str1.concat(...)`)**:
   - Calling `.concat()` does *not* alter `str1`. It creates a brand-new `String` object assigned to `str2`.
2. **String Pool & Memory Comparison (`==` vs `.equals()`)**:
   - `s1 == s2` is `true` because string literals are cached in the String Constant Pool.
   - `s1 == s3` is `false` because `new String()` forces creation of a separate object in non-pool heap memory.
   - `s1.equals(s3)` checks the character sequence, evaluating to `true`.
3. **StringBuffer In-Place Modifications**:
   - `.append()`, `.insert()`, and `.reverse()` modify the existing buffer directly without allocating new objects.

---

## 2. Real-World Program: Text Sanitizer & Audit Log Builder

**Filename:** `TextSanitizerAndFormatter.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: TextSanitizerAndFormatter.java
 * Teaches: Validating user inputs with String methods and dynamically assembling secure audit logs with StringBuffer.
 * Example Input/Output:
 *   Input Username: "  user_admin_99  ", Input Action: "UPDATE_PASSWORD"
 *   Output: Cleaned Username: USER_ADMIN_99, Formatted Log Entry: [LOG-TIMESTAMP] USER: USER_ADMIN_99 | ACTION: UPDATE_PASSWORD | STATUS: SUCCESS
 */
public class TextSanitizerAndFormatter {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("   USER SANITIZER & AUDIT LOG BUILDER     ");
        System.out.println("==========================================");

        // Step 1: Input Raw Username
        System.out.print("Enter raw username: ");
        String rawUsername = scanner.nextLine();

        // Validation using String methods
        String cleanedUsername = rawUsername.trim().toUpperCase();

        if (cleanedUsername.isEmpty()) {
            System.out.println("ERROR: Username cannot be blank.");
            scanner.close();
            return;
        }

        if (cleanedUsername.contains(" ")) {
            System.out.println("ERROR: Username must not contain spaces.");
            scanner.close();
            return;
        }

        System.out.println("Cleaned Username: " + cleanedUsername);

        // Step 2: Input Action
        System.out.print("Enter Action Performed (e.g. LOGIN, UPDATE_PASSWORD): ");
        String action = scanner.nextLine().trim();

        // Step 3: Dynamically Construct Audit Log Entry using StringBuffer
        StringBuffer logBuffer = new StringBuffer();

        long timestamp = System.currentTimeMillis();
        
        logBuffer.append("[LOG-")
                 .append(timestamp)
                 .append("] USER: ")
                 .append(cleanedUsername)
                 .append(" | ACTION: ")
                 .append(action)
                 .append(" | STATUS: SUCCESS");

        System.out.println("\n==========================================");
        System.out.println("         GENERATED AUDIT LOG RECORD       ");
        System.out.println("==========================================");
        System.out.println(logBuffer.toString());

        // Demonstrate StringBuffer reversal for audit signature hashing simulation
        StringBuffer signatureBuffer = new StringBuffer(cleanedUsername + ":" + timestamp);
        signatureBuffer.reverse();
        System.out.println("Security Reverse Hash Signature: " + signatureBuffer.toString());
        System.out.println("==========================================");

        scanner.close();
    }
}
```

### Code Explanation
1. **Input Sanitization (`trim()` & `toUpperCase()`)**:
   - `rawUsername.trim()` removes accidental leading and trailing whitespace.
   - `.toUpperCase()` normalizes input for consistent database queries.
2. **String Validation Checks (`isEmpty()`, `contains()`)**:
   - `.isEmpty()` guards against blank submissions.
   - `.contains(" ")` ensures the username is a single continuous token.
3. **Efficient Log Construction (`StringBuffer.append()`)**:
   - Chaining `.append()` calls builds the full log entry dynamically without creating wasteful intermediate `String` objects during concatenation.
4. **Buffer Reversal (`signatureBuffer.reverse()`)**:
   - In-place string reversal generates a mock security token efficiently.
