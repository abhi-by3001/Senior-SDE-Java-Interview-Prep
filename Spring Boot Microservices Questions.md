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

