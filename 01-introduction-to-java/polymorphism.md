# Unit 1: Introduction to Java

## Polymorphism

**Polymorphism** comes from Greek words meaning *"many forms"*. In Java, it allows objects belonging to different classes to be treated through a single common superclass type, executing specialized behaviors dynamically depending on the actual underlying object.

---

### Core Concepts

#### 1. Compile-Time Polymorphism (Method Overloading)
- Multiple methods in the **same class** share the **same method name**, but have **different parameter lists** (differing in argument count, type, or order).
- **Static Binding**: The compiler selects which method signature to execute at compile time based on the arguments passed into the call.

#### 2. Runtime Polymorphism (Method Overriding)
- A subclass defines a method with the exact same signature as a method in its superclass.
- **Dynamic Binding / Dynamic Method Dispatch**: The JVM decides which method body to execute **at runtime** based on the actual object created on the heap, rather than the type of the reference variable pointing to it.

#### 3. Upcasting
- Assigning a subclass object reference to a superclass reference variable:
  ```java
  Shape s = new Circle(); // Upcasting
  ```
- Allows writing generic methods that process any subclass instance without needing `if-else` type checks.

---

### Common Pitfalls

1. **Trying to call subclass-specific methods through a superclass reference**:
   - If `Circle` has a method `getRadius()`, writing `Shape s = new Circle(); s.getRadius();` fails to compile because `Shape` does not declare `getRadius()`. You must downcast: `((Circle) s).getRadius()`.
2. **Mistakenly altering method parameters when attempting to override**:
   - If parent class has `void draw(int quality)` and child class writes `void draw()`, Java treats this as **overloading**, not overriding. Use `@Override` to ensure the compiler flags mismatching signatures.

---

## 1. Demo: Overloading vs. Overriding

### `PolymorphismBasicsDemo.java`

```java
/**
 * Demonstrates compile-time method overloading and runtime dynamic method dispatch.
 */

// Method Overloading
class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }

    public double add(double a, double b) {
        return a + b;
    }
}

// Method Overriding & Dynamic Dispatch
class Shape {
    public void draw() {
        System.out.println("Drawing a generic shape.");
    }
}

class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Circle.");
    }
}

class Rectangle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Rectangle.");
    }
}

public class PolymorphismBasicsDemo {
    public static void main(String[] args) {
        
        System.out.println("=== 1. Compile-Time Polymorphism (Overloading) ===");
        Calculator calc = new Calculator();
        System.out.println("add(5, 10)       : " + calc.add(5, 10));
        System.out.println("add(5, 10, 15)   : " + calc.add(5, 10, 15));
        System.out.println("add(2.5, 3.7)    : " + calc.add(2.5, 3.7));

        System.out.println("\n=== 2. Runtime Polymorphism (Dynamic Dispatch) ===");
        Shape shape1 = new Circle();    // Upcasting
        Shape shape2 = new Rectangle(); // Upcasting

        shape1.draw(); // Calls Circle.draw()
        shape2.draw(); // Calls Rectangle.draw()
    }
}
```

### Detailed Code Walkthrough

1. **Compile-Time Resolution (`Calculator`)**:
   - `calc.add(5, 10)` matches `int add(int, int)`.
   - `calc.add(5, 10, 15)` matches `int add(int, int, int)`.
   - `calc.add(2.5, 3.7)` matches `double add(double, double)`.
   - All resolved at compile time.

2. **Runtime Resolution (`Shape shape1 = new Circle()`)**:
   - `shape1` is declared as type `Shape`, but points to a `Circle` object in heap memory.
   - When `shape1.draw()` is called, the JVM checks the actual object instance (`Circle`), executing `Circle.draw()` instead of `Shape.draw()`.

---

## 2. Real-World Program: Payment Gateway

### `PaymentProcessingGateway.java`

```java
import java.util.Scanner;

/**
 * Payment processing engine accepting CreditCardPayment, PayPalPayment, or CryptoPayment objects interchangeably.
 *
 * Example:
 *   Select Credit Card ($249.99)
 *   Output: Authenticating Credit Card ****-1234. Charged $249.99.
 */

class PaymentMethod {
    protected String transactionId;

    public PaymentMethod(String transactionId) {
        this.transactionId = transactionId;
    }

    public void processPayment(double amount) {
        System.out.println("Processing generic payment of $" + amount);
    }
}

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

class PayPalPayment extends PaymentMethod {
    private String email;

    public PayPalPayment(String transactionId, String email) {
        super(transactionId);
        this.email = email;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("REDIRECTING: PayPal API for user " + email);
        System.out.printf("SUCCESS: Charged $%.2f via PayPal. [TXN: %s]%n", amount, transactionId);
    }
}

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
    
    public static void executeCheckout(PaymentMethod payment, double orderTotal) {
        System.out.println("\n------------------------------------------");
        System.out.println("       INITIATING PAYMENT GATEWAY         ");
        System.out.println("------------------------------------------");
        payment.processPayment(orderTotal);
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

        executeCheckout(selectedPayment, orderTotal);
        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **Polymorphic Method Parameter (`executeCheckout(PaymentMethod payment, ...)` )**:
   - `executeCheckout` does not care whether `payment` is a `CreditCardPayment`, `PayPalPayment`, or `CryptoPayment`. It simply accepts any subclass of `PaymentMethod`.

2. **Decoupled Execution**:
   - Calling `payment.processPayment(orderTotal)` runs whichever payment implementation was chosen by the user at runtime. This avoids bloated `if-else` checks like `if (payment instanceof CreditCardPayment) ...`.
