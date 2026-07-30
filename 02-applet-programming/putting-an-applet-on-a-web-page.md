# Unit 2: Applet Programming

## Putting an Applet on a Web Page

To display an Applet on a web page, the web browser reads an HTML document containing tags that point to the compiled Java `.class` bytecode file.

---

### Core Concepts

#### 1. The Legacy `<applet>` Tag Attributes

The original standard syntax for embedding an Applet into HTML used the `<applet>` tag:

```html
<applet code="MyApplet.class" width="400" height="300">
    <param name="userName" value="Alice">
    Your browser does not support Java Applets.
</applet>
```

| Attribute | Required / Optional | Description |
|-----------|---------------------|-------------|
| `code` | **Required** | The compiled bytecode filename (e.g., `MyApplet.class`). Must match the class name case-sensitively. |
| `width` | **Required** | The display canvas width in pixels allocated on the web page layout. |
| `height` | **Required** | The display canvas height in pixels allocated on the web page layout. |
| `codebase` | Optional | Specifies a relative or absolute directory path where `.class` files reside if they are not in the same directory as the HTML file. |
| `archive` | Optional | Points to one or more `.jar` files containing pre-compiled classes and image/audio assets. |
| `align` | Optional | Controls alignment on the HTML page relative to surrounding text (`top`, `middle`, `bottom`, `left`, `right`). |

#### 2. Passing Parameters via `<param>` Tags
- `<param name="key" value="value">` tags are placed inside the opening and closing `<applet>` tags.
- Inside Java code, `getParameter("key")` reads the corresponding string value.

#### 3. Modern `<object>` and `<embed>` Alternative Syntax
When HTML 4.01 deprecated the `<applet>` tag in favor of standardized object embedding, developers transitioned to the `<object>` tag:

```html
<object type="application/x-java-applet" width="400" height="300">
    <param name="code" value="MyApplet.class">
    <param name="archive" value="applet-assets.jar">
</object>
```

---

### Common Pitfalls

1. **Specifying `.java` instead of `.class` in the `code` attribute**:
   - `code="MyApplet.java"` fails because browsers execute compiled `.class` bytecode, not source `.java` files. Always specify `code="MyApplet.class"`.
2. **Case sensitivity in file names**:
   - Java class names are case-sensitive. If the compiled class is `StudentApplet.class`, writing `code="studentapplet.class"` works on Windows test setups but fails when deployed to case-sensitive Linux web servers.
3. **Placing HTML and `.class` files in different folders without `codebase`**:
   - If `index.html` is in the root directory and `MyApplet.class` is in a `bin/` folder, the browser returns a `ClassNotFoundException` unless you add `codebase="bin/"`.

---

## 1. Demo: Embedding an Applet in HTML

### `WebPageAppletDemo.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Font;

public class WebPageAppletDemo extends Applet {

    private String pageTitle;
    private String authorName;

    @Override
    public void init() {
        setBackground(Color.WHITE);

        // Read parameter values from HTML <param> tags
        pageTitle = getParameter("title");
        if (pageTitle == null) pageTitle = "Untitled Page";

        authorName = getParameter("author");
        if (authorName == null) authorName = "Anonymous";

        showStatus("Applet loaded from HTML page successfully.");
    }

    @Override
    public void paint(Graphics g) {
        g.setColor(Color.DARK_GRAY);
        g.drawRect(10, 10, getWidth() - 20, getHeight() - 20);

        g.setFont(new Font("Serif", Font.BOLD, 16));
        g.setColor(Color.BLUE);
        g.drawString("Page Title: " + pageTitle, 30, 50);

        g.setFont(new Font("SansSerif", Font.PLAIN, 14));
        g.setColor(Color.BLACK);
        g.drawString("Author    : " + authorName, 30, 80);
    }
}
```

### Accompanying HTML File: `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Java Applet Web Page Integration</title>
</head>
<body>
    <h1>Student Portal - Embedded Java Applet</h1>
    <p>Below is an embedded Java Applet rendered directly inside the browser canvas:</p>

    <!-- Applet Embedding Tag -->
    <applet code="WebPageAppletDemo.class" width="450" height="150" align="center">
        <param name="title" value="Java Web Integration Fundamentals">
        <param name="author" value="Professor Smith">
        
        <!-- Fallback message if browser does not support applets -->
        <p><strong>Warning:</strong> Your browser does not support embedded Java Applets.</p>
    </applet>

    <hr>
    <p>Page footer text continuing below the Applet canvas.</p>
