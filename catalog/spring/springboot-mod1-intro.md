# What and Why Spring Boot?

SB is an opinionated bill of materials for different use cases (web, batch, observability, ..). The configured set of libraries is known to work well together. It does not generate code, it's not a plugin.

It integrates into Maven (or Gradle) like this: 

1. You point to a spring boot parent pom, which sets the spring framework version, includes some Maven plugins and sets the Java version.
2. You add further features by adding spring boot starter packages.

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <!-- this sets ${spring-framework.version} to 5.3.23 -->
  <version>2.7.5</version> 
</parent>


<dependencies>

  <!-- this pulls in 18 Jars for AOP, SLF4J, beans, .. -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <!-- no version, this is defined in parent -->
  </dependency>

  <!-- this pulls in spring-test-*.jar, junit, mockito, assertJ, .. -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <!-- no version, this is defined in parent -->
    <scope>test</scope>
  </dependency>

</dependencies>
```

## Spring Boot (Auto-) Configuration

Spring Boot automatically creates beans for the things you need. If it finds a particular dependency on the class path, preconfigured beans will be served, e.g. for HSQLDB. This is enabled with `@EnableAutoConfiguration`, which you rarely use directly though.

Doing it the complicate way:
```java
@SpringBootConfiguration // this is an extension of @Configuration
@ComponentScan
@EnableAutoConfiguration  
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

Can be shortened to:

```java
@SpringBootApplication(scanBasePackages="com.app") // if parameter is omitted, it starts from this class
public class Application {
  //
}
```

## Packaging

SB applications are usually served as an executable fat jar, which contains all dependencies including Tomcat. This can be executed directly with `java -jar fat.jar`. You create the far jar using the Maven or Gradle lifecycle commands (`gradle assemble` or `maven package`) and on top of this you can also start the app with `maven spring-boot:run`. This is the plugin that has been pulled in via the spring boot parent:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

When running `package`, 2 jars will be produced: the executable fat `my-app.jar` and the non-fat `my-app.jar.original` containing only the app classes.

## Integration Testing

In plain Spring, we use `@SpingJUnitConfig`, which we will now replace with `@SpringBootTest` going forward. When a test class is annotated as such, there is a search for other classes annotated with `SpringBootApplication` or, alternatively, you can point to the target class directly.

A test class pointing to an application:
```java
@SpringBootTest(classes=Application.class) // parameter optional, it'll search on its own
public class TransferServiceTests {
  @Test
  void testTransfer() {}
}

@SpringBootApplication(scanBasePackages="com.app")
public class Application {
  //
}
```

## A Smallest Spring Boot Project
Assuming a project called `myapp`.

`pom.xml`:
```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.7.5</version> 
</parent>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

`application.yaml`:
```yaml
logging:
  level:
    root: "ERROR"

spring:
  sql:
    init:
      schema-locations: "classpath:rewards/schema.sql"
      data-locations: "classpath:rewards/data.sql"
```

`Application.class`, which is a configuration class:
```java
@SpringBootApplication
public class Application {

  public static final String QUERY = "SELECT COUNT(*) FROM t_account";

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

  @Bean
  CommandLineRunner commandLineRunner(JdbcTemplate jdbcTemplate) {
    return args->System.out.println("There are " + jdbcTemplate.queryForObject(QUERY, Long.class) + " accounts.");
  }
}
```

DB setup classes:

`schema.sql`:
```sql
drop table T_ACCOUNT if exists;
create table T_ACCOUNT (ID integer identity primary key, NUMBER varchar(9), NAME varchar(50) not null, unique(NUMBER));
```

`data.sql`:
```sql
insert into T_ACCOUNT (NUMBER, NAME) values ('123456789', 'John and Jane Doe');
```


Now let's run this:

```shell
mvn package
java -jar myapp-0.0.1-SNAPSHOT.jar
> Hello, there are 1 accounts
```



