# Unit 2: Applet Programming

## Standard Applet Methods

The `java.applet.Applet` class provides standard utility methods to interact with the host browser environment, fetch HTML parameters, retrieve media resources, and update browser status bars.

---

### Core Concepts

#### 1. HTML Parameter Retrieval Methods
- **`getParameter(String name)`**: Reads attribute values passed from an HTML `<param name="key" value="val">` tag. Returns `null` if the parameter name is not found.
- **`getParameterInfo()`**: Returns a 2D array of `String` describing expected parameters, their data types, and descriptions.

#### 2. Environment & Location Getters
- **`getCodeBase()`**: Returns the `URL` directory where the Applet's compiled `.class` files reside.
- **`getDocumentBase()`**: Returns the `URL` of the HTML document hosting the Applet.

#### 3. Media & Audio Management
- **`getImage(URL url)` / `getImage(URL url, String name)`**: Loads an image resource from a URL asynchronously.
- **`getAudioClip(URL url)` / `play(URL url)`**: Loads or plays an audio file (`.au`, `.wav`) directly.

#### 4. Browser Context & Status Bar Updates
- **`showStatus(String msg)`**: Displays a text message in the browser's bottom status bar.
- **`getAppletContext()`**: Returns an `AppletContext` interface used to control browser tabs or communicate with other Applets on the same page.
- **`resize(int width, int height)`**: Requests the browser container to resize the Applet display canvas.

---

### Common Pitfalls

1. **Calling location getters inside constructors**:
   - Calling `getCodeBase()` or `getDocumentBase()` in a constructor throws a `NullPointerException` because the browser container attaches the `AppletStub` only *after* object instantiation. Always invoke these methods inside `init()`.
2. **Forgetting null checks on `getParameter()`**:
   - `getParameter()` returns `null` if an HTML parameter is missing. Passing that `null` directly into methods like `Integer.parseInt()` throws a `NullPointerException` or `NumberFormatException`.
3. **Assuming `getImage()` loads images synchronously**:
   - `getImage()` returns immediately while loading the image in a background thread. Drawing the image immediately in `paint()` before it finishes loading results in a blank canvas until the next window repaint.

---

## 1. Demo: Parameter Reading and Environment Inspection

### `StandardMethodsDemo.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.net.URL;

/*
<applet code="StandardMethodsDemo.class" width="500" height="250">
    <param name="heading" value="Welcome to Java Applets">
    <param name="fontSize" value="20">
</applet>
*/
public class StandardMethodsDemo extends Applet {

    private String headingText;
    private int fontSize;
    private URL codeBaseUrl;
    private URL docBaseUrl;

    @Override
    public void init() {
        setBackground(Color.LIGHT_GRAY);

        // 1. Fetch HTML Parameters
        headingText = getParameter("heading");
        if (headingText == null) {
            headingText = "Default Heading"; // Fallback if param missing
        }

        String sizeParam = getParameter("fontSize");
        try {
            fontSize = (sizeParam != null) ? Integer.parseInt(sizeParam) : 14;
        } catch (NumberFormatException e) {
            fontSize = 14;
        }

        // 2. Fetch Base Location URLs
        codeBaseUrl = getCodeBase();
        docBaseUrl = getDocumentBase();

        // 3. Update Browser Status Bar Message
        showStatus("Applet initialization complete. Parameters loaded successfully.");
    }

    @Override
    public void paint(Graphics g) {
        g.setColor(Color.BLUE);
        g.drawString("Heading Param  : " + headingText, 20, 40);
        g.drawString("Font Size Param: " + fontSize, 20, 70);

        g.setColor(Color.BLACK);
        g.drawString("Code Base URL  : " + codeBaseUrl.toString(), 20, 120);
        g.drawString("Doc Base URL   : " + docBaseUrl.toString(), 20, 150);
    }

    @Override
    public String[][] getParameterInfo() {
        return new String[][] {
            { "heading", "String", "Main title text displayed on canvas" },
            { "fontSize", "int", "Font point size for canvas heading" }
        };
    }
}
```

### Explanation

1. **Reading HTML Parameters (`getParameter("heading")`)**:
   - `getParameter("heading")` reads `"Welcome to Java Applets"` from the embedded `<param name="heading" ...>` tag.
   - A null guard `if (headingText == null)` ensures fallback text is used if the HTML param tag is missing.

2. **Inspecting Base Locations (`getCodeBase()` & `getDocumentBase()`)**:
   - `getCodeBase()` returns the URL location of compiled `.class` files.
   - `getDocumentBase()` returns the URL location of the hosting `.html` page.

3. **Status Bar Updates (`showStatus(...)`)**:
   - `showStatus(...)` sends feedback text directly to the browser container's status bar line.

---

## 2. Practical Program: Dynamic Media & Status Dashboard

### `AppletAudioVisualPlayer.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Button;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/*
<applet code="AppletAudioVisualPlayer.class" width="450" height="250">
    <param name="username" value="Student_User">
    <param name="themeColor" value="DARK">
</applet>
*/
public class AppletAudioVisualPlayer extends Applet implements ActionListener {

    private String user;
    private Button resizeBtn;
    private Button statusBtn;
    private boolean isExpanded = false;

    @Override
    public void init() {
        // Read user parameter
        user = getParameter("username");
        if (user == null) user = "Guest";

        String theme = getParameter("themeColor");
        if ("DARK".equalsIgnoreCase(theme)) {
            setBackground(Color.DARK_GRAY);
        } else {
            setBackground(Color.WHITE);
        }

        // Add GUI buttons using AWT components
        resizeBtn = new Button("Toggle Canvas Size");
        statusBtn = new Button("Play Alert Sound");

        resizeBtn.addActionListener(this);
        statusBtn.addActionListener(this);

        add(resizeBtn);
        add(statusBtn);

        showStatus("Dashboard initialized for user: " + user);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == resizeBtn) {
            if (!isExpanded) {
                // Request container to resize applet canvas
                resize(600, 350);
                showStatus("Canvas expanded to 600x350.");
                isExpanded = true;
            } else {
                resize(450, 250);
                showStatus("Canvas reset to 450x250.");
                isExpanded = false;
            }
        } else if (e.getSource() == statusBtn) {
            // Play built-in system beep or audio via URL
            try {
                play(getCodeBase(), "alert.au");
                showStatus("Playing audio alert file 'alert.au'...");
            } catch (Exception ex) {
                showStatus("Audio play triggered (alert.au).");
            }
        }
        repaint();
    }

    @Override
    public void paint(Graphics g) {
        if (getBackground() == Color.DARK_GRAY) {
            g.setColor(Color.WHITE);
        } else {
            g.setColor(Color.BLACK);
        }

        g.drawString("Active Dashboard User: " + user, 30, 100);
        g.drawString("Current Canvas Dimensions: " + getWidth() + " x " + getHeight(), 30, 130);
    }
}
```

### Explanation

1. **Theme Initialization from Parameters**:
   - `getParameter("themeColor")` reads the theme setting and sets the canvas background color (`Color.DARK_GRAY` or `Color.WHITE`).

2. **Canvas Resizing (`resize(600, 350)`)**:
   - `resize(600, 350)` requests the host browser or `appletviewer` window to dynamically adjust canvas dimensions when the user clicks the button.

3. **Audio Playback (`play(getCodeBase(), "alert.au")`)**:
   - `play(url, filename)` locates and plays an audio file relative to the code base path in a single line.