</body>
</html>
```

### Explanation

1. **Java Class Compilation**:
   - `WebPageAppletDemo.java` is compiled using `javac WebPageAppletDemo.java` to generate `WebPageAppletDemo.class`.

2. **HTML Tag Binding**:
   - `<applet code="WebPageAppletDemo.class" width="450" height="150">` tells the browser plugin to allocate a 450x150 pixel canvas area and load `WebPageAppletDemo.class`.

3. **Parameter Passing**:
   - `<param name="title" value="...">` passes the string `"Java Web Integration Fundamentals"`.
   - `getParameter("title")` inside `init()` reads this value and assigns it to `pageTitle` before `paint()` draws it onto the web page canvas.

---

## 2. Practical Program: Multi-Asset Web Dashboard Widget

### `InteractiveWebWidgetApplet.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Font;

public class InteractiveWebWidgetApplet extends Applet {

    private String siteName;
    private int refreshInterval;
    private Color widgetColor;

    @Override
    public void init() {
        siteName = getParameter("siteName");
        if (siteName == null) siteName = "Default Portal";

        String intervalParam = getParameter("refreshInterval");
        try {
            refreshInterval = (intervalParam != null) ? Integer.parseInt(intervalParam) : 30;
        } catch (NumberFormatException e) {
            refreshInterval = 30;
        }

        String colorParam = getParameter("theme");
        if ("BLUE".equalsIgnoreCase(colorParam)) {
            widgetColor = new Color(40, 90, 160);
        } else if ("GREEN".equalsIgnoreCase(colorParam)) {
            widgetColor = new Color(30, 130, 60);
        } else {
            widgetColor = Color.BLACK;
        }

        setBackground(new Color(240, 240, 240));
    }

    @Override
    public void paint(Graphics g) {
        // Draw Header Banner Box
        g.setColor(widgetColor);
        g.fillRect(15, 15, getWidth() - 30, 45);

        g.setColor(Color.WHITE);
        g.setFont(new Font("SansSerif", Font.BOLD, 16));
        g.drawString("WIDGET: " + siteName.toUpperCase(), 30, 43);

        // Draw Information Body
        g.setColor(Color.DARK_GRAY);
        g.setFont(new Font("SansSerif", Font.PLAIN, 13));
        g.drawString("Auto-Refresh Rate : Every " + refreshInterval + " seconds", 30, 90);
        g.drawString("Canvas Dimensions : " + getWidth() + " x " + getHeight() + " px", 30, 115);
    }
}
```

### Accompanying HTML File: `dashboard.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Enterprise Dashboard Widget Integration</title>
</head>
<body>
    <h2>Live Monitoring Dashboard</h2>

    <!-- Advanced Embedding with Codebase and Params -->
    <applet code="InteractiveWebWidgetApplet.class" 
            codebase="applets/" 
            width="400" 
            height="160">
        
        <param name="siteName" value="Financial Data Exchange">
        <param name="refreshInterval" value="15">
        <param name="theme" value="BLUE">

        <p>Java Applet support is disabled in this browser environment.</p>
    </applet>

</body>
</html>
```

### Explanation

1. **Directory Organization (`codebase="applets/"`)**:
   - `codebase="applets/"` instructs the browser that `InteractiveWebWidgetApplet.class` is located inside the `applets/` subfolder, keeping the HTML file in the root directory while organizing Java bytecode into a dedicated folder.

2. **Parameter Type Conversions**:
   - `Integer.parseInt(intervalParam)` converts the string `"15"` passed by `<param name="refreshInterval" value="15">` into an integer for logic operations.
   - Theme strings (`"BLUE"`, `"GREEN"`) map to RGB `Color` instances used during painting.
