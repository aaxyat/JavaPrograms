# Unit 3: GUI Programming

## Example GUI Programming

This reference guide provides **7 practical real-world Swing desktop applications**. Each example includes complete executable source code followed by an in-depth, line-by-line code explanation.

---

## 1. Interactive Student Grade Calculator

### `StudentGradeCalculatorGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JButton;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;

public class StudentGradeCalculatorGUI extends JFrame {

    private JTextField mathField;
    private JTextField scienceField;
    private JTextField englishField;
    private JLabel resultLabel;

    public StudentGradeCalculatorGUI() {
        setTitle("Student Grade Calculator");
        setSize(420, 260);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Panel
        JPanel headerPanel = new JPanel();
        headerPanel.setBackground(new Color(30, 58, 138));
        JLabel headerLabel = new JLabel("STUDENT MARKS & GRADE CALCULATOR");
        headerLabel.setFont(new Font("SansSerif", Font.BOLD, 14));
        headerLabel.setForeground(Color.WHITE);
        headerPanel.add(headerLabel);

        // Input Grid Panel (4 rows, 2 columns)
        JPanel inputPanel = new JPanel(new GridLayout(4, 2, 8, 8));

        inputPanel.add(new JLabel(" Math Marks (0-100):"));
        mathField = new JTextField();
        inputPanel.add(mathField);

        inputPanel.add(new JLabel(" Science Marks (0-100):"));
        scienceField = new JTextField();
        inputPanel.add(scienceField);

        inputPanel.add(new JLabel(" English Marks (0-100):"));
        englishField = new JTextField();
        inputPanel.add(englishField);

        JButton calculateButton = new JButton("Calculate Grade");
        inputPanel.add(calculateButton);

        resultLabel = new JLabel("Result: Awaiting calculation", JLabel.CENTER);
        resultLabel.setFont(new Font("SansSerif", Font.BOLD, 13));
        inputPanel.add(resultLabel);

        // Assembly
        setLayout(new BorderLayout(10, 10));
        add(headerPanel, BorderLayout.NORTH);
        add(inputPanel, BorderLayout.CENTER);

        // Action Listener
        calculateButton.addActionListener(e -> calculateGrade());
    }

