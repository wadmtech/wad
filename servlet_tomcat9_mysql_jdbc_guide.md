# Java Servlets + Apache Tomcat 9 + MySQL JDBC

## From-scratch guide, JDBC reference, and practical code snippets

This guide combines the basic Servlet application setup with a detailed
JDBC reference for `Connection`, `Statement`, `PreparedStatement`,
`ResultSet`, transactions, CRUD operations, and additional connection
examples.

> **Compatibility note:** Apache Tomcat 9 implements Servlet 4.0 and
> uses `javax.servlet.*`. Do not use `jakarta.servlet.*` imports in this
> Tomcat 9 example.

------------------------------------------------------------------------

## 1. Architecture: how the pieces work together

1.  The browser loads `index.html`.
2.  The user submits a form using HTTP GET or POST.
3.  Tomcat receives the request and maps the URL to a Servlet using
    `WEB-INF/web.xml`.
4.  The Servlet reads form fields with `request.getParameter(...)`.
5.  JDBC connects to MySQL.
6.  Java executes SQL through a `Statement` or `PreparedStatement`.
7.  A `ResultSet` is used to read rows returned by a `SELECT`.
8.  The Servlet writes an HTTP response using `response.getWriter()`.

  Technology          Responsibility
  ------------------- ------------------------------------------
  HTML                User interface and form
  Servlet             Handles HTTP requests and responses
  Tomcat 9            Runs the web application
  JDBC                Java API for database access
  MySQL               Stores data
  MySQL Connector/J   JDBC driver that communicates with MySQL

## 2. Folder structure

For a manually deployed, non-Maven application:

``` text
apache-tomcat-9.x/
└── webapps/
    └── StudentApp/
        ├── index.html
        └── WEB-INF/
            ├── web.xml
            ├── classes/
            │   └── StudentServlet.class
            └── lib/
                └── mysql-connector-j-<version>.jar
```

Keep `StudentServlet.java` in a source/development directory. Compile it
into `WEB-INF/classes`.

-   `index.html`: form shown to the user.
-   `WEB-INF/web.xml`: Servlet declaration and URL mapping.
-   `WEB-INF/classes`: compiled Java classes.
-   `WEB-INF/lib`: application-specific JARs, including MySQL
    Connector/J.
-   Files under `WEB-INF` cannot be requested directly by a browser.

## 3. Install and verify prerequisites

You need: - A JDK, including `java` and `javac` - Apache Tomcat 9 -
MySQL Server - MySQL Connector/J

Verify Java:

``` bash
java -version
javac -version
```

Download Tomcat 9 from <https://tomcat.apache.org/download-90.cgi>.

Download MySQL Connector/J from
<https://dev.mysql.com/downloads/connector/j/>.

Place the downloaded Connector/J JAR in:

``` text
StudentApp/WEB-INF/lib/
```

Choose a Connector/J release compatible with your Java and MySQL
versions.

## 4. Create the database

Run in MySQL Workbench or the MySQL command-line client:

``` sql
CREATE DATABASE servlet_db;
USE servlet_db;

CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL
);

SHOW TABLES;
DESC students;
SELECT * FROM students;
```

Example connection settings used throughout this guide:

``` text
Host: localhost
Port: 3306
Database: servlet_db
Username: root
Password: YOUR_MYSQL_PASSWORD
```

Replace the password with your actual local MySQL password. Do not
assume the password is `root`.

## 5. `index.html`

Save as `StudentApp/index.html`:

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Student Registration</title>
</head>
<body>
    <h1>Student Registration</h1>

    <form action="StudentServlet" method="post">
        <label for="name">Student Name:</label>
        <input type="text" id="name" name="name" required>

        <br><br>

        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>

        <br><br>
        <button type="submit">Register</button>
    </form>
</body>
</html>
```

Important points: - `action="StudentServlet"` is the URL path mapped to
the Servlet. - `method="post"` submits the form in the HTTP request
body. - `name="name"` and `name="email"` are the parameter names Java
reads. - `required` is browser-side validation; Java must still validate
input.

If the app is opened at `http://localhost:8080/StudentApp/`, the form
posts to `http://localhost:8080/StudentApp/StudentServlet`.

## 6. `WEB-INF/web.xml`

For Tomcat 9, use the Servlet 4.0 Java EE namespace:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
    xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
        http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
    version="4.0">

    <servlet>
        <servlet-name>StudentServlet</servlet-name>
        <servlet-class>StudentServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>StudentServlet</servlet-name>
        <url-pattern>/StudentServlet</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>
</web-app>
```

How it works: - `<servlet-name>` is a logical name used in the
configuration. - `<servlet-class>` names the Java class. This example
has no package, so the name is `StudentServlet`. - `<url-pattern>` maps
`/StudentServlet` to the declared Servlet. - The context path
(`/StudentApp`) is not included in `<url-pattern>`. - The welcome-file
makes the app root load `index.html`.

## 7. Servlet basics

A Servlet is a Java class managed by Tomcat. You do not write a `main()`
method for a normal Servlet.

``` java
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class StudentServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {
        // Handle GET
    }

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {
        // Handle POST
    }
}
```

-   `HttpServlet`: base class for HTTP Servlets.
-   `HttpServletRequest`: incoming request data.
-   `HttpServletResponse`: outgoing response.
-   `doGet()`: handles GET requests.
-   `doPost()`: handles POST requests.
-   `request.getParameter("name")`: reads a form parameter.
-   `response.setContentType(...)`: sets response content type.
-   `response.getWriter()`: writes text/HTML to the browser.

Tomcat manages the lifecycle: loading/instantiation, `init()`, request
handling through `service()` (which dispatches to `doGet()`/`doPost()`),
and eventually `destroy()`. A Servlet instance may handle multiple
requests, so do not store per-request data in mutable instance fields.

## 8. Compile and deploy manually

Assume `StudentServlet.java` is in your current directory and Tomcat is
extracted.

### macOS/Linux

Set your actual Tomcat path:

``` bash
export TOMCAT_HOME="/path/to/apache-tomcat-9.x"

