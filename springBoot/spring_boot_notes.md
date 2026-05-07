# 🌱 Spring Boot — Complete Notes (Zero to Running App)

> **Who is this for?** Anyone who wants to understand Spring Boot from scratch — what it is, how it works inside, all the key annotations, and how to build your first app.

---

## 🧠 First — What Even IS Spring Boot?

Think of it like this:

```
Spring Framework  =  A massive toolkit with 1000 tools
Spring Boot       =  That toolkit, pre-assembled for you, ready to use
```

**Before Spring Boot existed**, you had to write hundreds of lines of XML config just to start a project. Spring Boot **auto-configures** everything. You just add a dependency, and it "just works."

**One liner:** Spring Boot = Spring Framework + Auto-configuration + Embedded Server .

---

## 🏗️ The 4-Layer Architecture

Every Spring Boot app is structured in **4 layers**. Think of it like a restaurant:

```
┌─────────────────────────────────────────────────────────┐
│                  PRESENTATION LAYER                      │
│         (The waiter — talks to the customer)            │
│           @RestController / @Controller                  │
│              Handles HTTP requests/responses             │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   BUSINESS LAYER                         │
│         (The chef — does the actual cooking)            │
│                     @Service                             │
│        All your business logic lives here               │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  PERSISTENCE LAYER                       │
│     (The recipe book — knows how to store things)       │
│                    @Repository                           │
│       Translates Java objects ↔ database rows           │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   DATABASE LAYER                         │
│         (The fridge — actual data storage)              │
│              MySQL / PostgreSQL / Oracle                 │
│            CRUD operations happen here                  │
└─────────────────────────────────────────────────────────┘
```

**The rule:** Each layer only talks to the layer directly below it. The Controller never directly touches the database. Always goes through Service → Repository.

---

## ⚙️ How Spring Boot Starts — Step by Step

When you click **Run** on your app, here's what actually happens:

```
Step 1: main() method is called
        └─> SpringApplication.run(YourApp.class, args)

Step 2: Spring reads @SpringBootApplication annotation
        └─> This is actually 3 annotations in one (see below)

Step 3: @ComponentScan scans all packages
        └─> Finds every class with @Component, @Service,
            @Repository, @Controller, @RestController

Step 4: @Configuration + @EnableAutoConfiguration
        └─> Reads your pom.xml dependencies
        └─> Auto-configures what's needed
            (e.g., you added JPA? Spring auto-sets up DataSource)

Step 5: Beans are created and stored in IOC Container
        └─> IOC Container = Spring's object factory inside JVM
        └─> All @Bean objects live here, managed by Spring

Step 6: Embedded server starts (Tomcat by default)
        └─> Your app is now running at localhost:8080

Step 7: DispatcherServlet starts listening for HTTP requests
        └─> Routes incoming requests to the right @Controller
```

### Visual: IOC Container

```
                    ┌──────────────── JVM ────────────────────┐
                    │                                          │
  HTTP Request ──►  │  ┌──────────────────────────────────┐   │
                    │  │        IOC CONTAINER             │   │
                    │  │  ┌──────────┐  ┌──────────────┐  │   │
                    │  │  │  Bean:   │  │    Bean:     │  │   │
                    │  │  │ UserCtrl │  │  UserService │  │   │
                    │  │  └──────────┘  └──────────────┘  │   │
                    │  │  ┌──────────────────────────────┐ │   │
                    │  │  │      Bean: UserRepository    │ │   │
                    │  │  └──────────────────────────────┘ │   │
                    │  └──────────────────────────────────┘   │
                    └──────────────────────────────────────────┘
```

---

## 🧩 The @SpringBootApplication Mega-Annotation

This single annotation = three annotations combined:

```
@SpringBootApplication
        │
        ├──► @Configuration
        │         Tells Spring: "This class defines beans"
        │
        ├──► @EnableAutoConfiguration
        │         Tells Spring: "Auto-configure based on my dependencies"
        │         (e.g., spring-boot-starter-web → auto-sets up Tomcat)
        │
        └──► @ComponentScan
                  Tells Spring: "Scan this package and subpackages for components"
```

