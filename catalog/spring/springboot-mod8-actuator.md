# Actuator: Making Health Visible

Actuator is a SB module that helps with observability. As per usual, it starts with this:

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
# 
management:
 endpoints:
   web:
     exposure:
       # by default, only health is exposed
       include:health,beans,env,info
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













