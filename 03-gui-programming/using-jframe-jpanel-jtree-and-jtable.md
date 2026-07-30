# Unit 3: GUI Programming

## Using JFrame, JPanel, JTree, and JTable

Complex Java Swing desktop applications combine top-level window containers (`JFrame`), intermediate layout organizers (`JPanel`), hierarchical tree structures (`JTree`), and multi-column data grids (`JTable`).

---

### Core Concepts

#### 1. Top-Level Window Container (`JFrame`)
- `JFrame` serves as the main window host for a desktop application.
- It includes window decorations (title bar, minimize, maximize, and close buttons) and manages a content pane (`getContentPane()`) where panels and components are attached.

#### 2. Layout Organizer Panel (`JPanel`)
- `JPanel` is an invisible lightweight container used to group, align, and nest components.
- Panels can hold distinct layout managers (`FlowLayout`, `BorderLayout`, `GridLayout`) to organize sections of a window independently.

#### 3. Hierarchical Tree Control (`JTree`)
- `JTree` presents data visually in expandable parent-child node hierarchies (resembling file system directories).
- **Node Model**: Built using `DefaultMutableTreeNode` instances connected in a tree structure:
  ```java
  DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
  DefaultMutableTreeNode child = new DefaultMutableTreeNode("Child Node");
  root.add(child);
  JTree tree = new JTree(root);
  ```

#### 4. Tabular Data Grid Control (`JTable`)
- `JTable` displays two-dimensional tabular data in rows and columns.
- **Table Data & Headers**: Can be instantiated directly with arrays or managed dynamically using `DefaultTableModel`:
  ```java
  String[] columns = {"ID", "Name", "Role"};
  Object[][] data = { {1, "Alice", "Admin"}, {2, "Bob", "User"} };
  JTable table = new JTable(data, columns);
  ```
- **Crucial Requirement**: `JTable` **must** be placed inside a `JScrollPane` (`new JScrollPane(table)`). The `JScrollPane` automatically extracts and displays column headers at the top of the table.

---

### Common Pitfalls

1. **Adding `JTable` directly to a `JPanel` without `JScrollPane`**:
   - Adding `panel.add(table)` directly renders only table data cells, hiding column header titles completely. Always wrap tables: `panel.add(new JScrollPane(table))`.
2. **Mutating `JTree` nodes without updating `TreeModel`**:
   - Adding or removing nodes directly on a `DefaultMutableTreeNode` after the tree is visible does not refresh the screen automatically. You must call `((DefaultTreeModel) tree.getModel()).reload()` to trigger a visual redraw.
3. **Overwriting Content Pane Layouts**:
   - Replacing a `JFrame` layout without managing panels leads to overlapping components. Use nested `JPanel` containers to isolate distinct layout regions cleanly.

---

## 1. Demo: Structural Containers and Complex Data Controls

### `ContainersAndDataControlsDemo.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JTree;
import javax.swing.JTable;
import javax.swing.JScrollPane;
import javax.swing.JSplitPane;
import javax.swing.JLabel;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.Font;
import java.awt.Color;

/**
 * Demonstrates combining JFrame, JPanel, JTree, and JTable controls within a split-pane layout.
 */
public class ContainersAndDataControlsDemo extends JFrame {

