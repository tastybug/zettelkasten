# RESTful Applications with Spring Boot

A bit of background; Jakarta EE is a set of specifications for building cloud native Java applications. JAX-RS is one of those specifications, of which there are multiple implementations: Jersey, RESTEasy and Apache CXF (maybe more). All of these have Spring support. JAX-RS is a set of interfaces and annotations.

However, Spring MVC also offers REST support, but it's not JAX-RS compliant because both appeared at the same time, developed independently.

## Controller Endpoints: `GET`, `POST`, `PUT`, `DELETE`

### GET endpoints (see also previous module)

We already saw `@GetMapping("/api/path")`. This is syntactic sugar for `@RequestMapping(path = "/api/path", method = RequestMethod.GET)`. Note that for `HEAD`, `OPTIONS` and `TRACE`, there is no sugar and you _must_ use `@RequestMapping`.


### PUT endpoint

PUT endpoints do no return data, they are void methods. Here, we utilize `@ResponseStatus(HttpStatus.NO_CONTENT)` to indicate the status code.
Request bodies are handed in via `@RequestBody` - content negotiation and de-/serialization is again handled transparently based on the `Content-Type` sent by the client.


```java
@PutMapping("/store/orders/{id})
@ResponseStatus(HttpStatus.NO_CONTENT)
public void updateOrder(@PathVariable("id") long orderId,
                        @RequestBody Order orderUpdate) { .. }
```

### POST endpoint

With a POST request, the expectation is to return 201 and the `Location` header pointing to the newly created resource.

The basics:

```java
@PostMapping("/orders/{id}/items"
public ResponseEntity<Void> createItemInOrder(@PathVariable("id") long orderId,
                                              @RequestBody Item item) {

  orderService.addItemToOrder(orderId, item);

  return ResponseEntity
          .created(???) <- this expects the URI of the newly created resource
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


