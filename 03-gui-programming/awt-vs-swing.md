# Unit 3: GUI Programming

## AWT vs. Swing

Java provides two primary GUI frameworks in the standard library: **Abstract Window Toolkit (AWT)** and **Swing**. Understanding their structural differences is essential for choosing the right framework and building reliable desktop interfaces.

---

### Core Concepts

#### 1. Detailed Comparison Matrix

| Feature | AWT (`java.awt`) | Swing (`javax.swing`) |
|---------|------------------|-----------------------|
| **Component Type** | **Heavyweight** (Each component relies on a native OS window peer). | **Lightweight** (Written in 100% pure Java without native OS peers). |
| **Platform Appearance** | **Platform Dependent**. Uses the host operating system's native look and feel. | **Platform Independent**. Uniform appearance across all operating systems. |
| **Pluggable Look & Feel** | Not supported. | **Supported** (Metal, Nimbus, Windows, Motif look-and-feel themes). |
| **Speed & Performance** | Faster initial rendering due to OS native peer execution. | Slightly slower at initial startup, but highly optimized for complex UIs. |
| **Memory Footprint** | Higher OS resource overhead per component. | Lower OS resource overhead; managed entirely by the JVM. |
| **Double Buffering** | Not supported by default (requires manual double-buffer code to prevent flicker). | **Built-in automatically** to eliminate UI screen flickering. |
| **Component Richness** | Limited set of basic components (Button, Label, List). | Rich set of advanced controls (`JTable`, `JTree`, `JTabbedPane`, `JSlider`). |
| **Component Naming** | Prefix-less (`Button`, `Label`, `TextField`, `Frame`). | Prefixed with **'J'** (`JButton`, `JLabel`, `JTextField`, `JFrame`). |

---

#### 2. Component Class Mapping

```
AWT Component Class (java.awt)        Swing Component Class (javax.swing)
-----------------------------        -----------------------------------
Frame                                 JFrame
Panel                                 JPanel
Button                                JButton
Label                                 JLabel
TextField                             JTextField
TextArea                              JTextArea
Checkbox                              JCheckBox
Choice (Dropdown)                     JComboBox
List                                  JList
Dialog                                JDialog
```

---

### Common Pitfalls

1. **Mixing Heavyweight (AWT) and Lightweight (Swing) Components**:
   - Adding an AWT `Button` inside a Swing `JPanel` next to a `JButton` causes **Z-ordering display bugs**. Because AWT components use native OS peers, the operating system draws them on top of Swing components regardless of component layering. Avoid mixing AWT and Swing controls in the same window.
2. **Forgetting the 'J' Prefix in Swing**:
   - Accidentally writing `Button btn = new Button();` inside a Swing application imports `java.awt.Button` instead of `javax.swing.JButton`, leading to unexpected layout and styling behavior.
3. **Attempting to apply Look-and-Feel themes to AWT components**:
   - AWT controls rely on OS native window peers and ignore `UIManager.setLookAndFeel()`. Look-and-Feel themes apply exclusively to Swing (`javax.swing`) components.

---

## 1. Demo: Side-by-Side Comparison (AWT Frame vs. Swing JFrame)

### `AwtVsSwingComparisonDemo.java`

```java
import java.awt.Frame;
import java.awt.Button;
import java.awt.Label;
import java.awt.TextField;
import java.awt.FlowLayout;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

import javax.swing.JFrame;
import javax.swing.JButton;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JPanel;
import javax.swing.SwingUtilities;

/**
 * Creates two side-by-side desktop windows demonstrating AWT (heavyweight) vs Swing (lightweight) controls.
 */
public class AwtVsSwingComparisonDemo {

    // 1. Create Heavyweight AWT Window
    public static void createAwtWindow() {
        Frame awtFrame = new Frame("AWT Heavyweight Window (java.awt)");
        awtFrame.setLayout(new FlowLayout());
        awtFrame.setSize(380, 160);
        awtFrame.setLocation(100, 200);

        Label label = new Label("AWT Label:");
        TextField textField = new TextField("Native OS Text", 12);
        Button button = new Button("AWT Button");

        awtFrame.add(label);
        awtFrame.add(textField);
        awtFrame.add(button);

        // AWT requires explicit window listener for closing
        awtFrame.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                awtFrame.dispose();
            }
        });

        awtFrame.setVisible(true);
    }

    // 2. Create Lightweight Swing Window
    public static void createSwingWindow() {
        JFrame swingFrame = new JFrame("Swing Lightweight Window (javax.swing)");
        swingFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
        swingFrame.setSize(380, 160);
        swingFrame.setLocation(500, 200);

        JPanel panel = new JPanel(new FlowLayout());
        JLabel label = new JLabel("Swing Label:");
        JTextField textField = new JTextField("Pure Java Text", 12);
        JButton button = new JButton("Swing JButton");

        panel.add(label);
        panel.add(textField);
        panel.add(button);

        swingFrame.add(panel);
        swingFrame.setVisible(true);
    }

    public static void main(String[] args) {
        // Launch AWT Window
        createAwtWindow();

        // Launch Swing Window on Event Dispatch Thread
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                createSwingWindow();
            }
        });
    }
}
```

