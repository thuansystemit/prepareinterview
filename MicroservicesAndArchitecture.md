# Question 1. What is the difference between monolithic, SOA, and microservices?
# 🏗️ Monolithic vs SOA vs Microservices

## 1. Monolithic Architecture
- **Definition**: Entire application is built as a single, unified codebase.  
- **Characteristics**:
  - All modules (UI, business logic, data access) are tightly coupled.
  - Deployed as one unit (e.g., a WAR/JAR file).
  - Scaling = replicate the entire application.  
- **Pros**: Simple to develop, test, and deploy (for small apps).  
- **Cons**: Hard to scale, maintain, and update as app grows.

---

## 2. SOA (Service-Oriented Architecture)
- **Definition**: Application is built as a set of **loosely coupled services**, communicating via an **Enterprise Service Bus (ESB)**.  
- **Characteristics**:
  - Services can be large, sometimes handling multiple functions.
  - Communication typically via SOAP, XML, or ESB.
  - Encourages reuse of services across applications.  
- **Pros**: Decoupling, reuse, integration across heterogeneous systems.  
- **Cons**: ESB can become a **single point of failure**; heavier and more complex than microservices.

---

## 3. Microservices Architecture
- **Definition**: Application is decomposed into **small, independent services**, each focusing on a single business capability.  
- **Characteristics**:
  - Each service runs in its own process/container (e.g., Docker).
  - Communication usually via lightweight protocols (REST, gRPC, Kafka).
  - Services can be developed, deployed, and scaled independently.  
- **Pros**: High scalability, fault isolation, technology diversity, CI/CD friendly.  
- **Cons**: Complex distributed system, requires DevOps maturity, monitoring, and communication handling.

---

## 🔹 Key Differences

| Feature              | Monolithic 🏢             | SOA 🏛️                        | Microservices 🧩                |
|----------------------|---------------------------|--------------------------------|---------------------------------|
| **Granularity**      | Single large app          | Medium (coarse-grained)        | Small (fine-grained) services   |
| **Communication**    | In-process calls          | ESB (SOAP, XML, JMS)           | REST, gRPC, Kafka, messaging    |
| **Deployment**       | One unit                  | Services on ESB                | Independently deployable units  |
| **Scalability**      | Scale whole app           | Scale services (via ESB)       | Scale individual services       |
| **Tech Stack**       | Uniform                   | Shared but flexible            | Polyglot (any language/DB)      |
| **Complexity**       | Simple (small apps)       | Medium (ESB overhead)          | High (distributed systems)      |

---

## 👉 Rule of Thumb
- **Monolithic**: Good for small/simple apps.  
- **SOA**: Good for enterprise integration with legacy systems.  
- **Microservices**: Best for large-scale, cloud-native, CI/CD-heavy applications.  

# Question 2. How does **Spring Cloud** help with service discovery (Eureka, Consul)?
# 🌐 Service Discovery with Spring Cloud (Eureka & Consul)

## 1. Why Service Discovery?  
In a **microservices** environment:  
- Services run on dynamic hosts/ports (Docker, Kubernetes, cloud).  
- Hardcoding URLs is **not scalable**.  
- Service Discovery solves this by letting services **register themselves** and **discover others dynamically**.  

---

## 2. Spring Cloud Netflix **Eureka** 🐇  
- **Eureka Server**: A registry where services register themselves.  
- **Eureka Client**: Services that register with Eureka and discover other services.  

🔹 **Workflow**:  
1. Service starts → registers with Eureka (`Eureka Server`).  
2. Eureka stores service metadata (name, host, port, status).  
3. Other services query Eureka to find service instances.  

✅ Features:  
- Client-side discovery (clients get a list of available instances).  
- Self-preservation mode (tolerates network failures).  
- Tight integration with Spring Boot / Spring Cloud Netflix.  

---

## 3. Spring Cloud **Consul** 🧭  
- **Consul** (by HashiCorp) provides:  
  - **Service Discovery** (via DNS or HTTP API).  
  - **Health Checking** (built-in).  
  - **Key/Value store** for configuration.  
- Works with **Spring Cloud Consul** starter for integration.  

