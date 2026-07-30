# Unit 3: GUI Programming

## Event Handling (Mouse-Driven, Keyboard-Driven, and Other)

Java GUI applications use the **Delegation Event Model** to process user interactions. When a user clicks a mouse button, types a key, or selects an item, an **Event Source** generates an **Event Object** and delegates it to registered **Event Listener** interfaces.

---

### Core Concepts

#### 1. Delegation Event Model Architecture

```
  ┌────────────────┐         Generates Event Object         ┌──────────────────┐
  │  Event Source  │ ─────────────────────────────────────> │   Event Object   │
  │ (JButton/Panel)│                                        │(MouseEvent/Key..)│
  └────────────────┘                                        └────────┬─────────┘
          │                                                          │
          │ Registers                                                │ Dispatches
          v                                                          v
  ┌────────────────────────────────────────────────────────────────────────────┐
  │                              Event Listener                                │
  │        (MouseListener / KeyListener / ActionListener / ItemListener)        │
  └────────────────────────────────────────────────────────────────────────────┘
```

#### 2. Event Types, Listener Interfaces, and Adapters

| Event Category | Event Class | Listener Interface | Adapter Class | Core Handler Methods |
|----------------|-------------|--------------------|---------------|----------------------|
| **Mouse Clicks** | `MouseEvent` | `MouseListener` | `MouseAdapter` | `mouseClicked()`, `mousePressed()`, `mouseReleased()`, `mouseEntered()`, `mouseExited()` |
| **Mouse Motion** | `MouseEvent` | `MouseMotionListener` | `MouseMotionAdapter` | `mouseDragged()`, `mouseMoved()` |
| **Keyboard Input** | `KeyEvent` | `KeyListener` | `KeyAdapter` | `keyPressed()`, `keyReleased()`, `keyTyped()` |
| **Action Triggers** | `ActionEvent` | `ActionListener` | N/A | `actionPerformed()` |
| **Item State Changes**| `ItemEvent` | `ItemListener` | N/A | `itemStateChanged()` |
| **Focus Shifts** | `FocusEvent` | `FocusListener` | `FocusAdapter` | `focusGained()`, `focusLost()` |
| **Window Events** | `WindowEvent` | `WindowListener` | `WindowAdapter` | `windowClosing()`, `windowOpened()`, `windowIconified()` |

#### 3. Adapter Classes
When implementing a listener interface with multiple methods (such as `MouseListener`), you must provide code for all 5 methods—even if you only need one. **Adapter Classes** provide empty default implementations so you override only the specific event handlers you need.

---

### Common Pitfalls

1. **Forgetting `setFocusable(true)` for Keyboard Listeners**:
   - `KeyListener` events dispatch only to the component that currently has keyboard focus. Calling `panel.addKeyListener(...)` fails to receive keystrokes unless you also call `panel.setFocusable(true)` and `panel.requestFocusInWindow()`.
2. **Performing Expensive Operations Inside Event Handlers**:
   - Event listener methods execute directly on Swing's single **Event Dispatch Thread (EDT)**. Running heavy file I/O or network queries inside an event handler freezes the entire GUI window until the operation completes.
3. **Using `keyTyped()` for Arrow or Function Keys**:
   - `keyTyped()` captures only printable Unicode characters (letters, numbers, symbols). Non-printable control keys (Arrow keys, F1-F12, Escape, Shift) trigger `keyPressed()` and `keyReleased()`, not `keyTyped()`.

---

## 1. Demo: Mouse, Keyboard, and Focus Event Tracking

### `EventHandlingDemo.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.GridLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;

/**
 * Demonstrates MouseAdapter, KeyAdapter, and FocusAdapter event tracking.
 */
public class EventHandlingDemo extends JFrame {

    private JLabel mouseStatusLabel;
    private JLabel keyStatusLabel;
    private JLabel focusStatusLabel;
    private JPanel interactiveCanvas;
    private JTextField inputField;

