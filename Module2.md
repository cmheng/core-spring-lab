# Module 2: Java Configuration

## Quick Start with Java Configuration

```java
@Configuration
public class ApplicationConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository());
    }

    @Bean
    
}
```

## The Application Context

## Handling Multiple Configurations

## Bean Scopes