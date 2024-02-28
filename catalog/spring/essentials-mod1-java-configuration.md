# Spring Application Context

The Spring Application Context represents the Spring DI container, the thing that owns the lifecycle of Beans.
The Context can be created everywhere, whether it's applications or unit tests.

To create a Context explicitly, this is all that's needed:

```
ApplicationContext context = SpringApplication.run(ApplicationConfig.class);

// get a bean now
var service = context.getBean(SomeService.class)

@Test
void someTest() {

	var result = service.someMethod();

	assertEquals(result, "expectation");
}
```

## Handling multiple configurations and bringing them together

Large applications contain many beans, different layers or different logical modules. It's easy to have different configurations:

```
@Configuration
@Import({ApplicationConfig.class, WebConfig.class})
public class TestInfraConfig {
// test DBs and stuff
}

@Configuration
public class ApplicationConfig {
// app stuff
}

@Configuration
public class WebConfig {
// web stuff
}


// and now we can do:
ApplicationContext ctx = SpringApplication.run(TestInfraConfig.class);

```

## Bean Scopes

By default, every bean is a singleton. You can change this by setting this:

```
@Bean
@Scope("singleton") // or
@Scope("prototype") // or
@Scope("session") // or
@Scope("request")
public SomeService someService() {
 //
}
```

* `prototype`: every time you fetch the bean from the context, you get a new one.
* `singleton`: it's the same object on each fetch
* `session` for web environments: one per user session
* `request` for web environment: one per request

There are more specialized scopes (web socket scope, refresh scope, thread scope) and it's even possible to have a custom scope (e.g. instance per tenant), but this is rare.