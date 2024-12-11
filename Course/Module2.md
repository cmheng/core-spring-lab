# Module 2: Java Configuration

## Quick Start with Java Configuration

Configuration Instructions with Dependencies
```java
@Configuration
public class ApplicationConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository());
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource());
    }

    @Bean 
    public DataSource dataSource() {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName("org.postgresql.Driver");
        ds.setUrl("jdbc:postgresql://localhost/transfer");
        ds.setUsername("transfer-app");
        ds.setPassword("secret45");
        return ds;
    }
}
```

Configuration Instructions with Dependencies - Alternative
```java
@Configuration
public class ApplicationConfig {
    @Bean
    public TransferService transferService(AccountRepository repository) {
        return new TransferServiceImpl(repository);
    }

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean 
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("org.postgresql.Driver");
        dataSource.setUrl("jdbc:postgresql://localhost/transfer");
        dataSource.setUsername("transfer-app");
        dataSource.setPassword("secret45");
        return dataSource;
    }
}
```

Creating and Using the Application
```java
// Create application context from the configuration
ApplicationContext context = SpringApplication.run(ApplicationConfig.class);

// Look up a bean from the application context
TransferService service = context.getBean("transferService", TransferService.class);

// Use the bean
service.transfer(new MonetaryAmount("300.00"), "1", "2");
```

Accessing a Bean Programmatically
```java
ApplicationContext context = SpringApplication.run(...);

// Use bean id, a cast is needed
TransferService ts1 = (TransferService) context.getBean("transferService");

// Use typed method to avoid casting
TransferService ts2 = context.getBean("transferService", TransferService.class);

// No need for bean id if type is unique - recommended (use type whenever possible)
TransferService ts3 = context.getBean(TransferService.class);
```

Quick Start Summary
* Spring separates application configuration from application objects (beans)
* Spring manages your application objects
    - Creating them in the correct dependency order
    - Ensuring they are fully initialized before use
* Each bean is given a unique id / name

## The Application Context

Creating a Spring Application Context
* Spring application context represents Spring DI container
    - Spring beans are managed through the application context
* Spring application context can be created in any environment, including
    - Standalone application
    - Web application
    - JUnit test

Application Context Example - Creating Application Context in a System Test

```java
public class TransferServiceTests {
    private TransferService service;

    @BeforeEach
    public void setUp() {
        // Create application context from the configuration
        ApplicationContext context = SpringApplication.run(ApplicationConfig.class);

        // Look up a service
        service = context.getBean(TransferService.class);
    }

    @Test
    public void moneyTransfer() {
        Confirmation receipt = service.transfer(new MonetaryAmount("300.00"), "1", "2");
        Assert.assertEquals("500.00", receipt.getNewBalance());
    }
}
```

## Handling Multiple Configurations

Creating an Application Context from Multiple Configurations
* Your `@Configuration` class can get too big
    - Instead use multiple config. files combined with `@Import`
    - Defines a single Application Context
        * Beans sourced from multiple files

```java
@Configuration
@Import({ApplicationConfig.class, WebConfig.class})
public class InfrastructureConfig {
    ...
}
```

```java
@Configuration
public class ApplicationConfig {
    ...
}
```

```java
@Configuration
public class WebConfig {
    ...
}
```

Creating an Application Context from Multiple Files
* Separation of Concerns principle
    - Keep related beans in the same `@Configuration`
* Best Practice: separate "application" & "infrastructure"
    - Infrastructure often changes between environments

Mixed Configuration

```java
@Configuration
public class ApplicationConfig {
    @Bean
    public TransferService transferService(AccountRepository repository) {
        return new TransferServiceImpl(repository);
    }

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean 
    public DataSource dataSource() {
        ...
    }
}
```

Partitioning Configuration

```java
@Configuration
public class ApplicationConfig {
    @Bean
    public TransferService transferService(AccountRepository repository) {
        return new TransferServiceImpl(repository);
    }

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}
```

```java
@Configuration
@Import(ApplicationConfig.class)
public class TestInfrastructureConfig {

    @Bean 
    public DataSource dataSource() {
        ...
    }
}
```

```java
ApplicationContext ctx = SpringApplication.run(TestInfrastructureConfig.class)
```

Java Configuration with Dependency Injection
* Use `@Autowired` to inject a bean defined elsewhere

```java
@Configuration
public class ApplicationConfig {
    private final DataSource dataSource;

    @Autowired
    public ApplicationConfig(DataSource ds) {
        this.dataSource = ds;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}
```

```java
@Configuration
@Import(ApplicationConfig.class)
public class InfrastructureConfig {
    @Bean
    public DataSource dataSource() {
        DataSource ds = new BasicDataSource();
        ...
        return ds;
    }
}
```

... But Avoid "Tramp Data"

Bad: dataSource is a "tramp"!

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public AccountService accountService(DataSource ds) {
        return new AccountService(accountRepository(ds));
    }

    @Bean
    public AccountRepository accountRepository(DataSource ds) {
        return new JdbcAccountRepository(ds);
    }
}
```

Better: Pass acutal dependency
```java
@Configuration
public class ApplicationConfig {

    @Bean
    public AccountService accountService(AccountRepository repo) {
        return new AccountService(repo);
    }

    @Bean
    public AccountRepository accountRepository(DataSource ds) {
        return new JdbcAccountRepository(ds);
    }
}
```

## Bean Scopes

Bean Scope: Default
* Default scope is singleton

```java
@Bean
public AccountService accountService() {
    return ...
}
```

Equaivalent

```java
@Bean
@Scope("singleton")
public AccountService accountService() {
    return ...
}
```

```java
AccountService service1 = (AccountService) context.getBean("accountService");
AccountService service2 = (AccountService) context.getBean("accountService");
assert service1 == service2; // True - same object
```

Implications for Singleton Beans
* Typical Spring application - back-end web-server
    - Multiple requests in parallel
        * Handled by multiple threads
    - Implications:
        * Multiple threads accessing singleton beans at the same time
* Handle multi-threading issues
    - Use stateless or Immutable beans
    - Use synchronized (harder)
    - Use a different scope

Bean Scope: prototype
* Scope "prototype"
    - New instance created every time bean is referenced

```java
@Bean
@Scope("prototype")
public Action deviceAction() {
    return ...
}
```

```java
Action action1 = (Action) context.getBean("deviceAction");
Action action2 = (Action) context.getBean("deviceAction");
assert action1 != action2; // True - different objects
```

Common Spring Scopes
* The most commonly used scopes are:

    |||
    | -| - |
    | singleton | A single instance is used |
    | prototype | A new instance is created each time the bean is referenced |
    | session | A new instance is created once per user session - *web environment only*
    | request | A new instance is created once per request = *web environment only* |

Other Scopes
* Spring has other more specialized scopes
    - Web Socket scope
    - Refresh Scope
    - Thread Scope (defined but not registered as default)
* Custom scopes (rarely)
    - You define a factory for creating bean instances
    - Register to define a custom scope name

Dependancy Injection Summary
* Your object is handed with what it needs to work
    - Frees it from the burden of resolving its dependences
    - Simplifies your code, improves code reusabillity
* Promotes programming to interfaces
    - Conceals implementation details of dependencies
* Improves testability
    - Dependencies easily stubbed out for unit testing
* Allows for centralized control over object lifecycle
    - Opens the door for new possiblities