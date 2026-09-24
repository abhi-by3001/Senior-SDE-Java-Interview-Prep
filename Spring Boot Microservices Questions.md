Q1. What is IoC (Inversion of Control) and DI (Dependency Injection)?  
ANS - Normally a class creates its own dependencies (new UserRepository()). With IoC, that control is handed to the Spring container — it creates and manages objects (called beans) for you. DI is the mechanism: the container injects the needed dependency into a class instead of the class constructing it itself.

```java
@Service
class OrderService {
    private final PaymentClient paymentClient;

    // Constructor injection — preferred (immutable, testable, no null field window)
    @Autowired
    public OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }
}
```

Interview note: Prefer constructor injection over field injection (@Autowired directly on a field) — it makes dependencies explicit, allows final fields, and fails fast at startup if a bean is missing, plus it's easier to unit test without Spring.  
Field injection (what to avoid)
```java
@Service
public class OrderService {
    @Autowired
    private PaymentClient paymentClient;   // Spring injects this via reflection, after construction
}
```
Looks clean, but it hides problems.
Constructor injection (preferred)
"Makes dependencies explicit"
With constructor injection, anyone reading the class signature immediately sees everything it needs — the constructor parameter list is the dependency list. With field injection, you have to scroll through the whole class hunting for @Autowired fields to know what it depends on.

"Allows final fields"
A field can only be final if it's set in the constructor. Field injection sets the value after the object is constructed (via reflection), so the field can never be final — meaning nothing stops some other code from reassigning paymentClient later by accident. Constructor injection lets you make it final, so once set, it's locked for the object's whole lifetime.

"Fails fast at startup if a bean is missing"
```java
// Constructor injection: if PaymentClient bean doesn't exist,
// Spring can't even construct OrderService → app fails to start, error is loud and immediate.

// Field injection: OrderService constructs fine with an empty constructor,
// Spring injects null-ish/missing dependency, and you don't find out until
// some request actually calls paymentClient.someMethod() → NullPointerException at runtime, possibly in production.
```
Constructor injection surfaces a missing dependency the moment the app tries to boot. Field injection can let a broken wiring slip past startup entirely.
"Easier to unit test without Spring"
```java
// Constructor injection — trivial, no Spring container needed at all:
PaymentClient mockClient = mock(PaymentClient.class);
OrderService service = new OrderService(mockClient);

// Field injection — the field is private with no setter, so a plain unit test
// can't easily supply a mock. You either need reflection hacks (ReflectionTestUtils.setField)
// or you have to spin up a Spring test context just to get the wiring to work.
```

Q2. What are the common Spring stereotype annotations, and how do they differ?  
ANS -  
@Component -> Generic Spring-managed bean  
@Service -> Business/service-layer bean (semantic marker, same behavior as @Component)  
@Repository -> Data-access layer bean; also translates DB exceptions into Spring's unified DataAccessException  
@Controller	Web -> layer, returns view names  
@RestController -> @Controller + @ResponseBody — returns data (JSON) directly, not a view  
@Configuration -> Marks a class containing @Bean definitions  

Q3 - What are bulkheads?  
ANS - Simple analogy: ship bulkheads  
A ship has watertight compartments.  
If one compartment gets flooded, the water stays there. The rest of the ship stays dry and keeps floating.  
Without bulkheads, one leak sinks the whole ship.  
In software  
Your app has limited resources: threads, connections, memory.  
Without bulkhead:  
All requests share one big thread pool.  
A slow downstream service starts consuming all threads.  
Soon every thread is stuck waiting on that one slow service.  
Now your whole app is down — even requests that don’t need that service.  
With bulkhead:  
You give each downstream service its own small pool.  
If Service B becomes slow, only Service B’s pool fills up.  
Service A and Service C still have their own threads and keep working.  
Simple example  
Payment service → max 50 concurrent calls  
Recommendation service → max 10 concurrent calls  
Search service → max 20 concurrent calls  
If Recommendation becomes slow:  
Only 10 threads get stuck waiting on it.  
Payment and Search are unaffected.  
Your app stays alive.  
How it’s implemented  
Usually with a semaphore or a separate thread pool  


