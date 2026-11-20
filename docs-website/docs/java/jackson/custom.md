# 10. Custom Serializer and Deserializer
Sometimes default Jackson conversion is insufficient.

For example, you want to serialize a Java object in a custom format.


## 10.1 Custom Serializer
Suppose you want to serialize a User as:
```
"Supriyo - 29"
```

### Create a serializer:

```
public class UserSerializer
        extends JsonSerializer<User> {

    @Override
    public void serialize(
            User user,
            JsonGenerator generator,
            SerializerProvider provider
    ) throws IOException {

        generator.writeString(
            user.getName() + " - " + user.getAge()
        );
    }
}
```

### Register it using:

```
@JsonSerialize(using = UserSerializer.class)
public class User {
    // fields, getters and setters
}
```

## 10.2 Custom Deserializer
Suppose incoming JSON contains:
```
"Supriyo - 29"
```

You want to convert it into a User object.

```
public class UserDeserializer
        extends JsonDeserializer<User> {

    @Override
    public User deserialize(
            JsonParser parser,
            DeserializationContext context
    ) throws IOException {

        String value = parser.getText();

        String[] parts = value.split(" - ");

        User user = new User();
        user.setName(parts[0]);
        user.setAge(Integer.parseInt(parts[1]));

        return user;
    }
}
```

### Register:

```
@JsonDeserialize(using = UserDeserializer.class)
public class User {
    // fields, getters and setters
}
```

Use cases:
- Custom date formats.
- Legacy API integration.
- Special JSON structures.
- Data transformation during serialization/deserialization.

## 11. Custom Naming Strategies
Sometimes Java uses camelCase while an external API expects snake_case.

Example:

Java:
```
public class User {
    private String firstName;
    private String lastName;
}


Expected JSON:
{
    "first_name": "Supriyo",
    "last_name": "Chatterjee"
}
```

Instead of adding @JsonProperty to every field, configure a naming strategy:

```
mapper.setPropertyNamingStrategy(
    PropertyNamingStrategies.SNAKE_CASE
);
```

Other naming strategies:

|Strategy|	Example|
|--------|---------|
|LOWER_CAMEL_CASE|	firstName|
|UPPER_CAMEL_CASE|	FirstName|
|SNAKE_CASE|	first_name|
|KEBAB_CASE|	first-name|
|LOWER_CASE|	firstname|


12. Polymorphic Deserialization
This is an advanced but useful feature.

Suppose you have a parent class called Vehicle, with two child classes: Car and Bike.

```
public class Vehicle {
    private String brand;
}

public class Car extends Vehicle {
    private int doors;
}

public class Bike extends Vehicle {
    private boolean electric;
}
```

Now imagine receiving:
```
{
    "type": "car",
    "brand": "Toyota",
    "doors": 4
}
```

How will Jackson know whether it should create a Car or Bike object?

#### Use @JsonTypeInfo and @JsonSubTypes.

```
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)

@JsonSubTypes({
    @JsonSubTypes.Type(
        value = Car.class,
        name = "car"
    ),
    @JsonSubTypes.Type(
        value = Bike.class,
        name = "bike"
    )
})


public class Vehicle {
    private String brand;
}
```

#### Deserialize:
```
Vehicle vehicle = mapper.readValue(json, Vehicle.class);
```

Jackson will create a Car object based on the type property.

Use cases:
- Different notification types.
- Payment methods.
- Different event types.
- Multiple implementations of a common interface.

Security note: Avoid unrestricted default typing for untrusted JSON. Prefer explicit subtype mappings with known allowed classes.