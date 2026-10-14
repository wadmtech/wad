# Spring Boot + MongoDB --- Detailed CRUD Backend Guide

**Student Management REST API · Revised single document**

This guide explains the backend from project creation to testing. It
includes the code, the purpose of each layer, annotations, CRUD
operations, Postman testing, basic MockMvc syntax and a separate
detailed section for `ResponseEntity`.

**Intentionally omitted:** DTOs and dynamic filtering with
`MongoTemplate`. The `Student` model is used directly for requests and
responses.

## Contents

1.  Architecture and CRUD
2.  Project creation and Maven
3.  Folder structure and MongoDB configuration
4.  Main application class
5.  Student model
6.  Repository
7.  Service layer
8.  Exception handling
9.  CRUD controller
10. Postman testing
11. Basic MockMvc testing
12. `ResponseEntity` in detail
13. Running the app and troubleshooting
14. Revision sheet and checklist

------------------------------------------------------------------------

## 1. Architecture and CRUD

We will build a Student Management REST API. Each student has an ID,
name, email, department and age.

Example MongoDB document:

``` json
{
  "_id": "671abc1234567890abcdef12",
  "name": "Arun Kumar",
  "email": "arun@example.com",
  "department": "CSE",
  "age": 20
}
```

MongoDB stores data as documents. Spring Data MongoDB maps a Java
`Student` object to and from the database document.

### CRUD mapping

  Operation   Meaning                      HTTP method
  ----------- ---------------------------- -------------
  Create      Add a new student            `POST`
  Read        Retrieve students            `GET`
  Update      Modify an existing student   `PUT`
  Delete      Remove a student             `DELETE`

### Request flow

``` text
Client / Postman / Vue
          |
          | HTTP request with optional JSON body
          v
    StudentController
          |
          | calls Java service methods
          v
     StudentService
          |
          | calls repository methods
          v
   StudentRepository
          |
          | Spring Data MongoDB
          v
       MongoDB
          |
          | saved document / query result
          v
Controller returns HTTP response + JSON
```

  Layer        Responsibility                         Example
  ------------ -------------------------------------- -------------------------------
  Model        Defines student fields                 `Student.java`
  Repository   Reads/writes MongoDB records           `findAll()`, `save()`
  Service      Contains application logic             Check ID exists before delete
  Controller   Defines API routes and HTTP handling   `@GetMapping`
  Database     Persists records                       `students` collection

### What happens during a POST request?

1.  Postman sends a `POST /api/students` request with student JSON.
2.  Spring maps the URL and method to `StudentController.create()`.
3.  `@RequestBody` converts the JSON into a `Student` object.
4.  `@Valid` checks the model's validation constraints.
5.  The controller calls `StudentService.create(student)`.
6.  The service calls `StudentRepository.save(student)`.
7.  Spring Data MongoDB inserts the document and returns the saved
    student with its ID.
8.  The controller returns the object as JSON with HTTP `201 Created`.

Separating layers keeps each class focused and makes the application
easier to maintain and test.

## 2. Create the Spring Boot project

