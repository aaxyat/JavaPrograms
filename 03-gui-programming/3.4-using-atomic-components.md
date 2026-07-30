# Unit 3: GUI Programming

## Using Atomic Components

In Java Swing, **Atomic Components** (also referred to as leaf components) are self-contained GUI controls designed for single user-interface tasks—such as displaying text (`JLabel`), receiving clicks (`JButton`), capturing input (`JTextField`), or toggling options (`JCheckBox`). Unlike containers, atomic components do not hold other UI controls.

---

### Core Concepts

#### 1. Atomic Components vs. Containers

```
                    java.awt.Component
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
    java.awt.Container             javax.swing.JComponent
   (JPanel, JFrame, etc.)                   │
                                ┌───────────┴───────────┐
                                ▼                       ▼
                        Atomic Components       Complex Components
                     (JLabel, JButton, etc.)    (JTable, JTree, etc.)
```

- **Atomic Components**: Standalone controls that perform a single visual or interaction function (`JLabel`, `JButton`, `JTextField`, `JPasswordField`, `JCheckBox`, `JRadioButton`, `JToggleButton`).
- **Containers**: Structural elements designed to hold and arrange atomic components (`JPanel`, `JFrame`, `JDialog`).

#### 2. Core Atomic Components Reference

| Atomic Component | Class Name | Primary Purpose | Common Methods |
|------------------|------------|-----------------|----------------|
| **Label** | `JLabel` | Displays static text or icon images. | `setText()`, `setIcon()`, `setHorizontalAlignment()` |
| **Push Button** | `JButton` | Triggers action events on user click. | `addActionListener()`, `setEnabled()`, `setToolTipText()` |
| **Text Field** | `JTextField` | Captures single-line editable text. | `getText()`, `setText()`, `setEditable()` |
| **Password Field** | `JPasswordField` | Captures masked security inputs. | `getPassword()`, `setEchoChar()` |
| **Check Box** | `JCheckBox` | Toggles independent options on or off. | `isSelected()`, `setSelected()` |
| **Radio Button** | `JRadioButton` | Represents one option in a mutually exclusive group. | `isSelected()`, requires `ButtonGroup` |
| **Toggle Button** | `JToggleButton` | Two-state on/off push button. | `isSelected()`, `setText()` |

#### 3. Shared Atomic Component Properties
All atomic Swing components inherit from `JComponent`, providing shared styling and interaction methods:
- **`setEnabled(boolean)`**: Enables or grays out (disables) user interaction.
- **`setToolTipText(String)`**: Displays floating hover help text.
- **`setForeground(Color)` / `setBackground(Color)`**: Sets text and background colors.
- **`setFont(Font)`**: Sets typography font family, size, and weight.

---

### Common Pitfalls

1. **Treating Atomic Components as Containers**:
   - Calling `button.add(label)` throws an exception or fails visually because atomic components are not designed to host child controls. Always add atomic components into container panels (`JPanel`).
2. **Forgetting `ButtonGroup` for Radio Buttons**:
   - Adding `JRadioButton` controls to a panel without adding them to a `ButtonGroup` allows multiple radio buttons to be selected simultaneously.
3. **Using `getText()` on `JPasswordField`**:
   - `JPasswordField.getText()` is deprecated for security reasons because strings remain in JVM memory until garbage collection. Use `JPasswordField.getPassword()` which returns a clearable `char[]` array.

---

## 1. Demo: Atomic Components Showcase

### `AtomicComponentsDemo.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JButton;
import javax.swing.JTextField;
import javax.swing.JPasswordField;
import javax.swing.JCheckBox;
import javax.swing.JRadioButton;
import javax.swing.JToggleButton;
import javax.swing.ButtonGroup;
import javax.swing.JPanel;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Demonstrates essential atomic components (JLabel, JButton, JTextField, JPasswordField, JCheckBox, JRadioButton, JToggleButton).
 */
public class AtomicComponentsDemo extends JFrame {

    private JLabel statusLabel;
    private JTextField inputField;
    private JPasswordField pinField;
    private JCheckBox rememberCheckBox;
    private JRadioButton modeOptionA, modeOptionB;
    private JToggleButton powerToggleButton;
    private JButton executeButton;

    public AtomicComponentsDemo() {
        setTitle("Swing Atomic Components Demo");
        setSize(460, 360);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new GridLayout(7, 2, 8, 8));

        // 1. JLabel (Header & Description)
        JLabel titleLabel = new JLabel(" Atomic Controls:");
        titleLabel.setFont(new Font("SansSerif", Font.BOLD, 14));
        titleLabel.setForeground(Color.BLUE);

        statusLabel = new JLabel("Status: Ready");
        statusLabel.setToolTipText("Displays current system feedback");

