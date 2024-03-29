# Component Scanning

Component scan is the mechanism that looks for all Beans that are not explicitly covered by Configuration classes. It looks for _implicit_ configuration, which are classes annotated as `@Service`, `@Component`, `@Repository`, `@Controller`, `@RestController` and `@Configuration` which are the so called "Stereotype Annotations".

Component scans are recursive and span the whole classpath. When configured too broadly (`@ComponentScan("org")`), the startup time is impacted. This is how it's done explicitly:

```java
@Component
@ComponentScan("com.bank")
public class SomeConfig {
}
```

## Best Practice: Feature Packages with local Component Scans

Treating an application as a bunch of pluggable features can be done. Imagine a package structure as such:

```
com.application
com.application.restport
com.application.kafkaport
```

In this structure, `restport` and `kafkaport` can contain "anchor" classes at the top level, e.g. `com.application.restport.SomeRestService` and `com.application.kafkaport.SomeKafkaService`. Now you can plug them together declaratively, leaving out and pulling in whatever needed:

```java
@Configuration
@ComponentScan(basePackageClasses = {SomeKafkaService.class, SomeRestService.class})
public AppConfig {

}
```

## Autowiring Beans

There are 3 kinds of injections of Beans: field, constructor and constructor autowiring.

```java
@Component
public OuterClass {

 // via field which can even be private, but this makes testing harder
 @Autowired
 private AccountRepo accountRepo;

 // via constructor
 @Autowired // optional if its the only constructor
 public OuterClass(AccountRepo accountRepo) {
  this.accountRepo = accountRepo;
 }

 @Autowired // works just like the constructor approach
 public void setAccountRepo(AccountRepo repo) {
  this.accountRepo = repo;
 } 
```

## Autowiring Values

You can also wire in configuration values via `@Value`.

```java
@Component
public OuterClass {

 @Value("${user.region"})
 private String userRegion;

 public OuterClass(@Value("${user.region"} String userRegion) {
  this.userRegion = userRegion;
 }

 @Autowired // <- don't forget this
 public void setUserRegion(@Value("${user.region"} String userRegion) {
  this.userRegion = userRegion;
 } 
```

## Lazy Initialization

A Bean can be declared as lazy initialzed with `@Lazy(true)`, which menas it will be created on first use.
A lazy Bean is preventing the application from failing fast on problems though, so this should be avoided.

## @PostConstruct and @PreDestroy

For Beans that are part of the application, you can annotate Bean methods as such and the Spring Context will make sure to at least call `@PostConstruct` to pre-heat caches, log state and such. `@PreDestroy` otoh will be called too, but this is not guaranteed if the process dies too quickly.

For library code and such, there is another approach:

```java
@Configuration
public CacheConfig {

 @Bean(initMethod="populateCache", destroyMethod="flushCache")
 public ThirdPartyCache thirdPartyCache() {
   //
 }

 public void populateCache() {
  //
 }

 public void flushCache() {
  //
 }
}
```

These methods must exist in the Configuration class
