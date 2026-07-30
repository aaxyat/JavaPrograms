# Unit 8: Overview of Servlet and JSP

## Sample Program of Servlet and JSP

This section presents **3 complete, end-to-end, MVC-compliant web applications** combining Servlets (Controllers), Java Beans (Models), and JSPs (Views). Each example includes full source code followed by an in-depth, line-by-line code explanation.

---

## 1. User Login & Session Authentication Web System

### Source Code

#### `login.jsp` (Public Login Form View)
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <title>System Login Portal</title>
    <style>
        body { font-family: sans-serif; background: #f1f5f9; padding: 40px; }
        .card { max-width: 360px; margin: 0 auto; background: white; padding: 25px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .error { color: #dc2626; font-size: 14px; margin-bottom: 12px; }
        input[type="text"], input[type="password"] { width: 100%; padding: 8px; margin: 6px 0 16px 0; box-sizing: border-box; }
        button { width: 100%; padding: 10px; background: #2563eb; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>

<div class="card">
    <h2>Secure Login</h2>
    
    <%-- Render Error Message if Present --%>
    <%
        String errorMessage = (String) request.getAttribute("errorMessage");
        if (errorMessage != null) {
    %>
        <div class="error"><%= errorMessage %></div>
    <%
        }
    %>

    <form action="login" method="POST">
        <label>Username:</label>
        <input type="text" name="username" required>

        <label>Password:</label>
        <input type="password" name="password" required>

        <button type="submit">Log In</button>
    </form>
</div>

</body>
</html>
```

#### `LoginServlet.java` (Authentication Controller)
```java
package com.app;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet(name = "LoginServlet", urlPatterns = {"/login"})
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // Authenticate mock credentials (admin / admin123)
        if ("admin".equals(username) && "admin123".equals(password)) {
            // Create New HTTP Session
            HttpSession session = request.getSession(true);
            session.setAttribute("user", username);
            session.setAttribute("role", "ADMINISTRATOR");

            // Redirect to Protected Dashboard View
            response.sendRedirect("dashboard.jsp");
        } else {
            // Forward back to Login Page with Error Attribute
            request.setAttribute("errorMessage", "Invalid username or password!");
            request.getRequestDispatcher("login.jsp").forward(request, response);
        }
    }
}
```

#### `dashboard.jsp` (Protected User View)
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    // Verify Session Authentication
    String user = (String) session.getAttribute("user");
    String role = (String) session.getAttribute("role");

    if (user == null) {
        // Redirect Unauthenticated Users back to Login Page
        response.sendRedirect("login.jsp");
        return;
    }
%>
<!DOCTYPE html>
<html>
<head>
    <title>User Dashboard</title>
    <style>
        body { font-family: sans-serif; padding: 30px; }
        .box { background: #e0f2fe; border-left: 4px solid #0284c7; padding: 15px; }
    </style>
</head>
<body>

    <h2>Welcome to Your Dashboard, <%= user %>!</h2>
    
    <div class="box">
        <p><strong>Active Session User:</strong> <%= user %></p>
        <p><strong>Assigned Role:</strong> <%= role %></p>
        <p><strong>Session ID:</strong> <%= session.getId() %></p>
    </div>

    <br>
    <form action="logout" method="POST">
        <button type="submit">Log Out</button>
    </form>

</body>
</html>
```

#### `LogoutServlet.java` (Session Termination Controller)
```java
package com.app;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet(name = "LogoutServlet", urlPatterns = {"/logout"})
public class LogoutServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        // Invalidate Current HTTP Session
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }

        // Redirect back to Login Page
        response.sendRedirect("login.jsp");
    }
}
```

### Explanation

- **`login.jsp` Scriptlet & Expression**: Reads `request.getAttribute("errorMessage")`. If present, renders a red error div dynamically using `<%= errorMessage %>`.
- **`LoginServlet.java` Authentication Logic**: `doPost()` reads credentials. On success, it creates an `HttpSession` via `request.getSession(true)`, sets user attributes, and redirects to `dashboard.jsp`. On failure, it sets request attributes and forwards back to `login.jsp`.
- **`dashboard.jsp` Session Guard**: Checks `session.getAttribute("user")`. If `null`, it redirects unauthenticated users to `login.jsp` using `response.sendRedirect()`.
- **`LogoutServlet.java` Invalidation**: Calls `session.invalidate()` to purge session attributes from server memory.

---

## 2. Online Product Catalog & Shopping Cart App

### Source Code

#### `Product.java` (Model Java Bean)
```java
package com.app;

import java.io.Serializable;

public class Product implements Serializable {
    private static final long serialVersionUID = 8001L;

    private String id;
    private String name;
    private double price;

    public Product(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

#### `catalog.jsp` (Product Selection View)
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <title>E-Commerce Catalog</title>
    <style>
        body { font-family: sans-serif; padding: 25px; }
        .product { border: 1px solid #cbd5e1; padding: 12px; margin-bottom: 10px; max-width: 400px; border-radius: 6px; }
    </style>
</head>
<body>

    <h2>Product Storefront Catalog</h2>
    <a href="cart.jsp">View Shopping Cart</a>
    <hr>

    <div class="product">
        <h3>Wireless Headphones</h3>
        <p>Price: $50.00</p>
        <form action="cart" method="POST">
            <input type="hidden" name="id" value="P1">
            <input type="hidden" name="name" value="Wireless Headphones">
            <input type="hidden" name="price" value="50.00">
            <button type="submit">Add to Cart</button>
        </form>
    </div>

    <div class="product">
        <h3>Mechanical Keyboard</h3>
        <p>Price: $85.00</p>
        <form action="cart" method="POST">
            <input type="hidden" name="id" value="P2">
            <input type="hidden" name="name" value="Mechanical Keyboard">
            <input type="hidden" name="price" value="85.00">
            <button type="submit">Add to Cart</button>
        </form>
    </div>

</body>
</html>
```

#### `CartServlet.java` (Cart Management Controller)
```java
package com.app;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@WebServlet(name = "CartServlet", urlPatterns = {"/cart"})
public class CartServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        String id = request.getParameter("id");
        String name = request.getParameter("name");
        double price = Double.parseDouble(request.getParameter("price"));

        Product product = new Product(id, name, price);

        // Retrieve or Create Cart List in Session
        HttpSession session = request.getSession(true);
        @SuppressWarnings("unchecked")
        List<Product> cart = (List<Product>) session.getAttribute("cartList");

        if (cart == null) {
            cart = new ArrayList<>();
            session.setAttribute("cartList", cart);
        }

        cart.add(product);

        // Redirect to Shopping Cart View
        response.sendRedirect("cart.jsp");
    }
}
```

#### `cart.jsp` (Shopping Cart View)
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" import="java.util.List,com.app.Product" %>
<!DOCTYPE html>
<html>
<head>
    <title>Shopping Cart</title>
    <style>
        body { font-family: sans-serif; padding: 25px; }
        table { border-collapse: collapse; width: 450px; }
        th, td { border: 1px solid #94a3b8; padding: 8px; text-align: left; }
        th { background: #f1f5f9; }
    </style>
</head>
<body>

    <h2>Your Shopping Cart</h2>
    <a href="catalog.jsp">Continue Shopping</a>
    <br><br>

    <%
        List<Product> cart = (List<Product>) session.getAttribute("cartList");
        if (cart == null || cart.isEmpty()) {
    %>
        <p>Your shopping cart is currently empty.</p>
    <%
        } else {
            double grandTotal = 0.0;
    %>
        <table>
            <tr><th>Product ID</th><th>Product Name</th><th>Price</th></tr>
            <%
                for (Product p : cart) {
                    grandTotal += p.getPrice();
            %>
                <tr>
                    <td><%= p.getId() %></td>
                    <td><%= p.getName() %></td>
                    <td>$<%= String.format("%.2f", p.getPrice()) %></td>
                </tr>
            <%
                }
            %>
            <tr>
                <td colspan="2"><strong>Grand Total:</strong></td>
                <td><strong>$<%= String.format("%.2f", grandTotal) %></strong></td>
            </tr>
        </table>
    <%
        }
    %>

</body>
</html>
```

### Explanation

- **`Product.java` Model**: Java Bean holding item ID, name, and price.
- **`CartServlet.java` Session Cart Management**: Stores added products inside `List<Product> cartList` bound to `HttpSession`.
- **`cart.jsp` Dynamic Table Rendering**: Iterates through `cartList`, computing subtotal amounts and rendering cart item rows dynamically.

---

## 3. Student Registration & Grade Evaluation Portal

### Source Code

#### `Student.java` (Model Class)
```java
package com.app;

import java.io.Serializable;

public class Student implements Serializable {
    private static final long serialVersionUID = 8002L;

    private String name;
    private double percentage;
    private String grade;

    public Student(String name, double percentage, String grade) {
        this.name = name;
        this.percentage = percentage;
        this.grade = grade;
    }

    public String getName() { return name; }
    public double getPercentage() { return percentage; }
    public String getGrade() { return grade; }
}
```

#### `RegistrationServlet.java` (Controller)
```java
package com.app;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "RegistrationServlet", urlPatterns = {"/register-student"})
public class RegistrationServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        String name = request.getParameter("name");
        double math = Double.parseDouble(request.getParameter("math"));
        double science = Double.parseDouble(request.getParameter("science"));
        double english = Double.parseDouble(request.getParameter("english"));

        // Calculate Percentage and Grade
        double total = math + science + english;
        double percentage = total / 3.0;

        String grade;
        if (percentage >= 90) grade = "A+";
        else if (percentage >= 80) grade = "A";
        else if (percentage >= 70) grade = "B";
        else if (percentage >= 60) grade = "C";
        else grade = "F";

        Student student = new Student(name, percentage, grade);

        // Pass Model Object as Request Attribute
        request.setAttribute("studentModel", student);

        // Forward to View JSP
        request.getRequestDispatcher("result.jsp").forward(request, response);
    }
}
```

#### `register.html` (Input Form)
```html
<!DOCTYPE html>
<html>
<head>
    <title>Student Registration</title>
    <style>
        body { font-family: sans-serif; padding: 30px; }
        .form-box { max-width: 380px; }
        input { width: 100%; padding: 6px; margin: 4px 0 12px 0; }
        button { padding: 8px 16px; background: #16a34a; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>

    <h2>Student Grade Evaluator</h2>
    <div class="form-box">
        <form action="register-student" method="POST">
            <label>Student Full Name:</label>
            <input type="text" name="name" required>

            <label>Math Marks (0-100):</label>
            <input type="number" name="math" min="0" max="100" required>

            <label>Science Marks (0-100):</label>
            <input type="number" name="science" min="0" max="100" required>

            <label>English Marks (0-100):</label>
            <input type="number" name="english" min="0" max="100" required>

            <button type="submit">Evaluate Grade</button>
        </form>
    </div>

</body>
</html>
```

#### `result.jsp` (Report Card View)
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" import="com.app.Student" %>
<!DOCTYPE html>
<html>
<head>
    <title>Student Grade Report</title>
    <style>
        body { font-family: sans-serif; padding: 30px; }
        .report { background: #f8fafc; border: 1px solid #cbd5e1; padding: 20px; max-width: 400px; border-radius: 6px; }
        .pass { color: #16a34a; font-weight: bold; }
        .fail { color: #dc2626; font-weight: bold; }
    </style>
</head>
<body>

    <h2>Academic Evaluation Report</h2>

    <%
        Student student = (Student) request.getAttribute("studentModel");
        if (student != null) {
    %>
        <div class="report">
            <p><strong>Student Name:</strong> <%= student.getName() %></p>
            <p><strong>Overall Percentage:</strong> <%= String.format("%.2f%%", student.getPercentage()) %></p>
            <p><strong>Assigned Grade:</strong> 
                <span class="<%= student.getGrade().equals("F") ? "fail" : "pass" %>">
                    <%= student.getGrade() %>
                </span>
            </p>
        </div>
    <%
        } else {
    %>
        <p>No evaluation report found.</p>
    <%
        }
    %>
    <br>
    <a href="register.html">Evaluate Another Student</a>

</body>
</html>
```

### Explanation

- **`RegistrationServlet.java` Controller**: Processes marks, calculates percentages and letter grades, packages data inside a `Student` model object, and forwards to `result.jsp`.
- **`result.jsp` View**: Extracts `request.getAttribute("studentModel")` and renders formatted academic report cards dynamically.
