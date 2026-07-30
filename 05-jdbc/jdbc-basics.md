# Unit 5: JDBC

## JDBC Basics

Core JDBC operations center around four fundamental interfaces in `java.sql`: `DriverManager`, `Connection`, `Statement`, and `ResultSet`. Mastering these interfaces enables Java applications to perform complete CRUD (Create, Read, Update, Delete) database operations.

---

### Core Concepts

#### 1. Fundamental JDBC Objects & Workflow

```
   DriverManager.getConnection(url, user, pass)
                      │
                      ▼
                 Connection  (Manages DB session & transactions)
                      │
                      ▼
                 Statement   (Sends SQL queries to DB)
         ┌────────────┴────────────┐
         │                         │
         ▼                         ▼
  executeQuery(sql)         executeUpdate(sql)
  (For SELECT)              (For INSERT/UPDATE/DELETE)
         │                         │
         ▼                         ▼
     ResultSet                  int count
 (Iterate via rs.next())     (Number of affected rows)
```

#### 2. Key JDBC Interfaces & Method Summary

| Interface | Primary Responsibility | Essential Methods |
|-----------|------------------------|-------------------|
| **`DriverManager`** | Loads database drivers and creates active connections. | `getConnection(url, user, pass)` |
| **`Connection`** | Represents a active connection session with a database. | `createStatement()`, `prepareStatement()`, `close()`, `setAutoCommit()` |
| **`Statement`** | Executes static SQL queries against the database. | `executeQuery(sql)`, `executeUpdate(sql)`, `execute(sql)` |
| **`ResultSet`** | Tabular data grid returned by a `SELECT` query. | `next()`, `getString()`, `getInt()`, `getDouble()`, `wasNull()` |

#### 3. Standard Database Connection URL Formats

- **H2 (In-Memory)**: `jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1`
- **MySQL**: `jdbc:mysql://localhost:3306/dbname?useSSL=false`
- **PostgreSQL**: `jdbc:postgresql://localhost:5432/dbname`
- **SQLite**: `jdbc:sqlite:sample.db`

---

### Common Pitfalls

1. **Forgetting `rs.next()` Before Reading Data**:
   - A newly returned `ResultSet` cursor points *before* the first row. Calling `rs.getString(1)` without first invoking `rs.next()` throws a `SQLException: Before start of result set`.
2. **1-Based Column Indexing Mismatches**:
   - Unlike Java array indices which start at `0`, JDBC `ResultSet` column indices start at **`1`** (`rs.getString(1)` reads the first column).
3. **SQL Injection Vulnerabilities with `Statement`**:
   - Concatenating user inputs directly into raw SQL strings (`"SELECT * FROM users WHERE name = '" + name + "'"` Enables SQL injection attacks. Use `PreparedStatement` when handling dynamic user inputs.

---

## 1. Demo: Complete CRUD Workflow with Statement and ResultSet

### `JdbcCrudBasicsDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Demonstrates complete CRUD (Create, Read, Update, Delete) database operations using Statement and ResultSet.
 */
public class JdbcCrudBasicsDemo {

    private static final String DB_URL = "jdbc:h2:mem:crud_db;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void main(String[] args) {
        
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS);
             Statement stmt = conn.createStatement()) {

            // 1. CREATE TABLE
            System.out.println("=== 1. CREATE TABLE ===");
            stmt.execute("CREATE TABLE employees (id INT PRIMARY KEY, name VARCHAR(50), salary DOUBLE)");
            System.out.println("Created 'employees' table successfully.");

            // 2. INSERT (Create)
            System.out.println("\n=== 2. INSERT RECORDS ===");
            int rowsAdded = 0;
            rowsAdded += stmt.executeUpdate("INSERT INTO employees VALUES (1, 'Alice Smith', 75000.00)");
            rowsAdded += stmt.executeUpdate("INSERT INTO employees VALUES (2, 'Bob Jones', 62000.00)");
            rowsAdded += stmt.executeUpdate("INSERT INTO employees VALUES (3, 'Charlie Brown', 58000.00)");
            System.out.println("Total employee records inserted: " + rowsAdded);

            // 3. SELECT (Read)
            System.out.println("\n=== 3. READ RECORDS (SELECT) ===");
            printEmployees(stmt);

            // 4. UPDATE (Modify)
            System.out.println("\n=== 4. UPDATE RECORD ===");
            int rowsUpdated = stmt.executeUpdate("UPDATE employees SET salary = 80000.00 WHERE id = 1");
            System.out.println("Updated salary for employee #1 (Rows affected: " + rowsUpdated + ")");
            printEmployees(stmt);

            // 5. DELETE (Remove)
            System.out.println("\n=== 5. DELETE RECORD ===");
            int rowsDeleted = stmt.executeUpdate("DELETE FROM employees WHERE id = 3");
            System.out.println("Deleted employee record #3 (Rows affected: " + rowsDeleted + ")");
            printEmployees(stmt);

        } catch (SQLException e) {
            System.err.println("Database CRUD Exception: " + e.getMessage());
        }
    }

    private static void printEmployees(Statement stmt) throws SQLException {
        String query = "SELECT id, name, salary FROM employees";
        try (ResultSet rs = stmt.executeQuery(query)) {
            System.out.println("------------------------------------------");
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                double salary = rs.getDouble("salary");
                System.out.printf("ID: %d | Name: %-15s | Salary: $%.2f%n", id, name, salary);
            }
            System.out.println("------------------------------------------");
        }
    }
}
```

### Explanation

1. **`Statement.executeUpdate()` for Modifications**:
   - Handles DDL (`CREATE TABLE`) and DML (`INSERT`, `UPDATE`, `DELETE`), returning the count of affected database rows.

2. **`Statement.executeQuery()` for Data Retrieval**:
   - Executes `SELECT` statements, returning a `ResultSet` cursor that iterates over matching records via `rs.next()`.

---

## 2. Practical Program: Retail Inventory Management Database Tool

### `InventoryDatabaseManager.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Retail Store Inventory Database Manager demonstrating stock updates, low-stock threshold queries, and item removal.
 */
