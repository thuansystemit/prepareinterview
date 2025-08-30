# Question 1. Differences between `HashMap`, `Hashtable`, and `ConcurrentHashMap`

---

## 1. `HashMap`
- **Thread-safety**: ❌ Not thread-safe. Race conditions if accessed by multiple threads without synchronization.
- **Performance**: ✅ Very fast in single-threaded use (no synchronization overhead).
- **Nulls**: Allows **one `null` key** and multiple `null` values.
- **Iteration**: Fail-fast → throws `ConcurrentModificationException` if structurally modified while iterating.
- **Use case**: General-purpose, single-threaded applications (or use with external synchronization).

---

## 2. `Hashtable`
- **Thread-safety**: ✅ Thread-safe (all methods synchronized).
- **Performance**: ❌ Slower than `HashMap` (method-level synchronization).
- **Nulls**: ❌ No `null` keys or values allowed.
- **Iteration**: Fail-fast (since Java 1.8+).
- **Legacy**: Considered legacy (Java 1.0). Superseded by `ConcurrentHashMap`.
- **Use case**: Rarely recommended today.

---

## 3. `ConcurrentHashMap`
- **Thread-safety**: ✅ Thread-safe, more efficient than `Hashtable`.
  - Java 7: bucket-level locking (segments).
  - Java 8+: fine-grained locks + CAS (compare-and-swap).
- **Performance**: ✅ High performance in multi-threaded environments.
- **Nulls**: ❌ No `null` keys or values.
- **Iteration**: Weakly consistent (does not throw `ConcurrentModificationException`, reflects snapshot).
- **Use case**: Best for **multi-threaded** and **high-concurrency** scenarios.

---

## 🔑 Key Differences Table

| Feature              | HashMap          | Hashtable           | ConcurrentHashMap     |
|----------------------|------------------|---------------------|-----------------------|
| Thread-safe?         | ❌ No           | ✅ Yes (synchronized)| ✅ Yes (fine-grained) |
| Performance          | ✅ Fast (single) | ❌ Slow             | ✅ Faster (concurrent)|
| Null key allowed?    | ✅ One          | ❌ None             | ❌ None              |
| Null values allowed? | ✅ Multiple     | ❌ None             | ❌ None              |
| Iteration type       | Fail-fast        | Fail-fast           | Weakly consistent     |
| Introduced in        | Java 1.2         | Java 1.0 (legacy)   | Java 5 (modern)       |

---

## 👉 Rule of Thumb
- **Single-threaded** → `HashMap`
- **Multi-threaded (low concurrency)** → `Collections.synchronizedMap(new HashMap<>())`
- **Multi-threaded (high concurrency)** → `ConcurrentHashMap`
- **Avoid** `Hashtable` in new code

# Question 2. How does the `volatile` keyword work in Java? When would you use it?
# `volatile` in Java

The `volatile` keyword is a **variable modifier** in Java. It tells the JVM and threads:  

---

## 1. Visibility Guarantee
- Normally, threads may cache variables in their local CPU caches.  
- If one thread updates a variable, other threads might not immediately see the change.  
- Marking a variable as `volatile` ensures that **all reads and writes go directly to main memory**, not a thread’s cache.  
- This guarantees that all threads always see the **latest value**.

---

## 2. No Reordering Guarantee (since Java 5)
- The JVM and CPU may reorder instructions for optimization.  
- `volatile` prevents certain kinds of instruction reordering involving the volatile variable.  
- This makes it useful for implementing safe **happens-before relationships**.

---

## ❌ What `volatile` Does NOT Do
- It does **not** make compound actions atomic.  

```java
volatile int counter = 0;

public void increment() {
    counter++; // ❌ Not atomic: read + increment + write
}
```
- For atomicity, use synchronized or AtomicInteger.

# When to Use volatile
## 1. Simple flags or state indicators

```java
class Worker extends Thread {
    private volatile boolean running = true;

    public void run() {
        while (running) {
            // do some work
        }
    }

    public void stopWorker() {
        running = false; // other threads see this immediately
    }
}

```

## 2. Double-checked locking for Singleton

```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // safe with volatile
                }
            }
        }
        return instance;
    }
}

```

## 3. Status flags / configuration variables
- Use when multiple threads read/write values that don’t require atomic operations.

