# Unit 1: Introduction to Java

## Subheading: Array

An **Array** is a fixed-size, indexed data structure that stores multiple elements of the same data type in contiguous memory locations. Arrays provide efficient indexed access to collections of data.

### Key Concepts
1. **Array Declaration & Initialization**: Declared using brackets (e.g., `int[] numbers = new int[5];`) or directly with element literals (`int[] scores = {90, 85, 88};`).
2. **Zero-Based Indexing**: Array elements are accessed using zero-based indices from `0` to `length - 1`. Accessing invalid indices throws `ArrayIndexOutOfBoundsException`.
3. **Array Length Property**: The `array.length` property provides the total number of allocated slots.
4. **2D Arrays**: Multi-dimensional arrays (arrays of arrays) used to represent grid data, tables, or matrices.

---

## 1. Demo Program: Array Operations & Traversal

**Filename:** `ArrayBasicsDemo.java`

### Source Code
```java
/**
 * Program: ArrayBasicsDemo.java
 * Teaches: Declaring, initializing, traversing 1D arrays, calculating totals, and working with 2D matrices.
 * Usage: Demonstrates array indexing, enhanced for-loop, and 2D grid iteration.
 */
public class ArrayBasicsDemo {
    public static void main(String[] args) {
        
        // 1. 1D Array Initialization & Traversal
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

        // 3. 2D Array Matrix Demo
        System.out.println("\n=== 3. 2D Array (3x3 Matrix Grid) ===");
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

### Code Explanation
1. **1D Array Traversal (`for (int i = 0; i < numbers.length; i++)`)**:
   - Uses `numbers.length` to dynamically determine the array size, preventing index out-of-bounds errors.
   - Computes running `sum` and calculates `average` by casting `sum` to `double`.
2. **Enhanced For-Loop (`for (String fruit : fruits)`)**:
   - Simplifies element access without managing manual index variables.
3. **2D Array Nested Iteration (`matrix[row][col]`)**:
   - Outer loop iterates through rows (`matrix.length`), inner loop iterates through columns (`matrix[row].length`) to output matrix elements in a tab-separated format.

---

## 2. Real-World Program: Student Grade Tracker System

**Filename:** `StudentGradeTracker.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: StudentGradeTracker.java
 * Teaches: Storing, searching, and computing statistics (highest, lowest, average) on data stored in an array.
 * Example Input/Output:
 *   Input: 5 student scores -> [78.5, 92.0, 64.0, 88.5, 95.0]
 *   Output: Highest Score: 95.0, Lowest Score: 64.0, Average: 83.6
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

        // Allocate array dynamically based on user input
        double[] grades = new double[totalStudents];

        // Step 1: Input Student Grades into Array
        System.out.println("\nEnter grades for " + totalStudents + " students:");
        for (int i = 0; i < grades.length; i++) {
            System.out.print("Student #" + (i + 1) + " Score: ");
            grades[i] = scanner.nextDouble();
        }

        // Step 2: Compute Highest, Lowest, and Average
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

        // Step 3: Print Class Summary Report
        System.out.println("\n==========================================");
        System.out.println("            CLASS SUMMARY REPORT          ");
        System.out.println("==========================================");
        System.out.printf("Total Students Enrolled : %d%n", totalStudents);
        System.out.printf("Class Average Score     : %.2f%n", classAverage);
        System.out.printf("Highest Score Achieved  : %.2f%n", highest);
        System.out.printf("Lowest Score Achieved   : %.2f%n", lowest);

        // Step 4: Search Operation in Array
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

### Code Explanation
1. **Dynamic Array Allocation (`new double[totalStudents]`)**:
   - Prompts user for class size and allocates array space dynamically at runtime.
2. **Array Input Population (`grades[i] = scanner.nextDouble()`)**:
   - Loops from index `0` to `grades.length - 1` to fill the array with student marks.
3. **Statistical Aggregation (`highest`, `lowest`, `totalSum`)**:
   - Initializes `highest` and `lowest` to the first element `grades[0]`.
   - Iterates using for-each loop to update maximum/minimum values and accumulate `totalSum`.
4. **Array Search & Filter (`grade >= threshold`)**:
   - Scans through the array elements to count how many students met or exceeded the passing threshold.
