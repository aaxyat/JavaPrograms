# Unit 3: GUI Programming

## Using Swing Components

Swing (`javax.swing`) provides a flexible component architecture built on top of AWT. Swing components are pure Java lightweight controls that manage their own rendering, events, and layout positioning.

---

### Core Concepts

#### 1. Swing Component Overview
Swing components are categorized based on their role within a user interface:

| Component Category | Key Classes | Description |
|--------------------|-------------|-------------|
| **Top-Level Containers** | `JFrame`, `JDialog`, `JApplet` | Heavyweight window structures that serve as the root destination for all other components. |
| **Intermediate Containers** | `JPanel`, `JScrollPane`, `JSplitPane`, `JTabbedPane` | Non-top-level containers that group, layer, and organize child components. |
| **Atomic (Leaf) Controls** | `JLabel`, `JButton`, `JTextField`, `JCheckBox`, `JRadioButton` | Self-contained interactive controls that capture user input or display content. |
| **Complex Data Controls** | `JTable`, `JTree`, `JList`, `JComboBox` | Data-driven controls designed to visualize multi-row or hierarchical datasets. |

#### 2. Key Methods Shared Across Swing Components
Because all Swing controls extend `JComponent`, they share common behavior and customization methods:

- **Visual Customization**:
  - `setFont(Font font)`: Sets font family, style (Plain, Bold, Italic), and point size.
  - `setForeground(Color fg)`: Sets text or foreground rendering color.
  - `setBackground(Color bg)`: Sets background fill color.
  - `setOpaque(boolean isOpaque)`: Controls whether the component paints its background pixels.
- **State & Interaction**:
  - `setEnabled(boolean enabled)`: Enables or grays out user interaction.
  - `setVisible(boolean visible)`: Shows or hides the component.
  - `setToolTipText(String text)`: Displays hover guidance text.

---

### Common Pitfalls

1. **Forgetting `setOpaque(true)` on `JLabel`**:
   - `JLabel` instances are transparent by default in Swing. Calling `label.setBackground(Color.YELLOW)` has no visible effect unless you explicitly call `label.setOpaque(true)`.
2. **Calling GUI methods before component instantiation**:
   - Declaring `JButton submitBtn;` without calling `new JButton("Submit")` throws a `NullPointerException` when trying to configure or add the button to a panel.

---

## 1. Demo: Swing Component Customization

### `SwingComponentsBasicsDemo.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JButton;
import javax.swing.JTextField;
import javax.swing.JPanel;
import javax.swing.SwingUtilities;
import java.awt.FlowLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Demonstrates component properties (fonts, foreground/background colors, opacity, tooltips, and state toggling).
 */
public class SwingComponentsBasicsDemo extends JFrame {

    private JLabel titleLabel;
    private JTextField inputField;
    private JButton actionButton;
    private JButton toggleButton;

    public SwingComponentsBasicsDemo() {
        setTitle("Swing Component Properties Demo");
        setSize(420, 200);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        JPanel panel = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 15));

        // 1. Customized Opaque JLabel
        titleLabel = new JLabel(" Custom Swing Label ");
        titleLabel.setFont(new Font("Serif", Font.BOLD, 16));
        titleLabel.setOpaque(true);
        titleLabel.setBackground(Color.BLUE);
        titleLabel.setForeground(Color.WHITE);
        titleLabel.setToolTipText("An opaque label with custom background");

        // 2. Customized JTextField
        inputField = new JTextField(15);
        inputField.setFont(new Font("SansSerif", Font.PLAIN, 14));
        inputField.setToolTipText("Enter text here");

        // 3. Action Buttons
        actionButton = new JButton("Process Text");
        toggleButton = new JButton("Disable Input");

        actionButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String text = inputField.getText().trim();
                if (!text.isEmpty()) {
                    titleLabel.setText(" Entered: " + text + " ");
                }
            }
        });

        toggleButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                boolean currentState = inputField.isEnabled();
                inputField.setEnabled(!currentState);
                actionButton.setEnabled(!currentState);
                toggleButton.setText(currentState ? "Enable Input" : "Disable Input");
            }
        });

        panel.add(titleLabel);
        panel.add(inputField);
        panel.add(actionButton);
        panel.add(toggleButton);

        add(panel);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new SwingComponentsBasicsDemo().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Setting Opaque Backgrounds**:
   - `titleLabel.setOpaque(true)` allows `titleLabel.setBackground(Color.BLUE)` to paint a blue background block behind white text.

2. **Dynamic State Management (`setEnabled`)**:
   - Clicking `toggleButton` flips `inputField.setEnabled(...)` and `actionButton.setEnabled(...)`, graying out or restoring controls dynamically.

---

## 2. Practical Program: Multi-Choice Order Form Application

