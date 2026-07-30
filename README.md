# Java Programming Course Master Index

Welcome to the Java Programming Course repository. This course provides comprehensive theoretical concepts, architecture diagrams, common pitfalls, and practical source code across 8 core units.

---

## Course Navigation

| Unit | Unit Title | Primary Topics Covered | Unit Directory |
|:---:|-------|----------------|:----------:|
| **Unit 1** | Introduction to Java | Control Statements, Loops, Arrays, Strings, Vectors, OOP, Collections, Interfaces, Exceptions, Multithreading | [Unit 1 README](01-introduction-to-java/README.md) |
| **Unit 2** | Applet Programming | Applet Architecture, Lifecycles, Graphics Methods, Web Page Embedding, Parameter Passing, Applet vs Application | [Unit 2 README](02-applet-programming/README.md) |
| **Unit 3** | GUI Programming | AWT vs Swing, Layout Managers, Atomic Controls, Containers (`JFrame`, `JPanel`), `JTree`, `JTable`, Event Handling | [Unit 3 README](03-gui-programming/README.md) |
| **Unit 4** | Java IO | Stream Abstractions, Byte/Character Streams, Scanner API, File Management, Object Serialization | [Unit 4 README](04-java-io/README.md) |
| **Unit 5** | JDBC | JDBC Drivers, Database Setup, Connection Pooling, ResultSets, Update/Delete, `PreparedStatement` | [Unit 5 README](05-jdbc/README.md) |
| **Unit 6** | Socket Programming | Network Sockets, TCP/UDP Protocols, `InetAddress`, Server-Side & Client-Side Programming, Sample Systems | [Unit 6 README](06-socket-programming/README.md) |
| **Unit 7** | Distributed Application | Distributed Objects, RMI Architecture (Stub/Skeleton, RRL, Transport), RMI Registry, Multi-JVM Systems | [Unit 7 README](07-distributed-application/README.md) |
| **Unit 8** | Overview of Servlet and JSP | Servlet & JSP Architecture, Servlet Lifecycle, Tomcat WAR Layout (`web.xml` vs `@WebServlet`), MVC Web Apps | [Unit 8 README](08-servlet-and-jsp/README.md) |

---

## Detailed Topic Directory

