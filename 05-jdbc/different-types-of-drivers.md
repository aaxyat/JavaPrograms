# Unit 5: JDBC

## Different Types of Drivers

A **JDBC Driver** is a software component that enables a Java application to interact with a specific database. Database vendors implement JDBC driver interfaces to translate standard Java database calls into proprietary database protocols. JDBC drivers are categorized into four distinct architecture types.

---

### Core Concepts

#### 1. Architectural Overview of the 4 JDBC Driver Types

```
  Type 1: JDBC-ODBC Bridge           Type 2: Native-API Driver
  ┌──────────────┐                   ┌──────────────┐
  │  Java App    │                   │  Java App    │
  └──────┬───────┘                   └──────┬───────┘
         │ JDBC                             │ JDBC
  ┌──────▼───────┐                   ┌──────▼───────┐
  │ ODBC Bridge  │                   │ Native Driver│
  └──────┬───────┘                   └──────┬───────┘
         │ Native C                         │ Native C/C++
  ┌──────▼───────┐                   ┌──────▼───────┐
  │ Native ODBC  │                   │  DB Client   │
  └──────┬───────┘                   └──────┬───────┘
         │ DB Net                           │ DB Net
  ┌──────▼───────┐                   ┌──────▼───────┐
  │   Database   │                   │   Database   │
  └──────────────┘                   └──────────────┘

  Type 3: Network-Protocol Driver     Type 4: Thin / Pure Java Driver
  ┌──────────────┐                   ┌──────────────┐
  │  Java App    │                   │  Java App    │
  └──────┬───────┘                   └──────┬───────┘
         │ Pure Java Net                    │ Pure Java Net Sockets
  ┌──────▼───────┐                   │ (Direct Vendor Protocol)
  │ Middleware   │                   │
  └──────┬───────┘                   │
         │ DB Net                    │
  ┌──────▼───────┐                   ┌──────▼───────┐
  │   Database   │                   │   Database   │
  └──────────────┘                   └──────────────┘
```

#### 2. Detailed Technical Breakdown

##### Type 1: JDBC-ODBC Bridge Driver
- **Mechanism**: Converts JDBC method calls into Microsoft ODBC (Open Database Connectivity) calls, which then talk to the database.
- **Advantages**: Allowed Java to connect to databases in the 1990s before native Java drivers existed.
- **Disadvantages**: Slow performance due to double conversion layers. Requires configuring native ODBC data sources on every client desktop. **Removed in Java 8 and completely unavailable in Java 9+**.

##### Type 2: Native-API Driver (Partially Java Driver)
- **Mechanism**: Converts JDBC calls into database-specific C/C++ native client library calls (such as Oracle OCI or DB2 CLI libraries).
- **Advantages**: Faster than Type 1 because it bypasses the ODBC translation layer.
- **Disadvantages**: Requires installing vendor-specific C/C++ native client libraries (`.dll` on Windows, `.so` on Linux) on every client machine. Not platform-independent.

##### Type 3: Network-Protocol Driver (Middleware Driver)
- **Mechanism**: Converts JDBC calls into a vendor-neutral network protocol sent to a middleware application server. The middleware server translates the requests into database-specific socket protocols.
- **Advantages**: 100% Java client without native client libraries. Client code is database-independent because the middleware server handles database switching.
- **Disadvantages**: Introduces network latency due to the intermediate server hop. Requires deploying and maintaining dedicated middleware infrastructure.

##### Type 4: Thin Driver / Pure Java Driver (Direct-to-Database Driver)
- **Mechanism**: Converts JDBC calls directly into vendor-specific database network protocols using pure Java sockets.
- **Advantages**: **Industry Standard**. Written 100% in Java. Fully platform-independent, requiring zero client-side installation or native C/C++ binaries. Delivers maximum execution speed.
- **Disadvantages**: Driver JARs are vendor-specific (e.g., MySQL Connector/J, PostgreSQL JDBC Driver, Oracle Thin Driver).

#### 3. Driver Comparison Matrix

| Driver Category | Pure Java Client? | Platform Independent? | Performance | Client Installation Required? | Modern Status |
|-----------------|-------------------|------------------------|-------------|----------------───────────────|---------------|
| **Type 1 (Bridge)** | No | No | Low | Yes (ODBC Config) | **Obsolete / Removed** |
| **Type 2 (Native API)** | Partial | No | Medium-High | Yes (Native C/C++ DLLs) | Legacy / Rare |
| **Type 3 (Network)** | Yes | Yes | Medium | No | Niche Enterprise |
| **Type 4 (Thin)** | **Yes** | **Yes** | **High** | **No** | **Industry Standard** |

---

### Common Pitfalls

1. **Attempting to Use `sun.jdbc.odbc.JdbcOdbcDriver` on Modern Java**:
   - Compiling or running code referencing `JdbcOdbcDriver` on Java 8 or higher throws a `ClassNotFoundException` because Oracle removed the bridge driver.
2. **Missing Driver Classpath Dependencies**:
   - Modern JDBC drivers (Type 4) ship as external JAR files (e.g. `h2-2.2.xxx.jar`, `mysql-connector-j.jar`). Forgetting to add the driver JAR to your project build path causes `SQLException: No suitable driver found`.
3. **Deploying Type 2 Native Drivers to Containerized Environments**:
   - Type 2 drivers fail inside Docker containers unless the corresponding vendor `.so` native C++ libraries are compiled for the container OS image. Use Type 4 drivers for cloud containers.