Q- Your spring boot api normally responds in 200ms, but today it is taking 8 seconds , how would you troubleshoot it?  
(log analysis, db query optimization, hikari cp connection pool, thread dumps, jvm (heap, gc, cpu), external service, latency, monitoring and observability, production debugging approach)  
Q- How can synchronization related performance bottlenecks be minimized?  
Q- What is ThreadLocal, and how can it lead to memory leaks?  
Q- What is a BlockingQueue and where is it used in production systems?  
Q- What is the purpose of @PostConstruct and when is it executed?  
Q - How does Dependency Injection work internally in Spring Boot?  
Q - Explain the JVM Memory Model, including Hep, Stack and Metaspace?  
Q - How do you implement global exception handling in Spring Boot?  
Q - What are atomic variables and in which scenarios are they commonly used?  
Q - Why did you choose a microservices architecture instead of a monolith architecture?  
Q - What are the key differences between kafka and RabbitMQ, and when would you choose each?  
Q - Explain your CI/CD pipeline. Walk through your Jenkins build process?  
Q - What are the key features of Istio and Service Mesh in kubernetes?  
Q - How do you Dockerize a Spring Boot application?  
Q - How do you manage distributed transactions across microservices?  
Q - What is the difference between clustered and non-clustered indexes?  
Q - What is a RESTful API and difference between REST and SOAP?  
Q - What is Spring and why do we use Spring?  
Q - Difference between Spring and Spring boot?  
Q - How does autowiring work in spring?  
Q - What is Spring security?  
Q - Difference between HTTP and HTTPs?  
Q - How to find server crash reasons?  
Q - How to find server memory?  
Q - How do you debug high CPU or memory issues in JVM?  
Q - How do you capture heap/thread dumps?  
Q - How do microservices communicate?  
Q - How do you implement JWT authentication (also in spring boot)?  
Q - OAuth2 vs JWT?  
Q - RestTemplate vs WebClient?  
Q - What is idempotency in distributed systems?  
Q - First-level vs Second-level cache in JPA?
Q - What is N+1 problem?
Q - How to optimize slow queries?  
Q - How do you handle concurrent updates?  
Q - How to handle 1M+ transactions daily?  
Q - Saga Pattern vs 2PC?  
Q - How to ensure data consistency across services?  
Q - How to implement distributed locking?  
Q - How do you deploy microservices using Docker?  
Q - What is Circuit Breaker (circuit breaker pattern with Resilience4J) ?  
ANS - A circuit breaker is a state machine that stops calling a failing downstream service, so your app fails fast instead of wasting threads waiting and causing a cascading failure.  
In software:  
Closed → calls pass through.  
Open → calls fail immediately, no downstream traffic.  
Half-open → allow a few test calls. If OK, close. If not, stay open.  

