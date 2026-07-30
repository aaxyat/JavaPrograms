# Unit 5: JDBC

## Setting Up a Connection

A **JDBC Connection** (`java.sql.Connection`) establishes a physical session between a Java application and a database server. Through an active connection, applications execute SQL statements, manage transactions, and query database metadata.

---

### Core Concepts

#### 1. Connection Establishment Methods
The `DriverManager` class provides three overloaded `getConnection()` factory methods:

```java
// Method 1: URL with embedded credentials
Connection conn1 = DriverManager.getConnection("jdbc:h2:mem:db;USER=sa;PASSWORD=secret");

// Method 2: Separate URL, Username, and Password (Most Common)
Connection conn2 = DriverManager.getConnection("jdbc:h2:mem:db", "sa", "secret");

// Method 3: Using a Properties Object (Advanced Configuration)
Properties props = new Properties();
props.put("user", "sa");
props.put("password", "secret");
props.put("connectTimeout", "5000"); // 5-second connection timeout
Connection conn3 = DriverManager.getConnection("jdbc:h2:mem:db", props);
```

#### 2. Essential `Connection` Control Methods

| Method | Purpose | Description |
|--------|---------|-------------|
| **`isValid(int timeoutSeconds)`** | Validation | Verifies if the connection is active and responsive within the specified timeout. |
| **`setAutoCommit(boolean autoCommit)`** | Transactions | Toggles auto-commit mode (`true` by default). Set to `false` for manual transaction management. |
| **`commit()` / `rollback()`** | Transactions | Commits or rolls back statements executed within the current transaction block. |
| **`setReadOnly(boolean readOnly)`** | Optimization | Hints to the database driver that operations are read-only, enabling database query optimizations. |
| **`close()`** | Resource Cleanup | Closes the connection socket or returns the connection to a connection pool. |

#### 3. Connection Pooling Concept
Creating a physical TCP/IP socket connection to a remote database server is computationally expensive (often taking 50–200 milliseconds per connection). **Connection Pools** (such as HikariCP or Apache DBCP) maintain a pool of pre-established database connections:

```
  Java Application Threads                      Connection Pool                   Database Server
  ┌──────────────────────┐                    ┌───────────────────┐             ┌─────────────────┐
  │  Worker Thread #1    │ ──Borrow Connection──> │  [ Active Conn ]  │ ──────────> │  Database RDBMS │
  │  Worker Thread #2    │ <──Return Connection─ │  [ Idle Conn   ]  │             └─────────────────┘
  └──────────────────────┘                    └───────────────────┘
```

---

### Common Pitfalls

1. **Creating Physical Connections Per Request in Production**:
   - Calling `DriverManager.getConnection()` inside high-frequency HTTP request loops causes severe performance degradation. Use connection pools for multi-threaded applications.
2. **Hanging Threads Due to Missing Connection Timeouts**:
   - Omitting connection timeouts causes application threads to block indefinitely if a remote database server drops packets or goes offline. Pass timeout properties to `DriverManager`.
3. **Leaking Connections in Error Paths**:
   - If an exception occurs during SQL execution before `.close()` is reached, the connection remains open. Always manage connections with **try-with-resources** statements.

---

## 1. Demo: Advanced Connection Properties and Configuration

### `ConnectionSetupDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

/**
 * Demonstrates establishing JDBC connections using Properties objects, setting timeouts, and configuring auto-commit/read-only modes.
 */
public class ConnectionSetupDemo {

    private static final String DB_URL = "jdbc:h2:mem:config_db;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) {
        
        System.out.println("=== 1. ESTABLISHING CONNECTION VIA PROPERTIES OBJECT ===");

        // Configure connection properties
        Properties connectionProps = new Properties();
        connectionProps.put("user", "sa");
        connectionProps.put("password", "");
        connectionProps.put("PAGE_SIZE", "1024"); // Engine specific property

