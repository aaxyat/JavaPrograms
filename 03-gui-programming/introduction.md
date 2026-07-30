# Unit 3: GUI Programming

## Introduction to GUI Programming

Graphical User Interface (GUI) programming enables users to interact with applications visually using windows, buttons, text fields, menus, and mouse gestures instead of text-based command-line interfaces.

---

### Core Concepts

#### 1. Java GUI Frameworks (AWT & Swing)

Java provides two primary GUI toolkits for desktop application development:

| Toolkit | Package | Type | Description |
|---------|---------|------|-------------|
| **AWT (Abstract Window Toolkit)** | `java.awt` | Heavyweight | Original Java GUI toolkit. Binds directly to native operating system window peers. Platform-dependent appearance. |
| **Swing** | `javax.swing` | Lightweight | Built on top of AWT. Written entirely in pure Java without relying on native OS peers. Supports pluggable look-and-feel themes. |

#### 2. Component and Container Hierarchy

Java GUI architecture follows a composite pattern of **Components** and **Containers**:

```
java.awt.Component
   ├── java.awt.Container
   │     ├── java.awt.Window
   │     │     └── javax.swing.JFrame  (Top-level main window)
   │     └── javax.swing.JPanel        (Intermediate organizer container)
   └── javax.swing.JComponent
         ├── javax.swing.JButton       (Interactive button)
         ├── javax.swing.JLabel        (Display text / image)
         └── javax.swing.JTextField    (Single-line text input)
```

- **Component**: Any GUI element that can be displayed on screen (button, label, text field).
- **Container**: A component that can hold other components (frame, panel, dialog).

#### 3. Event-Driven Programming Model
Unlike sequential CLI programs that run from top to bottom, GUI applications operate on an **event loop**:
1. Application renders GUI and enters an idle state.
2. User performs an action (clicking a button, typing text).
3. The operating system creates an **Event Object** (e.g. `ActionEvent`).
4. Java dispatches the event to a registered **EventListener** (e.g. `ActionListener`).
5. The listener method (`actionPerformed()`) executes the response logic.

---

### Common Pitfalls

1. **Forgetting `setDefaultCloseOperation()`**:
   - Closing a `JFrame` window by clicking the 'X' button hides the window from view, but leaves the JVM process running in the background unless configured:
     ```java
     frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
     ```
2. **Forgetting `setVisible(true)`**:
   - `JFrame` windows are invisible by default when created. You must explicitly call `frame.setVisible(true)` as the final setup step.
3. **Modifying GUI components outside the Event Dispatch Thread (EDT)**:
   - Swing components are not thread-safe. Creating or modifying UI controls from background worker threads causes race conditions. Always launch Swing GUIs inside `SwingUtilities.invokeLater()`.

---

## 1. Demo: Basic Swing Window and Event Listener

### `GuiBasicsDemo.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JButton;
import javax.swing.JPanel;
import javax.swing.SwingUtilities;
import java.awt.FlowLayout;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Basic Swing desktop GUI application with text input, button click listener, and dynamic label update.
 */
public class GuiBasicsDemo {

    public static void createAndShowGUI() {
        // 1. Create Main Window Frame
        JFrame frame = new JFrame("Swing GUI Basics Demo");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 150);
        frame.setLocationRelativeTo(null); // Center window on screen

        // 2. Create Panel Container with FlowLayout
        JPanel panel = new JPanel(new FlowLayout());

        // 3. Create GUI Components
        JLabel nameLabel = new JLabel("Enter Name:");
        JTextField nameField = new JTextField(15);
        JButton submitBtn = new JButton("Greet Me");
        JLabel resultLabel = new JLabel("Status: Awaiting input...");

