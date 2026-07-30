# Unit 5: JDBC

## Introduction to JDBC

**JDBC (Java Database Connectivity)** is a standard Java API defined in the `java.sql` and `javax.sql` packages. It provides a uniform, database-independent interface for Java applications to connect to Relational Database Management Systems (RDBMS) such as MySQL, PostgreSQL, Oracle, SQLite, and H2.

---

### Core Concepts

#### 1. JDBC Architecture Overview

```
   ┌────────────────────────────────────────────────────────┐
   │                  Java Application                      │
   └───────────────────────────┬────────────────────────────┘
                               │
   ┌───────────────────────────▼────────────────────────────┐
   │              JDBC API (java.sql / javax.sql)           │
   └───────────────────────────┬────────────────────────────┘
                               │
   ┌───────────────────────────▼────────────────────────────┐
   │                  JDBC Driver Manager                   │
   └───────────────────────────┬────────────────────────────┘
                               │
   ┌───────────────────────────▼────────────────────────────┐
   │          JDBC Driver (Type 4 Pure Java Driver)         │
   └───────────────────────────┬────────────────────────────┘
                               │ Database Network Protocol
   ┌───────────────────────────▼────────────────────────────┐
   │                Database Server (RDBMS)                 │
   └────────────────────────────────────────────────────────┘
```

#### 2. The 4 Types of JDBC Drivers

| Driver Type | Name | Architecture | Characteristics |
|-------------|------|--------------|-----------------|
| **Type 1** | JDBC-ODBC Bridge | Translates JDBC calls to native ODBC calls. | Deprecated in Java 8. Requires client-side ODBC driver installation. |
| **Type 2** | Native-API Driver | Converts JDBC calls to database-specific C/C++ client library calls. | Requires vendor native C/C++ libraries on every client machine. |
| **Type 3** | Network-Protocol Driver | Sends JDBC calls to a middleware server, which translates them for the database. | Database-independent client; requires maintaining a middleware server. |
| **Type 4** | **Thin / Pure Java Driver** | Converts JDBC calls directly into vendor-specific database network protocols. | **Industry Standard.** Written entirely in Java. No client installation required. |

#### 3. Standard 5 Steps of a JDBC Database Operation

1. **Load the JDBC Driver**: Loads the driver class into memory (automatic via SPI in modern JDBC 4.0+ drivers).
2. **Establish Connection**: Call `DriverManager.getConnection(url, username, password)` to obtain a `Connection` handle.
3. **Create Statement**: Call `connection.createStatement()` or `connection.prepareStatement(sql)`.
4. **Execute Query / Update**:
   - `executeQuery(sql)`: Executes `SELECT` queries, returning a tabular `ResultSet`.
   - `executeUpdate(sql)`: Executes DML statements (`INSERT`, `UPDATE`, `DELETE`), returning an `int` count of affected rows.
5. **Process Results & Close Resources**: Iterate through the `ResultSet` and close `ResultSet`, `Statement`, and `Connection` handles.

---

### Common Pitfalls

1. **Leaking Database Connections**:
   - Failing to close database connections consumes server connection pool limits, eventually crashing the database server with "Too many connections" errors. Wrap connections in **try-with-resources** blocks.
2. **Confusing `executeQuery()` and `executeUpdate()`**:
   - Invoking `executeQuery()` on an `INSERT` or `UPDATE` statement throws a `SQLException`. Use `executeQuery()` exclusively for `SELECT` queries, and `executeUpdate()` for data modifications.
3. **Hardcoding Database Password Credentials**:
   - Hardcoding plain-text database credentials in source code exposes security vulnerabilities. Read credentials from environment variables (`System.getenv()`) or secure configuration properties.

---

## 1. Demo: Database Connection, Table Setup, and Query Execution

### `JdbcBasicsDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Demonstrates JDBC database connection setup, table creation, INSERT operations, and SELECT queries using an in-memory database.
 */
public class JdbcBasicsDemo {

