# Inside the Spring Container and the Bean Lifecycle

## Container Lifecycle Phases

The Spring container goes through 3 phases:

1. Initialization: Beans are created and depencency injection occurs
2. Usage: Beans are used during the application runtime
3. Destruction: Beans are released for GC

### Init Phase

This kicks off when `SpringApplication.run(AppConfig.class` is executed. Here, the Bean definitions (e.g. annotations) are loaded and then the Beans are created in the necessary order according to the configured dependency graph. All this is done in the `BeanFactory`, which happens to be the `ApplicationContext`.
This init phase comes with extension points that give you means to change things programmatically.


#### Extension Point 1: Changing Bean Configuration Before Bean Creation
There is an extension point in the `BeanFactory` that allows programmatic changes to the Bean definitions before the Bean creation happens. This is the `BeanFactoryPostProcessor` interface. This extension point is for example used by the `PropertySourcesPlaceholderConfigurer`, which looks up values for the `@Value("${foo}")` annotation. This behaviour could be overridden with a custom instance of `PropertySourcesPlaceholderConfigurer`.

This is how one can create a custom post processor:

```java
// notice that this is static!
@Bean
public static BeanFactoryPostProcessor deprecatedBeanWarner() {
  return new DeprecatedBeanWarner();
}

public class DeprecatedBeanWarner implements BeanFactoryPostProcessor {
  // example: log warnings when using classes marked as @Deprecated
  // this post processor already comes with Spring btw
}
```

#### Extension Point 2: Changing Beans Post-Creation

The `BeanPostProcessor` is an important extension point that can modify bean instances or even return other instances, replacing Bean types. It will run against every Bean. It can modify a Bean before and/or after Initialization (`BeforeInit` and `AfterInit`).

This is the `BeanPostProcessor` interface:

```java
public interface BeanPostProcessor {
  public Object postProcessBeforeInitialization(Object originalBean, String beanName);
  public Object postProcessAfterInitialization(Object originalBean, String beanName);
}
```