Use [Spring Initializr](https://start.spring.io/) or your IDE.

1.  Choose **Maven**.
2.  Choose **Java**.
3.  Select a Spring Boot version compatible with your installed JDK.
4.  Choose **Jar** packaging.
5.  Enter a group such as `com.example` and artifact such as
    `studentapi`.
6.  Add these dependencies:
    -   Spring Web
    -   Spring Data MongoDB
    -   Validation
    -   Spring Boot Starter Test
7.  Generate the project, unzip it and open it in your IDE.

### Why these dependencies?

  Dependency                 Purpose
  -------------------------- -----------------------------------------------------
  Spring Web                 REST controllers, HTTP methods and JSON handling
  Spring Data MongoDB        `MongoRepository` and Java/MongoDB mapping
  Validation                 `@NotBlank`, `@Email`, `@Min` and other constraints
  Spring Boot Starter Test   JUnit, MockMvc and testing utilities

Lombok is optional. This guide writes getters, setters and constructors
explicitly so you can understand the Java code without relying on
Lombok.

### Maven and `pom.xml`

Maven manages dependencies and builds the project. The `pom.xml` lists
required libraries and build configuration. Spring Initializr creates
the initial file for you.

Typical dependency section:

``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

`scope=test` means the testing library is needed for tests, not for the
running application. Keep the Spring Boot parent and build plugins
generated by Initializr.

## 3. Folder structure and MongoDB configuration

### Folder structure

``` text
src/main/java/com/example/studentapi/
    StudentApiApplication.java
    model/Student.java
    repository/StudentRepository.java
    service/StudentService.java
    controller/StudentController.java
    exception/ResourceNotFoundException.java
    exception/GlobalExceptionHandler.java

src/main/resources/application.properties

src/test/java/com/example/studentapi/
    controller/StudentControllerTest.java
```

Every Java file should use a package declaration matching its folder.
Keep `StudentApiApplication.java` at the root package
`com.example.studentapi`, so Spring can scan the subpackages.

### Option A: local MongoDB

In `src/main/resources/application.properties`:

``` properties
spring.application.name=student-api
spring.data.mongodb.uri=mongodb://localhost:27017/studentdb
server.port=8080
```

-   `localhost` is the machine running MongoDB.
-   `27017` is MongoDB's common default port.
-   `studentdb` is the database name.
-   The database and collection can be created when the first document
    is saved.

### Option B: MongoDB Atlas

Create a cluster, database user and network access rule for your
development IP. Then configure the connection URI:

``` properties
spring.data.mongodb.uri=mongodb+srv://<username>:<password>@<cluster-url>/studentdb
server.port=8080
```

Replace the placeholders with your actual values. Special characters in
passwords may need URL encoding. Do not commit credentials to Git.

### If the connection fails

-   For local MongoDB, confirm the database service is running.
-   For Atlas, check network access, username, password and URI.
-   Check the application console for the specific connection error.
-   Confirm `spring-boot-starter-data-mongodb` is in `pom.xml`.

## 4. Main application class

``` java
package com.example.studentapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudentApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(StudentApiApplication.class, args);
    }
}
```

-   `main(String[] args)` is the Java entry point.
-   `SpringApplication.run(...)` starts Spring and the embedded web
    server.
-   `@SpringBootApplication` combines configuration, auto-configuration
    and component scanning.

When the app starts, Spring creates the controller and service beans and
provides the repository implementation.

## 5. Model class --- `Student`

There are no DTOs in this version. The same `Student` class is used as
the incoming request object and outgoing response object.

``` java
package com.example.studentapi.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;

@Document(collection = "students")
public class Student {
    @Id
    private String id;

    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Email must be valid")
    @NotBlank(message = "Email is required")
    private String email;

    @NotBlank(message = "Department is required")
    private String department;

    @Min(value = 1, message = "Age must be positive")
    private int age;

    public Student() {}

    public Student(String name, String email, String department, int age) {
        this.name = name;
        this.email = email;
        this.department = department;
        this.age = age;
    }

    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getDepartment() { return department; }
    public void setDepartment(String department) { this.department = department; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

### Important annotations and fields

  --------------------------------------------------------------------------
  Code                                   Meaning
  -------------------------------------- -----------------------------------
  `@Document(collection = "students")`   Maps this class to the `students`
                                         collection

  `@Id`                                  Marks the identifier; MongoDB
                                         stores it as `_id`

  `private String id`                    Holds the ID returned after saving

  `@NotBlank`                            Rejects null, empty or
                                         whitespace-only strings when
                                         validation runs

  `@Email`                               Checks that the value has an
                                         email-like format

  `@Min(1)`                              Requires age to be at least 1 when
                                         validation runs

  No-argument constructor                Useful for object creation and
                                         mapping

  Getters/setters                        Let frameworks and other classes
                                         access fields
  --------------------------------------------------------------------------

**Version note:** newer Spring Boot versions use `jakarta.validation`
imports. Some older projects use `javax.validation`; match your course
project.

### Why is the ID a String?

MongoDB commonly uses an ObjectId as the document identifier. Spring
Data can map that ID to a Java `String`. When creating a student, leave
`id` unset so MongoDB can generate it.

## 6. Repository

The repository accesses the database. Spring Data generates its
implementation, so you do not need to write the basic CRUD methods
yourself.

``` java
package com.example.studentapi.repository;

import com.example.studentapi.model.Student;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface StudentRepository
        extends MongoRepository<Student, String> {
}
```

`MongoRepository<Student, String>` has two type parameters:

-   `Student`: the model managed by the repository.
-   `String`: the Java type of the ID field.

### Built-in methods

  -----------------------------------------------------------------------
  Method                              Purpose
  ----------------------------------- -----------------------------------
  `save(student)`                     Insert a new student or save
                                      changes to an existing student

  `findAll()`                         Return all students as
                                      `List<Student>`

  `findById(id)`                      Return an `Optional<Student>` for
                                      the ID

  `existsById(id)`                    Check whether the ID exists

  `deleteById(id)`                    Delete the document with that ID

  `count()`                           Count documents
  -----------------------------------------------------------------------

`Optional<Student>` means the student may or may not exist. The service
can call `orElseThrow(...)` to handle a missing student.

## 7. Service layer

The service is between the controller and repository. It contains
application logic, such as checking whether a student exists before
updating or deleting.

``` java
package com.example.studentapi.service;

import com.example.studentapi.model.Student;
import com.example.studentapi.repository.StudentRepository;
import com.example.studentapi.exception.ResourceNotFoundException;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class StudentService {
    private final StudentRepository repository;

    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }

    public Student create(Student student) {
        student.setId(null);
        return repository.save(student);
    }

    public List<Student> getAll() {
        return repository.findAll();
    }

    public Student getById(String id) {
        return repository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Student not found with id: " + id));
    }

    public Student update(String id, Student input) {
        Student existing = getById(id);
        existing.setName(input.getName());
        existing.setEmail(input.getEmail());
        existing.setDepartment(input.getDepartment());
        existing.setAge(input.getAge());
        return repository.save(existing);
    }

    public void delete(String id) {
        if (!repository.existsById(id)) {
            throw new ResourceNotFoundException(
                "Student not found with id: " + id);
        }
        repository.deleteById(id);
    }
}
```

### Explain the service code

#### `@Service`

Marks the class as a Spring-managed service-layer bean.

#### Constructor injection

``` java
private final StudentRepository repository;