    // In-memory H2 database connection URL
    private static final String DB_URL = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void main(String[] args) {
        
        // 1. Establish Database Connection using try-with-resources
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS);
             Statement stmt = conn.createStatement()) {

            System.out.println("1. Connected to Database Successfully: " + conn.getMetaData().getDatabaseProductName());

            // 2. Create Database Table (DDL)
            String createTableSql = "CREATE TABLE students (" +
                    "id INT PRIMARY KEY, " +
                    "name VARCHAR(50), " +
                    "gpa DOUBLE)";
            stmt.execute(createTableSql);
            System.out.println("2. Executed Table Creation DDL.");

            // 3. Insert Student Records (DML using executeUpdate)
            String insertSql1 = "INSERT INTO students VALUES (101, 'Alice Vance', 3.85)";
            String insertSql2 = "INSERT INTO students VALUES (102, 'Bob Miller', 3.42)";
            
            int rows1 = stmt.executeUpdate(insertSql1);
            int rows2 = stmt.executeUpdate(insertSql2);
            System.out.println("3. Inserted " + (rows1 + rows2) + " records into database.");

            // 4. Query Records (SELECT using executeQuery)
            String selectSql = "SELECT id, name, gpa FROM students";
            System.out.println("\n4. Executing SELECT Query:");
            System.out.println("------------------------------------------");
            
            try (ResultSet rs = stmt.executeQuery(selectSql)) {
                while (rs.next()) {
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    double gpa = rs.getDouble("gpa");

                    System.out.printf("Student ID: %d | Name: %-15s | GPA: %.2f%n", id, name, gpa);
                }
            }
            System.out.println("------------------------------------------");

        } catch (SQLException e) {
            System.err.println("JDBC Database Error: " + e.getMessage());
            System.err.println("SQL State : " + e.getSQLState());
            System.err.println("Error Code: " + e.getErrorCode());
        }
    }
}
```

### Explanation

1. **Establishing Connection (`DriverManager.getConnection`)**:
   - Connects to an in-memory database (`jdbc:h2:mem:testdb`), returning a `Connection` handle.

2. **DML Execution (`stmt.executeUpdate`)**:
   - `executeUpdate()` executes `INSERT` statements, returning the integer number of affected database rows.

3. **Query Iteration (`ResultSet.next()`)**:
   - `stmt.executeQuery(selectSql)` returns a `ResultSet`. `rs.next()` advances the cursor row-by-row, extracting column data via typed getters (`rs.getInt()`, `rs.getString()`, `rs.getDouble()`).

---

## 2. Practical Program: Database Connection & Metadata Health Auditor

### `DatabaseConnectivityAuditor.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.DatabaseMetaData;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Diagnostic database health auditor inspecting RDBMS metadata, driver specifications, and supported features.
 */
public class DatabaseConnectivityAuditor {

    private static final String DB_URL = "jdbc:h2:mem:auditdb;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void auditDatabaseSystem() {
        System.out.println("==========================================");
        System.out.println("     JDBC DATABASE METADATA HEALTH REPORT ");
        System.out.println("==========================================");

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS)) {

            DatabaseMetaData metaData = conn.getMetaData();

            System.out.println("Database Product Name    : " + metaData.getDatabaseProductName());
            System.out.println("Database Product Version : " + metaData.getDatabaseProductVersion());
            System.out.println("JDBC Driver Name         : " + metaData.getDriverName());
            System.out.println("JDBC Driver Version      : " + metaData.getDriverVersion());
            System.out.println("JDBC Major Version       : " + metaData.getJDBCMajorVersion());
            System.out.println("Max Connections Supported: " + metaData.getMaxConnections());
            System.out.println("Transaction Supported?   : " + metaData.supportsTransactions());
            System.out.println("Batch Updates Supported? : " + metaData.supportsBatchUpdates());

            System.out.println("\n=== SYSTEM CATALOG TABLES ===");
            try (ResultSet tables = metaData.getTables(null, null, "%", new String[]{"TABLE"})) {
                int tableCount = 0;
                while (tables.next()) {
                    tableCount++;
                    String tableName = tables.getString("TABLE_NAME");
                    String tableType = tables.getString("TABLE_TYPE");
                    System.out.printf("  Table #%d: %-25s [%s]%n", tableCount, tableName, tableType);
                }
                if (tableCount == 0) {
                    System.out.println("  No user tables created in current schema.");
                }
            }
            System.out.println("==========================================");

        } catch (SQLException e) {
            System.err.println("Database Connection Audit Failed: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        auditDatabaseSystem();
    }
}
```

### Explanation

1. **Extracting `DatabaseMetaData`**:
   - `conn.getMetaData()` extracts technical database attributes such as product name, database engine version, driver information, transaction capability, and maximum connections.

2. **Catalog Metadata Retrieval (`metaData.getTables()`)**:
   - Queries system schema catalogs to discover existing database tables without writing SQL queries.
