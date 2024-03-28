# Spring Security

Some basic concepts:

* Principal: the actor performing an action - could be a user or device
* Authentication: identifying a principal by checking the credentials
* Authorization: deciding what a principal can do
* Authority: a permission that enables access - such as a role
* Secured Resource: the resource to which the access is subject to authorization

Spring security is usable in any Spring probject. It separates business logic from security concerns. AuthN and authZ are decoupled as such that changes to either does not impact the other.

AuthN can be done with Basic, Form, X.509, OAuth, Cookies, SSO and others schemes. It can be stored in LDAP, DB, properties, DAOs.

Spring security uses a chain of filters ("SecurityFilterChain") to perform authN and authZ as well as managing the security session. The result of this is stored in thread local in the "security context", from which Principal and Authorities (Roles) are then taken.

1) `SecurityContextPersistenceFilter`: establishes the security context and maintains that between HTTP requests
2) `LogoutFilter`: clears security context on logout
3) `UsernamePasswordAuthenicationFilter`: put authN into the security context on login
4) `ExceptionTranslationFilter`: converts spring security exceptions into HTTP responses or redirects
5) `AuthorizationFilter`: handles authZ on web requests based on configured attributes and authorities (or "the rules")

## Default Security Setup

By default, a single user called "user" is set up. A password is generated for "user" and logged to stdout during boot. All URLs required a logged in user. Logging in is either done with BasicAuth or FormLogin, depending on the content negotiation.

## URL AuthZ

When setting up rules around URLs, the path matching rules are that it uses Spring MVC matching rules where available, otherwise "Ant style" matching is used:

* `/admin/*` matches `/admin/xyz`
* `/admin/**` matches any path under admin, like `/admin/foo/bar`

Rules are evaluated in order of declaration and the first that matches, wins. This is why you start with specific rules first and then mention generic rules.

Some URLs don't have to be secure. You can either work this into the `SecurityFilterChain` or set up a separate bean, the `WebSecurityCustomizer` to disable rules for select paths.

```java
@Configuration
public class SecurityConfig {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    // configure the chain here
    http.authorizeHttpRequests((authz) -> 
      authz
        .requestMatchers("/signup", "/about").permitAll()
        .requestMatchers(HttpMethod.PUT, "/accounts/edit").hasRole("ADMIN")
        .requestMatchers("/admin/**").hasAnyRole("ADMIN", "USER")
        .anyRequest().authenticated()); // if no other rule matches, do this
    return http.build();
  }

  @Bean
  public WebSecurityCustomizer webSecurityCustomizer() {
    // images and PR material is free for all!
    return (web) -> web
        .ignoring()
        .requestMatchers("/images/*", "/public-relations/*");
  }

  @Bean
  public InMemoryUserDetailsService userDetailsService() {
    // configure the AuthenticationManager
  }
}
```

## AuthN: Basic Auth

As a web request comes in, it passes `BasicAuthenticationFilter`. If `Authorization: Basic <base64 of user:password>` is present, the filter places an `Authentication` object in the security context - the authentication object is _not_ authenticated yet. Then an `AuthenticationManager` will check the `Authorization` object by delegating to a `AuthenticationProvider` (e.g. `OpenIDAuthenticationProvider`), ultimately finishing the AuthN. With this being done, the Authorities of the Principal are clear and are added to the `Authentication` object.
At this point, authN is done and we have a `Authentication` object in the security context having:
* `Authenticated: true/false`
* `Authorities: list of roles`

There are a number of out-of-the-box `AuthenticationProviders`:

* `DaoAuthenticationProvider`
* `LdapAuthenticationProvider`
* `OpenIDAuthenticationProvider`
* `RememberMeAuthenticationProvider`
* etc

The `DaoAuthenticationProvider` takes data from a `UserDetailService` bean, of which there are:
* `InMemoryUserDetailsManager`
* `JdbcUserDetailsManager`
* `LdapUserDetailsManager`

So as soon as a `UserDetailsManager` bean is declared, it will be auto-configure into a `DaoAuthenticationManager` and used.

In memory for testing purposes:
```java
@Bean
public InMemoryUserDetailsManager userDetailsService() {
  UserDetails user = User.withUsername("user").password(passwordEncoder.encode("user")).roles("USER").build();
  UserDetails admin = User.withUsername("admin").password(passwordEncoder.encode("admin")).roles("ADMIN").build();

  return new InMemoryUserDetailsManager(user, admin);
}
```

Database lookup is also possible. This will run `SELECT username, password, enabled FROM users WHERE username = ?` (check documentation for group support):

``` java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
  return new JdbcUserDetailsManager(dataSource);
}
```



