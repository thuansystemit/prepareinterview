# Question 1. Explain the difference between BeanFactory and ApplicationContext.
# 🔹 BeanFactory vs ApplicationContext in Spring

## 1. **BeanFactory**
- **Definition:** Core container, provides basic DI (Dependency Injection).
- **Lazy loading:** Beans are created only when requested (`getBean()`).
- **Lightweight:** Useful for memory-constrained environments.
- **Features:** Only supports basic bean instantiation and wiring.

---

## 2. **ApplicationContext**
- **Definition:** Advanced container built on top of `BeanFactory`.
- **Eager loading (by default):** Beans are created at startup for faster access.
- **Enterprise features:** 
  - Internationalization (i18n) support.
  - Event propagation (`ApplicationEventPublisher`).
  - Bean post-processing, AOP, declarative transactions.
  - Integration with Spring’s enterprise services.
- **Recommended:** Most commonly used in Spring applications.

---

## 🔑 Key Differences

| Aspect                  | **BeanFactory**                                   | **ApplicationContext**                            |
|--------------------------|--------------------------------------------------|--------------------------------------------------|
| **Bean Loading**         | Lazy (on first access)                           | Eager (at startup, by default)                   |
| **Memory usage**         | Lightweight                                      | Slightly heavier (more features)                  |
| **Internationalization** | ❌ Not supported                                  | ✅ Supported                                     |
| **Event Handling**       | ❌ Not supported                                  | ✅ Built-in (`publishEvent()`)                   |
| **AOP / Transactions**   | ❌ Not supported                                  | ✅ Supported                                     |
| **Usage**                | Simple apps / testing                            | Enterprise applications (preferred in real apps) |

---

## ✅ Summary
- **BeanFactory** → Basic container, lazy initialization, minimal features.  
- **ApplicationContext** → Full-featured container, enterprise support, preferred in most real-world Spring apps.  

# Question 2. What is dependency injection? Types (constructor, setter, field).

# 🔹 Dependency Injection (DI) in Spring

## 1. **What is Dependency Injection?**
- **Definition:** A design pattern where objects (dependencies) are provided to a class by the Spring IoC container instead of the class creating them itself.
- **Purpose:**  
  - Promotes loose coupling.  
  - Increases testability & maintainability.  
  - Manages object lifecycle automatically.  

Example without DI:
```java
class Car {
    private Engine engine = new Engine(); // tightly coupled
}
```
With DI:
```java
class Car {
    private Engine engine;
    Car(Engine engine) { this.engine = engine; } // dependency injected
}

```

## 2. Types of Dependency Injection

---

### ✅ Constructor Injection
- Dependencies are passed via the **constructor**.
- Ensures immutability (fields can be `final`).
- Makes testing easier (dependencies can be mocked).
- **Best choice** for mandatory dependencies.

```java
@Component
class Car {
    private final Engine engine;

    @Autowired
    public Car(Engine engine) {   // injected here
        this.engine = engine;
    }
}
```
## ✅ Setter Injection

- **Definition**: Dependencies are injected through setter methods **after object creation**.  
- **Best for**: Optional or changeable dependencies.  

### Advantages
- Allows modifying dependency after object is created.  
- Flexible for frameworks/tools.  

### Disadvantages
- Dependency is not guaranteed to be set at construction → risk of `NullPointerException` if not injected.  

### Example
```java
@Service
public class NotificationService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
## ❌ Field Injection (Not Recommended)

- **Definition**: Dependencies are injected **directly into class fields**.  
- **Best for**: Quick prototyping, simple demos.  

### Disadvantages
- Hard to test (no way to pass mocks without reflection).  
- Breaks immutability.  
- Hidden dependencies (not explicit in constructor).  

### Example
```java
@Service
public class NotificationService {
    @Autowired
    private UserRepository userRepository;
}
```
## ✅ Summary

| Aspect           | Constructor Injection | Setter Injection | Field Injection |
|------------------|-----------------------|-----------------|-----------------|
| **When injected** | At object creation    | After creation  | By framework    |
| **Best for**      | Mandatory deps        | Optional deps   | Quick demos     |
| **Immutability**  | ✔ Yes                 | ✘ No            | ✘ No            |
| **Testability**   | ✔ Easy                | ✔ OK            | ✘ Hard          |
| **Recommended?**  | ✅ Yes                | ⚠ Sometimes     | 🚫 Avoid        |

# Question 3. Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`.
## 🔹 Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`

