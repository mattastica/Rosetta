# RosettaCore

This module contains the core mapping and binding logic used throughout.

### Maven dependency

To use module on Maven-based projects, use following dependency:

```xml
<dependency>
  <groupId>com.hubspot.rosetta</groupId>
  <artifactId>RosettaCore</artifactId>
  <version>3.4</version>
</dependency>
```

(or whatever version is most up-to-date at the moment)

## Mapping vs. Binding

Mapping is used when mapping fields and values *to* an object.

Binding is a method for mapping object fields *to* (usually) SQL query parameters.


## Mapping Features

Because Rosetta is built on `jackson-databind`, you can use all the [Annotations](http://wiki.fasterxml.com/JacksonAnnotations) you know and love. Let's cover one common customization: `Enum` handling. Storing an enum in the database by name (ordinal, etc.) is less efficient, less maintainable, and affords fewer queries than a representation like `tinyint`. Say you have leads with some states:

```java
enum LeadState {
  VISITED(0),
  STARRED(1),
  EMAILED(2),
  BOUGHT(3)

  final int state;

  LeadState(int state) {
    this.state = state;
  }

  public final int getState() {
    return state;
  }
}
```

Standard stuff. And finding all pending leads is as simple as `select * from leads where state < 3`. But we still need to tell Rosetta how to translate the integer (from the database) to the enum value (on our pretend `Lead` class). Here's all it takes:

```java
enum LeadState {
  VISITED(0),
  STARRED(1),
  EMAILED(2),
  BOUGHT(3)

  final int state;

  LeadState(int state) {
    this.state = state;
  }

  @RosettaValue
  public final int getState() {
    return state;
  }
}
```

### Nested objects

Nested objects are supported using dot-notation. If you have a `Lead` object with a `FullName` object inside, it might look like:

```java
public class Lead {
  private FullName fullName;
  
  // getter/setter
}
```

```java
public class FullName {
  private String firstName;
  private String lastName;
  
  // getters/setters
}
```

Now a query like this will work as you might expect:

```sql
SELECT lead.firstName as `fullName.firstName`, last.lastName as `fullName.lastName`
FROM lead
```

## Mapping API

Although it's rare you'll use it independently, mapping from a `ResultSet` to an object is handled by `RosettaMapper`

## Binding Features

The binding side of the house affords the same functionality, though generally through more context-specific modules such as `RosettaJdbi`. It utilizes the same Jackson annotations, supports mapping nested object fields to prefixed query parameters, etc.

For example, consider the following classes (infer getters/setters):

```java
public class Parent {
  String name;
  Person child;
}

public class Person {
  String name;
  int age;
  Gender gender;
}

enum Gender {
  MALE(0),
  FEMALE(1),

  int digit;

  @RosettaValue
  public int forDb() {
    return digit;
  }
}
```

Would bind the following keys/values:

```json
{
  "name": "Jill",
  "child.age": 15,
  "child.gender": 0,
  "child.name": "Bill"
}
```
