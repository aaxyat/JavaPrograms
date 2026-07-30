# Unit 1: Introduction to Java

## Subheading: Polymorphism

**Polymorphism** (meaning "many forms") allows objects of different classes to be treated as objects of a common superclass, or methods to perform different operations based on parameters or runtime instances.

### Key Concepts
1. **Compile-Time Polymorphism (Static Binding / Method Overloading)**:
   - Multiple methods in the same class share the same name but differ in parameter types, number of parameters, or parameter sequence.
   - Resolved at compile time.
2. **Runtime Polymorphism (Dynamic Binding / Method Overriding)**:
   - A subclass overrides a method of its superclass.
   - The method call is resolved dynamically at runtime based on the actual object type, not the reference type.
3. **Dynamic Method Dispatch (Upcasting)**:
   - Superclass reference variable points to a subclass instance (e.g., `PaymentMethod p = new CreditCardPayment();`).
   - Enables writing flexible code that can process any derived class seamlessly.

---

## 1. Demo Program: Compile-Time & Runtime Polymorphism

**Filename:** `PolymorphismBasicsDemo.java`

### Source Code
```java
/**
 * Program: PolymorphismBasicsDemo.java
 * Teaches: Method Overloading (Compile-Time) and Method Overriding with Dynamic Method Dispatch (Runtime).
 * Usage: Demonstrates calculator method overloading and shape drawing dynamic dispatch.
 */

// 1. Compile-Time Polymorphism (Method Overloading)
class Calculator {
    // Add two integers
    public int add(int a, int b) {
        return a + b;
    }

    // Add three integers (Overloaded by parameter count)
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // Add two doubles (Overloaded by parameter type)
    public double add(double a, double b) {
        return a + b;
    }
}

// 2. Runtime Polymorphism (Method Overriding & Dynamic Dispatch)
class Shape {
    public void draw() {
        System.out.println("Drawing a generic shape.");
    }
}

class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Circle with radius r.");
    }
}

class Rectangle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Rectangle with width and height.");
    }
}

public class PolymorphismBasicsDemo {
    public static void main(String[] args) {
        
        // Compile-Time Polymorphism (Method Overloading)
        System.out.println("=== 1. Compile-Time Polymorphism (Overloading) ===");
        Calculator calc = new Calculator();
        System.out.println("add(5, 10)       : " + calc.add(5, 10));
        System.out.println("add(5, 10, 15)   : " + calc.add(5, 10, 15));
        System.out.println("add(2.5, 3.7)    : " + calc.add(2.5, 3.7));

        // Runtime Polymorphism (Dynamic Method Dispatch)
        System.out.println("\n=== 2. Runtime Polymorphism (Dynamic Dispatch) ===");
        Shape shape1 = new Circle();    // Upcasting
        Shape shape2 = new Rectangle(); // Upcasting

        shape1.draw(); // Calls Circle's draw() at runtime
        shape2.draw(); // Calls Rectangle's draw() at runtime
    }
}
```

### Code Explanation
1. **Method Overloading (`Calculator`)**:
   - The compiler determines which `add` signature to execute based on arguments passed (`(int, int)`, `(int, int, int)`, or `(double, double)`).
2. **Dynamic Method Dispatch (`Shape shape1 = new Circle()`)**:
   - Even though `shape1` is declared as type `Shape`, the Java Virtual Machine (JVM) invokes `Circle.draw()` dynamically at runtime based on the actual instantiated object on the heap.

---

## 2. Real-World Program: E-Commerce Payment Gateway

**Filename:** `PaymentProcessingGateway.java`

