# Unit 1: Introduction to Java

## Subheading: Interface and Packages

An **Interface** defines a contractual blueprint of abstract methods that implementing classes must fulfill. A **Package** organizes related classes and interfaces into distinct namespaces, preventing naming collisions and controlling access visibility across modules.

### Key Concepts
1. **Interfaces (`interface` / `implements`)**:
   - Contains method declarations without implementation (implicitly `public abstract`).
   - Classes can implement **multiple interfaces**, enabling multiple inheritance of behavior.
   - Modern Java interfaces support `default` (concrete fallback methods) and `static` utility methods.
2. **Packages (`package` / `import`)**:
   - `package com.edu.services;`: Specifies the folder hierarchy and namespace for a class file.
   - `import java.util.List;`: Brings external classes into scope.
3. **Access Modifiers across Packages**:
   - `public`: Accessible from any package.
   - `protected`: Accessible within the same package and by subclasses in other packages.
   - *Default (Package-Private)*: Accessible only within the same package.
   - `private`: Accessible only within the declaring class.

---

## 1. Demo Program: Interfaces & Package Structure

**Filename:** `InterfaceAndPackageDemo.java`

### Source Code
```java
/**
 * Program: InterfaceAndPackageDemo.java
 * Teaches: Interface definition, multiple interface implementation, default methods, static methods, and package statements.
 * Usage: Demonstrates interface implementation contracts and multiple inheritance.
 */

// Package declaration (organizes class into namespace)
// package com.education.demo;

// 1. First Interface (Contract for Resizable entities)
interface Resizable {
    void resize(double percentage); // Abstract method
}

// 2. Second Interface (Contract for Printable entities)
interface Printable {
    void printDetails(); // Abstract method

    // Default method (Java 8+)
    default void printWatermark() {
        System.out.println("[WATERMARK: CONFIDENTIAL DOCUMENT]");
    }

    // Static helper method inside interface
    static void printSystemHeader() {
        System.out.println("==========================================");
        System.out.println("         GRAPHICS ENGINE v2.0             ");
        System.out.println("==========================================");
    }
}

// 3. Class implementing Multiple Interfaces
class DocumentImage implements Resizable, Printable {
    private String filename;
    private double width;
    private double height;

    public DocumentImage(String filename, double width, double height) {
        this.filename = filename;
        this.width = width;
        this.height = height;
    }

    // Fulfilling Resizable contract
    @Override
    public void resize(double percentage) {
        this.width *= (1 + percentage / 100);
        this.height *= (1 + percentage / 100);
        System.out.println("Resized '" + filename + "' by " + percentage + "% -> New Dimensions: " + width + "x" + height);
    }

    // Fulfilling Printable contract
    @Override
    public void printDetails() {
        System.out.println("Image File: " + filename + " | Dimensions: " + width + " x " + height + " px");
    }
}

public class InterfaceAndPackageDemo {
    public static void main(String[] args) {
        
        // Calling Static Method from Interface directly
        Printable.printSystemHeader();

        // Instantiating Class Implementing Multiple Interfaces
        DocumentImage doc = new DocumentImage("Chart.png", 800.0, 600.0);

        doc.printDetails();
        doc.printWatermark(); // Invokes default method

        System.out.println("\n--- Triggering Resizable Interface Contract ---");
        doc.resize(25.0); // Resizes image dimensions by 25%
    }
}
```

### Code Explanation
1. **Multiple Interface Implementation (`implements Resizable, Printable`)**:
   - `DocumentImage` satisfies the contracts of both interfaces by implementing `resize()` and `printDetails()`.
2. **Default & Static Interface Features**:
   - `default void printWatermark()` provides a ready-to-use implementation inherited automatically.
   - `Printable.printSystemHeader()` runs directly from the interface namespace without requiring an object instance.

---

## 2. Real-World Program: Multi-Channel Cloud Notification Dispatcher

**Filename:** `CloudNotificationService.java`

### Source Code
```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * Program: CloudNotificationService.java
 * Teaches: Decoupling system components using interface contracts for multi-channel messaging (Email, SMS, Push).
 * Example Input/Output:
 *   Message: "Your order #9948 has shipped!"
 *   Output: Dispatches message through EmailNotifier, SmsNotifier, and PushNotifier simultaneously.
 */

// Interface Contract for Notification Providers
interface NotificationChannel {
    boolean sendNotification(String recipient, String message);
    String getChannelName();
}

// Channel 1: Email Notification Provider
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

// Channel 2: SMS Notification Provider
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

// Channel 3: Mobile Push Notification Provider
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

        // Registering Active Channels via Interface List
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

        // Broadcast to all channels using Interface Contract
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

### Code Explanation
1. **Interface Abstraction (`NotificationChannel`)**:
   - Standardizes the method signature `sendNotification(recipient, message)` so callers don't need to know individual delivery mechanics.
2. **Pluggable Architecture**:
   - New notification channels (e.g. WhatsApp, Slack) can be added simply by implementing `NotificationChannel` without changing existing dispatcher code.
