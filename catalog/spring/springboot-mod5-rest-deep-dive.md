# RESTful Applications with Spring Boot

A bit of background; Jakarta EE is a set of specifications for building cloud native Java applications. JAX-RS is one of those specifications, of which there are multiple implementations: Jersey, RESTEasy and Apache CXF (maybe more). All of these have Spring support. JAX-RS is a set of interfaces and annotations.

However, Spring MVC also offers REST support, but it's not JAX-RS compliant because both appeared at the same time, developed independently.

## Server Side Controller Endpoints: `GET`, `POST`, `PUT`, `DELETE`

### GET endpoints (see also previous module)

We already saw `@GetMapping("/api/path")`. This is syntactic sugar for `@RequestMapping(path = "/api/path", method = RequestMethod.GET)`. Note that for `HEAD`, `OPTIONS` and `TRACE`, there is no sugar and you _must_ use `@RequestMapping`.

See previous module for more examples.

```java
@GetMapping("/accounts/{accountId}")
public Account find(@PathVariable("accountId") long accountId) { .. }
```

### PUT endpoint

PUT endpoints do no return data, they are void methods. Here, we utilize `@ResponseStatus(HttpStatus.NO_CONTENT)` to indicate the status code.
Request bodies are handed in via `@RequestBody` - content negotiation and de-/serialization is again handled transparently based on the `Content-Type` sent by the client.


```java
@PutMapping("/store/orders/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void updateOrder(@PathVariable("id") long orderId,
                        @RequestBody Order orderUpdate) { .. }
```

### POST endpoint

With a POST request, the expectation is to return 201 and the `Location` header pointing to the newly created resource.

The basics:

```java
@PostMapping("/orders/{id}/items")
public ResponseEntity<Void> createItemInOrder(@PathVariable("id") long orderId,
                                              @RequestBody Item item) {

  orderService.addItemToOrder(orderId, item);

  return ResponseEntity
          .created(uri) //<- this expects the URI of the newly created resource
          .build();
}
```

How do you come up with the URI for the `Location` header? 2 options:

* use `UriComponentsBuilder`, allowing explicit (hard coded!) URI creation
* use `ServletUriComponentsBuilder`, which knows about the URL that invoked the current controller. We seem to have a winner here.

```java
// we must be in a controller method, e.g.
// POST http://server/store/orders/12345/items

URI location = ServletUriComponentsBuilder
.fromCurrentRequestUri()                   // framework stores this in the thread context
.path("/{itemId}")                         // leading slash!
.buildAndExpand("item A")                  // dumps URL encoded "item A" in "itemId"

// result: http://server/store/orders/12345/items/item%20A

return ResponseEntity.created(location).build();
```

### Delete endpoint

DELETE endpoints return 204 and an empty response.

```java
@DeleteMapping("/store/orders/{orderId}/items/{itemId}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteItemFromOrder(@PathVariable("orderId") long orderId,
                                @PathVariable("itemId") long itemId) {
  // ..
}
```

## Client Side Requests with RestTemplate

Creating a `RestTemplate` is simple: `RestTemplate template = new RestTemplate();`.

In Spring Boot, a auto-configured builder bean is available. Application properties such as those for Jackson are taken into account:

```java
@Autowired
public MyClass(RestTemplateBuilder builder) {
  this.restTemplate = builder.build();
}
```

If you want to hide the HTTP-ness as much as possible, go with these methods:

* `DELETE`: `delete(String url, Object... urlVariables)`
* `GET`: `getForObject(String url, Class<T> responseType, Object... urlVariables)`
* `HEAD`: `headForHeaders(String url, Object... urlVariables)`
* `OPTIONS`: `optionsForAllow(String url, Object... urlVariables)`
* `POST`:
  * `postForLocation(String url, Object request, Object... urlVariables)`
  * `postForObject(String url, Object request, Class<T> responseType, Object... urlVariables)`
* `PUT`: `put(String url, Object request, Object... urlVariables)`
* `PATCH`: `patchForObject(String url, Object request, Class<T> responseType, Object... urlVariables)`

A mildly complex example
```java
RestTemplate template = new RestTemplate();
String uri = "http://example.com/store/orders/{id}/items";

// get all items for an order 1
OrderItem[] items = template.getForObject(uri, OrderItem[].class, "1");

// POST a new item in order 1 and get back the location
OrderItem item = createItem(..);
URI itemLocation = template.postForLocation(uri, item, "1");

// PUT an update
item.setAmount(5);
template.put(itemLocation, item);

// DELETE the item
template.delete(itemLocation);
```

### More Controle `RequestEntity` and `ResponseEntity`

Sometimes you want access to request and response headers and body. More exposure to HTTP!

Request example:

```java
RequestEntity<OrderItem> request
  = RequestEntity
    .post(new URI(itemUrl))
    .getHeaders().add(HttpHeaders.AUTHORIZATION, "Basic " + getToken())
    .contentType(MediaType.APPLICATION_JSON)
    .body(newItem);

ResponseEntity<Void> response = template.exchange(request, Void.class);

assertEquals(response.getStatusCode(), HttpStatus.CREATED);
```

Response example:

```java
String uri = "http://example.com/store/otders/{id};

ResponseEntity<Order> response = template.getForEntity(uri, Order.class, "1");
assertEquals(response.getStatusCode(), HttpStatus.OK);

long modified = response.getHeaders().getLastModified();

Order order = response.getBody();
```

### Basic Mapping of Exceptions to Response Codes

On the server side, possible error cases need to be mapped to response codes. The way to go are `@ExceptionHandler` methods (or classes).

In a controller class:
```java
@ResponseStatus(HttpStatus.CONFLICT)
@ExceptionHandler({ DataIntegrityViolationException.class })
public void handleConflict(Exception ex) {
  logger.error("Exception is: ", ex);
  // just return empty 409
}
```

Alternatively, you can have centralized Advice classes instead of scattered "method handlers". No need to bind this to controllers - this Advice is active everywhere.

```java
@ControllerAdvice
public class ConflictControllerAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = {DataIntegrityViolationException.class})
    protected ResponseEntity<Object> handleConflict(DataIntegrityViolationException ex, WebRequest request) {
        return ResponseEntity.status(HttpStatus.CONFLICT.value())
                .build();
    }
}
```

### Isn't `RestTemplate` Deprecated? What About `WebClient`?

`RestTemplate` is mature and as of Spring 5, it is deprecated. Modern use cases like streaming are not supported and it is recommended to use the non-blocking reactive `WebClient` for those cases.





