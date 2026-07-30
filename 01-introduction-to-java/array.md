# Unit 1: Introduction to Java

## Arrays

An **Array** is a fixed-size container that holds multiple values of the same data type in a contiguous block of memory. Instead of declaring separate variables like `score1`, `score2`, and `score3`, an array lets you store them under a single variable name and access items using numerical index positions.

---

### Core Concepts

#### 1. Array Declaration & Initialization
- **Declaration with explicit size**: Allocates empty memory slots initialized with default values (`0` for numeric types, `false` for booleans, `null` for objects):
  ```java
  int[] numbers = new int[5]; // Array of 5 integers, indices 0 through 4
  ```
- **Direct literal initialization**: Allocates memory and populates values in one step:
  ```java
  int[] scores = {90, 85, 88}; // Length is automatically 3
  ```

#### 2. Zero-Based Indexing
- Array positions start at index `0` and end at index `length - 1`.
- Accessing or modifying elements uses bracket notation: `scores[0] = 95;`.
- Attempting to access an index outside `0` to `length - 1` (e.g. `scores[3]` on a 3-element array) throws an `ArrayIndexOutOfBoundsException` at runtime.

#### 3. The `length` Property
- `array.length` returns the fixed number of element slots allocated in memory.
- Note: `length` is a property for arrays (no parentheses), unlike `String.length()` which is a method call.

#### 4. Multi-Dimensional (2D) Arrays
- A 2D array is an array of arrays, represented as a grid of rows and columns:
  ```java
  int[][] matrix = new int[3][3]; // 3 rows, 3 columns
  ```
- Elements are accessed using row and column indices: `matrix[row][col]`.

---

### Common Pitfalls

1. **Confusing array size with maximum index**:
   - An array of size 10 has valid indices from `0` to `9`. Trying to read `arr[10]` crashes with `ArrayIndexOutOfBoundsException`.
2. **Trying to resize an array after creation**:
   - Array sizes are fixed at allocation. To add more items than the array size permits, you must allocate a new larger array and copy existing elements over, or use dynamic data structures like `ArrayList`.
3. **Printing arrays directly with `System.out.println(array)`**:
   - Printing an array object directly outputs its memory hash reference (e.g. `[I@15db9742`), not its elements. Use a loop or `Arrays.toString(array)` to view array contents.

---

## 1. Demo: Array Traversal and Operations

### `ArrayBasicsDemo.java`

```java
/**
 * Demonstrates 1D array traversal, sum and average calculation, and 2D matrix iteration.
 */
public class ArrayBasicsDemo {
    public static void main(String[] args) {
        
        // 1. 1D Array Traversal
        System.out.println("=== 1. 1D Array Operations ===");
        int[] numbers = {12, 45, 67, 23, 89};
        
        int sum = 0;
        System.out.print("Array Elements: ");
        for (int i = 0; i < numbers.length; i++) {
            System.out.print(numbers[i] + " ");
            sum += numbers[i];
        }
        System.out.println("\nTotal Sum: " + sum);
        System.out.println("Average: " + ((double) sum / numbers.length));

        // 2. Enhanced For-Loop (For-Each)
        System.out.println("\n=== 2. Enhanced For-Loop (For-Each) ===");
        String[] fruits = {"Apple", "Banana", "Cherry"};
        for (String fruit : fruits) {
            System.out.println("Fruit: " + fruit);
        }

        // 3. 2D Array Matrix
        System.out.println("\n=== 3. 2D Array (3x3 Grid) ===");
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        for (int row = 0; row < matrix.length; row++) {
            for (int col = 0; col < matrix[row].length; col++) {
                System.out.print(matrix[row][col] + "\t");
            }
            System.out.println();
        }
    }
}
```

### Detailed Code Walkthrough

1. **1D Array Loop (`for (int i = 0; i < numbers.length; i++)`)**:
   - Starts `i` at 0 and stops when `i` reaches `numbers.length` (5).
   - `numbers[i]` retrieves the value stored at index `i`.
   - `(double) sum / numbers.length`: Casts `sum` to `double` before dividing so Java performs floating-point division instead of truncating decimal remainders.

2. **Enhanced For-Loop (`for (String fruit : fruits)`)**:
   - Reads as: *"For each String `fruit` in the array `fruits`"*.
   - Automatically iterates from index 0 to the end without managing a counter variable. Ideal when you only need to read elements without modifying indices.

3. **2D Array Nested Loop**:
   - `matrix.length` returns the number of rows (3).
   - `matrix[row].length` returns the number of columns in that specific row (3).
   - `matrix[row][col]` accesses the value at row `row` and column `col`. `\t` prints a tab character for grid alignment.

---

## 2. Real-World Program: Student Grade Tracker

### `StudentGradeTracker.java`

```java
import java.util.Scanner;

/**
 * Stores student scores in an array, computing highest, lowest, average marks, and pass counts.
 *
 * Example:
 *   Input: 5 scores -> 78.5, 92.0, 64.0, 88.5, 95.0
 *   Output: Average: 83.60, Highest: 95.00, Lowest: 64.00
 */
public class StudentGradeTracker {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("       STUDENT GRADE TRACKER SYSTEM       ");
        System.out.println("==========================================");

        System.out.print("Enter total number of students: ");
        int totalStudents = scanner.nextInt();

        if (totalStudents <= 0) {
            System.out.println("ERROR: Number of students must be greater than zero.");
            scanner.close();
            return;
        }

        double[] grades = new double[totalStudents];

        System.out.println("\nEnter grades for " + totalStudents + " students:");
        for (int i = 0; i < grades.length; i++) {
            System.out.print("Student #" + (i + 1) + " Score: ");
            grades[i] = scanner.nextDouble();
        }

        double highest = grades[0];
        double lowest = grades[0];
        double totalSum = 0;

        for (double grade : grades) {
            totalSum += grade;
            if (grade > highest) {
                highest = grade;
            }
            if (grade < lowest) {
                lowest = grade;
            }
        }

        double classAverage = totalSum / grades.length;

        System.out.println("\n==========================================");
        System.out.println("            CLASS SUMMARY REPORT          ");
        System.out.println("==========================================");
        System.out.printf("Total Students Enrolled : %d%n", totalStudents);
        System.out.printf("Class Average Score     : %.2f%n", classAverage);
        System.out.printf("Highest Score Achieved  : %.2f%n", highest);
        System.out.printf("Lowest Score Achieved   : %.2f%n", lowest);

        System.out.print("\nEnter passing threshold score to check (e.g. 70): ");
        double threshold = scanner.nextDouble();

        int passCount = 0;
        for (double grade : grades) {
            if (grade >= threshold) {
                passCount++;
            }
        }

        System.out.println("Students scoring >= " + threshold + ": " + passCount + " out of " + totalStudents);
        System.out.println("==========================================");

        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **Dynamic Memory Allocation (`new double[totalStudents]`)**:
   - Reads `totalStudents` from user input at runtime and allocates an array of that exact length.

2. **Populating Array Elements**:
   - `grades[i] = scanner.nextDouble();` places each entered grade directly into slot `i`.

3. **Finding Minimum, Maximum, and Sum**:
   - `double highest = grades[0];` and `double lowest = grades[0];`: Initializes tracking variables to the first element in the array.
   - During traversal, if a grade is larger than `highest`, `highest` updates to that grade. If smaller than `lowest`, `lowest` updates.

4. **Filtering and Threshold Search**:
   - Loops through `grades` comparing each against `threshold`. `passCount++` tracks how many students passed.