Q - How do you monitor logs?  
Q - How does Spring Boot decide which auto-configuration to apply?  
Q - Difference between get() and load() in Hibernate?  
Q - How do you handle optimistic vs pessimistic locking in JPA?  
Q - Explain dirty checking in Hibernate?  
Q - Entity lifecycle states (Transient, Persistent, Detached, Removed)?  
Q - You have a product catalog service where multiple users can update stock quantity at the same time. How would you use JPA locking to prevent inconsistent data?  
Q - Why @Transactional can silently fail ?  
Q - ExecutorService, Callable and Future?  
Q - How do you handle exceptions in spring boot?  
Q - How do you handle a hot partition when one seller gets 10x traffic overnight?  
Q - How would you design an inventory system that stays consistent during a flash sale?  
Q - SQL vs NoSQL and how do you decide for a given access pattern?  
Q - What is var? Can it be used with generics?  
Q - What is an effectively final variable?  
Q - What is optional ? When should you use it ?  
Q - Checked vs Unchecked exceptions?  
Q - Can HashMap keys be Mutable? Why?  
Q - What is @ControllerAdvice?  
Q - Can interfaces have private methods ?  
Q - What types of methods can interfaces contain ?  
Q - Same default methods in two interfaces, how to resolve ?  
Q - How do you implement security across microservices?  
Q - Lazy vs Eager loading?  
Q - @PathVariable vs @RequestParam?  
Q - What are Spring profiles and how are they different from Maven profiles ?  
Q - Explain the Bean lifecycle?  
Q - @Primary vs @Qualifier?  
Q - Your API response jumps from 200 ms to 5 seconds , where do you look first?  
Q - The application works fine with 100 users but fails at 10,000 users. What breaks ?  
Q - CPU usage is normal, but API latency is high. What could be happening?  
Q - How do you identify whether the bottleneck is in your code, database, network or downstream service?  
Q - A query takes milliseconds in development but 10 seconds in production. Why?  
Q - Memory usage keeps increasing for several days and eventually the application crashes. How do you find the leak?  
Q - A downstream service becomes extremely slow. How do you prevent it from taking down your entire application?  
Q - When would you use Circuit breaker, retry timeout and Bulkhead?  
Q - Multiple application instances are running. How would you handle cache invalidation consistently?  
Q - Redis goes down, should you application also go down ? How would you design the fallback?  
Q - How would you design communication between microservices using synchronous and asynchronous approaches?  
Q - What is difference between API Gateway, Service Discovery and Load Balancing?  
Q - How do you handle distributed transactions? Explain Saga and Outbox pattern?  
Q - How do you ensure idempotency in a microservices based payment or order API?  

1. Your `HashMap` is being accessed by multiple threads. What problems can occur, and how would you fix them?  
2. Your application has a race condition causing duplicate payments. How would you identify and solve it?  
3. Your application creates thousands of threads and CPU usage spikes. What would you do?  
4. A REST API is taking 30 seconds because it calls three downstream services. How would you optimize it?  
5. Multiple threads are updating the same record. How would you ensure consistency?  
6. Your Spring Boot application startup time increased from 15 seconds to 2 minutes. How would you investigate?  
7. A circular dependency error appears after deployment. How would you resolve it?  
8. One API works locally but fails in production with `LazyInitializationException`. What could be the reason?  
9. A transaction partially updates data even though you expected a rollback. Why might this happen?  
10. Your application suddenly starts throwing `OutOfMemoryError`. How would you debug it?  
11. Service A calls Service B, which calls Service C. Service C is down. How would you prevent the failure from cascading?  
12. A customer reports that the same payment was processed twice. How would you prevent duplicate processing?  
13. One microservice becomes slow and starts affecting the entire system. What patterns would you implement?  
14. How would you trace a request across 15 microservices?  
15. One microservice must communicate with another. Would you choose REST, Kafka, or gRPC? Why?  
16. A Kafka consumer processes the same message twice. How would you handle it?  
17. One Kafka partition has much higher traffic than others. How would you fix it?  
18. Consumer lag keeps increasing. How would you investigate?  
19. A message fails repeatedly during processing. What should happen next?  
20. How would you guarantee message ordering for a customer?  
21. A query that used to take 50 ms now takes 10 seconds. How would you troubleshoot it?  
22. Your database CPU reaches 100% during peak hours. What steps would you take?  
23. Two transactions update the same row simultaneously. How would you handle concurrency?  
24. A table has grown to hundreds of millions of records. How would you improve performance?  
25. Would you choose optimistic locking or pessimistic locking for an inventory system? Why?  
26. Your API receives 100,000 requests per minute. How would you scale it?  
27. Users are abusing your API. How would you implement rate limiting?  
28. Your Redis cache crashes unexpectedly. How should your application behave?  
29. An EC2 instance needs to access S3 securely. How would you configure it without storing credentials?  
30. A deployment causes increased latency. How would you identify the root cause and roll back safely?  

Streams API questions - https://medium.com/@asishpanda444/stream-api-coding-qna-8df8682b7e2a


# Revision Notes: `wait()` and `notify()` in Java

## 1. Core Idea
- `wait()` → “I’ll release the lock and sleep. Wake me when things change.”
- `notify()` → “One sleeping thread can wake up now.”
- `notifyAll()` → “All sleeping threads can wake up now.”

