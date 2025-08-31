# Question 1. Explain the difference between EntityManager and Session.
# 🗄️ EntityManager vs Session (JPA vs Hibernate)

When working with JPA and Hibernate, you’ll often see **`EntityManager`** (JPA standard) and **`Session`** (Hibernate-specific).  
Both represent the persistence context, but there are key differences.

---

## 1. **Definition**
- **EntityManager**  
  - Part of **JPA specification** (`javax.persistence.EntityManager` / `jakarta.persistence.EntityManager`).  
  - Standardized API to manage entities in persistence context.  
  - Portable across JPA providers (Hibernate, EclipseLink, OpenJPA, etc.).

- **Session**  
  - Hibernate’s **native API** (`org.hibernate.Session`).  
  - Provides **all EntityManager capabilities**, plus **extra Hibernate-specific features**.

---

## 2. **Persistence Context**
- Both maintain a **first-level cache** (the persistence context).
- Both track entity states: **transient → persistent → detached → removed**.

---

## 3. **Operations**
- **EntityManager**:
  - `persist(entity)`
  - `merge(entity)`
  - `remove(entity)`
  - `find(Entity.class, id)`
  - `createQuery("JPQL...")`
  - Transactions typically managed by `@Transactional`.

- **Session** (superset of EntityManager):
  - Includes all JPA methods, but adds:  
    - `save()`, `saveOrUpdate()`, `update()`  
    - `get()` vs `load()` (lazy proxy vs immediate fetch)  
    - `createCriteria()`, `createNativeQuery()`  
    - Scrollable results, batch operations, etc.

---

## 4. **Transaction Management**
- **EntityManager**: Works with **`EntityTransaction`** or container-managed transactions.  
- **Session**: Uses **`Transaction`** object from Hibernate.  
- In Spring, both integrate with `@Transactional`.

---

## 5. **Query Language**
- **EntityManager**: Uses **JPQL** (Java Persistence Query Language).  
- **Session**: Supports **JPQL + HQL (Hibernate Query Language)** + **Criteria API** + **native SQL**.

---

## ✅ Summary Table

| Aspect                | EntityManager (JPA)                   | Session (Hibernate)                  |
|------------------------|----------------------------------------|---------------------------------------|
| API Source            | JPA standard (`jakarta.persistence`)   | Hibernate-specific (`org.hibernate`)  |
| Portability           | Portable across providers              | Tied to Hibernate                     |
| Features              | Basic CRUD, JPQL                      | All EntityManager + advanced features |
| Query Support         | JPQL                                  | JPQL, HQL, Criteria, SQL              |
| Transactions          | `EntityTransaction` / container        | Hibernate `Transaction`               |

---

## 👉 Best Practice
- Use **`EntityManager`** in your business code → portable, standard JPA.  
- Use **`Session`** only when you need **Hibernate-specific features** (batch processing, advanced criteria, scrollable results).  
- In Spring Boot, you often inject `EntityManager`, but you can unwrap a `Session`:

```java
@PersistenceContext
private EntityManager entityManager;

public void example() {
    Session session = entityManager.unwrap(Session.class);
    // Now you can use Hibernate-specific APIs
}
```
# Question 2. What are the different types of fetching strategies (lazy vs eager)?
# ⚡ Fetching Strategies in JPA/Hibernate (Lazy vs Eager)

When dealing with relationships (`@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`), JPA/Hibernate controls **when related entities are loaded** from the database.  
There are **two main fetching strategies**:

---

## 1. **Lazy Fetching**
- **Definition**: The associated entity/collection is **not loaded immediately**.  
  It’s loaded **only when accessed** for the first time.
- **Annotation**: `fetch = FetchType.LAZY` (default for collections).
- **Pros**:
  - Improves performance by **loading only what you need**.
  - Reduces initial queries if associations are rarely used.
- **Cons**:
  - May cause **LazyInitializationException** if accessed outside the persistence context (e.g., after session is closed).
- **Example**:

```java
@Entity
class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders; // Loaded only when getOrders() is called
}
```
## 2. **Eager Fetching**
- **Definition**: The associated entity/collection is loaded immediately with the parent entity (via join).
- **Annotation**: fetch = FetchType.EAGER (default for @ManyToOne and @OneToOne).
- **Pros:**:
  - Prevents LazyInitializationException (data already loaded).
  - Prevents LazyInitializationException (data already loaded).