### 1. `@Component`
- **Definition**: A generic stereotype annotation for any Spring-managed component.
- **Use case**: When the class does not fit into other specific roles.
- **Example**:
```java
  @Component
  public class EmailValidator {
      public boolean isValid(String email) {
          return email.contains("@");
      }
}
```
## 2. `@Service`

- **Definition**: Specialization of `@Component`, used for **service layer classes**.
- **Use case**: Encapsulates **business logic** (e.g., user registration, payment processing).

- **Extra**: Improves readability & clarity — makes it clear that the class belongs to the **service layer**.

- **Example**:
```java
import org.springframework.stereotype.Service;

  @Service
  public class UserService {
      
      public void registerUser(String name) {
          // Business logic (e.g., validations, rules)
          System.out.println("Registering user: " + name);
      }
  }
```
# Question 4. What is the Spring Bean lifecycle? (init, destroy, scopes).
# 🌱 Spring Bean Lifecycle

Spring manages bean creation, initialization, usage, and destruction.  

---

## 🔹 Lifecycle Phases

1. **Instantiation**  
   - Spring creates the bean instance (using constructor or factory method).

2. **Populate Properties**  
   - Spring injects dependencies (via constructor/setter/field).

3. **BeanNameAware & BeanFactoryAware** (optional)  
   - Callbacks if bean implements *aware* interfaces.

4. **Pre-initialization (BeanPostProcessor.beforeInitialization)**  
   - Custom logic before init method runs.

5. **Initialization**  
   - Runs methods annotated with `@PostConstruct` or specified in `initMethod`.  
   - Example: `@PostConstruct public void init() { ... }`

6. **Post-initialization (BeanPostProcessor.afterInitialization)**  
   - Additional custom logic after init.

7. **Bean is ready for use**  
   - Application can now use the bean.

8. **Destruction**  
   - When container shuts down → runs `@PreDestroy` or specified `destroyMethod`.  
   - Example: `@PreDestroy public void cleanup() { ... }`

---

## 🔹 Example

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Bean is initialized!");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean is being destroyed!");
    }
}
```
## 🔹 Bean Scopes

- **Singleton (default)** → One instance per Spring container.  
- **Prototype** → New instance every time requested.  
- **Request** → One bean per HTTP request (Web apps only).  
- **Session** → One bean per HTTP session (Web apps only).  
- **Application** → One bean per `ServletContext`.  
- **WebSocket** → One bean per WebSocket session.  

---

## ✅ Summary

- **Init phase** → `@PostConstruct` / `initMethod`.  
- **Destroy phase** → `@PreDestroy` / `destroyMethod`.  
- **Scope** decides bean lifecycle duration (singleton vs prototype vs web scopes).  
- **BeanPostProcessor** allows hooking into pre/post init.  

# Question 5. Explain how **AOP (Aspect-Oriented Programming)** works in Spring.
# ⚡ Aspect-Oriented Programming (AOP) in Spring

AOP is a programming paradigm that helps **separate cross-cutting concerns** (like logging, security, transactions) from business logic.

Instead of duplicating code across classes, we define these concerns once in an **Aspect** and apply them wherever needed.

---

## 🔹 Key Concepts in Spring AOP

- **Aspect** → A module that encapsulates cross-cutting logic (e.g., logging aspect).  
- **Join Point** → A point in the execution of a program (e.g., method call).  
- **Pointcut** → An expression that matches join points (e.g., all methods in a package).  
- **Advice** → The actual code to run at a join point.  
  - **@Before** → Run before method execution.  
  - **@AfterReturning** → Run after successful execution.  
  - **@AfterThrowing** → Run if method throws exception.  
  - **@After** → Run after method finishes (success or fail).  
  - **@Around** → Run before and after method execution (most powerful).  
- **Weaving** → Linking aspects with application code at runtime (Spring uses proxies for this).  

---

## 🔹 Example: Logging Aspect

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    // Pointcut: all methods in service package
    @Before("execution(* com.example.service.*.*(..))")
    public void logBeforeMethod() {
        System.out.println("👉 Method called!");
    }
}
```
## 🔹 How It Works in Spring