public StudentService(StudentRepository repository) {
    this.repository = repository;
}
```

Spring provides the repository through the constructor. This makes the
dependency explicit. Modern Spring does not require `@Autowired` on a
single constructor.

#### Create

``` java
public Student create(Student student) {
    student.setId(null);
    return repository.save(student);
}
```

The ID is cleared so this method behaves as a create operation rather
than accidentally updating an existing record if an ID was included in
the request. `save()` persists the document and returns the saved
student.

#### Get all

``` java
public List<Student> getAll() {
    return repository.findAll();
}
```

`findAll()` returns all documents as a Java list. The controller returns
the list as JSON.

#### Get by ID

``` java
public Student getById(String id) {
    return repository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException(
            "Student not found with id: " + id));
}
```

If the ID exists, return the student. Otherwise, throw
`ResourceNotFoundException`. The global handler will turn that into HTTP
404.

#### Update

``` java
Student existing = getById(id);
existing.setName(input.getName());
existing.setEmail(input.getEmail());
existing.setDepartment(input.getDepartment());
existing.setAge(input.getAge());
return repository.save(existing);
```

The service loads the existing student, copies editable fields from the
request and saves it. The existing ID is preserved.

#### Delete

The service checks `existsById`. If no student has that ID, it throws
the custom exception. Otherwise, it calls `deleteById`.

The service layer keeps business logic out of the controller, making the
controller easier to read and the logic easier to test.

## 8. Exception handling

A useful API should return a clear response when a student ID does not
exist.

### Custom exception

``` java
package com.example.studentapi.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

Extending `RuntimeException` lets the service throw this exception
without declaring it in each method signature.

### Global exception handler

This version uses `@ResponseStatus` rather than `ResponseEntity` for
exception handling.