🔹 **Workflow**:  
1. Service registers with Consul agent.  
2. Consul performs health checks.  
3. Other services discover instances via Consul DNS or REST API.  

✅ Features:  
- Strong health check support.  
- Supports both client-side and server-side discovery.  
- Provides distributed configuration out of the box.  

---

## 4. 🔹 Eureka vs Consul  

| Feature              | Eureka 🐇                       | Consul 🧭                        |
|----------------------|---------------------------------|----------------------------------|
| **Type**             | Java-only (Netflix OSS)         | Polyglot (works with any stack)  |
| **Health Checks**    | Basic (via heartbeats)          | Advanced (HTTP/TCP/gRPC checks)  |
| **Extra Features**   | Service Discovery only          | KV store, health, multi-dc       |
| **Integration**      | Spring Cloud Netflix            | Spring Cloud Consul              |
| **Ecosystem**        | Java / Spring Boot friendly     | Broad ecosystem (Go, Python, etc)|

---

## 👉 Rule of Thumb  
- Use **Eureka** if you are fully in the **Spring Boot/Java ecosystem** and want simple, lightweight discovery.  
- Use **Consul** if you need **cross-platform support**, **health checks**, and **KV configuration** in addition to discovery.  

# Question 3. Explain API Gateway patterns (Zuul, Spring Cloud Gateway, Nginx, Kong).

# 🚪 API Gateway Patterns

An **API Gateway** is a **single entry point** for all client requests to microservices.  
It handles routing, authentication, monitoring, and cross-cutting concerns.

---

## 1. Common API Gateway Patterns  

### 🔹 **Routing**
- Forward requests from clients to the appropriate backend service.  
- Example: `/users/** → User Service`, `/orders/** → Order Service`.

### 🔹 **Aggregation**
- Gateway combines data from multiple services into one response.  
- Example: `/dashboard` aggregates `User Service` + `Order Service` + `Notification Service`.

### 🔹 **Security**
- Centralized authentication (JWT, OAuth2, API Keys).  
- Rate limiting, throttling, and firewall-like protections.

### 🔹 **Cross-Cutting Concerns**
- Logging, tracing, metrics, and monitoring.  
- Response transformation (e.g., JSON ↔ XML).

---

## 2. Popular API Gateway Implementations  

### 🌐 **Zuul (Netflix OSS)**
- Java-based, part of **Spring Cloud Netflix**.  
- Supports dynamic routing, filters, and basic rate limiting.  
- **Downsides**: Synchronous (blocking I/O), older technology.  

✅ Best for: Legacy Spring Cloud Netflix stacks.  

---

### ⚡ **Spring Cloud Gateway**
- Built on **Spring WebFlux** (reactive, non-blocking).  
- Replaces Zuul in modern Spring projects.  
- Supports:  
  - Route matching & filters  
  - Load balancing with Ribbon/Eureka  
  - Rate limiting, security, circuit breakers (Resilience4J)  

✅ Best for: Spring Boot/Spring Cloud microservices.  

---

### 🌀 **Nginx**
- High-performance **reverse proxy and load balancer**.  
- Can be configured as an API Gateway with:  
  - Routing  
  - SSL termination  
  - Caching  
  - Rate limiting  
- **Open-source and commercial (NGINX Plus)**.  

✅ Best for: High-performance, polyglot systems.  

---

### 🦍 **Kong**
- Open-source, built on **NGINX + Lua**.  
- Provides **API Gateway + Service Mesh** capabilities.  
- Features:  
  - Authentication (JWT, OAuth2, Key auth)  
  - Rate limiting, logging, monitoring  
  - Plugin ecosystem (extensible with Lua/Go/JS)  
- Has a **cloud-native, Kubernetes-friendly** version (Kong Gateway).  

✅ Best for: Large, polyglot, cloud-native systems.  

---

## 3. 🔹 Comparison  

