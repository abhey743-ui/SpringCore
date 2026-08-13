# Spring IoC Container | Beans, `@Component`, `@Autowired` & `@Bean`

## Coder Army — Spring Boot Full Course #5

**Video:** [Spring IOC Container | Beans, @Component, @Autowired & @Bean | Spring Boot Full Course #5](https://www.youtube.com/watch?v=Az9tIgCdaRU)  
**Channel:** [Coder Army](https://www.youtube.com/@CoderArmy9)

> **Study goal:** Build a permanent, master-level mental model of the Spring IoC container, Spring Beans, component scanning, dependency injection, `@Component`, `@Autowired`, `@Bean`, `@Configuration`, bean resolution, bean names, common failures, and the way all of these pieces fit together inside a real Spring Boot application.

---

# Table of Contents

1. [The Core Mental Model](#1-the-core-mental-model)
2. [What Problem Existed Before Spring IoC?](#2-what-problem-existed-before-spring-ioc)
3. [The Real Root Problem: Object Creation and Object Wiring](#3-the-real-root-problem-object-creation-and-object-wiring)
4. [Dependency Injection Before Spring](#4-dependency-injection-before-spring)
5. [IoC vs DI — Do Not Mix These Up](#5-ioc-vs-di--do-not-mix-these-up)
6. [What Is the Spring IoC Container?](#6-what-is-the-spring-ioc-container)
7. [ApplicationContext vs BeanFactory](#7-applicationcontext-vs-beanfactory)
8. [What Exactly Is a Spring Bean?](#8-what-exactly-is-a-spring-bean)
9. [Bean Definition vs Bean Instance](#9-bean-definition-vs-bean-instance)
10. [How Spring Knows What Beans Exist](#10-how-spring-knows-what-beans-exist)
11. [Four Important Ways to Register Beans](#11-four-important-ways-to-register-beans)
12. [`@Component` — Automatic Component Discovery](#12-component--automatic-component-discovery)
13. [Component Scanning Under the Hood](#13-component-scanning-under-the-hood)
14. [`@Service`, `@Repository`, `@Controller`, and `@Component`](#14-service-repository-controller-and-component)
15. [`@Autowired` — Dependency Injection](#15-autowired--dependency-injection)
16. [Constructor Injection](#16-constructor-injection)
17. [Field Injection](#17-field-injection)
18. [Setter / Method Injection](#18-setter--method-injection)
19. [What Spring Actually Does During Autowiring](#19-what-spring-actually-does-during-autowiring)
20. [What Happens When Multiple Beans Match](#20-what-happens-when-multiple-beans-match)
21. [`@Primary` and `@Qualifier`](#21-primary-and-qualifier)
22. [`@Bean` — Explicit Bean Registration](#22-bean--explicit-bean-registration)
23. [`@Configuration` + `@Bean`](#23-configuration--bean)
24. [`@Component` vs `@Bean`](#24-component-vs-bean)
25. [Why `@Bean` Is Essential for Third-Party Classes](#25-why-bean-is-essential-for-third-party-classes)
26. [Bean Naming](#26-bean-naming)
27. [The Complete Startup Story](#27-the-complete-startup-story)
28. [Bean Lifecycle](#28-bean-lifecycle)
29. [Spring Boot and `@SpringBootApplication`](#29-spring-boot-and-springbootapplication)
30. [Package Scanning and Project Structure](#30-package-scanning-and-project-structure)
31. [Full Real-World Example: Payment System](#31-full-real-world-example-payment-system)
32. [Full Real-World Example: Notification System](#32-full-real-world-example-notification-system)
33. [Full Real-World Example: Multiple Implementations](#33-full-real-world-example-multiple-implementations)
34. [Testing Benefits of IoC and DI](#34-testing-benefits-of-ioc-and-di)
35. [Why Constructor Injection Is Usually the Best Default](#35-why-constructor-injection-is-usually-the-best-default)
36. [Common Mistakes and Why They Fail](#36-common-mistakes-and-why-they-fail)
37. [Troubleshooting Table](#37-troubleshooting-table)
38. [Deep Dive: Reflection, BeanPostProcessor, and Internal Wiring](#38-deep-dive-reflection-beanpostprocessor-and-internal-wiring)
39. [`@Configuration` Proxying and the `@Bean` Method Trap](#39-configuration-proxying-and-the-bean-method-trap)
40. [Bean Scopes — Important Extension of the Mental Model](#40-bean-scopes--important-extension-of-the-mental-model)
41. [Lazy Beans](#41-lazy-beans)
42. [Circular Dependencies](#42-circular-dependencies)
43. [When IoC Becomes a Problem](#43-when-ioc-becomes-a-problem)
44. [Real Production Architecture](#44-real-production-architecture)
45. [Interview-Level Questions](#45-interview-level-questions)
46. [Master Mental Models](#46-master-mental-models)
47. [Quick Reference Cheat Sheet](#47-quick-reference-cheat-sheet)
48. [References](#48-references)

---

# 1. The Core Mental Model

The entire lesson can be reduced to one sentence:

> **Instead of your application manually creating and connecting every important object, Spring creates and connects the objects for you.**

Imagine an e-commerce application.

Without dependency injection:

```text
OrderService
    |
    | new
    v
PaymentService
    |
    | new
    v
PaymentGateway
```

The application is saying:

> “I know exactly which concrete implementation to construct, and I will manage the dependency myself.”

With Spring:

```text
                 Spring IoC Container
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
     OrderService   PaymentService  PaymentGateway
          |              ^
          +--------------+
             injected by Spring
```

The application says:

> “I need a `PaymentService` here. Spring, give me the correct dependency.”

That is the fundamental shift.

---

# 2. What Problem Existed Before Spring IoC?

Before frameworks such as Spring became popular, Java applications often handled object creation and dependency wiring manually.

Suppose we build an order system.

```java
public class OrderService {

    private final PaymentService paymentService;

    public OrderService() {
        PaymentGateway paymentGateway = new StripePaymentGateway();
        this.paymentService = new PaymentService(paymentGateway);
    }
}
```

This works.

The problem is not that `new` is bad.

The problem is **who should be responsible for deciding what gets created, how it gets configured, and how objects are connected?**

As applications grow, this becomes painful.

## Pain point 1 — Tight coupling

```java
PaymentGateway gateway = new StripePaymentGateway();
```

`OrderService` is now coupled to `StripePaymentGateway`.

If you want PayPal:

```java
PaymentGateway gateway = new PaypalPaymentGateway();
```

You have to modify `OrderService`.

## Pain point 2 — Object creation spreads everywhere

Imagine 100 classes each manually constructing their dependencies.

```text
main()
  -> new Controller(...)
      -> new Service(...)
          -> new Repository(...)
              -> new DataSource(...)
```

Then another part of the application does the same thing.

You slowly end up with object-creation logic distributed across the entire codebase.

## Pain point 3 — Testing becomes harder

Suppose `OrderService` creates its own payment gateway.

In a unit test, how do you replace Stripe with a fake gateway?

You cannot easily say:

```java
new OrderService(fakeGateway)
```

if the class itself insists on constructing the real dependency.

## Pain point 4 — Configuration becomes scattered

Imagine the payment gateway needs:

- API key
- endpoint
- timeout
- retry policy
- proxy
- connection pool

Without centralized configuration, every place that constructs it may configure it differently.

## Pain point 5 — Lifecycle management

Some objects need:

- initialization
- cleanup
- connection management
- shutdown
- proxies
- transactions
- monitoring

It is difficult for every developer to consistently manage all of that manually.

---

# 3. The Real Root Problem: Object Creation and Object Wiring

Many beginners think DI is mainly about avoiding `new`.

That is too shallow.

The deeper issue is **separation of responsibilities**.

Suppose there are these classes:

```text
OrderController
       |
       v
OrderService
       |
       v
PaymentService
       |
       v
PaymentGateway
```

There are two separate questions:

1. **What does each class do?**
2. **Who creates and connects the classes?**

Good design tries to keep those responsibilities separate.

Spring takes responsibility for much of question 2.

Your classes focus more heavily on question 1.

---

# 4. Dependency Injection Before Spring

Dependency Injection is a design technique, not something invented specifically by Spring.

You can use DI using plain Java.

```java
public interface PaymentGateway {
    void charge(double amount);
}
```

```java
public class StripePaymentGateway implements PaymentGateway {

    @Override
    public void charge(double amount) {
        System.out.println("Charging using Stripe: " + amount);
    }
}
```

```java
public class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void pay(double amount) {
        gateway.charge(amount);
    }
}
```

And the wiring can still be manual:

```java
public class Main {

    public static void main(String[] args) {
        PaymentGateway gateway = new StripePaymentGateway();
        PaymentService paymentService = new PaymentService(gateway);

        paymentService.pay(1000);
    }
}
```

That is already Dependency Injection.

The dependency is supplied **from outside**.

Spring adds a container that can perform this wiring automatically and consistently.

---

# 5. IoC vs DI — Do Not Mix These Up

These concepts are related, but they are not identical.

| Concept | Meaning |
|---|---|
| IoC | A broad architectural idea that control of some behavior is moved away from application code. |
| DI | A concrete technique for supplying dependencies from outside a class. |
| Spring IoC Container | The Spring infrastructure that manages object creation, configuration, dependency resolution, lifecycle, and more. |
| `@Autowired` | One mechanism Spring can use to inject dependencies. |

A good mental model is:

```text
Inversion of Control
        |
        +-- Dependency Injection
                |
                +-- Constructor injection
                +-- Setter/method injection
                +-- Field injection
```

Spring's container is broader than merely `@Autowired`.

---

# 6. What Is the Spring IoC Container?

Spring's official documentation describes the `ApplicationContext` as the IoC container responsible for instantiating, configuring, and assembling beans. The container uses configuration metadata to know what it should create and how objects relate to each other. citeturn748830search1

Think of the container as an **object factory + dependency graph manager + lifecycle manager + configuration engine**.

A simplified picture:

```text
                  Spring ApplicationContext
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   Bean Definitions    Dependency Graph   Lifecycle Rules
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                    Managed Objects
                       (Beans)
```

The container does not just randomly create Java objects.

It works from **metadata**.

That metadata can come from:

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@Configuration`
- `@Bean`
- XML configuration
- programmatic registration
- other Spring infrastructure

Spring supports Java configuration, annotations, and XML configuration as forms of bean metadata. citeturn748830search1turn748830search0

---

# 7. ApplicationContext vs BeanFactory

Two names appear frequently in Spring discussions:

```text
BeanFactory
ApplicationContext
```

`BeanFactory` is the lower-level IoC container abstraction.

`ApplicationContext` is a richer container built for common application use cases.

A practical mental model:

```text
BeanFactory
    |
    +-- Core bean creation
    +-- Dependency resolution
    +-- Lifecycle machinery

ApplicationContext
    |
    +-- Everything above
    +-- Events
    +-- Resource loading
    +-- Message resolution
    +-- Environment integration
    +-- More application-oriented infrastructure
```

In normal Spring Boot applications, you usually interact with the `ApplicationContext` rather than directly building a raw `BeanFactory`.

---

# 8. What Exactly Is a Spring Bean?

A **Spring Bean is an object that is instantiated, assembled, and managed by the Spring IoC container.**

This does **not** mean:

> “Any Java object is automatically a Spring Bean.”

This is a very important distinction.

```java
PaymentService paymentService = new PaymentService();
```

This creates an ordinary Java object.

Spring does not automatically know about it.

But:

```java
@Component
public class PaymentService {
}
```

when component scanning discovers that class, Spring can register it as a bean.

Or:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Now Spring knows about the returned object as a bean definition.

---

# 9. Bean Definition vs Bean Instance

This distinction is extremely important.

## Bean definition

Think of a bean definition as a **recipe**.

It tells Spring things such as:

- bean name
- bean type
- how to instantiate it
- dependencies
- scope
- lifecycle metadata
- qualifiers
- initialization behavior

## Bean instance

This is the actual Java object in memory.

```text
Bean Definition
      |
      | recipe
      v
Spring Container
      |
      | creates / obtains
      v
Java Object
      |
      v
Bean Instance
```

This explains why Spring can know about a bean before the actual Java object has necessarily been created.

That matters for startup processing, dependency resolution, scopes, lazy creation, and other container behavior.

---

# 10. How Spring Knows What Beans Exist

The container needs bean metadata.

There are multiple ways to provide it.

The most common modern Spring Boot techniques are:

```text
@Component
@Service
@Repository
@Controller
        |
        v
Component scanning

@Configuration + @Bean
        |
        v
Explicit bean definition
```

Spring's classpath scanner looks for candidate components and creates corresponding bean definitions. By default, stereotype annotations such as `@Component`, `@Repository`, `@Service`, `@Controller`, and `@Configuration` are among the detected candidates. citeturn748830search3

---

# 11. Four Important Ways to Register Beans

The most useful four concepts for this lesson are:

| Technique | Main idea | Typical use |
|---|---|---|
| `@Component` | “Spring, discover this class.” | Your own application class. |
| `@Service` | Specialized `@Component` for service layer. | Business logic. |
| `@Repository` | Specialized `@Component` for persistence layer. | Data access. |
| `@Bean` | “Spring, use this factory method to create a bean.” | Third-party classes or custom creation/configuration. |

The key difference is:

```text
@Component
    = Spring discovers the CLASS

@Bean
    = Spring calls a METHOD whose return value becomes the bean
```

That single distinction will answer many interview questions.

---

# 12. `@Component` — Automatic Component Discovery

`@Component` tells Spring that the annotated class is a candidate for component scanning and can become a managed bean. Spring's classpath scanning infrastructure detects candidate components and registers bean definitions. citeturn748830search3

Example:

```java
package com.example.payment;

import org.springframework.stereotype.Component;

@Component
public class PaymentGateway {

    public void charge(double amount) {
        System.out.println("Charging: " + amount);
    }
}
```

Then:

```java
@Component
public class PaymentService {

    private final PaymentGateway paymentGateway;

    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void pay(double amount) {
        paymentGateway.charge(amount);
    }
}
```

If both classes are in a package scanned by Spring, Spring can discover both and connect them.

The application no longer needs:

```java
new PaymentGateway()
new PaymentService(paymentGateway)
```

inside normal business code.

---

# 13. Component Scanning Under the Hood

This is where beginners often think Spring is “magic.”

It is actually a sequence of framework operations.

A simplified version is:

```text
Application starts
       |
       v
Spring creates ApplicationContext
       |
       v
Configuration is processed
       |
       v
@ComponentScan / component scanning
       |
       v
Spring searches configured packages
       |
       v
Find candidate classes
       |
       v
Read annotation metadata
       |
       v
Create BeanDefinition metadata
       |
       v
Register BeanDefinitions
       |
       v
Later: instantiate beans as required
       |
       v
Resolve dependencies
       |
       v
Initialize beans
```

The important idea is that **component scanning first discovers metadata; dependency injection happens as part of bean creation/processing.**

### Why package structure matters

Suppose:

```text
com.example.app.Application
com.example.app.service.OrderService
com.example.app.repository.OrderRepository
```

This is naturally aligned with Spring Boot's conventional scanning behavior.

But if you place your service here:

```text
org.somewhere.else.service.OrderService
```

Spring may not find it unless you explicitly configure scanning.

---

# 14. `@Service`, `@Repository`, `@Controller`, and `@Component`

These annotations are related.

Spring describes `@Repository`, `@Service`, and `@Controller` as more specific stereotypes built on the general `@Component` concept. citeturn748830search3

Think of them as semantic labels:

```text
@Component
   |
   +-- @Service
   +-- @Repository
   +-- @Controller
```

## `@Component`

Generic Spring-managed component.

```java
@Component
public class PasswordEncoderUtil {
}
```

## `@Service`

Use for business/service logic.

```java
@Service
public class OrderService {
}
```

## `@Repository`

Use for persistence/data-access components.

```java
@Repository
public class OrderRepository {
}
```

## `@Controller`

Use for MVC controller components.

```java
@Controller
public class OrderController {
}
```

## Why not use `@Component` everywhere?

You technically can for many custom components.

But specialized stereotypes communicate architectural intent.

Compare:

```java
@Component
public class OrderService {
}
```

with:

```java
@Service
public class OrderService {
}
```

The second immediately tells another engineer:

> “This class represents service-layer behavior.”

---

# 15. `@Autowired` — Dependency Injection

`@Autowired` tells Spring to resolve and inject a matching dependency into an injection point.

Spring supports `@Autowired` on constructors, fields, and methods. If a class has a single constructor, explicit `@Autowired` on that constructor is not required. citeturn748830search10

Example:

```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Modern Spring style can simplify this to:

```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

because there is only one constructor.

---

# 16. Constructor Injection

Constructor injection is usually the strongest default for required dependencies.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final OrderRepository orderRepository;

    public OrderService(
            PaymentService paymentService,
            OrderRepository orderRepository) {
        this.paymentService = paymentService;
        this.orderRepository = orderRepository;
    }

    public void placeOrder(Order order) {
        orderRepository.save(order);
        paymentService.pay(order.totalAmount());
    }
}
```

The object cannot be constructed without its required dependencies.

That is valuable because it makes invalid states harder to represent.

### Constructor injection flow

```text
Spring wants OrderService
        |
        v
Find constructor
        |
        v
Need PaymentService
        |
        v
Find PaymentService bean
        |
        v
Need OrderRepository
        |
        v
Find OrderRepository bean
        |
        v
Call constructor
        |
        v
new OrderService(paymentService, orderRepository)
        |
        v
Managed Spring bean
```

### Why constructor injection is good

- Dependencies are explicit.
- Fields can be `final`.
- The object is fully initialized at construction time.
- Unit tests can instantiate the class directly.
- Missing dependencies fail early.
- It reduces hidden framework magic inside the class.

---

# 17. Field Injection

Field injection looks like this:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring creates the object and then injects the field.

Conceptually:

```text
new OrderService()
        |
        v
Object exists
        |
        v
Spring finds @Autowired field
        |
        v
Resolve PaymentService
        |
        v
Set field
```

This is convenient, but constructor injection is generally a better design default for required dependencies.

### Why field injection is less desirable

- Dependencies are less visible from the constructor.
- The field can be null before framework processing.
- Plain Java tests become less natural.
- Required dependencies are hidden in implementation details.
- `final` cannot be used for normal field injection.

Field injection is still important to understand because legacy code and existing projects may contain it.

---

# 18. Setter / Method Injection

Spring can inject through setter methods or other methods.

Example:

```java
@Component
public class NotificationService {

    private EmailClient emailClient;

    @Autowired
    public void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
}
```

This can be appropriate when a dependency is optional or intentionally mutable.

But for a dependency that the class fundamentally requires, constructor injection usually communicates the design better.

---

# 19. What Spring Actually Does During Autowiring

Suppose you write:

```java
@Service
public class OrderService {

    private final PaymentGateway gateway;

    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Spring has to answer:

> “Which `PaymentGateway` should I inject?”

A simplified resolution process is:

```text
Injection point:
PaymentGateway gateway
        |
        v
Find candidate beans assignable to PaymentGateway
        |
        v
+------------------------------+
| StripePaymentGateway         |
| PaypalPaymentGateway         |
| RazorpayPaymentGateway       |
+------------------------------+
        |
        v
Is there exactly one usable candidate?
        |
        +---- yes --> inject it
        |
        +---- no --> continue with disambiguation
                         |
                  @Primary / @Qualifier / other rules
                         |
                         v
                    resolve or fail
```

Spring's autowiring infrastructure uses candidate information from the container; if a single dependency has no unique matching bean, the injection can become ambiguous. Spring provides mechanisms such as `@Primary` and `@Qualifier` to make the selection explicit. citeturn748830search11turn748830search10

---

# 20. What Happens When Multiple Beans Match

Suppose:

```java
public interface PaymentGateway {
    void charge(double amount);
}
```

And you have:

```java
@Component
public class StripeGateway implements PaymentGateway {
    @Override
    public void charge(double amount) {
        System.out.println("Stripe");
    }
}
```

```java
@Component
public class PaypalGateway implements PaymentGateway {
    @Override
    public void charge(double amount) {
        System.out.println("PayPal");
    }
}
```

Then:

```java
@Service
public class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Spring sees at least two candidates.

```text
PaymentGateway
     |
     +----> StripeGateway
     |
     +----> PaypalGateway
```

There is no obvious unique answer.

This commonly leads to a `NoUniqueBeanDefinitionException`-style failure.

That failure is useful: Spring is refusing to silently choose an implementation that may be wrong.

---

# 21. `@Primary` and `@Qualifier`

## `@Primary`

Use `@Primary` when one implementation should be the default choice.

```java
@Component
@Primary
public class StripeGateway implements PaymentGateway {

    @Override
    public void charge(double amount) {
        System.out.println("Stripe payment");
    }
}
```

Now a normal `PaymentGateway` injection can prefer Stripe when multiple candidates exist.

## `@Qualifier`

Use `@Qualifier` when a specific injection point needs a specific implementation.

```java
@Component("stripeGateway")
public class StripeGateway implements PaymentGateway {

    @Override
    public void charge(double amount) {
        System.out.println("Stripe");
    }
}
```

```java
@Component("paypalGateway")
public class PaypalGateway implements PaymentGateway {

    @Override
    public void charge(double amount) {
        System.out.println("PayPal");
    }
}
```

Then:

```java
@Service
public class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(
            @Qualifier("stripeGateway") PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

### Mental model

```text
@Primary
    = “Use this one by default.”

@Qualifier
    = “For THIS injection point, use THIS one.”
```

---

# 22. `@Bean` — Explicit Bean Registration

`@Bean` is a **method-level** annotation.

It tells Spring:

> “Call this factory method and manage the returned object as a bean.”

Spring's documentation explicitly describes `@Bean` as the Java-configuration equivalent of the XML `<bean/>` element. The bean name defaults to the method name unless another name is supplied. citeturn410277search5

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }
}
```

Conceptually:

```text
@Configuration class
        |
        v
@Bean method
        |
        v
paymentGateway()
        |
        v
StripePaymentGateway object
        |
        v
Registered Spring Bean
```

---

# 23. `@Configuration` + `@Bean`

The most common explicit configuration style is:

```java
@Configuration
public class PaymentConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    @Bean
    public PaymentService paymentService(PaymentGateway gateway) {
        return new PaymentService(gateway);
    }
}
```

Notice something important.

The `paymentService()` method receives:

```java
PaymentGateway gateway
```

as a method parameter.

Spring resolves that dependency and supplies it.

This is a clean way to write explicit configuration without manually doing all the wiring inside the configuration method.

Spring documentation recommends `@Configuration` classes as a standard location for `@Bean` definitions, and configuration processing preserves managed inter-bean relationships. citeturn410277search1turn410277search2

---

# 24. `@Component` vs `@Bean`

This is one of the most important interview questions in Spring.

| Feature | `@Component` | `@Bean` |
|---|---|---|
| Placed on | Class | Method |
| Discovery style | Component scanning | Configuration processing |
| Who decides the object creation method? | Spring discovers class and constructs it | Your factory method constructs/returns it |
| Great for | Your application classes | Third-party classes / custom creation logic |
| Typical location | Service, repository, controller, helper | `@Configuration` class |
| Can configure construction logic directly? | Less explicit | Yes |

Think:

```text
@Component
“I wrote this class. Spring, discover and manage it.”
```

versus:

```text
@Bean
“I need Spring to manage this object, but I want to explicitly define how it is produced.”
```

---

# 25. Why `@Bean` Is Essential for Third-Party Classes

You cannot normally modify a third-party library class to add:

```java
@Component
```

Suppose a library provides:

```java
public class ExternalApiClient {

    public ExternalApiClient(String apiKey, int timeout) {
        // ...
    }
}
```

You own neither the source nor the class design.

Use:

```java
@Configuration
public class ExternalClientConfig {

    @Bean
    public ExternalApiClient externalApiClient() {
        return new ExternalApiClient(
                "secret-key",
                5000
        );
    }
}
```

Now Spring can inject it anywhere:

```java
@Service
public class OrderService {

    private final ExternalApiClient client;

    public OrderService(ExternalApiClient client) {
        this.client = client;
    }
}
```

This is one of the strongest practical reasons to understand `@Bean`.

---

# 26. Bean Naming

Spring beans have names.

For a simple `@Component`:

```java
@Component
public class PaymentService {
}
```

the default bean name is commonly derived from the class name, typically:

```text
paymentService
```

For a `@Bean` method:

```java
@Bean
public PaymentGateway paymentGateway() {
    return new StripePaymentGateway();
}
```

the default bean name is:

```text
paymentGateway
```

because the method name is used by default. citeturn410277search5

You can explicitly choose names:

```java
@Bean("primaryPaymentGateway")
public PaymentGateway paymentGateway() {
    return new StripePaymentGateway();
}
```

Names become important when working with:

- qualifiers
- multiple implementations
- conditional configuration
- legacy XML configuration
- bean lookup by name

---

# 27. The Complete Startup Story

This is the most important “under the hood” mental model.

A simplified Spring Boot startup flow is:

```text
main()
  |
  v
SpringApplication.run(...)
  |
  v
Create application context
  |
  v
Read configuration metadata
  |
  +--> @SpringBootApplication
  +--> component scanning
  +--> @Configuration classes
  +--> @Bean methods
  +--> auto-configuration metadata
  |
  v
Register BeanDefinitions
  |
  v
Run configuration / factory post-processing
  |
  v
Prepare BeanPostProcessors
  |
  v
Create non-lazy singleton beans
  |
  v
Resolve constructor dependencies
  |
  v
Instantiate objects
  |
  v
Apply dependency injection / post-processing
  |
  v
Initialization callbacks
  |
  v
Application is ready
```

This is simplified because Spring startup contains many more internal details, but it is a strong architecture-level mental model.

---

# 28. Bean Lifecycle

For a normal singleton bean, think of the lifecycle as:

```text
Bean Definition
      |
      v
Instantiation
      |
      v
Dependency Injection
      |
      v
Aware callbacks / container callbacks
      |
      v
BeanPostProcessor: before initialization
      |
      v
@PostConstruct / initialization callback
      |
      v
BeanPostProcessor: after initialization
      |
      v
READY FOR APPLICATION USE
      |
      v
Application shuts down
      |
      v
@PreDestroy / destroy callback
      |
      v
Bean destroyed
```

This is why Spring is more than an object factory.

It is managing the lifecycle around those objects.

---

# 29. Spring Boot and `@SpringBootApplication`

A Spring Boot application often begins with:

```java
@SpringBootApplication
public class ShopApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShopApplication.class, args);
    }
}
```

Spring Boot documents `@SpringBootApplication` as a convenience annotation combining key behavior including:

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

citeturn748830search5

That means a single annotation participates in the process that allows Spring Boot to discover your application components and apply auto-configuration.

## The component-scan implication

If your main application class is here:

```text
com.example.shop.ShopApplication
```

then a conventional structure is:

```text
com.example.shop
├── ShopApplication.java
├── controller
├── service
├── repository
├── config
└── client
```

Spring Boot recommends placing the main application class in a root package above the other application packages so component scanning applies naturally to the project. citeturn748830search4

---

# 30. Package Scanning and Project Structure

A clean Spring Boot project often looks like:

```text
src/main/java
└── com/example/shop
    ├── ShopApplication.java
    ├── controller
    │   └── OrderController.java
    ├── service
    │   └── OrderService.java
    ├── repository
    │   └── OrderRepository.java
    ├── payment
    │   ├── PaymentGateway.java
    │   └── StripePaymentGateway.java
    └── config
        └── PaymentConfig.java
```

The root package matters because it determines what component scanning naturally reaches.

Avoid putting the main class in an unrelated package and then wondering why Spring says:

```text
NoSuchBeanDefinitionException
```

for a class that clearly has `@Component`.

---

# 31. Full Real-World Example: Payment System

Let's build a realistic mini-domain.

## Step 1 — Interface

```java
package com.example.payment;

public interface PaymentGateway {
    void charge(double amount);
}
```

## Step 2 — Concrete implementation

```java
package com.example.payment;

import org.springframework.stereotype.Component;

@Component
public class StripePaymentGateway implements PaymentGateway {

    @Override
    public void charge(double amount) {
        System.out.println(
                "Charging " + amount + " using Stripe"
        );
    }
}
```

## Step 3 — Service depending on abstraction

```java
package com.example.payment;

import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    private final PaymentGateway paymentGateway;

    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void pay(double amount) {
        paymentGateway.charge(amount);
    }
}
```

## Step 4 — Application

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ShopApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShopApplication.class, args);
    }
}
```

## What Spring sees

```text
@Component
StripePaymentGateway
      |
      | implements
      v
PaymentGateway
      ^
      |
      | dependency
      |
PaymentService
      |
      | @Service
      v
Spring Container
```

Spring detects the component, registers its bean definition, creates the bean, and uses it to satisfy the `PaymentGateway` dependency of `PaymentService`.

---

# 32. Full Real-World Example: Notification System

Suppose an order service must send notifications.

Define an abstraction:

```java
public interface NotificationSender {
    void send(String destination, String message);
}
```

Email implementation:

```java
@Component
public class EmailNotificationSender implements NotificationSender {

    @Override
    public void send(String destination, String message) {
        System.out.println(
                "Sending email to " + destination + ": " + message
        );
    }
}
```

Service:

```java
@Service
public class NotificationService {

    private final NotificationSender sender;

    public NotificationService(NotificationSender sender) {
        this.sender = sender;
    }

    public void notifyCustomer(
            String email,
            String message) {
        sender.send(email, message);
    }
}
```

This design is powerful because `NotificationService` does not know that email is being used.

It knows only:

```java
NotificationSender
```

That is dependency inversion at work.

---

# 33. Full Real-World Example: Multiple Implementations

Now suppose production supports both email and SMS.

```java
@Component("emailSender")
public class EmailNotificationSender implements NotificationSender {

    @Override
    public void send(String destination, String message) {
        System.out.println("Email: " + destination);
    }
}
```

```java
@Component("smsSender")
public class SmsNotificationSender implements NotificationSender {

    @Override
    public void send(String destination, String message) {
        System.out.println("SMS: " + destination);
    }
}
```

Now this is ambiguous:

```java
public NotificationService(NotificationSender sender) {
    this.sender = sender;
}
```

because Spring sees:

```text
NotificationSender
   |
   +--> emailSender
   |
   +--> smsSender
```

Use a qualifier:

```java
@Service
public class NotificationService {

    private final NotificationSender sender;

    public NotificationService(
            @Qualifier("emailSender") NotificationSender sender) {
        this.sender = sender;
    }
}
```

Now the dependency graph is explicit.

---

# 34. Testing Benefits of IoC and DI

One of the biggest practical benefits is testability.

Suppose:

```java
public interface PaymentGateway {
    void charge(double amount);
}
```

Production uses:

```java
StripePaymentGateway
```

The test can use:

```java
FakePaymentGateway
```

Example:

```java
class FakePaymentGateway implements PaymentGateway {

    boolean charged;

    @Override
    public void charge(double amount) {
        charged = true;
    }
}
```

Then:

```java
@Test
void shouldChargeCustomer() {
    FakePaymentGateway fake = new FakePaymentGateway();

    PaymentService service = new PaymentService(fake);

    service.pay(500);

    assertTrue(fake.charged);
}
```

Notice something very important:

The test does not need Spring.

That is a strong sign that the class has a clean dependency boundary.

---

# 35. Why Constructor Injection Is Usually the Best Default

Consider:

```java
@Service
public class OrderService {

    private final PaymentGateway paymentGateway;
    private final OrderRepository orderRepository;

    public OrderService(
            PaymentGateway paymentGateway,
            OrderRepository orderRepository) {
        this.paymentGateway = paymentGateway;
        this.orderRepository = orderRepository;
    }
}
```

The constructor immediately communicates:

> “This service cannot function without these two collaborators.”

The dependency graph is visible in the API of the class.

Compare field injection:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentGateway paymentGateway;

    @Autowired
    private OrderRepository orderRepository;
}
```

A reader has to scan the fields to discover the dependencies.

Constructor injection is therefore usually preferred for mandatory dependencies.

---

# 36. Common Mistakes and Why They Fail

## Mistake 1 — `@Component` on a class outside the scan path

```text
com.example.app.ShopApplication

org.random.service.OrderService
```

Spring may never discover `OrderService`.

### Fix

Move it under the appropriate root package or explicitly configure component scanning.

---

## Mistake 2 — Creating your own object with `new`

```java
OrderService service = new OrderService(...);
```

If you expected Spring to intercept and manage that object, it will not magically become a Spring bean.

### Fix

Let Spring create the object or explicitly register it.

---

## Mistake 3 — Multiple implementations with no qualifier

```text
PaymentGateway
  +-- Stripe
  +-- PayPal
```

Spring cannot safely choose one.

### Fix

Use `@Primary`, `@Qualifier`, or redesign the injection point if the ambiguity reveals a deeper architectural issue.

---

## Mistake 4 — Expecting `@Autowired` to create a missing bean

This:

```java
@Autowired
PaymentGateway gateway;
```

does not mean:

> “Spring, invent a PaymentGateway implementation.”

It means:

> “Spring, resolve a bean candidate matching this dependency.”

There must be a matching bean definition.

---

## Mistake 5 — Forgetting the difference between `@Bean` and `@Component`

Incorrect mental model:

```text
@Bean = class annotation
```

Correct:

```text
@Bean = method annotation
```

---

## Mistake 6 — Injecting concrete classes everywhere

Prefer:

```java
PaymentGateway
```

over:

```java
StripePaymentGateway
```

when the business code does not need Stripe-specific behavior.

This keeps the design replaceable.

---

## Mistake 7 — Putting business logic inside configuration

Bad:

```java
@Configuration
public class AppConfig {

    @Bean
    public OrderService orderService() {
        // 300 lines of business logic
        // ...
        return new OrderService(...);
    }
}
```

Configuration should mainly express how components are assembled.

Business behavior belongs in the application/domain layers.

---

# 37. Troubleshooting Table

| Error / symptom | Likely reason | What to inspect |
|---|---|---|
| `NoSuchBeanDefinitionException` | No matching bean exists | Component scan, `@Bean`, configuration |
| `NoUniqueBeanDefinitionException` | Multiple candidates match | `@Primary`, `@Qualifier` |
| `UnsatisfiedDependencyException` | A dependency cannot be resolved | Nested exception and bean graph |
| `@Autowired` field is null in a unit test | Object was created with `new`, outside Spring | Test setup and object construction |
| `@Component` seems ignored | Class is outside scan range | Main application package / `@ComponentScan` |
| Third-party object unavailable for injection | Library class was not registered as a bean | Create an `@Bean` method |
| Bean exists but wrong implementation injected | Multiple candidates or qualifier issue | Candidate names and qualifiers |
| Circular dependency error | Beans depend on each other cyclically | Dependency graph |
| Bean created twice unexpectedly | Separate object creation paths or scope/configuration issue | `new`, scopes, configuration proxying |

### Debugging strategy

Start from the exception's **deepest cause**.

For example:

```text
UnsatisfiedDependencyException
    |
    +-- NoSuchBeanDefinitionException
```

The top-level message describes the impact.

The deepest nested exception usually tells you the actual missing piece.

---

# 38. Deep Dive: Reflection, BeanPostProcessor, and Internal Wiring

The framework must perform work that your source code does not explicitly show.

A simplified model is:

```text
Class metadata
    |
    v
BeanDefinition
    |
    v
Instantiation
    |
    v
Dependency processing
    |
    v
Initialization
    |
    v
Post-processing
    |
    v
Managed bean
```

Spring's annotation support uses `BeanPostProcessor` infrastructure to provide processing for annotations such as `@Autowired`. The official documentation identifies `AutowiredAnnotationBeanPostProcessor` as the processor responsible for annotation-driven autowiring behavior. citeturn748830search9turn410277search0

A simplified conceptual example:

```text
Spring creates object
       |
       v
Inspect bean metadata
       |
       v
Find @Autowired injection points
       |
       v
Resolve dependency from container
       |
       v
Inject dependency
       |
       v
Continue bean initialization
```

## Why BeanPostProcessor matters

Spring can add behavior around object creation without every application class manually calling framework APIs.

This mechanism is used far beyond `@Autowired`.

Examples of Spring infrastructure using post-processing concepts include:

- dependency injection
- AOP proxying
- lifecycle processing
- annotation handling
- other container integration features

Spring's documentation distinguishes `BeanPostProcessor`, which works with bean instances, from `BeanFactoryPostProcessor`, which works with bean-definition/configuration metadata. citeturn410277search0

---

# 39. `@Configuration` Proxying and the `@Bean` Method Trap

This is an advanced but extremely useful detail.

Consider:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    @Bean
    public PaymentService paymentService() {
        return new PaymentService(paymentGateway());
    }
}
```

A beginner may think:

```java
paymentGateway()
```

simply executes the Java method and creates a new object every time.

For a full `@Configuration` class, Spring applies configuration-class processing so inter-bean method calls can be redirected through the container. Spring's documentation explains that `@Configuration` classes are subclassed using CGLIB so calls can preserve container-managed singleton behavior. citeturn410277search2turn410277search6

Conceptually:

```text
paymentService()
       |
       v
paymentGateway()
       |
       | direct Java-looking call
       v
Configuration proxy intercepts
       |
       v
Ask container for paymentGateway bean
       |
       v
Return managed singleton instance
```

## Why this matters

This works differently when configuration is processed in “lite” mode, such as a non-`@Configuration` class or `@Configuration(proxyBeanMethods = false)`.

Spring documentation notes that in lite mode, `@Bean` methods behave as ordinary factory methods and direct calls are not intercepted by the container. citeturn410277search1

Therefore, with:

```java
@Configuration(proxyBeanMethods = false)
public class AppConfig {
```

prefer method parameters for bean dependencies:

```java
@Bean
public PaymentService paymentService(PaymentGateway gateway) {
    return new PaymentService(gateway);
}
```

instead of relying on:

```java
paymentGateway()
```

The parameter-based version makes the dependency explicit and works cleanly with configuration optimization.

---

# 40. Bean Scopes — Important Extension of the Mental Model

The default Spring bean scope is `singleton`: one bean instance per bean definition within a container. Spring also supports scopes such as `prototype`, `request`, `session`, `application`, and `websocket` in appropriate contexts. citeturn748830search8

## Singleton

```java
@Component
public class OrderService {
}
```

By default, Spring manages it as a singleton-scoped bean.

Think:

```text
Container
   |
   +--> OrderService instance #1
```

A second injection receives the same managed instance for that bean definition.

## Prototype

```java
@Component
@Scope("prototype")
public class ReportBuilder {
}
```

The container may create a new instance each time the prototype is requested.

Mental model:

```text
getBean()
   -> instance A

getBean()
   -> instance B

getBean()
   -> instance C
```

Scopes become important when reasoning about state.

A singleton service should generally avoid storing mutable request-specific state in instance fields.

---

# 41. Lazy Beans

Spring applications often initialize singleton beans eagerly by default.

A bean can be made lazy:

```java
@Component
@Lazy
public class ExpensiveClient {

    public ExpensiveClient() {
        System.out.println("Created");
    }
}
```

The conceptual difference:

```text
Eager
Application starts
    |
    v
Create bean
    |
    v
Ready
```

versus:

```text
Lazy
Application starts
    |
    v
Bean definition exists
    |
    v
First request for bean
    |
    v
Create bean
```

Lazy initialization can help when startup cost is expensive, but it can also move failures from application startup to first use.

That is a trade-off, not automatically an optimization.

---

# 42. Circular Dependencies

Consider:

```java
@Service
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

and:

```java
@Service
class PaymentService {
    private final OrderService orderService;

    PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

The graph is:

```text
OrderService
      |
      v
PaymentService
      |
      v
OrderService
      |
     ...
```

There is no clean starting point for constructor creation.

The container may report a circular-dependency-related startup failure depending on configuration and Spring version.

The best solution is usually not “find a clever annotation.”

The best solution is to ask:

> “Why do these two responsibilities need to depend directly on each other?”

Often the solution is to extract a third abstraction.

For example:

```text
OrderService --> OrderPolicy <-- PaymentService
```

instead of:

```text
OrderService <--> PaymentService
```

This usually produces a cleaner architecture.

---

# 43. When IoC Becomes a Problem

Spring IoC is powerful, but it is possible to overuse it.

## Do not make every tiny object a bean

Spring itself explains that fine-grained domain objects are not typically what you configure in the container; repositories and business components generally create/load domain objects as needed. citeturn748830search1

Not every class needs:

```java
@Component
```

Ask:

> “Does this object benefit from being container-managed?”

Good bean candidates often include:

- stateless services
- repositories
- controllers
- integrations
- clients
- infrastructure components
- configuration objects

A simple value object such as:

```java
public record Money(BigDecimal amount, Currency currency) {
}
```

does not normally need to be a Spring bean.

---

# 44. Real Production Architecture

A realistic backend might look like this:

```text
                         +-------------------+
                         |    HTTP Client    |
                         | Web / Mobile / API|
                         +---------+---------+
                                   |
                                   v
                         +-------------------+
                         | Controller Layer  |
                         | @RestController   |
                         +---------+---------+
                                   |
                         injected dependency
                                   |
                                   v
                         +-------------------+
                         | Service Layer     |
                         | @Service          |
                         +----+---------+----+
                              |         |
                              |         |
                              v         v
                    +-------------+   +-------------+
                    | Repository  |   | Payment     |
                    | @Repository |   | Gateway     |
                    +------+------+   +------+------+
                           |                 |
                           v                 v
                      +---------+      +-----------+
                      | Database|      | External  |
                      |         |      | Provider  |
                      +---------+      +-----------+
```

Spring's IoC container sits around the whole graph.

Conceptually:

```text
              SPRING APPLICATION CONTEXT
   +------------------------------------------------+
   |                                                |
   | Controller                                    |
   |      |                                         |
   |      v                                         |
   | Service                                        |
   |   |       \                                   |
   |   v         v                                 |
   | Repo      PaymentGateway                      |
   |   |           |                               |
   |   v           v                               |
   | DB        External API                        |
   |                                                |
   +------------------------------------------------+
```

The key architecture pattern is:

```text
Business objects depend on abstractions.
Spring supplies concrete collaborators.
```

---

# 45. Interview-Level Questions

## Q1. What is IoC?

IoC is the architectural principle of transferring control of certain object creation/configuration responsibilities away from application code to a framework/container.

## Q2. What is DI?

Dependency Injection is a technique where an object's dependencies are supplied from outside rather than the object constructing them itself.

## Q3. What is a Spring Bean?

A Spring Bean is an object managed by the Spring IoC container.

## Q4. What is the difference between `@Component` and `@Bean`?

`@Component` marks a class for component scanning; `@Bean` marks a method whose returned object is registered as a bean.

## Q5. When should I use `@Bean`?

Use it when you need explicit factory/configuration control, especially for third-party classes or objects requiring custom construction.

## Q6. Does `@Autowired` create an object?

Not conceptually by itself. It identifies an injection point where Spring should resolve and supply a dependency from its managed bean graph.

## Q7. Why does Spring say “no qualifying bean”?

Because the container could not find a suitable bean matching the required dependency.

## Q8. Why does Spring say “expected single matching bean but found 2”?

Because multiple beans match and Spring needs additional information such as `@Primary` or `@Qualifier`.

## Q9. Why is constructor injection preferred?

It makes required dependencies explicit, supports immutable fields, improves testability, and prevents partially initialized objects.

## Q10. Is a class with `@Component` automatically a bean everywhere?

Only if Spring actually processes it as a component candidate through the appropriate component scanning/configuration path.

## Q11. Can `@Bean` exist without `@Configuration`?

Yes. Spring supports `@Bean` methods on component classes as well, but behavior differs from full `@Configuration` processing because configuration-class proxying is not necessarily applied.

## Q12. What happens with two implementations of the same interface?

A single dependency injection point may become ambiguous; use a qualifier, primary bean, or another explicit selection strategy.

---

# 46. Master Mental Models

## Mental Model 1 — Spring is an object graph builder

Think of your application as:

```text
A -> B -> C
```

Spring's job is largely to build and maintain that graph from configuration metadata.

## Mental Model 2 — `@Component` tells Spring what to discover

```text
Class
 |
 +-- @Component
       |
       v
   component scan
       |
       v
   BeanDefinition
```

## Mental Model 3 — `@Bean` tells Spring how to create something

```text
@Bean method
    |
    v
factory method
    |
    v
object returned
    |
    v
managed bean
```

## Mental Model 4 — `@Autowired` connects nodes

```text
Service
   |
   | needs PaymentGateway
   v
Spring resolves matching bean
   |
   v
PaymentGateway implementation
```

## Mental Model 5 — IoC is bigger than `@Autowired`

The container can manage:

```text
Creation
Configuration
Dependencies
Lifecycle
Scopes
Post-processing
Proxies
Events / resources / environment
```

---

# 47. Quick Reference Cheat Sheet

## `@Component`

```java
@Component
public class PaymentService {
}
```

Use when Spring should discover your class through component scanning.

## `@Service`

```java
@Service
public class OrderService {
}
```

Use for service-layer semantics.

## `@Repository`

```java
@Repository
public class OrderRepository {
}
```

Use for persistence components.

## `@Controller`

```java
@Controller
public class OrderController {
}
```

Use for MVC controllers.

## `@Autowired`

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

With a single constructor, explicit `@Autowired` is usually unnecessary.

## `@Bean`

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }
}
```

Use for explicit bean creation/configuration.

## `@Primary`

```java
@Component
@Primary
public class StripeGateway implements PaymentGateway {
}
```

Use when one implementation should be the default candidate.

## `@Qualifier`

```java
public PaymentService(
        @Qualifier("stripeGateway") PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Use when a specific implementation must be selected.

---

# 48. References

## Video

- [Coder Army — Spring IOC Container | Beans, @Component, @Autowired & @Bean | Spring Boot Full Course #5](https://www.youtube.com/watch?v=Az9tIgCdaRU)
- [Coder Army YouTube channel](https://www.youtube.com/@CoderArmy9)

## Spring Framework — official documentation

- [The Spring IoC Container](https://docs.spring.io/spring-framework/reference/core/beans.html)
- [Container Overview](https://docs.spring.io/spring-framework/reference/core/beans/basics.html)
- [Bean Overview](https://docs.spring.io/spring-framework/reference/core/beans/definition.html)
- [Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)
- [Annotation-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html)
- [Using `@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [Autowiring Collaborators](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-autowire.html)
- [Basic Concepts: `@Bean` and `@Configuration`](https://docs.spring.io/spring-framework/reference/core/beans/java/basic-concepts.html)
- [Using the `@Bean` Annotation](https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html)
- [Using the `@Configuration` Annotation](https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html)
- [Container Extension Points](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html)
- [Bean Scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)

## Spring Boot — official documentation

- [`@SpringBootApplication`](https://docs.spring.io/spring-boot/reference/using/using-the-springbootapplication-annotation.html)
- [Structuring Your Code](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)

---

# Final Memory Anchor

When you see a Spring application, picture this:

```text
                         YOU WRITE
                            |
          +-----------------+------------------+
          |                                    |
          v                                    v
   Business Classes                      Configuration
          |                                    |
   @Service / @Component                 @Bean / @Configuration
          |                                    |
          +----------------+-------------------+
                           |
                           v
                  SPRING IOC CONTAINER
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
         Bean Graph     Lifecycle      Scopes
             |
             v
     Dependency Resolution
             |
       +-----+-----+
       |           |
       v           v
  @Primary    @Qualifier
       |           |
       +-----+-----+
             |
             v
       Correct Object
             |
             v
       Ready to Use
```

The cleanest way to remember the four central annotations is:

```text
@Component
    ↓
“Discover and manage this class.”

@Bean
    ↓
“Create and manage the object returned by this method.”

@Autowired
    ↓
“Resolve and inject this dependency.”

@Configuration
    ↓
“These methods/classes define configuration for the container.”
```

And the whole architecture reduces to:

```text
                WITHOUT SPRING

Class A ---> new Class B()
   |             |
   |             +--> new Class C()
   |
   +--> knows creation details


                WITH SPRING IOC

Class A ---> needs interface B
                |
                v
         Spring Container
                |
                +--> chooses B implementation
                |
                +--> creates B
                |
                +--> resolves B's dependencies
                |
                +--> injects B into A
```

That is the heart of **Inversion of Control + Dependency Injection + the Spring IoC Container**.

---

## Source-verification note

This document is aligned to the supplied Coder Army lesson title and to publicly available learner notes describing Lecture 5's sequence of topics. The YouTube page itself was not directly retrievable through the current web environment, so this document intentionally does **not** invent exact timestamps, private lecture wording, or claims that cannot be verified. The Spring-specific mechanics are cross-checked against current official Spring Framework and Spring Boot documentation.
