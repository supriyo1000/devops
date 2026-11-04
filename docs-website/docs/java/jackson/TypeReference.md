# TypeReference – Handling Generic Types

This is one of the most important Jackson concepts.

## 3.1 The problem with generic types
Suppose you have this JSON:
```
[
    {
        "name": "Supriyo",
        "age": 29
    },
    {
        "name": "Rahul",
        "age": 28
    }
]
```

You want to convert it into:
```
List<User>
```

You might try:
```
List<User> users =
        mapper.readValue(json, List.class);
```

But there is a problem.

Java's List.class does not contain information about its generic type (User).

Because of type erasure, Jackson cannot reliably determine that each list element should be a User.


## 3.2 Solution: TypeReference
```
import com.fasterxml.jackson.core.type.TypeReference;

List<User> users = mapper.readValue(
        json,
        new TypeReference<List<User>>() {}
);
```

Now Jackson knows that it should create a list containing User objects.


## 3.3 Why do we use {}?
```
new TypeReference<List<User>>() {}
```

This creates an anonymous subclass of TypeReference.

It allows Jackson to capture the generic type information at runtime through Java's generic superclass metadata.

#### Remember:
- List.class → Only knows it is a List.
- TypeReference<List<User>> → Knows it is a List of User objects.


## 3.4 Other TypeReference examples

### List of objects:
```
List<User> users = mapper.readValue(
    json,
    new TypeReference<List<User>>() {}
);
```

### Map of objects:
```
Map<String, User> users = mapper.readValue(
    json,
    new TypeReference<Map<String, User>>() {}
);
```

### Nested generic types:
```
Map<String, List<User>> data = mapper.readValue(
    json,
    new TypeReference<Map<String, List<User>>>() {}
);
```

## Generic API response:

Suppose your response class is:

```
public class ApiResponse<T> {
    private boolean success;
    private T data;

    // getters and setters
}
```

#### JSON:
```
{
    "success": true,
    "data": {
        "name": "Supriyo",
        "age": 29
    }
}
```

#### Conversion:
```
ApiResponse<User> response = mapper.readValue(
    json,
    new TypeReference<ApiResponse<User>>() {}
);
```

__Remember:__ Whenever you deserialize a generic collection or generic wrapper, consider using TypeReference.


## 3.5 TypeReference vs JavaType

Jackson provides another approach called JavaType.
```
JavaType type = mapper.getTypeFactory()
        .constructCollectionType(List.class, User.class);

List<User> users = mapper.readValue(json, type);
```

### For nested generic structures:
```
JavaType listType = mapper.getTypeFactory()
        .constructCollectionType(List.class, User.class);

JavaType mapType = mapper.getTypeFactory()
        .constructMapType(
            Map.class,
            String.class,
            listType
        );

Map<String, List<User>> data =
        mapper.readValue(json, mapType);
```