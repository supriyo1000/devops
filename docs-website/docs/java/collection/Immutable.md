# Immutable Collections

## 1. What is an Immutable Collection?

An immutable collection cannot be changed after creation.

You cannot:

+ add elements
+ remove elements
+ replace elements
+ clear the collection

Java provides convenient factory methods:
```
List.of(...)
Set.of(...)
Map.of(...)
```

## Immutable vs Unmodifiable

Unmodifiable does not necessarily mean immutable.

An immutable collection cannot be modified after creation. An unmodifiable collection is usually a read-only view of another collection; modifications to the original collection can still be visible through the unmodifiable view.

### Unmodifiable collection

An unmodifiable view prevents modification through that particular reference, but the original collection can still change.
```
List<String> original = new ArrayList<>();

original.add("Java");
original.add("Spring");

List<String> view = Collections.unmodifiableList(original);
```

You cannot modify through view:
```
view.add("Kafka"); // UnsupportedOperationException
```
But you can modify the original:
```
original.add("Kafka");

System.out.println(view);

Output:

[Java, Spring, Kafka]
```

## 3. what is List.of()
```
List.of()
├── Ordered list
├── Duplicates allowed
├── null NOT allowed (NullPointerException)
└── Immutable
```
Available since Java 9.
```
List<String> languages = List.of(
    "Java",
    "Python",
    "JavaScript"
);
```
You can read:
```
System.out.println(languages.get(0));
System.out.println(languages.size());
```
But you cannot modify it:
```
languages.add("Go");       // UnsupportedOperationException
languages.remove("Java");  // UnsupportedOperationException
languages.set(0, "C++");   // UnsupportedOperationException
```

## 4. Set.of()
```
Set.of()
├── Immutable (Cant undate)
├── Duplicates NOT allowed (IllegalArgumentException)
├── null NOT allowed (NullPointerException)
└── Iteration order should not be relied upon
```

## 5. Map.of()

Creates an immutable Map.

```
Map.of()
├── Immutable
├── Duplicate keys NOT allowed
├── null keys NOT allowed
└── null values NOT allowed
```

```
Map<Integer, String> users = Map.of(
    1, "Supriyo",
    2, "Rahul",
    3, "Amit"
);
```
Reading works normally:
```
System.out.println(users.get(1));

Output:

Supriyo
```
#### Modification does not:
```
users.put(4, "Rohit"); // UnsupportedOperationException
users.remove(1); // UnsupportedOperationException
Duplicate keys are not allowed
Map<Integer, String> map = Map.of(
    1, "Java",
    1, "Spring"
);
```
Result:
```
IllegalArgumentException
```

#### null keys and values are not allowed
```
Map.of(1, null); // NullPointerException
Map.of(null, "Java"); // NullPointerException
```

## 6. Collections.unmodifiableList()

Now let's understand the older approach.
```
List<String> original = new ArrayList<>();

original.add("Java");
original.add("Spring");

List<String> unmodifiable =
        Collections.unmodifiableList(original);
```
Trying to modify the unmodifiable reference:
```
unmodifiable.add("Kafka");
```
Throws:
```
UnsupportedOperationException
```
But:
```
original.add("Kafka");

System.out.println(unmodifiable);

Output:

[Java, Spring, Kafka]
```

## 8. Very Important Interview Example
```
List<String> original = new ArrayList<>();
original.add("A");
original.add("B");

List<String> immutable = List.of("A", "B");

List<String> unmodifiable =
        Collections.unmodifiableList(original);

original.add("C");

System.out.println(immutable);
System.out.println(unmodifiable);
```

## 9. Common Misunderstanding: Immutable Collection ≠ Immutable Objects

Consider:
```
List<List<String>> list = List.of(
    new ArrayList<>(List.of("Java"))
);
```
You cannot replace the inner list:
```
list.add(new ArrayList<>()); // UnsupportedOperationException
```

But the object inside can still be mutable:
```
list.get(0).add("Spring");

System.out.println(list);

Output:

[[Java, Spring]]
```

This is called the difference between:

+ shallow immutability
+ deep immutability

For now, remember:
```
An immutable collection does not guarantee that the objects stored inside it are immutable.
```

## Most important extra method to remember
for read only

```
List.copyOf(list)
Set.copyOf(set)
Map.copyOf(map)
```