## 🔑 Summary
- **Ensures visibility**: all threads see the latest value.  
- **Prevents reordering**: enforces happens-before guarantees.  
- **Not atomic**: use `AtomicXXX` or `synchronized` for compound operations.  
- **Best use cases**:  
  - flags  
  - stop signals  
  - configuration settings  
  - double-checked locking  

# Question 3. What are the key differences between synchronized and Lock API?
# 🔑 Key Differences: `synchronized` vs `Lock`

Both `synchronized` and the **`Lock` API** are used for thread synchronization in Java, but they differ in **capabilities, control, and flexibility**.

---

## Comparison Table

| Aspect | `synchronized` | `Lock` API |
|--------|----------------|------------|
| **Type** | A **keyword** built into the Java language. | An **interface** (`Lock`) with implementations (`ReentrantLock`, `ReentrantReadWriteLock`, etc.). |
| **Lock acquisition & release** | Automatically acquired at the start of the block/method and automatically released when the block/method exits (even if exceptions occur). | Must be acquired with `lock()` and released manually with `unlock()`. If `unlock()` is forgotten, deadlocks may occur. |
| **Fairness** | No fairness guarantees – threads compete randomly. | Some implementations (e.g., `ReentrantLock`) can be constructed with **fairness policies** (FIFO order). |
| **Interruptibility** | Threads waiting to acquire a synchronized lock **cannot be interrupted**. | Threads can attempt to acquire a lock in an **interruptible way** (`lockInterruptibly()`). |
| **Try without blocking** | Not possible. Thread blocks until lock is available. | Can use `tryLock()` to attempt acquiring the lock without waiting, or with a timeout (`tryLock(timeout, unit)`). |
| **Condition variables** | Can use only `wait()`, `notify()`, `notifyAll()` on the object monitor. | Provides multiple **`Condition` objects** via `newCondition()` for more flexible waiting/notification. |
| **Reentrant capability** | Implicitly **reentrant** – same thread can re-enter the lock. | `ReentrantLock` explicitly supports reentrancy. |
| **Performance** | Simple, less error-prone, optimized by JVM; good for most use cases. | More powerful for complex concurrency, but more verbose and requires careful handling. |
| **Debugging/Monitoring** | Limited monitoring via thread dumps (`synchronized` shows as BLOCKED). | Lock implementations provide methods like `isLocked()`, `getHoldCount()`, `hasQueuedThreads()`. |

---

## ✅ When to Use
- **Use `synchronized`** if:
  - Simpler synchronization is enough.
  - You want automatic lock release without boilerplate.
  - Readability and maintainability are more important.

- **Use `Lock` API** if:
  - You need **fairness policies**.
  - You want **tryLock / timed lock acquisition**.
  - You need **multiple conditions** per lock.
  - You need **interruptible lock acquisition**.
  - You want better control and monitoring of locks.

---

## 🔎 Example Comparison

### Using `synchronized`
```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```
# Question 4: Explain Java Memory Model (JMM) and `happens-before` relationship.
# 🧠 Java Memory Model (JMM) & Happens-Before Relationship

## 🔹 What is the Java Memory Model (JMM)?
- The **Java Memory Model (JMM)** defines how threads **interact through memory**.
- It specifies:
  1. **Visibility** → When one thread writes to a variable, when and how other threads can see it.
  2. **Ordering** → What order instructions appear to execute in across threads.
  3. **Atomicity** → Which operations are indivisible (cannot be interrupted).

⚠️ Without JMM, compilers, CPUs, and runtime optimizations could reorder instructions in ways that break multithreaded code.

---

## 🔹 Happens-Before Relationship
- A **happens-before** relationship is a guarantee by the JMM:
  > If action **A** happens-before action **B**, then:
  > - All effects (writes) of **A** are visible to **B**.
  > - **A** is executed before **B** in program order.

⚡ Happens-before is **not about time**, but about **visibility and ordering guarantees**.

---

## 🔹 Key Happens-Before Rules

1. **Program Order Rule**  
   - Within a single thread, statements appear to execute in order.  
   - Example:  
     ```java
     int a = 10;   // happens-before
     int b = a+1;  // this
     ```

2. **Monitor Lock Rule (synchronized)**  
   - Unlock on a monitor happens-before every subsequent lock on that monitor.  
   - Ensures visibility across threads.  
   - Example:  
     ```java
     synchronized(lock) { sharedVar = 1; } // unlock
     synchronized(lock) { int x = sharedVar; } // lock -> sees updated value
     ```