mkdir -p "$TOMCAT_HOME/webapps/StudentApp/WEB-INF/classes"

javac \
  -cp "$TOMCAT_HOME/lib/servlet-api.jar" \
  -d "$TOMCAT_HOME/webapps/StudentApp/WEB-INF/classes" \
  StudentServlet.java
```

### Windows Command Prompt

``` bat
set "TOMCAT_HOME=C:\apache-tomcat-9.x"

mkdir "%TOMCAT_HOME%\webapps\StudentApp\WEB-INF\classes"

javac -cp "%TOMCAT_HOME%\lib\servlet-api.jar" -d "%TOMCAT_HOME%\webapps\StudentApp\WEB-INF\classes" StudentServlet.java
```

`javac` compiles `.java` into `.class`. The compiler needs Tomcat's
`servlet-api.jar` to resolve `javax.servlet` types. Do not copy
`servlet-api.jar` into `WEB-INF/lib`; Tomcat supplies it at runtime.

## 9. Start MySQL and Tomcat

Start MySQL using your operating system's service manager, MySQL
Workbench, or the method used by your installation.

Start Tomcat:

macOS/Linux:

``` bash
"$TOMCAT_HOME/bin/startup.sh"
```

Windows:

``` bat
cd C:\apache-tomcat-9.x\bin
startup.bat
```

Test: - Tomcat home: <http://localhost:8080/> - App root:
<http://localhost:8080/StudentApp/> - Explicit HTML:
<http://localhost:8080/StudentApp/index.html>

Stop Tomcat:

macOS/Linux:

``` bash
"$TOMCAT_HOME/bin/shutdown.sh"
```

Windows:

``` bat
shutdown.bat
```

If you change `.java`, recompile it. If you change deployment
configuration, redeploy or restart as needed.

------------------------------------------------------------------------

# Part II --- JDBC from scratch

## 10. The JDBC workflow

The usual JDBC sequence is:

1.  Load the driver if needed.
2.  Open a `Connection`.
3.  Create a `Statement`, `PreparedStatement`, or `CallableStatement`.
4.  Bind parameters if using `PreparedStatement`.
5.  Execute SQL.
6.  Process a `ResultSet` if the query returns rows.
7.  Close resources.

Example:

``` java
Class.forName("com.mysql.cj.jdbc.Driver");

String url = "jdbc:mysql://localhost:3306/servlet_db";
String user = "root";
String password = "YOUR_MYSQL_PASSWORD";

try (Connection con =
         DriverManager.getConnection(url, user, password);
     PreparedStatement ps =
         con.prepareStatement("SELECT id, name, email FROM students");
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getInt("id"));
        System.out.println(rs.getString("name"));
        System.out.println(rs.getString("email"));
    }
}
```

Modern JDBC drivers usually register automatically, so
`Class.forName(...)` is often unnecessary. It is still common in
beginner examples to show how the driver class is loaded explicitly.

## 11. `DriverManager` and connection URLs

### `DriverManager.getConnection()`

``` java
Connection con = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/servlet_db",
    "root",
    "YOUR_MYSQL_PASSWORD"
);
```

Arguments: 1. JDBC URL 2. Database username 3. Database password

URL breakdown:

``` text
jdbc:mysql://localhost:3306/servlet_db
    │      │       │        │       └─ database
    │      │       │        └───────── port
    │      │       └────────────────── host
    │      └────────────────────────── MySQL JDBC protocol
    └───────────────────────────────── JDBC
```

Common connection URL examples:

``` java
// Local MySQL, default port
String url = "jdbc:mysql://localhost:3306/servlet_db";

// Explicit loopback address
String url2 = "jdbc:mysql://127.0.0.1:3306/servlet_db";

// MySQL on another host (replace with a real hostname)
String url3 = "jdbc:mysql://db.example.com:3306/servlet_db";

// URL with common options; configure SSL/time-zone settings appropriately
String url4 =
    "jdbc:mysql://localhost:3306/servlet_db?serverTimezone=UTC";
```

Do not use a hostname or URL option blindly. Configure remote database
access and TLS based on your environment. Never expose database
credentials in browser code.

### Store credentials outside source code (recommended)

For a simple local learning example, constants demonstrate the API, but
real applications should not hard-code secrets. For example, use
environment variables:

``` java
String url = System.getenv("DB_URL");
String user = System.getenv("DB_USER");
String password = System.getenv("DB_PASSWORD");

if (url == null || user == null || password == null) {
    throw new IllegalStateException("Database configuration is missing");
}