- **Cons**:
  - Can lead to N+1 query problem or fetching large object graphs unnecessarily.
  - Hurts performance if associations are rarely used.
- **Example**:
```java
@Entity
class User {
    @OneToOne(fetch = FetchType.EAGER)
    private Profile profile; // Always fetched with User
}
```
## 🔹 Default Fetch Types

| Association Type | Default Fetch Type |
|------------------|---------------------|
| `@OneToMany`     | **LAZY** |
| `@ManyToMany`    | **LAZY** |
| `@ManyToOne`     | **EAGER** |
| `@OneToOne`      | **EAGER** |

## 🔹 Best Practices

1. **Use `LAZY` by default** for performance.  
   → Avoid fetching unnecessary data.

2. **Use `EAGER` only when you always need the data**.  
   → Prevents N+1 query issues, but can load too much data.

3. **Fetch lazily-loaded associations safely**:
   - ✅ **JOIN FETCH** in JPQL/HQL  
   - ✅ **EntityGraph (`@NamedEntityGraph`)**  
   - ✅ **DTO projections** (Spring Data JPA)

---

## 🔎 Example with `JOIN FETCH`

```java
// Entity
@Entity
public class Order {
    @Id
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY) // Lazy by default
    private Customer customer;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();
}
```
```java
// Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Use JOIN FETCH to load customer and items in a single query
    @Query("SELECT o FROM Order o JOIN FETCH o.customer JOIN FETCH o.items WHERE o.id = :id")
    Order findOrderWithDetails(@Param("id") Long id);
}
```
## 🔹 SQL Generated
```java
SELECT o.*, c.*, i.* 
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items i ON i.order_id = o.id
WHERE o.id = ?
```
# ✅ Summary of JPA Fetch Strategies

- **Default Behavior**  
  - Collections (`@OneToMany`, `@ManyToMany`) → **LAZY**  
  - Single associations (`@OneToOne`, `@ManyToOne`) → **EAGER**  

- **Best Practice**  
  - Prefer **LAZY everywhere** → gives better performance & avoids unnecessary queries.  
  - Override to **EAGER only** when the association is *always* required.  

- **Safe Ways to Load Data**  
  - 🔹 `JOIN FETCH` in JPQL/HQL  
  - 🔹 `@EntityGraph` / `@NamedEntityGraph`  
  - 🔹 DTO projections in Spring Data JPA  

👉 Rule of Thumb: **Load only what you need, when you need it.**

# Question 3. Explain N+1 select problem and how to solve it.
# 🔎 N+1 Select Problem in JPA/Hibernate

## 1. What is the N+1 Problem?
- Happens when fetching a parent entity with **lazy associations**.  
- Hibernate executes **1 query for the parent** + **N queries for children** (one for each parent row).  
- Leads to poor performance on large datasets.

### Example
```java
List<Department> departments = em.createQuery("SELECT d FROM Department d", Department.class)
                                .getResultList();

for (Department d : departments) {
    // Triggers 1 extra query per department for employees (lazy load)
    System.out.println(d.getEmployees().size());
}
```
**SQL Executed:**
- SELECT * FROM department; -- 1 query
- SELECT * FROM employee WHERE dept_id=?; -- N queries (one per department)
⚠️ Total = N+1 queries → inefficient.
## 2. Solutions
### ✅ Option 1: JOIN FETCH (JPQL)
```java
List<Department> departments = em.createQuery(
    "SELECT d FROM Department d JOIN FETCH d.employees", Department.class)
    .getResultList();
```
### ✅ Option 2: @EntityGraph
```java
@EntityGraph(attributePaths = "employees")
@Query("SELECT d FROM Department d")
List<Department> findAllWithEmployees();
```
- Declarative way, avoids manual JOIN FETCH.
### ✅ Option 3: DTO Projections
```java
@Query("SELECT new com.example.DepartmentDTO(d.name, e.name) " +
       "FROM Department d JOIN d.employees e")
List<DepartmentDTO> findDepartmentEmployeeDTOs();
```
- Fetches only required fields, avoids loading full entities.
### ✅ Option 4: Batch Fetching (Hibernate-specific)
```java
@Entity
class Department {
    @OneToMany(mappedBy = "department")
    @BatchSize(size = 10)
    private List<Employee> employees;
}
```
- Reduces queries to 1 + (N / batch_size).