``` java
package com.example.studentapi.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> handleNotFound(
            ResourceNotFoundException ex) {
        return Map.of("error", ex.getMessage());
    }
}
```

How it works:

-   `@RestControllerAdvice` applies exception handling across REST
    controllers.
-   `@ExceptionHandler(ResourceNotFoundException.class)` selects the
    exception this method handles.
-   `@ResponseStatus(HttpStatus.NOT_FOUND)` sets HTTP status 404.
-   The returned `Map` becomes a JSON object such as
    `{"error":"Student not found with id: ..."}`.

If you use Java older than 9, `Map.of(...)` may not be available. You
can return an error class or a mutable map instead.

## 9. Controller --- complete CRUD API

The controller defines URLs and HTTP methods. It receives requests,
calls the service and returns the result. This initial version uses
direct return values and `@ResponseStatus`, not `ResponseEntity`.

``` java
package com.example.studentapi.controller;

import com.example.studentapi.model.Student;
import com.example.studentapi.service.StudentService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/students")
public class StudentController {
    private final StudentService service;

    public StudentController(StudentService service) {
        this.service = service;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Student create(@Valid @RequestBody Student student) {
        return service.create(student);
    }

    @GetMapping
    public List<Student> getAll() {
        return service.getAll();
    }

    @GetMapping("/{id}")
    public Student getById(@PathVariable String id) {
        return service.getById(id);
    }

    @PutMapping("/{id}")
    public Student update(@PathVariable String id,
                          @Valid @RequestBody Student student) {
        return service.update(id, student);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable String id) {
        service.delete(id);
    }
}
```

### Controller annotations

  -----------------------------------------------------------------------
  Annotation                          Purpose
  ----------------------------------- -----------------------------------
  `@RestController`                   Marks a REST controller; returned
                                      objects are serialized to JSON

  `@RequestMapping`                   Sets the shared URL prefix

  `@PostMapping`                      Handles POST

  `@GetMapping`                       Handles GET

  `@PutMapping`                       Handles PUT

  `@DeleteMapping`                    Handles DELETE

  `@PathVariable`                     Reads a value from the URL, such as
                                      the ID

  `@RequestBody`                      Converts incoming JSON into a Java
                                      object

  `@Valid`                            Runs validation constraints

  `@ResponseStatus`                   Sets a fixed HTTP status for a
                                      handler
  -----------------------------------------------------------------------

### How each endpoint works

**Create --- `POST /api/students`**

The JSON body is converted into a `Student`. The controller passes it to
`service.create(...)` and returns the saved student. `@ResponseStatus`
makes a successful create return HTTP 201 Created.

**Read all --- `GET /api/students`**

The controller calls `service.getAll()` and returns a `List<Student>`.
Spring converts the list into a JSON array. The default success status
is HTTP 200 OK.

**Read one --- `GET /api/students/{id}`**

`{id}` is a path variable. For example, `/api/students/abc123` passes
`abc123` into the `id` parameter. If found, the response is 200. If
missing, the exception handler returns 404.

**Update --- `PUT /api/students/{id}`**

The ID comes from the URL and the new field values come from the JSON
body. The service loads the existing record, updates its fields and
saves it. If the ID does not exist, the API returns 404.

**Delete --- `DELETE /api/students/{id}`**

The service checks that the student exists and deletes it.
`@ResponseStatus(HttpStatus.NO_CONTENT)` makes success return HTTP 204
No Content. A 204 response has no body.

### Why does Spring return JSON automatically?

Spring Web uses message converters, commonly Jackson, to serialize Java
objects into JSON and deserialize JSON request bodies into Java objects.
This is why the controller can return `Student` or `List<Student>`
directly.

## 10. Test the API in Postman

Postman is the manual testing workflow for this project. Start the app
and MongoDB, then create a record and reuse its ID for the remaining
operations.

### Before testing

1.  Start MongoDB locally or verify your Atlas connection.
2.  Run `StudentApiApplication.java`.
3.  Open Postman and create a new HTTP request.
4.  Use `http://localhost:8080` as the base URL.

### A. Create a student

**Method:** `POST`\
**URL:** `http://localhost:8080/api/students`