3. **Volatile Variable Rule**  
   - Write to a `volatile` field happens-before every subsequent read of that field.  
   - Example:  
     ```java
     volatile boolean flag = false;

     Thread 1: flag = true;   // happens-before
     Thread 2: if(flag) {...} // read sees true
     ```

4. **Thread Start Rule**  
   - `Thread.start()` happens-before the started thread begins execution.

5. **Thread Join Rule**  
   - All actions in a thread happen-before another thread successfully returns from `Thread.join()` on it.

6. **Transitivity Rule**  
   - If A happens-before B, and B happens-before C, then A happens-before C.

---

## 🔎 Example with Volatile
```java
class Example {
    private volatile boolean flag = false;

    public void writer() {
        flag = true; // happens-before the reader's check
    }

    public void reader() {
        if(flag) { // guaranteed to see true
            System.out.println("Flag is true");
        }
    }
}
```
# Question 5: Difference between `String`, `StringBuilder`, and `StringBuffer`.

# 📌 Difference between `String`, `StringBuilder`, and `StringBuffer`

## 🔑 Key Differences

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---------|----------|-----------------|----------------|
| **Mutability** | **Immutable** – once created, cannot be changed. | **Mutable** – can be modified without creating new objects. | **Mutable** – same as `StringBuilder`. |
| **Thread Safety** | ✅ Safe (immutable nature). | ❌ Not thread-safe. | ✅ Thread-safe (synchronized methods). |
| **Performance** | Slower for concatenation (creates new objects). | Faster than `String` and `StringBuffer` (no synchronization overhead). | Slower than `StringBuilder` (synchronization adds cost). |
| **Use Case** | Fixed text or rarely changing values. | When string is modified frequently in a **single-threaded** context. | When string is modified frequently in a **multi-threaded** context. |

---

## 🔎 Examples

### 1. `String` (Immutable)
```java
public class Example {
    public static void main(String[] args) {
        String s = "Hello";
        s = s + " World"; 
        // Creates a new String object, old one discarded
        System.out.println(s); // Output: Hello World
    }
}
```
## Summary

- **String** → Immutable, safe, but inefficient for repeated modifications.  
- **StringBuilder** → Mutable, fast, best for single-threaded string operations.  
- **StringBuffer** → Mutable, thread-safe, best for multi-threaded string operations.  


# Question 6. How do Garbage Collectors work in Java? Compare G1, CMS, and ZGC.
# ♻️ Garbage Collectors in Java

## 🔹 How Garbage Collectors Work
- **Garbage Collector (GC)** in Java automatically frees up memory by removing objects that are **no longer reachable** by any thread.
- Process generally involves:
  1. **Mark** → Find all live (reachable) objects.
  2. **Sweep** → Remove unreferenced (dead) objects.
  3. **Compact** → Rearrange objects to eliminate memory fragmentation (not always in all GCs).
- GC runs in the background and helps prevent memory leaks, but different algorithms balance **latency vs throughput vs footprint** differently.

---

## 🔹 Types of Garbage Collectors

### 1. **G1 (Garbage First) Collector**
- Default GC in **Java 9+**.
- Heap divided into **regions** (not just young/old gen).
- Uses **incremental parallel collection** of regions with most garbage first → reduces pause times.
- Performs background compaction → less fragmentation.
- **Best for**: Large heaps (multi-GB), balanced low-latency + throughput.

---

### 2. **CMS (Concurrent Mark-Sweep) Collector**
- Introduced before G1, mostly deprecated in Java 14+.
- Performs **concurrent marking** while application threads are running → reduces long pauses.
- Does **not compact** memory → may suffer from fragmentation.
- **Best for**: Low pause times, but not large heaps (can fragment, causing Full GC).
- Status: Deprecated (replaced by G1 and ZGC).

---

### 3. **ZGC (Z Garbage Collector)**
- Ultra-low latency collector (Java 11+).
- Works concurrently with the application → pause times usually **< 10ms**, regardless of heap size.
- Supports **very large heaps** (TB scale).
- Uses **colored pointers** and **load barriers** to manage references efficiently.
- **Best for**: Huge heaps, real-time or latency-sensitive applications.

---

## 🔹 Comparison Table