## 3. Best Practices
- Use JOIN FETCH or EntityGraph when you know you need the children.
- Use DTO projection for read-only queries.
- Avoid EAGER fetching by default (can cause other performance issues).
- Consider Hibernate’s @BatchSize for bulk fetching optimization.
### ✅ Summary
- Problem: N+1 = one parent query + N child queries.
- Solution: JOIN FETCH, EntityGraph, DTO projections, or batch fetching.
- Rule: Fetch only what you need, avoid blind EAGER loading.
  
# Question 4. What is optimistic vs pessimistic locking? How does JPA handle it?
# 🔒 Optimistic vs Pessimistic Locking in JPA

## 1. Optimistic Locking
- **Assumption**: Conflicts are **rare**.
- No DB-level locks; multiple transactions can read the same row.
- When updating, JPA checks if the entity was modified by someone else (via a **version field**).
- If version mismatch → **OptimisticLockException**.

### How it works
```java
@Entity
public class Product {
    @Id
    private Long id;

    private String name;

    @Version   // Used for optimistic locking
    private Integer version;
}
```

Flow:
1. Transaction A loads Product (version = 1).
2. Transaction B also loads Product (version = 1).
3. Transaction A updates Product → version becomes 2.
4. Transaction B tries to update Product → detects version mismatch → exception.

Pros:
- No DB lock → better concurrency.
- Works well in systems with low contention.

Cons:
- Updates may fail & require retry logic.

## 2. Pessimistic Locking
- Assumption: Conflicts are likely.
- DB row is locked (using SQL SELECT ... FOR UPDATE or similar).
- Other transactions must wait or fail when accessing the locked row.

**How it works**
```java
Product product = em.find(Product.class, id, LockModeType.PESSIMISTIC_WRITE);
```
Lock types:
- PESSIMISTIC_READ → others can read but not write.
- PESSIMISTIC_WRITE → exclusive lock (no read/write until released).
- PESSIMISTIC_FORCE_INCREMENT → also increments version (combination with optimistic).

Pros:
- Prevents conflicts at the cost of reduced concurrency.
-Safer for high-contention updates.

Cons:
- DB locks reduce throughput.
- Higher risk of deadlocks.

## 3. JPA Handling
Optimistic Locking
- Requires a @Version field.
- JPA automatically:
    - Adds version column in UPDATE/DELETE.
    - Throws OptimisticLockException if no rows updated.

Example SQL:
```java
UPDATE product SET name=?, version=version+1 WHERE id=? AND version=?;
```
Pessimistic Locking
- Use EntityManager or Spring Data:
```java
// JPA
em.find(Product.class, id, LockModeType.PESSIMISTIC_WRITE);

// Spring Data
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Product> findById(Long id);
```
### 4. When to Use?

| **Locking Type** | **Use Case**                                                                 |
|------------------|-------------------------------------------------------------------------------|
| **Optimistic**   | Default choice. Works best for low-conflict systems (e.g., web applications). |
| **Pessimistic**  | Use when conflicts are frequent or business rules require strict ordering (e.g., bank transfers). |

✅ Summary
- Optimistic → version check, no DB locks, better scalability.
- Pessimistic → DB-level locks, safe but less scalable.
- JPA supports both via @Version (optimistic) and LockModeType (pessimistic).

# Question 5. Difference between `CascadeType.ALL` and `orphanRemoval = true`.

# ⚖️ Difference between `CascadeType.ALL` and `orphanRemoval = true` in JPA

## 1. `CascadeType.ALL`
- Means **all entity operations** (persist, merge, remove, refresh, detach) on a **parent** are cascaded to its **children**.
- It controls **what happens to the child when the parent changes**.

### Example
```java
@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
}

@Entity
public class OrderItem {
    @Id
    private Long id;

    @ManyToOne
    private Order order;
}
```
Behavior:
- Saving Order → all OrderItem are saved.
- Deleting Order → all OrderItem are deleted.
- Updating Order → cascades to OrderItem.
👉 Cascading = "apply parent operation to children."

