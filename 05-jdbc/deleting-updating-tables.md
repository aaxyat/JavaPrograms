# Unit 5: JDBC

## Deleting/Updating Tables

Modifying database records and table structures requires executing DML (`UPDATE`, `DELETE`) and DDL (`ALTER TABLE`, `DROP TABLE`) statements in JDBC. Using **`PreparedStatement`**, **Batch Operations**, and **Transaction Management** ensures data integrity and prevents security vulnerabilities.

---

### Core Concepts

#### 1. SQL Statement Types for Table Operations

```
                       JDBC Modification Operations
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
     DDL Operations                                    DML Operations
(Data Definition Language)                        (Data Manipulation Language)
  ├── ALTER TABLE (Modify schema)                   ├── UPDATE (Modify rows)
  └── DROP TABLE  (Delete table)                    └── DELETE (Remove rows)
```

#### 2. PreparedStatement vs. Statement

Always use `PreparedStatement` for DML updates and deletes containing dynamic user inputs:

```java
// VULNERABLE to SQL Injection:
String unsafeSql = "DELETE FROM users WHERE username = '" + userInput + "'";
stmt.executeUpdate(unsafeSql);

// SAFE using PreparedStatement:
String safeSql = "DELETE FROM users WHERE username = ?";
try (PreparedStatement pstmt = conn.prepareStatement(safeSql)) {
    pstmt.setString(1, userInput);
    pstmt.executeUpdate();
}
```

- **SQL Injection Defense**: Parameter placeholders (`?`) separate code logic from user data.
- **Pre-compilation Efficiency**: The database parses, compiles, and optimizes the SQL execution plan once, reusing it for repeated parameter bindings.

#### 3. Batch Update Operations (`executeBatch`)
Sending hundreds of individual `UPDATE` statements over the network creates heavy latency. **Batch Processing** groups multiple DML operations into a single network transmission:

```java
try (PreparedStatement pstmt = conn.prepareStatement("UPDATE accounts SET balance = balance + ? WHERE id = ?")) {
    for (Account acc : accountList) {
        pstmt.setDouble(1, acc.getInterest());
        pstmt.setInt(2, acc.getId());
        pstmt.addBatch(); // Queue update into batch buffer
    }
    int[] updateCounts = pstmt.executeBatch(); // Send all updates in 1 network call
}
```

#### 4. Transaction Atomicity (`commit` & `rollback`)
To guarantee ACID compliance during destructive modifications:
1. Call `conn.setAutoCommit(false)` to disable automatic execution saving.
2. Execute `UPDATE` or `DELETE` statements.
3. Call `conn.commit()` if all operations succeed, or `conn.rollback()` inside `catch` blocks if any step fails.

---

### Common Pitfalls

1. **Omitting `WHERE` Clauses in `UPDATE` or `DELETE` Statements**:
   - `UPDATE employees SET salary = 50000` updates *every single row* in the entire database table. Always verify that `WHERE` conditions are specified correctly.
2. **Forgetting `conn.rollback()` in Catch Blocks**:
   - Disabling auto-commit without calling `conn.rollback()` on exception occurrences leaves uncommitted locks open on the database server.
3. **Leaving Large Batches Un-flushed**:
   - Adding millions of items to `addBatch()` without periodically calling `executeBatch()` causes `OutOfMemoryError` in JVM memory. Call `executeBatch()` every 1,000 iterations for massive datasets.

---

## 1. Demo: Parameterized Updates, Deletes, and Transaction Control

### `TableUpdateDeleteDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Demonstrates PreparedStatement UPDATE/DELETE queries, batch processing, and transactional rollback.
 */
public class TableUpdateDeleteDemo {

    private static final String DB_URL = "jdbc:h2:mem:update_demo_db;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) {
        
        try (Connection conn = DriverManager.getConnection(DB_URL, "sa", "");
             Statement stmt = conn.createStatement()) {

            // 1. Create Schema and Seed Initial Records
            stmt.execute("CREATE TABLE clients (id INT PRIMARY KEY, name VARCHAR(50), status VARCHAR(20), balance DOUBLE)");
            stmt.executeUpdate("INSERT INTO clients VALUES (1, 'Alpha Corp', 'ACTIVE', 10000.00)");
            stmt.executeUpdate("INSERT INTO clients VALUES (2, 'Beta Tech', 'PENDING', 4500.00)");
            stmt.executeUpdate("INSERT INTO clients VALUES (3, 'Gamma LLC', 'PENDING', 0.00)");

            System.out.println("=== INITIAL CLIENT RECORDS ===");
            printClients(stmt);

            // 2. Execute Parameterized UPDATE using PreparedStatement
            System.out.println("\n=== 1. PARAMETERIZED UPDATE OPERATION ===");
            String updateSql = "UPDATE clients SET status = ?, balance = balance + ? WHERE id = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(updateSql)) {
                pstmt.setString(1, "ACTIVE");
                pstmt.setDouble(2, 500.00);
                pstmt.setInt(3, 2);

                int rowsAffected = pstmt.executeUpdate();
                System.out.println("Updated Client #2 Status to ACTIVE (Rows affected: " + rowsAffected + ")");
            }
            printClients(stmt);

            // 3. Execute Parameterized DELETE using PreparedStatement
            System.out.println("\n=== 2. PARAMETERIZED DELETE OPERATION ===");
            String deleteSql = "DELETE FROM clients WHERE balance = ? AND status = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(deleteSql)) {
                pstmt.setDouble(1, 0.00);
                pstmt.setString(2, "PENDING");