| Feature              | Zuul 🐇 (Netflix) | Spring Cloud Gateway ⚡ | Nginx 🌐 | Kong 🦍 |
|----------------------|------------------|------------------------|---------|---------|
| **Tech**             | Java (Servlet)   | Java (Reactive)        | C       | Nginx + Lua |
| **Performance**      | Medium           | High (non-blocking)    | Very High | High |
| **Ecosystem**        | Spring Cloud     | Spring Cloud           | Polyglot | Polyglot |
| **Security**         | Basic            | OAuth2/JWT integration | Plugins | Plugins (rich) |
| **Extensibility**    | Filters          | Filters (Java/Reactive)| Config  | Plugins (Lua/Go/JS) |
| **Best for**         | Legacy Spring apps | Modern Spring apps | Lightweight reverse proxy | Enterprise API mgmt |

---

## 👉 Rule of Thumb
- Use **Spring Cloud Gateway** if your stack is Spring Boot-based.  
- Use **Nginx** for lightweight, high-performance reverse proxy needs.  
- Use **Kong** if you need full API management, plugins, and cloud-native scaling.  
- Avoid **Zuul** for new projects (legacy).  

# Question 4. How do you handle distributed transactions in microservices? (Saga, 2PC).
# 🔄 Handling Distributed Transactions in Microservices  

In a **monolith**, transactions are simple (ACID, single DB).  
In **microservices**, each service has its own DB → distributed transactions are required.  

---

## 1. ❌ Problems with Distributed Transactions
- Services use different databases.  
- Network failures make ACID guarantees hard.  
- Two main strategies: **2PC (Two-Phase Commit)** and **Saga**.  

---

## 2. ✅ Two-Phase Commit (2PC)
### 🔹 How it works:
1. **Prepare phase** – Coordinator asks all services to reserve resources.  
2. **Commit phase** – If all agree, coordinator commits; otherwise, rollback.  

### 🔹 Pros
- Strong consistency (all-or-nothing).  

### 🔹 Cons
- Slow, blocking.  
- Coordinator is a **single point of failure**.  
- Not scalable for large distributed systems.  

👉 Used rarely in modern microservices (better for tightly-coupled systems).  

---

## 3. ✅ Saga Pattern
A **Saga** is a sequence of **local transactions**.  
Each step updates its own DB and then triggers the next service.  
If something fails → **compensating transactions** roll back previous steps.  

### 🔹 Saga Approaches
#### a) **Choreography (Event-Driven)**
- Services publish events after local success.  
- Other services listen and react.  
- Example:  
  - `Order Service` → emits `OrderCreated`  
  - `Payment Service` → listens, processes payment, emits `PaymentCompleted`  
  - `Shipping Service` → listens, ships product.  

✅ Pros: Simple, no central orchestrator.  
❌ Cons: Hard to manage flows, risk of cyclic dependencies.  

---

#### b) **Orchestration (Central Controller)**
- A **Saga Orchestrator** tells services what to do.  
- Example:  
  - Orchestrator: "Create order" → Order Service  
  - Orchestrator: "Take payment" → Payment Service  
  - Orchestrator: "Ship order" → Shipping Service  
- If one step fails → orchestrator triggers compensating actions.  

✅ Pros: Centralized, clear flow.  
❌ Cons: Orchestrator can become a bottleneck.  

---

## 4. 🔹 Comparison  

| Feature             | 2PC ⚖️                | Saga 🔄 (Choreography) | Saga 🔄 (Orchestration) |
|---------------------|-----------------------|------------------------|-------------------------|
| **Consistency**     | Strong (ACID-like)   | Eventual consistency   | Eventual consistency    |
| **Scalability**     | Poor (blocking)      | High                   | High                    |
| **Complexity**      | Low (conceptually)   | Medium (event chaos)   | Higher (needs orchestrator) |
| **Failure Handling**| Coordinator rollback | Compensating tx        | Compensating tx         |
| **Use Case**        | Small, coupled DBs   | Decentralized systems  | Complex workflows       |

---

## 👉 Rule of Thumb
- **Prefer Saga** in microservices:
  - Use **Choreography** for simple, event-driven flows.  
  - Use **Orchestration** for complex workflows with clear control.  
- Avoid **2PC** unless you have a small, tightly controlled environment.  

# QUestion 5. How to implement inter-service communication (REST, gRPC, Kafka, RabbitMQ)?
# 🔗 Inter-Service Communication in Microservices  

