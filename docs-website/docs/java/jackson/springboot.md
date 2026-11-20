# 18. Spring Boot Integration – How Jackson Works Internally

This is particularly important for your Spring Boot development.

You usually don't need to manually call ObjectMapper in every controller.

Spring Boot uses Jackson through HttpMessageConverter, commonly MappingJackson2HttpMessageConverter, to handle JSON HTTP request and response bodies.

## 18.1 Java Object to JSON automatically

Controller:

```
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping
    public User getUser() {

        User user = new User();

        user.setName("Supriyo");
        user.setAge(29);

        return user;
    }
}


Response:
{
    "name": "Supriyo",
    "age": 29
}
```

### What happens internally?
```
Client Request
      |
      v
Spring DispatcherServlet
      |
      v
UserController
      |
      v
Returns User Object
      |
      v
Jackson HttpMessageConverter
      |
      v
ObjectMapper
      |
      v
JSON Response
```

## 18.2 JSON to Java Object automatically
```
@PostMapping
public String createUser(@RequestBody User user) {

    return "User created: " + user.getName();
}


Request:
{
    "name": "Supriyo",
    "age": 29
}
```

Spring uses Jackson to deserialize the request JSON into a User object.


## 18.3 Inject ObjectMapper
If you need manual JSON processing inside a service, inject the configured Spring Boot ObjectMapper.

```
@Service
public class JsonService {

    private final ObjectMapper objectMapper;

    public JsonService(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    public String convertToJson(User user)
            throws JsonProcessingException {

        return objectMapper.writeValueAsString(user);
    }
}
```

Important: Prefer reusing the Spring-managed ObjectMapper instead of creating a new one for every operation. Configure it during application setup rather than modifying it after use.

## 19. Jackson Modules
Modules extend Jackson's functionality to support additional Java types and formats.


|Module	|Purpose|
|-------|-------|
|JavaTimeModule	|Java 8 date/time classes|
|ParameterNamesModule	|Constructor parameter name support|
|Jdk8Module	|Java 8 Optional types|
|KotlinModule	| Kotlin data classes and Kotlin-specific features|
|AfterburnerModule	| Performance optimization for databinding in supported environments|

### Register a module:

```
mapper.registerModule(new JavaTimeModule());
```

### Register multiple modules:
```
mapper.registerModule(new JavaTimeModule());
mapper.registerModule(new Jdk8Module());
```

In Spring Boot, many modules are automatically detected and registered when present on the classpath.

## 20. Handling Optional and Enum Types


### 20.1 Optional

Java:

```
public class User {
    private String name;
    private Optional<String> email;
}
```

For Java 8 Optional support, include the jackson-datatype-jdk8 module.


### 20.2 Enum
Java:

```
public enum UserStatus {
    ACTIVE,
    INACTIVE,
    BLOCKED
}
```

Class:

```
public class User {
    private String name;
    private UserStatus status;
}


JSON:
{
    "name": "Supriyo",
    "status": "ACTIVE"
}
```

Jackson normally serializes enums using their names.

You can customize enum serialization using @JsonValue.

You can also configure:

```
mapper.enable(
    SerializationFeature.WRITE_ENUMS_USING_TO_STRING
);
```

This uses the enum's toString() representation.

## 21. Handling Unknown and Missing Values
There is an important difference between missing properties, null properties and default values.

Consider:
```
public class User {
    private String name;
    private int age;
}

Input:
{
    "name": "Supriyo"
}
```

Here, age is missing. Its Java primitive default value will normally be 0.

Now consider:
```
{
    "name": "Supriyo",
    "age": null
}
```

A null value for a primitive int is not valid by default in every Jackson configuration. You can enable strict handling:

```
mapper.enable(
    DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES
);
```

For nullable values, use wrapper classes:
```
private Integer age;
```

Remember:
- int → Cannot store null.
- Integer → Can store null.
- Missing JSON property → Usually leaves the Java default value.
- Explicit JSON null → Depends on target type and Jackson configuration.

## 22. Jackson Security and Best Practices
Jackson is frequently used to process external JSON, so security matters.

Important practices:
1. Do not enable unrestricted polymorphic default typing for untrusted JSON.
2. Avoid exposing passwords, tokens and internal fields in API responses.
3. Validate incoming request data using Bean Validation.
4. Avoid accepting unexpected fields in sensitive operations.
5. Use explicit allowed subtype mappings for polymorphic deserialization.
6. Reuse a configured ObjectMapper.
7. Avoid logging sensitive JSON request bodies.
8. Keep Jackson dependencies updated.


Example:

```
public class UserRequest {

    @NotBlank
    private String name;

    @Email
    private String email;

    // getters and setters
}
```

Controller:

```
@PostMapping
public ResponseEntity<String> createUser(
        @Valid @RequestBody UserRequest request) {

    return ResponseEntity.ok("User created");
}
```

Remember that Jackson handles JSON conversion, whereas Bean Validation handles application-level input constraints.