### Source Code
```java
import java.util.Scanner;

/**
 * Program: PaymentProcessingGateway.java
 * Teaches: Building a polymorphic payment processing engine supporting multiple payment methods seamlessly.
 * Example Input/Output:
 *   Select Payment: Credit Card ($150.00)
 *   Output: Processing Credit Card Payment of $150.00 via Encrypted Gateway. Status: APPROVED
 */

// Base Payment Class
class PaymentMethod {
    protected String transactionId;

    public PaymentMethod(String transactionId) {
        this.transactionId = transactionId;
    }

    public void processPayment(double amount) {
        System.out.println("Processing generic payment of $" + amount);
    }
}

// Subclass 1: Credit Card Payment
class CreditCardPayment extends PaymentMethod {
    private String cardNumber;

    public CreditCardPayment(String transactionId, String cardNumber) {
        super(transactionId);
        this.cardNumber = cardNumber;
    }

    @Override
    public void processPayment(double amount) {
        String maskedCard = "****-****-****-" + cardNumber.substring(cardNumber.length() - 4);
        System.out.println("AUTHENTICATING: Credit Card " + maskedCard);
        System.out.printf("SUCCESS: Charged $%.2f to Credit Card. [TXN: %s]%n", amount, transactionId);
    }
}

// Subclass 2: PayPal Payment
class PayPalPayment extends PaymentMethod {
    private String email;

    public PayPalPayment(String transactionId, String email) {
        super(transactionId);
        this.email = email;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("REDIRECTING: PayPal API for user " + email);
        System.out.printf("SUCCESS: Charged $%.2f via PayPal Account. [TXN: %s]%n", amount, transactionId);
    }
}

// Subclass 3: Crypto Payment
class CryptoPayment extends PaymentMethod {
    private String walletAddress;

    public CryptoPayment(String transactionId, String walletAddress) {
        super(transactionId);
        this.walletAddress = walletAddress;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("VERIFYING BLOCKCHAIN: Wallet " + walletAddress.substring(0, 6) + "...");
        System.out.printf("SUCCESS: Transferred $%.2f equivalent Crypto. [TXN: %s]%n", amount, transactionId);
    }
}

public class PaymentProcessingGateway {
    
    // Polymorphic Method accepting any PaymentMethod subclass instance
    public static void executeCheckout(PaymentMethod payment, double orderTotal) {
        System.out.println("\n------------------------------------------");
        System.out.println("       INITIATING PAYMENT GATEWAY         ");
        System.out.println("------------------------------------------");
        payment.processPayment(orderTotal); // Polymorphic Dispatch
        System.out.println("------------------------------------------");
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("       E-COMMERCE CHECKOUT SYSTEM         ");
        System.out.println("==========================================");

        double orderTotal = 249.99;
        System.out.printf("Cart Order Total: $%.2f%n", orderTotal);

        System.out.println("\nSelect Payment Method:");
        System.out.println("1. Credit Card");
        System.out.println("2. PayPal");
        System.out.println("3. Crypto Wallet");
        System.out.print("Choice (1-3): ");

        int choice = scanner.nextInt();
        scanner.nextLine();

        PaymentMethod selectedPayment = null;
        String txnId = "TXN-" + System.currentTimeMillis();

        switch (choice) {
            case 1:
                System.out.print("Enter 16-digit Credit Card Number: ");
                String card = scanner.nextLine().trim();
                selectedPayment = new CreditCardPayment(txnId, card);
                break;
            case 2:
                System.out.print("Enter PayPal Account Email: ");
                String email = scanner.nextLine().trim();
                selectedPayment = new PayPalPayment(txnId, email);
                break;
            case 3:
                System.out.print("Enter Crypto Wallet Address: ");
                String wallet = scanner.nextLine().trim();
                selectedPayment = new CryptoPayment(txnId, wallet);
                break;
            default:
                System.out.println("Invalid selection.");
                scanner.close();
                return;
        }

        // Polymorphic Call Execution
        executeCheckout(selectedPayment, orderTotal);

        scanner.close();
    }
}
```

### Code Explanation
1. **Polymorphic Parameter (`executeCheckout(PaymentMethod payment, ...)` )**:
   - The checkout engine accepts a reference of type `PaymentMethod`, allowing it to process `CreditCardPayment`, `PayPalPayment`, or `CryptoPayment` interchangeably.
2. **Runtime Resolution**:
   - Calling `payment.processPayment(orderTotal)` executes the specific payment logic of whichever object was instantiated at runtime without needing ugly `if-else` type checks.