Microservices need to talk to each other. There are **two main styles**:  
- **Synchronous (request/response)** → REST, gRPC  
- **Asynchronous (event-driven)** → Kafka, RabbitMQ  

---

## 1. 🌐 REST (HTTP/JSON)
- **How it works**: One service calls another via HTTP.  
- **Tools**: `RestTemplate`, `WebClient`, OpenFeign.  

✅ Pros  
- Simple, widely adopted.  
- Human-readable (JSON).  
- Easy to debug (Postman, curl).  

❌ Cons  
- High latency (HTTP overhead).  
- Tight coupling (service must be online).  
- No streaming support.  

👉 Best for **simple, synchronous requests** (e.g., user → profile service).  

---

## 2. ⚡ gRPC (HTTP/2 + Protobuf)
- **How it works**: Uses Protocol Buffers (`.proto`) and HTTP/2.  
- **Features**: Streaming, contract-first design.  

✅ Pros  
- High performance (binary format).  
- Built-in streaming (server/client/bidirectional).  
- Strong typing & auto code generation.  

❌ Cons  
- Harder debugging (binary).  
- Steeper learning curve.  

👉 Best for **high-performance, low-latency microservices** (e.g., real-time systems).  

---

## 3. 📩 Kafka (Event Streaming)
- **How it works**: Services publish events to **topics**; other services subscribe.  
- **Pattern**: Event-driven, log-based messaging.  

✅ Pros  
- Scalable, fault-tolerant.  
- Stores event history (replayable).  
- Great for analytics + stream processing.  

❌ Cons  
- Complex setup (brokers, zookeepers, etc.).  
- At-least-once delivery → requires idempotency.  

👉 Best for **event sourcing, streaming pipelines, pub-sub** (e.g., order events, analytics).  

---

## 4. 📨 RabbitMQ (Message Queue)
- **How it works**: Services send messages to **queues**.  
- **Pattern**: Producer → Queue → Consumer(s).  

✅ Pros  
- Reliable message delivery (ack, retries).  
- Flexible routing (fanout, topic, direct exchange).  
- Simpler than Kafka for basic messaging.  

❌ Cons  
- Not built for high-throughput event streaming.  
- No event replay (messages consumed once).  

👉 Best for **work queues, background jobs, async tasks** (e.g., email sending).  

---

## 🔹 Comparison

| Feature             | REST 🌐        | gRPC ⚡       | Kafka 📩              | RabbitMQ 📨        |
|---------------------|---------------|--------------|-----------------------|-------------------|
| **Style**           | Sync (req/resp)| Sync/Streaming| Async (event streaming)| Async (messaging) |
| **Performance**     | Medium        | High         | Very High             | Medium            |
| **Data format**     | JSON/XML      | Protobuf     | Binary (log)          | Binary/Text       |
| **Best for**        | Simple calls  | Low-latency  | Event-driven pipelines| Queues & jobs     |
| **Durability**      | No            | No           | Yes (log replay)      | Yes (queue ack)   |
| **Scalability**     | Limited       | High         | Very High             | High              |

---

## 👉 Rule of Thumb
- Use **REST** for external APIs (easy integration).  
- Use **gRPC** for internal service-to-service calls where performance matters.  
- Use **Kafka** for event-driven systems, analytics, and stream processing.  
- Use **RabbitMQ** for task queues and background jobs.  

# Question 6. How to secure microservices (OAuth2, JWT, Keycloak, API Gateway)?

# 🔐 Securing Microservices (OAuth2, JWT, Keycloak, API Gateway)

Security in microservices is critical because multiple services communicate over the network.  
The main challenges are **authentication**, **authorization**, and **token propagation**.  

---

## 1. 🔑 OAuth2
- **How it works**: Delegates authentication to an **Authorization Server** (Auth0, Keycloak, Okta).  
- **Flow**:  
  1. User logs in → Auth Server issues **access token**.  
  2. Client includes token in requests (e.g., `Authorization: Bearer <token>`).  
  3. Resource servers (microservices) validate the token.  

✅ Pros  
- Widely adopted.  
- Standardized (RFC 6749).  
- Supports delegation (login via Google, Facebook, etc.).  

❌ Cons  
- Complex (different flows: auth code, client credentials, password, etc.).  

