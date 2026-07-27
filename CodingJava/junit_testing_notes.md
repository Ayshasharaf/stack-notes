# 🧪 JUnit Testing — Complete Notes

---

## 🧠 First — What Even IS JUnit?

**JUnit** is the standard testing framework for Java. You write small programs that **automatically check** whether your real code behaves correctly.

```
Your Code (production)     →  Calculator.add(2, 3)
Your Test (verification)   →  assertEquals(5, Calculator.add(2, 3))
JUnit Runner               →  Runs all tests, reports pass/fail
```

**One liner:** JUnit = a way to write repeatable, automated checks so you catch bugs before users do.

**Why bother?**

- Refactor safely — tests tell you if you broke something
- Document behavior — tests show how code is supposed to work
- Faster feedback — run 500 checks in seconds, not manual clicking
- Required in real jobs — every serious Java/Spring project uses it

---

## 📦 JUnit 4 vs JUnit 5 (Know This First)


|              | JUnit 4                      | JUnit 5 (Jupiter)                      |
| ------------ | ---------------------------- | -------------------------------------- |
| Package      | `org.junit`                  | `org.junit.jupiter`                    |
| Status       | Legacy, still everywhere     | **Current standard**                   |
| Annotations  | `@Before`, `@After`, `@Test` | `@BeforeEach`, `@AfterEach`, `@Test`   |
| Assertions   | `org.junit.Assert`           | `org.junit.jupiter.api.Assertions`     |
| Architecture | Monolithic                   | Modular (Jupiter + Vintage + Platform) |


**Rule of thumb:** Learn **JUnit 5** first. You'll still see JUnit 4 in older codebases.

```
JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
          (engine)         (new API)        (run old JUnit 4 tests)
```

---

## 🏗️ Anatomy of a Test

Every JUnit test follows the same shape:

```
┌─────────────────────────────────────────────────────────┐
│  ARRANGE  →  Set up inputs, objects, mocks              │
│  ACT      →  Call the method you're testing             │
│  ASSERT   →  Verify the result is what you expect       │
└─────────────────────────────────────────────────────────┘
```

This pattern is called **AAA** (Arrange-Act-Assert). Some teams use **Given-When-Then** — same idea.

```java
@Test
void shouldAddTwoNumbers() {
    // ARRANGE
    Calculator calculator = new Calculator();

    // ACT
    int result = calculator.add(2, 3);

    // ASSERT
    assertEquals(5, result);
}
```

**Naming convention:** `should[ExpectedBehavior]When[Condition]`

```java
@Test
void shouldReturnZeroWhenDividingByZero() { ... }

@Test
void shouldThrowExceptionWhenInputIsNull() { ... }
```

---

## ⚙️ Project Setup

### Maven (`pom.xml`)

```xml
<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.11.4</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.5.2</version>
        </plugin>
    </plugins>
</build>
```

### Gradle (`build.gradle`)

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.11.4'
}

test {
    useJUnitPlatform()
}
```

### File & Package Conventions

```
src/
├── main/java/com/example/Calculator.java      ← production code
└── test/java/com/example/CalculatorTest.java  ← test code (mirror package)
```

- Test class name: `ClassNameTest` or `ClassNameTests`
- Test methods: `void` return type, no parameters
- Tests live in `src/test/java`, **never** in `src/main/java`

---

## 🏷️ Core Annotations (JUnit 5)

### Test execution


| Annotation            | What it does                                         |
| --------------------- | ---------------------------------------------------- |
| `@Test`               | Marks a method as a test case                        |
| `@RepeatedTest(n)`    | Run the same test N times                            |
| `@ParameterizedTest`  | Run test with multiple input sets                    |
| `@TestFactory`        | Dynamic, programmatic test generation                |
| `@Disabled("reason")` | Skip this test (like commenting it out, but tracked) |


### Lifecycle hooks


| Annotation    | Runs                                                      |
| ------------- | --------------------------------------------------------- |
| `@BeforeAll`  | Once before **all** tests in the class (must be `static`) |
| `@AfterAll`   | Once after **all** tests in the class (must be `static`)  |
| `@BeforeEach` | Before **every** test method                              |
| `@AfterEach`  | After **every** test method                               |


```java
class UserServiceTest {

    static UserRepository repo;

    @BeforeAll
    static void initDatabase() {
        repo = new InMemoryUserRepository();
    }

