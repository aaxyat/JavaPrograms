# Unit 8: Overview of Servlet and JSP

## Configuring Apache Tomcat to Host Servlet/JSP Files

**Apache Tomcat** is an open-source Java Servlet Container and JSP Engine implementing the Jakarta Servlet and JavaServer Pages specifications. It provides an HTTP web server environment for executing Servlets and rendering JSPs.

---

### Core Concepts

#### 1. Standard Web Application Directory Structure (`WAR` Layout)

A Java web application must follow a strict directory layout to be recognized by Tomcat:

```
  my-web-app/                             <-- Application Root Directory
  ├── index.jsp                           <-- Public JSP Web Page
  ├── login.html                          <-- Public Static HTML Page
  ├── css/                                <-- Public Static Assets (CSS)
  │   └── styles.css
  ├── js/                                 <-- Public Static Assets (JS)
  │   └── app.js
  └── WEB-INF/                            <-- PRIVATE / PROTECTED DIRECTORY
      ├── web.xml                         <-- Deployment Descriptor (XML Config)
      ├── classes/                        <-- Compiled Java Bytecode (.class & Servlets)
      │   └── com/app/LoginServlet.class
      └── lib/                            <-- Third-Party Library JARs (.jar files)
          └── h2-2.2.224.jar
```

> **Security Guard**: Files located inside `WEB-INF/` can **never** be accessed directly by a browser URL (e.g. typing `http://localhost:8080/my-web-app/WEB-INF/web.xml` returns HTTP `404 Not Found`). Only server-side Java code inside Servlets can access `WEB-INF/` resources.

#### 2. Deployment Configuration: `web.xml` vs. `@WebServlet`

##### Option A: XML Deployment Descriptor (`WEB-INF/web.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <display-name>Enterprise Web Portal</display-name>

    <!-- Servlet Declaration -->
    <servlet>
        <servlet-name>UserLoginServlet</servlet-name>
        <servlet-class>com.app.LoginServlet</servlet-class>
    </servlet>

    <!-- Servlet URL Mapping -->
    <servlet-mapping>
        <servlet-name>UserLoginServlet</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>

    <!-- Default Welcome Files -->
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

</web-app>
```

##### Option B: Java Annotations (`@WebServlet`)
In Servlet 3.0+, annotations replace verbose XML configuration:

```java
package com.app;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;

@WebServlet(name = "UserLoginServlet", urlPatterns = {"/login", "/auth"})
public class LoginServlet extends HttpServlet {
    // Servlet logic...
}
```

#### 3. Essential Apache Tomcat Configuration Files

| Configuration File | Path in Tomcat | Purpose |
|--------------------|----------------|---------|
| **`server.xml`** | `tomcat/conf/server.xml` | Configures HTTP connector ports (default `8080`), SSL/TLS ports (`8443`), thread pool sizes, and virtual hosts. |
| **`web.xml` (Global)**| `tomcat/conf/web.xml` | Sets global MIME type mappings, global welcome files, default servlets, and session timeout defaults (30 mins). |
| **`context.xml`** | `tomcat/conf/context.xml` | Configures shared JNDI DataSources, database connection pools, and context-wide parameters. |

---

### Common Pitfalls

1. **Placing Public Pages Inside `WEB-INF/`**:
   - Placing `index.jsp` inside `WEB-INF/` prevents users from opening the page via their browser address bar. Place public HTML/JSP files in the root folder, outside `WEB-INF/`.
2. **Port 8080 Collision**:
   - If another service (such as Oracle or Jenkins) is using port `8080`, Tomcat fails to start with `java.net.BindException: Address already in use`. Change the port to `8090` in `conf/server.xml`:
     ```xml
     <Connector port="8090" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
     ```
3. **Placing Uncompiled `.java` Files in `classes/`**:
   - Tomcat executes compiled `.class` bytecode inside `WEB-INF/classes/`. Placing `.java` source files there causes a `ClassNotFoundException`.

---

## 1. Demo: Programmatic `web.xml` Deployment Descriptor Generator

### `WebXmlDeploymentConfigDemo.java`

```java
import java.io.File;
import java.io.PrintWriter;
import java.io.IOException;

/**
 * Generator utility producing valid WEB-INF/web.xml deployment descriptor markup.
 */
public class WebXmlDeploymentConfigDemo {