👉 Best for **secure API authentication & delegated login**.  

---

## 2. 🪪 JWT (JSON Web Token)
- **How it works**: Token issued by Auth Server, **digitally signed** (HMAC/RSA).  
- **Structure**: `header.payload.signature`  
  - **Header**: Algorithm (e.g., HS256)  
  - **Payload**: Claims (user, roles, expiry)  
  - **Signature**: Integrity check  

✅ Pros  
- Stateless (no DB lookup needed).  
- Portable across services.  
- Fast verification.  

❌ Cons  
- Token revocation is hard (once issued, valid until expiry).  
- Token can be large (base64 encoded).  

👉 Best for **stateless authentication** in microservices.  

---

## 3. 🏰 Keycloak (Identity Provider)
- **How it works**: Open-source **Identity and Access Management (IAM)** solution.  
- Provides:  
  - Single Sign-On (SSO)  
  - OAuth2, OpenID Connect, SAML  
  - User management (roles, groups, MFA)  
- **Integration**: Microservices validate tokens from Keycloak.  

✅ Pros  
- Full-featured IAM.  
- Easy integration with Spring Security.  
- Supports social login, LDAP, Active Directory.  

❌ Cons  
- Adds another service to maintain.  

👉 Best for **enterprise-level IAM** and **centralized auth management**.  

---

## 4. 🌐 API Gateway (Security Layer)
- **How it works**: API Gateway (Zuul, Spring Cloud Gateway, Kong, Nginx) acts as **entry point**.  
- Responsibilities:  
  - Token validation (JWT, OAuth2).  
  - Rate limiting, throttling.  
  - SSL termination.  
  - Request forwarding to microservices.  

✅ Pros  
- Centralized security enforcement.  
- Microservices stay lightweight.  
- Easier policy updates (done at gateway).  

❌ Cons  
- Single point of failure if not scaled.  
- Needs HA setup.  

👉 Best for **centralizing authentication and request filtering**.  

---

## 🔹 Architecture Example

```text
[ Client ] → [ API Gateway ] → [ Microservices ]
                  |
             [ Auth Server (Keycloak/OAuth2) ]
```
- Client requests token from Auth Server.
- API Gateway validates token before forwarding.
- Microservices trust the token (no need to re-authenticate).
## 🔹 Comparison

| Feature        | OAuth2 🔑          | JWT 🪪           | Keycloak 🏰        | API Gateway 🌐        |
|----------------|-------------------|-----------------|-------------------|-----------------------|
| **Type**       | Protocol          | Token format    | IAM solution      | Entry point layer     |
| **State**      | Stateful/Stateless| Stateless       | Stateful          | Stateless/Stateful    |
| **Revocation** | Yes               | Hard            | Yes               | N/A                   |
| **Best for**   | Delegation        | Microservices   | Enterprise IAM    | Central enforcement   |

---

## 👉 Rule of Thumb
- Use **OAuth2 + JWT** for **stateless authentication**.  
- Use **Keycloak** (or Auth0/Okta) for **centralized user management**.  
- Use an **API Gateway** to **enforce security policies in one place**.  

# Question 7. What is circuit breaker? How does Resilience4j / Hystrix help?

# ⚡ Circuit Breaker Pattern

## ❓ What is a Circuit Breaker?
A **circuit breaker** is a design pattern used to prevent cascading failures in distributed systems.  
It works like an **electrical circuit breaker**:  
- When a service keeps failing, the breaker "opens" and stops sending requests to that service.  
- After some time, it switches to **half-open** state to test if the service has recovered.  
- If successful, it closes again and resumes normal traffic.

---

## 🔄 States of a Circuit Breaker
- **Closed** ✅ → Requests flow normally. Failures are counted.
- **Open** ❌ → Requests are blocked (fast-fail) to avoid stressing the failing service.
- **Half-Open** ⚠️ → A few test requests are allowed. If they succeed, breaker closes; if not, it stays open.

---

## 🛠 Implementations in Java/Spring
### 1. Hystrix (Netflix OSS) ⚠️ *(Deprecated)*
- Popular in early microservices.
- Provided circuit breaker, fallback, thread isolation.
- Deprecated since 2018 (maintenance mode).

