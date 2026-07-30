# Unit 1: Introduction to Java

## Interface and Packages

**Interfaces** define behavioral contracts for classes, while **Packages** organize related classes into distinct namespaces to manage code structure and access permissions.

---

### Core Concepts

#### 1. What is an Interface?
- An **Interface** (`interface`) is a reference type in Java that specifies abstract methods an implementing class must fulfill.
- **Contract Model**: An interface specifies *what* a class must do, leaving *how* it does it up to the implementing class.
- **Multiple Inheritance**: While a Java class can extend only one superclass, it can implement multiple interfaces (`implements InterfaceA, InterfaceB`).
- **Modern Interface Features**:
  - **`default` methods**: Provides a default concrete method implementation inside an interface that implementing classes inherit automatically.
  - **`static` methods**: Helper methods defined inside an interface that can be invoked directly (`InterfaceName.methodName()`).

#### 2. What is a Package?
- A **Package** (`package`) groups related classes and interfaces into a directory hierarchy.
- **Namespace Collision Prevention**: Prevents naming conflicts. Two classes named `User` can exist in the same project as long as they live in different packages (`com.app.model.User` vs `com.app.service.User`).
- **`import` Statement**: Brings classes from other packages into scope (`import java.util.Scanner;`).

#### 3. Access Modifiers Across Packages

| Modifier | Same Class | Same Package | Subclass (Other Pkg) | World (Other Pkg) |
|----------|------------|--------------|----------------------|-------------------|
| `public` | Yes | Yes | Yes | Yes |
| `protected` | Yes | Yes | Yes | No |
| *Default* | Yes | Yes | No | No |
| `private` | Yes | No | No | No |

---

### Common Pitfalls

1. **Forgetting to implement all abstract methods in a class**:
   - If a class implements an interface, it must implement *every* non-default abstract method declared by that interface, or else be declared as an `abstract class`.
2. **Package directory structure mismatch**:
   - Declaring `package com.edu.models;` at the top of a file requires the file to be stored physically inside the `com/edu/models` folder on disk.

---

## 1. Demo: Interfaces and Package Syntax

### `InterfaceAndPackageDemo.java`

```java
/**
 * Demonstrates defining interfaces, implementing multiple interfaces, default methods, and static methods.
 */

interface Resizable {
    void resize(double percentage);
}

interface Printable {
    void printDetails();

    default void printWatermark() {
        System.out.println("[WATERMARK: CONFIDENTIAL DOCUMENT]");
    }

    static void printSystemHeader() {
        System.out.println("==========================================");
        System.out.println("         GRAPHICS ENGINE v2.0             ");
        System.out.println("==========================================");
    }
}

class DocumentImage implements Resizable, Printable {
    private String filename;
    private double width;
    private double height;

    public DocumentImage(String filename, double width, double height) {
        this.filename = filename;
        this.width = width;
        this.height = height;
    }

    @Override
    public void resize(double percentage) {
        this.width *= (1 + percentage / 100);
        this.height *= (1 + percentage / 100);
        System.out.println("Resized '" + filename + "' by " + percentage + "% -> New Dimensions: " + width + "x" + height);
    }

    @Override
    public void printDetails() {
        System.out.println("Image File: " + filename + " | Dimensions: " + width + " x " + height + " px");
    }
}

public class InterfaceAndPackageDemo {
    public static void main(String[] args) {
        
        Printable.printSystemHeader();

        DocumentImage doc = new DocumentImage("Chart.png", 800.0, 600.0);

        doc.printDetails();
        doc.printWatermark();

        System.out.println("\n--- Triggering Resizable Interface Contract ---");
        doc.resize(25.0);
    }
}
```

### Detailed Code Walkthrough

1. **Multiple Interface Implementation (`implements Resizable, Printable`)**:
   - `DocumentImage` promises to fulfill the contracts of both `Resizable` (by defining `resize()`) and `Printable` (by defining `printDetails()`).

2. **Default & Static Interface Features**:
   - `Printable.printSystemHeader()` runs directly from the interface namespace.
   - `doc.printWatermark()` runs the default implementation provided inside `Printable` without requiring `DocumentImage` to rewrite it.

---

## 2. Real-World Program: Multi-Channel Notification Service

### `CloudNotificationService.java`

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * Multi-channel notification dispatcher using the NotificationChannel interface.
 *
 * Example:
 *   Message: "Your order #9948 has shipped!"
 *   Output: Sends message through EmailNotifier, SmsNotifier, and PushNotifier implementations.
 */

interface NotificationChannel {
    boolean sendNotification(String recipient, String message);
    String getChannelName();
}

class EmailNotifier implements NotificationChannel {
    @Override
    public boolean sendNotification(String recipient, String message) {
        System.out.println(" [EMAIL SERVICE] -> Sending Email to <" + recipient + ">: \"" + message + "\"");
        return true;
    }

    @Override
    public String getChannelName() {
        return "SMTP Email Channel";
    }
}

class SmsNotifier implements NotificationChannel {
    @Override
    public boolean sendNotification(String recipient, String message) {
        System.out.println(" [SMS GATEWAY] -> Sending SMS to Phone (" + recipient + "): \"" + message + "\"");
        return true;
    }

    @Override
    public String getChannelName() {
        return "Twilio SMS Channel";
    }
}

class PushNotifier implements NotificationChannel {
    @Override
    public boolean sendNotification(String recipient, String message) {
        System.out.println(" [PUSH SERVICE] -> Pushing Alert to Device Token [" + recipient + "]: \"" + message + "\"");
        return true;
    }

    @Override
    public String getChannelName() {
        return "Firebase Push Channel";
    }
}

public class CloudNotificationService {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("==========================================");
        System.out.println("     ENTERPRISE NOTIFICATION DISPATCHER   ");
        System.out.println("==========================================");

        List<NotificationChannel> activeChannels = new ArrayList<>();
        activeChannels.add(new EmailNotifier());
        activeChannels.add(new SmsNotifier());
        activeChannels.add(new PushNotifier());

        System.out.print("Enter notification broadcast message: ");
        String alertMessage = scanner.nextLine().trim();

        System.out.print("Enter recipient contact (Email / Phone / Token): ");
        String recipient = scanner.nextLine().trim();

        System.out.println("\n------------------------------------------");
        System.out.println("     BROADCASTING ACROSS ALL CHANNELS     ");
        System.out.println("------------------------------------------");

        int successCount = 0;
        for (NotificationChannel channel : activeChannels) {
            System.out.println("Channel: " + channel.getChannelName());
            boolean status = channel.sendNotification(recipient, alertMessage);
            if (status) successCount++;
            System.out.println();
        }

        System.out.println("------------------------------------------");
        System.out.println("Broadcast Complete! Successfully sent to " + successCount + "/" + activeChannels.size() + " channels.");
        System.out.println("==========================================");

        scanner.close();
    }
}
```

### Detailed Code Walkthrough

1. **Decoupled Architecture**:
   - `CloudNotificationService` operates on `List<NotificationChannel>`, storing `EmailNotifier`, `SmsNotifier`, and `PushNotifier` objects together.
   - The broadcast loop invokes `channel.sendNotification(recipient, alertMessage)` on each object polymorphically, ensuring the main dispatcher does not need separate code branches for each messaging technology.