Connection con = DriverManager.getConnection(url, user, password);
```

Set those environment variables in the environment where Tomcat runs. A
terminal variable set only in one shell may not be inherited by a
separately launched Tomcat process.

### Connection validity and state

``` java
boolean closed = con.isClosed();
boolean autoCommit = con.getAutoCommit();
boolean valid = con.isValid(2);
```

-   `isClosed()`: checks whether the connection has been closed.
-   `getAutoCommit()`: checks auto-commit mode.
-   `isValid(timeoutSeconds)`: asks the driver whether the connection is
    valid, subject to driver support.

Do not use these as a substitute for handling `SQLException` during real
operations.

## 12. `Connection`: important methods

  Method                               Purpose
  ------------------------------------ -------------------------------------
  `DriverManager.getConnection(...)`   Obtain a JDBC connection
  `createStatement()`                  Create a `Statement`
  `prepareStatement(sql)`              Create a parameterized statement
  `prepareCall(sql)`                   Create a `CallableStatement`
  `setAutoCommit(false)`               Begin explicit transaction control
  `getAutoCommit()`                    Check auto-commit mode
  `commit()`                           Commit a transaction
  `rollback()`                         Roll back uncommitted changes
  `isClosed()`                         Check whether closed
  `isValid(seconds)`                   Ask whether the connection is valid
  `getMetaData()`                      Obtain database metadata
  `close()`                            Release the connection

### Simple connection test

``` java
try (Connection con = DriverManager.getConnection(url, user, password)) {
    System.out.println("Connected: " + !con.isClosed());
    System.out.println(con.getMetaData().getDatabaseProductName());
}
```

### Reuse one connection for several statements in one operation

``` java
try (Connection con = DriverManager.getConnection(url, user, password)) {
    try (PreparedStatement ps1 =
             con.prepareStatement("SELECT COUNT(*) FROM students");
         ResultSet rs = ps1.executeQuery()) {

        if (rs.next()) {
            System.out.println("Student count: " + rs.getInt(1));
        }
    }

    try (PreparedStatement ps2 =
             con.prepareStatement("SELECT name FROM students ORDER BY name");
         ResultSet rs = ps2.executeQuery()) {

        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}
```

Open one connection for the unit of work, then reuse it for multiple
statements. Do not open a new connection unnecessarily for each query
inside that same operation.

## 13. `Statement` vs `PreparedStatement` vs `CallableStatement`

### `Statement`

Use for fixed SQL without user-provided values:

``` java
try (Statement st = con.createStatement();
     ResultSet rs = st.executeQuery(
         "SELECT id, name FROM students ORDER BY name")) {

    while (rs.next()) {
        System.out.println(rs.getInt("id") + ": " + rs.getString("name"));
    }
}
```

### `PreparedStatement`

Preferred for most application SQL, especially when values are dynamic:

``` java
String sql = "SELECT id, name, email FROM students WHERE id = ?";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setInt(1, 2);

    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            System.out.println(rs.getString("name"));
        } else {
            System.out.println("Student not found");
        }
    }
}
```

### `CallableStatement`

Use for a database stored procedure:

``` java
try (CallableStatement cs = con.prepareCall("{call get_students()}");
     ResultSet rs = cs.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
}
```

A stored procedure must already exist in MySQL. Use
`registerOutParameter(...)` and the corresponding getter methods when a
procedure uses OUT parameters.

**Security rule:** Never concatenate user input into SQL. A
`PreparedStatement` binds values separately and helps prevent SQL
injection. A `?` placeholder is for a value, not a table name, column
name, or SQL keyword. For dynamic sort columns or directions, validate
against an allowlist.

## 14. `PreparedStatement`: parameter setters

Indexes start at **1**, not 0.

  Setter                         Typical use
  ------------------------------ ---------------
  `setInt(i, value)`             Integer
  `setLong(i, value)`            Long integer
  `setString(i, value)`          Text
  `setDouble(i, value)`          Double
  `setFloat(i, value)`           Float
  `setBoolean(i, value)`         Boolean
  `setDate(i, java.sql.Date)`    SQL date
  `setTimestamp(i, Timestamp)`   SQL timestamp
  `setNull(i, Types.VARCHAR)`    SQL `NULL`
  `setObject(i, value)`          General value

Example:

``` java
String sql = "INSERT INTO students (name, email) VALUES (?, ?)";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setString(1, "Arun");
    ps.setString(2, "arun@example.com");

    int rows = ps.executeUpdate();
    System.out.println(rows + " row(s) inserted");
}
```

Set SQL NULL, not the literal string `"NULL"`:

``` java
ps.setNull(2, java.sql.Types.VARCHAR);
```

The target column must permit NULL.

### Date example

``` java
java.time.LocalDate date = java.time.LocalDate.of(2026, 10, 14);
java.sql.Date sqlDate = java.sql.Date.valueOf(date);
ps.setDate(1, sqlDate);
```

### Timestamp example

``` java
java.sql.Timestamp timestamp =
    java.sql.Timestamp.valueOf(java.time.LocalDateTime.now());
ps.setTimestamp(1, timestamp);
```

Use the setter corresponding to the SQL column type. For monetary
amounts, use an appropriate decimal type such as MySQL `DECIMAL` and
Java `BigDecimal`, rather than floating-point values.

## 15. SQL execution methods: which one to use?

  -----------------------------------------------------------------------
  Method                  Return type             Typical use
  ----------------------- ----------------------- -----------------------
  `executeQuery()`        `ResultSet`             SELECT

  `executeUpdate()`       `int`                   INSERT, UPDATE, DELETE,
                                                  and applicable DDL

  `execute()`             `boolean`               Generic execution when
                                                  result type may vary

  `addBatch()`            `void`                  Add a parameter set or
                                                  SQL statement to a
                                                  batch

  `executeBatch()`        `int[]`                 Execute a batch

  `getGeneratedKeys()`    `ResultSet`             Retrieve generated keys
                                                  when
                                                  supported/requested

  `getUpdateCount()`      `int`                   Retrieve current update
                                                  count when applicable

  `clearParameters()`     `void`                  Clear bound parameter
                                                  values

  `close()`               `void`                  Release statement
                                                  resources
  -----------------------------------------------------------------------

### `executeQuery()` --- SELECT

``` java
try (PreparedStatement ps =
         con.prepareStatement("SELECT * FROM students");
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
}
```

### `executeUpdate()` --- INSERT, UPDATE, DELETE

``` java
try (PreparedStatement ps =
         con.prepareStatement("DELETE FROM students WHERE id = ?")) {

    ps.setInt(1, 5);
    int rows = ps.executeUpdate();

    System.out.println(rows + " row(s) deleted");
}
```

### `execute()` --- generic SQL

``` java
boolean hasResultSet = ps.execute();

if (hasResultSet) {
    try (ResultSet rs = ps.getResultSet()) {
        while (rs.next()) {
            // Read result
        }
    }
} else {
    int count = ps.getUpdateCount();
    System.out.println("Update count: " + count);
}
```

For ordinary CRUD code, use `executeQuery()` for SELECT and
`executeUpdate()` for INSERT, UPDATE, and DELETE.

### Generated auto-increment ID

``` java
String sql = "INSERT INTO students (name, email) VALUES (?, ?)";

