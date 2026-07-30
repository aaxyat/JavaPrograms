# Unit 2: Applet Programming

## Passing Parameters to Applets

Passing parameters from HTML allows web developers to configure and customize an Applet's behavior, text, colors, and layout directly from an HTML document without modifying or recompiling the underlying Java source code.

---

### Core Concepts

#### 1. The HTML `<param>` Tag
Parameters are defined using `<param>` elements placed inside the opening and closing `<applet>` tags:

```html
<applet code="MyParamApplet.class" width="400" height="200">
    <param name="username" value="JohnDoe">
    <param name="score" value="95">
    <param name="isPremium" value="true">
</applet>
```

#### 2. Reading Parameters in Java (`getParameter()`)
- Inside the Applet's `init()` method, call `getParameter("paramName")` to retrieve the corresponding string value:
  ```java
  String user = getParameter("username");
  ```
- **Return Type**: `getParameter()` **always returns a `String`** (or `null` if the parameter name does not exist in the HTML file).

#### 3. Converting Data Types
Because `getParameter()` returns `String`, you must convert numeric and boolean parameter values explicitly:
- **Integers**: `int score = Integer.parseInt(getParameter("score"));`
- **Doubles**: `double rate = Double.parseDouble(getParameter("rate"));`
- **Booleans**: `boolean flag = Boolean.parseBoolean(getParameter("isPremium"));`
- **Colors**: `Color col = Color.decode(getParameter("colorHex"));`

#### 4. Documenting Parameters (`getParameterInfo()`)
Overriding `getParameterInfo()` exposes documentation about expected parameters to applet container tools:

```java
@Override
public String[][] getParameterInfo() {
    return new String[][] {
        { "username", "String", "The display name of the user" },
        { "score", "int", "Initial player starting score" }
    };
}
```

---

### Common Pitfalls

1. **Forgetting Defensive Null Checks**:
   - If an HTML author omits a parameter tag, `getParameter()` returns `null`. Passing `null` to `Integer.parseInt(null)` throws a `NumberFormatException`. Always use fallback default values:
     ```java
     String input = getParameter("score");
     int score = (input != null) ? Integer.parseInt(input) : 0;
     ```
2. **Case Sensitivity Mismatches**:
   - `getParameter("userName")` will fail to match `<param name="username" ...>` if letter casing differs. Always match casing exactly between HTML and Java code.
3. **Reading Parameters Inside Constructor**:
   - Calling `getParameter()` inside the class constructor throws a `NullPointerException` because the Applet environment stub is attached only when entering `init()`.

---

## 1. Demo: Multi-Type Parameter Retrieval

### `AppletParameterDemo.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Font;

/*
<applet code="AppletParameterDemo.class" width="450" height="220">
    <param name="playerName" value="Alex_Hunter">
    <param name="playerLevel" value="12">
    <param name="soundEnabled" value="true">
</applet>
*/
public class AppletParameterDemo extends Applet {

    private String playerName;
    private int playerLevel;
    private boolean soundEnabled;

    @Override
    public void init() {
        setBackground(Color.LIGHT_GRAY);

        // 1. Read String Parameter with Null Guard
        playerName = getParameter("playerName");
        if (playerName == null) {
            playerName = "Guest Player";
        }

        // 2. Read and Parse Integer Parameter with Error Guard
        String levelStr = getParameter("playerLevel");
        try {
            playerLevel = (levelStr != null) ? Integer.parseInt(levelStr) : 1;
        } catch (NumberFormatException e) {
            playerLevel = 1; // Default fallback level
        }

        // 3. Read and Parse Boolean Parameter
        String soundStr = getParameter("soundEnabled");
        soundEnabled = (soundStr != null) && Boolean.parseBoolean(soundStr);
    }

    @Override
    public void paint(Graphics g) {
        g.setFont(new Font("SansSerif", Font.BOLD, 16));
        g.setColor(Color.BLUE);
        g.drawString("=== PLAYER CONFIGURATION DATA ===", 30, 40);

        g.setFont(new Font("SansSerif", Font.PLAIN, 14));
        g.setColor(Color.BLACK);
        g.drawString("Player Name   : " + playerName, 30, 80);
        g.drawString("Current Level : " + playerLevel, 30, 110);
        g.drawString("Audio Status  : " + (soundEnabled ? "Enabled" : "Disabled"), 30, 140);
    }