1. Spring creates a **proxy** around the target bean.  
2. When a method (join point) is called, the proxy checks if any aspect applies.  
3. If a matching **pointcut** is found, the related **advice** runs.  
4. Finally, the target method executes.  

---

## ✅ Benefits

- Clean separation of cross-cutting concerns.  
- Reduces boilerplate (logging, security, transactions).  
- Improves maintainability.  

---

## ⚠ Pitfalls

- Debugging can be harder (proxies hide actual flow).  
- Works only with **Spring beans**.  
- By default, Spring AOP uses **proxy-based weaving** (runtime, not compile-time).  

# Question 6. How does Spring Boot auto-configuration work internally?

# 🔹 Spring Boot Auto-Configuration

Spring Boot auto-configuration automatically configures your Spring application based on the **dependencies** in the classpath and the **beans defined in the context**, reducing the need for manual configuration.

---

## 🔹 How It Works Internally

1. **@SpringBootApplication**
   - This annotation is a combination of:
     - `@Configuration`
     - `@EnableAutoConfiguration`
     - `@ComponentScan`
   - The key part for auto-configuration is `@EnableAutoConfiguration`.

2. **@EnableAutoConfiguration**
   - Triggers Spring Boot’s **auto-configuration mechanism**.
   - Internally uses `@Import(AutoConfigurationImportSelector.class)` to import all auto-configuration classes.

3. **SpringFactoriesLoader**
   - Spring Boot reads **`META-INF/spring.factories`** in all dependencies to find available auto-configuration classes.
   - Example entry in `spring-boot-autoconfigure`:
     ```
     org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
     org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
     ```

4. **Conditional Configuration**
   - Most auto-configuration classes use **conditional annotations** like:
     - `@ConditionalOnClass` → Apply only if a class is present.
     - `@ConditionalOnMissingBean` → Apply only if no existing bean of that type exists.
     - `@ConditionalOnProperty` → Apply based on property values in `application.properties` or `application.yml`.

5. **Bean Creation**
   - Spring Boot creates and injects beans defined in auto-configuration classes **only if conditions are met**.
   - This allows **sensible defaults** while still allowing custom user-defined beans to override them.

6. **Customization**
   - You can override auto-configured beans by defining your **own beans** with the same type in your context.
   - Use `@ConditionalOnMissingBean` to let auto-configurations back off.

---

## 🔹 Example

Suppose you include **spring-boot-starter-data-jpa**:

1. Spring Boot sees `DataSourceAutoConfiguration` in classpath.  
2. Checks if a `DataSource` bean already exists.  
3. If not, it auto-configures one using `spring.datasource.*` properties.  
4. Similarly, it configures `EntityManagerFactory` and `TransactionManager` if Hibernate classes are present.

---

## ✅ Summary

- Spring Boot auto-configuration reduces boilerplate by **automatically configuring beans** based on classpath and user properties.  
- **Core mechanisms**: `@EnableAutoConfiguration`, `SpringFactoriesLoader`, conditional annotations.  
- **Customizable**: User-defined beans override defaults; properties can fine-tune behavior.  

# Question 7. How do you implement custom starters in Spring Boot?
# 🔹 Implementing Custom Spring Boot Starters

A **Spring Boot starter** is essentially a reusable dependency that bundles **common libraries, auto-configuration, and properties** to simplify setup for multiple projects.

---

## 🔹 Steps to Create a Custom Starter