## 2. orphanRemoval = true
- Deals with entity lifecycle when a child is removed from the parent's collection.
- If orphanRemoval = true, removing the child from the parent’s collection deletes it from the database automatically.
- Without it, the child is just disassociated but still exists in the database.

Example
```java
@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```
Behavior:
```java
order.getItems().remove(item); 
// With orphanRemoval=true → item is deleted from DB
// Without orphanRemoval → item.order_id = NULL in DB (still exists)

```
👉 Orphan removal = "delete child when it’s no longer referenced by the parent."

### 🔑 3. Key Differences: `CascadeType.ALL` vs `orphanRemoval = true`

| Feature              | `CascadeType.ALL`                                      | `orphanRemoval = true`                                    |
|----------------------|--------------------------------------------------------|-----------------------------------------------------------|
| **Purpose**          | Propagates parent operations to children               | Deletes child when removed from parent’s collection        |
| **When triggered?**  | On `persist`, `merge`, `remove`, etc. of **parent**    | When child is **dereferenced** from parent                |
| **Example**          | Saving/deleting parent → child also saved/deleted      | `order.getItems().remove(item)` → item deleted from DB     |
| **Works without remove()?** | ✅ Yes (propagation happens automatically)        | ❌ No, only when explicitly removed from collection        |
| **Relationship meaning** | Lifecycle tied but not strict ownership             | Parent is the **true owner** of child’s lifecycle          |

---


### ✅ Summary
- CascadeType.ALL → "Do with children whatever you do with parent."
- orphanRemoval = true → "If child is kicked out of parent’s collection, delete it."
- They are often used together to ensure children are managed fully by parent.

# Question 6. Explain `@JoinColumn` vs `@JoinTable`.

# 🔎 `@JoinColumn` vs `@JoinTable` in JPA

## 1. `@JoinColumn`
- Used when the relationship is mapped with a **foreign key column** in one table.
- Common in **@ManyToOne** and **@OneToOne** associations.
- Simpler, because it just adds a **column** in the owning entity’s table.

### Example: `@ManyToOne` with `@JoinColumn`
```java
@Entity
class Order {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "customer_id") // FK in orders table
    private Customer customer;
}
```
📌 orders table will have a column customer_id as a foreign key.

## 2. @JoinTable
- Used when the relationship is mapped with a separate join table.
- Common in @ManyToMany associations, but can also be used in @OneToMany or @OneToOne.
- Provides more flexibility (e.g., if you want to store extra columns like timestamps, roles, etc.).
Example: @ManyToMany with @JoinTable
```java
@Entity
class Student {
    @Id @GeneratedValue
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "student_course", // join table name
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}
```
📌 Here, JPA creates a student_course join table with columns student_id and course_id.
# ✅ Summary: `@JoinColumn` vs `@JoinTable`

| Feature        | `@JoinColumn`                                      | `@JoinTable`                                     |
|----------------|----------------------------------------------------|--------------------------------------------------|
| **Where used?**| `@ManyToOne`, `@OneToOne` (sometimes others)       | Mainly `@ManyToMany` (can be used in others)     |
| **Implementation** | Adds a **foreign key column**                  | Creates a **new join table**                     |
| **Complexity** | Simpler                                            | More flexible, supports extra metadata           |
| **Use case**   | FK in one table is enough                          | Relationship needs a separate table              |

👉 **Rule of thumb:**
- Use **`@JoinColumn`** for simple parent-child relations.  
- Use **`@JoinTable`** for many-to-many or when you want a separate table to represent the association.

# Question 7. How does `@Inheritance` work in JPA? (SINGLE_TABLE, JOINED, TABLE_PER_CLASS).
# 🔑 `@Inheritance` in JPA

JPA allows mapping an **entity inheritance hierarchy** to relational tables.  
You annotate the **base entity class** with `@Inheritance(strategy = …)`.

---

## 1. `SINGLE_TABLE` (default)

- **All classes** in the hierarchy are stored in **one table**.
- A **discriminator column** (like `DTYPE`) is used to distinguish subtypes.
- ✅ **Pros**: Fast queries, simple schema.  
- ❌ **Cons**: Lots of `NULL` columns for fields not used by all subclasses.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
public abstract class Payment { ... }

@Entity
public class CreditCardPayment extends Payment { ... }