                int rowsDeleted = pstmt.executeUpdate();
                System.out.println("Deleted Pending Clients with Zero Balance (Rows deleted: " + rowsDeleted + ")");
            }
            printClients(stmt);

            // 4. Demonstrate Transaction Rollback on Error
            System.out.println("\n=== 3. TRANSACTION ROLLBACK SIMULATION ===");
            conn.setAutoCommit(false); // Begin transaction block

            try (PreparedStatement updatePstmt = conn.prepareStatement("UPDATE clients SET balance = balance - ? WHERE id = ?")) {
                updatePstmt.setDouble(1, 5000.00);
                updatePstmt.setInt(2, 1);
                updatePstmt.executeUpdate();

                // Simulate unexpected failure before commit
                boolean errorEncountered = true;
                if (errorEncountered) {
                    throw new SQLException("Simulated network outage during transfer!");
                }

                conn.commit();
            } catch (SQLException ex) {
                System.err.println("Transaction Failed: " + ex.getMessage() + " -> Rolling back changes...");
                conn.rollback(); // Rollback transaction state
            } finally {
                conn.setAutoCommit(true); // Restore default mode
            }

            System.out.println("Records After Rollback (Original Balance Preserved):");
            printClients(stmt);

        } catch (SQLException e) {
            System.err.println("Database Modification Exception: " + e.getMessage());
        }
    }

    private static void printClients(Statement stmt) throws SQLException {
        try (ResultSet rs = stmt.executeQuery("SELECT id, name, status, balance FROM clients")) {
            System.out.println("---------------------------------------------------------");
            while (rs.next()) {
                System.out.printf("ID: %d | Name: %-15s | Status: %-10s | Balance: $%.2f%n",
                        rs.getInt("id"), rs.getString("name"), rs.getString("status"), rs.getDouble("balance"));
            }
            System.out.println("---------------------------------------------------------");
        }
    }
}
```

### Explanation

1. **`PreparedStatement` Security & Binding**:
   - `pstmt.setString()`, `pstmt.setDouble()`, and `pstmt.setInt()` bind positional parameters (`?`) safely before executing `.executeUpdate()`.

2. **Transaction Rollback (`conn.rollback()`)**:
   - When an exception occurs after `setAutoCommit(false)`, `conn.rollback()` restores original database state before uncommitted modifications affect persistent records.

---

## 2. Practical Program: Enterprise Account Maintenance System

### `EnterpriseAccountManager.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Enterprise maintenance manager performing batch status updates, soft deletes, and DDL table drops.
 */
public class EnterpriseAccountManager {

    private static final String DB_URL = "jdbc:h2:mem:account_mgr_db;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(DB_URL, "sa", "");
             Statement stmt = conn.createStatement()) {

            // 1. Initialize Account Schema
            stmt.execute("CREATE TABLE user_accounts (" +
                    "account_id INT PRIMARY KEY, " +
                    "owner_name VARCHAR(50), " +
                    "account_type VARCHAR(20), " +
                    "is_active BOOLEAN)");

            stmt.executeUpdate("INSERT INTO user_accounts VALUES (1001, 'David Miller', 'SAVINGS', true)");
            stmt.executeUpdate("INSERT INTO user_accounts VALUES (1002, 'Emma Wilson', 'CHECKING', true)");
            stmt.executeUpdate("INSERT INTO user_accounts VALUES (1003, 'Frank Thomas', 'SAVINGS', true)");

            System.out.println("==========================================");
            System.out.println("     ENTERPRISE ACCOUNT MAINTENANCE      ");
            System.out.println("==========================================");

            // 2. Batch Update Accounts
            String batchSql = "UPDATE user_accounts SET account_type = ? WHERE account_id = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(batchSql)) {
                
                pstmt.setString(1, "PREMIUM_SAVINGS");
                pstmt.setInt(2, 1001);
                pstmt.addBatch();

                pstmt.setString(1, "PREMIUM_CHECKING");
                pstmt.setInt(2, 1002);
                pstmt.addBatch();

                int[] batchResults = pstmt.executeBatch();
                System.out.println("Executed Batch Update (Total statements: " + batchResults.length + ")");
            }

            // 3. Soft Delete Operation (Set is_active = false)
            String softDeleteSql = "UPDATE user_accounts SET is_active = false WHERE account_id = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(softDeleteSql)) {
                pstmt.setInt(1, 1003);
                int softDeletedRows = pstmt.executeUpdate();
                System.out.println("Soft-deleted Account #1003 (Rows deactivated: " + softDeletedRows + ")");
            }

            // Print Active Accounts
            printActiveAccounts(stmt);

            // 4. DDL DROP TABLE Operation
            System.out.println("\n=== PURGING TABLE SCHEMA (DROP TABLE) ===");
            stmt.execute("DROP TABLE user_accounts");
            System.out.println("Table 'user_accounts' dropped successfully.");

            System.out.println("==========================================");

        } catch (SQLException e) {
            System.err.println("Account Management Error: " + e.getMessage());
        }
    }

    private static void printActiveAccounts(Statement stmt) throws SQLException {
        System.out.println("\n=== CURRENT ACTIVE ACCOUNTS ===");
        try (ResultSet rs = stmt.executeQuery("SELECT * FROM user_accounts WHERE is_active = true")) {
            while (rs.next()) {
                System.out.printf("ID: %d | Owner: %-15s | Type: %-18s | Active: %b%n",
                        rs.getInt("account_id"), rs.getString("owner_name"),
                        rs.getString("account_type"), rs.getBoolean("is_active"));
            }
        }
    }
}
```

### Explanation

1. **Batch Execution (`executeBatch()`)**:
   - Accumulates multiple parameterized update statements using `addBatch()`, executing them in a single database transmission.

2. **Soft Deletes vs. DDL Drops**:
   - Demonstrates soft deletion (`is_active = false`) for safe audit trails alongside structural table purging (`DROP TABLE`).