### 1. Create a New Maven/Gradle Module
- Name it like `mycompany-mystarter`.
- Add `spring-boot-starter-parent` as the parent in `pom.xml` (if Maven).

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>
```
### 2. Create Auto-Configuration Class
- Use @Configuration and @ConditionalOnMissingBean to define default beans.
- Mark class with @AutoConfiguration (Spring Boot 3+) or @EnableAutoConfiguration (older versions).
```java
    package com.example.mystarter;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;

    @Configuration
    public class MyStarterAutoConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public MyService myService() {
            return new MyService(); // default bean
        }
    }
```
### 3. Add spring.factories or META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
- Spring Boot uses this file to discover auto-configuration classes.
For Spring Boot 3+:
```shell
# src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.mystarter.MyStarterAutoConfiguration
```

### 4. Add Custom Properties (Optional)
- Create application.properties or application.yml for default configuration.
- Bind properties using @ConfigurationProperties.
```java
@ConfigurationProperties(prefix = "mystarter")
public class MyStarterProperties {
    private String greeting = "Hello";
    // getters and setters
}
```
### 5. Publish the Starter
- Package as a JAR and publish to your private Maven repository or company Nexus/Artifactory.
- Other projects can now include it as a dependency:
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>mycompany-mystarter</artifactId>
    <version>1.0.0</version>
</dependency>
```
## ✅ Summary: Custom Spring Boot Starter

- **Custom starter** = reusable library + auto-configuration + optional properties.

### Steps:
1. **Create module & add dependencies**  
   - Include `spring-boot-autoconfigure` and any required libraries.

2. **Write auto-configuration class with conditional beans**  
   - Use `@Configuration` and `@ConditionalOnMissingBean`.

3. **Register in `spring.factories` or `AutoConfiguration.imports`**  
   - Allows Spring Boot to automatically discover the configuration.

4. **(Optional) Define properties via `@ConfigurationProperties`**  
   - Enable configurable defaults for your starter.

5. **Publish & reuse in multiple projects**  
   - Package as a JAR and add it as a dependency in other applications.

# Question 8. Difference between `@Transactional(propagation=...)` options.
# 🔹 `@Transactional` Propagation Options in Spring

**Transactional propagation** defines how a transaction behaves when a method is called **inside another transaction**.

---

## 🔹 Common Propagation Types

| Propagation Type         | Behavior                                                                                   | Typical Use Case |
|--------------------------|-------------------------------------------------------------------------------------------|-----------------|
| **REQUIRED**             | Join existing transaction if present; else create new.                                     | Default, most methods |
| **REQUIRES_NEW**         | Suspend existing transaction and create a **new** one.                                     | Independent operations that must commit regardless of outer tx |
| **NESTED**               | Executes within a **nested** transaction if a transaction exists; else behaves like REQUIRED. Supports rollback of inner transaction without affecting outer. | Complex transactions where partial rollback is needed |
| **MANDATORY**            | Must run within an existing transaction; throws exception if none exists.                 | Enforce transactional context |
| **SUPPORTS**             | Runs within transaction if exists; else executes **non-transactionally**.                 | Optional transactional behavior |
| **NOT_SUPPORTED**        | Executes **non-transactionally**, suspending any existing transaction.                    | Non-transactional operations |
| **NEVER**                | Must execute **non-transactionally**; throws exception if a transaction exists.          | Strict non-transactional requirement |

---

## 🔹 Example Usage

```java
@Service
public class OrderService {

    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder() {
        // joins existing tx or starts new one
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAudit() {
        // always executes in a new tx, independent of outer tx
    }
}
```
## ✅ Summary
- REQUIRED → Default, join or create transaction.
- REQUIRES_NEW → Always new, suspends outer.
- NESTED → Inner savepoints, allows partial rollback.
- MANDATORY → Must have existing transaction.
- SUPPORTS → Use existing if available.
- NOT_SUPPORTED / NEVER → Execute outside of transaction.

# Question 8. Difference between `@Transactional(propagation=...)` options.

# 🔹 Difference between `@Transactional(propagation=...)` Options in Spring