They are used for **inter-thread communication** on a shared object.

---

## 2. Key Methods (from `Object` class)

| Method | Meaning |
|---|---|
| `wait()` | Release lock + wait until notified |
| `wait(timeout)` | Wait with max timeout |
| `notify()` | Wake **one** waiting thread |
| `notifyAll()` | Wake **all** waiting threads |

---

## 3. Golden Rules

1. **Must be inside `synchronized`**  
   Otherwise → `IllegalMonitorStateException`.

2. **`wait()` releases the lock**  
   Lets other threads enter and change state.

3. **`notify()` does NOT release the lock immediately**  
   The woken thread waits until the notifying thread exits the `synchronized` block.

4. **Always use `while`, never `if`**  
   Re-check condition after waking. Handles spurious wakeups and race conditions.

5. **Call on the same object whose lock you hold**  
   Inside a `synchronized` method, it’s `this`.

---

## 4. Standard Pattern

```java
synchronized void someMethod() {
    while (conditionIsNotReady) {
        wait();
    }
    // do work
    notifyAll(); // or notify()
}
```

---

## 5. Concise Code Example: Producer–Consumer

```java
class Box {
    private int item = 0; // 0 = empty, 1 = full

    synchronized void put() throws InterruptedException {
        while (item != 0) {          // wait if full
            wait();
        }
        item = 1;
        System.out.println("Produced");
        notifyAll();                 // wake consumer
    }

    synchronized void get() throws InterruptedException {
        while (item == 0) {          // wait if empty
            wait();
        }
        item = 0;
        System.out.println("Consumed");
        notifyAll();                 // wake producer
    }
}

public class Demo {
    public static void main(String[] args) {
        Box box = new Box();

        Thread producer = new Thread(() -> {
            try { box.put(); } catch (InterruptedException e) {}
        });

        Thread consumer = new Thread(() -> {
            try { box.get(); } catch (InterruptedException e) {}
        });

        consumer.start();
        producer.start();
    }
}
```

**Output:**
```
Produced
Consumed
```
(Order may vary.)

---

## 6. What Happens Step-by-Step

| Step | Consumer Thread | Producer Thread | Lock Owner |
|---|---|---|---|
| 1 | Enters `get()`, gets lock | — | Consumer |
| 2 | `item == 0` → calls `wait()` | — | **Consumer releases lock** |
| 3 | Sleeping | Enters `put()`, gets lock | Producer |
| 4 | Sleeping | Sets `item = 1`, calls `notifyAll()` | Producer |
| 5 | Still sleeping (can’t run yet) | Exits `put()`, releases lock | Nobody |
| 6 | Wakes, re-acquires lock, re-checks `while` | — | Consumer |
| 7 | `item == 1` → exits loop, consumes | — | Consumer |

---

## 7. Common Mistakes

| Mistake | Fix |
|---|---|
| `wait()` outside `synchronized` | Put inside `synchronized` |
| Using `if` instead of `while` | Always use `while` |
| Calling `notify()` before `wait()` | Signal is lost → use proper condition loop |
| Thinking `notify()` releases lock | It doesn’t; lock released at end of synchronized block |
| Using different objects for `wait`/`notify` | Use the same shared object |

---

## 8. One-Line Summary

> **`wait()` = release lock + sleep.  
> `notify()` = wake one sleeper, but keep lock until done.  
> Always check condition in a `while` loop inside `synchronized`.**


# Revision Notes: ReentrantLock in Java (with Tricky Points)

## 1. What Is a Lock?
- Only **one thread at a time** can enter a critical section.
- Others must wait outside until the lock is released.

---

## 2. What Does "Reentrant" Mean?
- **Reentrant = a thread that already holds the lock can acquire it again** without blocking.
- The lock keeps an internal **hold count**.
- Thread must `unlock()` **the same number of times** it `lock()`ed.

```java
lock.lock();   // count = 1
lock.lock();   // count = 2
lock.unlock(); // count = 1
lock.unlock(); // count = 0 → fully released
```

