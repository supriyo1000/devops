# 16. Working with JSON Files

Jackson can read and write JSON files directly.

## 16.1 Write JSON to a file

```
User user = new User();
user.setName("Supriyo");
user.setAge(29);

mapper.writeValue(
    new File("user.json"),
    user
);
```

## 16.2 Read JSON from a file
```
User user = mapper.readValue(
    new File("user.json"),
    User.class
);
```

## 16.3 Read a JSON array from a file

```
List<User> users = mapper.readValue(
    new File("users.json"),
    new TypeReference<List<User>>() {}
);
```

## 16.4 Read JSON from an InputStream

```
try (InputStream input =
        new FileInputStream("user.json")) {

    User user = mapper.readValue(input, User.class);
}
```


# 17. Common Jackson Exceptions

You should understand these because they frequently appear when developing REST APIs.


|Exception|	Meaning|	Common reason|
|---------|--------|-----------------|
|JsonProcessingException|	General JSON processing failure|	Invalid JSON or mapping issue|
|JsonParseException|	JSON syntax error|	Missing comma, invalid syntax|
|JsonMappingException|	Mapping problem|	|JSON doesn't match Java structure|
|UnrecognizedPropertyException|	Unknown property|	JSON contains an unexpected field|
|MismatchedInputException|	Input type mismatch|	String supplied where an integer is expected|
|InvalidDefinitionException|	Invalid Java type definition|	Unsupported or incorrectly configured type|

### Example: Handle Jackson exceptions

```
try {

    User user = mapper.readValue(json, User.class);

} catch (JsonProcessingException e) {

    System.out.println("JSON processing failed: "
            + e.getOriginalMessage());
}
```

For a Spring Boot REST API, these failures can also be handled centrally using @RestControllerAdvice and @ExceptionHandler.