Choose **Body → raw → JSON**:

``` json
{
  "name": "Arun Kumar",
  "email": "arun@example.com",
  "department": "CSE",
  "age": 20
}
```

Click **Send**. Expected: HTTP `201 Created` and a JSON object
containing the saved fields plus an ID. Copy the ID for the next
requests.

### B. Get all students

**Method:** `GET`

``` text
http://localhost:8080/api/students
```

Expected: HTTP `200 OK` and a JSON array.

### C. Get one student

**Method:** `GET`

``` text
http://localhost:8080/api/students/PASTE_ID_HERE
```

Replace `PASTE_ID_HERE` with the ID returned by POST. Expected: HTTP
`200 OK` and one student object.

### D. Update a student

**Method:** `PUT`

``` text
http://localhost:8080/api/students/PASTE_ID_HERE
```

Choose **Body → raw → JSON**:

``` json
{
  "name": "Arun K",
  "email": "arun.k@example.com",
  "department": "CSE",
  "age": 21
}
```

Expected: HTTP `200 OK` and the updated object. Use the same ID in the
URL; the service preserves that ID.

### E. Delete a student

**Method:** `DELETE`

``` text
http://localhost:8080/api/students/PASTE_ID_HERE
```

Expected: HTTP `204 No Content`. An empty response body is correct for
204.

### F. Check the not-found case

After deleting the student, send a GET request using the same ID.
Expected: HTTP `404 Not Found` with a JSON error message from
`GlobalExceptionHandler`.

### Postman checklist

-   Use the correct HTTP method and URL.
-   For POST and PUT, choose **Body → raw → JSON**.
-   Check both the status code and response body.
-   Copy the ID from POST rather than guessing it.
-   Create first, read it, update it, delete it, then verify the
    missing-ID behavior.

## 11. MockMvc --- basic test syntax

MockMvc is for automated tests written in `src/test/java`. It simulates
HTTP requests to the controller without opening a real network port.
Since your practical testing is in Postman, focus on the basic structure
and what each assertion means.

### Minimal GET controller test

``` java
package com.example.studentapi.controller;

import com.example.studentapi.service.StudentService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(StudentController.class)
class StudentControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private StudentService service;

    @Test
    void getAllStudents() throws Exception {
        mockMvc.perform(get("/api/students"))
               .andExpect(status().isOk());
    }
}
```

This shows the core pattern. A complete test may configure the service
mock to return a value. `@WebMvcTest` loads the web layer, so the
service is mocked rather than connecting to MongoDB. Some newer Spring
versions offer a replacement for `@MockBean`; use the annotation taught
in your course.

### The parts to remember

  ---------------------------------------------------------------------------------------------
  Expression                                                Meaning
  --------------------------------------------------------- -----------------------------------
  `mockMvc.perform(get("/api/students"))`                   Simulates a GET request

  `.andExpect(status().isOk())`                             Checks that the response status is
                                                            200

  `.andExpect(content().contentType("application/json"))`   Checks the response content type

  `jsonPath("$.name").value("Arun")`                        Checks that JSON field `name` has
                                                            value `Arun`
  ---------------------------------------------------------------------------------------------

Other status checks:

``` java
status().isOk()          // 200 OK
status().isCreated()     // 201 Created
status().isNoContent()   // 204 No Content
status().isNotFound()    // 404 Not Found
```

A POST test commonly uses `post(...)`, `contentType("application/json")`
and `content(...)` to send JSON. If your course only expects `perform`
and `andExpect`, learn that pattern first.

## 12. `ResponseEntity` --- detailed separate section

**`ResponseEntity<T>` represents the whole HTTP response.** Unlike
returning a `Student` directly, it allows a controller method to choose
the status code, body and optional response headers.

### What does `T` mean?

  Type                              Meaning
  --------------------------------- ----------------------------------
  `ResponseEntity<Student>`         Body contains one `Student`
  `ResponseEntity<List<Student>>`   Body contains a list of students
  `ResponseEntity<String>`          Body contains text
  `ResponseEntity<Void>`            No response body

### Why use `ResponseEntity`?