        // 2. JTextField & JPasswordField
        JLabel userLabel = new JLabel(" Username:");
        inputField = new JTextField(15);
        inputField.setToolTipText("Enter account handle");

        JLabel pinLabel = new JLabel(" Security PIN:");
        pinField = new JPasswordField(15);

        // 3. JCheckBox
        JLabel checkLabel = new JLabel(" Preference:");
        rememberCheckBox = new JCheckBox("Remember My Credentials");

        // 4. JRadioButton & ButtonGroup
        JLabel radioLabel = new JLabel(" Mode Selection:");
        modeOptionA = new JRadioButton("Standard", true);
        modeOptionB = new JRadioButton("Express");
        ButtonGroup modeGroup = new ButtonGroup();
        modeGroup.add(modeOptionA);
        modeGroup.add(modeOptionB);

        JPanel radioContainer = new JPanel();
        radioContainer.add(modeOptionA);
        radioContainer.add(modeOptionB);

        // 5. JToggleButton
        JLabel toggleLabel = new JLabel(" System Power:");
        powerToggleButton = new JToggleButton("Power OFF");
        powerToggleButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                if (powerToggleButton.isSelected()) {
                    powerToggleButton.setText("Power ON");
                    powerToggleButton.setBackground(Color.GREEN);
                } else {
                    powerToggleButton.setText("Power OFF");
                    powerToggleButton.setBackground(Color.LIGHT_GRAY);
                }
            }
        });

        // 6. JButton (Trigger Execution)
        executeButton = new JButton("Submit Atomic Form");
        executeButton.setToolTipText("Click to validate and process atomic controls");
        executeButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                processAtomicForm();
            }
        });

        // Add Controls to Grid Container
        add(titleLabel);
        add(statusLabel);
        add(userLabel);
        add(inputField);
        add(pinLabel);
        add(pinField);
        add(checkLabel);
        add(rememberCheckBox);
        add(radioLabel);
        add(radioContainer);
        add(toggleLabel);
        add(powerToggleButton);
        add(new JLabel(" Action:"));
        add(executeButton);
    }

    private void processAtomicForm() {
        String username = inputField.getText().trim();
        int pinLen = pinField.getPassword().length;
        boolean remember = rememberCheckBox.isSelected();
        String mode = modeOptionA.isSelected() ? "Standard" : "Express";
        boolean power = powerToggleButton.isSelected();

        if (username.isEmpty() || pinLen == 0) {
            JOptionPane.showMessageDialog(this, "Please complete Username and PIN fields.", "Validation Notice", JOptionPane.WARNING_MESSAGE);
            return;
        }

        String summary = String.format("Atomic Data Captured:\nUsername: %s\nPIN Length: %d chars\nRemember: %b\nMode: %s\nPower State: %b",
                username, pinLen, remember, mode, power);

        JOptionPane.showMessageDialog(this, summary, "Form Data Submitted", JOptionPane.INFORMATION_MESSAGE);
        statusLabel.setText("Status: Form Processed (" + username + ")");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new AtomicComponentsDemo().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Atomic Controls Showcase**:
   - Demonstrates individual atomic component creation (`JLabel`, `JTextField`, `JPasswordField`, `JCheckBox`, `JRadioButton`, `JToggleButton`, and `JButton`).

2. **Tooltips (`setToolTipText`)**:
   - `executeButton.setToolTipText(...)` attaches hover guidance text displayed when the mouse hovers over the button.

3. **Dynamic State Toggling (`JToggleButton`)**:
   - `powerToggleButton.isSelected()` toggles button label text (`Power ON` vs `Power OFF`) and background color dynamically.

---

## 2. Practical Program: Smart Home IoT Control Dashboard

### `InteractiveControlPanelApp.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JButton;
import javax.swing.JCheckBox;
import javax.swing.JRadioButton;
import javax.swing.JToggleButton;
import javax.swing.ButtonGroup;
import javax.swing.JPanel;
import javax.swing.BorderFactory;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * IoT Control Dashboard utilizing atomic components for device status, toggles, mode selections, and master override triggers.
 */
public class InteractiveControlPanelApp extends JFrame {

    private JLabel statusBannerLabel;
    private JToggleButton masterPowerToggle;
    private JCheckBox securityAlarmCheckBox;
    private JCheckBox motionSensorCheckBox;
    private JRadioButton ecoModeRadio, performanceModeRadio;
    private JButton applySettingsButton;
    private JButton emergencyResetButton;

    public InteractiveControlPanelApp() {
        setTitle("Smart Home IoT Control Panel");
        setSize(480, 380);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Section (JLabel)
        statusBannerLabel = new JLabel("SYSTEM STATUS: ONLINE", JLabel.CENTER);
        statusBannerLabel.setFont(new Font("SansSerif", Font.BOLD, 15));
        statusBannerLabel.setOpaque(true);
        statusBannerLabel.setBackground(new Color(34, 197, 94));
        statusBannerLabel.setForeground(Color.WHITE);

        // Section 1: Atomic Power Toggle (JToggleButton)
        JPanel powerPanel = new JPanel(new GridLayout(1, 2, 10, 10));
        powerPanel.setBorder(BorderFactory.createTitledBorder("1. Master Power Switch"));
        masterPowerToggle = new JToggleButton("Main Power: ACTIVE", true);
        masterPowerToggle.setBackground(new Color(220, 252, 231));
        powerPanel.add(new JLabel("System Grid State:"));
        powerPanel.add(masterPowerToggle);

        // Section 2: Security Toggles (JCheckBox)
        JPanel securityPanel = new JPanel(new GridLayout(1, 2));
        securityPanel.setBorder(BorderFactory.createTitledBorder("2. Security Subsystems"));
        securityAlarmCheckBox = new JCheckBox("Perimeter Alarm", true);
        motionSensorCheckBox = new JCheckBox("Night Motion Detectors", false);
        securityPanel.add(securityAlarmCheckBox);
        securityPanel.add(motionSensorCheckBox);

        // Section 3: Operating Modes (JRadioButton + ButtonGroup)
        JPanel modePanel = new JPanel(new GridLayout(1, 2));
        modePanel.setBorder(BorderFactory.createTitledBorder("3. HVAC Operation Mode"));
        ecoModeRadio = new JRadioButton("Eco-Saving Mode", true);
        performanceModeRadio = new JRadioButton("Max Comfort Mode");
        ButtonGroup hvacGroup = new ButtonGroup();
        hvacGroup.add(ecoModeRadio);
        hvacGroup.add(performanceModeRadio);
        modePanel.add(ecoModeRadio);
        modePanel.add(performanceModeRadio);

        // Center Container Assembly
        JPanel centerContainer = new JPanel(new GridLayout(3, 1, 8, 8));
        centerContainer.add(powerPanel);
        centerContainer.add(securityPanel);
        centerContainer.add(modePanel);

        // Section 4: Action Buttons (JButton)
        JPanel actionPanel = new JPanel();
        applySettingsButton = new JButton("Apply Atomic Configuration");
        emergencyResetButton = new JButton("Emergency Shutdown");
        emergencyResetButton.setBackground(new Color(239, 68, 68));
        emergencyResetButton.setForeground(Color.WHITE);

        actionPanel.add(applySettingsButton);
        actionPanel.add(emergencyResetButton);

        // Event Listeners
        masterPowerToggle.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                if (masterPowerToggle.isSelected()) {
                    masterPowerToggle.setText("Main Power: ACTIVE");
                    statusBannerLabel.setText("SYSTEM STATUS: ONLINE");
                    statusBannerLabel.setBackground(new Color(34, 197, 94));
                } else {
                    masterPowerToggle.setText("Main Power: OFFLINE");
                    statusBannerLabel.setText("SYSTEM STATUS: OFFLINE");
                    statusBannerLabel.setBackground(Color.GRAY);
                }
            }
        });

        applySettingsButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String mode = ecoModeRadio.isSelected() ? "Eco Mode" : "Performance Mode";
                boolean alarm = securityAlarmCheckBox.isSelected();
                statusBannerLabel.setText("CONFIG UPDATED: " + mode + " | Alarm: " + (alarm ? "ON" : "OFF"));
            }
        });

        emergencyResetButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                masterPowerToggle.setSelected(false);
                masterPowerToggle.setText("Main Power: OFFLINE");
                securityAlarmCheckBox.setSelected(false);
                motionSensorCheckBox.setSelected(false);
                statusBannerLabel.setText("SYSTEM EMERGENCY SHUTDOWN COMPLETED");
                statusBannerLabel.setBackground(new Color(239, 68, 68));
            }
        });

        // Frame Layout
        setLayout(new BorderLayout(10, 10));
        add(statusBannerLabel, BorderLayout.NORTH);
        add(centerContainer, BorderLayout.CENTER);
        add(actionPanel, BorderLayout.SOUTH);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new InteractiveControlPanelApp().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Atomic Status Indicator (`JLabel`)**:
   - `statusBannerLabel.setOpaque(true)` enables background color rendering on the `JLabel`, allowing real-time green/red status updates across system actions.

2. **Master Reset via Atomic Buttons**:
   - `emergencyResetButton` triggers state resets on `masterPowerToggle`, `securityAlarmCheckBox`, and `motionSensorCheckBox`, updating `statusBannerLabel` instantly.