| Feature                | **G1**                          | **CMS**                       | **ZGC**                          |
|-------------------------|---------------------------------|-------------------------------|----------------------------------|
| **Pause times**         | Low (tens of ms)               | Low (but can spike)           | Ultra-low (<10 ms)               |
| **Heap size support**   | Multi-GB                       | Medium (up to a few GB)       | Very large (multi-TB)            |
| **Compaction**          | Yes (region-based)             | No (can fragment)             | Yes (concurrent)                 |
| **Throughput**          | High, balanced                 | Good, but suffers from compaction | High, designed for low-latency |
| **Concurrency**         | Parallel + Concurrent          | Concurrent mark, sweep        | Almost fully concurrent           |
| **Status**              | Default (since Java 9)         | Deprecated (Java 14)          | Experimental → Stable (Java 15+) |

---

## ✅ Summary
- **CMS** → Legacy low-pause collector, but suffers from fragmentation → deprecated.  
- **G1** → General-purpose default, good balance of throughput and low pause times.  
- **ZGC** → Next-gen, ultra-low pause times, handles huge heaps efficiently.  

# Question 7. What is the difference between checked and unchecked exceptions?
# ⚠️ Checked vs Unchecked Exceptions in Java

## 🔹 Checked Exceptions
- **Definition**: Exceptions that must be either **caught** or **declared** in the method signature using `throws`.
- Checked at **compile time** → compiler enforces handling.
- Usually represent **recoverable conditions** (things the programmer can anticipate and handle).
- **Examples**:
  - `IOException`
  - `SQLException`
  - `FileNotFoundException`

### Example
```java
import java.io.*;

public class CheckedExample {
    public static void main(String[] args) {
        try {
            FileReader fr = new FileReader("test.txt"); // FileNotFoundException
        } catch (IOException e) {
            System.out.println("File not found!");
        }
    }
}
```
## 🔹 Unchecked Exceptions

- **Definition**: Exceptions that do **not** need to be declared in the method signature or explicitly caught.  
- **Checked at**: Runtime (the compiler does not enforce handling).  
- **Purpose**: Usually represent **programming errors** such as logic mistakes or misuse of APIs.  
- **Examples**:  
  - `NullPointerException`  
  - `ArrayIndexOutOfBoundsException`  
  - `ArithmeticException`  
  - `IllegalArgumentException`  

```java
public class UncheckedExample {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3};
        System.out.println(numbers[5]); // ArrayIndexOutOfBoundsException at runtime
    }
}
```
## 🔹 Key Differences

| Aspect                | Checked Exceptions                              | Unchecked Exceptions                       |
|-----------------------|------------------------------------------------|--------------------------------------------|
| **Checked at**        | Compile time                                   | Runtime                                    |
| **Handling required?**| Yes (must catch or declare with `throws`)       | No (optional)                              |
| **Hierarchy**         | Subclass of `Exception` (but **not** `RuntimeException`) | Subclass of `RuntimeException`       |
| **Represents**        | Anticipated **recoverable issues**              | Programming errors / logic bugs            |
| **Examples**          | `IOException`, `SQLException`                  | `NullPointerException`, `ArithmeticException` |

## ✅ Summary

- **Checked exceptions** → Recoverable, must be declared or handled (**compile-time enforcement**).  
- **Unchecked exceptions** → Programming errors, not enforced by compiler, occur at **runtime**.  

# Question 8. How does `CompletableFuture` differ from `Future`?
# 🔑 Difference between `Future` and `CompletableFuture`

## 🔹 `Future` (Java 5)
- Introduced in **Java 5** (`java.util.concurrent`).
- Represents the result of an asynchronous computation.
- **Limitations**:
  - Cannot be manually completed.
  - Blocking methods (`get()`) → you wait until the task finishes.
  - No straightforward way to chain multiple tasks.
  - No built-in exception handling.

### Example with `Future`
```java
import java.util.concurrent.*;

public class FutureExample {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(1000);
            return 42;
        });

        // Blocks until result is available
        System.out.println(future.get());

        executor.shutdown();
    }
}
```
# Question 9. How does `Optional` help avoid `NullPointerException`? Any pitfalls?
# 🔑 How `Optional` Helps Avoid `NullPointerException`

## 🔹 What is `Optional`?
- `Optional<T>` is a **container object** that may or may not contain a non-null value.
- Instead of returning `null`, a method can return an `Optional<T>` to indicate **absence of a value**.

---

## 🔹 How it Helps
1. **Explicitly communicates absence**
   - Instead of returning `null`, methods return `Optional.empty()`.
   - Caller knows they must **deal with the absence case**.