try (PreparedStatement ps = con.prepareStatement(
        sql, Statement.RETURN_GENERATED_KEYS)) {

    ps.setString(1, "Arun");
    ps.setString(2, "arun@example.com");
    ps.executeUpdate();

    try (ResultSet keys = ps.getGeneratedKeys()) {
        if (keys.next()) {
            int id = keys.getInt(1);
            System.out.println("Generated ID: " + id);
        }
    }
}
```

## 16. `ResultSet`: reading query results

A `ResultSet` represents the rows returned by a query. Its cursor starts
before the first row.

``` java
try (PreparedStatement ps =
         con.prepareStatement("SELECT id, name, email FROM students");
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        int id = rs.getInt("id");
        String name = rs.getString("name");
        String email = rs.getString("email");

        System.out.println(id + " " + name + " " + email);
    }
}
```

### `if (rs.next())` vs `while (rs.next())`

Use `if` when expecting at most one row, for example a search by primary
key:

``` java
if (rs.next()) {
    System.out.println(rs.getString("name"));
} else {
    System.out.println("Student not found");
}
```

Use `while` to process every row.

### Common getter methods

  Getter                         Reads
  ------------------------------ ------------------------
  `getInt("id")`                 Integer
  `getLong("id")`                Long
  `getString("name")`            Text
  `getDouble("marks")`           Double
  `getBoolean("active")`         Boolean
  `getDate("dob")`               SQL date
  `getTimestamp("created_at")`   Timestamp
  `getObject("column")`          General object
  `getMetaData()`                Result column metadata

You can use a column label or a 1-based column index. Prefer column
names/labels for readability.

### Important ResultSet methods

  -----------------------------------------------------------------------
  Method                              Purpose
  ----------------------------------- -----------------------------------
  `next()`                            Move to next row

  `wasNull()`                         Check whether the last retrieved
                                      value was SQL NULL

  `getRow()`                          Current row number for supported
                                      cursors

  `findColumn("name")`                Find a column index

  `getMetaData()`                     Inspect returned columns

  `isBeforeFirst()` / `isAfterLast()` Inspect cursor position where
                                      meaningful

  `close()`                           Release result resources
  -----------------------------------------------------------------------

### Handling SQL NULL

Primitive getters can return a default value when the SQL value is NULL.
For example, `getInt()` returns `0` for SQL NULL.

``` java
int marks = rs.getInt("marks");

if (rs.wasNull()) {
    System.out.println("Marks unavailable");
} else {
    System.out.println("Marks: " + marks);
}
```

Call `wasNull()` immediately after the getter you want to check.

### ResultSet metadata

``` java
try (PreparedStatement ps =
         con.prepareStatement("SELECT * FROM students");
     ResultSet rs = ps.executeQuery()) {

    ResultSetMetaData md = rs.getMetaData();

    for (int i = 1; i <= md.getColumnCount(); i++) {
        System.out.println(md.getColumnLabel(i));
        System.out.println(md.getColumnTypeName(i));
    }
}
```

Useful metadata methods include `getColumnCount()`, `getColumnName(i)`,
`getColumnLabel(i)`, `getColumnTypeName(i)`, and `getColumnType(i)`.

## 17. CRUD examples

Assume the `students` table defined earlier.

### Create: INSERT

``` java
String sql = "INSERT INTO students (name, email) VALUES (?, ?)";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setString(1, "Arun");
    ps.setString(2, "arun@example.com");
    int rows = ps.executeUpdate();
}
```

### Read: SELECT all students

``` java
String sql = "SELECT id, name, email FROM students ORDER BY id";

