# Component Scanning


## Annotation-based Configuration

Before - *Explicit* Bean Definition (Covered in Previous Module)
* Configuration is external to bean-class
    - Separation of concerns
    - Java-based dependency injection

```java
@Configuration
public class TransferModuleConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository());
    }

    @Bean
    public AccountRepository accountRepository() {

    }
}
```

After - *Implicit* Configuration (Covered in this module)
* Annotation-based configuration within bean-class
* Component-scanning

```java
@Component
public class TransferServiceImpl implements TransferService {

    public TransferServiceImpl(AccountRepository repo) {
        this.accountRepository = repo;
    }
}
```

```java
@Configuration
public class AnnotationConfig {
    // No bean definition needed any more
}
```

Usage of @Autowired
* Constructor-injection (recommended practice)
```java
@Autowired // Optional if this is the only constructor
public TransferServiceImpl(AccountRepository a) {
    this.accountRepository = a;
}
```
* Method-injection
```java
@Autowired
public void setAccountRepository(AccountRepository a) {
    this.accountRepository = a;
}
```
* Field-injection
```java
@Autowired
private AccountRepository accountRepository;
```

@Autowired Dependencies: Required or Optional?
* Default behavior: required
```java
// Exception if no dependency found
@Autowired
public void setAccountRepository(AccountRepository a) {
    this.accountRepository = a;
}
```
* Use *required* attribute to override default behavior
```java
// Only inject if dependency exists
@Autowired(required=false)
public void setAccountRepository(AccountRepository a) {
    this.accountRepository = a;
}
```

Java 8 Optional<T>
* Another way to inject optional dependencies
    - OPtional<T> introduced to reduce null pointer errors
```java
@Autowired
public void setAccountService(AccountService accountService) {
    this.accountService = accountService;
}

public void doSomething() {
    if (accountService != null) {
        // do something
    }
}
```

```java
@Autowired
public void setAccountService(Optional<AccountService> accountService) {
    this.accountService = accountService;
}

public void doSomething() {
    accountService.ifPresent(s-> {
        // do something
    });
}
```

Constructor vs Setter Dependency Injection
* Spring doesn't care (can use either)
    - But which is better?

    | Constructors | Setters |
    | - | - |
    | Mandatory dependencies | Circular dependencies possible |
    | Dependencies can be immutable | Dependencies are mutable |
    | Concise (pass several params at once) | Could be verbose for several params |
    | | Inherited automatically |

* Follow the same rules as standard Java
    - Constructor injection is generally preferred
    - Be consistent across our project team

Autowiring and Disambiguation - 1

Which one should get injected?

```java
@Component
public class TransferServiceImpl implements TransferService {
    @Autowired // optional
    public TransferServiceImpl(AccountRepository accountRepository) {...}
}
```

```java
@Component
public class JpaAccountRepository implements AccountRepository {...}
```

```java
@Component
public class JdbcAccountRepository implements AccountRepository {...}
```

At startup: NoSuchBeanDefinitionException, no unique bean of type [AccountRepository] is defined: expected single bean but found 2...

Autowiring and Disambiguation - 2
* Use of the @Qualifier annotation

```java
@Component("transferService")
public class TransferServiceImpl implements TransferService {
    @Autowired
    public TransferServiceImpl(@Qualifier("jdbcAnnocationRepository") AccountRepository accountRepository) {...}
}
```

```java
@Component("jdbcAccountRepository")
public class JpaAccountRepository implements AccountRepository {...}
```

```java
@Component("jpaAccountRepository")
public class JdbcAccountRepository implements AccountRepository {...}
```

Autowiring and Disambiguation - 3
* Autowired resolution rules
    1. Look for unique bean of required type
    2. Use @Qualifier if supplied
    3. Try to find a matching bean by name

Component Names
* When not specified
    - Names are auto-generated
        * De-capitalized non-qualified class name by default
        * But will pick up implementation details from class name
    - Recommendation: never rely on generated names!
* When specified
    - Allow disambiguation when 2 bean classes implement the same interface

Using @Value to set Attributes
* Constructor-injection
```java
@Autowired // Optional if this is the only constructor
public TransferServiceImpl(@Value("${daily.limit}") int max) {
    this.maxTransfersPerDay = max;
}
```

* Method-injection
```java
@Autowired
public void setDailyLimit(@Value("${daily.limit}") int max) {
    this.maxTransfersPerDay = max;
}
```

* Field-injection
```java
@Value("#{environment['daily.limit']}")
int maxTransfersPerDay;
```

Delayed Initialization
* Beans normally created on startup when applicaiton context created
* Lazy beans created first time used
    - When dependency injected
    - By ApplicationContext.getBean methods
* Useful if bean's dependencies not available at startup
```java
@Lazy @Component
public class MailService {
    public MailService(@Value("smtp:...") String url) {
        // connect to mail-server
    }
}
```

Annotations syntax vs Java Config
* Similar options are available

Annotations
```java
@Component("transferService")
@Scope("prototype")
@Profile("dev")
@Lazy(true)
public class TransferServiceImpl implements TransferService{
    @Autowired
    public TransferServiceImpl (AccountRepository accRep) {...}
}
```