### Explanation

1. **AWT Window Construction (`createAwtWindow`)**:
   - Uses `java.awt.Frame`, `Label`, `TextField`, and `Button`.
   - Requires adding a `WindowAdapter` to handle the `windowClosing` event manually because AWT `Frame` does not support `setDefaultCloseOperation()`.

2. **Swing Window Construction (`createSwingWindow`)**:
   - Uses `javax.swing.JFrame`, `JLabel`, `JTextField`, and `JButton`.
   - Uses `setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE)` for built-in window closing behavior.

---

## 2. Practical Program: Advanced Swing Feature Showcase

### `ComponentShowcaseApp.java`

```java
import javax.swing.JFrame;
import javax.swing.JTabbedPane;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTable;
import javax.swing.JScrollPane;
import javax.swing.JTree;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.JProgressBar;
import javax.swing.JButton;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Demonstrates advanced Swing components unavailable in legacy AWT (JTabbedPane, JTable, JTree, JProgressBar).
 */
public class ComponentShowcaseApp extends JFrame {

    public ComponentShowcaseApp() {
        setTitle("Advanced Swing Component Showcase");
        setSize(550, 350);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        JTabbedPane tabbedPane = new JTabbedPane();

        // TAB 1: JTable Component
        JPanel tablePanel = new JPanel(new BorderLayout());
        String[] columns = {"ID", "Product Name", "Price ($)", "Stock"};
        Object[][] data = {
            {101, "Laptop Computer", 999.99, 15},
            {102, "Wireless Mouse", 25.50, 120},
            {103, "Mechanical Keyboard", 85.00, 45},
            {104, "HD Monitor 27\"", 220.00, 30}
        };
        JTable table = new JTable(data, columns);
        tablePanel.add(new JScrollPane(table), BorderLayout.CENTER);
        tabbedPane.addTab("Data Table (JTable)", tablePanel);

        // TAB 2: JTree Component
        JPanel treePanel = new JPanel(new BorderLayout());
        DefaultMutableTreeNode root = new DefaultMutableTreeNode("Project Workspace");
        DefaultMutableTreeNode srcFolder = new DefaultMutableTreeNode("src/");
        srcFolder.add(new DefaultMutableTreeNode("Main.java"));
        srcFolder.add(new DefaultMutableTreeNode("Utils.java"));
        DefaultMutableTreeNode docsFolder = new DefaultMutableTreeNode("docs/");
        docsFolder.add(new DefaultMutableTreeNode("README.md"));
        root.add(srcFolder);
        root.add(docsFolder);

        JTree tree = new JTree(root);
        treePanel.add(new JScrollPane(tree), BorderLayout.CENTER);
        tabbedPane.addTab("File Tree (JTree)", treePanel);

        // TAB 3: JProgressBar Component
        JPanel progressPanel = new JPanel();
        JLabel statusLabel = new JLabel("Task Status: Ready");
        JProgressBar progressBar = new JProgressBar(0, 100);
        progressBar.setStringPainted(true);
        JButton startButton = new JButton("Simulate Task Progress");

        startButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                progressBar.setValue(75);
                statusLabel.setText("Task Status: Processing (75% Complete)");
            }
        });

        progressPanel.add(statusLabel);
        progressPanel.add(progressBar);
        progressPanel.add(startButton);
        tabbedPane.addTab("Progress Bar", progressPanel);

        add(tabbedPane);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new ComponentShowcaseApp().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **`JTabbedPane` Organization**:
   - Organizes multiple views (`JTable`, `JTree`, `JProgressBar`) into tabbed cards, a feature impossible in native AWT without complex custom drawing.

2. **Advanced Data Displays**:
   - **`JTable`**: Renders structured tabular rows and columns with interactive column resizing.
   - **`JTree`**: Renders hierarchical tree nodes using `DefaultMutableTreeNode`.
   - **`JProgressBar`**: Provides visual feedback for task progress via `progressBar.setValue()`.