    @Override
    public String[][] getParameterInfo() {
        return new String[][] {
            { "playerName", "String", "Name of the active game profile" },
            { "playerLevel", "int", "Current unlocked player level" },
            { "soundEnabled", "boolean", "Toggle audio sound effects" }
        };
    }
}
```

### Explanation

1. **Defensive Parsing Strategy**:
   - `playerName = getParameter("playerName"); if (playerName == null) ...`: Protects against missing parameter tags by supplying `"Guest Player"`.
   - `try { Integer.parseInt(levelStr); } catch (...)`: Prevents invalid string inputs (such as `"twelve"`) from crashing the Applet.

2. **Boolean Conversion**:
   - `Boolean.parseBoolean(soundStr)` parses `"true"` (case-insensitive) to `true`, and any other value to `false`.

---

## 2. Practical Program: Dynamic Web Announcement Banner

### `DynamicBannerApplet.java`

```java
import java.applet.Applet;
import java.awt.Graphics;
import java.awt.Color;
import java.awt.Font;

/*
<applet code="DynamicBannerApplet.class" width="550" height="180">
    <param name="message" value="SPECIAL NOTICE: Midterm Exam Scheduled for Friday!">
    <param name="bgColorHex" value="#1E293B">
    <param name="textColorHex" value="#F8FAFC">
    <param name="fontSize" value="18">
    <param name="showBorder" value="true">
</applet>
*/
public class DynamicBannerApplet extends Applet {

    private String bannerMessage;
    private Color backgroundColor;
    private Color textColor;
    private int fontSize;
    private boolean showBorder;

    @Override
    public void init() {
        // Read Banner Message
        bannerMessage = getParameter("message");
        if (bannerMessage == null) {
            bannerMessage = "Welcome to Our Academic Portal";
        }

        // Read and Parse Background Hex Color (#RRGGBB)
        String bgHex = getParameter("bgColorHex");
        try {
            backgroundColor = (bgHex != null) ? Color.decode(bgHex) : Color.WHITE;
        } catch (NumberFormatException e) {
            backgroundColor = Color.WHITE;
        }
        setBackground(backgroundColor);

        // Read and Parse Text Hex Color
        String textHex = getParameter("textColorHex");
        try {
            textColor = (textHex != null) ? Color.decode(textHex) : Color.BLACK;
        } catch (NumberFormatException e) {
            textColor = Color.BLACK;
        }

        // Read and Parse Font Size
        String fontStr = getParameter("fontSize");
        try {
            fontSize = (fontStr != null) ? Integer.parseInt(fontStr) : 16;
        } catch (NumberFormatException e) {
            fontSize = 16;
        }

        // Read Border Flag
        String borderStr = getParameter("showBorder");
        showBorder = (borderStr != null) && Boolean.parseBoolean(borderStr);
    }

    @Override
    public void paint(Graphics g) {
        // Draw Optional Outer Border Box
        if (showBorder) {
            g.setColor(textColor);
            g.drawRect(5, 5, getWidth() - 10, getHeight() - 10);
            g.drawRect(7, 7, getWidth() - 14, getHeight() - 14);
        }

        // Render Customized Announcement Message
        g.setColor(textColor);
        g.setFont(new Font("SansSerif", Font.BOLD, fontSize));
        g.drawString(bannerMessage, 25, getHeight() / 2 + 5);
    }
}
```

### Explanation

1. **Hex Color Decoding (`Color.decode(hex)`)**:
   - `Color.decode("#1E293B")` parses HTML-style hexadecimal color strings into Java `Color` objects.

2. **Dynamic Centering & Layout**:
   - `g.drawString(bannerMessage, 25, getHeight() / 2 + 5)` uses `getHeight()` to center the announcement text vertically within whatever canvas dimensions are set in HTML.
