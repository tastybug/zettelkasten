# JDBC Templates

`JdbcTemplates` are threat safe and reusable. They hide a lot complexity behind a clean abstraction. Exceptions will be thrown out of it. Custom behaviour can be wired in.

## Problems with Plain JDBC

There is a lot of boilerplate code necessary to run a plain JDBC query: connection handling, dealing with the checked exception, mapping from ResultSet to return type. You mix different concerns in the same place.

```java
public List<Person> findByLastName(String lastName) {
  List<Person> persons = new ArrayList<>();
  String query = "SELECT first_name, age FROM person WHERE last_name=?;

  try (Connection con = dataSource.getConnection();
       PreparedStatement ps = con.prepareStatement(query)) {
    ps.setString(1, lastName);
    try (ResultSet rs = ps.executeQuery()) {
      while (rs.next()) {
        persons.add(new Person(rs.getString("first_name"), ...));
      }
    }
  } catch (SQLException e} {
    // forced to deal with this
  }

  return persons;
}
```

## Basic Use of `JdbcTemplate`

`JdbcTemplate` can query for simple types, maps and domain objects (subject to mapping).

```java
@Repository
public class JdbcCustomerRepository implements CustomerRepository {
  private JdbcTemplate jdbcTemplate;

  public JdbcCustomerRepository(JdbcTemplate templ) {
    this.jdbcTemplate = templ;
  }

  // no try/catch enforced due to unchecked exception
  public int getCustomerCount() {
    String sql = "SELECT COUNT(*) FROM customer";
    return jdbcTemplate.queryForObject(sql, Integer.class);
  }
}
```

### Binding Values to Parameterized Queries

```java
public int getCountOfPeopleAboveAgeInCountry(Nationality nationality, int age) {
  String sql = "SELECT COUNT(*) FROM person WHERE age > ? AND nationality = ?";
  return jdbcTemplate.queryForObject(
    sql, // query
    Integer.class, // return type
    age, // binds to first ?
    nationality.toString() // binds to second ?
  );
}
```

### Inserts and Updates

Any non-select goes via `update(..)`.

```java
public int insertPerson(Person p) {
  int numberOfRowsAffected = jdbcTemplate.update(
    "INSERT INTO person (first_name, last_name, age) VALUES (?, ?, ?),
    person.getFirstName(),
    person.getLastName(),
    person.getAge()
  );
  return numberOfRowsAffected;
}
```

```java
public int updateAge(Person p) {
  int numberOfRowsAffected = jdbcTemplate.update(
    "UPDATE person SET age = ? WHERE id = ?,
    person.getAge(),
    person.getId()
  );
  return numberOfRowsAffected;
}
```

## Working With `ResultSets`



