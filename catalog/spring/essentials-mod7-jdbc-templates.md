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

### Basic `queryForMap` and `queryForList`

There are 2 very basic ways of returning `ResultSet` data: `queryForMap(..)` and `queryForList(..)` return Map and List of Map respectively, without mapping content to a type. Use this for ad hoc debugging and testing to see what the DB provides.

Single result expected? `queryForMap`!
```java
public Map<String, Object> getPersonInfo() {
  // returns Map: {ID=50, FIRST_NAME="Foo", LAST_NAME="Bar"}
  return jdbcTemplate.queryForMap("SELECT * from PERSON WHERE id = 50");
}
```

Multiple results expected? `queryForList` gives you List of Maps with K/V pairs.
```java
public List<Map<String, Object>> getPersons() {
  // returns List of Map:
  // [  {ID=50, FIRST_NAME="Foo", LAST_NAME="Bar"}, {ID=51, FIRST_NAME="Foo", LAST_NAME="Baz"} ]
  return jdbcTemplate.queryForList("SELECT * from PERSON");
}
```

### Mapping to Domain Objects (single to single, list to list or list to aggregate)

Usually you want to map the `ResultSet` into your `Account`, `Person` or whatever domain object there is. You can do this programmatically, avoiding ORM layers where the data model is simple.

There are 3 constructs to for mapping purposes, each of which you likely use in form of a lambda:
* `RowMapper: (resultset, rownum) -> {return 1 line mapped}`
* `ResultSetExtractor: (resultset) -> { return aggregate}`
* (`RowCallbackHandler` not investigated here)


Mapping a single result using `RowMapper`:
```java
public Person getPerson(int id) {
  return jdbcTemplate.queryForObject(
    "SELECT first_name, last_name FROM person WHERE id = ?",
    (rs, rowNum) -> return Person(rs.getString("first_name), rs.getString("last_name")),
    id
  );
} 
```

Mapping multiple results using `RowMapper`:
```java
public List<Person> getAllPersons() {
  return jdbcTemplate.query(
    "SELECT first_name, last_name FROM person",
    (rs, rowNum) -> return Person(rs.getString("first_name), rs.getString("last_name"))
  );
} 
```

Turning a list of results into a single object. Here you use `ResultSetExtractor` to aggregate data:

```java
public Order getOrderWithLineItems(String orderNumber) {
  return jdbcTemplate.query(
    // query with join
    "SELECT .. FROM order o, item i <and so on> order_number = ?",
    // first bracket is a cast!
    (ResultSetExtractor<Order>)(rs) -> {
      Order order = null;
      while (rs.next()) {
        if (order == null) {
          order = new Order(rs.getLong("order_number"), rs.getString("name"));
        }
        order.addLineItem(mapLineItem(rs));
      }
      return order;
    },
    orderNumber
  );
}

private LineItem mapLineItem(ResultSet rs) {
  // mapping code to map to line item domain type
}
```