try (PreparedStatement ps = con.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getInt("id"));
        System.out.println(rs.getString("name"));
        System.out.println(rs.getString("email"));
    }
}
```

### Read: SELECT one student by ID

``` java
String sql = "SELECT id, name, email FROM students WHERE id = ?";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setInt(1, 2);

    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            System.out.println(rs.getString("name"));
        } else {
            System.out.println("Student not found");
        }
    }
}
```

### Update

``` java
String sql = "UPDATE students SET email = ? WHERE id = ?";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setString(1, "newemail@example.com");
    ps.setInt(2, 1);

    int rows = ps.executeUpdate();
}
```

### Delete

``` java
String sql = "DELETE FROM students WHERE id = ?";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setInt(1, 1);
    int rows = ps.executeUpdate();
}
```

**Always check your `WHERE` clause.** An UPDATE or DELETE without a
suitable WHERE condition can affect every row.

## 18. Batch processing

Use batches for many similar SQL operations:

``` java
String sql = "INSERT INTO students (name, email) VALUES (?, ?)";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setString(1, "Arun");
    ps.setString(2, "arun@example.com");
    ps.addBatch();

    ps.setString(1, "Priya");
    ps.setString(2, "priya@example.com");
    ps.addBatch();

    ps.setString(1, "Ravi");
    ps.setString(2, "ravi@example.com");
    ps.addBatch();

    int[] results = ps.executeBatch();
}
```

-   `addBatch()`: adds the current parameters to the batch.
-   `executeBatch()`: executes the batch.
-   `clearBatch()`: clears pending batch entries.

Batching can improve performance, but driver behavior varies. Use
sensible batch sizes for large imports.

## 19. Transactions: commit and rollback

By default, JDBC connections typically use auto-commit mode.

Use explicit transactions when multiple operations must succeed or fail
together:

``` java
try {
    con.setAutoCommit(false);

    // Execute the first SQL operation.
    // Execute the second SQL operation.

    con.commit();

} catch (SQLException e) {
    con.rollback();
    throw e;
}
```

-   `setAutoCommit(false)`: enable explicit transaction control.
-   `commit()`: commit pending changes.
-   `rollback()`: undo uncommitted changes.
-   `setAutoCommit(true)`: restore automatic commits.

In production code, handle rollback failures too, and restore connection
state if a connection will be reused. A transaction requires a
transactional database engine such as InnoDB.

## 20. Servlet example: insert form data into MySQL

This complete example uses the HTML form and `web.xml` from earlier
sections.

``` java
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class StudentServlet extends HttpServlet {

    private static final String URL =
        "jdbc:mysql://localhost:3306/servlet_db";
    private static final String USER = "root";
    private static final String PASSWORD = "YOUR_MYSQL_PASSWORD";

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {

        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        String name = request.getParameter("name");
        String email = request.getParameter("email");

        if (name == null || name.trim().isEmpty()
                || email == null || email.trim().isEmpty()) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                               "Name and email are required");
            return;
        }

        String sql = "INSERT INTO students (name, email) VALUES (?, ?)";

        try {
            Class.forName("com.mysql.cj.jdbc.Driver");

            try (Connection con =
                     DriverManager.getConnection(URL, USER, PASSWORD);
                 PreparedStatement ps = con.prepareStatement(sql)) {

                ps.setString(1, name.trim());
                ps.setString(2, email.trim());

                int rows = ps.executeUpdate();

                try (PrintWriter out = response.getWriter()) {
                    if (rows > 0) {
                        out.println("<h1>Registration successful!</h1>");
                        out.println("<p>Name: " + escapeHtml(name) + "</p>");
                        out.println("<p>Email: " + escapeHtml(email) + "</p>");
                    } else {
                        out.println("<h1>Registration failed.</h1>");
                    }
                }
            }

        } catch (ClassNotFoundException | SQLException e) {
            getServletContext().log("Student registration failed", e);
            if (!response.isCommitted()) {
                response.sendError(
                    HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
                    "Unable to register student"
                );
            }
        }
    }

    private String escapeHtml(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#39;");
    }
}
```

For a classroom example, constants show the connection clearly. For a
real application, read credentials from protected configuration or
environment variables. Validate all user input on the server. For a
production app, prefer redirect-after-POST and a separate view/template
instead of constructing HTML directly in a Servlet.

## 21. Display database rows in the browser

`System.out.println()` writes to the server console, not the browser.
Use `response.getWriter()` for the HTTP response.

A `doGet()` method can execute SELECT and write an HTML table:

``` java
@Override
protected void doGet(HttpServletRequest request,
                     HttpServletResponse response)
        throws ServletException, IOException {

    response.setContentType("text/html;charset=UTF-8");

    String sql = "SELECT id, name, email FROM students ORDER BY id";

    try (PrintWriter out = response.getWriter()) {
        out.println("<h1>Students</h1>");
        out.println("<table border='1'>");
        out.println("<tr><th>ID</th><th>Name</th><th>Email</th></tr>");

        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement ps = con.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                out.println("<tr>");
                out.println("<td>" + rs.getInt("id") + "</td>");
                out.println("<td>" + escapeHtml(rs.getString("name")) + "</td>");
                out.println("<td>" + escapeHtml(rs.getString("email")) + "</td>");
                out.println("</tr>");
            }
        } catch (SQLException e) {
            getServletContext().log("Unable to retrieve students", e);
            if (!response.isCommitted()) {
                response.sendError(500, "Unable to retrieve students");
            }
            return;
        }

        out.println("</table>");
    }
}
```

The existing `web.xml` mapping can dispatch GET to `doGet()` and POST to
`doPost()` on the same Servlet.

## 22. CallableStatement and stored procedures

Example stored procedure in MySQL:

``` sql
DELIMITER //
CREATE PROCEDURE get_students()
BEGIN
    SELECT * FROM students;
END //
DELIMITER ;
```

Java:

``` java
try (CallableStatement cs = con.prepareCall("{call get_students()}");
     ResultSet rs = cs.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
}
```

Use `CallableStatement` when invoking stored procedures. For OUT
parameters, use `registerOutParameter(...)` before executing and then
read the output using the appropriate getter.

## 23. Database metadata

### `DatabaseMetaData`

``` java
DatabaseMetaData md = con.getMetaData();

System.out.println(md.getDatabaseProductName());
System.out.println(md.getDatabaseProductVersion());
System.out.println(md.getDriverName());
```

Other methods include `getTables()` and `getColumns()`.

### `ResultSetMetaData`

``` java
try (PreparedStatement ps = con.prepareStatement("SELECT * FROM students");
     ResultSet rs = ps.executeQuery()) {

    ResultSetMetaData md = rs.getMetaData();

    for (int i = 1; i <= md.getColumnCount(); i++) {
        System.out.println(md.getColumnLabel(i));
        System.out.println(md.getColumnTypeName(i));
    }
}
```

Useful methods: `getColumnCount()`, `getColumnName(i)`,
`getColumnLabel(i)`, `getColumnTypeName(i)`, and `getColumnType(i)`.

## 24. Scrollable ResultSets

The default ResultSet is usually forward-only. You can request a
scrollable one:

``` java
try (PreparedStatement ps = con.prepareStatement(
        "SELECT * FROM students",
        ResultSet.TYPE_SCROLL_INSENSITIVE,
        ResultSet.CONCUR_READ_ONLY);
     ResultSet rs = ps.executeQuery()) {

    if (rs.last()) {
        System.out.println("Last row number: " + rs.getRow());
    }

    rs.beforeFirst();
    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
}
```

Other navigation methods, when supported: - `previous()` - `first()` -
`last()` - `absolute(n)` - `relative(n)` - `beforeFirst()` -
`afterLast()`

Scrollable cursors are not necessary for most beginner CRUD
applications.

## 25. Resource management: try-with-resources

Prefer:

``` java
try (Connection con = DriverManager.getConnection(url, user, password);
     PreparedStatement ps = con.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        // Process rows
    }
}
```

Java automatically closes the resources in reverse declaration order,
including if an exception occurs. This prevents leaked connections,
statements, and result sets.

For nested scopes, such as inserting data, the connection and statement
are closed when their block ends. If a `ResultSet` is opened, close it
as well, or use try-with-resources around it.

## 26. Connection pooling: what to use beyond beginner examples

For a small learning application, `DriverManager.getConnection(...)` is
straightforward. In a deployed application, opening a new physical
database connection for every request can be inefficient.

A connection pool maintains reusable connections. In Tomcat, you can
configure a JDBC `DataSource` and obtain connections from it:

``` java
import java.sql.Connection;
import javax.naming.InitialContext;
import javax.sql.DataSource;

