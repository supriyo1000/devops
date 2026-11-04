# Jackson Tree Model – JsonNode
Sometimes you don't know the exact JSON structure in advance.

For example, you receive JSON from an external API and only need to read a few fields.

In that situation, you can use JsonNode.


## 7.1 Read JSON using JsonNode

```
JsonNode root = mapper.readTree(json);

String name = root.get("name").asText();
int age = root.get("age").asInt();
```

## 7.2 Access nested JSON
```
String city = root
        .get("address")
        .get("city")
        .asText();
```

## 7.3 Safe access using path()
```
String phone = root.path("phone").asText("Not Available");
```

#### Difference:

+ get() -->	Returns null if the field does not exist
+ path() --> 	Returns a MissingNode if the field does not exist


## 7.4 Check whether a field exists
```
if (root.has("name")) {
    System.out.println(root.get("name").asText());
}
```

## 7.5 Iterate over JSON fields
```
Iterator<Map.Entry<String, JsonNode>> fields =
        root.fields();

while (fields.hasNext()) {
    Map.Entry<String, JsonNode> field = fields.next();

    System.out.println(field.getKey());
    System.out.println(field.getValue());
}
```

## 7.6 Modify JSON using ObjectNode
```
ObjectNode objectNode = (ObjectNode) root;

objectNode.put("age", 30);
objectNode.put("city", "Kolkata");

objectNode.remove("password");
```

## 7.7 Create JSON manually
```
ObjectNode user = mapper.createObjectNode();

user.put("name", "Supriyo");
user.put("age", 29);

ArrayNode skills = user.putArray("skills");

skills.add("Java");
skills.add("React");

String json = mapper.writeValueAsString(user);
```

Remember: Use JsonNode when you need flexible JSON processing without creating a Java class for every JSON structure.

# 8. Jackson Streaming API – JsonParser and JsonGenerator
Jackson has three main ways to process JSON:

1. Data Binding – Convert JSON directly into Java objects.
2. Tree Model – Work with JSON as a tree of nodes.
3. Streaming API – Read or write JSON token by token.


## 8.1 JsonParser – Read JSON
```
JsonFactory factory = new JsonFactory();

try (JsonParser parser = factory.createParser(json)) {

    while (parser.nextToken() != null) {

        System.out.println(parser.currentToken());

        if (parser.currentToken() == JsonToken.VALUE_STRING) {
            System.out.println(parser.getText());
        }
    }
}
```

## 8.2 JsonGenerator – Write JSON
```
JsonFactory factory = new JsonFactory();

StringWriter writer = new StringWriter();

try (JsonGenerator generator =
        factory.createGenerator(writer)) {

    generator.writeStartObject();

    generator.writeStringField("name", "Supriyo");
    generator.writeNumberField("age", 29);

    generator.writeEndObject();
}

System.out.println(writer);


Output:
{
  "name": "Supriyo",
  "age": 29
}
```

### When should you use Streaming API?

- Processing very large JSON files.
- Processing JSON without loading the entire document into memory.
- Reading large data streams.
- When you need fine-grained control over JSON parsing.
Interview point: Data Binding is convenient, Tree Model is flexible, and Streaming API is useful for memory-efficient processing.

# 9. Java 8 Date and Time Handling
Jackson needs the Java Time module to handle Java 8 date/time classes properly.

#### Common classes:

- LocalDate
- LocalTime
- LocalDateTime
- ZonedDateTime
- OffsetDateTime


## 9.1 Dependency
```
Maven:
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```

When using Spring Boot dependency management, normally you don't need to specify the version.


## 9.2 Register JavaTimeModule

```
ObjectMapper mapper = new ObjectMapper();

mapper.registerModule(new JavaTimeModule());

mapper.disable(
    SerializationFeature.WRITE_DATES_AS_TIMESTAMPS
);
```

## 9.3 Example
```
public class Employee {

    private String name;

    private LocalDate joiningDate;

    private LocalDateTime createdAt;

    // getters and setters
}


JSON:
{
    "name": "Supriyo",
    "joiningDate": "2026-11-03",
    "createdAt": "2026-11-03T10:30:00"
}
```

## 9.4 Custom date format

```
@JsonFormat(pattern = "dd-MM-yyyy")
private LocalDate joiningDate;
```

__Spring Boot note:__ Spring Boot's Jackson auto-configuration normally registers the Java Time module when the dependency is available. You usually don't need to manually create another ObjectMapper.