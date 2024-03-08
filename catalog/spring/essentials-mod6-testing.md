# Testing Spring Applications

In production, you start your java process which somehow sets up a Spring Context, maybe with `ApplicationContext context = SpringApplication.run(ApplicationConfig.class);`.

When testing, something similar happens: you have a test runner executing test classes which again create a Spring Context - it will be a Context with a different configuration though. It will replace a bunch of infrastructure parts and maybe not even load some elements of the application.
In an integration test, you'll use `org.springframework:spring-test`.

## Basics

A basic test class:
```
@ExtendWith(SpringExtension.class) // run with spring support
@ContextConfiguration(classes={SystemTestConfig.class}) // test configuration
public class TransferServiceTest {

  @Autowired // this works thanks to SpringExtension, otherwise you'd need to use @BeforeEach and build context programmatically
  private TransferService transferService;

  @Test
  void shouldTransferMoney() {
    var result = transferService.transfer(..);
    // ...
  }
}
```

Composite annotations to make life easier: `@SpringJUnitConfig(SystemTestConfig.class)`: Combines `@ExtendsWith(SpringExtension.class)` with `@ContextConfiguration`

### Inner Class Context Configuration

You can use `@SpringJUnitConfig` without specifying the configuration. The SpringRunner will look for locally declared configuration in the test class then.

```
@SprungJUnitConfig
public class JdbcAccountRepoTest {

  @Test
  public void someTest() {}

  // this will be picked up
  @Configuration
  @Import(SystemTestConfig.class)
  static class TestConfiguration {

    @Bean // override the bean with a test alternative
    public DataSource dataSource() {...}
  }
}
```

### I Want a New Context With Every Test: `@DirtiesContext`

The test Context is shared among test classes for the sake of speed. Since beans are usually stateless, this is mostly fine.
If you want to force the recreation of the Spring Context, use `@DirtiesContext` on the class or on a test case.

### Overriding Properties w/o Profiles

You can ovrride application properties via annotation. The values given in the annotation have a higher precedence that other values.
Keep in mind that profiles are a cleaner approach, see below.

```
@SpringJUnitConfig(SystemTestConfig.class)
@TestPropertySource(properties = {"username=foo", "password=bar"},
                    locations = "classpath:/test.properties")
public class TransferServiceTests {
  ...
}
```

## Profiles for Different Run Modes

You can already abstract away the runtime environment by providing environment variables, providing JDBC URLs and whatnot to make differences in prod and dev stages manageable. The concept of profiles takes this one step further and also changes the composition of your application by disabling/enabling Beans, replacing beans with Mocks and so on.

