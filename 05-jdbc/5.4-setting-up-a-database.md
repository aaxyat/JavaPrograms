# Unit 5: JDBC

## Setting Up a Database

Before a Java application can execute SQL queries, a database engine must be provisioned, database credentials configured, and JDBC driver libraries linked to the project build path.

---

### Core Concepts

#### 1. Embedded vs. Server-Based Databases

Java applications utilize two primary database setup models:

```
  Embedded Database Mode (H2 / SQLite)       Server-Based RDBMS Mode (MySQL / PostgreSQL)
  ┌─────────────────────────────────────┐    ┌────────────────────┐    ┌────────────────────┐
  │         Java JVM Process            │    │  Java JVM Process  │    │  Database Server   │
  │  ┌───────────────────────────────┐  │    │  ┌──────────────┐  │    │  ┌──────────────┐  │
  │  │  Application Logic            │  │    │  │  App Logic   │  │    │  │ RDBMS Engine │  │
  │  └──────────────┬────────────────┘  │    │  └──────┬───────┘  │    │  └──────▲───────┘  │
  │                 │ Embedded Calls    │    │         │ Network  │    │         │ TCP/IP   │
  │  ┌──────────────▼────────────────┐  │    │  ┌──────▼───────┐  │    │         │ Port     │
  │  │ Embedded DB Engine (H2.jar)   │  │    │  │ Type 4 Driver │ ──┼────┼─────────┘ 3306   │
  │  └───────────────────────────────┘  │    │  └──────────────┘  │    │                    │
  └─────────────────────────────────────┘    └────────────────────┘    └────────────────────┘
```

| Deployment Mode | Typical Engines | Setup Complexity | Best For | Connection URL Example |
|-----------------|-----------------|------------------|----------|------------------------|
| **Embedded Mode** | H2, SQLite, Derby | **Zero Setup**: Runs in-process inside JVM. | Unit testing, local development, offline desktop apps. | `jdbc:h2:./data/appdb` |
| **Server Mode** | MySQL, PostgreSQL, Oracle | **Service Setup**: Requires installing or running Docker containers. | Production enterprise apps, multi-user web systems. | `jdbc:mysql://localhost:3306/appdb` |

#### 2. Standard 4-Step Database Provisioning Workflow

1. **Provision Engine**: Launch an embedded database engine in code, or start a server database service (e.g. via Docker container `docker run -p 3306:3306 -e MYSQL_DATABASE=appdb mysql`).
2. **Add JDBC Driver Dependency**:
   - **Maven (`pom.xml`)**:
     ```xml
     <dependency>
         <groupId>com.h2database</groupId>
         <artifactId>h2</artifactId>
         <version>2.2.224</version>
     </dependency>
     ```
3. **Configure Connection Properties**: Store connection parameters (`url`, `username`, `password`, `driver-class`) in `application.properties` or environment variables.
4. **Execute Schema Migration Scripts**: Run SQL DDL scripts (`schema.sql`) to initialize database tables, primary keys, and seed reference data.

---

### Common Pitfalls

1. **Port Mismatches & Connection Timeouts**:
   - Trying to connect to MySQL on PostgreSQL's default port (`5432`) throws a `ConnectionRefusedException`. Standard default ports:
     - **MySQL / MariaDB**: `3306`
     - **PostgreSQL**: `5432`
     - **Oracle**: `1521`
     - **Microsoft SQL Server**: `1433`
2. **Hardcoding Absolute File Paths for Embedded Databases**:
   - `jdbc:h2:C:/Users/name/data/db` fails when deployed to team members' machines or cloud Linux servers. Use relative project paths (`jdbc:h2:./data/db`) or in-memory mode (`jdbc:h2:mem:db`).
3. **Access Denied / Permission Errors**:
   - Connecting with a user account lacking `CREATE TABLE` permissions fails during schema setup. Verify database user privileges (`GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';`).

---

## 1. Demo: Embedded Persistent Database Setup & Schema Initialization

### `DatabaseSetupDemo.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.io.File;

/**
 * Demonstrates provisioning a persistent embedded file database, creating directories, and executing schema DDL.
 */
public class DatabaseSetupDemo {

    // Relative file database path inside scratch folder
    private static final String DB_DIR = "scratch/database_store";
    private static final String DB_URL = "jdbc:h2:./" + DB_DIR + "/company_db;AUTO_SERVER=TRUE";
    private static final String DB_USER = "admin";
    private static final String DB_PASS = "admin123";

