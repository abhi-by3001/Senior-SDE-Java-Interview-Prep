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



