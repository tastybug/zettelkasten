# Web Applications with Spring MVC and RESTful Applications

Spring MVC is a web framework following the MVC pattern. This can serve server side web rendering and REST. Spring Boot helps as usual with autoconfiguration.
This comes in 2 flavors (we concentrate on 1st):

1. classic Web Servlets based on Java EE Servlets with filters and listeners
2. newer Web Reactive (Webflux): non-blocking, efficient

The deliverable of such a project is either a Far JAR, which comes with a bundled web container (Tomcat, Netty, Undertow) or a WAR file that can be deployed to an external Servlet container. The embedded approach is more wide spread these days.

It all starts with the starter dependency, which transitively brings `-web`, `-webmvc`, `spring-boot-starter`, `jackson`, `tomcat-embedded` and more:

```xml
<dependency> 
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## Creating a Simple Controller

A Controller is called from a Dispatcher Servlet, which takes care of message conversion in- and outbound. Message converters are auto-configured if they are found on the classpath (Jackson for JSON/XML, JAXB for XML, GSON for JSON, ..).

![Dispatchers, Controllers and Message Converters](./dispatchers-and-controllers.jpg)

A controller is a POJO, annotated with the stereotype `@Controller`. You annotate handler methods with `@GetMapping`, `@PostMapping` and so on to have the Dispatcher Servlet call them.

```java
@Controller
public class AccountController

  // @ResponseBody marks the response as a REST response.
  // This disables the View handling subsystem (you don't get a HTML page there).
  @GetMapping("/accounts")
  public @ResponseBody List<Account> list() {..}
```

As a convenience annotation, `@RestController` exists. It marks all responses in a class as `@ResponseBody`:

```java
@RestController
public class AccountController {
  @GetMapping("/accounts")
  public List<Account> list() {..}
}
```

### Accessing the Request Body, Headers and other Parameters

Spring Web makes it very easy to get any kind of element of a request by simply mentioning them in the parameter of the method:

* `Principal` is the user
* `@RequestParam("accountId") int accountId` for query params: <http://host/account?accountId=1234>
* `HttpServletRequest` to get the whole request (this makes testing harder b/c you have to mock this type)
* `HttpSession` for what exactly?
* `Locale` for the locale
* `@RequestHeader("Authorization")` for header values
* ...

Example for <http://host/account?accountId=1234>:
```java
@GetMapping("/account")
public Account find(@RequestParam("accountId") long accountId) { .. }
```

With URL templating, you can access random path elements. Use `{bla}` placeholders and access them with `@PathVariable("bla")`.

Example for <http://host/account/1234>
```java
@GetMapping("/accounts/{accountId}")
public Account find(@PathVariable("accountId") long accountId) { .. }
```

> For `@RequestParam` and `@PathVariable` you can omit an explicit name if the parameter name matches the method parameter name

### A Complex Example
```java
@GetMapping("/orders/{orderId}/items/{itemId}")
public OrderItem item(
  @PathVariable long orderId,
  @PathVariable long itemId,
  @RequestParam(required=false) Boolean onlyIfAvailable, // null if not specified
  Locale locale,
  Principal user) {
  // ..
}
```

## Message Converters

If you take in a request, the caller provides an `Accept: mimetype` header to indicate the acceptable format. In the response, the host indicates the delivered format with `Content-Type: mimetype`. This is called "Content Negotiation" and this should not have an impact on the controller logic.
Message Converts (auto-configured by Spring Boot) take care of mapping the domain objects to these formats so that the developer does not have to do this programmatically.

Alternatively, you can build responses fully explicitly, providing headers and content using a fluent API with `ResponseEntity`.
```java
// with String response
ResponseEntity<String> response = ResponseEntity.ok().contentType(MediaType.TEXT_PLAIN).body("Hello!");

// or with domain object response
ResponseEntity<Account> response = ResponseEntity.ok().lastModified(order.lastModified()).body(accountObject);
```







