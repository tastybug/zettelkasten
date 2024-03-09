# Tranaction Management w/ Spring

Why:

1. Transactions are a means to achieve ACID.
2. Sharing the same connection across multiple operations ("Connection per Unit-of-Work")

## Transaction Management with Spring

Spring separates between __demarcation__ (marking the boundaries of the transaction) from the transaction __implementation__.
Demarcation happens with AOP (around advice) in a declarative manner (`@org.springframework.transaction.annotation.Transactional`), but can also be done programmatically. Implementation otoh is done via `PlatformTransactionManager` abstraction, which come with various implementations:

* `DataSourceTransactionManager`
* `JmsTransactionManager`
* `JpaTransactionManager`
* `JtaTransactionManager`
* ..

> Make sure to use Spring's `@Transactional` annotiation, not the `javax` one. The latter one also works, but has fewer attributes and thus fewer options to configure behaviour.

### The Most Basic Setup

Setting this up in a basic case involves:
1. setting up a `PlatformTransactionManager` bean
2. declaring the transactional methods
3. add `@EnableTransactionManagement` to a configuration class

```java
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
  @Bean
  public PlatformTransactionManager transactionManager(DataSource ds) {
    return new DataSourceTransactionManager(ds);
  }
}

@Transactional // applies to all methods
public class RewardNetworkImpl implements RewardNetwork {

  @Transactional(timeout=45) // overrides attributes from the class level
  public RewardConfirmation rewardAccountFor(Dining d) {
    // ..
  }
}
```

### When Rollbacks Happen

The transaction handling is done with an AOP `around` advice. The advice starts the transaction on entry and commits when the delegate returns. By default, and this is configurable, a rollback happens on `RuntimeException`s from the delegate, whereas checked exceptions do not cause a rollback.

The transaction is bound to the current thread and holds a JDBC connection. `JdbcTemplate` used within a `@Transactional` method automatically use the managed db connection.
