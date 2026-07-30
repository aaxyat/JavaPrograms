# Unit 1: Introduction to Java

## Subheading: Working with Collections

The **Java Collections Framework** (`java.util`) provides an architecture to store, manipulate, and operate on groups of objects. It offers standardized interfaces and implementations for lists, sets, and key-value maps.

### Key Concepts
1. **`List` Interface (`ArrayList`, `LinkedList`)**:
   - Ordered collection that allows duplicate elements.
   - `ArrayList`: Fast indexed access (`O(1)`), slower middle insertion/deletion.
   - `LinkedList`: Fast insertion/deletion at endpoints (`O(1)`), slower indexed access (`O(n)`).
2. **`Set` Interface (`HashSet`, `TreeSet`)**:
   - Collection that contains **no duplicate elements**.
   - `HashSet`: Unordered, offers `O(1)` lookup performance.
   - `TreeSet`: Sorted set maintaining elements in ascending natural order (`O(log n)`).
3. **`Map` Interface (`HashMap`, `TreeMap`)**:
   - Maps unique **Keys** to **Values** (`Key -> Value`).
   - `HashMap`: Unordered key-value pairs providing constant-time lookup by key.
4. **Traversal**: Iterating collections via `Iterator`, enhanced for-loops, or `.forEach()` methods.

---

## 1. Demo Program: List, Set, and Map Operations

**Filename:** `CollectionsBasicsDemo.java`

### Source Code
```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.HashMap;
import java.util.List;
import java.util.Set;
import java.util.Map;
import java.util.Iterator;

/**
 * Program: CollectionsBasicsDemo.java
 * Teaches: Core interfaces of Java Collections Framework: List (ArrayList), Set (HashSet), and Map (HashMap).
 * Usage: Demonstrates adding, removing, duplicate handling, key-value lookup, and iteration techniques.
 */
public class CollectionsBasicsDemo {
    public static void main(String[] args) {
        
        // 1. List Demo (ArrayList - Ordered, Allows Duplicates)
        System.out.println("=== 1. List Operations (ArrayList) ===");
        List<String> cities = new ArrayList<>();
        cities.add("New York");
        cities.add("London");
        cities.add("Tokyo");
        cities.add("London"); // Duplicate allowed

        System.out.println("Cities List: " + cities);
        System.out.println("Element at index 1: " + cities.get(1));

        // 2. Set Demo (HashSet - Unordered, Unique Elements Only)
        System.out.println("\n=== 2. Set Operations (HashSet) ===");
        Set<String> uniqueTags = new HashSet<>();
        uniqueTags.add("Java");
        uniqueTags.add("OOP");
        uniqueTags.add("Collections");
        uniqueTags.add("Java"); // Duplicate ignored automatically

        System.out.println("Unique Tags Set: " + uniqueTags);
        System.out.println("Contains 'OOP'? " + uniqueTags.contains("OOP"));

        // 3. Map Demo (HashMap - Key-Value Pairs)
        System.out.println("\n=== 3. Map Operations (HashMap) ===");
        Map<Integer, String> studentMap = new HashMap<>();
        studentMap.put(101, "Alice");
        studentMap.put(102, "Bob");
        studentMap.put(103, "Charlie");

        System.out.println("Student ID 102 Name: " + studentMap.get(102));

        // 4. Traversing Map Entries
        System.out.println("\n=== 4. Map Entry Traversal ===");
        for (Map.Entry<Integer, String> entry : studentMap.entrySet()) {
            System.out.println("ID: " + entry.getKey() + " | Name: " + entry.getValue());
        }

        // 5. Iterator Traversal Demo
        System.out.println("\n=== 5. Traversing List with Iterator ===");
        Iterator<String> iterator = cities.iterator();
        while (iterator.hasNext()) {
            System.out.println("City Item: " + iterator.next());
        }
    }
}
```

### Code Explanation
1. **List Operations (`ArrayList`)**:
   - Preserves insertion order and permits duplicates (`"London"` appears twice).
2. **Set Duplicate Filtering (`HashSet`)**:
   - Automatically rejects duplicate entries (`"Java"` added twice result in a single store).
3. **Map Key-Value Lookup (`HashMap.get(key)`)**:
   - Retrieves values instantaneously (`O(1)`) using unique key identifiers (`studentMap.get(102)` returns `"Bob"`).
