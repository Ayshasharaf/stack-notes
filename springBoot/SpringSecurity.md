# Spring Security : A Practical Learning Guide

A hands-on, step-by-step guide to understanding and implementing Spring Security , from zero configuration to full JWT-based stateless authentication.

---

## Table of Contents

1. [Core Principles](#1-core-principles)
2. [Authentication Flow](#2-authentication-flow)
3. [Setting Up the Project](#3-setting-up-the-project)
4. [Adding the Spring Security Dependency](#4-adding-the-spring-security-dependency)
5. [Basic Authentication & Writing a Security Filter Chain](#5-basic-authentication--writing-a-security-filter-chain)
6. [Stateless Authentication (Removing Session Cookies)](#6-stateless-authentication-removing-session-cookies)
7. [In-Memory Authentication](#7-in-memory-authentication)
8. [Role-Based Authorization](#8-role-based-authorization)
9. [Moving Credentials to a Database (H2)](#9-moving-credentials-to-a-database-h2)
10. [Database Authentication with JdbcUserDetailsManager](#10-database-authentication-with-jdbcuserdetailsmanager)
11. [Password Encoding with BCrypt](#11-password-encoding-with-bcrypt)
12. [JWT Authentication](#12-jwt-authentication)
13. [Quick Reference / Cheat Sheet](#13-quick-reference--cheat-sheet)

---



## 1. Core Principles

Before touching any code, keep these security principles in mind , they apply to every decision you make while securing an application:


| Principle                      | What it means in practice                                                         |
| ------------------------------ | --------------------------------------------------------------------------------- |
| **Least privilege**            | Give each user/role only the access they need, nothing more.                      |
| **Secure by design**           | Security isn't bolted on at the end, it's part of the architecture from day one. |
| **Fail-safe defaults**         | If something goes wrong, the system should deny access by default, not allow it.  |
| **Secure communication**       | Use HTTPS/TLS; never send credentials or tokens over plain HTTP.                  |
| **Input validation**           | Never trust client input, validate everything on the server.                     |
| **Auditing & logging**         | Log authentication attempts and security-relevant events for traceability.        |
| **Regular updates & patching** | Keep dependencies (especially security libraries) up to date.                     |


---



## 2. Authentication Flow

Understanding *how* Spring Security processes a request is the single most useful mental model you can build. Here's the flow, end  end:

![Spring Security](SpringSecurityFlow.jpg)
Spring Security Authentication Flow Diagram, Security Filter Chain → AuthenticationManager → AuthenticationProvider → UserDetailsService → PasswordEncoder → SecurityContext

**Step by step:**

1. **Every incoming request** passes through a chain of security filters (the *Security Filter Chain*).
2. When a login request arrives, the **authentication filter** extracts the username/password and wraps them in an `Authentication` object (typically a `UsernamePasswordAuthenticationToken`).
3. This `Authentication` object is handed to the `AuthenticationManager` : the central API that defines how authentication is performed. In practice, this is almost always a `ProviderManager`.
4. The `ProviderManager` delegates the actual work to an `AuthenticationProvider` (e.g. `DaoAuthenticationProvider`), which uses:
  - `PasswordEncoder` : encodes/verifies the submitted password.
  - `UserDetailsService` : loads the user's data (username, password hash, roles) from wherever it's stored (memory, database, etc.).
5. If credentials match, a fully populated `UserDetails` object is returned : containing the user's roles/permissions and account status flags (`isEnabled`, `isAccountNonExpired`, etc.).
6. The resulting `Authentication` object is stored in the `SecurityContext` (held by `SecurityContextHolder`) for the duration of the request.
7. The request is now allowed to proceed to the controller.

> **Key interfaces to remember:** `UserDetails`, `UserDetailsService`, `AuthenticationManager`, `AuthenticationProvider`, `PasswordEncoder`.

---



## 3. Setting Up the Project

Start with a minimal REST controller to have something to test security against:

```java
@RestController
public class GreetingsController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello World";
    }
}
```

> 💡 **Tip:** Run the Spring Boot app after *every* change described below so you can see exactly what each configuration line does.

---



## 4. Adding the Spring Security Dependency

Add the Spring Boot Security starter to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**What happens immediately after adding this:**

- Spring Security auto-secures **every endpoint** by default.
- It auto-generates a default login page.
- On startup, it **prints a random generated password** to the console , this is your login password until you configure your own.

To set a fixed username/password instead, add this to `application.properties`:

```properties
spring.security.user.name=aysha
spring.security.user.password=aysha@test
```

Run and visit: `localhost:8080/hello` , you'll be redirected to a login form.

---



## 5. Basic Authentication & Writing a Security Filter Chain

To understand what Spring Security configures for you by default, look at `ServletWebSecurityAutoConfiguration` (in IntelliJ: double-Shift → search the class name).

It reveals the default filter chain bean, roughly equivalent to:

```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(requests -> requests.anyRequest().authenticated());
    http.formLogin(withDefaults());
    http.httpBasic(withDefaults());
    return http.build();
}
```

This is your starting template. Copy it into your **own** `SecurityConfig.java` so you can customize it:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(requests ->
                requests.anyRequest().authenticated()); // every request must be authenticated

        // http.formLogin(withDefaults()); // disabled below
        http.httpBasic(withDefaults()); // use HTTP Basic auth instead

        return http.build();
    }
}
```

**What changes when you comment out** `formLogin`**:**

- The default login *page* disappears.
- Instead, the browser shows a native **Basic Auth popup** (username/password dialog).

Run and test: `localhost:8080/hello`

![Browser Basic Auth prompt / Postman request-response headers screenshot](helloRequest_Response.png)
---



## 6. Stateless Authentication (Removing Session Cookies)

By default, once you log in, Spring Security creates a session and issues a `JSESSIONID` cookie so you stay logged in across requests. For REST APIs (especially ones meant to work with JWTs later), you usually want **no sessions at all** , every request must authenticate itself independently.

Disable session creation like this:

```java
http.sessionManagement(session ->
        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

**Full config at this stage:**

```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(requests ->
            requests.anyRequest().authenticated());

    http.sessionManagement(session ->
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

    // http.formLogin(withDefaults());
    http.httpBasic(withDefaults());

    return http.build();
}
```

**Result:** No cookies are issued. You can verify this in the browser's DevTools → Network → Cookies tab, or in Postman's Cookies panel.
![No cookies](NoCookies.png)
---



## 7. In-Memory Authentication

Before wiring up a real database, it's useful to define users directly in code. This is done with `InMemoryUserDetailsManager`, which implements `UserDetailsManager` (which in turn extends `UserDetailsService`).

```java
@Bean
public UserDetailsService userDetailsService() {

    UserDetails user1 = User.withUsername("user1")
            .password("{noop}aysha@test")
            .roles("USER")
            .build();

    UserDetails admin = User.withUsername("admin")
            .password("{noop}admin@test")
            .roles("ADMIN")
            .build();

    return new InMemoryUserDetailsManager(user1, admin);
}
```

> ⚠️ `{noop}` **is a placeholder password encoder** that means "no encoding , store as plain text." It's fine for learning/testing, but **never use it in a real application**. We'll replace it with `BCryptPasswordEncoder` in [Section 11](#11-password-encoding-with-bcrypt).

Run and test with either the browser's Basic Auth popup or Postman:

![Postman Basic Auth tab with username "admin" and password filled in, showing 200 OK response](RolesAdmin1.png)
![Postman Basic Auth tab with username "user1" and password filled in, showing 200 OK response](RolesUser1.png)
---



## 8. Role-Based Authorization

Once users have roles, you can restrict individual endpoints using `@PreAuthorize`.

```java
@PreAuthorize("hasRole('USER')")
@GetMapping("/user")
public String userEndpoint() {
    return "Hello User :)";
}

@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin")
public String adminEndpoint() {
    return "Hello Admin :)";
}
```

Method-level security (`@PreAuthorize`) is **not** enabled by default , you need to turn it on explicitly:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    // ...
}
```

**Result:** if a `USER` tries to hit `/admin` (or vice-versa), they get a `403 Forbidden`.

![Postman showing 403 Forbidden response when a USER role hits the /admin endpoint](PreAuthRoles.png)

---



## 9. Moving Credentials to a Database (H2)

Hardcoding users is fine for a demo, but real apps need persistent user storage. H2 is a lightweight in-memory/file database perfect for learning this.

**Dependencies** (add via Spring Initializr or manually):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
```

`application.properties`**:**

```properties
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:test
```

You can connect to this database directly from IntelliJ (via the Database icon), or through the built-in web console at:

```
localhost:8080/h2-console
```
![IntelliJ Data Sources and Drivers dialog configuring H2 in-memory connection](H2idea.png)

### Fixing the H2 console + Spring Security conflict

By default, Spring Security blocks access to `/h2-console`, and because the H2 console runs inside an `<iframe>`, Spring Security's frame protection blocks it too. Fix all of this with:

```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(requests ->
            requests.requestMatchers("/h2-console/**").permitAll()
                    .anyRequest().authenticated());

    http.sessionManagement(session ->
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

    http.httpBasic(withDefaults());

    http.csrf(csrf -> csrf.ignoringRequestMatchers("/h2-console/**")); // H2 console doesn't send CSRF tokens
    http.headers(headers -> headers.frameOptions(frameOptions -> frameOptions.sameOrigin())); // allow the iframe

    return http.build();
}
```

Now `/h2-console` loads and logs in without issues.

![H2 web console showing sample SQL script screen](H2Console.png)

---



## 10. Database Authentication with JdbcUserDetailsManager

Instead of storing users in a Java class, let Spring Security read/write them directly from your database using `JdbcUserDetailsManager`.

```java
@Autowired
DataSource dataSource;

@Bean
public UserDetailsService userDetailsService() {

    UserDetails user1 = User.withUsername("user1")
            .password("{noop}aysha@test")
            .roles("USER")
            .build();

    UserDetails admin = User.withUsername("admin")
            .password("{noop}admin@test")
            .roles("ADMIN")
            .build();

    JdbcUserDetailsManager userDetailsManager = new JdbcUserDetailsManager(dataSource);
    userDetailsManager.createUser(user1);
    userDetailsManager.createUser(admin);

    return userDetailsManager;
}
```

Running this will throw an error the first time , `JdbcUserDetailsManager` expects specific tables (`users`, `authorities`) that don't exist yet.

**Fix:** create a `schema.sql` file in `src/main/resources` using Spring Security's official schema definition:

> Reference: `users.ddl` [on the Spring Security GitHub repo](https://github.com/spring-projects/spring-security/blob/main/core/src/main/resources/org/springframework/security/core/userdetails/jdbc/users.ddl)

Copy that DDL into `schema.sql`, restart the app , the error is gone, and the `USERS` / `AUTHORITIES` tables now exist. Confirm this in the H2 console:

![H2 console SELECT * FROM USERS showing user1 and admin rows](SQL.png)

Test with Postman using the created credentials , you should get a `200 OK` and the response body from your protected endpoint.

---


## 11. Password Encoding with BCrypt

Storing plain-text passwords (even the `{noop}` placeholder) is not acceptable outside of learning exercises. Replace it with **BCrypt**, an industry-standard adaptive hashing algorithm.

**1. Define a** `PasswordEncoder` **bean:**

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

**2. Use it when creating users , remove** `{noop}` **and encode the password instead:**

```java
UserDetails user1 = User.withUsername("user1")
        .password(passwordEncoder().encode("Aysha@test"))
        .roles("USER")
        .build();

UserDetails admin = User.withUsername("admin")
        .password(passwordEncoder().encode("admin@test"))
        .roles("ADMIN")
        .build();
```

**Result:** passwords in the database are now stored as BCrypt hashes (e.g. `$2a$10$VfVjATgIlXncFI2FR...`), never in plain text.

![H2 console showing USERS table with hashed BCrypt passwords](BCrypt.png)
---



## 12. JWT Authentication

Basic Auth is fine for learning, but real-world APIs almost always use **JWT (JSON Web Tokens)** for stateless authentication , no server-side session, no cookies, just a signed token the client sends with every request.

### Why JWT?

- Tokens carry an **expiration time** , no indefinite sessions.
- The client authenticates once and reuses the **token** for subsequent requests.
- The payload can be decoded (though not modified) by anyone , never put secrets in it.



### How it works

![JWT Authentication sequence diagram , User → Client App → Server login request, server generates JWT with secret key, returns JWT, client uses JWT for further requests](jwt.png)

1. User submits login credentials via the client app.
2. Server validates credentials and **generates a JWT**, signed with a secret key.
3. Server returns the JWT to the client.
4. The client includes this JWT (usually in the `Authorization: Bearer <token>` header) on every subsequent request , no need to re-send username/password.



### Anatomy of a JWT

A JWT has three dot-separated parts: `header.payload.signature`

![JWT token structure diagram showing header, payload, and signature sections color-coded](jwtHPS.png)

| Part          | Contents                                                             | Example                                                             |
| ------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Header**    | Algorithm & token type                                               | `{ "alg": "RS256", "typ": "JWT" }`                                  |
| **Payload**   | Claims : user data, expiry, roles                                    | `{ "name": "...", "exp": 1703037180, "sub": "...", "admin": true }` |
| **Signature** | Cryptographic signature ensuring the token hasn't been tampered with | *(binary, not human-readable)*                                      |


> ⚠️ The payload is **base64-encoded, not encrypted** , anyone can decode and read it. Never store passwords or sensitive secrets in the payload.



### Implementation

**1. Add the JJWT dependencies** (check [jwtk/jjwt on GitHub](https://github.com/jwtk/jjwt#maven) for the latest version):

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.13.0</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if you prefer Gson -->
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
```

**2. Create a** `jwt` **package** with these four classes:


| Class               | Responsibility                                                                                                                           |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `JwtUtils`          | Generates, parses, and validates JWTs; extracts claims like the username.                                                                |
| `AuthTokenFilter`   | A custom filter placed in the security chain that intercepts each request, extracts the token, and validates it.                         |
| `AuthEntryPointJwt` | Custom handler for unauthorized access attempts , logs the error and returns a clean JSON error response instead of a default HTML page. |
| `SecurityConfig`    | Wires everything together , registers the JWT filter in the Spring Security filter chain.                                                |


**3. Configure the JWT secret & expiration** in `application.properties`:

```properties
spring.app.jwtSecret=YourStrongSecretKeyHere!#12345
spring.app.jwtExpirationMs=86400000
```

> ⚠️ Use a strong, randomly generated secret in production , never commit it to source control. Consider loading it from an environment variable instead. Also use a realistic expiration (e.g. `86400000` ms = 24 hours) rather than an extremely long value.



### Testing the flow

**Step 1 : Sign in and receive a token:**

```http
POST localhost:8080/signin
Content-Type: application/json

{
  "username": "admin",
  "password": "admin@test"
}
```

Response:

```json
{
  "username": "admin",
  "roles": ["ROLE_ADMIN"],
  "jwtToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsIm..."
}
```

**Step 2 : Use the token on protected endpoints:**

```http
GET localhost:8080/admin
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsIm...
```

 ![Postman screenshot showing Authorization header with Bearer token and 200 OK "Hello Admin :)" response](jwtToken.png)

---



## 13. Quick Reference / Cheat Sheet


| Concept                              | Class / Annotation                                          |
| ------------------------------------ | ----------------------------------------------------------- |
| Central authentication logic         | `AuthenticationManager` (usually `ProviderManager`)         |
| Delegates auth to a provider         | `AuthenticationProvider` (e.g. `DaoAuthenticationProvider`) |
| Loads user from a source             | `UserDetailsService`                                        |
| Users defined in code                | `InMemoryUserDetailsManager`                                |
| Users stored in a database           | `JdbcUserDetailsManager`                                    |
| Encodes/verifies passwords           | `PasswordEncoder` → use `BCryptPasswordEncoder`             |
| Holds the current authenticated user | `SecurityContextHolder` / `SecurityContext`                 |
| Disables sessions (stateless)        | `SessionCreationPolicy.STATELESS`                           |
| Restrict endpoint by role            | `@PreAuthorize("hasRole('...')")`                           |
| Enable method-level security         | `@EnableMethodSecurity`                                     |
| Main security configuration          | `SecurityFilterChain` bean inside a `@Configuration` class  |




### Suggested learning order

```
1. Get a REST endpoint running
2. Add spring-boot-starter-security → see default lockdown
3. Write your own SecurityFilterChain (Basic Auth)
4. Make it stateless (no sessions)
5. Define users in-memory
6. Add role-based restrictions (@PreAuthorize)
7. Move users to a real database (H2 + JdbcUserDetailsManager)
8. Encode passwords properly (BCrypt)
9. Replace Basic Auth with JWT for a production-style stateless API
```

---

*Note: Spring Security's APIs evolve between major versions , always cross-check class/method names (e.g.* `SecurityFilterChain`*, lambda-based DSL) against the version you're using via the [official Spring Security documentation](https://docs.spring.io/spring-security/reference/index.html).*