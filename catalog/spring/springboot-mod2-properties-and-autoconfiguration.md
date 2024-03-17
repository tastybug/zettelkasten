# Properties and More On Auto Configuration

Options for defining and loading properties and overriding autoconfiguration.


## Externalized Properties

In Spring, we commonly externalize properties to files and then consume them with `@PropertySource`, but the location is not standardized.
Spring Boot looks for `application.properties` and makes that available via `@Value` and `Environment` by automatically creating a `@PropertySource` for that. 

### Supported File Locations

The following locations are taken into account:

* in a `/config` package in the classpath
* in a `/config` directory in the working directory
* in the working directory
* at the root of the classpath

### Profile

Spring Boot looks for `application-${PROFILE}.properties` file, but still loads `application.properties`. The former has a higher precedence.

### Formats

* YAML: if `spring-boot-starter` is in the classpath - this brings snakeyaml.jar with it which adds the YAML support; indents must be 2 spaces (do not use tabs as it's not supported by YAML)
* Properties

Note that YAML supports multiple profiles in the same file:

```yaml
# always loaded!
spring.datasource:
  driver: org.postgresql.Driver

---
spring:
  profile: local
  datasource:
    url: jdbc:postgresql://localhost/somedb

---
spring:
  profile: prod
  datasource:
    url: jdbc:postgresql://prodserver/somedb
```

### Syntactic Sugar in Property Loading

Loading properties can be cumbersome if you have to repeat prefixes often:

```java
@Configuration
public class ClientConfiguration {
  @Value(${rewards.client.host}) String host;
  @Value(${rewards.client.port}) int port;
  @Value(${rewards.client.user}) String user;
  @Value(${rewards.client.timeout}) int timeout;
}
```

You can also use `@ConfigurationProperties` to load a subdocument with less typing. This mechanism is AOP based and needs to be enabled either by: `@EnableConfigurationProperties(ClientConfiguration.class)` on the application class turns the prop class into a managed bean:

```java
@ConfigurationProperties(prefix="rewards.client")
public class ClientConfiguration {
  private String host;
  private int port
  private String user;
  private int timeout;

  // there need to be setters!
}

@SpringBootApplication
@EnableConfigurationProperties(ClientConfiguration.class)
public class MyApp { ... }
```

**OR BY HAVING**: `ConfigurationPropertiesScan` on the application class turns all prop classes into a bean:

```java

@ConfigurationProperties(prefix="rewards.client")
public class ClientConfiguration {
  private String host;
  private int port
  private String user;
  private int timeout;

  // there need to be setters!
}

@SpringBootApplication
@ConfigurationPropertiesScan
public class MyApp { ... }
```

**OR BY HAVING** `@Component` on the properties class itself does the trick too:

```java
@Component
@ConfigurationProperties(prefix="rewards.client")
public class ClientConfiguration {
  private String host;
  private int port
  private String user;
  private int timeout;

  // there need to be setters!
}
```

## More on Autoconfiguration

Autoconfiguration is enabled with `@EnableAutoConfiguration` or indirectly with `@SpringBootApplication`. Autoconfiguration means that a range of beans are automatically configured for you - ready to be used. We use this feature extensively whenever we add properties to `application.yaml`, e.g. for Kafka consumers. If a broker is configured according to the convention, autoconfiguration does 

### How it Works
Technically, it's "just" a number of upstream configuration classes shipped with the spring boot dependencies. A list of all these configs can be found in file [spring-boot-autoconfigure.jar/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports). These configurations are processed AFTER the custom configurations done by the user, so custom beans always take precedence. The package `org.springframework.boot.autoconfigure` in `spring-boot-autoconfigure.jar` contains  the autoconfiguration implementations.

These configurations are heavily conditional. They kick in when beans are already or not yet defined, certain dependencies are on the classpath, certain properties are set. Some of the conditions used are:
* `@ConditionalOnBean`: is a bean there?
* `@ConditionalOnMissingBean`: is a bean missing?
* `@ConditionalOnClass`: is there a particular class on the classpath?
* `@ConditionalOnMissingClass`: is a particular class _not_ on the classpath?
* `@ConditionalOnProperty`: is a property set?

Example of conditional bean creation as used in `JdbcTemplateAutoConfiguration.class`:
```java
// create jdbcTemplate bean only if datasource is available
@Bean
@ConditionalOnBean(DataSource.class)
public JdbcTemplate jdbcTemplate(DataSource ds) {
  return new JdbcTemplate(ds);
}
```

### Overriding Autoconfig

Autoconfig is set up so that overrides are easy to accomplish. Since it comes down to influencing the `@ConditionalXX` checks, you have 4 options:

* set expected properties
* explicitly define your own beans
* explicitly disable autoconfig
* change dependencies or their versions