        try (Connection conn = DriverManager.getConnection(DB_URL, connectionProps)) {

            // 1. Verify Connection Health
            boolean isValid = conn.isValid(3); // 3-second timeout validation
            System.out.println("Connection Validated? : " + isValid);
            System.out.println("Database Engine      : " + conn.getMetaData().getDatabaseProductName());

            // 2. Configure Connection Settings
            conn.setAutoCommit(false); // Disable auto-commit for transaction control
            System.out.println("Auto-Commit Mode      : " + conn.getAutoCommit());

            conn.setReadOnly(true);    // Enable read-only mode optimization
            System.out.println("Read-Only Mode       : " + conn.isReadOnly());

            System.out.println("Catalog / Schema     : " + conn.getCatalog());

            // Commit manual configuration
            conn.commit();
            System.out.println("Configuration changes committed successfully.");

        } catch (SQLException e) {
            System.err.println("Connection Setup Exception: " + e.getMessage());
            System.err.println("SQL State: " + e.getSQLState());
        }
    }
}
```

### Explanation

1. **Using `Properties` for Driver Configuration**:
   - `Properties` passes custom parameters (credentials, page sizes, network timeouts) to `DriverManager.getConnection()`.

2. **Connection Health Check (`conn.isValid(3)`)**:
   - Validates that the connection is alive within a 3-second timeout window before executing SQL queries.

3. **Transaction & Optimization Controls**:
   - `setAutoCommit(false)` disables automatic commits, requiring explicit `conn.commit()` calls.

---

## 2. Practical Program: Simulated High-Performance Connection Pool

### `ConnectionPoolManagerApp.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

/**
 * Lightweight Connection Pool implementation demonstrating connection reuse and pool lifecycle management.
 */
class SimpleConnectionPool {
    private final String dbUrl;
    private final String dbUser;
    private final String dbPass;
    private final List<Connection> connectionPool;
    private final List<Connection> usedConnections;
    private static final int INITIAL_POOL_SIZE = 5;

    public SimpleConnectionPool(String dbUrl, String dbUser, String dbPass) throws SQLException {
        this.dbUrl = dbUrl;
        this.dbUser = dbUser;
        this.dbPass = dbPass;
        this.connectionPool = new ArrayList<>(INITIAL_POOL_SIZE);
        this.usedConnections = new ArrayList<>(INITIAL_POOL_SIZE);

        // Pre-establish pool connections
        for (int i = 0; i < INITIAL_POOL_SIZE; i++) {
            connectionPool.add(createPhysicalConnection());
        }
    }

    private Connection createPhysicalConnection() throws SQLException {
        return DriverManager.getConnection(dbUrl, dbUser, dbPass);
    }

    public synchronized Connection getConnection() throws SQLException {
        if (connectionPool.isEmpty()) {
            if (usedConnections.size() < INITIAL_POOL_SIZE * 2) {
                connectionPool.add(createPhysicalConnection());
            } else {
                throw new SQLException("Connection pool exhausted! Maximum limit reached.");
            }
        }

        Connection conn = connectionPool.remove(connectionPool.size() - 1);
        usedConnections.add(conn);
        return conn;
    }

    public synchronized void releaseConnection(Connection conn) {
        if (conn != null && usedConnections.remove(conn)) {
            connectionPool.add(conn);
        }
    }

    public synchronized void shutdown() throws SQLException {
        for (Connection conn : connectionPool) {
            if (conn != null && !conn.isClosed()) conn.close();
        }
        for (Connection conn : usedConnections) {
            if (conn != null && !conn.isClosed()) conn.close();
        }
        connectionPool.clear();
        usedConnections.clear();
        System.out.println("Connection Pool shut down completely. All sockets closed.");
    }

    public synchronized int getIdleCount() { return connectionPool.size(); }
    public synchronized int getActiveCount() { return usedConnections.size(); }
}

public class ConnectionPoolManagerApp {

    private static final String DB_URL = "jdbc:h2:mem:pool_db;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     SIMULATED CONNECTION POOL MANAGER    ");
        System.out.println("==========================================");

        try {
            SimpleConnectionPool pool = new SimpleConnectionPool(DB_URL, "sa", "");
            System.out.printf("Initialized Connection Pool -> Idle: %d | Active: %d%n",
                    pool.getIdleCount(), pool.getActiveCount());

            // Borrow Connection #1
            Connection conn1 = pool.getConnection();
            System.out.printf("Borrowed Conn #1 -> Idle: %d | Active: %d%n", pool.getIdleCount(), pool.getActiveCount());

            // Borrow Connection #2
            Connection conn2 = pool.getConnection();
            System.out.printf("Borrowed Conn #2 -> Idle: %d | Active: %d%n", pool.getIdleCount(), pool.getActiveCount());

            // Release Connection #1 back to pool
            pool.releaseConnection(conn1);
            System.out.printf("Released Conn #1 -> Idle: %d | Active: %d%n", pool.getIdleCount(), pool.getActiveCount());

            // Shutdown pool
            pool.shutdown();
            System.out.println("==========================================");

        } catch (SQLException e) {
            System.err.println("Pool Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Pre-allocating Database Connections**:
   - `SimpleConnectionPool` establishes 5 physical database connections upfront during initialization.

2. **Borrow and Release Pattern**:
   - Applications borrow idle connections using `pool.getConnection()` and return them via `pool.releaseConnection(conn)`, eliminating the overhead of creating new TCP connections for every database task.
