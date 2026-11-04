# Jackson 

## 1. What is Jackson?

Jackson is a Java library used to process JSON (JavaScript Object Notation).

It provides conversion between:

+ Java Object → JSON (Serialization)
+ JSON → Java Object (Deserialization)

Example:

Java object:
```
public class User {
    private String name;
    private int age;

    // getters and setters
}
```
```
JSON:

{
    "name": "Supriyo",
    "age": 29
}
```

Jackson performs this conversion automatically.

### Why do we use Jackson?
+ Convert Java objects into JSON.

+ Convert JSON into Java objects.

+ Handle nested objects and collections.

+ Handle generic types.

+ Customize JSON properties.

+ Handle dates and Java 8 time classes.

+ Ignore or include specific fields.

+ Handle unknown JSON properties.

+ Validate and process JSON structures.


## 2. ObjectMapper – The Most Important Class

ObjectMapper is the central class of Jackson's Databind API.

It is responsible for serialization and deserialization.
```
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();
```

### Remember these three methods:

+ writeValueAsString()  -->   Java object → JSON String

+ readValue() --> JSON String → Java object

+ convertValue() --> Java object → another Java type

### 2.1 Serialization – Java Object to JSON
```
ObjectMapper mapper = new ObjectMapper();

User user = new User();
user.setName("Supriyo");
user.setAge(29);

String json = mapper.writeValueAsString(user);

System.out.println(json);

Output:

{"name":"Supriyo","age":29}
```

For pretty-printed JSON:
```
String json = mapper
        .writerWithDefaultPrettyPrinter()
        .writeValueAsString(user);
```

### 2.2 Deserialization – JSON to Java Object
```
String json = """
    {
        "name": "Supriyo",
        "age": 29
    }
    """;

User user = mapper.readValue(json, User.class);

System.out.println(user.getName());

Output:

Supriyo
```


### 2.3 ConvertValue – One Java Object to Another

Useful when you want to convert a DTO into another DTO or a Map.
```
User user = new User();
user.setName("Supriyo");
user.setAge(29);

Map<String, Object> map =
        mapper.convertValue(user, Map.class);
```

You can also convert a Map into a Java object:
```
User user = mapper.convertValue(map, User.class);
```