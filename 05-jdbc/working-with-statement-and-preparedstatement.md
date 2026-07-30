# Unit 5: JDBC

## Working with Statement and PreparedStatement

JDBC provides two primary interfaces for sending SQL commands to a database: **`Statement`** for static SQL queries and **`PreparedStatement`** for parameterized SQL queries. Understanding their architectural differences is essential for writing secure, high-performance database applications.

---

### Core Concepts

#### 1. Architectural Comparison

```
  Statement Architecture (Static & Un-compiled)
  ┌──────────┐   Raw SQL String with Concatenated Data    ┌──────────────────┐
  │ Java App │ ─────────────────────────────────────────> │ Database Engine  │
  └──────────┘                                            │ 1. Parse SQL     │
                                                          │ 2. Compile Plan  │
                                                          │ 3. Execute Query │
                                                          └──────────────────┘

  PreparedStatement Architecture (Pre-compiled with Placeholders)
  ┌──────────┐   1. Prepare SQL Template ("SELECT * FROM users WHERE id = ?")
  │          │ ─────────────────────────────────────────> ┌──────────────────┐
  │ Java App │                                            │ Database Engine  │
  │          │   2. Send Only Parameter Data (id = 101)   │ (Compiles Once & │
  │          │ ─────────────────────────────────────────> │ Caches Plan)     │
  └──────────┘                                            └──────────────────┘
```

#### 2. Detailed Technical Comparison Table

| Feature / Aspect | `Statement` | `PreparedStatement` |
|------------------|-------------|---------------------|
| **SQL Type** | Static SQL strings without parameters. | Dynamic SQL queries with positional `?` placeholders. |
| **Compilation** | Parsed and compiled by RDBMS **every time** it executes. | Parsed and compiled **once** by RDBMS; plan cached for repeated executions. |
| **SQL Injection Security** | **Vulnerable**: Susceptible to malicious SQL injection attacks if input is concatenated. | **Immune**: Parameters are passed separately and treated strictly as literal data. |
| **Performance** | Slower for repetitive execution due to repeated query compilation. | Faster for repetitive execution due to cached database query plans. |
| **Type Safety** | Requires manual string formatting and quotes around text/dates. | Provides type-safe setters (`setInt()`, `setString()`, `setDouble()`, `setDate()`). |

#### 3. Essential `PreparedStatement` Setter Methods

| Method | Target SQL Column Type | Example Usage |
|--------|------------------------|---------------|
| `setInt(index, val)` | `INT`, `INTEGER` | `pstmt.setInt(1, 101)` |
| `setString(index, val)` | `VARCHAR`, `TEXT` | `pstmt.setString(2, "Alice")` |
| `setDouble(index, val)` | `DOUBLE`, `DECIMAL` | `pstmt.setDouble(3, 99.50)` |
| `setBoolean(index, val)`| `BOOLEAN`, `BIT` | `pstmt.setBoolean(4, true)` |
| `setDate(index, val)` | `DATE` | `pstmt.setDate(5, java.sql.Date.valueOf("2026-07-30"))` |
| `setNull(index, sqlType)`| Any SQL `NULL` | `pstmt.setNull(6, java.sql.Types.VARCHAR)` |

---

### Common Pitfalls

1. **1-Based Positional Parameter Indexing**:
   - Parameter placeholders (`?`) in a `PreparedStatement` are indexed starting at **`1`**, not `0`. Calling `pstmt.setString(0, "value")` throws a `SQLException: Parameter index out of bounds`.
2. **Re-instantiating `PreparedStatement` Inside Loops**:
   - Creating a new `PreparedStatement` instance inside a `for` loop defeats the pre-compilation performance gain. Create the `PreparedStatement` once outside the loop and update parameters inside the loop body.
3. **Mismatched Setter Types**:
   - Calling `pstmt.setInt(1, "101")` or calling `setString()` on a date column without proper formatting causes type cast exceptions from the database driver.

---

## 1. Demo: SQL Injection Comparison and PreparedStatement Performance

### `StatementVsPreparedStatementDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Demonstrates SQL Injection vulnerability in Statement vs immunity in PreparedStatement, plus performance benchmarking.
 */
public class StatementVsPreparedStatementDemo {

    private static final String DB_URL = "jdbc:h2:mem:pstmt_demo_db;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) {
        