DataSource ds = (DataSource) new InitialContext()
    .lookup("java:comp/env/jdbc/StudentDB");

try (Connection con = ds.getConnection()) {
    // Use the borrowed connection.
}
```

This requires configuring the JNDI resource in Tomcat and making the
JDBC driver available to the configured resource. Do not copy this
snippet into the earlier example without completing that configuration.

Always close a pooled `Connection` when finished. With a pool, `close()`
generally returns the connection to the pool rather than shutting down
the physical database connection.

## 27. Troubleshooting

  ------------------------------------------------------------------------------------------------------
  Error                                                Likely cause              What to check
  ---------------------------------------------------- ------------------------- -----------------------
  HTTP 404                                             Wrong URL or app not      `webapps/StudentApp`,
                                                       deployed                  context path, URL
                                                                                 mapping

  HTTP 500                                             Exception in Servlet      Tomcat logs

  `ClassNotFoundException: com.mysql.cj.jdbc.Driver`   JDBC driver missing       Connector/J JAR in
                                                                                 `WEB-INF/lib`

  `No suitable driver`                                 Driver unavailable or bad JDBC URL and driver JAR
                                                       URL                       

  `Access denied for user`                             Credentials/permissions   Username, password,
                                                                                 grants

  `Unknown database`                                   Database missing          Create `servlet_db`

  `Communications link failure`                        MySQL stopped or wrong    MySQL service and port
                                                       port                      

  `package javax.servlet does not exist`               Missing compile classpath Use Tomcat's
                                                                                 `lib/servlet-api.jar`

  `UnsupportedClassVersionError`                       Java version mismatch     Compiler and runtime
                                                                                 versions

  HTTP 405                                             HTTP method not handled   Form method and
                                                                                 `doGet()`/`doPost()`
  ------------------------------------------------------------------------------------------------------

Debug one layer at a time: 1. Does `index.html` load? 2. Is Tomcat
running? 3. Does the URL match `web.xml`? 4. Does `StudentServlet.class`
exist? 5. Is Connector/J available? 6. Is MySQL running? 7. Are database
credentials correct? 8. What do the Tomcat logs say?

## 28. Quick reference: which function when?

  Need                            Use
  ------------------------------- ---------------------------------
  Open MySQL connection           `DriverManager.getConnection()`
  Create parameterized SQL        `con.prepareStatement(sql)`
  Read rows                       `executeQuery()` + `ResultSet`
  Insert/update/delete            `executeUpdate()`
  Execute generic SQL             `execute()`
  Bind an integer                 `setInt()`
  Bind text                       `setString()`
  Bind SQL NULL                   `setNull()`
  Move through rows               `rs.next()`
  Read a text column              `rs.getString()`
  Read an integer column          `rs.getInt()`
  Check last value for SQL NULL   `rs.wasNull()`
  Retrieve generated ID           `ps.getGeneratedKeys()`
  Run a batch                     `addBatch()` + `executeBatch()`
  Commit transaction              `con.commit()`
  Undo uncommitted work           `con.rollback()`
  Call a stored procedure         `CallableStatement`
  Inspect returned columns        `ResultSetMetaData`
  Inspect database/driver         `DatabaseMetaData`
  Release resources               `close()` or try-with-resources

## 29. Final run checklist

-   [ ] JDK is installed (`java -version`, `javac -version`).
-   [ ] MySQL server is running.
-   [ ] Database `servlet_db` and table `students` exist.
-   [ ] Tomcat 9 is installed and starts.
-   [ ] `index.html` is inside `StudentApp`.
-   [ ] `web.xml` uses the Servlet 4.0 Java EE namespace.
-   [ ] `StudentServlet.java` compiles successfully.
-   [ ] `StudentServlet.class` is inside `WEB-INF/classes`.
-   [ ] Connector/J is inside `WEB-INF/lib`.
-   [ ] Credentials are correct.
-   [ ] `http://localhost:8080/StudentApp/` opens.
-   [ ] Submitting the form inserts a row.
-   [ ] `SELECT * FROM students;` shows the inserted row.

------------------------------------------------------------------------

## Key takeaways

-   `Connection` represents the database session.
-   `PreparedStatement` is the default choice for parameterized SQL.
-   `executeQuery()` returns a `ResultSet`; use it for SELECT.
-   `executeUpdate()` returns an affected-row count; use it for INSERT,
    UPDATE, DELETE.
-   `ResultSet.next()` moves the cursor to the next row.
-   `getString()`, `getInt()`, and other getters read values from the
    current row.
-   `commit()` and `rollback()` control explicit transactions.
-   Use try-with-resources to close JDBC resources.
-   Tomcat 9 uses `javax.servlet.*`.
-   Do not store user-specific request data in Servlet instance fields.
-   For larger deployments, consider a configured JDBC `DataSource` and
    connection pool.


---

# Part III — More Servlet examples: `doGet()`, `doPost()`, and `GreetServlet`

This section adds reusable Servlet snippets for common practical and exam scenarios. These examples target **Tomcat 9**, so imports use `javax.servlet.*`.

## 30. `doGet()` vs `doPost()`

| Feature | GET / `doGet()` | POST / `doPost()` |
|---|---|---|
| Common purpose | Retrieve a page or data | Submit data that creates or changes something |
| Form method | `method="get"` | `method="post"` |
| Data location | Often in the URL query string | Usually in the request body |
| Bookmark/share URL | Query parameters can be bookmarked | Form body is not represented in the URL |
| Sensitive data | Do not put secrets in URLs | POST alone does not encrypt data; use HTTPS |
| Typical example | Search students, show greeting | Register a student, update a record |

GET is not inherently secure, and POST does not automatically encrypt data. Use HTTPS for confidentiality. Choose the method based on the intended HTTP operation.

## 31. Simple `GreetServlet`: take a name and greet the user

### A. HTML form