-   Return different status codes depending on the result.
-   Include a response body with the chosen status.
-   Return an empty response for operations such as delete.
-   Add headers when needed.

### Most important syntax

**HTTP 200 OK**

``` java
return ResponseEntity.ok(student);
```

Returns HTTP 200 and puts `student` in the body.

**HTTP 201 Created**

``` java
return ResponseEntity.status(HttpStatus.CREATED).body(student);
```

Returns HTTP 201 and the saved student object.

**HTTP 204 No Content**

``` java
return ResponseEntity.noContent().build();
```

Returns HTTP 204 with no response body. `build()` completes a response
that has no body.

**HTTP 400 Bad Request**

``` java
return ResponseEntity.badRequest().body("Invalid request");
```

Returns HTTP 400 and a message body.

**HTTP 404 Not Found**

``` java
return ResponseEntity.notFound().build();
```

Returns HTTP 404 with no body. To include an error message, use
`status(HttpStatus.NOT_FOUND).body(...)`.

### Common methods

  --------------------------------------------------------------------------------------------
  Code                                                     Result
  -------------------------------------------------------- -----------------------------------
  `ResponseEntity.ok(body)`                                200 OK + body

  `ResponseEntity.status(HttpStatus.CREATED).body(body)`   201 Created + body

  `ResponseEntity.noContent().build()`                     204 No Content

  `ResponseEntity.badRequest().body(message)`              400 Bad Request + body

  `ResponseEntity.notFound().build()`                      404 Not Found
  --------------------------------------------------------------------------------------------

### Complete alternative `StudentController` using `ResponseEntity`

This is an alternative to the controller in Section 9. **Do not create a
second class with the same name.** To use `ResponseEntity`, replace the
previous `StudentController` implementation with this one.

