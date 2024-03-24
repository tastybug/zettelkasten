# Spring Boot Testing Framework

The framework allows "slice" testing with slice being what port is in hexagonal architecture. A slice can be a web facade, the database (JPA slice) and others.

It starts with adding the dependency. This pulls in JUnit5, some annotations, AssertJ, Hamcrest, Mockitp, JSONAssert and others:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

## Integration Testing Example with `TestRestTemplate`

Here is a simple setup with `TestRestTemplate` in an integration test scenario that uses an embedded Servlet Container.
The `TestRestTemplate` is a bit different from regualar `RestTemplate` in that it can use paths (not only URLs) and non-200 responses from the service will not cause an exception to be thrown. That makes testing for actual status code easier.

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class AccountClientTest {

  @Autowired
  private TestRestTemplate restTemplate; // this will know the random port

  @Test
  piublic void addAndDeleteBeneficiary() {
    // given: a newly created benficiary
    String addPath = "/accounts/{accountId}/beneficiaries";
    URI newLocation = restTemplate.postForLocation(addPath, "David", "1");

    // when: GETting it
    Beneficiary beneficiary = restTemplate.getForObject(newLocation, Beneficiary.class);

    // then: the beneficiary is not null and has its name
    assertThat(beneficiary.getName()).isEqualTo("David");

    // given: a deleted beneficiary
    restTemplate.delete(newBeneficiaryLocation);

    // when: GETting the deleted beneficiary
    ResponseEntity<Beneficiary> response = restTemplate.getForEntity(newLocation, Beneficiary.class);

    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
  }
}
```

## Web Slice Testing: Unit Testing Controllers with Spring MockMVC

Spring MVC Testing is part of the Spring Framework, found in `spring-test.jar`.

MockMVC is the unit testing for controllers: it checks that parameters are correctly bound, that the right kind of exception mapping is performed and so on - without starting a Servlet Container. All this can also be covered with integration tests, but some edge cases are hard to cover there. 

Take this controller for example: we'd like to see all the annotations taking effect and the `accountManager` being called correctly:

```java
@PutMapping("/account/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void updateAccount(@RequestBody account, @PathVariable("id") long id) {
  accountManager.update(id, account);
}
```

The account manager is a delegate that we would mock with `@MockBean`.

> @Mock vs @MockBean..
> * `@Mock` is Mockito - you use it when Spring Context is not needed, e.g. for basic unit tests.
> * `@MockBean` is part of Spring Boot; it creates a mock bean for those not present in the context OR replaces a bean with a mock bean when it is present

Here is a test implementation:

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;

// @SpringBootTest(webEnvironment = WebEnvironment.MOCK) // comes via @WebMvcTest
// @AutoConfigureMockMvc // comes via @WebMvcTest
@WebMvcTest(AccountController.class)
public class AccountControllerTests {

  @Autowired
  MockMvc mockMvc;

  @MockBean
  private AccountManager accountManager;

  @Test
  public void testBasicGet() {
    given(accountManager.getAccount(1234))
      .willReturn(new Account("123456789", "John Doe"))

    mockMvc.perform(
              get("/accounts/{id}", "1234")
              .accept(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().contentType("application/json"))
            .andExpect(jsonPath("name").value("John Doe"))
            .andExpect(jsonPath("number").value("123456789"));

    // redundant, but possible
    verify(accountManager).getAccount(1234);
  }

  @Test
  public void testPut() {
    mockMvc.perform(
              put("/accounts/{id}", "1234")
              .content("{ ... }")
              .contentType("application/json")
            ).andExpect(status().isNoContent());
  }
}
```

## Repository Slice Testing with `@DataJpaTest`

Annotating a test class with `@DataJpaTest` will load only `@Repository` beans and exclude all other `@Components`. It auto-configures `TestEntityManager` and uses an in-memory database.
All explicitly or auto-configured `DataSource`s are replaced, but with `@AutoConfigureTestDatabase`, this can be influenced.

> For other use cases, there is also annotations like `@DataMongoTest`, `@DataJdbcTest`, `@DataLdapTest` and others.






