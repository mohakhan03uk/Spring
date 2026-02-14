# 🌱 Spring ObjectProvider
------------------------------------------------------------------------

# 1️. What is ObjectProvider?

ObjectProvider`<T>`{=html} is a lazy, on-demand dependency retrieval mechanism in Spring.

It allows: - Runtime bean resolution - Safe optional dependency injection - Proper prototype usage inside singleton - Streaming multiple beans of same type

It extends ObjectFactory`<T>`{=html} and is tightly integrated with BeanFactory.

------------------------------------------------------------------------

# 2️. Why ObjectProvider is Needed

Problem:

Injecting prototype into singleton:

``` java
@Service
public class OrderService {

    @Autowired
    private ReportGenerator reportGenerator; // prototype
}
```

Result: Prototype created only once during singleton creation.

Solution:

``` java
@Autowired
private ObjectProvider<ReportGenerator> provider;

public void process() {
    ReportGenerator generator = provider.getObject();
}
```

Each call to getObject() returns a new instance.

------------------------------------------------------------------------

# 3️. Deep Internal Flow: How getObject() Works

Step-by-step internal flow:

1.  Spring injects ObjectProvider instead of actual bean.
2.  ObjectProvider holds reference to BeanFactory.
3.  When getObject() is called: → It delegates to BeanFactory.getBean()
4.  BeanFactory checks:
    -   Scope
    -   BeanDefinition
    -   Lifecycle rules
5.  Bean is created (if needed).
6.  Dependencies injected.
7.  Initialization callbacks executed.
8.  Bean returned to caller.

For prototype: Each getObject() → new BeanFactory.getBean() call → new
instance.

For singleton: BeanFactory returns existing cached instance.

------------------------------------------------------------------------

# 4️. ObjectProvider Important Methods

-   getObject()
-   getIfAvailable()
-   getIfUnique()
-   stream()

Example:

``` java
provider.stream().forEach(bean -> bean.execute());
```

------------------------------------------------------------------------

# 5️. ObjectProvider vs Provider (JSR-330)

  Feature                    ObjectProvider   Provider (javax.inject)
  -------------------------- ---------------- -------------------------
  Part of                    Spring           JSR-330 standard
  Lazy resolution            Yes              Yes
  Optional retrieval         Yes              No
  getIfAvailable             Yes              No
  Stream support             Yes              No
  Spring-specific features   Yes              No
  Portability                Lower            Higher

Provider example:

``` java
@Inject
private Provider<ReportGenerator> provider;

ReportGenerator gen = provider.get();
```

Provider is framework-neutral but limited in features.

ObjectProvider is more powerful inside Spring ecosystem.

------------------------------------------------------------------------

# 6️. Advanced Scenario-Based Interview Questions

Scenario 1: You injected ObjectProvider for singleton bean. What happens? → getObject() returns same cached singleton instance.

Scenario 2: Prototype bean heavy object. getObject() called 1000 times. → 1000 new instances created.

Scenario 3: Multiple implementations exist. getIfUnique() called. → Returns bean only if exactly one candidate exists.

Scenario 4: Optional dependency missing. getIfAvailable() used. → Returns null without throwing exception.

Scenario 5: You frequently use ObjectProvider in many services. → Indicates possible design smell or overuse of dynamic lookup.

Scenario 6: Using stream() when multiple strategies exist. → Enables Strategy pattern implementation dynamically.

Scenario 7: ObjectProvider inside @PostConstruct method. → Bean resolved during initialization phase.

Scenario 8: Using ObjectProvider for performance optimization. → Delays heavy bean creation until needed.

------------------------------------------------------------------------

# 7️. Lifecycle Diagram Including ObjectProvider

Injection Phase:

Singleton Bean Created ↓ Spring injects ObjectProvider (NOT actual bean) ↓ Application running

Runtime Phase:

Call provider.getObject() ↓ Delegates to BeanFactory.getBean() ↓ Scope check: - Singleton → return cached instance - Prototype → create new instance ↓ Initialization callbacks executed ↓ Bean returned to caller

Important: Prototype destruction still NOT automatic.

------------------------------------------------------------------------

# 8️. Senior-Level Design Advice

Use ObjectProvider when: - Mixing singleton and prototype - Need lazy heavy dependency - Need optional injection - Implementing dynamic strategy resolution

Avoid overuse: Frequent programmatic lookups may indicate poor architecture.

------------------------------------------------------------------------

# 9️. Summary

ObjectProvider is a Spring-specific, lazy dependency provider. It delegates to BeanFactory at runtime via getObject(). It solvesprototype-in-singleton problems, supports optional retrieval, and allows dynamic bean resolution with stream().

------------------------------------------------------------------------
