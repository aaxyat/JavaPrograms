# Unit 8: Overview of Servlet and JSP

## Introduction to Servlet and JSP and its Architecture

**Java Servlets** and **JavaServer Pages (JSP)** form the foundation of Java enterprise web application development. Servlets provide Java-based server-side request processing logic, while JSPs allow developers to embed dynamic Java content directly inside HTML web pages.

---

### Core Concepts

#### 1. What is a Java Servlet?
A **Servlet** is a Java class executing inside a Web Container (such as Apache Tomcat) that extends `javax.servlet.http.HttpServlet` (or `jakarta.servlet.http.HttpServlet`). Servlets receive incoming HTTP requests (`HttpServletRequest`), process business logic or database queries, and return dynamic HTTP responses (`HttpServletResponse`).

#### 2. What is JavaServer Pages (JSP)?
**JSP** is a server-side template technology allowing developers to write standard HTML markup combined with dynamic JSP tags:
- **Scriptlets (`<% ... %>`)**: Enclose raw Java code statements executed on the server.
- **Expressions (`<%= ... %>`)**: Evaluate Java expressions and output the resulting string directly into HTML responses.
- **Declarations (`<%! ... %>`)**: Declare class-level variables or helper methods.
- **Directives (`<%@ page ... %>`)**: Configure page attributes (e.g. `<%@ page contentType="text/html;charset=UTF-8" %>`).

> **Key Architectural Insight**: JSPs are not interpreted HTML files. When a user requests a `.jsp` page for the first time, the Web Container automatically translates the JSP into a Java Servlet class (`.java`), compiles it into bytecode (`.class`), and executes it as a standard Servlet.

#### 3. Servlet & JSP Request Processing Architecture

```
  Web Browser (Client)                     Web Container (Apache Tomcat)
  ┌──────────────────┐  1. HTTP Request    ┌───────────────────────────────────┐
  │  Chrome / Edge   │ ──────────────────> │  2. Inspect URL & Map to Servlet  │
  └────────▲─────────┘                     └─────────────────┬─────────────────┘
           │                                                 │
           │                                ┌────────────────▼──────────────────┐
           │                                │  3. Execute Servlet Service Method│
           │                                │     (Controller Logic / Database) │
           │                                └────────────────┬──────────────────┘
           │                                                 │
           │                                ┌────────────────▼──────────────────┐
           │ 4. HTTP Response               │  4. Forward to JSP Template       │
           │    (Dynamic HTML Payload)      │     (Renders View & HTML Response)│
           └─────────────────────────────── └───────────────────────────────────┘
```

#### 4. The Servlet Lifecycle

A Servlet transitions through three primary lifecycle phases managed automatically by the Web Container:

```
  [ Container Starts ] ──> init(ServletConfig) ──> service(req, res) ──> destroy() ──> [ Unloaded ]
                             (Called once)       (Called per request)  (Called once)
```

1. **`init(ServletConfig config)`**: Invoked once when the container loads and instantiates the Servlet. Used for initialization tasks like opening database connection pools or reading configuration parameters.
2. **`service(HttpServletRequest req, HttpServletResponse res)`**: Invoked for every incoming HTTP request. Dispatches requests to specific handler methods (`doGet()`, `doPost()`, `doPut()`, `doDelete()`) based on the HTTP method.
3. **`destroy()`**: Invoked once when the container unloads the Servlet (such as during server shutdown) to release system resources.

#### 5. MVC (Model-View-Controller) Architecture Pattern

| Component | Architecture Role | Technology Used | Responsibility |
|-----------|-------------------|-----------------|----------------|
| **Model** | Business State & Data | Java Beans / POJOs | Represents business logic, data models, and database operations. |
| **Controller** | Request Processor | Java Servlet | Intercepts HTTP requests, validates form inputs, updates the Model, and routes requests. |
| **View** | User Interface | JSP / HTML | Renders dynamic HTML content to the client browser based on data stored in the Model. |

---

### Common Pitfalls

1. **Multithreading Race Conditions in Servlets**:
   - The Web Container creates only **one single instance** of each Servlet class in memory, creating a new thread for every concurrent HTTP request. Storing user-specific data in Servlet instance variables (`private String username;`) causes severe cross-user data leakage. Store state in local method variables or HTTP sessions.
2. **Writing Business Logic Inside JSP Files**:
   - Placing database connections or complex calculations inside JSP scriptlets (`<% ... %>`) violates MVC separation of concerns, making web applications difficult to debug and maintain. Keep business logic in Servlets and Models.
3. **Forgetting `RequestDispatcher.forward()` vs `response.sendRedirect()`**:
   - `forward()` happens server-side within the same request scope, keeping request attributes intact. `sendRedirect()` sends an HTTP `302` response instructing the browser to issue a brand-new HTTP request, losing original request attributes.

---

## 1. Demo: HttpServlet Lifecycle and Request Handler

### `ServletArchitectureDemo.java`

