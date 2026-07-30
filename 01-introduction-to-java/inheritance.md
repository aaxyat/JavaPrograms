# Unit 1: Introduction to Java

## Subheading: Inheritance

**Inheritance** is a core Object-Oriented Programming pillar that allows a child class (subclass) to inherit attributes and methods from a parent class (superclass) using the `extends` keyword. It promotes code reusability and establishes hierarchical relationships.

### Key Concepts
1. **Superclass & Subclass**:
   - **Superclass (Parent)**: The base class whose properties are inherited.
   - **Subclass (Child)**: The derived class that inherits from the superclass and can add custom fields/methods.
2. **Types of Inheritance Supported**:
   - **Single Inheritance**: Class B extends Class A.
   - **Multilevel Inheritance**: Class C extends Class B, which extends Class A.
   - **Hierarchical Inheritance**: Classes B and C both extend Class A.
   - *(Note: Multiple class inheritance is not supported in Java to prevent the Diamond Problem).*
3. **The `super` Keyword**:
   - `super()` calls the constructor of the parent class.
   - `super.variable` or `super.method()` accesses superclass members hidden or overridden by the subclass.
4. **Method Overriding (`@Override`)**: A subclass provides a specific implementation of a method already defined in its superclass with identical signature.

---

## 1. Demo Program: Inheritance Types & Method Overriding

**Filename:** `InheritanceBasicsDemo.java`

### Source Code
```java
/**
 * Program: InheritanceBasicsDemo.java
 * Teaches: Superclass/subclass hierarchy, single and multilevel inheritance, super keyword, and method overriding.
 * Usage: Demonstrates attribute inheritance and polymorphic method execution.
 */

// Superclass (Parent)
class Animal {
    String name;

    public Animal(String name) {
        this.name = name;
    }

    public void makeSound() {
        System.out.println(name + " makes a generic animal sound.");
    }
}

// Single Inheritance: Mammal extends Animal
class Mammal extends Animal {
    boolean hasFur;

    public Mammal(String name, boolean hasFur) {
        super(name); // Call Superclass Constructor
        this.hasFur = hasFur;
    }

    public void displayMammalInfo() {
        System.out.println("Mammal Name: " + name + " | Has Fur: " + hasFur);
    }
}

// Multilevel Inheritance: Dog extends Mammal
class Dog extends Mammal {
    String breed;

    public Dog(String name, boolean hasFur, String breed) {
        super(name, hasFur); // Call Mammal Constructor
        this.breed = breed;
    }

    // Method Overriding
    @Override
    public void makeSound() {
        System.out.println(name + " (" + breed + ") barks: Woof! Woof!");
    }
}

public class InheritanceBasicsDemo {
    public static void main(String[] args) {
        
        System.out.println("=== 1. Superclass Object ===");
        Animal genericAnimal = new Animal("Generic Creature");
        genericAnimal.makeSound();

        System.out.println("\n=== 2. Multilevel Inheritance & Overriding ===");
        Dog myDog = new Dog("Buddy", true, "Golden Retriever");
        myDog.displayMammalInfo(); // Inherited from Mammal
        myDog.makeSound();        // Overridden in Dog
    }
}
```

### Code Explanation
1. **Superclass Constructor Invocation (`super(name)`)**:
   - `Mammal` uses `super(name)` to pass the `name` attribute to the `Animal` base class constructor.
2. **Multilevel Extension (`Dog -> Mammal -> Animal`)**:
   - `Dog` inherits all fields and methods from both `Mammal` (`hasFur`, `displayMammalInfo()`) and `Animal` (`name`).
3. **Method Overriding (`@Override makeSound()`)**:
   - `Dog` replaces the generic `Animal.makeSound()` behavior with specific barking functionality.

---

## 2. Real-World Program: Corporate Employee Payroll System