### 2. Resilience4j (Modern) 🌟
- Lightweight, functional-style, recommended for Spring Boot.
- Modules:
  - **CircuitBreaker** → Prevents cascading failures.
  - **RateLimiter** → Controls request rates.
  - **Retry** → Retries failed calls.
  - **Bulkhead** → Isolates failures (limits concurrent calls).
  - **TimeLimiter** → Timeout handling.
- Works well with **Spring Cloud**.

---

## ✅ Example with Resilience4j

```java
@CircuitBreaker(name = "myService", fallbackMethod = "fallback")
public String callRemoteService() {
    return restTemplate.getForObject("http://remote-service/api", String.class);
}

public String fallback(Throwable ex) {
    return "Remote service unavailable, fallback response!";
}
```
### 🔹 When to Use
- Protect critical microservices from downstream failures.
- Avoid thread exhaustion from long waits/timeouts.
- Provide graceful degradation with fallbacks.
### 👉 Rule of Thumb
- Use Resilience4j (modern, modular, Spring Boot friendly).
- Use circuit breakers when calling unreliable or slow remote services.
- Combine with bulkheads + retries + rate limiters for full resiliency.

# Quesition 8. How do you handle configuration management? (Spring Cloud Config, Vault).

# ⚙️ Configuration Management in Microservices

Managing configurations (URLs, feature flags, secrets, DB creds) across multiple services is critical.

---

## 🛠 Approaches

### 1. **Spring Cloud Config**
- Centralized config server for Spring Boot apps.
- Stores config in Git/SVN/local files.
- Supports profiles (dev/test/prod).
- Clients fetch config at startup or via `/actuator/refresh`.
- Can encrypt/decrypt sensitive properties.

✅ Best for: Spring Boot ecosystems needing **centralized, version-controlled config**.

---

### 2. **HashiCorp Vault**
- Secure secret and config management.
- Provides **dynamic secrets** (e.g., DB creds with TTL).
- Policy-based access control.
- Strong integration with Kubernetes, AWS, Consul.
- Handles key rotation and auditing.

✅ Best for: Systems requiring **enterprise-grade security & secret rotation**.

---

## 🔐 Best Practices
- Externalize configs (no hardcoding).
- Encrypt secrets (Vault or Config Server).
- Use environment-specific profiles.
- Automate updates (refresh without redeploy).
- Principle of least privilege for secret access.

---

## 🔹 Comparison

| Feature                 | Spring Cloud Config ☁️ | HashiCorp Vault 🔐 |
|--------------------------|------------------------|---------------------|
| Primary Use             | App configs            | Secrets & sensitive data |
| Storage                 | Git, SVN, local files  | Secure KV, dynamic secrets |
| Security                | Basic encryption       | Strong encryption, access policies |
| Rotation                | Manual                 | Automatic (DB creds, tokens) |
| Integration             | Spring Boot ecosystem  | Multi-platform (K8s, AWS, etc.) |
| Best For                | Config consistency     | Secret management & security |

---

## 👉 Rule of Thumb
- Use **Spring Cloud Config** → when configs are mostly non-sensitive and you need **centralized management + Git versioning**.  
- Use **Vault** → when managing **secrets, credentials, and strong security policies**.  
- Often, teams **combine both**: Config Server for general configs, Vault for secrets.

# Question 9. What patterns do you use for database per microservice? (Shared vs Separate DB).
# 🗄️ Database Patterns in Microservices

In microservices, deciding how to manage databases is crucial for scalability, autonomy, and consistency.  
Two main patterns exist: **Shared Database** and **Database per Service**.

---

## 1. **Shared Database Pattern**
- All microservices connect to a single database.
- Services may share tables or schemas.

### ✅ Pros
- Simple to implement.
- Easy cross-service queries (JOINs).
- Centralized data management.

### ❌ Cons
- Tight coupling → schema change affects many services.
- Harder to scale independently.
- Breaks microservice autonomy.
- Can become a monolith bottleneck.

### 🔹 Best Use Case
- Small apps or transitional phase from monolith to microservices.

---

## 2. **Database per Service Pattern**
- Each microservice owns its own database.
- No direct DB access between services (only via APIs or events).

