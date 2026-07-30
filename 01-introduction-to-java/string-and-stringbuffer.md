# Unit 1: Introduction to Java

## String and StringBuffer

Java handles text primarily through two classes in `java.lang`: `String` and `StringBuffer`. The fundamental difference lies in **mutability**—whether a text object can be modified in memory after creation.

---

### Core Concepts

#### 1. `String` Immutability & String Constant Pool
- `String` objects are **immutable**. Once created, their character contents cannot be altered.
- **String Constant Pool**: To save memory, Java stores literal strings (e.g. `"Java"`) in a special pool inside heap memory. If two variables are initialized with the same string literal, Java points both variables to the exact same memory reference in the pool.
- **String Operations Allocation**: Modifying a `String` (e.g., calling `.concat()` or `.toUpperCase()`) does not alter the original string. It creates a brand-new `String` object in memory.

#### 2. `StringBuffer` Mutability
- `StringBuffer` represents a **mutable** character sequence.
- Modifying a `StringBuffer` (e.g., `.append()` or `.insert()`) edits the internal character buffer directly without allocating new objects.
- **Thread-Safety**: Methods in `StringBuffer` are synchronized, making it safe for concurrent multi-threaded text processing.

#### 3. Comparing Strings (`==` vs `.equals()`)
- **`==` Operator**: Checks if two variables refer to the **same memory address**.
- **`.equals()` Method**: Checks if two strings contain the **same sequence of characters**.
- Always use `.equals()` when comparing string values.

#### 4. Essential Methods Quick Reference

| Class | Method | Description |
|-------|--------|-------------|
| `String` | `.length()` | Returns character count. |
| `String` | `.charAt(index)` | Returns character at specified zero-based index. |
| `String` | `.trim()` | Removes leading and trailing whitespace. |
| `String` | `.substring(start, end)` | Extracts a section of text from `start` index up to `end - 1`. |
| `StringBuffer` | `.append(val)` | Appends text to the end of the existing buffer. |
| `StringBuffer` | `.insert(offset, val)` | Inserts text at a specific index offset. |
| `StringBuffer` | `.reverse()` | Reverses characters in place. |

---

### Common Pitfalls

1. **Expecting `.concat()` or `.replace()` to modify the original string**:
   - `String s = "hello"; s.concat(" world");` leaves `s` as `"hello"`. You must capture the returned reference: `s = s.concat(" world");`.
2. **Using `==` to compare user input strings**:
   - Strings created at runtime via `Scanner` reside outside the String Constant Pool. `input == "secret"` returns `false` even if the user typed `"secret"`. Use `input.equals("secret")`.
3. **Using string concatenation inside long loops**:
   - Writing `String s = ""; for (...) { s += i; }` creates thousands of temporary `String` objects in memory, severely slowing down execution. Use `StringBuffer` or `StringBuilder` inside loops instead.

---

## 1. Demo: String Immutability vs. StringBuffer Mutability

### `StringVsStringBufferDemo.java`

```java
/**
 * Demonstrates String immutability, reference vs value equality, and StringBuffer in-place edits.
 */
public class StringVsStringBufferDemo {
    public static void main(String[] args) {
        
        // 1. String Immutability
        System.out.println("=== 1. String Immutability ===");
        String str1 = "Hello";
        String str2 = str1.concat(" World");
        
        System.out.println("Original str1: " + str1); // Hello
        System.out.println("New str2     : " + str2); // Hello World

        // 2. Reference vs Value Comparison
        System.out.println("\n=== 2. String Comparison (== vs .equals()) ===");
        String s1 = "Java";
        String s2 = "Java";
        String s3 = new String("Java");

        System.out.println("s1 == s2      : " + (s1 == s2));      // true (Same literal pool reference)
        System.out.println("s1 == s3      : " + (s1 == s3));      // false (Different heap object)
        System.out.println("s1.equals(s3) : " + s1.equals(s3));  // true (Same text content)

        // 3. StringBuffer Operations
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

### Detailed Code Walkthrough

1. **Immutability Behavior**:
   - `str1.concat(" World")` leaves `str1` unchanged as `"Hello"`. The resulting combined string `"Hello World"` is assigned to new variable `str2`.

2. **String Pool vs. Heap Memory**:
   - `s1` and `s2` share the literal `"Java"` in the String Constant Pool, so `s1 == s2` returns `true`.
   - `s3 = new String("Java")` forces creation of a new object on heap memory outside the pool. `s1 == s3` evaluates to `false` because they occupy different memory addresses.
   - `s1.equals(s3)` evaluates to `true` because both contain the character sequence `'J', 'a', 'v', 'a'`.

3. **StringBuffer Modification**:
   - `buffer.append(" Programming")` modifies the original memory buffer directly. Printing `buffer` displays `"Java Programming"`.
   - `buffer.insert(4, " Language")` inserts text starting at index 4.
   - `buffer.reverse()` reverses all characters in place.

---

## 2. Real-World Program: Text Sanitizer & Audit Log Builder

### `TextSanitizerAndFormatter.java`

```java
import java.util.Scanner;

/**
 * Validates usernames using String methods and formats audit logs using StringBuffer.
 *
 * Example:
 *   Input: Username "  user_admin_99  ", Action "UPDATE_PASSWORD"
 *   Output: Cleaned: USER_ADMIN_99, Log: [LOG-TIMESTAMP] USER: USER_ADMIN_99 | ACTION: UPDATE_PASSWORD | STATUS: SUCCESS
 */
public class TextSanitizerAndFormatter {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("   USER SANITIZER & AUDIT LOG BUILDER     ");
        System.out.println("==========================================");

        System.out.print("Enter raw username: ");
        String rawUsername = scanner.nextLine();

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

        System.out.print("Enter Action Performed (e.g. LOGIN, UPDATE_PASSWORD): ");
        String action = scanner.nextLine().trim();

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

        StringBuffer signatureBuffer = new StringBuffer(cleanedUsername + ":" + timestamp);
        signatureBuffer.reverse();
        System.out.println("Security Hash Signature: " + signatureBuffer.toString());
        System.out.println("==========================================");

        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **Input Sanitization**:
   - `rawUsername.trim()` strips whitespace from both ends of the input.
   - `.toUpperCase()` converts all letters to uppercase to standardize formatting.

2. **Validation Rules**:
   - `.isEmpty()` checks if the string length is 0.
   - `.contains(" ")` verifies that the sanitized username contains no internal spaces.

3. **Building Logs Efficiently**:
   - `logBuffer.append(...)` constructs a multi-part audit log without generating temporary `String` objects during string concatenation.