**Filename:** `EmployeePayrollSystem.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: EmployeePayrollSystem.java
 * Teaches: Modeling a real-world enterprise payroll hierarchy using inheritance (Employee -> FullTimeEmployee / ContractEmployee).
 * Example Input/Output:
 *   Full-Time: Base $5000, Bonus $1000 -> Total: $6000.00
 *   Contract: $50/hr * 120 hrs -> Total: $6000.00
 */

// Base Superclass
class Employee {
    protected int empId;
    protected String name;
    protected String department;

    public Employee(int empId, String name, String department) {
        this.empId = empId;
        this.name = name;
        this.department = department;
    }

    public double calculatePay() {
        return 0.0; // Default implementation to be overridden
    }

    public void printPayStub() {
        System.out.println("\n------------------------------------------");
        System.out.println("              PAYROLL STUB                ");
        System.out.println("------------------------------------------");
        System.out.println("Employee ID : " + empId);
        System.out.println("Name        : " + name);
        System.out.println("Department  : " + department);
        System.out.printf("Net Pay     : $%.2f%n", calculatePay());
        System.out.println("------------------------------------------");
    }
}

// Subclass 1: Full-Time Employee (Salaried + Bonus)
class FullTimeEmployee extends Employee {
    private double monthlySalary;
    private double performanceBonus;

    public FullTimeEmployee(int empId, String name, String department, double monthlySalary, double performanceBonus) {
        super(empId, name, department);
        this.monthlySalary = monthlySalary;
        this.performanceBonus = performanceBonus;
    }

    @Override
    public double calculatePay() {
        return monthlySalary + performanceBonus;
    }
}

// Subclass 2: Contract Employee (Hourly Rate * Hours Worked)
class ContractEmployee extends Employee {
    private double hourlyRate;
    private int hoursWorked;

    public ContractEmployee(int empId, String name, String department, double hourlyRate, int hoursWorked) {
        super(empId, name, department);
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
    }

    @Override
    public double calculatePay() {
        return hourlyRate * hoursWorked;
    }
}

public class EmployeePayrollSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("      ENTERPRISE PAYROLL SYSTEM           ");
        System.out.println("==========================================");

        // 1. Instantiate Full-Time Employee
        System.out.println("\n--- Full-Time Employee Entry ---");
        System.out.print("Enter ID: ");
        int ftId = scanner.nextInt();
        scanner.nextLine();
        System.out.print("Enter Name: ");
        String ftName = scanner.nextLine();
        System.out.print("Enter Monthly Salary ($): ");
        double salary = scanner.nextDouble();
        System.out.print("Enter Bonus ($): ");
        double bonus = scanner.nextDouble();

        FullTimeEmployee ftEmp = new FullTimeEmployee(ftId, ftName, "Engineering", salary, bonus);

        // 2. Instantiate Contract Employee
        System.out.println("\n--- Contract Employee Entry ---");
        System.out.print("Enter ID: ");
        int ctId = scanner.nextInt();
        scanner.nextLine();
        System.out.print("Enter Name: ");
        String ctName = scanner.nextLine();
        System.out.print("Enter Hourly Rate ($): ");
        double rate = scanner.nextDouble();
        System.out.print("Enter Hours Worked: ");
        int hours = scanner.nextInt();

        ContractEmployee ctEmp = new ContractEmployee(ctId, ctName, "Consulting", rate, hours);

        // Print Pay Stubs
        ftEmp.printPayStub();
        ctEmp.printPayStub();

        scanner.close();
    }
}
```

### Code Explanation
1. **Superclass Abstraction (`Employee`)**:
   - Encapsulates common employee metadata (`empId`, `name`, `department`) and shared behavior (`printPayStub()`).
2. **Specialized Subclasses (`FullTimeEmployee` & `ContractEmployee`)**:
   - `FullTimeEmployee` calculates compensation as `monthlySalary + performanceBonus`.
   - `ContractEmployee` calculates compensation as `hourlyRate * hoursWorked`.
3. **Polymorphic Method Execution**:
   - Both subclasses override `calculatePay()`, allowing `printPayStub()` to compute distinct net pay figures seamlessly.
