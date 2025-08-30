# Senior Java + Spring Interview Questions

## 1. Core Java
- Explain differences between `HashMap`, `ConcurrentHashMap`, and `Hashtable`.
- How does the `volatile` keyword work in Java? When would you use it?
- What are the key differences between `synchronized` and `Lock` API?
- Explain Java Memory Model (JMM) and `happens-before` relationship.
- Difference between `String`, `StringBuilder`, and `StringBuffer`.
- How do Garbage Collectors work in Java? Compare G1, CMS, and ZGC.
- What is the difference between checked and unchecked exceptions?
- How does `CompletableFuture` differ from `Future`?
- How does `Optional` help avoid `NullPointerException`? Any pitfalls?
- What is immutability? How do you create an immutable class?

---

## 2. Spring Framework (Core, Boot, Data, Security)
- Explain the difference between **BeanFactory** and **ApplicationContext**.
- What is dependency injection? Types (constructor, setter, field).
- Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`.
- What is the Spring Bean lifecycle? (init, destroy, scopes).
- Explain how **AOP (Aspect-Oriented Programming)** works in Spring.
- How does Spring Boot auto-configuration work internally?
- How do you implement custom starters in Spring Boot?
- Difference between `@Transactional(propagation=...)` options.
- How does Spring handle circular dependencies?
- How do you secure a REST API with Spring Security + JWT?
- What are the main differences between **Spring MVC** and **Spring WebFlux**?

---

## 3. JPA / Hibernate
- Explain the difference between `EntityManager` and `Session`.
- What are the different types of fetching strategies (lazy vs eager)?
- Explain **N+1 select problem** and how to solve it.
- What is optimistic vs pessimistic locking? How does JPA handle it?
- Difference between `CascadeType.ALL` and `orphanRemoval = true`.
- Explain `@JoinColumn` vs `@JoinTable`.
- How does `@Inheritance` work in JPA? (SINGLE_TABLE, JOINED, TABLE_PER_CLASS).
- How does the first-level and second-level cache work in Hibernate?
- How do you handle database migrations in Spring Boot? (Liquibase/Flyway).
- How do you optimize JPA queries for large datasets?

---

## 4. Microservices & Architecture
- What is the difference between monolithic, SOA, and microservices?
- How does **Spring Cloud** help with service discovery (Eureka, Consul)?
- Explain API Gateway patterns (Zuul, Spring Cloud Gateway, Nginx, Kong).
- How do you handle distributed transactions in microservices? (Saga, 2PC).
- How to implement inter-service communication (REST, gRPC, Kafka, RabbitMQ)?
- How to secure microservices (OAuth2, JWT, Keycloak, API Gateway)?
- What is circuit breaker? How does Resilience4j / Hystrix help?
- How do you handle configuration management? (Spring Cloud Config, Vault).
- What patterns do you use for database per microservice? (Shared vs Separate DB).
- How to implement observability? (Logs, Metrics, Tracing with Zipkin/Jaeger).

---

## 5. Performance, Testing, and DevOps
- How do you troubleshoot high CPU or memory usage in a Java app?
- What is the difference between JMeter and Gatling for load testing?
- How do you use caching in Spring? (`@Cacheable`, Redis, Hazelcast).
- How do you test Spring Boot applications (unit, integration, slice tests)?
- How do you mock external services in tests? (WireMock, TestContainers).
- What are best practices for Dockerizing a Spring Boot application?
- How do you optimize JVM startup time and memory footprint?
- How do you tune HikariCP (connection pool) for high load?
- How do you use Kubernetes to deploy Spring Boot microservices?
- What’s your approach to CI/CD for Java projects?

---

## 6. Advanced Topics
- How do you implement custom annotations in Spring?
- What is reactive programming, and when would you use WebFlux vs MVC?
- How do you handle large file uploads in Spring Boot?
- Explain how Kafka works with Spring Boot (producer, consumer, streams).
- How do you prevent SQL injection in JPA queries?
- How to migrate from a monolithic system to microservices?
- What are idempotency strategies for APIs?
- How do you implement rate limiting in microservices?
- What is zero-downtime deployment for Spring Boot apps?
- Explain event-driven architecture and its pros/cons.
