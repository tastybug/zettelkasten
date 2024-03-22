# RESTful Applications with Spring Boot

A bit of background; Jakarta EE is a set of specifications for building cloud native Java applications. JAX-RS is one of those specifications, of which there are multiple implementations: Jersey, RESTEasy and Apache CXF (maybe more). All of these have Spring support. JAX-RS is a set of interfaces and annotations.

However, Spring MVC also offers REST support, but it's not JAX-RS compliant because both appeared at the same time, developed independently.

## GET endpoints (see also previous module)

We already saw `@GetMapping("/api/path")`. This is syntactic sugar for `@RequestMapping(path = "/api/path", method = RequestMethod.GET)`. Note that for `HEAD`, `OPTIONS` and `TRACE`, there is no sugar and you _must_ use `@RequestMapping`.