    @BeforeEach
    void setUp() {
        repo.clear();  // fresh state per test
    }

    @Test
    void shouldCreateUser() { ... }

    @AfterEach
    void tearDown() {
        // cleanup if needed
    }

    @AfterAll
    static void shutdown() {
        repo.close();
    }
}
```

### Grouping & organization


| Annotation                             | What it does                            |
| -------------------------------------- | --------------------------------------- |
| `@DisplayName("readable name")`        | Human-friendly name in test reports     |
| `@Nested`                              | Group related tests in an inner class   |
| `@Tag("fast")` / `@Tag("integration")` | Filter tests by category                |
| `@Order(1)`                            | Control execution order (use sparingly) |


```java
@DisplayName("Calculator Tests")
class CalculatorTest {

    @Nested
    @DisplayName("Addition")
    class Addition {
        @Test
        void shouldAddPositiveNumbers() { ... }
    }

    @Nested
    @DisplayName("Division")
    class Division {
        @Test
        void shouldDivideEvenly() { ... }
    }
}
```

---

## ✅ Assertions — The Heart of Testing

Import: `import static org.junit.jupiter.api.Assertions.*;`

### Equality & truth

```java
assertEquals(expected, actual);           // most common
assertEquals(expected, actual, "message"); // with custom failure message
assertNotEquals(unexpected, actual);

assertTrue(condition);
assertFalse(condition);

assertNull(value);
assertNotNull(value);
```

### Exceptions

```java
// JUnit 5 style (preferred)
Exception ex = assertThrows(IllegalArgumentException.class, () -> {
    calculator.divide(10, 0);
});
assertEquals("Cannot divide by zero", ex.getMessage());
```

### Collections & arrays

```java
assertArrayEquals(new int[]{1, 2, 3}, result);

assertIterableEquals(List.of("a", "b"), actualList);

assertAll(
    () -> assertEquals("Alice", user.getName()),
    () -> assertEquals(25, user.getAge()),
    () -> assertTrue(user.isActive())
);  // runs ALL assertions, reports all failures at once
```

### Timeouts

```java
@Test
void shouldCompleteWithinTwoSeconds() {
    assertTimeout(Duration.ofSeconds(2), () -> {
        slowService.process();
    });
}
```

### Soft assertions (third-party)

JUnit itself is "hard assert" — first failure stops the test. For multiple checks, use `assertAll()` (built-in) or **AssertJ** (popular library, see below).

---

## 🔁 Parameterized Tests

Run the same logic with different inputs — avoids copy-pasting test methods.

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 5, 10})
void shouldBePositive(int number) {
    assertTrue(number > 0);
}

@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "0, 0, 0",
    "-1, 1, 0"
})
void shouldAdd(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}

@ParameterizedTest
@MethodSource("provideUsernames")
void shouldValidateUsername(String username, boolean valid) {
    assertEquals(valid, validator.isValid(username));
}

static Stream<Arguments> provideUsernames() {
    return Stream.of(
        Arguments.of("alice", true),
        Arguments.of("", false),
        Arguments.of("x", false)
    );
}
```

**Sources you'll use often:**

- `@ValueSource` — single parameter, simple values
- `@CsvSource` — multiple parameters, inline table
- `@CsvFileSource` — read from a CSV file
- `@MethodSource` — complex objects, custom logic

---

## 🎭 Mocking — Testing in Isolation

Unit tests should test **one class at a time**. If `UserService` depends on `UserRepository` (database), you don't want a real DB in unit tests.