**Code:**
```java
@SpringBootApplication  // ← this one line does the work of 3
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

## 🏷️ All Key Annotations — Explained Simply

### 📦 Stereotype Annotations (Who are you?)

| Annotation | Layer | What it does |
|---|---|---|
| `@Component` | Any | Generic bean. "Hey Spring, manage this class." |
| `@Controller` | Presentation | Handles HTTP requests, returns HTML views |
| `@RestController` | Presentation | `@Controller` + `@ResponseBody`. Returns JSON/XML (used for APIs) |
| `@Service` | Business | Marks a class as business logic. No functional difference from @Component, but semantically clear. |
| `@Repository` | Persistence | Marks DAO class. Also enables Spring's DB exception translation. |

---

### 🌐 Request Mapping Annotations (Which URL am I handling?)

```java
@RestController
@RequestMapping("/api/users")   // ← base path for all methods below
public class UserController {

    @GetMapping("/{id}")        // GET /api/users/123
    public User getUser(@PathVariable Long id) { ... }

    @PostMapping("/")           // POST /api/users/
    public User createUser(@RequestBody User user) { ... }

    @PutMapping("/{id}")        // PUT /api/users/123
    public User updateUser(@PathVariable Long id, @RequestBody User user) { ... }

    @DeleteMapping("/{id}")     // DELETE /api/users/123
    public void deleteUser(@PathVariable Long id) { ... }
}
```

| Annotation | HTTP Method | Use case |
|---|---|---|
| `@GetMapping` | GET | Fetch data |
| `@PostMapping` | POST | Create new data |
| `@PutMapping` | PUT | Update existing data |
| `@DeleteMapping` | DELETE | Delete data |
| `@PatchMapping` | PATCH | Partial update |

---

### 🔧 Dependency Injection Annotations (Who gives me my dependencies?)

> **Dependency Injection** = instead of you creating objects yourself (`new UserService()`), Spring creates them and hands them to you. This is what the IOC Container does.

```java
@RestController
public class UserController {

    // Option 1: @Autowired on field (common but field injection)
    @Autowired
    private UserService userService;

    // Option 2: Constructor injection (RECOMMENDED — best practice)
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;  // Spring injects this automatically
    }
}
```

| Annotation | What it does |
|---|---|
| `@Autowired` | Spring injects the dependency automatically |
| `@Qualifier("name")` | When multiple beans of same type exist, specify which one |
| `@Primary` | Mark one bean as the default when multiple exist |
| `@Bean` | In @Configuration class — manually define a bean |

---

### 📝 Request Parameter Annotations (How do I read the request?)

```java
@GetMapping("/search")
public List<User> search(
    @RequestParam String name,           // ?name=Alice
    @PathVariable Long id,               // /users/123
    @RequestBody UserDTO body,           // JSON body (POST/PUT)
    @RequestHeader("Authorization") String token  // from headers
) { ... }
```

| Annotation | Reads from |
|---|---|
| `@RequestParam` | URL query params `?key=value` |
| `@PathVariable` | URL path `/users/{id}` |
| `@RequestBody` | HTTP request body (JSON → Java object) |
| `@RequestHeader` | HTTP headers |
| `@ResponseBody` | Converts Java object → JSON in response |

---

### 🗄️ JPA / Database Annotations (How do I map to DB?)

```java
@Entity                          // This class = a database table
@Table(name = "users")           // Table name (optional if class name matches)
public class User {

    @Id                          // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    private Long id;

    @Column(name = "full_name", nullable = false)
    private String name;