---

## 3. `synchronized` Is Also Reentrant (Automatic)

```java
synchronized void outer() {
    inner();   // same thread, already holds lock → OK
}
synchronized void inner() {
    // runs fine
}
```

- JVM tracks hold count **internally** (hidden from you).
- **No `getHoldCount()` needed** — it's automatic.
- Lock released **automatically** when the synchronized block/method exits.

---

## 4. `ReentrantLock` vs `synchronized` — Quick Compare

| Feature | `synchronized` | `ReentrantLock` |
|---|---|---|
| Reentrant | ✅ | ✅ |
| Hold count visible? | ❌ (hidden) | ✅ `getHoldCount()` |
| Auto unlock? | ✅ | ❌ (must use `finally`) |
| `tryLock()` | ❌ | ✅ |
| Timed lock | ❌ | ✅ |
| Interruptible wait | ❌ | ✅ `lockInterruptibly()` |
| Fairness option | ❌ | ✅ `new ReentrantLock(true)` |
| Multiple conditions | ❌ (only 1 wait room) | ✅ multiple `Condition`s |
| Simpler code | ✅ | ❌ |
| Speed (modern JVM) | Similar | Similar |

**Rule of thumb:** Use `synchronized` unless you need the extra features.

---

## 5. Basic Usage — Golden Rule

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();
try {
    // critical section
} finally {
    lock.unlock();   // ALWAYS in finally!
}
```

**Why `finally`?** If an exception is thrown and you forget to unlock → everyone waits forever.

---

## 6. Reentrancy Demo with `getHoldCount()`

```java
ReentrantLock lock = new ReentrantLock();

void outer() {
    lock.lock();
    try {
        System.out.println(lock.getHoldCount()); // 1
        inner();
    } finally {
        lock.unlock();
    }
}

void inner() {
    lock.lock();
    try {
        System.out.println(lock.getHoldCount()); // 2
    } finally {
        lock.unlock();   // count → 1
    }
}
```

---

## 7. ⚠️ TRICKY POINT: Does Calling a Non-Synchronized Method Release the Lock?

## **NO. Never.**

> The lock is released **ONLY** when:
> - The `synchronized` block/method ends (automatic), OR
> - You call `unlock()` on `ReentrantLock` (manual), OR
> - You call `wait()` / `condition.await()` (temporarily).

```java
synchronized void outer() {
    System.out.println("Holding lock");
    inner();   // NOT synchronized — but lock is STILL held!
    System.out.println("Still holding lock");
}

void inner() {
    System.out.println("Running, but outer still holds the lock");
}
```

**What happens:**
1. Thread A enters `outer()` → gets lock.
2. Thread A calls `inner()` → **lock NOT released**.
3. Thread B tries `outer()` → **BLOCKED** until A finishes `outer()` completely.

Same with `ReentrantLock`:

```java
void outer() {
    lock.lock();
    try {
        inner();   // lock still held
    } finally {
        lock.unlock();   // released ONLY here
    }
}
```

### When is lock released?

| Situation | Lock released? |
|---|---|
| Call non-synchronized method while holding lock | ❌ No |
| Exit synchronized block/method | ✅ Yes (auto) |
| Call `unlock()` | ✅ Yes (manual) |
| Call `wait()` / `await()` | ✅ Yes (temporarily, re-acquired after waking) |

**Lesson:** Keep synchronized blocks **short**. Don't call slow methods while holding the lock.

---

## 8. `tryLock()` — Don't Wait, Just Try

```java
if (lock.tryLock()) {
    try { /* work */ } finally { lock.unlock(); }
} else {
    System.out.println("Busy, doing something else");
}
```

**Timed version:**

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try { /* work */ } finally { lock.unlock(); }
} else {
    System.out.println("Gave up after 2 seconds");
}
```

---

## 9. ⚠️ TRICKY POINT: `lockInterruptibly()` — "I Can Be Woken Up"

### `lock()` vs `lockInterruptibly()`