        // 4. Attach Event Listener to Button
        submitBtn.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String inputName = nameField.getText().trim();
                if (!inputName.isEmpty()) {
                    resultLabel.setText("Hello, " + inputName + "! Welcome to Swing GUI.");
                } else {
                    resultLabel.setText("Please enter your name above.");
                }
            }
        });

        // 5. Add Components to Panel Container
        panel.add(nameLabel);
        panel.add(nameField);
        panel.add(submitBtn);
        panel.add(resultLabel);

        // 6. Attach Panel to Frame and Make Visible
        frame.add(panel);
        frame.setVisible(true);
    }

    public static void main(String[] args) {
        // Schedule GUI creation on Event Dispatch Thread (EDT)
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                createAndShowGUI();
            }
        });
    }
}
```

### Explanation

1. **Thread-Safe Event Dispatch Launch (`SwingUtilities.invokeLater`)**:
   - Ensures that window creation and component setup execute on Swing's dedicated Event Dispatch Thread (EDT).

2. **Window Container Configuration (`JFrame`)**:
   - `frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)` ensures that closing the window terminates the application process cleanly.
   - `frame.setLocationRelativeTo(null)` centers the window on screen.

3. **Event Listener Registration (`addActionListener`)**:
   - Attaches an anonymous `ActionListener` implementation to `submitBtn`.
   - When clicked, `actionPerformed()` reads input from `nameField.getText()` and updates `resultLabel.setText()`.

---

## 2. Practical Program: User Registration Form Application

### `UserRegistrationForm.java`

```java
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JPasswordField;
import javax.swing.JCheckBox;
import javax.swing.JButton;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

/**
 * Desktop User Registration Form with layout management, input fields, password masking, and validation.
 */
public class UserRegistrationForm extends JFrame implements ActionListener {

    private JTextField usernameField;
    private JTextField emailField;
    private JPasswordField passwordField;
    private JCheckBox termsCheckBox;
    private JButton registerButton;
    private JButton clearButton;

    public UserRegistrationForm() {
        // Set Frame Properties
        setTitle("User Registration Portal");
        setSize(420, 260);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Use 6x2 GridLayout (6 rows, 2 columns with 10px spacing)
        setLayout(new GridLayout(6, 2, 10, 10));

        // Instantiate Controls
        JLabel userLabel = new JLabel(" Username:");
        usernameField = new JTextField();

        JLabel emailLabel = new JLabel(" Email Address:");
        emailField = new JTextField();

        JLabel passLabel = new JLabel(" Password:");
        passwordField = new JPasswordField();

        JLabel termsLabel = new JLabel(" Terms & Conditions:");
        termsCheckBox = new JCheckBox("I accept all terms");

        registerButton = new JButton("Register Account");
        clearButton = new JButton("Clear Form");

        // Register Action Listeners
        registerButton.addActionListener(this);
        clearButton.addActionListener(this);

        // Add Controls to Grid
        add(userLabel);
        add(usernameField);
        add(emailLabel);
        add(emailField);
        add(passLabel);
        add(passwordField);
        add(termsLabel);
        add(termsCheckBox);
        add(registerButton);
        add(clearButton);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == registerButton) {
            handleRegistration();
        } else if (e.getSource() == clearButton) {
            clearFields();
        }
    }

    private void handleRegistration() {
        String username = usernameField.getText().trim();
        String email = emailField.getText().trim();
        String password = new String(passwordField.getPassword());
        boolean acceptedTerms = termsCheckBox.isSelected();

        // Validation Checks
        if (username.isEmpty() || email.isEmpty() || password.isEmpty()) {
            JOptionPane.showMessageDialog(this, "ERROR: All fields are required.", "Registration Failed", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (!email.contains("@") || !email.contains(".")) {
            JOptionPane.showMessageDialog(this, "ERROR: Invalid email format.", "Registration Failed", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (!acceptedTerms) {
            JOptionPane.showMessageDialog(this, "NOTICE: You must accept terms and conditions.", "Terms Required", JOptionPane.WARNING_MESSAGE);
            return;
        }

        // Success Confirmation Dialog
        String successMsg = "Account created successfully!\nUsername: " + username + "\nEmail: " + email;
        JOptionPane.showMessageDialog(this, successMsg, "Registration Successful", JOptionPane.INFORMATION_MESSAGE);
        clearFields();
    }

    private void clearFields() {
        usernameField.setText("");
        emailField.setText("");
        passwordField.setText("");
        termsCheckBox.setSelected(false);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new UserRegistrationForm().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **`JPasswordField` Usage**:
   - `passwordField.getPassword()` returns a `char[]` array representing masked password characters, which is converted to a string for validation.

2. **Grid Layout Alignment (`GridLayout(6, 2, 10, 10)`)**:
   - Arranges UI elements into neat rows and columns with horizontal and vertical gaps.

3. **Dialog Popups (`JOptionPane`)**:
   - `JOptionPane.showMessageDialog(...)` displays modal message dialogs for error warnings and success confirmations.