### `OrderFormApplication.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JComboBox;
import javax.swing.JRadioButton;
import javax.swing.ButtonGroup;
import javax.swing.JCheckBox;
import javax.swing.JTextArea;
import javax.swing.JScrollPane;
import javax.swing.JButton;
import javax.swing.JPanel;
import javax.swing.BorderFactory;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.BorderLayout;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Multi-component Swing application combining dropdown menus, radio buttons, checkboxes, and scrollable text output.
 */
public class OrderFormApplication extends JFrame {

    private JComboBox<String> serviceCombo;
    private JRadioButton standardRadio, expressRadio;
    private JCheckBox giftWrapBox, insuranceBox;
    private JTextArea summaryArea;
    private JButton calculateButton;

    public OrderFormApplication() {
        setTitle("Service Order Form Application");
        setSize(480, 420);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Panel 1: Service Selection
        JPanel servicePanel = new JPanel(new GridLayout(1, 2, 5, 5));
        servicePanel.setBorder(BorderFactory.createTitledBorder("1. Select Service Package"));
        String[] services = {"Standard Courier ($15)", "Express Air ($30)", "International Cargo ($75)"};
        serviceCombo = new JComboBox<>(services);
        servicePanel.add(new JLabel("Service Type:"));
        servicePanel.add(serviceCombo);

        // Panel 2: Delivery Speed (Radio Buttons)
        JPanel speedPanel = new JPanel(new GridLayout(1, 2));
        speedPanel.setBorder(BorderFactory.createTitledBorder("2. Delivery Priority"));
        standardRadio = new JRadioButton("Standard Ground", true);
        expressRadio = new JRadioButton("Priority Air (+ $10)");
        ButtonGroup speedGroup = new ButtonGroup();
        speedGroup.add(standardRadio);
        speedGroup.add(expressRadio);
        speedPanel.add(standardRadio);
        speedPanel.add(expressRadio);

        // Panel 3: Add-on Options (Checkboxes)
        JPanel addonPanel = new JPanel(new GridLayout(1, 2));
        addonPanel.setBorder(BorderFactory.createTitledBorder("3. Add-on Services"));
        giftWrapBox = new JCheckBox("Gift Packaging (+ $5)");
        insuranceBox = new JCheckBox("Shipping Insurance (+ $8)");
        addonPanel.add(giftWrapBox);
        addonPanel.add(insuranceBox);

        // Top Options Container
        JPanel optionsContainer = new JPanel(new GridLayout(3, 1, 5, 5));
        optionsContainer.add(servicePanel);
        optionsContainer.add(speedPanel);
        optionsContainer.add(addonPanel);

        // Bottom Summary & Action Controls
        calculateButton = new JButton("Compute Total & Generate Summary");
        summaryArea = new JTextArea(6, 35);
        summaryArea.setEditable(false);
        JScrollPane summaryScroll = new JScrollPane(summaryArea);

        JPanel bottomPanel = new JPanel(new BorderLayout(5, 5));
        bottomPanel.add(calculateButton, BorderLayout.NORTH);
        bottomPanel.add(summaryScroll, BorderLayout.CENTER);

        // Frame Layout
        setLayout(new BorderLayout(10, 10));
        add(optionsContainer, BorderLayout.NORTH);
        add(bottomPanel, BorderLayout.CENTER);

        calculateButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                computeOrderSummary();
            }
        });
    }

    private void computeOrderSummary() {
        double total = 0.0;
        StringBuilder summary = new StringBuilder();
        summary.append("==========================================\n");
        summary.append("         ORDER SUMMARY SUMMARY            \n");
        summary.append("==========================================\n");

        int serviceIndex = serviceCombo.getSelectedIndex();
        double[] serviceRates = {15.0, 30.0, 75.0};
        total += serviceRates[serviceIndex];
        summary.append("Service Package : ").append(serviceCombo.getSelectedItem()).append("\n");

        if (expressRadio.isSelected()) {
            total += 10.0;
            summary.append("Priority Air    : Yes (+ $10.00)\n");
        } else {
            summary.append("Priority Air    : No\n");
        }

        if (giftWrapBox.isSelected()) {
            total += 5.0;
            summary.append("Gift Packaging  : Yes (+ $5.00)\n");
        }
        if (insuranceBox.isSelected()) {
            total += 8.0;
            summary.append("Insurance       : Yes (+ $8.00)\n");
        }

        summary.append("------------------------------------------\n");
        summary.append(String.format("TOTAL COST      : $%.2f%n", total));
        summary.append("==========================================\n");

        summaryArea.setText(summary.toString());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new OrderFormApplication().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Combining Swing Components**:
   - Demonstrates building a multi-section form using `JComboBox`, `JRadioButton`, `JCheckBox`, `JTextArea`, `JScrollPane`, and `BorderFactory` titled panels.