---

## 1. Demo: Dynamic Driver Loading and Capabilities Inspection

### `DriverTypeArchitectureDemo.java`

```java
import java.sql.DriverManager;
import java.sql.Driver;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Enumeration;

/**
 * Demonstrates discovering registered JDBC drivers, inspecting driver properties, and identifying driver type metadata.
 */
public class DriverTypeArchitectureDemo {

    public static void main(String[] args) {
        
        System.out.println("==========================================");
        System.out.println("   REGISTERED JDBC DRIVERS ARCHITECTURE   ");
        System.out.println("==========================================");

        // 1. Enumerate all registered JDBC drivers loaded in the JVM
        Enumeration<Driver> drivers = DriverManager.getDrivers();

        int driverCount = 0;
        while (drivers.hasMoreElements()) {
            driverCount++;
            Driver driver = drivers.nextElement();
            
            System.out.println("Driver #" + driverCount + ": " + driver.getClass().getName());
            System.out.println("  -> Major Version  : " + driver.getMajorVersion());
            System.out.println("  -> Minor Version  : " + driver.getMinorVersion());
            System.out.println("  -> JDBC Compliant?: " + driver.jdbcCompliant());
            
            // Type 4 Drivers are pure Java implementations
            boolean isPureJava = !driver.getClass().getName().contains("odbc") && 
                                 !driver.getClass().getName().contains("native");
            System.out.println("  -> Architecture   : " + (isPureJava ? "Type 4 (Thin / Pure Java Driver)" : "Legacy Native Driver"));
            System.out.println("------------------------------------------");
        }

        if (driverCount == 0) {
            System.out.println("No drivers registered via Service Provider Interface (SPI). Loading H2 Driver explicitly...");
            try {
                // Explicit Type 4 driver registration
                Class.forName("org.h2.Driver");
                System.out.println("H2 Driver loaded dynamically via Class.forName().");
            } catch (ClassNotFoundException e) {
                System.err.println("Driver Loading Error: " + e.getMessage());
            }
        }

        // 2. Connect using Type 4 Pure Java In-Memory Driver
        String testUrl = "jdbc:h2:mem:driver_test_db;DB_CLOSE_DELAY=-1";
        try (Connection conn = DriverManager.getConnection(testUrl, "sa", "")) {
            System.out.println("\nSuccessfully connected using Pure Java Driver!");
            System.out.println("Connected RDBMS: " + conn.getMetaData().getDatabaseProductName());
        } catch (SQLException e) {
            System.err.println("Connection Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Enumerating Loaded Drivers (`DriverManager.getDrivers()`)**:
   - Queries the JVM `DriverManager` registry to discover active JDBC driver instances.

2. **Automatic SPI Registration**:
   - JDBC 4.0+ drivers include a `META-INF/services/java.sql.Driver` entry in their JAR file, registering drivers automatically without needing `Class.forName()`.

---

## 2. Practical Program: Multi-Driver Performance Benchmark Tool

### `DriverPerformanceBenchmarkApp.java`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Performance benchmarking utility measuring connection setup latency and query execution speed for Type 4 drivers.
 */
public class DriverPerformanceBenchmarkApp {

    private static final String DB_URL = "jdbc:h2:mem:benchmark_db;DB_CLOSE_DELAY=-1";
    private static final String DB_USER = "sa";
    private static final String DB_PASS = "";

    public static void runBenchmark() {
        System.out.println("==========================================");
        System.out.println("  TYPE 4 PURE JAVA DRIVER BENCHMARK SUITE ");
        System.out.println("==========================================");

        // 1. Benchmark Connection Establishment Time
        long startTime = System.nanoTime();
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS)) {
            long connectionTime = System.nanoTime() - startTime;
            System.out.printf("1. Connection Establishment Latency : %.3f ms%n", connectionTime / 1_000_000.0);

            // 2. Benchmark Batch Setup & Query Execution
            try (Statement stmt = conn.createStatement()) {
                stmt.execute("CREATE TABLE benchmark_log (id INT PRIMARY KEY, payload VARCHAR(100))");

                long insertStart = System.nanoTime();
                for (int i = 1; i <= 1000; i++) {
                    stmt.executeUpdate("INSERT INTO benchmark_log VALUES (" + i + ", 'Payload Data Segment #" + i + "')");
                }
                long insertTime = System.nanoTime() - insertStart;
                System.out.printf("2. 1,000 Single Row Inserts Time   : %.3f ms%n", insertTime / 1_000_000.0);

                long queryStart = System.nanoTime();
                try (ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM benchmark_log")) {
                    if (rs.next()) {
                        int count = rs.getInt(1);
                        long queryTime = System.nanoTime() - queryStart;
                        System.out.printf("3. Aggregate SELECT Query Time (%d) : %.3f ms%n", count, queryTime / 1_000_000.0);
                    }
                }
            }

        } catch (SQLException e) {
            System.err.println("Benchmark Execution Error: " + e.getMessage());
        }

        System.out.println("==========================================");
    }

    public static void main(String[] args) {
        runBenchmark();
    }
}
```

### Explanation

1. **Measuring Connection Latency**:
   - Uses `System.nanoTime()` to record the time taken by a Type 4 Pure Java driver to establish a memory socket connection.

2. **Benchmarking Execution Speed**:
   - Evaluates row insertion and aggregate query speeds in milliseconds (`/ 1_000_000.0`).