public class InventoryDatabaseManager {

    private static final String DB_URL = "jdbc:h2:mem:inventory_db;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void initializeInventoryDatabase(Connection conn) throws SQLException {
        try (Statement stmt = conn.createStatement()) {
            stmt.execute("CREATE TABLE inventory (" +
                    "item_id INT PRIMARY KEY, " +
                    "item_name VARCHAR(60), " +
                    "unit_price DOUBLE, " +
                    "stock_quantity INT)");
            
            stmt.executeUpdate("INSERT INTO inventory VALUES (501, 'Wireless Mouse', 25.50, 45)");
            stmt.executeUpdate("INSERT INTO inventory VALUES (502, 'Mechanical Keyboard', 85.00, 8)");
            stmt.executeUpdate("INSERT INTO inventory VALUES (503, 'USB-C Cable', 12.00, 3)");
            stmt.executeUpdate("INSERT INTO inventory VALUES (504, '27-Inch Monitor', 299.99, 12)");
        }
    }

    public static void displayInventory(Connection conn, String headerTitle, String sqlQuery) throws SQLException {
        System.out.println("\n=== " + headerTitle + " ===");
        System.out.println("---------------------------------------------------------");
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sqlQuery)) {
            
            while (rs.next()) {
                int id = rs.getInt("item_id");
                String name = rs.getString("item_name");
                double price = rs.getDouble("unit_price");
                int stock = rs.getInt("stock_quantity");

                System.out.printf("Item #%d | Name: %-20s | Price: $%6.2f | Stock: %d%n",
                        id, name, price, stock);
            }
        }
        System.out.println("---------------------------------------------------------");
    }

    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS)) {

            // Initialize database schema and data
            initializeInventoryDatabase(conn);
            displayInventory(conn, "INITIAL INVENTORY CATALOG", "SELECT * FROM inventory");

            // Query low-stock alert items (stock < 10)
            displayInventory(conn, "LOW-STOCK ALERT REPORT (Stock < 10)", "SELECT * FROM inventory WHERE stock_quantity < 10");

            // Restock items (Update stock count)
            try (Statement stmt = conn.createStatement()) {
                int updatedRows = stmt.executeUpdate("UPDATE inventory SET stock_quantity = stock_quantity + 20 WHERE item_id = 503");
                System.out.println("\n[RESTOCK] Added 20 units to USB-C Cable (Rows modified: " + updatedRows + ")");
            }

            // Display updated inventory
            displayInventory(conn, "INVENTORY CATALOG AFTER RESTOCK", "SELECT * FROM inventory");

        } catch (SQLException e) {
            System.err.println("Inventory Database Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Structured Database Operations**:
   - `initializeInventoryDatabase()` creates tables and populates sample inventory records.

2. **Reusable Query Helper (`displayInventory`)**:
   - Takes SQL query strings dynamically, executing queries and rendering clean tabular output.