Java Configuration
```java
@Configuration
public class TransferConfiguration {
    
    @Bean(name="transferService")
    @Scope("prototype")
    @Profile("dev")
    @Lazy("true")
    public TransferService tsvc() {
        return new TransferServiceImpl(accountRepository());
    }
}
```

## Configuration Choices

Autowiring Constructors
* If a class only has a default constructor
    - Nothing to annotate
* If a class has only one non-default constructor
    - It is the only constructor available, Spring will call it
    - `@Autowired` is optional
* If a class has more than one constructor
    - Spring invokes zero-argument constructor by default (if it exists)
    0 Or you *must* annotate with `@Autowired` the one you want Spring to use

About Component Scanning
* Components are scanned at startup
    - JAR dependencies also scanned!
    - Could result in slower startup time if too many files scanned
        * Especially for large applications
* What are the best practices?

Component Scanning Best Practices
* Really bad:
```java
@ComponentScan({"org", "com"})
```

* Still bad:
```java
@ComponentScan({"com"})
```

* OK:
```java
@ComponentScan({"com.bank.app"})
```

* Optimizied:
```java
@ComponentScan({"com.bank.app.repository", "com.bank.app.service", "com.bank.app.controller"})
```

Mixing Java Config and Annotations
* Common approach selecting when to use:
    - Annotations
        * In some cases, Spring will give no other choice!
        * Stereotype annotations
    - Java Configuration
        * When is it desired or required to keep beans decoupled from Spring (legacy code, or code that can be used outside of Spring runtime)
        * When managing configurations in a single logical location is an important issue
        * Can be used for all classes (not just your own)

## Adding Startup and Shutdown Behaviors

@PostConstruct and @PreDestroy
* Add behavior at startup and shutdown
```java
public class JdbcAccountRepository {
    
    // Method called at startup after all dependencies are injected
    @PostConstruct
    void populateCache() {}

    // Method called at shutdown prior to destroying the bean instance
    @PreDestroy
    void flushCache() {}
}
```

Annotated methods can have any visibility but must take not parameters and only return void.

* Beans are created in the usual ways:
    - Returned from @Bean methods
    - Found and created by the component-scanner
* Spring then invokes these methods automatically
    - During bean-creation process
* These are not Spring annotations
    - Defined by JSR-250, part of Java since Java 6
    - In `javax.annotation` package
    - Support by Spring, and by Java EE

@PostConstruct
* Called after setter injections are performed
```java
public class JdbcAccountRepository {
    private DataSource dataSource;

    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @PostConstruct
    public void populateCache() {
        Connection con = dataSource.getConnection(); //...
    }
}
```
Constructor injection -> Setter injection -> @PostConstruct method(s) called

@PreDestroy
* PreDestroy methods called if application shuts down normally. Not if the process dies or is killed.
* Called when a `ConfigurableApplicationContext' is closed
    - Useful for releasing resources & 'cleaning up'
    - Not called for prototype beans

```java
ConfigurableApplicationContext context = SpringApplication.run(...);
...
context.close() // Trigger call of all @PreDestroy annotated methods
```

Lifecycle Method Attributes of @Bean Annotation
* Alternatively, @Bean has options to define these life-cycle methods
```java
@Bean(initMethod="populateCache", destroyMethod="flushCache")
public AccountRepository accountRepository() {
    // ...
}
```

* Which scheme to use?
    - Use `@PostConstruct/@PreDestroy` for your own classes
    - Use Lifecycle Method attributes of `@Bean` annotation for classes you didn't write and can't annotate

Use a JVM Shutdown Hook
* Shutdown hooks
    - Automatically run when JVM shuts down
* SpringApplication.run
    - Does this automatically
    - Returns a ConfigurableApplicationContext

## Stereotype and Meta Annotations

Stereotype Annotations
* Component scanning also checks for annotations that are themselves annotated with @Component
    - So-called stereotype annotations

```java
@Service("transferService")
public class TransferServiceImpl implements TransferService {...}
```

@Service annotation is part of the Spring framework

Predefined Stereotype Annotations
* Spring framework stereotype annotations
    - @Component
        * @Service
        * @Repostiory
        * @Controller
        * @RestController
        * @Configuration

Meta-annotations
* Annotation which can be used to annotate other annotations
    - e.g. all service beans should be configurable using component scanning and be transactional

```java
@ComponentScan("...)
```

```java
@MyTransactionalService
public class TransferServiceImpl implements TransferService {...}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Service    // meta-annotation
@Transactional(timeout=60)
public @interface MyTransactionalService {
    String value() default "";
}
```

Summary
* Spring beans can be defined
    - Explicity using @Bean methods inside configuration class
    - Implicitly using @Component and component-scanning
* Applications can use both
    - Implicit for your classes
    - Explicit for the rest - prefer for large apps
* Can perform initalization and clean-up
    - Use @PostConstruct and @PreDestroy
* Use Spring's stereotypes and/or define your own meta annotations