@Entity
public class BankTransferPayment extends Payment { ... }
```
### 🗄️ Example: Generated Table (SINGLE_TABLE strategy)

| id | amount | dtype               | cardNumber | bankAccount |
|----|--------|---------------------|------------|-------------|
| 1  | 100    | CreditCardPayment   | 1234       | NULL        |
| 2  | 200    | BankTransferPayment | NULL       | ABC123      |
### 🏗️ JOINED Strategy

- Each entity has its own table.  
- Subclass table contains only subclass-specific fields and a **foreign key** to the parent table.

✅ **Pros**:  
- Normalized schema (no nullable columns).  
- Cleaner data model.  

❌ **Cons**:  
- Requires `JOIN` queries to fetch subclass data → can be slower.  

---

**Example Tables:**

**Payment (Parent Table)**  
| id | amount |
|----|--------|
| 1  | 100    |
| 2  | 200    |

**CreditCardPayment (Child Table)**  
| id | cardNumber |
|----|------------|
| 1  | 1234       |

**BankTransferPayment (Child Table)**  
| id | bankAccount |
|----|-------------|
| 2  | ABC123      |

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Payment { ... }

@Entity
public class CreditCardPayment extends Payment { ... }

@Entity
public class BankTransferPayment extends Payment { ... }
```
Generated Tables:
- payment(id, amount)
- credit_card_payment(id, cardNumber)
- bank_transfer_payment(id, bankAccount)
Querying a CreditCardPayment does a JOIN between tables.

### 3. TABLE_PER_CLASS
Each subclass has its own table, with all fields (inherited + specific).
No discriminator column, no joins.
✅ Pros: Fast subclass queries.
❌ Cons: Duplicated columns across tables, harder to query polymorphically (UNIONs).
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Payment { ... }

@Entity
public class CreditCardPayment extends Payment { ... }

@Entity
public class BankTransferPayment extends Payment { ... }
```
Generated Tables:
- credit_card_payment(id, amount, cardNumber)
- bank_transfer_payment(id, amount, bankAccount)
No payment table.
✅ **Summary**

| Strategy        | Tables | Performance                              | Schema        | Use Case                     |
|-----------------|--------|------------------------------------------|---------------|------------------------------|
| **SINGLE_TABLE** | 1      | Fast                                     | Null columns  | Small/simple hierarchies     |
| **JOINED**       | Many   | Medium                                   | Normalized    | Complex hierarchies          |
| **TABLE_PER_CLASS** | Many   | Fast for subclasses, slow for polymorphic queries | Duplicate data | Rare, use sparingly |

👉 **Rule of thumb:**  
- Use **SINGLE_TABLE** by default (fast, simple).  
- Use **JOINED** if you care about normalization.  
- Avoid **TABLE_PER_CLASS** unless you have a strong reason.  

# Question 8. How does the first-level and second-level cache work in Hibernate?

# Hibernate Caching: First-Level vs Second-Level

## 🔹 1. First-Level Cache (Session Cache)
- **Scope**: Tied to a **Hibernate `Session`** (one unit of work).
- **Enabled by default**: Always on, cannot be disabled.
- **How it works**:
  - Hibernate checks the **session cache** before hitting the DB.
  - If entity is found → returned from cache.
  - If not → query DB, then store entity in cache.
  - Same session + same ID = no extra DB query.

### Example
```java
Session session = sessionFactory.openSession();

// First query → hits DB
User user1 = session.get(User.class, 1);

// Second query (same session) → from cache
User user2 = session.get(User.class, 1);

System.out.println(user1 == user2); // true
session.close();
```

### Cache Control
session.evict(entity) → remove one entity from cache.
session.clear() → clear all entities from cache.
Closing the session destroys the cache.

✅ Best for transaction-level caching.

## 🔹 2. Second-Level Cache (SessionFactory Cache)
- Scope: Shared across multiple sessions (application-wide).
- Not enabled by default → must configure with a provider.
- Purpose: Store entities/collections across sessions to reduce DB load.

Providers
- Ehcache
- Infinispan
- Caffeine
- Hazelcast, etc.
### Configuration (Spring Boot)
```properties
# Enable 2nd level cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

# Enable query cache (optional)
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

