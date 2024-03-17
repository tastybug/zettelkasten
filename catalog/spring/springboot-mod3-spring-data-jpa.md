# Spring Boot and Spring Data for State Stores

Spring Data offers a consistent model for working with different databases, NoSQL, RDBMS, Hadoop, Caches and so on.
Spring Data JPA is a subset of Spring Data, offering (again) similar APIs to use NoSQL and RDBMS via Templates, common exceptions and the ability to create simple Repositories. Paging, CRUD, custom queries and sorting then work the same across all datastores.

## (Auto-) Configuration is Easy

It starts with adding the dependency, which transitively will bring AOP, JDBC, Hibernate (as the JPA provider), Transaction API and some other stuff.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

If JPA is on the classpath, Spring Boot autoconfigures:

* DataSource
* EntityManagerFactoryBean
* JpaTransactionManager

__Without Spring Boot__, this is what would have to be configured to make it work:

```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {

  HibernateJoaVendorAdapter adapter = new HibernateJoaVendorAdapter();
  adapter.setShowSql(true);
  adapter.setGenerateDdl(true);
  adapter.setDatabase(Database.HSQL);

  Properties props = new Properties();
  props.setProperty("hibernate.format_sql", "true");

  LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
  emfb.setDataSource(dataSource);
  emfb.setPackagesToScan("rewards.internal"); // where the JPA-annotated entity POJOs are
  emfb.setJpaProperties(props);
  emfb.setJpaVendorAdapter(adapter);

  return emfb;
}
```

and with Spring Boot, it's mostly properties:

```java
@SpringBootApplication
@EntityScan("rewards.internal") // if it's not in the base package
public class Application { .. }
```

```yaml
spring:
  jpa:
    database: default # let Hibernate detect it
    show-sql: true
    hibernate:
      ddl-auto: update # validate, update, create, create-drop
    properties:
      hibernate:
        format_sql: true
        foo: bar
```

## Creating Repositories

Setting up JPA repositories follows the same instructions all the time:

1. annotate domain pojos, turning them into entities
2. define repositories as interfaces (extending `CrudRepository<T, K>` or `Repository<T, K>`)

Spring Data will then at runtime implement this by scanning for those interfaces. All repositories are picked up via `@SpringBootApplication`, unless they are not in a sub-package. Then, you can use `@EnableJpaRepositories(basePackages="com.myapp.repos")`.


### Creating Domain Entities

You have your domain classes that you need to annotate accordingly. `@EntityScan("rewards.internal")` picks those up.
Annotations are specific to the database type, with this being for RDBMS:

```java
@Entity
@Table
public class Customer {
  @Id
  @GeneratedValue(strategy = GerationType.AUTO)
  private Long id;
  private Date orderDate;
  private String email;
}
```

This is MongoDB (NoSQL):

```java
@Document
public class Customer { .. }
```

This is GemFire (a distributed cache I think):

```java
@Region // map to a region
public class Account { .. }
```

and so on.

### `Repository`, `CrudRepository` and other kinds

Both kinds are typed: first is the entity stored, the second is the primary key.
`Repository` is a pure marker interface, where everything is done programmatically. 

```java
@Repository
public interface CustomerRepository extends Repository<Customer, Long> { .. }
```

`CrudRepository` brings CRUD, `count()`, `save()`, `find()`, `findAll` and other standard methods.

```java
@Repository
public interface CustomerRepository extends CrudRepository<Customer, Long> { .. }
```

But it get better: there is also `PagingAndSortingRepository` extending from `CrudRepository` offering `Iterable<T> findAll(Sort)` and `Page<T> findAll(Pageable)`.

```java
@Repository
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> { .. }
```

### Extending the Repository with Custom Methods

The method names are converted into queries. The grammar is as such:

* `find(First)By<MemberOfEntity><Op>`, e.g. `findFirstByNameIgnoreCase`
* is there something similar for commands? not sure

Whenever the grammar gets too unwieldy, you can add `@Query(some_query)` and use any method name.

```java
public interface CustomerRepository extends CrudRepository<Customer, Long> {
  public Customer findFirstByEmail(String email);
  public List<Customer> findByOrderDateLessThan(Date someDate);
  public List<Customer> findByOrderDateBetween(Date d1, Date d2);

  @Query("SELECT c FROM Customer c WHERE c.emailAddress = ?1) // ?1 maps to first parameter
  Customer findByEmail(String email);

  // this is the query language of the underlying db; here it's JPQL/"Java Persistence Query Language"
  @Query("SELECT c FROM Customer c WHERE c.email NOT LIKE '%@%'")
  public List<Customer> findInvalidEmails();
}
```





