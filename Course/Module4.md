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


## Adding Startup and Shutdown Behaviors


## Stereotype and Meta Annotations