    private void calculateGrade() {
        try {
            double math = Double.parseDouble(mathField.getText().trim());
            double science = Double.parseDouble(scienceField.getText().trim());
            double english = Double.parseDouble(englishField.getText().trim());

            if (math < 0 || math > 100 || science < 0 || science > 100 || english < 0 || english > 100) {
                JOptionPane.showMessageDialog(this, "Marks must be between 0 and 100.", "Input Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            double total = math + science + english;
            double percentage = total / 3.0;

            String grade;
            if (percentage >= 90) grade = "A+ (Outstanding)";
            else if (percentage >= 80) grade = "A (Excellent)";
            else if (percentage >= 70) grade = "B (Good)";
            else if (percentage >= 60) grade = "C (Satisfactory)";
            else grade = "F (Fail)";

            String resultText = String.format("Percentage: %.2f%% | Grade: %s", percentage, grade);
            resultLabel.setText(resultText);
            
            if (grade.startsWith("F")) {
                resultLabel.setForeground(Color.RED);
            } else {
                resultLabel.setForeground(new Color(22, 101, 52));
            }

        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(this, "Please enter valid numeric marks for all subjects.", "Format Error", JOptionPane.WARNING_MESSAGE);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new StudentGradeCalculatorGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–11**: Imports necessary Swing GUI controls (`JFrame`, `JPanel`, `JLabel`, `JTextField`, `JButton`, `JOptionPane`), layout managers, and color/font classes.
- **Line 13–18**: Declares class `StudentGradeCalculatorGUI` extending `JFrame` and defines instance variables for subject text fields and the result feedback label.
- **Line 20–24**: Configures main window attributes—sets window title to `"Student Grade Calculator"`, dimensions to 420x260 pixels, exit close operation, and centers the window on screen.
- **Line 26–31**: Creates `headerPanel` with a dark blue background (`Color(30, 58, 138)`) and attaches a bold white header label.
- **Line 33–48**: Instantiates `inputPanel` using a `4x2 GridLayout`. Adds label and text field pairs for Math, Science, and English marks.
- **Line 50–54**: Creates `calculateButton` and `resultLabel`, adding both into the grid container.
- **Line 56–58**: Sets window layout to `BorderLayout` and places `headerPanel` at the top (`NORTH`) and `inputPanel` in the center (`CENTER`).
- **Line 60–61**: Attaches an action listener lambda expression to `calculateButton` that triggers `calculateGrade()`.
- **Line 64–75**: `calculateGrade()` extracts text from input fields, converts them to `double` values using `Double.parseDouble()`, and validates that all marks fall within the valid `0-100` range.
- **Line 77–84**: Computes total marks and average percentage, evaluating the corresponding letter grade (`A+`, `A`, `B`, `C`, `F`).
- **Line 86–92**: Formats the percentage to 2 decimal places (`%.2f%%`), updates `resultLabel`, and colors failed grades red or passing grades green.
- **Line 94–96**: Catches `NumberFormatException` if a user types non-numeric characters, displaying an error popup via `JOptionPane`.
- **Line 98–102**: Launches the GUI on the Event Dispatch Thread (EDT) inside `main()`.

---

## 2. Real-Time Currency Converter Application

### `CurrencyConverterGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JComboBox;
import javax.swing.JButton;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.BorderLayout;
import java.awt.Font;
import java.awt.Color;

public class CurrencyConverterGUI extends JFrame {

    private JTextField amountField;
    private JComboBox<String> fromCurrencyCombo;
    private JComboBox<String> toCurrencyCombo;
    private JLabel convertedAmountLabel;

    public CurrencyConverterGUI() {
        setTitle("Real-Time Currency Converter");
        setSize(450, 240);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Banner
        JPanel bannerPanel = new JPanel();
        bannerPanel.setBackground(new Color(13, 148, 136));
        JLabel bannerLabel = new JLabel("CURRENCY CONVERTER PORTAL");
        bannerLabel.setFont(new Font("SansSerif", Font.BOLD, 15));
        bannerLabel.setForeground(Color.WHITE);
        bannerPanel.add(bannerLabel);

        // Form Grid (4 rows, 2 columns)
        JPanel formPanel = new JPanel(new GridLayout(4, 2, 10, 10));

        formPanel.add(new JLabel(" Enter Amount:"));
        amountField = new JTextField("100.00");
        formPanel.add(amountField);

        String[] currencies = {"USD (US Dollar)", "EUR (Euro)", "GBP (British Pound)", "NPR (Nepalese Rupee)"};

        formPanel.add(new JLabel(" From Currency:"));
        fromCurrencyCombo = new JComboBox<>(currencies);
        formPanel.add(fromCurrencyCombo);

        formPanel.add(new JLabel(" To Currency:"));
        toCurrencyCombo = new JComboBox<>(currencies);
        toCurrencyCombo.setSelectedIndex(3); // Default to NPR
        formPanel.add(toCurrencyCombo);

        JButton convertButton = new JButton("Convert Currency");
        formPanel.add(convertButton);

        convertedAmountLabel = new JLabel("Converted Amount: --", JLabel.CENTER);
        convertedAmountLabel.setFont(new Font("SansSerif", Font.BOLD, 13));
        formPanel.add(convertedAmountLabel);

        // Assembly
        setLayout(new BorderLayout(10, 10));
        add(bannerPanel, BorderLayout.NORTH);
        add(formPanel, BorderLayout.CENTER);

        convertButton.addActionListener(e -> performConversion());
    }

    private void performConversion() {
        try {
            double inputAmount = Double.parseDouble(amountField.getText().trim());
            if (inputAmount < 0) {
                convertedAmountLabel.setText("Amount cannot be negative!");
                return;
            }

            int fromIndex = fromCurrencyCombo.getSelectedIndex();
            int toIndex = toCurrencyCombo.getSelectedIndex();

            // Exchange rates relative to 1 USD base: [USD=1.0, EUR=0.92, GBP=0.79, NPR=133.50]
            double[] usdExchangeRates = {1.0, 0.92, 0.79, 133.50};

            // Convert input to USD base first, then convert from USD to target currency
            double amountInUSD = inputAmount / usdExchangeRates[fromIndex];
            double finalConvertedAmount = amountInUSD * usdExchangeRates[toIndex];

            String toCurrencyCode = ((String) toCurrencyCombo.getSelectedItem()).substring(0, 3);
            convertedAmountLabel.setText(String.format("Result: %.2f %s", finalConvertedAmount, toCurrencyCode));

        } catch (NumberFormatException ex) {
            convertedAmountLabel.setText("Invalid numeric amount entered.");
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new CurrencyConverterGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–10**: Imports Swing GUI components (`JFrame`, `JPanel`, `JLabel`, `JTextField`, `JComboBox`, `JButton`), layouts, and styling classes.
- **Line 12–17**: Declares class extending `JFrame` and initializes UI fields for input amount, currency selection dropdowns, and conversion result label.
- **Line 19–23**: Sets main window frame title, dimensions (450x240 pixels), exit close policy, and centers window.
- **Line 25–30**: Creates teal banner panel (`Color(13, 148, 136)`) and adds white title text.
- **Line 32–37**: Creates `formPanel` using a `4x2 GridLayout` and adds the amount input text field initialized to `"100.00"`.
- **Line 39–47**: Populates `fromCurrencyCombo` and `toCurrencyCombo` dropdowns with currency options (`USD`, `EUR`, `GBP`, `NPR`). Sets target dropdown default index to 3 (`NPR`).
- **Line 49–54**: Adds conversion action button and output label `convertedAmountLabel`.
- **Line 56–58**: Assembles banner and form panel inside frame layout.
- **Line 60**: Connects button click to `performConversion()`.
- **Line 63–70**: `performConversion()` parses input amount and checks against negative values.
- **Line 72–76**: Defines an exchange rate array relative to USD (`USD=1.0`, `EUR=0.92`, `GBP=0.79`, `NPR=133.50`).
- **Line 78–80**: Converts input to base USD value first, then multiplies by the target currency exchange rate.
- **Line 82–83**: Formats the converted result to 2 decimal places and displays the currency symbol code.
- **Line 85–87**: Catches invalid non-numeric inputs gracefully.

---

## 3. Digital Stopwatch and Lap Timer

### `DigitalStopwatchGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JButton;
import javax.swing.JTextArea;
import javax.swing.JScrollPane;
import javax.swing.Timer;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.Font;
import java.awt.Color;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

public class DigitalStopwatchGUI extends JFrame {

    private JLabel timeDisplayLabel;
    private JTextArea lapLogArea;
    private Timer timer;
    private int elapsedSeconds = 0;
    private int lapCounter = 1;
    private boolean isRunning = false;

    public DigitalStopwatchGUI() {
        setTitle("Digital Stopwatch & Lap Timer");
        setSize(450, 360);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Top Time Display Panel
        JPanel timePanel = new JPanel();
        timePanel.setBackground(new Color(15, 23, 42));
        timeDisplayLabel = new JLabel("00:00:00", JLabel.CENTER);
        timeDisplayLabel.setFont(new Font("Monospaced", Font.BOLD, 36));
        timeDisplayLabel.setForeground(new Color(56, 189, 248));
        timePanel.add(timeDisplayLabel);

        // Control Buttons Panel
        JPanel buttonPanel = new JPanel(new FlowLayout());
        JButton startPauseBtn = new JButton("Start");
        JButton lapBtn = new JButton("Record Lap");
        JButton resetBtn = new JButton("Reset");

        buttonPanel.add(startPauseBtn);
        buttonPanel.add(lapBtn);
        buttonPanel.add(resetBtn);

        // Lap History Text Area
        lapLogArea = new JTextArea(8, 30);
        lapLogArea.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(lapLogArea);

        // Swing Timer updating every 1000ms (1 second)
        timer = new Timer(1000, new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                elapsedSeconds++;
                updateTimerDisplay();
            }
        });

        // Button Event Handlers
        startPauseBtn.addActionListener(e -> {
            if (!isRunning) {
                timer.start();
                isRunning = true;
                startPauseBtn.setText("Pause");
            } else {
                timer.stop();
                isRunning = false;
                startPauseBtn.setText("Start");
            }
        });

        lapBtn.addActionListener(e -> {
            if (isRunning) {
                lapLogArea.append(String.format("Lap #%d : %s%n", lapCounter++, timeDisplayLabel.getText()));
            }
        });

        resetBtn.addActionListener(e -> {
            timer.stop();
            isRunning = false;
            elapsedSeconds = 0;
            lapCounter = 1;
            startPauseBtn.setText("Start");
            updateTimerDisplay();
            lapLogArea.setText("");
        });

        // Assembly
        setLayout(new BorderLayout(8, 8));
        add(timePanel, BorderLayout.NORTH);
        add(buttonPanel, BorderLayout.CENTER);
        add(scrollPane, BorderLayout.SOUTH);
    }

    private void updateTimerDisplay() {
        int hours = elapsedSeconds / 3600;
        int minutes = (elapsedSeconds % 3600) / 60;
        int seconds = elapsedSeconds % 60;
        timeDisplayLabel.setText(String.format("%02d:%02d:%02d", hours, minutes, seconds));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new DigitalStopwatchGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–12**: Imports Swing components including `javax.swing.Timer` for scheduled background ticks.
- **Line 14–20**: Declares fields for `timeDisplayLabel`, `lapLogArea`, `Timer`, `elapsedSeconds` counter, and state boolean `isRunning`.
- **Line 22–26**: Sets window frame dimensions (450x360 pixels), title, and exit policy.
- **Line 28–33**: Creates a dark blue time display panel using a large monospaced font (`00:00:00`) rendered in cyan (`Color(56, 189, 248)`).
- **Line 35–42**: Creates control buttons: `startPauseBtn`, `lapBtn`, and `resetBtn`.
- **Line 44–46**: Creates a read-only `JTextArea` wrapped in `JScrollPane` for recording lap times.
- **Line 48–55**: Instantiates `javax.swing.Timer(1000, ...)` configured to fire an action event every 1000 milliseconds (1 second), incrementing `elapsedSeconds` and updating the screen display.
- **Line 57–67**: `startPauseBtn` listener toggles `timer.start()` and `timer.stop()`, updating button label between `"Start"` and `"Pause"`.
- **Line 69–73**: `lapBtn` appends current formatted time string into `lapLogArea`.
- **Line 75–83**: `resetBtn` stops the timer, resets counter variables to zero, and clears recorded lap logs.
- **Line 87–92**: `updateTimerDisplay()` calculates hours, minutes, and seconds from total `elapsedSeconds` using division and modulus operators, formatting output as `HH:MM:SS`.

---

## 4. Desktop To-Do List & Task Manager

### `TodoListManagerGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JButton;
import javax.swing.JList;
import javax.swing.DefaultListModel;
import javax.swing.JScrollPane;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.Font;
import java.awt.Color;

public class TodoListManagerGUI extends JFrame {

    private JTextField taskInputField;
    private DefaultListModel<String> listModel;
    private JList<String> taskList;
    private JLabel countLabel;

    public TodoListManagerGUI() {
        setTitle("Desktop To-Do Task Manager");
        setSize(480, 380);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Input Panel
        JPanel inputPanel = new JPanel(new FlowLayout());
        inputPanel.add(new JLabel("New Task:"));
        taskInputField = new JTextField(20);
        JButton addButton = new JButton("Add Task");
        inputPanel.add(taskInputField);
        inputPanel.add(addButton);

        // Task JList backed by DefaultListModel
        listModel = new DefaultListModel<>();
        taskList = new JList<>(listModel);
        taskList.setFont(new Font("SansSerif", Font.PLAIN, 14));
        JScrollPane scrollPane = new JScrollPane(taskList);

        // Bottom Action Toolbar Panel
        JPanel actionPanel = new JPanel(new FlowLayout());
        JButton removeButton = new JButton("Remove Selected");
        JButton clearAllButton = new JButton("Clear All");
        countLabel = new JLabel("Pending Tasks: 0");

        actionPanel.add(removeButton);
        actionPanel.add(clearAllButton);
        actionPanel.add(countLabel);

        // Add Task Action
        addButton.addActionListener(e -> {
            String taskText = taskInputField.getText().trim();
            if (!taskText.isEmpty()) {
                listModel.addElement(taskText);
                taskInputField.setText("");
                updateCount();
            } else {
                JOptionPane.showMessageDialog(this, "Task description cannot be empty.", "Input Warning", JOptionPane.WARNING_MESSAGE);
            }
        });

        // Remove Selected Task Action
        removeButton.addActionListener(e -> {
            int selectedIndex = taskList.getSelectedIndex();
            if (selectedIndex != -1) {
                listModel.remove(selectedIndex);
                updateCount();
            } else {
                JOptionPane.showMessageDialog(this, "Please select a task from the list to remove.", "Selection Warning", JOptionPane.WARNING_MESSAGE);
            }
        });

        // Clear All Tasks Action
        clearAllButton.addActionListener(e -> {
            listModel.clear();
            updateCount();
        });

        // Assembly
        setLayout(new BorderLayout(8, 8));
        add(inputPanel, BorderLayout.NORTH);
        add(scrollPane, BorderLayout.CENTER);
        add(actionPanel, BorderLayout.SOUTH);
    }

    private void updateCount() {
        countLabel.setText("Pending Tasks: " + listModel.getSize());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new TodoListManagerGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–12**: Imports Swing components including `JList` and its dynamic data model `DefaultListModel`.
- **Line 14–19**: Declares class fields: `taskInputField`, `listModel` for managing list items, `taskList` for visual rendering, and `countLabel`.
- **Line 21–25**: Configures main window attributes (480x380 pixels, title, and centering).
- **Line 27–33**: Sets up `inputPanel` containing text field and `addButton`.
- **Line 35–39**: Instantiates `DefaultListModel<String>` and binds it to `JList<String> taskList`. Wraps `taskList` inside `JScrollPane`.
- **Line 41–48**: Creates `actionPanel` with `removeButton`, `clearAllButton`, and `countLabel`.
- **Line 50–60**: `addButton` listener reads text from `taskInputField`, adds it to `listModel` via `listModel.addElement()`, clears input field, and updates pending task count.
- **Line 62–71**: `removeButton` listener checks `taskList.getSelectedIndex()`. If an item is highlighted (`selectedIndex != -1`), it removes the element via `listModel.remove(selectedIndex)`.
- **Line 73–76**: `clearAllButton` clears the entire model via `listModel.clear()`.
- **Line 85–87**: `updateCount()` updates `countLabel` with current `listModel.getSize()`.

---

## 5. Simple Notepad Text Editor with File I/O

### `SimpleNotepadGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JTextArea;
import javax.swing.JScrollPane;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JButton;
import javax.swing.JFileChooser;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.Font;
import java.io.File;
import java.io.PrintWriter;
import java.io.IOException;

public class SimpleNotepadGUI extends JFrame {

    private JTextArea textArea;
    private JLabel statusLabel;

    public SimpleNotepadGUI() {
        setTitle("Simple Notepad Text Editor");
        setSize(600, 420);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Text Area Setup
        textArea = new JTextArea();
        textArea.setFont(new Font("Monospaced", Font.PLAIN, 14));
        textArea.setLineWrap(true);
        textArea.setWrapStyleWord(true);
        JScrollPane scrollPane = new JScrollPane(textArea);

        // Top Control Toolbar Panel
        JPanel toolbarPanel = new JPanel(new FlowLayout(FlowLayout.LEFT));
        JButton saveButton = new JButton("Save File");
        JButton clearButton = new JButton("Clear Text");
        toolbarPanel.add(saveButton);
        toolbarPanel.add(clearButton);

        // Bottom Status Bar
        statusLabel = new JLabel(" Words: 0 | Characters: 0");
        JPanel statusPanel = new JPanel(new FlowLayout(FlowLayout.LEFT));
        statusPanel.add(statusLabel);

        // Document Key Listener for Real-Time Word Counting
        textArea.addCaretListener(e -> updateWordCount());

        // Clear Action
        clearButton.addActionListener(e -> {
            textArea.setText("");
            updateWordCount();
        });

        // Save File Action using JFileChooser
        saveButton.addActionListener(e -> saveFileToDisk());

        // Assembly
        setLayout(new BorderLayout());
        add(toolbarPanel, BorderLayout.NORTH);
        add(scrollPane, BorderLayout.CENTER);
        add(statusPanel, BorderLayout.SOUTH);
    }

    private void updateWordCount() {
        String text = textArea.getText().trim();
        int charCount = text.length();
        int wordCount = text.isEmpty() ? 0 : text.split("\\s+").length;
        statusLabel.setText(String.format(" Words: %d | Characters: %d", wordCount, charCount));
    }

    private void saveFileToDisk() {
        JFileChooser fileChooser = new JFileChooser();
        int choice = fileChooser.showSaveDialog(this);

        if (choice == JFileChooser.APPROVE_OPTION) {
            File selectedFile = fileChooser.getSelectedFile();
            try (PrintWriter writer = new PrintWriter(selectedFile)) {
                writer.print(textArea.getText());
                JOptionPane.showMessageDialog(this, "File saved successfully to: " + selectedFile.getAbsolutePath(), "Save Success", JOptionPane.INFORMATION_MESSAGE);
            } catch (IOException ex) {
                JOptionPane.showMessageDialog(this, "Error saving file: " + ex.getMessage(), "File Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new SimpleNotepadGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–14**: Imports Swing components including `JFileChooser` for file selection dialogs and java.io classes for writing files.
- **Line 16–19**: Declares class extending `JFrame` with fields for `textArea` and `statusLabel`.
- **Line 21–25**: Configures main window attributes (600x420 pixels, title, and centering).
- **Line 27–32**: Instantiates `textArea`, sets a clean monospaced font, enables line wrapping (`setLineWrap(true)` and `setWrapStyleWord(true)`), and wraps it in a `JScrollPane`.
- **Line 34–39**: Creates `toolbarPanel` with `"Save File"` and `"Clear Text"` buttons.
- **Line 41–44**: Creates `statusLabel` displaying word and character counts.
- **Line 46–47**: Attaches a `CaretListener` to `textArea` so word and character counts update automatically whenever the text cursor moves or text changes.
- **Line 49–53**: `clearButton` clears all text and updates status counts.
- **Line 55–56**: `saveButton` triggers `saveFileToDisk()`.
- **Line 64–69**: `updateWordCount()` computes total character count (`text.length()`) and splits text by whitespace regex (`\\s+`) to compute word count.
- **Line 71–84**: `saveFileToDisk()` opens a native file save dialog via `JFileChooser`. When the user chooses a file path, `PrintWriter` writes the text area content to disk and displays a confirmation dialog.

---

## 6. Secure User Authentication & Login System

### `UserLoginAuthenticationGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JPasswordField;
import javax.swing.JCheckBox;
import javax.swing.JButton;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.GridLayout;
import java.awt.BorderLayout;
import java.awt.Font;
import java.awt.Color;

public class UserLoginAuthenticationGUI extends JFrame {

    private JTextField usernameField;
    private JPasswordField passwordField;
    private JCheckBox showPasswordCheckBox;
    private JLabel attemptsLabel;
    private int remainingAttempts = 3;

    public UserLoginAuthenticationGUI() {
        setTitle("Secure System Login Portal");
        setSize(420, 260);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Title Banner
        JPanel bannerPanel = new JPanel();
        bannerPanel.setBackground(new Color(30, 41, 59));
        JLabel bannerTitle = new JLabel("AUTHENTICATION REQUIRED");
        bannerTitle.setFont(new Font("SansSerif", Font.BOLD, 14));
        bannerTitle.setForeground(Color.WHITE);
        bannerPanel.add(bannerTitle);

        // Input Form Panel (4 rows, 2 columns)
        JPanel formPanel = new JPanel(new GridLayout(4, 2, 8, 8));

        formPanel.add(new JLabel(" Username:"));
        usernameField = new JTextField();
        formPanel.add(usernameField);

        formPanel.add(new JLabel(" Password:"));
        passwordField = new JPasswordField();
        formPanel.add(passwordField);

        formPanel.add(new JLabel(" Options:"));
        showPasswordCheckBox = new JCheckBox("Show Password");
        formPanel.add(showPasswordCheckBox);

        JButton loginButton = new JButton("Login");
        formPanel.add(loginButton);

        attemptsLabel = new JLabel("Attempts Left: 3", JLabel.CENTER);
        attemptsLabel.setForeground(new Color(225, 29, 72));
        formPanel.add(attemptsLabel);

        // Show/Hide Password Toggle Listener
        showPasswordCheckBox.addActionListener(e -> {
            if (showPasswordCheckBox.isSelected()) {
                passwordField.setEchoChar((char) 0); // Unmask password text
            } else {
                passwordField.setEchoChar('•'); // Mask password with bullet
            }
        });

        // Login Action Listener
        loginButton.addActionListener(e -> authenticateUser());

        // Frame Layout Assembly
        setLayout(new BorderLayout(10, 10));
        add(bannerPanel, BorderLayout.NORTH);
        add(formPanel, BorderLayout.CENTER);
    }

    private void authenticateUser() {
        String username = usernameField.getText().trim();
        String password = new String(passwordField.getPassword());

        // Mock Credentials Verification (admin / admin123)
        if ("admin".equals(username) && "admin123".equals(password)) {
            JOptionPane.showMessageDialog(this, "ACCESS GRANTED! Welcome, " + username + ".", "Authentication Success", JOptionPane.INFORMATION_MESSAGE);
            remainingAttempts = 3;
            attemptsLabel.setText("Attempts Left: 3");
            usernameField.setText("");
            passwordField.setText("");
        } else {
            remainingAttempts--;
            attemptsLabel.setText("Attempts Left: " + remainingAttempts);

            if (remainingAttempts > 0) {
                JOptionPane.showMessageDialog(this, "INVALID CREDENTIALS! Please try again.", "Login Failed", JOptionPane.ERROR_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(this, "ACCOUNT LOCKED! Exceeded maximum login attempts.", "Security Alert", JOptionPane.ERROR_MESSAGE);
                usernameField.setEnabled(false);
                passwordField.setEnabled(false);
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new UserLoginAuthenticationGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–11**: Imports Swing components including `JPasswordField`, `JCheckBox`, and `JOptionPane`.
- **Line 13–19**: Declares class extending `JFrame` with inputs for `usernameField`, `passwordField`, `showPasswordCheckBox`, `attemptsLabel`, and counter `remainingAttempts = 3`.
- **Line 21–25**: Configures main window attributes (420x260 pixels, title, exit operation, and centering).
- **Line 27–32**: Creates header banner panel styled with a dark slate background (`Color(30, 41, 59)`).
- **Line 34–55**: Sets up `formPanel` using a `4x2 GridLayout` containing username input, masked password field, show password checkbox, login button, and remaining attempts tracking label.
- **Line 57–63**: `showPasswordCheckBox` listener toggles `passwordField.setEchoChar((char) 0)` to unmask password text when checked, or `setEchoChar('•')` to re-mask when unchecked.
- **Line 65**: Attaches `authenticateUser()` action listener to login button.
- **Line 72–74**: `authenticateUser()` extracts username and retrieves password as a `String` from `passwordField.getPassword()`.
- **Line 76–82**: Validates credentials against mock values (`"admin"` / `"admin123"`). If valid, displays success popup via `JOptionPane` and resets attempt count.
- **Line 83–94**: If invalid, decrements `remainingAttempts`. If attempts reach `0`, locks input fields (`setEnabled(false)`) and displays an account lockout alert.

---

## 7. E-Commerce Storefront & Shopping Cart

### `EcommerceShoppingCartGUI.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JComboBox;
import javax.swing.JSpinner;
import javax.swing.SpinnerNumberModel;
import javax.swing.JButton;
import javax.swing.JTable;
import javax.swing.table.DefaultTableModel;
import javax.swing.JScrollPane;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.Font;
import java.awt.Color;

public class EcommerceShoppingCartGUI extends JFrame {

    private JComboBox<String> productCombo;
    private JSpinner quantitySpinner;
    private DefaultTableModel cartTableModel;
    private JTable cartTable;
    private JLabel totalAmountLabel;
    private double grandTotal = 0.0;

    // Catalog Data: Names and Unit Prices
    private final String[] productNames = {"Wireless Headphones ($50.00)", "Gaming Mouse ($25.00)", "Mechanical Keyboard ($80.00)", "USB-C Hub ($30.00)"};
    private final double[] productPrices = {50.00, 25.00, 80.00, 30.00};

    public EcommerceShoppingCartGUI() {
        setTitle("E-Commerce Shopping Cart System");
        setSize(650, 420);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Top Catalog Selection Panel
        JPanel catalogPanel = new JPanel(new FlowLayout(FlowLayout.LEFT, 8, 8));
        catalogPanel.setBackground(new Color(241, 245, 249));

        catalogPanel.add(new JLabel("Product:"));
        productCombo = new JComboBox<>(productNames);
        catalogPanel.add(productCombo);

        catalogPanel.add(new JLabel("Qty:"));
        quantitySpinner = new JSpinner(new SpinnerNumberModel(1, 1, 10, 1));
        catalogPanel.add(quantitySpinner);

        JButton addToCartBtn = new JButton("Add to Cart");
        catalogPanel.add(addToCartBtn);

        // Center Shopping Cart JTable
        String[] cartColumns = {"Item Name", "Unit Price ($)", "Quantity", "Subtotal ($)"};
        cartTableModel = new DefaultTableModel(cartColumns, 0);
        cartTable = new JTable(cartTableModel);
        JScrollPane cartScroll = new JScrollPane(cartTable);

        // Bottom Action & Total Summary Panel
        JPanel summaryPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT, 15, 8));
        JButton removeItemBtn = new JButton("Remove Selected");
        JButton checkoutBtn = new JButton("Checkout");
        totalAmountLabel = new JLabel("Grand Total: $0.00");
        totalAmountLabel.setFont(new Font("SansSerif", Font.BOLD, 14));

        summaryPanel.add(removeItemBtn);
        summaryPanel.add(checkoutBtn);
        summaryPanel.add(totalAmountLabel);

        // Event Handlers
        addToCartBtn.addActionListener(e -> addItemToCart());
        removeItemBtn.addActionListener(e -> removeSelectedItem());
        checkoutBtn.addActionListener(e -> processCheckout());

        // Assembly
        setLayout(new BorderLayout(8, 8));
        add(catalogPanel, BorderLayout.NORTH);
        add(cartScroll, BorderLayout.CENTER);
        add(summaryPanel, BorderLayout.SOUTH);
    }

    private void addItemToCart() {
        int selectedIndex = productCombo.getSelectedIndex();
        String name = productNames[selectedIndex].split(" \\(")[0];
        double price = productPrices[selectedIndex];
        int quantity = (Integer) quantitySpinner.getValue();
        double subtotal = price * quantity;

        cartTableModel.addRow(new Object[]{name, String.format("%.2f", price), quantity, String.format("%.2f", subtotal)});
        recalculateTotal();
    }

    private void removeSelectedItem() {
        int selectedRow = cartTable.getSelectedRow();
        if (selectedRow != -1) {
            cartTableModel.removeRow(selectedRow);
            recalculateTotal();
        } else {
            JOptionPane.showMessageDialog(this, "Please select an item in the cart table to remove.", "Selection Notice", JOptionPane.WARNING_MESSAGE);
        }
    }

    private void recalculateTotal() {
        grandTotal = 0.0;
        for (int i = 0; i < cartTableModel.getRowCount(); i++) {
            double subtotal = Double.parseDouble((String) cartTableModel.getValueAt(i, 3));
            grandTotal += subtotal;
        }
        totalAmountLabel.setText(String.format("Grand Total: $%.2f", grandTotal));
    }

    private void processCheckout() {
        if (cartTableModel.getRowCount() == 0) {
            JOptionPane.showMessageDialog(this, "Your shopping cart is empty!", "Checkout Error", JOptionPane.WARNING_MESSAGE);
            return;
        }

        String receiptMsg = String.format("ORDER CONFIRMED!\nTotal Items: %d\nAmount Paid: $%.2f\nThank you for shopping with us!",
                cartTableModel.getRowCount(), grandTotal);

        JOptionPane.showMessageDialog(this, receiptMsg, "Checkout Success", JOptionPane.INFORMATION_MESSAGE);
        cartTableModel.setRowCount(0);
        recalculateTotal();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new EcommerceShoppingCartGUI().setVisible(true));
    }
}
```

### Explanation

- **Line 1–13**: Imports Swing controls (`JSpinner`, `SpinnerNumberModel`, `JTable`, `DefaultTableModel`, `JOptionPane`) and layout classes.
- **Line 15–24**: Declares class extending `JFrame` and initializes product catalog parallel arrays (`productNames` and `productPrices`).
- **Line 26–30**: Configures window title, size (650x420 pixels), exit behavior, and screen centering.
- **Line 32–43**: Constructs `catalogPanel` toolbar with product dropdown `JComboBox`, quantity spinner `JSpinner` (constrained between 1 and 10), and `"Add to Cart"` button.
- **Line 45–48**: Creates `cartTableModel` with 4 columns (`"Item Name"`, `"Unit Price ($)"`, `"Quantity"`, `"Subtotal ($)"`), binding it to `cartTable` inside `JScrollPane`.
- **Line 50–57**: Constructs `summaryPanel` containing `"Remove Selected"` button, `"Checkout"` button, and bold `totalAmountLabel`.
- **Line 59–62**: Connects action listeners to `addItemToCart()`, `removeSelectedItem()`, and `processCheckout()`.
- **Line 69–77**: `addItemToCart()` reads selected product details and quantity, calculates subtotal (`price * quantity`), adds a new row to `cartTableModel`, and calls `recalculateTotal()`.
- **Line 79–87**: `removeSelectedItem()` checks `cartTable.getSelectedRow()`, removes the highlighted row from `cartTableModel`, and recalculates totals.
- **Line 89–96**: `recalculateTotal()` iterates over all cart table rows, sums up subtotal column values, and updates `totalAmountLabel`.
- **Line 98–108**: `processCheckout()` verifies the cart is not empty, displays an order confirmation popup via `JOptionPane`, resets the cart table (`setRowCount(0)`), and clears total amounts.