4. **Map Entry Iteration (`studentMap.entrySet()`)**:
   - Accesses key-value pairs cleanly using `Map.Entry` objects.

---

## 2. Real-World Program: Library Catalog & Member Borrowing System

**Filename:** `LibraryCatalogManager.java`

### Source Code
```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.HashMap;
import java.util.List;
import java.util.Set;
import java.util.Map;
import java.util.Scanner;

/**
 * Program: LibraryCatalogManager.java
 * Teaches: Combining List, Set, and Map to build a real-world Library Management System.
 * Example Input/Output:
 *   Register Member 501, Borrow "Java Concurrency", View Borrowed Books
 *   Output: Member 501 Borrowed List: ["Java Concurrency"], Available Stock updated.
 */
public class LibraryCatalogManager {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // 1. List: Catalog of all available book titles
        List<String> bookCatalog = new ArrayList<>();
        bookCatalog.add("Effective Java");
        bookCatalog.add("Clean Code");
        bookCatalog.add("Java Concurrency in Practice");
        bookCatalog.add("Design Patterns");

        // 2. Set: Unique registered member IDs
        Set<Integer> registeredMembers = new HashSet<>();
        registeredMembers.add(1001);
        registeredMembers.add(1002);
        registeredMembers.add(1003);

        // 3. Map: Member ID -> List of borrowed book titles
        Map<Integer, List<String>> memberBorrowMap = new HashMap<>();

        System.out.println("==========================================");
        System.out.println("       CITY CENTRAL LIBRARY SYSTEM        ");
        System.out.println("==========================================");

        boolean active = true;

        while (active) {
            System.out.println("\n--- MAIN MENU ---");
            System.out.println("1. View Available Book Catalog");
            System.out.println("2. Borrow a Book");
            System.out.println("3. View Member Borrowed Books");
            System.out.println("4. Exit");
            System.out.print("Select Choice (1-4): ");

            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1:
                    System.out.println("\n--- CURRENT BOOK CATALOG (" + bookCatalog.size() + " Titles) ---");
                    for (int i = 0; i < bookCatalog.size(); i++) {
                        System.out.println("  " + (i + 1) + ". " + bookCatalog.get(i));
                    }
                    break;

                case 2:
                    System.out.print("Enter Member ID: ");
                    int memberId = scanner.nextInt();
                    scanner.nextLine();

                    // Check member registration via Set
                    if (!registeredMembers.contains(memberId)) {
                        System.out.println("ACCESS DENIED: Member ID #" + memberId + " is not registered.");
                        break;
                    }

                    System.out.print("Enter Book Title to Borrow: ");
                    String bookTitle = scanner.nextLine().trim();

                    // Search book catalog via List
                    if (!bookCatalog.contains(bookTitle)) {
                        System.out.println("ERROR: Book '" + bookTitle + "' is not available in catalog.");
                        break;
                    }

                    // Process borrow action: Add to Map
                    memberBorrowMap.putIfAbsent(memberId, new ArrayList<>());
                    memberBorrowMap.get(memberId).add(bookTitle);
                    bookCatalog.remove(bookTitle); // Remove from available catalog

                    System.out.println("SUCCESS: Book '" + bookTitle + "' issued to Member #" + memberId);
                    break;

                case 3:
                    System.out.print("Enter Member ID to query: ");
                    int queryId = scanner.nextInt();

                    List<String> borrowed = memberBorrowMap.get(queryId);
                    if (borrowed == null || borrowed.isEmpty()) {
                        System.out.println("Member #" + queryId + " has no borrowed books.");
                    } else {
                        System.out.println("Books borrowed by Member #" + queryId + ": " + borrowed);
                    }
                    break;

                case 4:
                    System.out.println("Thank you for using City Central Library System!");
                    active = false;
                    break;

                default:
                    System.out.println("Invalid choice selected.");
                    break;
            }
        }

        scanner.close();
    }
}
```

### Code Explanation
1. **Catalog Management (`List<String> bookCatalog`)**:
   - Maintains available inventory titles, allowing dynamic removal upon borrowing.
2. **Membership Authentication (`Set<Integer> registeredMembers`)**:
   - Provides instantaneous constant-time check (`registeredMembers.contains(memberId)`) to verify member access rights.
3. **Relation Mapping (`Map<Integer, List<String>> memberBorrowMap`)**:
   - Connects each `memberId` to their personal dynamic `List` of borrowed book titles using `.putIfAbsent()`.
