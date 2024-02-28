# External Properties, Profiles and SpEL

## External Properties and the Environment

Hard coded values are bad. Get this stuff from the Environment, be it files, environment variables and such.

`Environment` represents the loaded properties from the runtime environment. It gets the values from different sources in a particular order (this list is simplified):

1. JVM System Properties (`System.getProperty()`); this is the stuff that comes in via `java -jar jarName -Dproperty=value`.
2. System Environment Variables (`System.getenv()`); which is POSIX process environment variables.
3. Properties files


```
@Bean
public DataSource dataSource(Environment env) {
  String dbDriver = env.getProperty("db.driver");
  // ..
}

```

### Where does Environment get its data from?

For property file contents, how does the `Environment` get the data? There are conventions on where it looks, but one can also be explicit:

```
@Configuration
@PropertySource("classpath:/com/org/config/app.properties")
@PropertySource("file:config/local.properties")
@PropertySource("http://bla.com/config.properties")
public ApplicationConfig {
 
}
```

Conventions that I'm aware of: it looks for `$PROFILENAME.properties` and `$PROFILENAME.yaml` at the root of the classpath and ingests that.

### Bypassing Environment by using @Value

Usually you don't interact with `Environment` and inject values directly:

```
@Value("${db.driver:someDefault}")
private String someValue;
```

This will fail if the value is missing.

## Spring Profiles

Profiles deal with separating beans for different cases, usually by stage (dev, stage, prod).
By putting a profile on a `@Configuration` or `@Bean`, you assign it to a profile.

Case 1:
```
@Configuration
@Profile("embedded")
public class DevConfig {
 // some in-memory db
}
```

Case 2:

```
@Configuration
public InfraConfig {

 @Bean
 @Profile("embedded")
 public DataSource dataSourceDev() {
   // hsqldb
 }

 @Bean
 @Profile("!embedded") 
 public DataSource dataSourceProd() {
   // mysql
 }

}
```

### Activating a Profile

For runtime purposes, you have to set them before creating the context:  Either using `-Dspring.profiles.active=embedded,jpa` or as such:

```
System.setProperty("spring.profiles.active", "embedded,jpa");
SpringApplication.run(AppConfig.class);

```

In Integration Tests situtations you can use `@ActiveProfiles(...)`. See later module for details.

## SpEL: Spring Expression Language usage for configuration purposes

`#{}` vs. `${}`.

Examples:

* `@Value("${user.region}") private String region;` 
* `@Value("#{environment['user.region']}") private String region;`
* `@Value("#{systemProperties['user.region']}") private String region;` 
* `@Value("#{regionStrategyBean.getRegion}") private String region;` (regionStrategyBean must be a valid bean in the context)

Available default references are `environment`, `systemProperties` and `systemEnvironment`. SpEL allows to create custom functions and references, which is heavily used by Spring Security and other Spring modules.