Entity Annotation

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    @Id
    private Long id;
    private String name;
}
```
```java
Session s1 = sessionFactory.openSession();
User user1 = s1.get(User.class, 1); // DB hit, stored in L1 + L2
s1.close();

Session s2 = sessionFactory.openSession();
User user2 = s2.get(User.class, 1); // From L2 cache, no DB hit
s2.close();
```
✅ Best for read-mostly data (reference/lookup tables).
## 🔹 Key Differences

| Feature              | First-Level Cache (L1)        | Second-Level Cache (L2)          |
|----------------------|-------------------------------|----------------------------------|
| **Scope**            | Session (per transaction)     | SessionFactory (application-wide)|
| **Enabled by default** | ✅ Yes                       | ❌ No (configure manually)        |
| **Life cycle**       | Ends when session closes      | Persists across sessions          |
| **Usage**            | Avoid repeated queries in same session | Reduce DB load across sessions |
| **Providers**        | Hibernate internal            | Ehcache, Infinispan, etc.        |

---

👉 **Summary**  
- **First-level cache**: always on, session-scoped.  
- **Second-level cache**: optional, shared across sessions, requires provider.

# Question 9. How do you handle database migrations in Spring Boot? (Liquibase/Flyway).

# 🔹 Database Migrations in Spring Boot (Flyway / Liquibase)

Spring Boot supports both **Flyway** and **Liquibase** out of the box.

---

## 1. Flyway

### ✅ How it Works
- Uses **versioned SQL or Java migration scripts**.
- Scripts are executed in order based on version numbers.
- Tracks applied migrations in a table (`flyway_schema_history`).

### ⚙️ Setup
**Maven**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```
Gradle
```gradle
implementation 'org.flywaydb:flyway-core'
```
📂 Folder Structure
By default, Flyway looks for scripts in:
```java
src/main/resources/db/migration/
```
📝 Migration File Example
```sql
-- V1__create_user_table.sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);

-- V2__add_age_column.sql
ALTER TABLE users ADD age INT;
```
⚙️ Application Properties
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```

## 2. Liquibase
✅ How it Works
Uses XML, YAML, JSON, or SQL "changelogs" to define schema changes.
Tracks changes in databasechangelog and databasechangeloglock tables.
More descriptive and flexible than Flyway (supports rollback, complex changes, etc.).
⚙️ Setup
Maven
```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```
Gradle
```gradle
implementation 'org.liquibase:liquibase-core'
```
📂 Changelog File Example (XML)
src/main/resources/db/changelog/db.changelog-master.xml
```xml
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
      http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet id="1" author="thuan">
        <createTable tableName="users">
            <column name="id" type="BIGINT" autoIncrement="true" primaryKey="true"/>
            <column name="username" type="VARCHAR(50)"/>
            <column name="email" type="VARCHAR(100)"/>
        </createTable>
    </changeSet>

    <changeSet id="2" author="thuan">
        <addColumn tableName="users">
            <column name="age" type="INT"/>
        </addColumn>
    </changeSet>

</databaseChangeLog>
```
⚙️ Application Properties
```properties
spring.liquibase.enabled=true
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
```
## 🔹 Flyway vs Liquibase

| Feature             | Flyway 🐦                        | Liquibase 🧩                    |
|---------------------|----------------------------------|---------------------------------|
| **Script Type**     | Mostly SQL (Java possible)       | XML, YAML, JSON, or SQL         |
| **Learning Curve**  | Easy, lightweight                | More complex, flexible          |
| **Rollback Support**| Limited                          | Strong rollback capabilities    |
| **Tracking Tables** | `flyway_schema_history`           | `databasechangelog`             |
| **Best For**        | Simple migrations, fast adoption | Complex migrations, enterprise  |

---

👉 **Rule of Thumb**  
- Use **Flyway** if you want something **simple, SQL-based, and fast**.  
- Use **Liquibase** if you need **rollback, cross-database support, or complex migration logic**.

# Question 10. How do you optimize JPA queries for large datasets?

# 🔹 Optimizing JPA Queries for Large Datasets

When working with **JPA/Hibernate** on large datasets, performance tuning is critical.  

---

## 1. Use Pagination (`setFirstResult` / `setMaxResults`)
Never fetch millions of rows into memory at once. Process results in **pages**.

```java
TypedQuery<User> query = em.createQuery("SELECT u FROM User u", User.class);
query.setFirstResult(0);      // offset
query.setMaxResults(1000);    // limit (page size)
List<User> users = query.getResultList();
```
✅ Keeps memory usage low.
✅ Works well for APIs (return data page by page).
## 2. Use Projections (DTOs or scalar queries)
Fetching entire entities (with all fields and relationships) is costly.
If you only need a subset of fields, use projections.

```java
// DTO projection
@Query("SELECT new com.example.dto.UserDTO(u.id, u.username) FROM User u")
List<UserDTO> findAllUsernames();
```
✅ Avoids unnecessary column loads.
✅ Faster queries, less serialization overhead.

## 3. Avoid N+1 Problem (use JOIN FETCH or EntityGraph)
JPA defaults to lazy loading, which can trigger many small queries.

```java
// Bad: triggers N+1 queries
List<User> users = em.createQuery("SELECT u FROM User u").getResultList();
for (User u : users) {
    System.out.println(u.getOrders().size()); // lazy load = N queries
}