<details>
<summary><strong>Unit 1: Introduction to Java (12 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 1.1 | Control Statements | `ControlFlowDemo.java`, `AtmWithdrawalSystem.java` | [1.1-control-statements.md](01-introduction-to-java/1.1-control-statements.md) |
| 1.2 | Looping | `LoopingDemo.java`, `ShoppingCartCheckout.java` | [1.2-looping.md](01-introduction-to-java/1.2-looping.md) |
| 1.3 | Array | `ArrayBasicsDemo.java`, `StudentGradeTracker.java` | [1.3-array.md](01-introduction-to-java/1.3-array.md) |
| 1.4 | String and StringBuffer | `StringVsStringBufferDemo.java`, `TextSanitizerAndFormatter.java` | [1.4-string-and-stringbuffer.md](01-introduction-to-java/1.4-string-and-stringbuffer.md) |
| 1.5 | Vector | `VectorBasicsDemo.java`, `PrintJobQueueManager.java` | [1.5-vector.md](01-introduction-to-java/1.5-vector.md) |
| 1.6 | Class and Objects | `ClassAndObjectDemo.java`, `BankAccountManager.java` | [1.6-class-and-objects.md](01-introduction-to-java/1.6-class-and-objects.md) |
| 1.7 | Inheritance | `InheritanceBasicsDemo.java`, `EmployeePayrollSystem.java` | [1.7-inheritance.md](01-introduction-to-java/1.7-inheritance.md) |
| 1.8 | Polymorphism | `PolymorphismBasicsDemo.java`, `PaymentProcessingGateway.java` | [1.8-polymorphism.md](01-introduction-to-java/1.8-polymorphism.md) |
| 1.9 | Working with Collections | `CollectionsBasicsDemo.java`, `LibraryCatalogManager.java` | [1.9-working-with-collections.md](01-introduction-to-java/1.9-working-with-collections.md) |
| 1.10 | Interface and Packages | `InterfaceAndPackageDemo.java`, `CloudNotificationService.java` | [1.10-interface-and-packages.md](01-introduction-to-java/1.10-interface-and-packages.md) |
| 1.11 | Exception Handling | `ExceptionHandlingDemo.java`, `OnlineBankingTransferSystem.java` | [1.11-exception-handling.md](01-introduction-to-java/1.11-exception-handling.md) |
| 1.12 | Multi-threaded Programming | `MultithreadingBasicsDemo.java`, `TicketBookingSystem.java` | [1.12-multi-threaded-programming.md](01-introduction-to-java/1.12-multi-threaded-programming.md) |

</details>

<details>
<summary><strong>Unit 2: Applet Programming (5 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 2.1 | Introduction to Java Applets | `HelloWorldApplet.java`, `AppletLifecycleDemo.java` | [2.1-introduction.md](02-applet-programming/2.1-introduction.md) |
| 2.2 | Standard Applet Methods | `StandardMethodsDemo.java`, `AppletAudioVisualPlayer.java` | [2.2-standard-applet-methods.md](02-applet-programming/2.2-standard-applet-methods.md) |
| 2.3 | Putting an Applet on a Web Page | `WebPageAppletDemo.java`, `InteractiveWebWidgetApplet.java` | [2.3-putting-an-applet-on-a-web-page.md](02-applet-programming/2.3-putting-an-applet-on-a-web-page.md) |
| 2.4 | Passing Parameters to Applets | `AppletParameterDemo.java`, `DynamicBannerApplet.java` | [2.4-passing-parameters-to-applets.md](02-applet-programming/2.4-passing-parameters-to-applets.md) |
| 2.5 | Comparison Between Applet and Application | `AppletVsApplicationDemo.java`, `SystemEnvironmentAuditor.java` | [2.5-comparison-between-applet-and-application.md](02-applet-programming/2.5-comparison-between-applet-and-application.md) |

</details>

<details>
<summary><strong>Unit 3: GUI Programming (7 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 3.1 | Introduction to GUI Programming | `GuiBasicsDemo.java`, `UserRegistrationForm.java` | [3.1-introduction.md](03-gui-programming/3.1-introduction.md) |
| 3.2 | AWT vs. Swing | `AwtVsSwingComparisonDemo.java`, `ComponentShowcaseApp.java` | [3.2-awt-vs-swing.md](03-gui-programming/3.2-awt-vs-swing.md) |
| 3.3 | Using Swing Components | `SwingComponentsBasicsDemo.java`, `OrderFormApplication.java` | [3.3-using-swing-components.md](03-gui-programming/3.3-using-swing-components.md) |
| 3.4 | Using Atomic Components | `AtomicComponentsDemo.java`, `InteractiveControlPanelApp.java` | [3.4-using-atomic-components.md](03-gui-programming/3.4-using-atomic-components.md) |
| 3.5 | Using JFrame, JPanel, JTree and JTable | `ContainersAndDataControlsDemo.java`, `EnterpriseAssetManager.java` | [3.5-using-jframe-jpanel-jtree-and-jtable.md](03-gui-programming/3.5-using-jframe-jpanel-jtree-and-jtable.md) |
| 3.6 | Event Handling | `EventHandlingDemo.java`, `InteractiveCanvasPainter.java` | [3.6-event-handling.md](03-gui-programming/3.6-event-handling.md) |
| 3.7 | Example GUI Programming | `StudentGradeCalculatorGUI.java`, `CurrencyConverterGUI.java`, `DigitalStopwatchGUI.java`, `TodoListManagerGUI.java`, `SimpleNotepadGUI.java`, `UserLoginAuthenticationGUI.java`, `EcommerceShoppingCartGUI.java` | [3.7-example-gui-programming.md](03-gui-programming/3.7-example-gui-programming.md) |

</details>

<details>
<summary><strong>Unit 4: Java IO (5 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 4.1 | Introduction to IO in Java | `IoBasicsDemo.java`, `DataStreamAuditor.java` | [4.1-introduction-to-io-in-java.md](04-java-io/4.1-introduction-to-io-in-java.md) |
| 4.2 | Working with Input/Output APIs | `IoApiBasicsDemo.java`, `LogFileProcessor.java` | [4.2-working-with-input-output-apis.md](04-java-io/4.2-working-with-input-output-apis.md) |
| 4.3 | Working with Scanner Class | `ScannerBasicsDemo.java`, `CsvStudentDataImporter.java` | [4.3-working-with-scanner-class.md](04-java-io/4.3-working-with-scanner-class.md) |
| 4.4 | Working with Files | `FileOperationsDemo.java`, `DirectorySizeAndCleanUpTool.java` | [4.4-working-with-files.md](04-java-io/4.4-working-with-files.md) |
| 4.5 | Working with Object Serialization | `SerializationBasicsDemo.java`, `EnterpriseSessionManager.java` | [4.5-working-with-object-serialization.md](04-java-io/4.5-working-with-object-serialization.md) |

</details>

<details>
<summary><strong>Unit 5: JDBC (8 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 5.1 | Introduction to JDBC | `JdbcBasicsDemo.java`, `DatabaseConnectivityAuditor.java` | [5.1-introduction.md](05-jdbc/5.1-introduction.md) |
| 5.2 | JDBC Basics | `JdbcCrudBasicsDemo.java`, `InventoryDatabaseManager.java` | [5.2-jdbc-basics.md](05-jdbc/5.2-jdbc-basics.md) |
| 5.3 | Different Types of Drivers | `DriverTypeArchitectureDemo.java`, `DriverPerformanceBenchmarkApp.java` | [5.3-different-types-of-drivers.md](05-jdbc/5.3-different-types-of-drivers.md) |
| 5.4 | Setting Up a Database | `DatabaseSetupDemo.java`, `AutomatedDatabaseInitializerApp.java` | [5.4-setting-up-a-database.md](05-jdbc/5.4-setting-up-a-database.md) |
| 5.5 | Setting Up a Connection | `ConnectionSetupDemo.java`, `ConnectionPoolManagerApp.java` | [5.5-setting-up-a-connection.md](05-jdbc/5.5-setting-up-a-connection.md) |
| 5.6 | Retrieving Values from Result Sets | `ResultSetRetrievalDemo.java`, `DynamicDataGridExporter.java` | [5.6-retrieving-values-from-result-sets.md](05-jdbc/5.6-retrieving-values-from-result-sets.md) |
| 5.7 | Deleting/Updating Tables | `TableUpdateDeleteDemo.java`, `EnterpriseAccountManager.java` | [5.7-deleting-updating-tables.md](05-jdbc/5.7-deleting-updating-tables.md) |
| 5.8 | Working with Statement and PreparedStatement | `StatementVsPreparedStatementDemo.java`, `SecureUserAuthenticationPortal.java` | [5.8-working-with-statement-and-preparedstatement.md](05-jdbc/5.8-working-with-statement-and-preparedstatement.md) |

</details>

<details>
<summary><strong>Unit 6: Socket Programming (5 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 6.1 | Overview of Socket Programming | `NetworkAddressResolverDemo.java`, `PortScannerAuditor.java` | [6.1-overview-of-socket-programming.md](06-socket-programming/6.1-overview-of-socket-programming.md) |
| 6.2 | Introduction of APIs Related to Socket Programming | `InetAddressApiDemo.java`, `NetworkInterfaceAuditor.java` | [6.2-introduction-of-apis-related-to-socket-programming.md](06-socket-programming/6.2-introduction-of-apis-related-to-socket-programming.md) |
| 6.3 | Server Side Programming [TCP and UDP] | `TcpAndUdpServerDemo.java`, `MultiThreadedEchoServer.java` | [6.3-server-side-programming.md](06-socket-programming/6.3-server-side-programming.md) |
| 6.4 | Client Side Programming [TCP and UDP] | `TcpAndUdpClientDemo.java`, `InteractiveChatClient.java` | [6.4-client-side-programming.md](06-socket-programming/6.4-client-side-programming.md) |
| 6.5 | Sample Programs | `ChatServer.java`, `ChatClient.java`, `FileServer.java`, `FileClient.java`, `UdpHeartbeatServer.java`, `UdpHeartbeatClient.java`, `MiniHttpWebServer.java` | [6.5-sample-programs.md](06-socket-programming/6.5-sample-programs.md) |

</details>

<details>
<summary><strong>Unit 7: Distributed Application (5 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 7.1 | Introduction to Distributed Objects | `DistributedObjectConceptDemo.java`, `DistributedNodeHealthMonitor.java` | [7.1-introduction-to-distributed-objects.md](07-distributed-application/7.1-introduction-to-distributed-objects.md) |
| 7.2 | Overview of RMI | `RmiOverviewDemo.java`, `RemoteComputeServiceAuditor.java` | [7.2-overview-of-rmi.md](07-distributed-application/7.2-overview-of-rmi.md) |
| 7.3 | RMI Architecture | `RmiArchitectureSimulator.java`, `RmiRegistryInspectorApp.java` | [7.3-rmi-architecture.md](07-distributed-application/7.3-rmi-architecture.md) |
| 7.4 | Creating Distributed Application Using RMI | `ComputeService.java`, `ComputeServiceImpl.java`, `ComputeServer.java`, `ComputeClient.java`, `BankService.java`, `BankServiceImpl.java`, `BankServer.java`, `BankClient.java` | [7.4-creating-distributed-application-using-rmi.md](07-distributed-application/7.4-creating-distributed-application-using-rmi.md) |
| 7.5 | Example Distributed Programs | `PatientRecord.java`, `HospitalService.java`, `HospitalServiceImpl.java`, `HospitalServer.java`, `HospitalClient.java`, `ExamPaper.java`, `ExamService.java`, `ExamServiceImpl.java`, `ExamServer.java`, `ExamClient.java`, `FlightInfo.java`, `FlightReservationService.java`, `FlightReservationServiceImpl.java`, `FlightServer.java`, `FlightClient.java` | [7.5-example-distributed-programs.md](07-distributed-application/7.5-example-distributed-programs.md) |

</details>

<details>
<summary><strong>Unit 8: Overview of Servlet and JSP (3 Topics)</strong></summary>

| # | Topic | Key Programs Included | Reference File |
|---|-------|----------------------|----------------|
| 8.1 | Introduction to Servlet and JSP and its Architecture | `ServletArchitectureDemo.java`, `MvcWebArchitectureAuditor.java` | [8.1-introduction-to-servlet-and-jsp-and-its-architecture.md](08-servlet-and-jsp/8.1-introduction-to-servlet-and-jsp-and-its-architecture.md) |
| 8.2 | Configuring Apache Tomcat to Host Servlet/JSP Files | `WebXmlDeploymentConfigDemo.java`, `TomcatWarStructureVerifier.java` | [8.2-configuring-apache-tomcat-to-host-servlet-jsp-files.md](08-servlet-and-jsp/8.2-configuring-apache-tomcat-to-host-servlet-jsp-files.md) |
| 8.3 | Sample Program of Servlet and JSP | `LoginServlet.java`, `LogoutServlet.java`, `login.jsp`, `dashboard.jsp`, `Product.java`, `CartServlet.java`, `catalog.jsp`, `cart.jsp`, `Student.java`, `RegistrationServlet.java`, `register.html`, `result.jsp` | [8.3-sample-program-of-servlet-and-jsp.md](08-servlet-and-jsp/8.3-sample-program-of-servlet-and-jsp.md) |

</details>