        try (Connection conn = DriverManager.getConnection(DB_URL, "sa", "");
             Statement stmt = conn.createStatement()) {

            // 1. Setup User Credentials Table
            stmt.execute("CREATE TABLE users (id INT PRIMARY KEY, username VARCHAR(50), password VARCHAR(50))");
            stmt.executeUpdate("INSERT INTO users VALUES (1, 'admin', 'SuperSecretPass')");
            stmt.executeUpdate("INSERT INTO users VALUES (2, 'john_doe', 'Password123')");

            System.out.println("=== 1. DEMONSTRATING SQL INJECTION IN RAW STATEMENT ===");
            
            // Malicious input designed to bypass authentication (' OR '1'='1)
            String maliciousInput = "' OR '1'='1";
            String unsafeSql = "SELECT * FROM users WHERE username = 'admin' AND password = '" + maliciousInput + "'";
            System.out.println("Executing Unsafe Query: " + unsafeSql);

            try (ResultSet rs = stmt.executeQuery(unsafeSql)) {
                if (rs.next()) {
                    System.out.println("ALERT: SQL INJECTION SUCCESSFUL! Bypassed auth as user: " + rs.getString("username"));
                }
            }

            // 2. Demonstrating Immunity with PreparedStatement
            System.out.println("\n=== 2. DEMONSTRATING PREPAREDSTATEMENT IMMUNITY ===");
            String safeSql = "SELECT * FROM users WHERE username = ? AND password = ?";
            
            try (PreparedStatement pstmt = conn.prepareStatement(safeSql)) {
                pstmt.setString(1, "admin");
                pstmt.setString(2, maliciousInput); // Passed as literal parameter

                try (ResultSet rs = pstmt.executeQuery()) {
                    if (rs.next()) {
                        System.out.println("Authenticated successfully.");
                    } else {
                        System.out.println("SAFE: Malicious input treated as literal password string. Authentication failed as expected!");
                    }
                }
            }

            // 3. Performance Benchmark Across 2,000 Iterations
            System.out.println("\n=== 3. REPETITIVE EXECUTION PERFORMANCE BENCHMARK ===");
            stmt.execute("CREATE TABLE metrics (id INT PRIMARY KEY, val INT)");

            // Benchmark Statement Execution Speed
            long startStmt = System.currentTimeMillis();
            for (int i = 1; i <= 1000; i++) {
                stmt.executeUpdate("INSERT INTO metrics VALUES (" + i + ", " + (i * 10) + ")");
            }
            long timeStmt = System.currentTimeMillis() - startStmt;
            System.out.println("1,000 Statement Single Inserts        : " + timeStmt + " ms");

            // Benchmark PreparedStatement Execution Speed
            String insertPstmtSql = "INSERT INTO metrics VALUES (?, ?)";
            long startPstmt = System.currentTimeMillis();
            try (PreparedStatement pstmt = conn.prepareStatement(insertPstmtSql)) {
                for (int i = 1001; i <= 2000; i++) {
                    pstmt.setInt(1, i);
                    pstmt.setInt(2, i * 10);
                    pstmt.executeUpdate();
                }
            }
            long timePstmt = System.currentTimeMillis() - startPstmt;
            System.out.println("1,000 PreparedStatement Single Inserts: " + timePstmt + " ms");

        } catch (SQLException e) {
            System.err.println("JDBC Exception: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **SQL Injection Defense**:
   - In raw `Statement`, concatenating `' OR '1'='1` alters the SQL syntax tree. In `PreparedStatement`, positional parameters (`?`) force the engine to treat the string strictly as literal password data.

2. **Pre-compilation Reuse**:
   - Reusing a single `PreparedStatement` across 1,000 iterations bypasses SQL parsing overhead, accelerating database modification speed.

---

## 2. Practical Program: Secure Banking Authentication & Transaction Gateway

### `SecureUserAuthenticationPortal.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Secure banking authentication and transaction management portal powered by PreparedStatement queries.
 */
public class SecureUserAuthenticationPortal {

    private static final String DB_URL = "jdbc:h2:mem:bank_auth_db;DB_CLOSE_DELAY=-1";

    public static boolean authenticateUser(Connection conn, String username, String inputPassword) {
        String authSql = "SELECT user_id, is_locked FROM bank_users WHERE username = ? AND password_hash = ?";
        
        try (PreparedStatement pstmt = conn.prepareStatement(authSql)) {
            pstmt.setString(1, username);
            pstmt.setString(2, inputPassword);

            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    boolean isLocked = rs.getBoolean("is_locked");
                    if (isLocked) {
                        System.out.println("AUTH FAILED: Account " + username + " is locked for security.");
                        return false;
                    }
                    System.out.println("AUTH SUCCESS: Welcome, " + username + " (User ID: " + rs.getInt("user_id") + ")");
                    return true;
                }
            }
        } catch (SQLException e) {
            System.err.println("Authentication SQL Error: " + e.getMessage());
        }
        System.out.println("AUTH FAILED: Invalid credentials for user " + username);
        return false;
    }

    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(DB_URL, "sa", "");
             Statement stmt = conn.createStatement()) {

            // Initialize User Database
            stmt.execute("CREATE TABLE bank_users (" +
                    "user_id INT PRIMARY KEY, " +
                    "username VARCHAR(50), " +
                    "password_hash VARCHAR(64), " +
                    "is_locked BOOLEAN)");

            stmt.executeUpdate("INSERT INTO bank_users VALUES (101, 'alice_banking', 'Hash_Secure_Pass99', false)");
            stmt.executeUpdate("INSERT INTO bank_users VALUES (102, 'bob_locked', 'Hash_Pass_123', true)");

            System.out.println("==========================================");
            System.out.println("   SECURE BANKING AUTHENTICATION SYSTEM   ");
            System.out.println("==========================================");

            // Test 1: Valid Login
            authenticateUser(conn, "alice_banking", "Hash_Secure_Pass99");

            // Test 2: Invalid Password
            authenticateUser(conn, "alice_banking", "WrongPassword");

            // Test 3: Locked Account
            authenticateUser(conn, "bob_locked", "Hash_Pass_123");

            // Test 4: SQL Injection Attempt
            authenticateUser(conn, "admin' OR '1'='1", "' OR '1'='1");

            System.out.println("==========================================");

        } catch (SQLException e) {
            System.err.println("Database Setup Failure: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Parameter Binding in Authentication**:
   - `pstmt.setString(1, username)` and `pstmt.setString(2, inputPassword)` bind user input parameters safely, preventing authentication bypasses via SQL injection.
