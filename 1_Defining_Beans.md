
# Spring – Chapter 1

The first thing you need to learn in Spring is adding object instances (which we call beans) to the Spring context.\
You can imagine the Spring context as a bucket in which you add the instances you expect Spring to be able to manage.\
Spring can see only the instances you add to its context.

In practice, this “bucket” is the IoC container, and everything Spring can inject, configure, or lifecycle-manage must live inside it.

You can add beans to the Spring context in three ways: using the @Bean annotation, using stereotype annotations (like **@Component**), and doing it programmatically.

A supporting mechanism is **@ComponentScan**, which tells Spring where to look for classes annotated with stereotype annotations. Without component scanning, stereotype annotations alone are not enough.

Using the **@Bean** annotation to add instances to the Spring context enables you to add any kind of object instance as a bean and even multiple instances of the same kind to the Spring context. From this point of view, this approach is more flexible  than using stereotype annotations. Still, it requires you to write more code because you need to write a separate method in the configuration class for each independent instance added to the context.

This approach is especially useful for:
- Third-party classes you cannot annotate
- Creating multiple beans of the same type with different configurations
- Applying custom initialization logic

Using **stereotype** annotations, you can create beans for only the application classes  with a specific annotation (e.g., @Component). This configuration approach  requires writing less code, which makes your configuration more comfortable to  read. 
You’ll prefer this approach over the @Bean annotation for classes that you  define and can annotate.

Common stereotype annotations include:
- @Component
- @Service
- @Repository
- @Controller

These annotations work only when **@ComponentScan** is enabled, either explicitly or implicitly (for example, via @SpringBootApplication).

Using the **registerBean()** method enables you to implement custom logic for adding beans to the Spring context. Remember, you can use this approach only with Spring 5 and later.

This programmatic approach is useful when:
- Bean creation depends on runtime conditions
- You need full control over bean registration
- You are building frameworks or advanced infrastructure code

---

## Example Code

### 1. Using @Bean

```java
@Configuration
public class AppConfig {

    @Bean
    public String appName() {
        return "MySpringApp";
    }

    @Bean
    public MyClass myClass(){
         return new MyClass();
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
    var context = new AnnotationConfigApplicationContext(AppCnfig.class);

    String s = context.getBean(String.class);
    System.out.println(s);

    MyClass myClass = context.getBean(MyClass.class);
    doXYZ(myXlass);
  }
}
```

Explanation:
- @Configuration marks this class as a source of bean definitions
- @Bean tells Spring to add the returned object to the context
- The method name becomes the bean name by default
- You don’t need to do any explicit casting. Spring looks for a bean of the type you requested in its context. If such a bean doesn’t exist, Spring will throw an exception.
---

### 2. Using Stereotype Annotations + @ComponentScan

```java
@Component
public class UserService {
    public void process() {
        System.out.println("Processing user");
    }
}
```

```java
@Configuration
@ComponentScan(basePackages = "com.example.app")
public class AppConfig {
}
```

Explanation:
- @Component marks the class as a Spring-managed bean
- @ComponentScan tells Spring where to search for annotated classes
- Without @ComponentScan, UserService would not be added to the context

---

### 3. Programmatic Bean Registration

```java
@Configuration
public class ProgrammaticConfig {
    @Bean
    public ApplicationContextInitializer<GenericApplicationContext> initializer() {
        return context -> context.registerBean(
            Integer.class,
            () -> 42
        );
    }
}
```

Explanation:
- registerBean() dynamically adds a bean at runtime
- Useful for conditional or computed bean definitions
- Available only in Spring 5+
---

### 4. Multiple Bean of Same Type
- A wrong way : we are creating 2 Beans of MyClass then accessing it.
  
  ```java
  @Configuration
  public class AppConfig {
    @Bean
    public MyClass myClass_1(){
         return new MyClass();
    }
    @Bean
    public MyClass myClass_2(){
         return new MyClass();
    }
    @Bean
    public String appName() {
        return "MySpringApp";
    }
  }
  ```

    ```java
      public class Main {
          public static void main(String[] args) {
          var context = new AnnotationConfigApplicationContext(AppCnfig.class);
      
          String s = context.getBean(String.class);
          System.out.println(s);
      
          MyClass myClass = context.getBean(MyClass.class);
          doXYZ(myXlass);
        }
      }
    ```

  - You will get an exception on this line because Spring cannot guess which of the 2 MyClass instances you refer to.
  
  ```java
      Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type
      'main.MyClass' available: expected single matching bean but found 2:
      myClass_1, myClass_2
      at ...
  ```
- Right way :
   - using Beans name
     ```java
      MyClass myClass_1 = context.getBean("myClass_1", MyClass.class);
      MyClass myClass_2 = context.getBean("myClass_xyz", MyClass.class);
     ```
     ```java
       @Configuration
       public class AppConfig {
         @Bean
         public MyClass myClass_1(){
              return new MyClass();
         }
         @Bean(name = "myClass_xyz")
         public MyClass myClass_2(){
             return new MyClass();
         }
       }
     ```
- @Bean : “Take the object returned by this method and put it into the Spring context as a bean.”
- @Bean(name = "xyz")  : name is an explicit attribute
- @Bean(value = "xyz") : value is an alias for name  : Slightly less explicit : Used sometimes for consistency with other annotations
- @Bean("xyz") : This uses Java’s single-attribute shortcut  : value is the default attribute :  Most concise form
- Multiple bean names (aliases) : @Bean(name = {"xyz", "primaryxyz"})  : or : @Bean({"xyz", "primaryxyz"})
```java
     public @interface Bean {
        String[] name() default {};
        String[] value() default {};
    }
     // value is the default attribute
    // Java lets you omit value = when only one attribute is set
    // Spring treats name and value the same internally
```
---
### Primary Bean
- A primary bean is the one Spring will choose if it has multiple options and you don’t specify a name; the primary bean is simply Spring’s default choice.
- In other words: @Primary resolves ambiguity when Spring performs type-based injection and finds more than one matching bean.
Without `@Primary` (or `@Qualifier`), Spring throws a `NoUniqueBeanDefinitionException`.

---

#### Example: Multiple beans with one primary

```java
@Configuration
public class AppConfig {

    @Bean
    @Primary
    public PaymentService creditCardPaymentService() {
        return new CreditCardPaymentService();
    }

    @Bean
    public PaymentService paypalPaymentService() {
        return new PaypalPaymentService();
    }
}
```

#### Explanation

- Two beans of type `PaymentService` exist in the Spring context
- One of them is marked with `@Primary`
- Spring uses the primary bean as the default choice

---

#### Injection without specifying a bean name

```java
@Autowired
private PaymentService paymentService;
```

Spring injects `creditCardPaymentService` because it is marked as primary.

---

#### Injection with @Qualifier overrides @Primary

```java
@Autowired
@Qualifier("paypalPaymentService")
private PaymentService paymentService;
```

Spring injects `paypalPaymentService`, even though it is not primary.

**Rule:** `@Qualifier` always wins over `@Primary`.

---

#### Using @Primary with stereotype annotations

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {
}
```

```java
@Service
public class PaypalPaymentService implements PaymentService {
}
```

This behaves the same as using `@Primary` with `@Bean`, but requires less configuration code.

---

#### Key Takeaways

- Use `@Primary` when multiple beans of the same type exist
- The primary bean becomes Spring’s default choice
- `@Primary` works with `@Bean` and stereotype annotations
- `@Qualifier` overrides `@Primary`
- Without either, Spring fails with a bean ambiguity error
