# Unit 1: Introduction to Java

## Inheritance

**Inheritance** lets a child class (subclass) acquire fields and methods from a parent class (superclass) using the `extends` keyword. It forms hierarchical relationships and eliminates duplicate code across related classes.

---

### Core Concepts

#### 1. Superclass & Subclass
- **Superclass (Parent Class)**: The base class that contains general attributes and methods shared by child entities.
- **Subclass (Child Class)**: The specialized class that inherits members from the superclass using `extends`. It can add new fields/methods or override existing behavior.

#### 2. Supported Inheritance Types
- **Single Inheritance**: A class extends one direct parent class (`B extends A`).
- **Multilevel Inheritance**: A class extends a child class (`C extends B`, where `B extends A`). Class `C` inherits fields from both `B` and `A`.
- **Hierarchical Inheritance**: Multiple child classes extend the same parent class (`B extends A` and `C extends A`).
- **Why No Multiple Inheritance for Classes?**: Java does not allow a class to extend multiple parent classes (`C extends A, B`) to prevent the **Diamond Problem**—ambiguity over which parent method implementation to execute when both parents declare identical methods. Multiple behavior inheritance is handled via **Interfaces**.

#### 3. The `super` Keyword
- **`super()`**: Calls the parent class constructor. Must be the **very first statement** inside a child class constructor.
- **`super.method()`**: Invokes a parent class method that was overridden in the child class.

#### 4. Method Overriding (`@Override`)
- Occurs when a subclass defines a method with the **exact same name, return type, and parameters** as a method in its superclass.
- `@Override` annotation tells the compiler to check that the method matches a signature in the parent class, catching typos early.

---

### Common Pitfalls

1. **Forgetting that private parent members are not directly accessible in subclasses**:
   - `private` fields in a superclass belong exclusively to that superclass. Subclasses cannot access them directly unless the superclass provides `public` / `protected` getters or uses the `protected` modifier.
2. **Not placing `super()` on line 1 of a subclass constructor**:
   - Calling `super()` after initializing child fields causes a compilation error. Java requires the superclass part of an object to be constructed first.
3. **Confusing Overloading with Overriding**:
   - **Overloading**: Same method name, *different* parameters, within the *same* class (compile-time).
   - **Overriding**: Same method name, *identical* parameters, across *parent-child* classes (runtime).

---

## 1. Demo: Inheritance Hierarchy and Method Overriding

### `InheritanceBasicsDemo.java`

```java
/**
 * Demonstrates single and multilevel inheritance, constructor chaining with super(), and method overriding.
 */

class Animal {
    String name;

    public Animal(String name) {
        this.name = name;
    }

    public void makeSound() {
        System.out.println(name + " makes a generic animal sound.");
    }
}

class Mammal extends Animal {
    boolean hasFur;

    public Mammal(String name, boolean hasFur) {
        super(name);
        this.hasFur = hasFur;
    }

    public void displayMammalInfo() {
        System.out.println("Mammal Name: " + name + " | Has Fur: " + hasFur);
    }
}

class Dog extends Mammal {
    String breed;

    public Dog(String name, boolean hasFur, String breed) {
        super(name, hasFur);
        this.breed = breed;
    }

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

### Explanation

1. **Base Class `Animal`**:
   - Defines state `name` and behavior `makeSound()`.

2. **Constructor Chaining (`Mammal` and `Dog`)**:
   - `Mammal` constructor calls `super(name);`, passing `name` up to `Animal`.
   - `Dog` constructor calls `super(name, hasFur);`, passing both arguments up to `Mammal`.

3. **Multilevel Inheritance & Overriding**:
   - `Dog` object `myDog` inherits `displayMammalInfo()` from `Mammal` and `name` from `Animal`.
   - Calling `myDog.makeSound()` executes the overridden barking version defined inside `Dog`, overriding the generic `Animal.makeSound()`.

---

## 2. Real-World Program: Employee Payroll System

### `EmployeePayrollSystem.java`

```java
import java.util.Scanner;

/**
 * Enterprise payroll system modeling FullTimeEmployee and ContractEmployee subclasses under a shared Employee base.
 *
 * Example:
 *   Full-Time: Base $5000, Bonus $1000 -> Total: $6000.00
 *   Contract: $50/hr * 120 hrs -> Total: $6000.00
 */

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
        return 0.0;
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

        ftEmp.printPayStub();
        ctEmp.printPayStub();

        scanner.close();
    }
}
```

### Explanation

1. **Protected Fields (`protected int empId`)**:
   - `protected` modifier allows child classes (`FullTimeEmployee`, `ContractEmployee`) to access `empId`, `name`, and `department` directly while blocking access from unrelated external classes.

2. **Polymorphic Method Overriding**:
   - Base `Employee` class provides a placeholder `calculatePay()` returning `0.0`.
   - `FullTimeEmployee` overrides `calculatePay()` to compute `monthlySalary + performanceBonus`.
   - `ContractEmployee` overrides `calculatePay()` to compute `hourlyRate * hoursWorked`.
   - `printPayStub()` lives in the parent class `Employee`, but calling `ftEmp.printPayStub()` invokes `FullTimeEmployee.calculatePay()` dynamically.