**Mockito** is the standard mocking library. It ships with Spring Boot Test.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository userRepository;   // fake dependency

    @InjectMocks
    UserService userService;         // real class under test, mocks injected

    @Test
    void shouldReturnUserWhenFound() {
        // ARRANGE
        User mockUser = new User(1L, "Alice");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));

        // ACT
        User result = userService.getUser(1L);

        // ASSERT
        assertEquals("Alice", result.getName());
        verify(userRepository).findById(1L);  // confirm it was called
    }
}
```

### Key Mockito methods


| Method                            | Purpose                                          |
| --------------------------------- | ------------------------------------------------ |
| `when(...).thenReturn(...)`       | Stub a method return value                       |
| `when(...).thenThrow(...)`        | Stub an exception                                |
| `verify(mock).method()`           | Assert a method was called                       |
| `verify(mock, never()).method()`  | Assert a method was NOT called                   |
| `verify(mock, times(2)).method()` | Assert call count                                |
| `@Mock`                           | Create a mock object                             |
| `@InjectMocks`                    | Create real object, inject mocks into it         |
| `@Spy`                            | Partial mock — real object, some methods stubbed |


**When to mock vs not mock:**

```
UNIT TEST        →  Mock external dependencies (DB, APIs, file system)
INTEGRATION TEST →  Use real dependencies (or test containers)
E2E TEST         →  Full stack, slowest, fewest tests
```

---

## 🌱 Spring Boot Testing

If you're using Spring Boot (see `springBoot/spring_boot_notes.md`), testing has extra layers.

### Test slices — only load part of the context


| Annotation        | Loads                                |
| ----------------- | ------------------------------------ |
| `@WebMvcTest`     | Controllers only (mock services)     |
| `@DataJpaTest`    | JPA repositories + in-memory DB      |
| `@JsonTest`       | JSON serialization                   |
| `@SpringBootTest` | **Full** application context (heavy) |


```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        when(userService.getUser(1L)).thenReturn(new UserDto("Alice"));

        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    UserRepository userRepository;

    @Test
    void shouldFindByEmail() {
        userRepository.save(new User("alice@example.com"));
        Optional<User> found = userRepository.findByEmail("alice@example.com");
        assertTrue(found.isPresent());
    }
}
```

### Full integration test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserApiIntegrationTest {

    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void shouldCreateUser() {
        ResponseEntity<UserDto> response =
            restTemplate.postForEntity("/users", new CreateUserRequest("Bob"), UserDto.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
    }
}
```

**Spring Boot test dependency** (usually already in your `pom.xml`):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

This bundles: JUnit 5, Mockito, AssertJ, Hamcrest, Spring Test, JSONassert.

---

## 🧰 Useful Libraries (Beyond Core JUnit)


| Library            | What it adds                                                                |
| ------------------ | --------------------------------------------------------------------------- |
| **AssertJ**        | Fluent, readable assertions: `assertThat(result).isEqualTo(5).isPositive()` |
| **Mockito**        | Mocking (above)                                                             |
| **Testcontainers** | Real Docker containers for integration tests (Postgres, Redis, etc.)        |
| **WireMock**       | Mock external HTTP APIs                                                     |
| **Awaitility**     | Test async / eventual consistency                                           |
| **JaCoCo**         | Code coverage reports                                                       |


### AssertJ example

```java
import static org.assertj.core.api.Assertions.*;

assertThat(users)
    .hasSize(3)
    .extracting(User::getName)
    .containsExactly("Alice", "Bob", "Charlie");
```

---

## 🏃 Running Tests

### Command line

```bash
# Maven — all tests
mvn test

# Maven — one class
mvn test -Dtest=CalculatorTest

# Maven — one method
mvn test -Dtest=CalculatorTest#shouldAddTwoNumbers

# Gradle
./gradlew test
./gradlew test --tests "com.example.CalculatorTest"
```

### IDE (IntelliJ / VS Code)

- Green play button next to class or method
- Right-click → Run Tests
- Coverage: Run with Coverage (IntelliJ)

### What happens when you run tests

```
1. Build tool finds all *Test classes in src/test/java
2. JUnit Platform discovers @Test methods
3. @BeforeAll → @BeforeEach → @Test → @AfterEach (repeat) → @AfterAll
4. Reporter outputs: passed ✅ / failed ❌ / skipped ⏭️
5. Build fails if any test fails (CI/CD gate)
```

---

## 📐 The Testing Pyramid

```
        /\
       /  \        E2E Tests (few)
      /    \       Full app, browser/API, slow
     /──────\
    /        \     Integration Tests (some)
   /          \    DB, Spring context, external services
  /────────────\
 /              \  Unit Tests (many)
/________________\ Fast, isolated, mock dependencies
```

**Practical ratios (rough guide):**

- **70%** unit tests — fast, cheap, run on every commit
- **20%** integration tests — verify components work together
- **10%** E2E tests — critical user flows only

---

## ✍️ What to Test (and What NOT to)

### DO test

- Business logic (calculations, validations, transformations)
- Edge cases (null, empty, boundary values, errors)
- Public API of your classes
- Bug fixes (write a test that reproduces the bug first)

### DON'T test

