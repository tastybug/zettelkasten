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