Spring’s `@Transactional` supports different **propagation behaviors**, which define how transactions behave when a method is called inside another transactional method.

---

## 🔹 Propagation Types

### 1. `REQUIRED` (default)
- Joins the current transaction if one exists, otherwise starts a new one.
- ✅ Most commonly used.

```java
@Transactional(propagation = Propagation.REQUIRED)
public void doSomething() {
    // uses existing transaction, or creates new if none
}
```
### 2. REQUIRES_NEW
- Always creates a new transaction, suspends any existing one.
- ✅ Useful when the operation must be independent.
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logAction() {
    // runs in new transaction, even if caller has one
}
```
### 3. SUPPORTS
- If a transaction exists, join it; if not, run without a transaction.
- ✅ Useful for read-only operations.
```java
@Transactional(propagation = Propagation.SUPPORTS)
public void getData() {
    // transaction if available, otherwise non-transactional
}
```
### 4. NOT_SUPPORTED
- Always runs without a transaction.
- Suspends existing transaction if present.
```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void doAsyncJob() {
    // no transaction here
}
```
### 5. NEVER
- Must run without a transaction.
- ❌ Throws IllegalTransactionStateException if a transaction exists.
```java
@Transactional(propagation = Propagation.NEVER)
public void doNonTxTask() {
    // fails if called inside a transaction
}
```
### 6. MANDATORY
- Must run inside an existing transaction.
- ❌ Throws exception if no transaction is active.
```java
@Transactional(propagation = Propagation.MANDATORY)
public void updateAccount() {
    // requires existing transaction
}
```
### 7. NESTED
- Executes within a nested transaction if a parent exists.
- If the nested transaction fails, only the nested part rolls back (parent can continue).
- ✅ Requires JDBC savepoints support.
```java
@Transactional(propagation = Propagation.NESTED)
public void partialUpdate() {
    // nested transaction inside parent
}
```
# ✅ Summary Table: Transaction Propagation

| Propagation   | Behavior                                                |
|---------------|---------------------------------------------------------|
| REQUIRED      | Join existing or create new (default)                   |
| REQUIRES_NEW  | Always new transaction, suspends caller                 |
| SUPPORTS      | Join if exists, else run non-transactional              |
| NOT_SUPPORTED | Always non-transactional, suspends caller               |
| NEVER         | Must be non-transactional (error if exists)             |
| MANDATORY     | Must run inside existing transaction (error if none)    |
| NESTED        | Nested transaction with savepoint rollback              |

---

## 👉 Best Practices
- Use **REQUIRED** for most business logic.  
- Use **REQUIRES_NEW** for logging, auditing, retries.  
- Use **NESTED** when partial rollback is needed.  

# Question 9. How does Spring handle circular dependencies?
# 🔄 How Does Spring Handle Circular Dependencies?

Spring tries to resolve **circular dependencies** (when two or more beans depend on each other) using different strategies depending on bean scope and injection type.

---

## 1. Singleton Beans (Default)
- Spring uses a **3-level caching mechanism** during bean creation:
  1. **Singleton Objects** – fully initialized beans.  
  2. **Early References** – half-initialized beans (created but not fully injected).  
  3. **Singleton Factories** – factories to create proxies or early references.  

➡️ This allows Spring to inject a **reference placeholder** before the bean is fully initialized, breaking the cycle.

---

## 2. Constructor Injection
- ❌ Circular dependencies with **constructor injection** cannot be resolved automatically.  
- Spring throws a `BeanCurrentlyInCreationException`.  
- Solution:  
  - Refactor to use **setter injection** or **`@Lazy` injection**.  
  - Redesign to remove circular dependency.

---

## 3. Setter / Field Injection
- ✅ Supported by Spring’s early reference mechanism.  
- Spring creates the first bean, exposes an early reference, and injects it into the second bean.  
- After initialization, it replaces the early reference with the fully initialized bean.

---

## 4. Prototype Beans
- ❌ Circular dependencies with **prototype scope** are not supported.  
- Spring cannot cache prototype beans, so no early reference exists.

---

## 5. Best Practices
- Avoid circular dependencies—they usually indicate **poor design**.  
- Use **interfaces** or **service extraction** to break cycles.  
- Consider **`@Lazy`** on one side of the dependency.  
- For constructor injection, introduce a **third bean** (mediator) to decouple dependencies.

---

✅ **Summary**
- **Singleton + setter/field injection** → handled via early references.  
- **Constructor injection** → not supported (exception).  
- **Prototype beans** → not supported.  
- **Best solution** → redesign to remove cycles.  

# Question 10. How do you secure a REST API with Spring Security + JWT?
# Secure a REST API with Spring Security + JWT

Below is a concise, practical guide (with code snippets) to secure a Spring Boot REST API using JWTs. It includes the flow, required components, example implementations, and best practices.

---

## 🔁 Authentication flow (high-level)
1. Client sends credentials (`/auth/login`).
2. Server authenticates credentials (via `AuthenticationManager` / `UserDetailsService`).
3. Server issues a JWT (access token) — signed, contains user id/roles, short expiry.
4. Client sends JWT in `Authorization: Bearer <token>` header for protected endpoints.
5. Server verifies token on each request (filter) and sets `SecurityContext`.
6. Optionally: use refresh tokens to obtain new access tokens.

---

## ✅ Key components
- **`UserDetailsService`**: load user by username (DB).
- **`PasswordEncoder`**: store & verify hashed passwords (BCrypt).
- **`AuthenticationController`**: `/login`, `/refresh` endpoints.
- **`JwtTokenProvider` / `JwtUtil`**: create & validate tokens.
- **`JwtAuthenticationFilter`**: `OncePerRequestFilter` to parse & validate token and set `SecurityContext`.
- **`SecurityConfig`**: configure `SecurityFilterChain`, allow unauthenticated access to auth endpoints, enforce statelessness.

---

## ⚙ Example code (Spring Boot 3+ / Spring Security 6 style)

### 1) `application.properties`
```properties
# JWT config
jwt.secret=change-this-secret-to-a-strong-one
jwt.expiration-ms=900000          # 15 minutes
jwt.refresh-expiration-ms=2592000000 # 30 days (for refresh tokens)
```
### 2) JWT utility (JwtTokenProvider)
```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.*;
import java.util.stream.Collectors;

