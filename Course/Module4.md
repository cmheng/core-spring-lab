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


## Configuration Choices


## Adding Startup and Shutdown Behaviors


## Stereotype and Meta Annotations