// Good: JOIN FETCH
List<User> users = em.createQuery(
    "SELECT u FROM User u JOIN FETCH u.orders"
).getResultList();
```
Or with EntityGraph:
```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```
✅ Reduces queries from N+1 → 1.
## 4. Use Batch Fetching (for lazy collections)
If you must fetch many entities/collections, configure batch size:
```java
spring.jpa.properties.hibernate.default_batch_fetch_size=50
```

## 5. Use Streaming for Huge Reads
When processing millions of rows, use streaming instead of loading into memory.

```java
Stream<User> stream = em.createQuery("SELECT u FROM User u", User.class)
                        .setHint(QueryHints.HINT_FETCH_SIZE, 1000)
                        .getResultStream();

stream.forEach(user -> {
    // process user
    em.detach(user); // free memory
});
```

✅ Processes rows in chunks.
✅ Prevents OutOfMemoryError.

## 6. Use Indexes in the Database
- Add indexes to columns used in WHERE, JOIN, and ORDER BY.
- Use @Index on entities:

```java
@Entity
@Table(indexes = @Index(name = "idx_user_email", columnList = "email"))
public class User { ... }
```

✅ Drastically speeds up query execution.

## 7. Use Read-Only Transactions for Reporting
If you only read data, tell Hibernate not to track entities:

```java
@Transactional(readOnly = true)
public List<User> findUsers() {
    return em.createQuery("SELECT u FROM User u", User.class)
             .setHint("org.hibernate.readOnly", true)
             .getResultList();
}
```
Or globally:

```properties
spring.jpa.properties.hibernate.readOnly=true
```
✅ Skips dirty-checking → better performance.

## 8. Consider Native SQL for Heavy Analytics
JPA is great for CRUD, but for complex aggregations, joins, analytics,
use native queries or database views.
```java
@Query(value = "SELECT u.id, COUNT(o.id) as order_count " +
               "FROM users u JOIN orders o ON u.id=o.user_id " +
               "GROUP BY u.id",
       nativeQuery = true)
List<Object[]> findUserOrderCounts();
```

✅ Lets the database do the heavy lifting.

### 9. Cache Strategically
- Use first-level cache (Session) automatically.
- Enable second-level cache for frequently read entities.
- Use query cache for common read queries.

```java
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
@Entity
public class Country { ... }
```

✅ Reduces DB hits for reference data.

---

## 🔹 Summary of Best Practices

| Strategy                     | Benefit                          |
|------------------------------|----------------------------------|
| **Pagination**               | Prevents memory overflow         |
| **DTO projections**          | Loads only needed columns        |
| **JOIN FETCH / EntityGraph** | Avoids N+1 queries               |
| **Batch fetching**           | Reduces lazy load queries        |
| **Streaming with fetch size**| Handles very large datasets      |
| **Indexes**                  | Speeds up DB lookups             |
| **Read-only transactions**   | Skips unnecessary tracking       |
| **Native SQL for analytics** | Offloads work to DB              |
| **Caching (L2, query cache)**| Reduces DB hits                  |

---

👉 **Rule of Thumb**  
- For **read-heavy apps** → optimize with **projections, batching, caching**.  
- For **write-heavy apps** → focus on **indexes, batching inserts/updates, and transaction boundaries**.