Create `greet.html` inside `StudentApp`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Greeting Form</title>
</head>
<body>
    <h1>Enter your name</h1>

    <form action="GreetServlet" method="get">
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required>
        <button type="submit">Greet me</button>
    </form>
</body>
</html>
```

When the user enters `Arun`, the browser requests a URL similar to:

```text
http://localhost:8080/StudentApp/GreetServlet?name=Arun
```

The `name` in `name="name"` is the request parameter key.

### B. `GreetServlet.java`

Save this as `GreetServlet.java`:

```java
import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class GreetServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {

        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        String name = request.getParameter("name");

        if (name == null || name.trim().isEmpty()) {
            name = "Guest";
        } else {
            name = name.trim();
        }

        try (PrintWriter out = response.getWriter()) {
            out.println("<!DOCTYPE html>");
            out.println("<html><head><title>Greeting</title></head><body>");
            out.println("<h1>Hello, " + escapeHtml(name) + "!</h1>");
            out.println("<p>Welcome to Java Servlets.</p>");
            out.println("<a href='greet.html'>Go back</a>");
            out.println("</body></html>");
        }
    }

    private String escapeHtml(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#39;");
    }
}
```

### C. Add the Servlet mapping to `WEB-INF/web.xml`

Add the following elements inside the existing `<web-app>` element, before `</web-app>`:

```xml
<servlet>
    <servlet-name>GreetServlet</servlet-name>
    <servlet-class>GreetServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>GreetServlet</servlet-name>
    <url-pattern>/GreetServlet</url-pattern>
</servlet-mapping>
```

Do not create a second `<web-app>` root. Add these elements to the existing `web.xml`.

### D. Compile both Servlets

From the directory containing `StudentServlet.java` and `GreetServlet.java`:

macOS/Linux:

```bash
javac -cp "$TOMCAT_HOME/lib/servlet-api.jar" \
  -d "$TOMCAT_HOME/webapps/StudentApp/WEB-INF/classes" \
  StudentServlet.java GreetServlet.java
```

Windows:

```bat
javac -cp "%TOMCAT_HOME%\lib\servlet-api.jar" -d "%TOMCAT_HOME%\webapps\StudentApp\WEB-INF\classes" StudentServlet.java GreetServlet.java
```

Open:

```text
http://localhost:8080/StudentApp/greet.html
```

Enter a name and submit the form. The Servlet should display `Hello, <name>!`.

## 32. `GreetServlet` using POST instead of GET

If you prefer not to put the name in the URL, change the form to:

```html
<form action="GreetServlet" method="post">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
    <button type="submit">Greet me</button>
</form>
```

Then implement `doPost()` instead:

```java
@Override
protected void doPost(HttpServletRequest request,
                      HttpServletResponse response)
        throws ServletException, IOException {

    request.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");

    String name = request.getParameter("name");

    if (name == null || name.trim().isEmpty()) {
        name = "Guest";
    } else {
        name = name.trim();
    }

    try (PrintWriter out = response.getWriter()) {
        out.println("<h1>Hello, " + escapeHtml(name) + "!</h1>");
    }
}
```

The helper `escapeHtml()` from the earlier complete class should also be included in this class. If the form uses POST but the Servlet only implements `doGet()`, Tomcat normally responds with HTTP 405 because the inherited POST handler is not implemented for your intended operation.

## 33. A Servlet that supports both GET and POST

Sometimes you want the same greeting behavior for both methods. Put the shared logic in a helper method:

```java
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class GreetServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {
        greet(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        greet(request, response);
    }

    private void greet(HttpServletRequest request,
                       HttpServletResponse response)
            throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        String name = request.getParameter("name");
        if (name == null || name.trim().isEmpty()) {
            name = "Guest";
        }

        try (PrintWriter out = response.getWriter()) {
            out.println("<h1>Hello, " + escapeHtml(name.trim()) + "!</h1>");
        }
    }

    private String escapeHtml(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#39;");
    }
}
```

Use a helper method when GET and POST should share the same processing logic. Remember that GET and POST can have different semantics even if the response code is shared.

## 34. `doGet()` snippet: read query parameters

Example URL:

```text
http://localhost:8080/StudentApp/SearchServlet?name=Arun&city=Chennai
```

Read the parameters:

```java
String name = request.getParameter("name");
String city = request.getParameter("city");
```

A complete minimal handler:

```java
@Override
protected void doGet(HttpServletRequest request,
                     HttpServletResponse response)
        throws ServletException, IOException {

    response.setContentType("text/plain;charset=UTF-8");

    String name = request.getParameter("name");
    String city = request.getParameter("city");

    try (PrintWriter out = response.getWriter()) {
        out.println("Name: " + (name == null ? "Not provided" : name));
        out.println("City: " + (city == null ? "Not provided" : city));
    }
}
```

This demonstrates parameter reading. If you display user-controlled values in HTML, escape them; plain text responses avoid HTML interpretation.

## 35. `doPost()` snippet: read form data

HTML:

```html
<form action="DetailsServlet" method="post">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

Servlet handler:

```java
@Override
protected void doPost(HttpServletRequest request,
                      HttpServletResponse response)
        throws ServletException, IOException {

    request.setCharacterEncoding("UTF-8");

    String name = request.getParameter("name");
    String email = request.getParameter("email");

    response.setContentType("text/html;charset=UTF-8");

    try (PrintWriter out = response.getWriter()) {
        out.println("<h2>Received details</h2>");
        out.println("<p>Name: " + escapeHtml(name == null ? "" : name) + "</p>");
        out.println("<p>Email: " + escapeHtml(email == null ? "" : email) + "</p>");
    }
}
```

Include the `escapeHtml()` helper from the earlier examples. For POST requests, call `request.setCharacterEncoding("UTF-8")` before reading parameters if you need UTF-8 form decoding.

## 36. `doGet()` with JDBC: search one student

HTML form:

```html
<form action="FindStudentServlet" method="get">
    <label for="id">Student ID:</label>
    <input type="number" id="id" name="id" min="1" required>
    <button type="submit">Find</button>
</form>
```