2. **Safe access to value**
   - Use methods like `isPresent()`, `ifPresent()`, `orElse()`, `orElseGet()`, `orElseThrow()` instead of direct `null` checks.

3. **Functional style**
   - Supports chaining with `map`, `flatMap`, `filter`.

---

## 🔹 Example

### Without `Optional` (risk of NPE)
```java
public String getUserName(User user) {
    return user.getName(); // Risk of NullPointerException if user is null
}
```
### With Optional
```java
import java.util.Optional;

public String getUserName(Optional<User> user) {
    return user
            .map(User::getName)        // safely extract name if user is present
            .orElse("Unknown");        // default value if user is empty
}
```
* 👉 Here, Optional forces the caller to handle the "no value" case safely.
# 🔹 Java Optional

## 🔑 Common Methods
- `Optional.of(value)` → Wrap non-null value (**throws NPE if null**).
- `Optional.ofNullable(value)` → Wrap value or empty if null.
- `Optional.empty()` → Represents no value.
- `isPresent()` / `isEmpty()` → Check presence of value.
- `ifPresent(consumer)` → Run code if value is present.
- `orElse(default)` → Provide default value if empty.
- `orElseGet(supplier)` → Provide lazy default if empty.
- `orElseThrow(exceptionSupplier)` → Throw exception if empty.

---

## ⚠️ Pitfalls & Misuse
1. **Using `Optional.get()` directly**  
   - Throws `NoSuchElementException` if empty → defeats the purpose.  
   - ✅ Use `orElse()` / `orElseThrow()` instead.  

2. **Using `Optional` in fields**  
   - Not recommended (e.g., in JPA entities).  
   - Designed for **method return values**, not storage.  

3. **Serialization issues**  
   - Some frameworks (Jackson, Hibernate, JPA) may not handle `Optional` fields well.  

4. **Performance overhead**  
   - Small overhead from wrapping/unwrapping.  
   - Matters only in **tight loops / high-performance code**.  

5. **Overusing it**  
   - Not every nullable return should be `Optional`.  
   - Sometimes `null` + documentation is more appropriate.  
   - Best for **API boundaries** where absence of value is expected.  

---

## ✅ Summary
- `Optional` is a **safer alternative** to returning `null`.  
- Encourages **explicit handling** of missing values.  
- Avoid pitfalls: don’t misuse `get()`, don’t use in fields, and don’t overuse it.  

# Question 10. What is immutability? How do you create an immutable class?
# 🔹 Immutability in Java

## 📌 What is Immutability?
- An **immutable object** is an object whose state **cannot be changed** after it is created.
- All fields are set during construction and never modified later.
- Examples in Java: `String`, `Integer`, `LocalDate`.

---

## ✅ Benefits
- **Thread-safety** → Immutable objects can be safely shared across threads without synchronization.
- **Cache-friendly** → Can be reused freely since state never changes.
- **Predictability** → No unexpected changes → fewer bugs.

---

## 🛠️ How to Create an Immutable Class
Steps to follow:
1. **Declare the class as `final`** → prevents subclassing.
2. **Make all fields `private` and `final`** → ensures state cannot change after construction.
3. **Initialize fields via constructor** → no setters.
4. **No setters** → do not provide methods to modify fields.
5. **Defensive copies**  
   - For mutable objects (like `Date`, collections), return **copies** instead of direct references.

---

## 🧑‍💻 Example

```java
import java.util.Date;

public final class Person {
    private final String name;
    private final int age;
    private final Date birthDate; // mutable object

    public Person(String name, int age, Date birthDate) {
        this.name = name;
        this.age = age;
        // Defensive copy to protect immutability
        this.birthDate = new Date(birthDate.getTime());
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Date getBirthDate() {
        // Return defensive copy, not original
        return new Date(birthDate.getTime());
    }
}
```
# 🚫 Wrong Practices

- ❌ **Providing setters (`setName()`, `setAge()`)**  
  Breaks immutability since fields can be modified after creation.

- ❌ **Returning mutable objects directly (`birthDate`)**  
  Allows external code to change internal state.

---

# ✅ Summary

- **Immutable class** → object’s state cannot change once created.  
- **Key rules:**  
  - `final class` → prevents subclassing.  
  - `private final fields` → ensures fields are initialized once.  
  - **No setters** → prevents modification after construction.  
  - **Defensive copies** → avoid exposing mutable objects.  
- **Benefits:**  
  - Thread-safety  
  - Reusability  
  - Reliability & fewer bugs  
