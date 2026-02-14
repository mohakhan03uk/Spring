# 🌱 Spring Bean Scopes 
------------------------------------------------------------------------

# 1️. What is Bean Scope in Spring?

In Spring Framework, *bean scope* defines:

-   How many instances of a bean Spring creates
-   How long those instances live inside the ApplicationContext
-   How Spring manages those instances

Spring provides two core scopes:

-   Singleton (default)
-   Prototype

------------------------------------------------------------------------

# 2️. Singleton Scope (Default)

## Definition

Only **one instance per ApplicationContext**.

Every request for that bean returns the **same object**.

## Example

``` java
@Component
public class PaymentService {
}
```

## Characteristics

-   Created at context startup (eager by default)
-   Shared across entire application
-   Fully managed lifecycle

## Lazy Initialization

``` java
@Lazy
@Component
public class OrderService {
}
```

## Use Cases

-   Stateless services
-   Business logic
-   Database repositories
-   Utility services
-   Configuration classes

## Pros

-   Memory efficient
-   High performance
-   Fully managed lifecycle

## Cons

-   Thread safety concerns
-   Shared mutable state risks

------------------------------------------------------------------------

# 3️. Prototype Scope

## Definition

A **new instance is created every time** the bean is requested.

``` java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

## Characteristics

-   New object each request
-   Spring manages creation only
-   No automatic destruction

## Use Cases

-   Stateful processing objects
-   Per-request data holders
-   Temporary business objects

## Pros

-   No shared state
-   Safer for mutable objects

## Cons

-   Higher memory usage
-   Lifecycle not fully managed

------------------------------------------------------------------------

# 4️. Singleton vs Prototype -- Comparison Table

  Feature                 Singleton            Prototype
  ----------------------- -------------------- ------------------
  Default Scope           Yes                  No
  Instances               One per context      New each request
  Lifecycle Managed       Fully                Creation only
  Thread Safety Concern   High                 Low
  Memory Usage            Lower                Higher
  Best For                Stateless services   Stateful objects

------------------------------------------------------------------------

# 5️. Common Mistake: Prototype Inside Singleton

Injecting prototype into singleton:

``` java
@Service
public class OrderService {

    @Autowired
    private ReportGenerator reportGenerator;
}
```

The prototype is created only once during singleton creation. The prototype bean is created when Spring looking for it to inject somewhere, no new object creation while/just accessing it.

## Correct Solution

Use ObjectProvider:

``` java
@Autowired
private ObjectProvider<ReportGenerator> provider;

public void process() {
    ReportGenerator generator = provider.getObject();
}
```

------------------------------------------------------------------------

# 6️. Top 10 Problems When Using Wrong Scope

1.  Race conditions
2.  Data corruption
3.  Concurrency bugs
4.  Memory leaks
5.  Performance degradation
6.  Unexpected object reuse
7.  Startup delays
8.  Lifecycle mismanagement
9.  Hard-to-debug behavior
10. Incorrect dependency behavior

------------------------------------------------------------------------

# 7️. 15 Questions on Bean Scope

1.  What is bean scope in Spring?
2.  What is the default scope?
3.  Difference between singleton and prototype?
4.  Is Spring singleton same as GoF singleton?
5.  When should you use prototype scope?
6.  What happens if prototype is injected into singleton?
7.  How does Spring manage bean lifecycle?
8.  What is eager vs lazy initialization?
9.  Is singleton thread-safe?
10. How to make singleton thread-safe?
11. Does Spring manage destruction of prototype beans?
12. What is ObjectProvider?
13. What is @Lookup?
14. How does scope affect performance?
15. How do scopes behave in multi-threaded apps?

------------------------------------------------------------------------

# 8️. Tricky Scenario-Based Questions

### Scenario 1

You have a CounterService singleton with mutable state. What can go wrong? → Race conditions.

### Scenario 2

Prototype injected into singleton. Why same instance reused? → Injection happens once at singleton creation.

### Scenario 3

Heavy singleton bean slows startup. → Use @Lazy.

### Scenario 4

Prototype bean holding DB connection. → Risk of resource leak.

### Scenario 5

Stateless service defined as prototype. → Unnecessary memory overhead.

------------------------------------------------------------------------

# 9️. Deep Internal Working of Spring Bean Lifecycle

## For Singleton

1.  Load BeanDefinition
2.  Instantiate bean
3.  Populate dependencies
4.  Call BeanPostProcessor (before init)
5.  Call @PostConstruct
6.  Bean ready
7.  Context shutdown → @PreDestroy

## For Prototype

1.  Load BeanDefinition
2.  Instantiate bean
3.  Populate dependencies
4.  Call initialization callbacks
5.  Return to caller
6.  No destruction callback by Spring

------------------------------------------------------------------------

# 10 Quick Summary

Bean scope controls instance creation and lifecycle.\
Singleton creates one shared instance per context.\
Prototype creates a new instance per request.\
Singleton must be stateless or thread-safe.\
Prototype is useful for stateful objects but not fully
lifecycle-managed.

------------------------------------------------------------------------

# ✅ Design Advice

-   Default to singleton
-   Keep singleton stateless
-   Use prototype only when required
-   Avoid mixing scopes without proper provider mechanism

------------------------------------------------------------------------
