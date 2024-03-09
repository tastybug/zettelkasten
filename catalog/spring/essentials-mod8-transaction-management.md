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

```
@Transactional
void method() {
  // trigger rollback
  throw new RuntimeException();
}
```

The transaction is bound to the current thread and holds a JDBC connection. `JdbcTemplate` used within a `@Transactional` method automatically use the managed db connection.

## Transaction Propagation

If you have 2 methods (`A` and `B`) both annotated as `@Transactional` and `A` calls `B`, you have to ask what the intended behaviour is. Do you want to shared the transaction? Avoid sharing? Something else? 
This can be configured in the annotation as such: `@Transactional(propagation=..)`:

* `propagation=Propagation.REQUIRED`: default; as `B`, do execute in the existing transaction or as `A` create a new transaction
* `propagation=Propagation.REQUIRES_NEW`: always create a dedicated transaction
* `propagation=Propagation.NESTED`: if there is a pre-existing transaction, execute a save point; if the nested transaction fails, roll back to the save point
* `propagation=Propagation.SUPPORTS`: if there is an pre-existing transaction, use that, otherwise proceed non-transactional
* `propagation=Propagation.MANDATORY`: there must be a pre-existing transaction to be used, otherwise throw exception
* `propagation=Propagation.NEVER`: throws an exception if there is a pre-existing transaction
* `propagation=Propagation.NOT_SUPPORTED`: ignore a pre-existing transaction if present and do your thing non-transactionally

> Warning, AOP uses proxies. Having 2 annotated methods in the same class calling each other lateraly will cause the 2nd method to not be subject to the configured advice. Example: `@Transaction(propagation=Propagation.REQUIRED) methodA(..)` calling `@Transaction(propagation=Propagation.REQUIRES_NEW) methodB(..)` means that `methodB` will NOT get a new transaction. Instead it will use the transaction from `methodA`. Call `methodB` from the outside to circumvent this.

## Rollback Rules

As described above, rollbacks happen on `RuntimeException`s and subclasses of it. But this behaviour can be tweaked: you can ignore certain exceptions or include checked exceptions to trigger rollbacks too.

```
@Transactional(rollbackFor=SomeCheckedException.class,
               noRollbackFor={JmxException.class, MailException.class})
void someMethod() throws Exception {
  // ...
}
```

## Makeing Test Cases @Transactional Causes Auto-Rollbacks

There is a neat feature: if you annotate a test case as `@Transactional`, the transaction is always rolled back. No need to clean up the DB programmatically. However, this is only true for outer transactions. If inside the system under test another inner transaction is started, that will not be affected.

So it's not terribly reliable.
