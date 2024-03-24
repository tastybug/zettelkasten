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