    public EventHandlingDemo() {
        setTitle("Delegation Event Model Demo");
        setSize(550, 380);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // 1. Interactive Mouse Canvas Panel
        interactiveCanvas = new JPanel();
        interactiveCanvas.setBackground(Color.WHITE);
        interactiveCanvas.setFocusable(true);

        JLabel canvasInstruction = new JLabel("Click or Move Mouse Inside This Area", JLabel.CENTER);
        canvasInstruction.setFont(new Font("SansSerif", Font.PLAIN, 14));
        interactiveCanvas.add(canvasInstruction);

        // 2. Status Output Panel
        JPanel statusPanel = new JPanel(new GridLayout(3, 1, 4, 4));
        statusPanel.setBackground(new Color(241, 245, 249));

        mouseStatusLabel = new JLabel(" Mouse Status : Waiting for mouse interaction...");
        keyStatusLabel = new JLabel(" Key Status   : Press any key inside input box below...");
        focusStatusLabel = new JLabel(" Focus Status : Click inside input box to focus...");

        statusPanel.add(mouseStatusLabel);
        statusPanel.add(keyStatusLabel);
        statusPanel.add(focusStatusLabel);

        // 3. Text Input Box for Keyboard & Focus Events
        inputField = new JTextField();
        inputField.setFont(new Font("SansSerif", Font.PLAIN, 14));

        // Attach Mouse Event Listeners using MouseAdapter
        interactiveCanvas.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                mouseStatusLabel.setText(String.format(" Mouse Clicked : Button %d at (%d, %d)", 
                        e.getButton(), e.getX(), e.getY()));
                interactiveCanvas.setBackground(new Color(224, 242, 254));
            }

            @Override
            public void mouseEntered(MouseEvent e) {
                mouseStatusLabel.setText(" Mouse Pointer : ENTERED canvas area");
            }

            @Override
            public void mouseExited(MouseEvent e) {
                mouseStatusLabel.setText(" Mouse Pointer : EXITED canvas area");
                interactiveCanvas.setBackground(Color.WHITE);
            }
        });

        interactiveCanvas.addMouseMotionListener(new MouseAdapter() {
            @Override
            public void mouseMoved(MouseEvent e) {
                mouseStatusLabel.setText(String.format(" Mouse Motion  : Position (%d, %d)", e.getX(), e.getY()));
            }
        });

        // Attach Keyboard Event Listener using KeyAdapter
        inputField.addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                String keyText = KeyEvent.getKeyText(e.getKeyCode());
                keyStatusLabel.setText(String.format(" Key Pressed   : Code = %d (%s)", e.getKeyCode(), keyText));
            }
        });

        // Attach Focus Event Listener using FocusAdapter
        inputField.addFocusListener(new FocusAdapter() {
            @Override
            public void focusGained(FocusEvent e) {
                focusStatusLabel.setText(" Focus Status : Input box GAINED focus");
                inputField.setBackground(new Color(254, 243, 199));
            }

            @Override
            public void focusLost(FocusEvent e) {
                focusStatusLabel.setText(" Focus Status : Input box LOST focus");
                inputField.setBackground(Color.WHITE);
            }
        });

        // Assembly
        setLayout(new BorderLayout(8, 8));
        add(interactiveCanvas, BorderLayout.CENTER);
        add(statusPanel, BorderLayout.NORTH);
        add(inputField, BorderLayout.SOUTH);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new EventHandlingDemo().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Using `MouseAdapter`**:
   - Overrides `mouseClicked()`, `mouseEntered()`, `mouseExited()`, and `mouseMoved()` without needing to supply empty implementations for `mousePressed()` or `mouseReleased()`.

2. **Using `KeyAdapter` (`keyPressed`)**:
   - `e.getKeyCode()` retrieves the physical key code (e.g. `VK_ENTER`, `VK_UP`). `KeyEvent.getKeyText(e.getKeyCode())` converts raw key codes to human-readable names.

3. **Using `FocusAdapter` (`focusGained` & `focusLost`)**:
   - Updates `focusStatusLabel` and input background color whenever `inputField` gains or loses keyboard cursor focus.

---

## 2. Practical Program: Interactive Sketchpad and Character Stamping Tool

### `InteractiveCanvasPainter.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JComboBox;
import javax.swing.JButton;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.Color;
import java.awt.Graphics;
import java.awt.Point;
import java.awt.Font;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.ItemListener;
import java.awt.event.ItemEvent;
import java.util.ArrayList;
import java.util.List;

/**
 * Interactive drawing canvas capturing mouse dragging for freehand drawing, mouse clicks for stamping text, and key presses.
 */
public class InteractiveCanvasPainter extends JFrame {

    private DrawingPanel canvasPanel;
    private JComboBox<String> colorCombo;
    private JButton clearButton;
    private JLabel infoLabel;
    private Color currentColor = Color.BLACK;