| | `lock()` | `lockInterruptibly()` |
|---|---|---|
| Can be interrupted while waiting? | ❌ No | ✅ Yes |
| Throws on interrupt? | No | `InterruptedException` |
| Can give up? | No | Yes |

### Story:
> **`lock()`** → You wait outside a door forever. Even if your boss calls, you ignore everything.
> **`lockInterruptibly()`** → You wait, but your phone is on. If boss calls, you pick up and leave.

### Code:

```java
try {
    lock.lockInterruptibly();
    try {
        // work
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    System.out.println("Interrupted while waiting — giving up");
}
```

**When useful?**
- Cancellation (user clicks "Cancel")
- Timeouts
- Deadlock avoidance
- Responsive UI

---

## 10. ⚠️ TRICKY POINT: Fair Lock and Starvation

### What is starvation?
> A thread waits **forever** (or very long) because other threads keep getting the lock first.

### Fair vs Unfair Lock

```java
ReentrantLock unfairLock = new ReentrantLock();        // default
ReentrantLock fairLock   = new ReentrantLock(true);    // fair = FIFO
```

| | Unfair (default) | Fair (`true`) |
|---|---|---|
| Order | Scheduler decides | FIFO (queue) |
| Speed | Faster | Slower |
| Starvation possible? | Yes | No |
| Throughput | Higher | Lower |

**When to use fair?** When starvation is unacceptable (real-time systems, fair allocation). Otherwise, unfair is fine — it's faster.

---

## 11. ⚠️ TRICKY POINT: `Condition` — Two Waiting Rooms

### The problem with `synchronized` + `notifyAll()`

Only **ONE waiting room** for all threads.

- 3 producers waiting (buffer full)
- 2 consumers waiting (buffer empty)
- Consumer takes item → `notifyAll()` wakes **all 5** — including other consumers with nothing to do!
- This is the **"thundering herd" problem** — wasted wakeups.

### The fix: `ReentrantLock` + 2 Conditions

```java
ReentrantLock lock = new ReentrantLock();
Condition notFull  = lock.newCondition();  // waiting room for PRODUCERS
Condition notEmpty = lock.newCondition();  // waiting room for CONSUMERS
```

| Waiting Room | Who waits? | When? |
|---|---|---|
| `notFull` | Producers | Buffer is full |
| `notEmpty` | Consumers | Buffer is empty |

| Action | Signal | Who wakes? |
|---|---|---|
| Producer adds | `notEmpty.signal()` | **Only consumers** |
| Consumer takes | `notFull.signal()` | **Only producers** |

### Full Code:

```java
class Buffer {
    private int item = 0;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull  = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    void put() throws InterruptedException {
        lock.lock();
        try {
            while (item == 1) notFull.await();  // producer sleeps
            item = 1;
            System.out.println("Produced");
            notEmpty.signal();                   // wake ONLY consumer
        } finally {
            lock.unlock();
        }
    }

    void get() throws InterruptedException {
        lock.lock();
        try {
            while (item == 0) notEmpty.await(); // consumer sleeps
            item = 0;
            System.out.println("Consumed");
            notFull.signal();                    // wake ONLY producer
        } finally {
            lock.unlock();
        }
    }
}
```

### `synchronized` vs `Condition` mapping:

| `synchronized` | `ReentrantLock` |
|---|---|
| `wait()` | `condition.await()` |
| `notify()` | `condition.signal()` |
| `notifyAll()` | `condition.signalAll()` |

**Big win:** Precise signaling → fewer wasted wakeups → better performance.

---

## 12. Common Mistakes

| Mistake | Fix |
|---|---|
| Forgetting `unlock()` | Always in `finally` |
| Unlocking more times than locking | Match each `lock()` with one `unlock()` |
| Unlocking from a different thread | Only the lock owner can unlock |
| Returning early without unlock | Use `try/finally` |
| `await()` without holding lock | Must hold lock first |
| Thinking non-synchronized call releases lock | It does NOT |
| Using `if` instead of `while` with `await()` | Always use `while` |

---

## 13. When to Use What?