    public static String generateWebXmlContent(String appTitle, String servletName, String servletClass, String urlPattern) {
        StringBuilder xml = new StringBuilder();
        xml.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
        xml.append("<web-app xmlns=\"http://xmlns.jcp.org/xml/ns/javaee\"\n");
        xml.append("         xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n");
        xml.append("         xsi:schemaLocation=\"http://xmlns.jcp.org/xml/ns/javaee\n");
        xml.append("                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd\"\n");
        xml.append("         version=\"4.0\">\n\n");

        xml.append("    <display-name>").append(appTitle).append("</display-name>\n\n");

        // Register Servlet
        xml.append("    <servlet>\n");
        xml.append("        <servlet-name>").append(servletName).append("</servlet-name>\n");
        xml.append("        <servlet-class>").append(servletClass).append("</servlet-class>\n");
        xml.append("    </servlet>\n\n");

        // Map Servlet URL Pattern
        xml.append("    <servlet-mapping>\n");
        xml.append("        <servlet-name>").append(servletName).append("</servlet-name>\n");
        xml.append("        <url-pattern>").append(urlPattern).append("</url-pattern>\n");
        xml.append("    </servlet-mapping>\n\n");

        // Welcome File List
        xml.append("    <welcome-file-list>\n");
        xml.append("        <welcome-file>index.jsp</welcome-file>\n");
        xml.append("        <welcome-file>index.html</welcome-file>\n");
        xml.append("    </welcome-file-list>\n\n");

        xml.append("</web-app>");
        return xml.toString();
    }

    public static void main(String[] args) {
        File webXmlFile = new File("scratch/web_app/WEB-INF/web.xml");
        webXmlFile.getParentFile().mkdirs();

        String xmlContent = generateWebXmlContent(
                "Student Portal Web App",
                "RegistrationServlet",
                "com.portal.RegistrationServlet",
                "/register"
        );

        try (PrintWriter writer = new PrintWriter(webXmlFile)) {
            writer.print(xmlContent);
            System.out.println("Successfully generated deployment descriptor at: " + webXmlFile.getAbsolutePath());
            System.out.println("\nGenerated web.xml Markup:\n" + xmlContent);
        } catch (IOException e) {
            System.err.println("Failed to write web.xml: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **`web.xml` Schema Generation**:
   - Generates compliant Java EE 4.0 XML deployment descriptor markup mapping `/register` to `com.portal.RegistrationServlet`.

---

## 2. Practical Program: Tomcat WAR Structure Inspector & Auditor

### `TomcatWarStructureVerifier.java`

```java
import java.io.File;

/**
 * Diagnostic tool auditing web application directory layouts for Tomcat deployment compliance.
 */
public class TomcatWarStructureVerifier {

    public static void verifyWebLayout(File appRoot) {
        System.out.println("==========================================");
        System.out.println("  TOMCAT WEB APP STRUCTURE VERIFIER      ");
        System.out.println("==========================================");
        System.out.println("Auditing Application Root: " + appRoot.getAbsolutePath());

        boolean rootExists = appRoot.exists() && appRoot.isDirectory();
        System.out.println("  -> App Root Directory Exists? : " + rootExists);

        if (!rootExists) {
            System.err.println("CRITICAL ERROR: Application root folder missing.");
            return;
        }

        File webInf = new File(appRoot, "WEB-INF");
        File webXml = new File(webInf, "web.xml");
        File classesDir = new File(webInf, "classes");
        File libDir = new File(webInf, "lib");
        File indexJsp = new File(appRoot, "index.jsp");

        System.out.println("  -> WEB-INF Private Folder?    : " + (webInf.exists() && webInf.isDirectory()));
        System.out.println("  -> WEB-INF/web.xml File?     : " + webXml.exists());
        System.out.println("  -> WEB-INF/classes Folder?   : " + (classesDir.exists() && classesDir.isDirectory()));
        System.out.println("  -> WEB-INF/lib Folder?       : " + (libDir.exists() && libDir.isDirectory()));
        System.out.println("  -> Public index.jsp File?    : " + indexJsp.exists());

        System.out.println("------------------------------------------");
        if (webInf.exists() && (webXml.exists() || classesDir.exists())) {
            System.out.println("VERDICT: Web Application Layout is Tomcat Compliant!");
        } else {
            System.out.println("VERDICT: Incomplete Web Application Layout.");
        }
        System.out.println("==========================================");
    }

    public static void main(String[] args) {
        File mockApp = new File("scratch/web_app");
        new File(mockApp, "WEB-INF/classes").mkdirs();
        new File(mockApp, "WEB-INF/lib").mkdirs();

        verifyWebLayout(mockApp);
    }
}
```

### Explanation

1. **WAR Directory Audit**:
   - Inspects `WEB-INF/`, `WEB-INF/web.xml`, `WEB-INF/classes/`, and `WEB-INF/lib/` to verify compliance with Apache Tomcat application layout standards.
