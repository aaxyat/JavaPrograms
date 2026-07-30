# Unit 1: Introduction to Java

## Working with Collections

The **Java Collections Framework** (`java.util`) provides data structures and algorithms to store, group, and manipulate objects dynamically.

---

### Core Concepts

#### 1. `List` Interface (`ArrayList`, `LinkedList`)
- An ordered collection (also called a sequence) that allows **duplicate elements**.
- **`ArrayList`**: Resizable array implementation. Excellent for reading elements by index (`O(1)`), but slower when inserting or deleting elements in the middle of large lists (`O(n)`).
- **`LinkedList`**: Doubly-linked list implementation. Fast at adding or removing elements at the start or end (`O(1)`), but slower at random index access (`O(n)`).

#### 2. `Set` Interface (`HashSet`, `TreeSet`)
- A collection that contains **no duplicate elements**.
- **`HashSet`**: Stores elements in a hash table. Does not guarantee iteration order, but offers constant-time (`O(1)`) performance for add, remove, and search operations.
- **`TreeSet`**: Stores elements in a Red-Black tree. Keeps elements sorted in natural ascending order (or by a custom Comparator) with `O(log n)` performance.

#### 3. `Map` Interface (`HashMap`, `TreeMap`)
- Maps unique **Keys** to **Values** (`Key -> Value`). Maps cannot contain duplicate keys.
- **`HashMap`**: Unordered key-value storage offering constant-time (`O(1)`) lookups by key.
- **`TreeMap`**: Sorts entries by key order (`O(log n)`).

#### 4. Traversal Mechanisms
- **Enhanced For-Loop**: `for (String s : list)`
- **`Iterator`**: `Iterator<String> it = list.iterator();` (Allows removing items safely while looping).
- **`Map.Entry` Loop**: `for (Map.Entry<K, V> entry : map.entrySet())`

---

### Common Pitfalls

1. **Trying to store primitive types directly in Collections**:
   - Collections store objects, not primitives (`int`, `double`, `char`). Use wrapper classes (`Integer`, `Double`, `Character`). Java automatically handles boxing/unboxing: `List<Integer> numbers = new ArrayList<>();`.
2. **Modifying a collection inside a standard for-each loop**:
   - Calling `list.remove(item)` inside `for (String item : list)` throws `ConcurrentModificationException`. Use an `Iterator` and call `iterator.remove()` instead.

---

## 1. Demo: List, Set, and Map Operations

### `CollectionsBasicsDemo.java`

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.HashMap;
import java.util.List;
import java.util.Set;
import java.util.Map;
import java.util.Iterator;

/**
 * Demonstrates ArrayList, HashSet, HashMap, duplicate handling, key-value lookup, and iteration.
 */
public class CollectionsBasicsDemo {
    public static void main(String[] args) {
        
        // 1. List Demo (ArrayList)
        System.out.println("=== 1. List Operations (ArrayList) ===");
        List<String> cities = new ArrayList<>();
        cities.add("New York");
        cities.add("London");
        cities.add("Tokyo");
        cities.add("London"); // Duplicates allowed

        System.out.println("Cities List: " + cities);
        System.out.println("Element at index 1: " + cities.get(1));

        // 2. Set Demo (HashSet)
        System.out.println("\n=== 2. Set Operations (HashSet) ===");
        Set<String> uniqueTags = new HashSet<>();
        uniqueTags.add("Java");
        uniqueTags.add("OOP");
        uniqueTags.add("Collections");
        uniqueTags.add("Java"); // Duplicate ignored

        System.out.println("Unique Tags Set: " + uniqueTags);
        System.out.println("Contains 'OOP'? " + uniqueTags.contains("OOP"));

        // 3. Map Demo (HashMap)
        System.out.println("\n=== 3. Map Operations (HashMap) ===");
        Map<Integer, String> studentMap = new HashMap<>();
        studentMap.put(101, "Alice");
        studentMap.put(102, "Bob");
        studentMap.put(103, "Charlie");

        System.out.println("Student ID 102 Name: " + studentMap.get(102));

        // 4. Map Entry Traversal
        System.out.println("\n=== 4. Map Entry Traversal ===");
        for (Map.Entry<Integer, String> entry : studentMap.entrySet()) {
            System.out.println("ID: " + entry.getKey() + " | Name: " + entry.getValue());
        }

        // 5. Iterator Traversal
        System.out.println("\n=== 5. Traversing List with Iterator ===");
        Iterator<String> iterator = cities.iterator();
        while (iterator.hasNext()) {
            System.out.println("City Item: " + iterator.next());
        }
    }
}
```

### Detailed Code Walkthrough

1. **`List` Behavior**:
   - `cities.add("London")` adds `"London"` a second time. `ArrayList` maintains insertion order and retains both duplicates.

2. **`Set` Behavior**:
   - `uniqueTags.add("Java")` is executed twice, but `HashSet` ignores duplicate entries. Printing `uniqueTags` shows `"Java"` only once.

3. **`Map` Lookup**:
   - `studentMap.put(102, "Bob")` maps integer key `102` to `"Bob"`. `studentMap.get(102)` retrieves `"Bob"` in `O(1)` time.

---

## 2. Real-World Program: Library Catalog Manager

### `LibraryCatalogManager.java`

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.HashMap;
import java.util.List;
import java.util.Set;
import java.util.Map;
import java.util.Scanner;

/**
 * Library catalog system using List for inventory, Set for member IDs, and Map for borrowed books.
 *
 * Example:
 *   Member 1001 borrows "Clean Code"
 *   Output: Book issued, stock updated.
 */
public class LibraryCatalogManager {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        List<String> bookCatalog = new ArrayList<>();
        bookCatalog.add("Effective Java");
        bookCatalog.add("Clean Code");
        bookCatalog.add("Java Concurrency in Practice");
        bookCatalog.add("Design Patterns");

        Set<Integer> registeredMembers = new HashSet<>();
        registeredMembers.add(1001);
        registeredMembers.add(1002);
        registeredMembers.add(1003);

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

                    if (!registeredMembers.contains(memberId)) {
                        System.out.println("ACCESS DENIED: Member ID #" + memberId + " is not registered.");
                        break;
                    }

                    System.out.print("Enter Book Title to Borrow: ");
                    String bookTitle = scanner.nextLine().trim();

                    if (!bookCatalog.contains(bookTitle)) {
                        System.out.println("ERROR: Book '" + bookTitle + "' is not available in catalog.");
                        break;
                    }

                    memberBorrowMap.putIfAbsent(memberId, new ArrayList<>());
                    memberBorrowMap.get(memberId).add(bookTitle);
                    bookCatalog.remove(bookTitle);

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

### Detailed Code Walkthrough

1. **Combining Collections**:
   - `bookCatalog` (`List`): Holds available inventory.
   - `registeredMembers` (`Set`): Performs member authorization checks in `O(1)` time using `.contains(memberId)`.
   - `memberBorrowMap` (`Map<Integer, List<String>>`): Maps a member ID to their list of borrowed book titles. `putIfAbsent()` ensures a new `ArrayList` is created for first-time borrowers.