@Component
public class JwtTokenProvider {

    private final Key key;
    private final long validityInMs;
    private final long refreshValidityInMs;

    public JwtTokenProvider(
            @Value("${jwt.secret}") String secret,
            @Value("${jwt.expiration-ms}") long validityInMs,
            @Value("${jwt.refresh-expiration-ms}") long refreshValidityInMs) {
        this.key = Keys.hmacShaKeyFor(secret.getBytes());
        this.validityInMs = validityInMs;
        this.refreshValidityInMs = refreshValidityInMs;
    }

    public String createAccessToken(String username, Collection<String> roles) {
        Date now = new Date();
        Date expiry = new Date(now.getTime() + validityInMs);

        return Jwts.builder()
                .setSubject(username)
                .claim("roles", roles)
                .setIssuedAt(now)
                .setExpiration(expiry)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }

    public String createRefreshToken(String username) {
        Date now = new Date();
        Date expiry = new Date(now.getTime() + refreshValidityInMs);

        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(now)
                .setExpiration(expiry)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }

    public String getUsername(String token) {
        return Jwts.parserBuilder().setSigningKey(key).build()
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }

    @SuppressWarnings("unchecked")
    public List<String> getRoles(String token) {
        var claims = Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody();
        Object roles = claims.get("roles");
        if (roles instanceof List) {
            return ((List<?>) roles).stream().map(Object::toString).collect(Collectors.toList());
        }
        return List.of();
    }
}
```

### 3) UserDetailsService (load from DB)
```java
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.*;
import org.springframework.stereotype.Service;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepo; // your JPA repository

    public CustomUserDetailsService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        var user = userRepo.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        var authorities = user.getRoles().stream()
                .map(r -> new SimpleGrantedAuthority(r.getName()))
                .toList();

        return new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(), // hashed
                true, true, true, true,
                authorities
        );
    }
}
```
### 4) JwtAuthenticationFilter
```java
import jakarta.servlet.FilterChain;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.*;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.util.StringUtils;