``` java
package com.example.studentapi.controller;

import com.example.studentapi.model.Student;
import com.example.studentapi.service.StudentService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/students")
public class StudentController {
    private final StudentService service;

    public StudentController(StudentService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<Student> create(
            @Valid @RequestBody Student student) {
        Student saved = service.create(student);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @GetMapping
    public ResponseEntity<List<Student>> getAll() {
        return ResponseEntity.ok(service.getAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Student> getById(@PathVariable String id) {
        return ResponseEntity.ok(service.getById(id));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Student> update(
            @PathVariable String id,
            @Valid @RequestBody Student student) {
        Student updated = service.update(id, student);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable String id) {
        service.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Explain the return types

-   `create()` returns `ResponseEntity<Student>` because the response
    contains the saved student and status 201.
-   `getAll()` returns `ResponseEntity<List<Student>>` because the body
    is a list and status 200.
-   `getById()` returns `ResponseEntity<Student>` because one student is
    returned with status 200.
-   `update()` returns `ResponseEntity<Student>` because the updated
    object is returned with status 200.
-   `delete()` returns `ResponseEntity<Void>` because the response has
    no body and status 204.

### `ResponseEntity` vs direct return vs `@ResponseStatus`

  -----------------------------------------------------------------------
  Approach                            Use it when
  ----------------------------------- -----------------------------------
  Return `Student` directly           Default success status is enough,
                                      typically 200 OK

  `@ResponseStatus`                   A fixed status should always apply
                                      to that handler, such as 201 for
                                      create

  `ResponseEntity<T>`                 The method needs to control
                                      status/body or choose between
                                      different outcomes
  -----------------------------------------------------------------------

**Remember:** `ResponseEntity` is not mandatory. It is a convenient way
to control the HTTP response explicitly.

## 13. Run the application

### Using your IDE

1.  Open the project and wait for Maven to download dependencies.
2.  Start MongoDB locally or confirm your Atlas URI.
3.  Run `StudentApiApplication.java`.
4.  Check the console for successful startup and port 8080.
5.  Open Postman and test `POST /api/students` first.

### Using the terminal

macOS / Linux:

``` bash
./mvnw spring-boot:run
```

Windows:

``` bat
mvnw.cmd spring-boot:run
```

If Maven is installed globally:

``` bash
mvn spring-boot:run
```

Optional: build a JAR.

``` bash
./mvnw clean package
java -jar target/<generated-jar-name>.jar
```

## 14. Common errors and troubleshooting

### MongoDB connection error

Check whether MongoDB is running. For Atlas, confirm credentials,
network access rules and URI. The application console usually provides
the underlying error.

### 404 when fetching an ID

Confirm the ID was copied correctly from the POST response and that the
record was not deleted. A missing record deliberately produces 404 in
this guide.

### Validation annotations do not run

Confirm `spring-boot-starter-validation` is in `pom.xml`, `@Valid` is
present on the controller request body and imports match the Spring Boot
version.

### Cannot deserialize JSON

Check valid JSON syntax, matching property names and a no-argument
constructor plus getters/setters in `Student`.

### Repository bean not found

Check the package declaration and make sure `StudentRepository` is
beneath the main application package and extends
`MongoRepository<Student, String>`.

### Duplicate controller mapping

Do not keep both controller versions as separate classes with the same
route mappings. The `ResponseEntity` controller is an alternative
implementation; replace the original controller if you choose to use it.

## 15. Quick revision sheet

  -----------------------------------------------------------------------
  Annotation / concept                Purpose
  ----------------------------------- -----------------------------------
  `@SpringBootApplication`            Bootstraps Spring Boot and
                                      component scanning

  `@Document`                         Maps a class to a MongoDB
                                      collection

  `@Id`                               Marks the document ID

  `MongoRepository<T, ID>`            Provides database CRUD operations

  `@Service`                          Marks the service/business logic
                                      class

  `@RestController`                   Defines REST endpoints and JSON
                                      responses

  `@RequestMapping`                   Defines a common route prefix

  `@GetMapping` / `@PostMapping`      Maps GET / POST requests

  `@PutMapping` / `@DeleteMapping`    Maps PUT / DELETE requests

  `@PathVariable`                     Reads values from the URL path

  `@RequestBody`                      Converts request JSON to a Java
                                      object

  `@Valid`                            Runs validation rules

  `@ResponseStatus`                   Sets a fixed HTTP status

  `ResponseEntity<T>`                 Controls status, body and optional
                                      headers

  `@RestControllerAdvice`             Handles exceptions centrally

  MockMvc `perform` / `andExpect`     Simulates a request and checks the
                                      result
  -----------------------------------------------------------------------

### CRUD mapping

``` text
CREATE   -> POST    /api/students
READ     -> GET     /api/students
READ ONE -> GET     /api/students/{id}
UPDATE   -> PUT     /api/students/{id}
DELETE   -> DELETE  /api/students/{id}
```

### Common HTTP status codes

  -----------------------------------------------------------------------
  Status                  Meaning                 Typical use
  ----------------------- ----------------------- -----------------------
  200 OK                  Successful request      GET, successful PUT

  201 Created             Resource created        POST

  204 No Content          Success with no body    DELETE

  400 Bad Request         Request is invalid      Invalid input

  404 Not Found           Resource does not exist Unknown student ID

  500 Internal Server     Unexpected server error Unhandled application
  Error                                           failure
  -----------------------------------------------------------------------

## 16. Final checklist

-   [ ] The application has the correct Maven dependencies.
-   [ ] `application.properties` contains a working MongoDB URI.
-   [ ] `Student` uses `@Document` and `@Id`.
-   [ ] `StudentRepository` extends `MongoRepository<Student, String>`.
-   [ ] `StudentService` contains create, get-all, get-by-ID, update and
    delete.
-   [ ] `StudentController` maps all five CRUD operations.
-   [ ] `GlobalExceptionHandler` returns 404 for a missing student.
-   [ ] Postman can create, retrieve, update and delete a student.
-   [ ] You know the basic MockMvc pattern:
    `perform(...).andExpect(...)`.
-   [ ] You can explain `ResponseEntity` and identify its common
    methods.
-   [ ] DTOs and dynamic filtering with `MongoTemplate` remain omitted.

**Design note:** using the model directly is useful for learning basic
CRUD. Larger applications often use DTOs to separate the public API
contract from the database model. They are intentionally omitted here to
match the requested setup.