    public static void main(String[] args) {
        
        // 1. Ensure Local Storage Folder Exists
        File dbFolder = new File(DB_DIR);
        if (!dbFolder.exists()) {
            dbFolder.mkdirs();
            System.out.println("Created database storage folder: " + dbFolder.getAbsolutePath());
        }

        // 2. Establish Database Connection (Auto-creates database file if missing)
        System.out.println("\nConnecting to Embedded File Database...");
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS);
             Statement stmt = conn.createStatement()) {

            System.out.println("Database Connection Established Successfully!");
            System.out.println("RDBMS Engine: " + conn.getMetaData().getDatabaseProductName());

            // 3. Initialize Schema (DDL)
            String createDepartmentsTable = "CREATE TABLE IF NOT EXISTS departments (" +
                    "dept_id INT PRIMARY KEY, " +
                    "dept_name VARCHAR(50) NOT NULL, " +
                    "location VARCHAR(50))";

            stmt.execute(createDepartmentsTable);
            System.out.println("Schema Migration: Executed 'departments' table DDL.");

            // 4. Populate Initial Seed Data if Table is Empty
            ResultSet countRs = stmt.executeQuery("SELECT COUNT(*) FROM departments");
            if (countRs.next() && countRs.getInt(1) == 0) {
                stmt.executeUpdate("INSERT INTO departments VALUES (10, 'Engineering', 'Building A')");
                stmt.executeUpdate("INSERT INTO departments VALUES (20, 'Human Resources', 'Building B')");
                System.out.println("Seed Data Migration: Inserted initial department records.");
            }

            // 5. Query and Display Configured Database Contents
            System.out.println("\n=== CURRENT DATABASE DEPARTMENTS ===");
            try (ResultSet rs = stmt.executeQuery("SELECT * FROM departments")) {
                while (rs.next()) {
                    System.out.printf("Dept ID: %d | Name: %-20s | Location: %s%n",
                            rs.getInt("dept_id"), rs.getString("dept_name"), rs.getString("location"));
                }
            }

        } catch (SQLException e) {
            System.err.println("Database Setup Failure: " + e.getMessage());
            System.err.println("SQL State: " + e.getSQLState());
        }
    }
}
```

### Explanation

1. **Persistent Embedded Database (`AUTO_SERVER=TRUE`)**:
   - `jdbc:h2:./scratch/database_store/company_db` creates database files on disk under the relative project path. `AUTO_SERVER=TRUE` allows multiple process connections.

2. **Idempotent DDL Execution (`CREATE TABLE IF NOT EXISTS`)**:
   - Using `IF NOT EXISTS` prevents errors when running initialization scripts against already-created tables.

---

## 2. Practical Program: Automated SQL Script Provisioning & Migration Utility

### `AutomatedDatabaseInitializerApp.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.io.PrintWriter;
import java.io.File;
import java.io.FileReader;
import java.io.BufferedReader;
import java.io.IOException;

/**
 * Automated database provisioning tool reading external SQL migration scripts and executing DDL/DML setup.
 */
public class AutomatedDatabaseInitializerApp {

    private static final String DB_URL = "jdbc:h2:mem:migration_db;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void executeSqlScript(Connection conn, File sqlScriptFile) throws IOException, SQLException {
        System.out.println("Reading SQL Migration Script: " + sqlScriptFile.getName());
        
        StringBuilder sqlBuffer = new StringBuilder();
        try (BufferedReader reader = new BufferedReader(new FileReader(sqlScriptFile))) {
            String line;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.isEmpty() || line.startsWith("--")) continue; // Skip comments
                sqlBuffer.append(line).append(" ");
            }
        }

        // Split statements by semicolon delimiter
        String[] statements = sqlBuffer.toString().split(";");
        
        try (Statement stmt = conn.createStatement()) {
            int executedCount = 0;
            for (String singleSql : statements) {
                singleSql = singleSql.trim();
                if (!singleSql.isEmpty()) {
                    stmt.execute(singleSql);
                    executedCount++;
                }
            }
            System.out.println("Migration Completed: Executed " + executedCount + " SQL statements successfully.");
        }
    }

    public static void main(String[] args) {
        File migrationScript = new File("scratch/schema_v1.sql");
        migrationScript.getParentFile().mkdirs();

        // 1. Generate Mock SQL Migration Script File
        try (PrintWriter writer = new PrintWriter(migrationScript)) {
            writer.println("-- SQL Migration Script V1");
            writer.println("CREATE TABLE products (product_id INT PRIMARY KEY, title VARCHAR(100), price DOUBLE);");
            writer.println("INSERT INTO products VALUES (1, 'Gaming Monitor', 299.99);");
            writer.println("INSERT INTO products VALUES (2, 'Mechanical Keyboard', 89.50);");
        } catch (IOException e) {
            System.err.println("Script creation failed: " + e.getMessage());
            return;
        }

        // 2. Execute Automated Provisioning
        System.out.println("==========================================");
        System.out.println("  AUTOMATED DATABASE PROVISIONING SUITE   ");
        System.out.println("==========================================");

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS)) {

            executeSqlScript(conn, migrationScript);

            // Audit Provisioned Table
            System.out.println("\n=== AUDITING PROVISIONED TABLE ===");
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT * FROM products")) {
                while (rs.next()) {
                    System.out.printf("Product #%d | Title: %-22s | Price: $%.2f%n",
                            rs.getInt("product_id"), rs.getString("title"), rs.getDouble("price"));
                }
            }
            System.out.println("==========================================");

        } catch (IOException | SQLException e) {
            System.err.println("Provisioning Failed: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Script Parsing & Execution**:
   - Reads `schema_v1.sql`, strips comment lines (`--`), splits single statements by semicolon `;`, and executes them sequentially.

2. **Automated Migration Auditing**:
   - Queries provisioned tables immediately to verify successful table creation and seed data insertion.
