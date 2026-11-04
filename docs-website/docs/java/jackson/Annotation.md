# Jackson Annotations

Jackson annotations allow you to customize how Java properties are converted into JSON.

These are very important in Spring Boot REST API development.

Assume we have a User class.

## 4.1 @JsonProperty

Changes the JSON property name.

```
public class User {

    @JsonProperty("user_name")
    private String name;

    private int age;
}


JSON:
{
    "user_name": "Supriyo",
    "age": 29
}
```

__Use case:__  When your Java naming convention and external JSON naming convention are different.


## 4.2 @JsonIgnore

Completely ignores a property during serialization and deserialization.

```
public class User {

    private String name;

    @JsonIgnore
    private String password;
}


Output:
{
    "name": "Supriyo"
}
```

__Use case:__ Prevent exposing internal fields in API responses.

Note: For security-sensitive data, also ensure your input and update logic cannot modify protected properties unexpectedly.


## 4.3 @JsonIgnoreProperties
Ignores selected properties or unknown properties.
```
@JsonIgnoreProperties({"password", "secretKey"})
public class User {

    private String name;
    private String password;
    private String secretKey;
}
```

Another commonly used form:
```
@JsonIgnoreProperties(ignoreUnknown = true)
public class User {
    private String name;
}


Suppose JSON contains:
{
    "name": "Supriyo",
    "age": 29,
    "city": "Kolkata"
}
```

Jackson ignores age and city if they have no matching properties.

Use case: When external APIs send additional fields that your Java class does not need.


## 4.4 @JsonInclude
Controls which properties should be included in serialized JSON.

```
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {

    private String name;
    private String email;
}


If email is null:
{
    "name": "Supriyo"
}
```

### Common options:

|Option|	Meaning|
|------|-----------|
|ALWAYS	| Include all properties|
|NON_NULL |	Exclude null properties|
|NON_EMPTY |	Exclude null and empty values|
|NON_DEFAULT |	Exclude default values|

Example:
```
@JsonInclude(JsonInclude.Include.NON_EMPTY)
private List<String> roles;
```

## 4.5 @JsonFormat
Controls date, time, or value formatting.
```
public class User {

    @JsonFormat(pattern = "dd-MM-yyyy")
    private LocalDate dateOfBirth;
}


Output:
{
    "dateOfBirth": "01-07-1997"
}
```

## 4.6 @JsonAlias
Allows multiple JSON property names during deserialization.
```
public class User {

    @JsonAlias({"user_name", "username"})
    private String name;
}


Both inputs are accepted:
{"user_name": "Supriyo"}


{"username": "Supriyo"}
```

Important: @JsonAlias does not change the serialized property name. For that, use @JsonProperty.


## 4.7 @JsonGetter
Defines a method as a JSON getter.
```
public class User {

    private String name;

    @JsonGetter("full_name")
    public String getName() {
        return name;
    }
}


Output:
{
    "full_name": "Supriyo"
}
```

## 4.8 @JsonSetter
Defines a method as a JSON setter.

```
public class User {

    private String name;

    @JsonSetter("user_name")
    public void setName(String name) {
        this.name = name;
    }
}
```

Useful when incoming JSON property names differ from Java properties.


## 4.9 @JsonPropertyOrder
Controls the order of serialized JSON properties.
```
@JsonPropertyOrder({"name", "age", "email"})
public class User {
    private String email;
    private int age;
    private String name;
}
```

## 4.10 @JsonCreator
Used when Jackson needs to create an object through a constructor or factory method.

```
public class User {

    private final String name;
    private final int age;

    @JsonCreator
    public User(
        @JsonProperty("name") String name,
        @JsonProperty("age") int age
    ) {
        this.name = name;
        this.age = age;
    }
}
```

Use case: Immutable classes, constructor-based deserialization and classes without a default constructor.


## 4.11 @JsonValue
Defines which value represents an entire object during serialization.

```
public enum Status {

    ACTIVE("A"),
    INACTIVE("I");

    private final String code;

    Status(String code) {
        this.code = code;
    }

    @JsonValue
    public String getCode() {
        return code;
    }
}


Output for Status.ACTIVE:
"A"
```

## 4.12 @JsonUnwrapped
Flattens a nested object into its parent JSON object.

```
public class Address {
    private String city;
    private String country;
}

public class User {

    private String name;

    @JsonUnwrapped
    private Address address;
}


Output:
{
    "name": "Supriyo",
    "city": "Kolkata",
    "country": "India"
}
```

## 4.13 @JsonAnyGetter and @JsonAnySetter
Useful for dynamic JSON properties.

```
public class User {

    private Map<String, Object> properties = new HashMap<>();

    @JsonAnyGetter
    public Map<String, Object> getProperties() {
        return properties;
    }

    @JsonAnySetter
    public void setProperty(String key, Object value) {
        properties.put(key, value);
    }
}
```

#### @JsonAnyGetter serializes map entries as normal JSON properties.

#### @JsonAnySetter captures otherwise-unmapped JSON properties into the map.