    public ContainersAndDataControlsDemo() {
        // 1. Configure Main JFrame Window
        setTitle("JFrame, JPanel, JTree & JTable Integration Demo");
        setSize(700, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // 2. Create Header JPanel
        JPanel headerPanel = new JPanel();
        headerPanel.setBackground(new Color(30, 41, 59));
        JLabel headerLabel = new JLabel("ENTERPRISE SYSTEM EXPLORER");
        headerLabel.setFont(new Font("SansSerif", Font.BOLD, 16));
        headerLabel.setForeground(Color.WHITE);
        headerPanel.add(headerLabel);

        // 3. Construct JTree (Left View)
        DefaultMutableTreeNode companyRoot = new DefaultMutableTreeNode("Company Organization");
        
        DefaultMutableTreeNode engDept = new DefaultMutableTreeNode("Engineering");
        engDept.add(new DefaultMutableTreeNode("Software Team"));
        engDept.add(new DefaultMutableTreeNode("QA Testing Team"));

        DefaultMutableTreeNode hrDept = new DefaultMutableTreeNode("Human Resources");
        hrDept.add(new DefaultMutableTreeNode("Recruiting"));
        
        companyRoot.add(engDept);
        companyRoot.add(hrDept);

        JTree orgTree = new JTree(companyRoot);
        JScrollPane treeScrollPane = new JScrollPane(orgTree);

        // 4. Construct JTable (Right View)
        String[] columnHeaders = {"Emp ID", "Full Name", "Department", "Status"};
        Object[][] employeeData = {
            {1001, "Sarah Jenkins", "Software Team", "Active"},
            {1002, "Michael Chang", "Software Team", "On Leave"},
            {1003, "David Ross", "QA Testing Team", "Active"},
            {1004, "Emma Watson", "Human Resources", "Active"}
        };
        JTable employeeTable = new JTable(employeeData, columnHeaders);
        JScrollPane tableScrollPane = new JScrollPane(employeeTable);

        // 5. Use JSplitPane to Separate Tree and Table Views
        JSplitPane splitPane = new JSplitPane(JSplitPane.HORIZONTAL_SPLIT, treeScrollPane, tableScrollPane);
        splitPane.setDividerLocation(220);

        // 6. Assemble Components inside Main JFrame Content Pane
        setLayout(new BorderLayout());
        add(headerPanel, BorderLayout.NORTH);
        add(splitPane, BorderLayout.CENTER);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new ContainersAndDataControlsDemo().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Top-Level `JFrame` Setup**:
   - `setTitle()`, `setSize()`, and `setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)` establish window dimensions and process termination handling.

2. **Organizing Headers with `JPanel`**:
   - `headerPanel` acts as an intermediate container styling the header background dark blue (`Color(30, 41, 59)`).

3. **Constructing `JTree`**:
   - Uses `DefaultMutableTreeNode` to construct a hierarchical organizational chart. `JScrollPane(orgTree)` ensures vertical scrollbars appear if nodes expand beyond window height.

4. **Constructing `JTable`**:
   - Instantiates `employeeTable` with 2D data arrays and column headers. Wrapping inside `JScrollPane(employeeTable)` displays table column titles ("Emp ID", "Full Name", "Department", "Status").

---

## 2. Practical Program: Enterprise Asset Management System

### `EnterpriseAssetManager.java`

```java
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JTree;
import javax.swing.JTable;
import javax.swing.JScrollPane;
import javax.swing.JSplitPane;
import javax.swing.JLabel;
import javax.swing.BorderFactory;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.TreePath;
import javax.swing.table.DefaultTableModel;
import javax.swing.event.TreeSelectionListener;
import javax.swing.event.TreeSelectionEvent;
import javax.swing.SwingUtilities;
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;

/**
 * Enterprise IT asset manager mapping JTree department selection dynamically to a JTable asset list.
 */
public class EnterpriseAssetManager extends JFrame {

    private JTree departmentTree;
    private JTable assetTable;
    private DefaultTableModel tableModel;
    private JLabel statusLabel;

    public EnterpriseAssetManager() {
        setTitle("Enterprise IT Asset Manager");
        setSize(780, 450);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Header Banner
        JPanel bannerPanel = new JPanel(new BorderLayout());
        bannerPanel.setBackground(new Color(15, 23, 42));
        bannerPanel.setBorder(BorderFactory.createEmptyBorder(10, 15, 10, 15));

        JLabel titleLabel = new JLabel("IT HARDWARE & ASSET INVENTORY");
        titleLabel.setFont(new Font("SansSerif", Font.BOLD, 16));
        titleLabel.setForeground(Color.WHITE);
        bannerPanel.add(titleLabel, BorderLayout.WEST);

        // Left Component: JTree Department Hierarchy
        DefaultMutableTreeNode root = new DefaultMutableTreeNode("All Enterprise Assets");
        DefaultMutableTreeNode serverDept = new DefaultMutableTreeNode("Data Center Servers");
        DefaultMutableTreeNode workstationDept = new DefaultMutableTreeNode("Office Workstations");
        DefaultMutableTreeNode mobileDept = new DefaultMutableTreeNode("Mobile Devices");

        root.add(serverDept);
        root.add(workstationDept);
        root.add(mobileDept);

        departmentTree = new JTree(root);
        JScrollPane treeView = new JScrollPane(departmentTree);
        treeView.setBorder(BorderFactory.createTitledBorder("Departments"));

        // Right Component: JTable Asset Grid
        String[] columns = {"Asset Tag", "Device Name", "Assigned User", "Value ($)"};
        tableModel = new DefaultTableModel(columns, 0);
        assetTable = new JTable(tableModel);
        JScrollPane tableView = new JScrollPane(assetTable);
        tableView.setBorder(BorderFactory.createTitledBorder("Inventory Grid"));

        // Split Pane Container
        JSplitPane splitPane = new JSplitPane(JSplitPane.HORIZONTAL_SPLIT, treeView, tableView);
        splitPane.setDividerLocation(240);

        // Bottom Status Bar Panel
        JPanel statusPanel = new JPanel(new BorderLayout());
        statusPanel.setBorder(BorderFactory.createEtchedBorder());
        statusLabel = new JLabel(" Select a department node on the left tree to filter asset records.");
        statusPanel.add(statusLabel, BorderLayout.WEST);

        // Register JTree Selection Event Listener
        departmentTree.addTreeSelectionListener(new TreeSelectionListener() {
            @Override
            public void valueChanged(TreeSelectionEvent e) {
                TreePath path = e.getPath();
                String selectedNode = path.getLastPathComponent().toString();
                updateAssetTable(selectedNode);
            }
        });

        // Populate Default Initial Assets
        updateAssetTable("All Enterprise Assets");

        // Layout Assembly
        setLayout(new BorderLayout());
        add(bannerPanel, BorderLayout.NORTH);
        add(splitPane, BorderLayout.CENTER);
        add(statusPanel, BorderLayout.SOUTH);
    }

    private void updateAssetTable(String nodeName) {
        tableModel.setRowCount(0); // Clear current table rows

        if ("Data Center Servers".equals(nodeName) || "All Enterprise Assets".equals(nodeName)) {
            tableModel.addRow(new Object[]{"AST-9012", "Dell PowerEdge R750", "Server Rack A1", "8,500.00"});
            tableModel.addRow(new Object[]{"AST-9013", "HP ProLiant DL380", "Server Rack A2", "7,200.00"});
        }

        if ("Office Workstations".equals(nodeName) || "All Enterprise Assets".equals(nodeName)) {
            tableModel.addRow(new Object[]{"AST-4051", "ThinkPad P16 Workstation", "John Doe (Eng)", "2,400.00"});
            tableModel.addRow(new Object[]{"AST-4052", "MacStudio M2 Ultra", "Alice Smith (Design)", "3,900.00"});
        }

        if ("Mobile Devices".equals(nodeName) || "All Enterprise Assets".equals(nodeName)) {
            tableModel.addRow(new Object[]{"AST-1102", "iPad Pro 12.9\"", "Sales Team", "1,100.00"});
        }

        statusLabel.setText(" Displaying " + tableModel.getRowCount() + " asset records for category: [" + nodeName + "]");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new EnterpriseAssetManager().setVisible(true);
            }
        });
    }
}
```

### Explanation

1. **Dynamic Data Table (`DefaultTableModel`)**:
   - Uses `DefaultTableModel` to manage table rows dynamically. `tableModel.setRowCount(0)` clears existing rows, and `tableModel.addRow(...)` inserts new filtered asset rows.

2. **JTree Selection Listener (`addTreeSelectionListener`)**:
   - `valueChanged(TreeSelectionEvent e)` intercepts node selection events when users click on tree nodes, dynamically updating `assetTable` records.