public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtTokenProvider tokenProvider;
    private final UserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtTokenProvider tokenProvider, UserDetailsService uds) {
        this.tokenProvider = tokenProvider;
        this.userDetailsService = uds;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws java.io.IOException, jakarta.servlet.ServletException {

        String header = req.getHeader("Authorization");
        String token = null;
        if (StringUtils.hasText(header) && header.startsWith("Bearer ")) {
            token = header.substring(7);
        }

        if (token != null && tokenProvider.validateToken(token)) {
            String username = tokenProvider.getUsername(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            var auth = new UsernamePasswordAuthenticationToken(
                    userDetails, null, userDetails.getAuthorities());

            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        chain.doFilter(req, res);
    }
}
```
### 5) SecurityConfig (register filter, stateless)
```java
import org.springframework.context.annotation.*;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    private final JwtTokenProvider jwtTokenProvider;
    private final CustomUserDetailsService userDetailsService;

    public SecurityConfig(JwtTokenProvider jwtTokenProvider, CustomUserDetailsService uds) {
        this.jwtTokenProvider = jwtTokenProvider;
        this.userDetailsService = uds;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(org.springframework.security.config.annotation.web.builders.HttpSecurity http) throws Exception {
        var jwtFilter = new JwtAuthenticationFilter(jwtTokenProvider, userDetailsService);

        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**", "/public/**").permitAll()
                .anyRequest().authenticated()
            );

        http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
### 6) AuthenticationController (login & refresh)
```java
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.*;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

import java.util.stream.Collectors;

@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthenticationManager authManager;
    private final JwtTokenProvider tokenProvider;
    private final UserRepository userRepo; // to persist refresh tokens if desired

    public AuthController(AuthenticationManager authManager, JwtTokenProvider tokenProvider, UserRepository userRepo) {
        this.authManager = authManager;
        this.tokenProvider = tokenProvider;
        this.userRepo = userRepo;
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest req) {
        Authentication auth = authManager.authenticate(
                new UsernamePasswordAuthenticationToken(req.username(), req.password()));

        var user = (org.springframework.security.core.userdetails.User) auth.getPrincipal();
        var roles = user.getAuthorities().stream().map(a -> a.getAuthority()).collect(Collectors.toList());

        String access = tokenProvider.createAccessToken(user.getUsername(), roles);
        String refresh = tokenProvider.createRefreshToken(user.getUsername());

        // Optionally persist refresh token (or store a revocation id)
        return ResponseEntity.ok(new AuthResponse(access, refresh));
    }

    @PostMapping("/refresh")
    public ResponseEntity<?> refresh(@RequestBody RefreshRequest r) {
        if (!tokenProvider.validateToken(r.refreshToken())) {
            return ResponseEntity.status(401).build();
        }
        String username = tokenProvider.getUsername(r.refreshToken());
        var user = userRepo.findByUsername(username).orElseThrow();
        var roles = user.getRoles().stream().map(rn -> rn.getName()).toList();
        String newAccess = tokenProvider.createAccessToken(username, roles);
        return ResponseEntity.ok(Map.of("accessToken", newAccess));
    }
}
```
```java
// DTO records for brevity
public record AuthRequest(String username, String password) {}
public record AuthResponse(String accessToken, String refreshToken) {}
public record RefreshRequest(String refreshToken) {}
```
## 🔐 Best practices & considerations

- **Use HTTPS always** — JWTs are bearer tokens.  
- **Short-lived access tokens** (e.g., 5–15 minutes) + **refresh tokens**.  
- **Store refresh tokens safely** (DB with revocation support), or use **rotating refresh tokens** to prevent reuse.  
- **Don't store sensitive info** in JWT claims (no passwords).  
- **Use strong signing key** (sufficient length) and prefer **asymmetric keys (RS256)** for extra security if appropriate.  
- **Validate tokens fully**: signature, expiry, issuer/audience (if used).  
- **Revoke tokens**: maintain a blacklist or use refresh-token DB with a token id (`jti`) and revocation status.  
- **Avoid storing sessions server-side** — access tokens are stateless; you may persist refresh tokens for revocation.  
- **Handle token expiry gracefully** on the client side (refresh flow).  
- **CORS**: configure properly if browser clients will call the API.  
- **CSRF**: usually disabled for stateless APIs (but be aware when using cookie-based tokens).  
- **Rate-limiting / brute-force protection** for the login endpoint.

---

## ✅ Summary

- Use `UserDetailsService` + `PasswordEncoder` to authenticate credentials.  
- Create JWTs (access + refresh) with short/long expirations.  
- Add a `OncePerRequestFilter` to validate token and set `SecurityContext`.  
- Configure `SecurityFilterChain` to be stateless and permit `/auth/**`.  
- Implement refresh & revocation strategies for long-term security.

---

## ▶️ Would you like me to:
- Provide a **complete GitHub-ready project skeleton** with these classes wired up?  
- Show how to **sign tokens with RSA (RS256)** (key generation + code)?  
- Add **refresh token rotation and revoke** examples (DB schema + code)?

# Question 11. What are the main differences between Spring MVC and Spring WebFlux?
# 🌐 Spring MVC vs Spring WebFlux

## 1. **Programming Model**
- **Spring MVC**: Imperative, thread-per-request model.
- **Spring WebFlux**: Reactive, event-loop model (non-blocking I/O).

---

## 2. **Concurrency Model**
- **Spring MVC**: Uses a **Servlet API** with a fixed thread pool; each request occupies a thread until completion.
- **Spring WebFlux**: Uses **Reactor (Project Reactor, Mono/Flux)** and asynchronous event loop (like Node.js).

---

## 3. **API Style**
- **Spring MVC**:  
  - Returns objects or `ResponseEntity` directly.  
  - Controllers use `@RestController`, `@RequestMapping`.  
  - Synchronous blocking operations.  

- **Spring WebFlux**:  
  - Returns **`Mono<T>`** (0..1 result) or **`Flux<T>`** (0..N results).  
  - Controllers also use `@RestController`, `@RequestMapping`, but responses are reactive streams.  

---

## 4. **Use Cases**
- **Spring MVC**:  
  - Best for **traditional web apps** and **blocking I/O** (e.g., JPA with relational DBs).  
  - Well-suited for most enterprise apps.  

- **Spring WebFlux**:  
  - Best for **high concurrency, streaming data, and microservices**.  
  - Ideal when using **non-blocking APIs** (e.g., reactive MongoDB, R2DBC, WebClient).  

---

## 5. **Performance**
- **Spring MVC**: Performance limited by thread pool size.  
- **Spring WebFlux**: Better scalability under heavy load (since it doesn’t block threads).  

---

## 6. **Compatibility**
- **Spring MVC**: Built on **Servlet 3.1+**.  
- **Spring WebFlux**: Can run on **Servlet containers** (Tomcat, Jetty) or **Netty** (event loop).  

---

## ✅ Summary

| Feature              | Spring MVC (Servlet)          | Spring WebFlux (Reactive) |
|----------------------|-------------------------------|----------------------------|
| Programming Model    | Imperative, blocking          | Reactive, non-blocking     |
| Return Types         | `Object`, `ResponseEntity`    | `Mono<T>`, `Flux<T>`       |
| Concurrency Model    | Thread-per-request            | Event-loop, async          |
| Best For             | Traditional apps, blocking DB | Streaming, high concurrency|
| Underlying Runtime   | Servlet containers            | Servlet containers OR Netty|

👉 **Rule of thumb**:  
- If your stack is blocking (JPA, JDBC) → **Spring MVC**.  
- If using reactive DBs, WebClient, or need high scalability → **Spring WebFlux**.