| Scenario | Use |
|---|---|
| Simple mutual exclusion | `synchronized` |
| Need `tryLock` / timeout | `ReentrantLock` |
| Need interruptible wait | `ReentrantLock` |
| Need fairness | `ReentrantLock(true)` |
| Need multiple conditions | `ReentrantLock` + `Condition` |
| Just want simplicity | `synchronized` |

---

## 14. One-Line Summaries

> **ReentrantLock** = manual lock you can re-enter, with extras: `tryLock`, timeouts, fairness, multiple conditions. Always `unlock()` in `finally`.

> **Reentrant** = a thread holding the lock can lock it again. Count must go to 0 to release.

> **Non-synchronized call** does NOT release the lock. Lock held until block ends or `unlock()`.

> **`lockInterruptibly()`** = waiting thread can be interrupted and give up.

> **Starvation** = thread waits forever because others keep jumping ahead. Fix = fair lock.

> **Condition** = separate waiting rooms for producers and consumers → wake only the right group.



# SQL Cheat Sheet: JOINs & GROUP BY

## 1. SQL JOINs Overview
SQL JOINs are used to combine rows from two or more tables based on a related column between them.

### Types of JOINs
*   **`INNER JOIN`**: Returns rows that have matching values in **both** tables.
*   **`LEFT (OUTER) JOIN`**: Returns **all** rows from the left table, plus matching rows from the right table. If no match is found, `NULL` is returned for the right table.
*   **`RIGHT (OUTER) JOIN`**: Returns **all** rows from the right table, plus matching rows from the left table. If no match is found, `NULL` is returned for the left table.
*   **`FULL (OUTER) JOIN`**: Returns **all** rows when there is a match in either the left or right table. Missing values are filled with `NULL`.
*   **`CROSS JOIN`**: Returns the Cartesian product of the two tables (every possible combination of rows).
*   **`SELF JOIN`**: A standard join (inner or left) where a table is joined to itself.

### Standard JOIN Syntax
```sql
SELECT 
    table1.column1, 
    table2.column2
FROM table1
<JOIN_TYPE> JOIN table2 
    ON table1.common_column = table2.common_column;
```

---

## 2. The GROUP BY Clause
The `GROUP BY` clause groups rows that have the same values into summary rows, typically used alongside aggregate functions like `COUNT()`, `SUM()`, `AVG()`, `MAX()`, or `MIN()`.

### Standard GROUP BY Syntax
```sql
SELECT 
    column_to_group_by, 
    AGGREGATE_FUNCTION(column_to_summarize)
FROM table_name
WHERE condition -- Filters raw rows BEFORE grouping
GROUP BY column_to_group_by
HAVING aggregate_condition; -- Filters groups AFTER grouping
```

### Combined Example (JOIN + GROUP BY)
```sql
SELECT 
    customers.customer_id,
    customers.customer_name,
    SUM(orders.order_amount) AS total_spent
FROM customers
INNER JOIN orders 
    ON customers.customer_id = orders.customer_id
GROUP BY 
    customers.customer_id, 
    customers.customer_name;
```

---

## 3. Five Critical Rules for GROUP BY & HAVING

### Rule 1: The Non-Aggregated Column Rule (The Golden Rule)
Every column listed in your `SELECT` clause that is **not** wrapped inside an aggregate function **must** be explicitly listed in the `GROUP BY` clause.
*   ❌ **Wrong:** `SELECT department, job_title, AVG(salary) FROM employees GROUP BY department;`
*   **Correct:** `SELECT department, job_title, AVG(salary) FROM employees GROUP BY department, job_title;`

### Rule 2: The Logical Execution Order
SQL processes queries in a specific order, which dictates what data is available at each stage:
1. `FROM` & `JOIN` *(Gathers data sources)*
2. `WHERE` *(Filters raw rows)*
3. `GROUP BY` *(Splits data into buckets)*
4. `HAVING` *(Filters summarized buckets)*
5. `SELECT` *(Computes and displays output columns)*
6. `ORDER BY` *(Sorts the final output)*

