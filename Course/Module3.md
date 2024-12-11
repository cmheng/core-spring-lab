# More on Java Configuration


## Use Enternal Properties

Setting property values
* Consider this bean definition from the previous module:

```java
@Bean 
public DataSource dataSource() {
    BasicDataSource ds = new BasicDataSource();
    dataSource.setDriverClassName("org.postgresql.Driver");
    dataSource.setUrl("jdbc:postgresql://localhost/transfer");
    dataSource.setUsername("transfer-app");
    dataSource.setPassword("secret45");
    return ds;
}
```

* Hard-coding these properties is a Bad practice
    - Better practice is to "externalize" these properties
    - One way to externalize them is by using property files

Spring's Environment Abstraction - 1
* Environment bean represents loaded properties from runtime environment
* Properties derived from various sources, in this order:
    - JVM System Properties - System.getProperty()
    - System Environment Variables - System.getenv()
    - java Properties Files

Spring's Environment Abstraction - 2

```java
@Configuration
public class DbConfig {

    @Bean 
    public DataSource dataSource(Environment env) {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName(env.getProperty("db.driver"));
        ds.setUrl(env.getProperty("db.url"));
        ds.setUsername(env.getProperty("db.user"));
        ds.setPassword(env.getProperty("db.password"));
        return ds;
    }
}
```

app.properties
```
db.driver=org.postgresql.Driver
db.url=jdbc:postgresql://localhost/transfer
db.user=transfer-app
db.password=secret45
```

Property Sources
* Environment bean obtains values from "property sources"
    - Environment variables and Java System Properties always populated automatically
    - @PropertySource contributes additional properties
    - Available resource prefixes: classpath: file: http:

```Java
@Configuration
@PropertySource("classpath:/com/organization/config/app.properties")
@PropertySource("file:config/local.properties")
public class ApplicationConfig {
    ...
}
```

Accessing Properties using @Value

```java
@Configuration
public class DbConfig {
    @Bean
    public DataSource dataSource(
            @Value("${db.driver}") String driver,
            @Value("${db.url}") String url,
            @Value("${db.user}") String user,
            @Value("${db.password}") String pwd) {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(user);
        ds.setPassword(pwd);
        return ds;
    }
}
```

## Spring Profiles

Profiles - Beans can be grouped into Profiles
* Profiles can represent environment: dev, test, production
* Or implementation: "jdbc", "jpa"
* Or deployment platform: "on-premise", "cloud"
* Beans included / excluded based on profile membership
* Beans with no profile always available

Defining Profiles - 1
* Using `@Profile` annotation on configuration class
    - Everything in Configuration belong to the profile

Nothing in this configuration will be used unless "embedded" profile is chosen as one of the active profiles
```java
@Configuration
@Profile("embedded")
public class Devconfig {

    @Bean
    public DataSource dataSource() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        return builder.setName("testdb")
                .setType(EmbeddedDatabaseType.HSQL)
                .addScript("classpath:/testdb/schema.db")
                .addScript("classpath:/testdb/test-data.db").build();
    }
}
```

Defining Profiles - 2
* Using `@Profile` annotation on @Bean methods

```java
@Configuration
public class DataSourceConfig {
    @Bean(name="dataSource")
    @Profile("embedded")
    public DataSource dataSourceForDev() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        return builder.setName("testdb")...
    }


    @Bean(name="dataSource")
    @Profile("!embedded")
    public DataSource dataSourceForProd() {
        BasicDataSource dataSource = new BasicDataSource();
        ...
        return dataSource;
    }
}
```

Defining Profiles - 3
* Beans when a profile is not active

```java
@Configuration
@Profile("cloud")
public class DevConfig {
    ...
}
```

```java
@Configuration
@Profile("!cloud")
public class ProdConfig {
    ...
}
```

Ways to Activate Profiles
* Profiles must be activated at run-time
    - System property via command-line
    ```
    -Dspring.profiles.active=embedded,jpa
    ```
    - System property programmatically
    ```java
    System.setProperty("spring.profiles.active", "embedded, jpa");
    SpringApplication.run(AppConfig.class);
    ```
    - Integration Test only: @ActiveProfiles

Property Source selection
* @Profile can control with @PropertySources are included in the Environment

```java
@Configuration
@Profile("local")
@PropertySource("local.properties")
class DevConfig{...}
```

```java
@Configuration
@Profile("cloud")
@PropertySource("cloud.properties")
class ProdConfig{...}
```

## Spring Expression Language (SpEL)

SpEL examples - Using @Value

```java
@Configuration
class TaxConfig {
    @Value ("#{systemProperties['user.region']}") String region;

    @Bean
    public TaxCalculator taxCalculator1() {
        return new TaxCaluclator(region);
    }

     @Bean
    public TaxCalculator taxCalculator2(@Value ("#{systemProperties['user.region']}") String region, ...) {
        return new TaxCaluclator(region);
    }
}
```

SpEL - Accessing Spring Beans

```java
class StrategyBean {
    private KeyGenerator gen = new KeyGenerator.getInstance("Blowfish");
    public KeyGenerator getKeyGenerator() {return gen;}
}
```

```java
@Configuration
class StrategyConfig {

    @Bean
    public StrategyBean strategyBean() {
        return new StrategyBean();
    }
}
```

```java
@Configuration
@Import(StrategyConfig.class)
class AnotherConfig {
    
    @Value("#{strategyBean.keyGenerator}") KeyGenerator kgen;
}
```

Accessing Properties
* Can access properties via the environment
    - These are equivalent
    ```java
    @Value("${daily.limit}")
    int maxTransfersPerDay;
    ```
    ```java
    @Value("#{environment['daily.limit']}")
    int maxTransfersPerDay;
    ```
* Properties are Strings
```java
@Value("#{new Integer(environment['daily.limit']) * 2}") // OK
@Value("#{new java.net.URL(environment['home.page']).host}") // OK
@Value("${daily.limit * 2}") // NOT OK
```

Fallback Values
* Provviding a fall-back value
    - if `daily.limit` undefined, use fall-back value
    ```java
    @Autowired
    public TransferServiceImpl(@Value("${daily.limit : 100000}") int max) {
        this.maxTransfersPerDay = max;
    }
    ```
    - For SpEL, use the "Elvis" operator ?:
    ```java
    @Autowired
    public TransferServiceImpl(@Value("#{environment['daily.limit'] ?: 100000}") int max) {
        this.maxTransfersPerDay = max;
    }
    ```

SpEL
* EL Attributes can be:
    - Spring beans (like strategyBean)
    - Implicit references
        * Spring's *environment, systemProperties, systemEnvironment* available by default
        * Others depending on context
* SpEL allows to create custom functions and references
    - Widely used in Spring projects
        * Spring Security, Spring WebFlow
        * Spring Batch, Spring Integration
    - Each may add their own implict references

Summary
* Property values are easily externalized using Spring's Environment abstraction
* Profiles are used to group sets of Beans
* Spring Expression Language