    public InteractiveCanvasPainter() {
        setTitle("Interactive Sketchpad & Character Stamper");
        setSize(700, 480);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Top Toolbar Controls
        JPanel toolbarPanel = new JPanel(new FlowLayout(FlowLayout.LEFT, 10, 5));
        toolbarPanel.setBackground(new Color(226, 232, 240));

        toolbarPanel.add(new JLabel("Brush Color:"));
        String[] colors = {"Black", "Red", "Blue", "Green"};
        colorCombo = new JComboBox<>(colors);
        toolbarPanel.add(colorCombo);

        clearButton = new JButton("Clear Canvas");
        toolbarPanel.add(clearButton);

        infoLabel = new JLabel("Click & Drag mouse to draw. Click canvas and type keys to stamp text.");
        toolbarPanel.add(infoLabel);

        // Center Drawing Canvas
        canvasPanel = new DrawingPanel();

        // Color ComboBox Item Listener
        colorCombo.addItemListener(new ItemListener() {
            @Override
            public void itemStateChanged(ItemEvent e) {
                if (e.getStateChange() == ItemEvent.SELECTED) {
                    String selected = (String) colorCombo.getSelectedItem();
                    switch (selected) {
                        case "Red": currentColor = Color.RED; break;
                        case "Blue": currentColor = Color.BLUE; break;
                        case "Green": currentColor = new Color(34, 197, 94); break;
                        default: currentColor = Color.BLACK; break;
                    }
                }
            }
        });

        // Clear Button Action Listener
        clearButton.addActionListener(e -> {
            canvasPanel.clearCanvas();
            canvasPanel.requestFocusInWindow();
        });

        // Assembly
        setLayout(new BorderLayout());
        add(toolbarPanel, BorderLayout.NORTH);
        add(canvasPanel, BorderLayout.CENTER);
    }

    // Custom Canvas Panel
    private class DrawingPanel extends JPanel {
        private final List<Point> points = new ArrayList<>();
        private final List<Color> pointColors = new ArrayList<>();
        private final List<StampedText> stampedTexts = new ArrayList<>();
        private Point currentCursorPoint = new Point(100, 100);

        public DrawingPanel() {
            setBackground(Color.WHITE);
            setFocusable(true);

            // Mouse Dragging & Click Handling
            MouseAdapter mouseHandler = new MouseAdapter() {
                @Override
                public void mousePressed(MouseEvent e) {
                    currentCursorPoint = e.getPoint();
                    addPoint(e.getPoint());
                    requestFocusInWindow(); // Claim keyboard focus on click
                }

                @Override
                public void mouseDragged(MouseEvent e) {
                    currentCursorPoint = e.getPoint();
                    addPoint(e.getPoint());
                }
            };

            addMouseListener(mouseHandler);
            addMouseMotionListener(mouseHandler);

            // Keyboard Typing Listener for Text Stamping
            addKeyListener(new KeyAdapter() {
                @Override
                public void keyTyped(KeyEvent e) {
                    char typedChar = e.getKeyChar();
                    if (typedChar != KeyEvent.CHAR_UNDEFINED && !Character.isISOControl(typedChar)) {
                        stampedTexts.add(new StampedText(String.valueOf(typedChar), new Point(currentCursorPoint), currentColor));
                        currentCursorPoint.x += 12; // Advance cursor right for next letter
                        repaint();
                    }
                }
            });
        }

        private void addPoint(Point p) {
            points.add(p);
            pointColors.add(currentColor);
            repaint();
        }

        public void clearCanvas() {
            points.clear();
            pointColors.clear();
            stampedTexts.clear();
            repaint();
        }

        @Override
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);

            // Render Freehand Mouse Points
            for (int i = 0; i < points.size(); i++) {
                g.setColor(pointColors.get(i));
                Point p = points.get(i);
                g.fillOval(p.x - 3, p.y - 3, 6, 6);
            }

            // Render Keyboard Stamped Text
            g.setFont(new Font("Monospaced", Font.BOLD, 18));
            for (StampedText st : stampedTexts) {
                g.setColor(st.color);
                g.drawString(st.text, st.position.x, st.position.y);
            }

            // Render Active Cursor Position Box
            g.setColor(Color.LIGHT_GRAY);
            g.drawRect(currentCursorPoint.x, currentCursorPoint.y - 12, 10, 15);
        }
    }

    private static class StampedText {
        String text;
        Point position;
        Color color;

        public StampedText(String text, Point position, Color color) {
            this.text = text;
            this.position = position;
            this.color = color;
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new InteractiveCanvasPainter().setVisible(true));
    }
}
```

### Explanation

1. **Mouse Drag & Click Integration**:
   - `addMouseListener` and `addMouseMotionListener` capture mouse coordinates during `mousePressed()` and `mouseDragged()`, storing points in a list for `paintComponent()` rendering.

2. **Keyboard Stamping (`keyTyped`)**:
   - Captures typed characters (`e.getKeyChar()`) and draws them on the canvas at `currentCursorPoint`.

3. **Focus Acquisition (`requestFocusInWindow()`)**:
   - `mousePressed()` calls `requestFocusInWindow()` on `DrawingPanel` so the canvas immediately captures keystrokes when clicked.