    @Column(unique = true)
    private String email;
}
```

| Annotation | What it does |
|---|---|
| `@Entity` | Maps class to a DB table |
| `@Table` | Customize table name |
| `@Id` | Marks primary key field |
| `@GeneratedValue` | Auto-generate ID (auto-increment) |
| `@Column` | Customize column (name, nullable, unique, etc.) |
| `@OneToMany` | 1-to-many relationship (one User → many Orders) |
| `@ManyToOne` | Many-to-one (many Orders → one User) |
| `@ManyToMany` | Many-to-many relationship |
| `@JoinColumn` | Specifies the foreign key column |
| `@Transient` | Field is NOT persisted to DB |

---

### 🔩 Configuration Annotations

```java
@Configuration                   // This class contains bean definitions
public class AppConfig {

    @Bean                        // Method returns a bean Spring manages
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

```java
@Value("${app.secret.key}")      // Reads from application.properties
private String secretKey;

@ConfigurationProperties(prefix = "app.mail")  // Maps entire prefix to class
public class MailProperties {
    private String host;
    private int port;
    // getters/setters...
}
```

---

### 🔄 Transaction Annotation

```java
@Service
public class PaymentService {

    @Transactional   // If any step fails, EVERYTHING is rolled back
    public void transfer(Long fromId, Long toId, double amount) {
        debit(fromId, amount);    // step 1
        credit(toId, amount);     // step 2 — if this fails, step 1 is undone
    }
}
```

---

### ✅ Validation Annotations

```java
public class UserDTO {
    @NotNull
    @NotBlank
    private String name;

    @Email
    private String email;

    @Min(18) @Max(100)
    private int age;

    @Size(min = 8, max = 20)
    private String password;
}

// In controller:
public ResponseEntity<User> create(@Valid @RequestBody UserDTO dto) { ... }
// @Valid triggers validation on the DTO
```

---

## 🌐 How an HTTP Request Flows Through Your App

```
Browser/Client
     │
     │  GET /api/users/42
     ▼
DispatcherServlet  ─────────────────────────────────────────
     │                   (Spring's front controller)
     │  "Who handles /api/users/42?"
     ▼
@RestController (UserController)
     │  @GetMapping("/{id}")
     │  getUser(42)
     │
     │  "I need to fetch user data"
     ▼
@Service (UserService)
     │  getUserById(42)
     │  (business logic runs here — validation, transformations, etc.)
     │
     │  "I need to query the database"
     ▼
@Repository (UserRepository)
     │  findById(42)
     │
     ▼
Database (MySQL/PostgreSQL)
     │  SELECT * FROM users WHERE id = 42
     │
     └──► Returns User object back up the chain
                    ▲
               @Service
                    ▲
             @RestController
                    ▲
     Spring converts User → JSON automatically
                    ▲
     Response: { "id": 42, "name": "Alice" }
                    ▲
              Browser/Client ✅
```

---

## 🔨 Build Your First Spring Boot App — Step by Step

### Step 1: Create the Project

Go to **https://start.spring.io** and select:
- Project: **Maven**
- Language: **Java**
- Spring Boot: **3.x.x** (latest stable)
- Dependencies: `Spring Web`, `Spring Data JPA`, `H2 Database` (in-memory DB for testing)

Download and unzip. Open in IntelliJ or VS Code.

---

### Step 2: Understand the Structure

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/com/example/myapp/
│   │   │   ├── MyAppApplication.java    ← Entry point
│   │   │   ├── controller/
│   │   │   │   └── UserController.java
│   │   │   ├── service/
│   │   │   │   └── UserService.java
│   │   │   ├── repository/
│   │   │   │   └── UserRepository.java
│   │   │   └── model/
│   │   │       └── User.java
│   │   └── resources/
│   │       └── application.properties   ← Config file
├── pom.xml                              ← Dependencies
```

---

### Step 3: The Entry Point

```java
// MyAppApplication.java
@SpringBootApplication
public class MyAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyAppApplication.class, args);
    }
}
```

---

### Step 4: The Model (Entity)

```java
// model/User.java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    // Constructors, Getters, Setters
    public User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // getters and setters...
}
```

---

### Step 5: The Repository

```java
// repository/UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // JpaRepository gives you: findAll(), findById(), save(), deleteById() for FREE
    
    // Custom query — Spring generates SQL from the method name!
    Optional<User> findByEmail(String email);
    List<User> findByNameContaining(String keyword);
}
```

> **Magic:** You don't write any SQL. Spring reads the method name and generates the query automatically.

---

### Step 6: The Service

```java
// service/UserService.java
@Service
public class UserService {

