# Actuator: Making Health Visible

Actuator is a SB module that helps with observability. As per usual, it starts with the starter dependency which brings with it `micrometer` and other transitive deps:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

At a basic level, it will a number of endpoints, but not all are exposed by default:

* `/actuator/info`: you can put anything here and by default, this is empty. You could return build time, version, name and so on.
* `/actuator/health`: by default, this just returns `{ "status": "UP" } as a composite status across database connectivity etc.
* `/actuator/metrics`: a list of generic and custom metric names provided by the application
  * `/actuator/metrics/<metric-name>`: returns then the content of a metric
* `/actuator/beans` all beans in the context
* `/actuator/env` all properties in `Environment`
* `/actuator/configprops` collated list of all `@ConfigurationProperties`
* `/actuator/loggers` query and modify logging levels
* `/actuator/mappings` MVC request mappings
* `/actuator/conditions` conditions used by auto-configuration
* `/actuator/shutdown` shuts down the application
* ...

There is a difference between being enabled and being exposed. Enabled means, the endpoint has been created and the actuator bean is in the context. By default, all endpoints are enabled except for `/actuator/shutdown`.
Exposed otoh means that you can reach the endpoint via HTTP or JMX. JMX exposure is only given when `spring.jmx.enabled=true` is set. HTTP is only available when Spring MVC, WebFlux or Jersey is used. In addition to this, you need to declare which endpoints you want to be exposed (by default, only `health` is exposed):

```yaml
# by default, only health is exposed
management.endpoints.web.exposure.include:health,beans,env,info
```

## Securing Actuator Endpoints with Spring Security

Someone might have changed the actuator base path, so hard coding paths here is dangerous. This is a base-path agnostic way of setting security up:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
 http.authorizeHttpRequests((authz) -> authz
   .requestMatchers(EndpointRequest.to(HealthEndpoint.class)).permitAll()
   .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ACTUATOR")
   .anyRequest().authenticated();
 return http.build();
}
```

## Setting Up Custom Metrics

The library of choice to support this micrometer, which is offering Counters, Gauges, Timers and DistributionSummary in a vendor neutral way. A metric is to be registered with the auto-configured `MeterRegistry`. Custom metric names will show up in `/actuator/metrics` and can be accessed via `/actuator/metrics/my-metric`.

Micrometer uses dimensional metrics, where you can add K/V pairs, like `method:GET` and `status:200`. These metrics can thus have "high dimensionality" and even "high cardinality", if keys have many distinct values.

Here are two ways to track the execution time: explicitly and implicitliy via `@Timed("metric-name")`.
```java
@RestController
public class OrderController {

  private Timer timer;

  public class OrderController(MeterRegistry mr) {
    this.timer = mr.timer("order.submit");
  }

  @PostMapping("/orders")
  public Order placeOrder( ... ) {
    # explicit timer measurement
    return timer.record( () -> { /* logic here */ } );
  }

  @GetMapping
  @Timed("orders.summary") // provides count, mean, max, total
  public List<Order> orderSummary() { .. }
}
```

Using a builder to create a metric:
```java
@RestController
public class RewardController {
  private final DistributionSummary summary;

  public RewardController(MeterRegistry mr) {
    summary = DistributionSummary.builder("reward.summary")
                                  .baseUnit("dollars")
                                  .register(mr);
  }

  @PostMapping("/rewards")
  public ResponseEntity<Void> create(@RequestBody Reward reward) {
    // DS provides: count, total, max for it's metric
    summary.record(reward.amount());
    /* more logic here */
  }
}
```

and this is how the metric looks in the endpoint:

```json
{
  "name": "reward.summary",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 3
    },
    {
      "statistic": "TOTAL",
      "value": 10
    },
    ... 
  ]
}
```

## Health Indicators

`/actuator/health` returns a composite status indicating the health of the application. The underlying metrics are auto-configured and cover things like DB, diskspace, JMS, Mail and others. If you set `management.endpoint.health.show-details=always`, you get a detailed breakdown of all metrics that went into the health status.

Whenever the endpoint is accessed, the metric values are computed, which can be costly. Or maybe you want to avoid taking certain metrics into account for the health endpoint, because you want to avoid the container scheduler from killing your application _based on certain metrics that should not be relevant for that decision_.
To facilitate that, you can setup groups. A list of all groups can be seen under `/actuator/health`

```
# group some health indicators that k8s should take into account
management.endpoint.health.group.myk8shealth.include=diskSpace,db

# for manual access, I want to see db only
management.endpoint.health.group.web.include=db
management.endpoint.health.group.myk8shealth.show-details=always
```

Now you can access the group endpoint under `/actuator/health/myk8shealth` and `/actuator/health/web`

### Custom Health Indicators

It's easy to set up a customer health indicator. You have 2 options:

* have a class extend `AbstractHealthIndicctor` and override `doHealthCheck()`, where you return a `Health` object
* have a class implement `HealthIndicator`, override the `health()` method and return a `Health` object

```java
@Component
public class CustomHealthCheck implements HealthIndicator {
  @Override
  public Health health() {
    return Health.down().withDetail("foo", "bar").build();
  }
}
```

will lead to this response from the health endpoint:

```json
{
  "status": "DOWN",
  "details": {
    "customHealthCheck": {
      "status": "DOWN",
      "details": {
        "foo": "bar"
      }
    },
    "db": {
      "status": "UP",
      ...
    },
    ...
  }  
}
```