### ✅ Pros
- Loose coupling, true service autonomy.
- Independent schema evolution.
- Enables polyglot persistence (different DBs for different services).
- Easier independent scaling & deployment.

### ❌ Cons
- Harder to query across services (need API composition or CQRS).
- Distributed transactions problem (requires Saga/Outbox/2PC).
- Higher ops complexity (many databases to manage).

### 🔹 Best Use Case
- Large-scale, domain-driven microservices.
- Teams requiring **independent scaling and deployment**.

---

## 🔐 Hybrid Approaches
- **CQRS + Event Sourcing** → for read-heavy systems.
- **API Composition** → aggregate data across services.
- **Database-per-service + Shared Read Replica** → isolate writes but share read-only data.

---

## 🔹 Comparison

| Feature                  | Shared DB 🏢               | DB per Service 🗂️       |
|---------------------------|---------------------------|--------------------------|
| Coupling                 | High                     | Low                      |
| Scalability              | Limited                  | Independent              |
| Schema Evolution         | Risky (one change breaks all) | Safe per service   |
| Cross-Service Queries    | Easy (JOINs)             | Hard (API/Event needed)  |
| Autonomy                 | Weak                     | Strong                   |
| Ops Complexity           | Low                      | High                     |

---

## 👉 Rule of Thumb
- Start with **DB per service** for true independence.  
- Use **Shared DB** only as a temporary solution or when the system is small/simple.  
- For cross-service queries, rely on **API Composition or CQRS/Event Sourcing** instead of direct DB joins.

# Question 10. How to implement observability? (Logs, Metrics, Tracing with Zipkin/Jaeger).

# 🔍 Observability in Microservices

Observability is about **understanding the internal state of your system** by collecting **logs, metrics, and traces**.  
It helps in debugging, monitoring, and improving system reliability.

---

## 1. **Logging**
- **Purpose**: Capture events and errors for troubleshooting.
- **Tools**: Logback, Log4j2, ELK stack (Elasticsearch, Logstash, Kibana), Loki.
- **Best Practices**:
  - Structured logs (JSON format) for easier parsing.
  - Include **traceId, spanId** to correlate across services.
  - Avoid sensitive data in logs.

```java
// Example with Spring Boot + SLF4J
@Slf4j
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable String id) {
        log.info("Fetching user with id {}", id);
        return userService.getUser(id);
    }
}
```
## 2. Metrics
- **Purpose**: Measure system health and performance (CPU, memory, request rate, error rate).
- **Tools**: Micrometer (Spring Boot), Prometheus, Grafana.
- **Best Practices**:
  - Track request latency, throughput, error counts.
  - Use tags/labels to differentiate services, endpoints, or status codes.
```java
// Example: Micrometer counter in Spring Boot
@Autowired
private MeterRegistry meterRegistry;

public void processRequest() {
    meterRegistry.counter("requests_processed", "service", "user").increment();
}
```
## 3. Distributed Tracing
- **Purpose**: Track requests as they flow through multiple services.
- **Tools**: Zipkin, Jaeger, OpenTelemetry.
- **Best Practices**:
  - Propagate traceId and spanId across service calls.
  - Instrument key service calls and external HTTP/gRPC calls.
  - Visualize request paths to identify bottlenecks.
```java
// Spring Cloud Sleuth + Zipkin integration
// application.properties
spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.probability=1.0
```
## 4. 🔹 Integration Flow
```text
[Client Request]
      |
      v
[Microservice A] --log, metrics, trace--> [Observability stack]
      |
      v
[Microservice B] --log, metrics, trace--> [Observability stack]
      |
      v
[Monitoring/Tracing Dashboards: Kibana, Grafana, Zipkin/Jaeger]
```
- Logs → centralized storage (ELK/Loki).
- Metrics → Prometheus → Grafana dashboards.
- Traces → Zipkin/Jaeger UI for request flow visualization.

👉 Rule of Thumb
- Always include structured logs + correlation IDs.
- Track metrics for SLA and reliability monitoring.
- Use distributed tracing to understand end-to-end request flow.
- Combine logs, metrics, and traces for full observability.