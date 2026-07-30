# Unit 2: Applet Programming

## Comparison Between Applet and Application

Java supports two primary program execution models: standalone **Applications** and web-embedded **Applets**. Understanding their structural and operational differences highlights how Java evolved from desktop software to secure browser-based components.

---

### Core Concepts

#### 1. Detailed Comparison Table

| Feature / Aspect | Java Application | Java Applet |
|------------------|------------------|-------------|
| **Entry Point** | Executed via `public static void main(String[] args)`. | Container-driven lifecycle (`init()`, `start()`, `stop()`, `destroy()`). No `main()` method required. |
| **Execution Host** | Runs standalone directly on the operating system JVM. | Runs embedded inside a web browser container or `appletviewer`. |
| **Installation** | Must be installed or stored locally on the client machine. | Downloaded dynamically over HTTP whenever a web page opens. |
| **Security Access** | Unrestricted access to local file system, memory, and devices. | Restricted inside a security **sandbox** (cannot read/write local files or launch local executables). |
| **Network Sockets** | Can open network socket connections to any IP address or host. | Can connect only back to the host domain from which it was downloaded. |
| **Termination** | Calling `System.exit(0)` terminates the JVM process. | Destroyed by the browser container (`destroy()`). `System.exit()` is forbidden. |
| **UI Framework** | Console output (`System.out`) or Swing/AWT desktop frames (`JFrame`). | Extends `java.applet.Applet` or `javax.swing.JApplet` canvas embedded in HTML. |

---

### Common Pitfalls

1. **Calling `System.exit(0)` inside an Applet**:
   - Calling `System.exit(0)` inside an applet attempts to shut down the entire web browser host process, which triggers a `SecurityException` inside the browser sandbox.
2. **Attempting Local File I/O in an Applet**:
   - Standard applets cannot read or write files on the client disk (`new FileOutputStream(...)`). Doing so throws a `SecurityException` unless the applet is digitally signed.
3. **Confusing Window Framing**:
   - Standard applications must create a window container (`Frame` or `JFrame`) explicitly and call `setVisible(true)`. Applets are rendered inside a pre-allocated HTML canvas area provided by the browser.

---

## 1. Demo: Dual-Mode Class (Application & Applet Hybrid)

### `AppletVsApplicationDemo.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Frame;
import java.awt.Color;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

/*
<applet code="AppletVsApplicationDemo.class" width="450" height="200">
</applet>
*/
public class AppletVsApplicationDemo extends Applet {

    private String executionMode = "Applet Mode (Browser / AppletViewer)";

    // Applet Initialization
    @Override
    public void init() {
        setBackground(Color.LIGHT_GRAY);
    }

    // Applet Canvas Rendering
    @Override
    public void paint(Graphics g) {
        g.setColor(Color.BLUE);
        g.drawString("Execution Environment:", 30, 60);
        g.setColor(Color.BLACK);
        g.drawString(executionMode, 30, 90);
        g.drawString("Has main() method? Yes (Hybrid Class)", 30, 120);
    }

    // Standalone Application Entry Point
    public static void main(String[] args) {
        System.out.println("[APPLICATION MODE] Launching standalone window...");

        // Create desktop frame container
        Frame frame = new Frame("Standalone Application Host");
        AppletVsApplicationDemo appInstance = new AppletVsApplicationDemo();
        
        appInstance.executionMode = "Standalone Application Mode (Desktop JVM)";

        // Initialize and add applet component into desktop frame
        appInstance.init();
        appInstance.start();
        frame.add(appInstance);

        frame.setSize(450, 200);
        frame.setVisible(true);

        // Handle desktop window closing
        frame.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                System.out.println("[APPLICATION MODE] Closing desktop window...");
                appInstance.stop();
                appInstance.destroy();
                System.exit(0);
            }
        });
    }
}
```

### Explanation

1. **Hybrid Class Design**:
   - `AppletVsApplicationDemo` extends `Applet` so the browser or `appletviewer` can launch it via `init()` and `paint()`.
   - It also defines `public static void main(String[] args)`, allowing it to run as a standalone desktop application via `java AppletVsApplicationDemo`.

2. **Standalone Execution Flow (`main`)**:
   - Instantiates a desktop window (`Frame`), embeds the Applet instance inside it, and invokes `init()` and `start()` manually.
   - Attaches a `WindowAdapter` listener to catch window close events and invoke `System.exit(0)`.

---

## 2. Practical Program: System Environment Permission Auditor

### `SystemEnvironmentAuditor.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.io.File;

/*
<applet code="SystemEnvironmentAuditor.class" width="500" height="220">
</applet>
*/
public class SystemEnvironmentAuditor extends Applet {

    private String userHomeStatus;
    private String fileAccessStatus;

    @Override
    public void init() {
        setBackground(Color.WHITE);

        // Test 1: Accessing System Property
        try {
            String userHome = System.getProperty("user.home");
            userHomeStatus = "ALLOWED: " + userHome;
        } catch (SecurityException e) {
            userHomeStatus = "BLOCKED by Sandbox SecurityManager";
        }

        // Test 2: Local File Access
        try {
            File testFile = new File("C:/test.txt");
            boolean exists = testFile.exists();
            fileAccessStatus = "ALLOWED: Checked file existence (" + exists + ")";
        } catch (SecurityException e) {
            fileAccessStatus = "BLOCKED by Sandbox SecurityManager";
        }
    }

    @Override
    public void paint(Graphics g) {
        g.setColor(Color.DARK_GRAY);
        g.drawRect(10, 10, getWidth() - 20, getHeight() - 20);

        g.setColor(Color.BLUE);
        g.drawString("=== SECURITY PERMISSION AUDIT REPORT ===", 25, 40);

        g.setColor(Color.BLACK);
        g.drawString("System Property (user.home):", 25, 80);
        g.setColor(userHomeStatus.startsWith("ALLOWED") ? new Color(0, 120, 0) : Color.RED);
        g.drawString("  -> " + userHomeStatus, 25, 100);

        g.setColor(Color.BLACK);
        g.drawString("Local Disk File Access:", 25, 140);
        g.setColor(fileAccessStatus.startsWith("ALLOWED") ? new Color(0, 120, 0) : Color.RED);
        g.drawString("  -> " + fileAccessStatus, 25, 160);
    }
}
```

### Explanation

1. **Security Exception Trapping**:
   - Encloses sensitive calls (`System.getProperty("user.home")` and `new File(...)`) inside `try-catch (SecurityException e)` blocks.

2. **Sandbox Behavior Verification**:
   - When executed inside an applet sandbox container, the browser's `SecurityManager` blocks local disk access, setting status strings to `"BLOCKED"`.