    private final UserRepository userRepository;

    // Constructor injection (best practice)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

### Step 7: The Controller

```java
// controller/UserController.java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET all users
    @GetMapping
    public List<User> getAll() {
        return userService.getAllUsers();
    }

    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<User> getById(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    // POST create user
    @PostMapping
    public ResponseEntity<User> create(@RequestBody User user) {
        User saved = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    // DELETE user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### Step 8: application.properties

```properties
# For H2 in-memory DB (testing — data resets on restart)
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.h2.console.enabled=true          # Access at /h2-console in browser

# Auto-create tables from your @Entity classes
spring.jpa.hibernate.ddl-auto=create-drop

# Show SQL in console (useful for debugging)
spring.jpa.show-sql=true

# Server port (default is 8080)
server.port=8080
```

---

### Step 9: Run & Test

```bash
# In terminal
./mvnw spring-boot:run

# Or just run MyAppApplication.java from your IDE
```

Test with **Postman** or **curl**:

```bash
# Create a user
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

# Get all users
curl http://localhost:8080/api/users

# Get user by ID
curl http://localhost:8080/api/users/1

# Delete user
curl -X DELETE http://localhost:8080/api/users/1
```

---

## 📋 Quick Reference Cheat Sheet

```
ANNOTATION          │  LAYER        │  PURPOSE
────────────────────┼───────────────┼──────────────────────────────────
@SpringBootApplication  Entry point    App bootstrap (3-in-1)
@RestController     │  Controller   │  Handle HTTP, return JSON
@Service            │  Service      │  Business logic
@Repository         │  Repository   │  DB access
@Entity             │  Model        │  Map class to DB table
@Autowired          │  Any          │  Inject dependency
@GetMapping         │  Controller   │  Handle GET requests
@PostMapping        │  Controller   │  Handle POST requests
@PutMapping         │  Controller   │  Handle PUT requests
@DeleteMapping      │  Controller   │  Handle DELETE requests
@PathVariable       │  Controller   │  Read from URL path /users/{id}
@RequestParam       │  Controller   │  Read from query ?name=Alice
@RequestBody        │  Controller   │  Read JSON body
@Transactional      │  Service      │  DB transaction (all or nothing)
@Value              │  Any          │  Inject from application.properties
@Column             │  Model        │  Customize DB column
@Id                 │  Model        │  Primary key field
@GeneratedValue     │  Model        │  Auto-increment ID
```

---

## 💡 Key Concepts to Remember

**IOC (Inversion of Control)**
> Normally YOU create objects. With Spring, Spring creates and manages them. You just say "I need a UserService" and Spring gives it to you. That's IOC.

**Dependency Injection (DI)**
> Spring "injects" (passes) required objects into your class automatically. You don't call `new`. Spring does it.

**Bean**
> Any object managed by Spring's IOC Container. When you annotate a class with @Component, @Service, @Repository, or @Controller — it becomes a bean.

**Auto-configuration**
> Spring Boot reads your dependencies (pom.xml) and automatically configures everything needed. Added `spring-boot-starter-web`? Tomcat is auto-configured. Added JPA? DataSource is auto-configured.

**DispatcherServlet**
> Spring's "traffic controller." Every HTTP request hits this first. It decides which @Controller method should handle it based on URL and HTTP method mapping.

---

## 🛠️ Common pom.xml Dependencies

```xml
<!-- REST API -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA + Database ORM -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- MySQL (for production) -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>

<!-- H2 (for testing/dev — in-memory DB) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- Lombok (auto-generates getters/setters/constructors) -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- Security (JWT, login, etc.) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

*Notes compiled from: GeeksForGeeks Spring Boot internals — simplified and expanded.*
