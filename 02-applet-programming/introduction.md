# Unit 2: Applet Programming

## Introduction to Java Applets

Java Applets were small Java programs executed inside a web browser using a browser plugin. 

> **Important Historical Context**: Java Applets were deprecated in Java 9 and officially removed in Java 17. Modern web browsers no longer support NPAPI plugins, meaning Applets cannot run in modern browsers today. However, studying Applets is historically valuable because they pioneered client-side web interactivity, component lifecycles, sandboxed browser security models, and event-driven graphical interfaces that laid the foundation for modern web technologies like WebAssembly, JavaScript UI frameworks, and HTML5 Canvas.

---

### Core Concepts

#### 1. What Was a Java Applet?
- An **Applet** is a Java class that extends `java.applet.Applet` (or `javax.swing.JApplet`).
- Unlike standard Java desktop applications, an Applet does **not** use a `main()` method. Instead, its execution is controlled entirely by a web browser or the `appletviewer` tool through a specific lifecycle.

#### 2. The Applet Lifecycle
An Applet transitions through five execution states managed automatically by the browser container:

```
    [ Browser Loads Page ]
              |
              v
           init()     <--- Called once on initialization
              |
              v
           start()    <--- Called when page becomes visible
              |
              v
           paint()    <--- Called whenever display needs redraw
              |
              v
           stop()     <--- Called when navigating away from page
              |
              v
          destroy()   <--- Called once before unloading from memory
```

1. **`init()`**: Called once when the browser loads the Applet. Used to initialize variables, load images, or set up UI layout.
2. **`start()`**: Called immediately after `init()` and every time the user visits or returns to the web page.
3. **`paint(Graphics g)`**: Called whenever the Applet window needs to draw or refresh graphics on screen.
4. **`stop()`**: Called when the user leaves the web page or minimizes the browser tab. Pauses background animations or tasks.
5. **`destroy()`**: Called once when the browser closes or unloads the Applet from memory to clean up resources.

#### 3. Graphics & AWT Rendering
- Applets render visuals using the `Graphics` object (`java.awt.Graphics`).
- Key drawing methods:
  - `g.drawString("Text", x, y)`: Draws text at coordinate `(x, y)`.
  - `g.drawRect(x, y, width, height)` / `g.fillRect(...)`: Draws outlined or filled rectangles.
  - `g.setColor(Color.BLUE)`: Sets the current drawing color.

---

### Common Pitfalls

1. **Adding a `main()` method expecting it to run as an Applet**:
   - Browsers and `appletviewer` ignore `main()`. Applets require extending `Applet` and overriding lifecycle methods.
2. **Attempting to run Applets in modern web browsers**:
   - Modern browsers (Chrome, Firefox, Edge) blocked NPAPI plugins years ago. To view Applet execution today, compile using an older JDK (JDK 8) and launch via the command-line utility `appletviewer HelloWorldApplet.java`.
3. **Performing heavy tasks inside `paint()`**:
   - The `paint()` method is called repeatedly by the window manager whenever screen repainting occurs. Placing network calls or initialization code inside `paint()` causes lag and flickering. Move setup code to `init()`.

---

## 1. Demo: Hello World Applet

### `HelloWorldApplet.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Font;

/*
<applet code="HelloWorldApplet.class" width="400" height="200">
</applet>
*/
public class HelloWorldApplet extends Applet {

    @Override
    public void init() {
        // Set background color of the Applet window
        setBackground(Color.LIGHT_GRAY);
    }

    @Override
    public void paint(Graphics g) {
        // Set font style and color
        g.setFont(new Font("SansSerif", Font.BOLD, 18));
        g.setColor(Color.BLUE);

        // Render text on screen at pixel coordinates x=50, y=80
        g.drawString("Hello, Java Applet World!", 50, 80);

        // Draw a decorative rectangle
        g.setColor(Color.RED);
        g.drawRect(40, 50, 310, 60);
    }
}
```

### Explanation

1. **HTML Embedded Tag Comment**:
   - The comment `/* <applet code="..." width="..." height="..."></applet> */` allows the command-line tool `appletviewer` to run the file directly without creating a separate HTML file on disk.

2. **Extending `Applet`**:
   - `public class HelloWorldApplet extends Applet` inherits the browser container interface and AWT window behavior.

3. **`init()` Method**:
   - `setBackground(Color.LIGHT_GRAY)` runs once when the applet loads to fill the window canvas background.

4. **`paint(Graphics g)` Method**:
   - `g.setFont(...)` and `g.setColor(...)` configure the graphics brush context.
   - `g.drawString("Hello, Java Applet World!", 50, 80)` draws text starting 50 pixels from the left border and 80 pixels down from the top border.
   - `g.drawRect(40, 50, 310, 60)` draws a red rectangle outline around the message.

---

## 2. Practical Program: Applet Lifecycle Monitor

### `AppletLifecycleDemo.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;

/*
<applet code="AppletLifecycleDemo.class" width="500" height="300">
</applet>
*/
public class AppletLifecycleDemo extends Applet {

    private StringBuilder logBuffer;

    @Override
    public void init() {
        logBuffer = new StringBuilder();
        appendLog("1. init() called - Applet initialized");
        setBackground(Color.WHITE);
    }

    @Override
    public void start() {
        appendLog("2. start() called - Applet execution started");
    }

    @Override
    public void stop() {
        appendLog("3. stop() called - Applet paused");
    }

    @Override
    public void destroy() {
        // Called when browser unloads the applet
        System.out.println("4. destroy() called - Applet resources released");
    }

    private void appendLog(String message) {
        logBuffer.append(message).append("\n");
        System.out.println("[CONSOLE LOG] " + message);
        repaint(); // Request screen redraw
    }

    @Override
    public void paint(Graphics g) {
        g.setColor(Color.BLACK);
        g.drawString("=== APPLET LIFECYCLE MONITOR ===", 20, 30);

        String[] lines = logBuffer.toString().split("\n");
        int yPosition = 60;

        g.setColor(Color.DARK_GRAY);
        for (String line : lines) {
            g.drawString(line, 20, yPosition);
            yPosition += 25; // Move down 25 pixels for next line
        }
    }
}
```

### Explanation

1. **Tracking State Transitions**:
   - Overrides `init()`, `start()`, `stop()`, and `destroy()` to track how the browser container calls each lifecycle method during execution.

2. **Triggering Repaints (`repaint()`)**:
   - `appendLog()` calls `repaint()`, which tells the container to schedule a call to `paint(Graphics g)` to update the on-screen display.

3. **Dynamic Canvas Rendering**:
   - `paint(Graphics g)` splits `logBuffer` into individual text lines and loops through them, incrementing `yPosition` by 25 pixels per line to display a live event log on screen.