- Framework code (don't test that Spring `@Autowired` works)
- Getters/setters with no logic
- Private methods directly (test through public methods)
- Third-party libraries (they have their own tests)

### Good test qualities — **F.I.R.S.T.**


| Letter | Meaning                                            |
| ------ | -------------------------------------------------- |
| **F**  | Fast — milliseconds, not seconds                   |
| **I**  | Independent — tests don't depend on each other     |
| **R**  | Repeatable — same result every run                 |
| **S**  | Self-validating — pass or fail, no manual checking |
| **T**  | Timely — write tests with (or before) the code     |


---

## 🧩 Common Patterns & Recipes

### 1. Testing void methods (side effects)

```java
@Test
void shouldSendEmailWhenOrderPlaced() {
    orderService.placeOrder(order);
    verify(emailService).sendConfirmation(order.getEmail());
}
```

### 2. Testing optional / nullable results

```java
@Test
void shouldReturnEmptyWhenUserNotFound() {
    when(repo.findById(99L)).thenReturn(Optional.empty());
    assertTrue(userService.findUser(99L).isEmpty());
}
```

### 3. Testing collections

```java
@Test
void shouldFilterActiveUsers() {
    List<User> result = userService.getActiveUsers(allUsers);
    assertEquals(2, result.size());
    assertTrue(result.stream().allMatch(User::isActive));
}
```

### 4. Test data builders (reduce boilerplate)

```java
// Instead of 10 lines of setup in every test:
User user = UserTestBuilder.aUser()
    .withName("Alice")
    .withEmail("alice@test.com")
    .build();
```

### 5. `@TempDir` for file tests

```java
@Test
void shouldWriteFile(@TempDir Path tempDir) {
    Path file = tempDir.resolve("output.txt");
    fileWriter.write(file, "hello");
    assertEquals("hello", Files.readString(file));
}
```

---

## 🚫 Common Mistakes


| Mistake                                         | Fix                                                 |
| ----------------------------------------------- | --------------------------------------------------- |
| Testing implementation, not behavior            | Test **what** it returns, not **how** it calculates |
| One giant test method                           | One logical behavior per test                       |
| Shared mutable state between tests              | Use `@BeforeEach` to reset state                    |
| Tests that depend on order                      | Each test must stand alone                          |
| No assertions                                   | Every test needs at least one assert/verify         |
| Over-mocking                                    | Mock only what you don't own                        |
| Flaky tests (random failures)                   | Fix timing issues, don't use `Thread.sleep`         |
| Ignoring failing tests with `@Disabled` forever | Fix or delete — disabled tests rot                  |


---

## 📊 Code Coverage

**Coverage** = % of lines/branches your tests execute.

```bash
# Maven + JaCoCo
mvn test jacoco:report
# Report at: target/site/jacoco/index.html
```

**Targets (industry rough guide):**

- 80%+ line coverage on business logic is healthy
- 100% is rarely worth the cost
- **High coverage ≠ good tests** — you can hit every line with useless asserts

Focus on **meaningful** tests, not chasing 100%.

---

## 🔄 TDD — Test-Driven Development (Optional but Good to Know)

```
RED    →  Write a failing test
GREEN  →  Write minimum code to pass
REFACTOR →  Clean up, tests keep you safe
```

```java
// 1. RED — test doesn't compile or fails
@Test
void shouldDiscountTenPercentForPremiumMembers() {
    assertEquals(90, calculator.applyDiscount(100, Membership.PREMIUM));
}

// 2. GREEN — implement just enough
public int applyDiscount(int price, Membership membership) {
    if (membership == Membership.PREMIUM) return (int)(price * 0.9);
    return price;
}

// 3. REFACTOR — extract constants, improve naming, etc.
```

You don't have to do TDD always, but understanding the loop helps in interviews and team discussions.

---

## 🗺️ JUnit 4 → JUnit 5 Migration Cheat Sheet


| JUnit 4                             | JUnit 5                                    |
| ----------------------------------- | ------------------------------------------ |
| `@Before`                           | `@BeforeEach`                              |
| `@After`                            | `@AfterEach`                               |
| `@BeforeClass`                      | `@BeforeAll`                               |
| `@AfterClass`                       | `@AfterAll`                                |
| `@Ignore`                           | `@Disabled`                                |
| `@Category`                         | `@Tag`                                     |
| `@RunWith(SpringRunner.class)`      | `@ExtendWith(SpringExtension.class)`       |
| `Assert.assertEquals`               | `Assertions.assertEquals`                  |
| `@Test(expected = Exception.class)` | `assertThrows(Exception.class, () -> ...)` |


---

## 📚 References & Where to Learn

### Official docs (start here)


| Resource              | URL                                                                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| JUnit 5 User Guide    | [https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/)                                                             |
| JUnit 5 API (Javadoc) | [https://junit.org/junit5/docs/current/api/](https://junit.org/junit5/docs/current/api/)                                                                           |
| Mockito Docs          | [https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html) |
| AssertJ Docs          | [https://assertj.github.io/doc/](https://assertj.github.io/doc/)                                                                                                   |
| Spring Boot Testing   | [https://docs.spring.io/spring-boot/reference/testing/index.html](https://docs.spring.io/spring-boot/reference/testing/index.html)                                 |


### Tutorials & courses


| Resource                                                                                | Why it's good                    |
| --------------------------------------------------------------------------------------- | -------------------------------- |
| [JUnit 5 Tutorial — Baeldung](https://www.baeldung.com/junit-5)                         | Best practical Java blog series  |
| [Mockito Tutorial — Baeldung](https://www.baeldung.com/mockito-series)                  | Deep Mockito coverage            |
| [Spring Boot Testing — Baeldung](https://www.baeldung.com/spring-boot-testing)          | Controller, JPA, integration     |
| [Testcontainers Guide](https://testcontainers.com/guides/)                              | Real DB/Redis in tests           |
| [JUnit in Action, 3rd ed.](https://www.manning.com/books/junit-in-action-third-edition) | Book — thorough, JUnit 5 focused |


### Practice


| Resource                                                  | What to do                                      |
| --------------------------------------------------------- | ----------------------------------------------- |
| [Exercism — Java track](https://exercism.org/tracks/java) | Small exercises with test-driven practice       |
| [LeetCode](https://leetcode.com/)                         | Write JUnit tests for your solutions            |
| Your own Spring Boot projects                             | Add tests to controllers and services you build |


### YouTube (search these)

- "JUnit 5 tutorial" — Amigoscode or Java Brains
- "Mockito tutorial Java"
- "Spring Boot unit testing"
- "Test Driven Development Java"

---

## ✅ Checklist — "Do I Know JUnit?"

Use this to self-assess:

- [ ] I can set up JUnit 5 in Maven or Gradle
- [ ] I understand Arrange-Act-Assert
- [ ] I use `@Test`, `@BeforeEach`, `@DisplayName`
- [ ] I can write `assertEquals`, `assertThrows`, `assertAll`
- [ ] I can write a `@ParameterizedTest` with `@CsvSource`
- [ ] I can mock a dependency with Mockito (`when` / `verify`)
- [ ] I know the difference between unit and integration tests
- [ ] I can run tests from CLI (`mvn test`) and IDE
- [ ] I can write a `@WebMvcTest` for a Spring controller
- [ ] I know what NOT to test (getters, framework internals)
- [ ] I can read a test failure stack trace and fix the bug

---

## 🔗 Related Notes in This Repo

- `springBoot/spring_boot_notes.md` — Spring layers, where tests fit in
- `CodingJava/oop_notes.md` — Classes you'll be testing
- `CodingJava/datastructures.md` — Great practice for writing tests on algorithms

---

## 🧾 Quick Copy-Paste Starter Template

```java
package com.example;

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("UserService")
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @BeforeEach
    void setUp() {
        // per-test setup
    }

    @Test
    @DisplayName("should return user when ID exists")
    void shouldReturnUserWhenIdExists() {
        // arrange
        User expected = new User(1L, "Alice");
        when(userRepository.findById(1L)).thenReturn(java.util.Optional.of(expected));

        // act
        User actual = userService.getUser(1L);

        // assert
        assertNotNull(actual);
        assertEquals("Alice", actual.getName());
        verify(userRepository).findById(1L);
    }

    @ParameterizedTest
    @CsvSource({ "0", "-1", "-100" })
    @DisplayName("should reject invalid IDs")
    void shouldRejectInvalidIds(long invalidId) {
        assertThrows(IllegalArgumentException.class,
            () -> userService.getUser(invalidId));
    }
}
```

---

*Last updated: July 2026 — JUnit 5.11.x, Mockito 5.x, Spring Boot 3.x*