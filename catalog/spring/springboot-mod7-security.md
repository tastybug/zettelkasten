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

It starts with adding a starter dependency:
```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

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
  // see below for more details
  PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatinPasswordEncoder();

  UserDetails user = User.withUsername("user").password(passwordEncoder.encode("user")).roles("USER").build();
  UserDetails admin = User.withUsername("admin").password(passwordEncoder.encode("admin")).roles("ADMIN").build();

  return new InMemoryUserDetailsManager(user, admin);
}
```

Database lookup is also possible. This will run `SELECT username, password, enabled FROM users WHERE username = ?` unless configured otherwise (check documentation for group support):

``` java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
  return new JdbcUserDetailsManager(dataSource);
}
```

### AuthN in MockMVC

MockMVC takes the real SecurityFilterChain into account, because you want to test that your security rules work as expected. But MockMVC takes the burden of Authentication away by populating the security context with `Authentication` objects as needed. That's why you don't have to make a `Authorization` header part of your request.

You can set a username and password, if that's somehow relevant for the application code, e.g. because you want to log the principal's name or there is method security. The `roles` attribute is critical for the security filter chain (b/c roles on URLs).
```java
@Test
@WithMockUser(username = "user", password = "user", roles = {"USER"})
public void accountDetails_with_user_credentials_should_return_200() throws Exception {
```

#### Custom UserDetailsService

If you have a custom `UserDetailsService` (those return UserDetails by name) that you want to rely on, you can do this:

```java
@Component
public class CustomUserDetailsService implements UserDetailsService {
  @Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		User.UserBuilder builder = User.builder();
     builder.username(username);
     builder.password(passwordEncoder.encode(username));
     builder.roles("USER");
     return builder.build();
	}
}
```

And in your MockMVC test:
```java
@Test
@WithUserDetails("joe")
public void accountDetails_accessible() throws Exception {
  // ..
}
```



### On Password Hashing

Passwords are supposed to be hashed instead of stored in plain text. Hashing algorithms sometimes get deprecated and replaced and this will be true for the future: your service will use a hashing alg that will be deprecated at some point.
Spring Security supports having different hashing strategies in place at the same time, so that your old old admin account can still log in after years.

The `PasswordEncoder` produces a string containing the hash, the salt, the alg and maybe more depending on the alg: `(bcrypt) lasdjkhasjdajksdhakjshdkajhdkjashdkjahsd`. This will be stored and compared against with the provided password. When the `PasswordEncoder` checks the target hash, it knows how to hash the given password.

By default, `bcrypt` is used:
```java
PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatinPasswordEncoder();
passwordEncoder.encode("user")
```

### Adding AuthN to the `SecurityFilterChain`

For basic auth:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
  // configure the chain here
  http.authorizeHttpRequests((authz) -> authz
        .requestMatchers( ... )
        .anyRequest().authenticated());
      .httpBasic(withDefaults()); // enables basic auth
  return http.build();
}
```

For form login:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
  // configure the chain here
  http.authorizeHttpRequests((authz) -> authz
        .requestMatchers( ... )
        .anyRequest().authenticated());
      .formLogin(form -> form
        .loginPage("/login")
        .permitAll()
      )
      .logout(logout -> logout
        .logoutSuccessUrl("/home") // default: /login?logout
        .permitAll()
      );
  return http.build();
}
```

A form login page:

```html
<form action="/login" method="POST">
  <input type="text" name="username"/>
  <br/>
  <input type="password" name="password"/>
  <br/>
  <input type="submit" name="submit" value="LOGIN"/>  
</form>
```

## Method Security

Security is not necessarily limited to endpoints and URL, it can span to methods inside the application. Best practice is to enforce authZ on the SERVICE layer, not below, not above. During Code Reviews, make sure that programmatic access does not bypass services.
Spring Security uses AOP for this and annotations can be either Spring's own or JSR-250.

It starts with:
```java
@EnableMethodSecurity
```
Example:
```java
public class ItemManager {
  // members can access their own order items here
  @PreAuthorize("hasRole('MEMBER') && #order.owner.name == principal.username")
  public Item findItem(Order order, login itemNumber) {
    // ..
  }
}
```