```java
import javax.servlet.ServletException;
import javax.servlet.ServletConfig;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;
import java.io.IOException;
import java.util.Date;

/**
 * Demonstrates HttpServlet lifecycle methods (init, doGet, doPost, destroy) and request parameter extraction.
 */
public class ServletArchitectureDemo extends HttpServlet {

    private String serverName;
    private int requestCounter;

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        serverName = "Enterprise Java Servlet Container";
        requestCounter = 0;
        System.out.println("[SERVLET LIFECYCLE] init() executed -> Initialized " + serverName);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        synchronized (this) {
            requestCounter++; // Increment request count
        }

        // Set response content type and character encoding
        response.setContentType("text/html;charset=UTF-8");

        // Extract HTTP GET Request Parameters
        String userParam = request.getParameter("name");
        if (userParam == null || userParam.trim().isEmpty()) {
            userParam = "Guest User";
        }

        // Render HTML response directly from Servlet
        try (PrintWriter out = response.getWriter()) {
            out.println("<!DOCTYPE html>");
            out.println("<html><head><title>Servlet Architecture Demo</title></head>");
            out.println("<body style='font-family: sans-serif; padding: 20px;'>");
            out.println("<h2>Servlet Architecture & Lifecycle Demo</h2>");
            out.println("<p><strong>Server Name:</strong> " + serverName + "</p>");
            out.println("<p><strong>Greeting:</strong> Hello, " + userParam + "!</p>");
            out.println("<p><strong>Total Requests Processed:</strong> " + requestCounter + "</p>");
            out.println("<p><strong>Server Timestamp:</strong> " + new Date() + "</p>");
            out.println("<hr>");
            out.println("<form method='POST' action='ServletArchitectureDemo'>");
            out.println("  <label>Update Status Message: </label>");
            out.println("  <input type='text' name='status' placeholder='Enter status'>");
            out.println("  <button type='submit'>Submit POST</button>");
            out.println("</form>");
            out.println("</body></html>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        response.setContentType("text/html;charset=UTF-8");
        String statusParam = request.getParameter("status");

        try (PrintWriter out = response.getWriter()) {
            out.println("<!DOCTYPE html>");
            out.println("<html><body>");
            out.println("<h3>HTTP POST Request Handled Successfully</h3>");
            out.println("<p>Received Status Parameter: <strong>" + statusParam + "</strong></p>");
            out.println("<a href='ServletArchitectureDemo'>Back to GET Form</a>");
            out.println("</body></html>");
        }
    }

    @Override
    public void destroy() {
        System.out.println("[SERVLET LIFECYCLE] destroy() executed -> Releasing Servlet resources.");
        super.destroy();
    }
}
```

### Explanation

1. **`init(ServletConfig config)` Initialization**:
   - `init()` executes once when Tomcat loads the Servlet, setting up `serverName` and `requestCounter`.

2. **Handling `doGet()` and `doPost()`**:
   - `doGet()` extracts query parameters (`request.getParameter("name")`), sets HTML response headers, and renders output.
   - `doPost()` processes submitted form data separately.

3. **Thread Safety (`synchronized`)**:
   - `synchronized (this)` guards `requestCounter++` against thread race conditions across concurrent client HTTP requests.

---

## 2. Practical Program: MVC Architecture Request Auditor

### `MvcWebArchitectureAuditor.java`

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

/**
 * Controller Servlet demonstrating MVC request attribute assignment, session handling, and view forwarding.
 */
@WebServlet(name = "MvcWebArchitectureAuditor", urlPatterns = {"/audit-request"})
public class MvcWebArchitectureAuditor extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        System.out.println("[MVC CONTROLLER] Intercepted incoming HTTP GET request.");

        // 1. Extract Request Metadata
        String clientIp = request.getRemoteAddr();
        String userAgent = request.getHeader("User-Agent");
        String queryString = request.getQueryString();

        // 2. Set Request Scope Attributes (Model Data for View)
        request.setAttribute("auditClientIp", clientIp);
        request.setAttribute("auditUserAgent", userAgent);
        request.setAttribute("auditQueryString", (queryString == null ? "None" : queryString));

        // 3. Manage HTTP Session Scope Attribute
        HttpSession session = request.getSession(true);
        Integer visitCount = (Integer) session.getAttribute("sessionVisitCount");
        if (visitCount == null) {
            visitCount = 1;
        } else {
            visitCount++;
        }
        session.setAttribute("sessionVisitCount", visitCount);

        System.out.println("[MVC CONTROLLER] Forwarding model data to JSP View (/audit-result.jsp)...");

        // 4. Server-Side Request Forwarding to JSP View
        request.getRequestDispatcher("/audit-result.jsp").forward(request, response);
    }
}
```

### Explanation

1. **Annotation Configuration (`@WebServlet`)**:
   - `@WebServlet(urlPatterns = {"/audit-request"})` maps the URL endpoint dynamically without XML configuration.

2. **Scope Management**:
   - `request.setAttribute()` attaches short-lived data for the current request. `session.setAttribute()` preserves data across multiple client requests.
