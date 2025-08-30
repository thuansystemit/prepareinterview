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
