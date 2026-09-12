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