Servlet handler (assumes the class has the same database URL, username, password, and imports as `StudentServlet`):

```java
@Override
protected void doGet(HttpServletRequest request,
                     HttpServletResponse response)
        throws ServletException, IOException {

    response.setContentType("text/html;charset=UTF-8");

    String rawId = request.getParameter("id");

    int id;
    try {
        id = Integer.parseInt(rawId);
        if (id <= 0) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                               "ID must be positive");
            return;
        }
    } catch (NumberFormatException | NullPointerException e) {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                           "A valid student ID is required");
        return;
    }

    String sql = "SELECT id, name, email FROM students WHERE id = ?";

    try (PrintWriter out = response.getWriter();
         Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
         PreparedStatement ps = con.prepareStatement(sql)) {

        ps.setInt(1, id);

        try (ResultSet rs = ps.executeQuery()) {
            if (rs.next()) {
                out.println("<h1>Student found</h1>");
                out.println("<p>ID: " + rs.getInt("id") + "</p>");
                out.println("<p>Name: " + escapeHtml(rs.getString("name")) + "</p>");
                out.println("<p>Email: " + escapeHtml(rs.getString("email")) + "</p>");
            } else {
                out.println("<h1>No student found</h1>");
            }
        }
    } catch (SQLException e) {
        getServletContext().log("Student lookup failed", e);
        if (!response.isCommitted()) {
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
                               "Unable to search for student");
        }
    }
}
```

This example shows a useful combination: `doGet()` reads an ID from the query string, `PreparedStatement` binds the ID, `executeQuery()` runs SELECT, and `ResultSet.next()` checks whether a matching row exists.

## 37. `doPost()` with JDBC: update a student's email

HTML:

```html
<form action="UpdateEmailServlet" method="post">
    <label for="id">Student ID:</label>
    <input type="number" id="id" name="id" min="1" required>

    <label for="email">New email:</label>
    <input type="email" id="email" name="email" required>

    <button type="submit">Update email</button>
</form>
```

Servlet handler:

```java
@Override
protected void doPost(HttpServletRequest request,
                      HttpServletResponse response)
        throws ServletException, IOException {

    request.setCharacterEncoding("UTF-8");

    int id;
    try {
        id = Integer.parseInt(request.getParameter("id"));
        if (id <= 0) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                               "ID must be positive");
            return;
        }
    } catch (NumberFormatException | NullPointerException e) {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                           "A valid ID is required");
        return;
    }

    String email = request.getParameter("email");
    if (email == null || email.trim().isEmpty()) {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                           "Email is required");
        return;
    }

    String sql = "UPDATE students SET email = ? WHERE id = ?";

    try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
         PreparedStatement ps = con.prepareStatement(sql)) {

        ps.setString(1, email.trim());
        ps.setInt(2, id);

        int rows = ps.executeUpdate();

        response.setContentType("text/html;charset=UTF-8");
        try (PrintWriter out = response.getWriter()) {
            if (rows > 0) {
                out.println("<h1>Email updated successfully</h1>");
            } else {
                out.println("<h1>No student found for that ID</h1>");
            }
        }
    } catch (SQLException e) {
        getServletContext().log("Email update failed", e);
        if (!response.isCommitted()) {
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
                               "Unable to update email");
        }
    }
}
```

For a production application, validate email syntax and business rules server-side. Also consider redirect-after-POST so refreshing the result page does not repeat a form submission.

## 38. URL mapping examples

Every Servlet must be reachable through a URL mapping, whether configured in `web.xml` or using annotations such as `@WebServlet`.

For a `web.xml`-based assignment, add declarations like these **inside the existing `<web-app>` element**:

```xml
<servlet>
    <servlet-name>GreetServlet</servlet-name>
    <servlet-class>GreetServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>GreetServlet</servlet-name>
    <url-pattern>/GreetServlet</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>FindStudentServlet</servlet-name>
    <servlet-class>FindStudentServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>FindStudentServlet</servlet-name>
    <url-pattern>/FindStudentServlet</url-pattern>
</servlet-mapping>
```

If using several independent examples, each Java class must be compiled into `WEB-INF/classes`. Do not add duplicate `<servlet-name>` mappings, and ensure the class name in `<servlet-class>` exactly matches the compiled class.

## 39. Quick Servlet patterns

### Read one parameter

```java
String value = request.getParameter("fieldName");
```

### Return plain text

```java
response.setContentType("text/plain;charset=UTF-8");
try (PrintWriter out = response.getWriter()) {
    out.println("Hello!");
}
```

### Return HTML

```java
response.setContentType("text/html;charset=UTF-8");
try (PrintWriter out = response.getWriter()) {
    out.println("<h1>Hello!</h1>");
}
```

### Reject invalid input

```java
response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Invalid input");
return;
```

### Redirect after successful POST

```java
response.sendRedirect(request.getContextPath() + "/StudentListServlet");
```

Use a redirect when you want the browser to make a new request after processing a POST. The destination must be mapped and implemented.

### Log an exception

```java
getServletContext().log("Operation failed", e);
```

Log details on the server, and avoid returning SQL errors or credentials to the browser.

## 40. Servlet exam reminders

- Tomcat 9 uses `javax.servlet.*`.
- `doGet()` handles GET; `doPost()` handles POST.
- `request.getParameter()` reads query/form parameters.
- Call `request.setCharacterEncoding("UTF-8")` before reading POST parameters when needed.
- `response.getWriter()` writes text or HTML into the response.
- `response.sendRedirect()` tells the browser to make a new request.
- `RequestDispatcher.forward()` transfers processing server-side to another resource.
- `RequestDispatcher.include()` includes another resource's output in the current response.
- `PreparedStatement` is preferred for parameterized SQL.
- `executeQuery()` returns a `ResultSet`; `executeUpdate()` returns an update count.
- Close JDBC resources with try-with-resources.
- Do not keep request-specific values in Servlet instance fields.