### Rule 3: `WHERE` vs. `HAVING` Separation
*   Use **`WHERE`** to filter individual rows before grouping. It **cannot** contain aggregate functions (e.g., `WHERE SUM(sales) > 10` is an error).
*   Use **`HAVING`** to filter the aggregated results after grouping (e.g., `HAVING SUM(sales) > 10` is correct).

### Rule 4: Alias Restrictions
Because `SELECT` runs *after* `GROUP BY` and `HAVING` (see Rule 2), standard SQL does not allow you to use column aliases created in the `SELECT` clause inside your `GROUP BY` or `HAVING` clauses.
*   ❌ **Wrong:** `SELECT region AS r, COUNT(*) FROM sales GROUP BY r HAVING r = 'East';`
*   **Correct:** `SELECT region AS r, COUNT(*) FROM sales GROUP BY region HAVING region = 'East';`

### Rule 5: `NULL` Grouping
If the column you are grouping by contains `NULL` values, the database engine treats them as a single value and will merge all `NULL` records into **one single row** in the final output.



# Java Sorting Cheatsheet: Comparable, Comparator, Lists, and Arrays

## 1. Comparable vs Comparator

| Feature | `Comparable` | `Comparator` |
| :--- | :--- | :--- |
| **Package** | `java.lang` | `java.util` |
| **Method** | `int compareTo(T o)` | `int compare(T o1, T o2)` |
| **Logic Location** | **Internal:** Inside the target class. | **External:** In a separate class or lambda. |
| **Modifies Class?** | **Yes:** Modifies original source code. | **No:** Leaves original class untouched. |
| **Strategies** | Only **one** default natural sorting order. | **Multiple** custom sorting strategies. |
| **Syntax** | `Collections.sort(list)` | `Collections.sort(list, customComparator)` |

### Comparable Implementation Example
```java
import java.util.*;

class Product implements Comparable<Product> {
    private String name;
    private double price;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    // Natural sorting by price (ascending)
    @Override
    public int compareTo(Product other) {
        return Double.compare(this.price, other.price);
    }

    @Override
    public String toString() { return name + ": $" + price; }
}

public class ComparableMain {
    public static void main(String[] args) {
        List<Product> list = new ArrayList<>(Arrays.asList(
            new Product("Laptop", 1200),
            new Product("Phone", 800)
        ));
        
        Collections.sort(list); 
        System.out.println(list); // [Phone: $800.0, Laptop: $1200.0]
    }
}
```

### Comparator Implementation Example
```java
import java.util.*;

class Product {
    private String name;
    private double price;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName() { return name; }
    public double getPrice() { return price; }

    @Override
    public String toString() { return name + ": $" + price; }
}

public class ComparatorMain {
    public static void main(String[] args) {
        List<Product> list = new ArrayList<>(Arrays.asList(
            new Product("Laptop", 1200),
            new Product("Phone", 800)
        ));

        // Strategy 1: Sort externally by Name using Lambda expression
        list.sort((p1, p2) -> p1.getName().compareTo(p2.getName()));
        
        // Strategy 2: Sort externally by Price using Comparator instance methods
        list.sort(Comparator.comparingDouble(Product::getPrice).reversed()); 
    }
}
```

---

## 2. Lists (`List<T>`)

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ListSort {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));

        // 🟢 INCREASING / ASCENDING ORDER
        Collections.sort(list); 
        // Alternative syntax: list.sort(Comparator.naturalOrder());

        // 🔴 DECREASING / DESCENDING ORDER
        Collections.sort(list, Collections.reverseOrder());
        // Alternative syntax: list.sort(Comparator.reverseOrder());
    }
}
```

---

## 3. Object Arrays (`Integer[]`, `String[]`)

```java
import java.util.Arrays;
import java.util.Collections;

public class ObjectArraySort {
    public static void main(String[] args) {
        Integer[] array = {5, 2, 8, 1, 9};

        // 🟢 INCREASING / ASCENDING ORDER
        Arrays.sort(array);

        // 🔴 DECREASING / DESCENDING ORDER
        Arrays.sort(array, Collections.reverseOrder());
    }
}
```

---


