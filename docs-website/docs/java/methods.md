# Java Methods

+ __java.util.Arrays__ : view,transform,manipulate array, without need of manual loop.

+ __Arrays.asList__ : fixed size list view of an array.cannot add/remove element.

+ __Arrays.stream__ : to convert an array into **stream**. used for mapping,filtering,reducing data.

1. **Object Stream** : Stream<String>, Stream<Integer>, Stream<Character>.
2. **Primitive Stream** : IntStream, doubleStream, longStream.

+ __Arrays_toString__ : java do not have toString() implementation. it prints memory location by default.used to print content of 2d array.

+ __Arrays.deepToString__ : used to print multi dimensional array.

|Situation|Method|
|---------|------|
|Object -> Object| map()|
|Object -> int| mapToInt()|
|int -> Object| mapToObj()|
|many -> flatten| flatMap()|
|many(int) -> flatten| flatMapToInt()|


### map()

```ruby
List<String> list = List.of("1","2","3");

list.stream().map(e -> Integer.parseInt(e))
```

+ string -> integer
+ still object -> object

### 🔹 2. mapToInt()

👉 Object → primitive int

```ruby
list.stream()
    .mapToInt(s -> Integer.parseInt(s))
```

Now:

IntStream


### 🔹 3. mapToObj()

👉 Primitive → Object

```ruby
IntStream.of(1,2,3)
    .mapToObj(i -> i + "x")
```

Now:

Stream<String>


### 🔹 4. flatMap()

👉 When 1 input gives MANY outputs

Example:
```ruby
List<List<Integer>> list = List.of(List.of(1,2), List.of(3,4));

list.stream()
    .flatMap(l -> l.stream())
```

👉 Output:

[1,2,3,4]


### 🔹 5. flatMapToInt()

👉 Same as above BUT for int
```ruby
String[] arr = {"ab","cd"};

Arrays.stream(arr)
    .flatMapToInt(s -> s.chars())
```
👉 Output:

IntStream → 97,98,99,100


### 🔹 6. String::chars

👉 Very important

```ruby
"ab".chars()
```

Gives:

IntStream → [97, 98]

NOT characters ❗


### 🔹 7. (char) c

👉 Convert ASCII → actual character

```ruby
97 → 'a'
.mapToObj(c -> (char)c)
```

### 🔹 8. String::valueOf

👉 Convert anything → String

```ruby
String.valueOf(10) → "10"
```

### 🔹 9. Integer.parseInt()

👉 String → primitive int
```ruby
Integer.parseInt("10") → 10
```

### 🔹 10. Integer.intValue()

👉 Object → primitive
```ruby
Integer x = 10;
x.intValue() → 10
```

### 11. String[]::new

used to create a new array of Strings

+ stream api to array of String.
+ Mapping and Collecting

### 12. chars()

gives IntStream

"ab" -> [97,98]

### 13. c -> (char)c

intstream to character

[97,98] -> "ab"

### HOW TO FIND SOLUTION IN EXAMS/INTERVIEWS

Never think about syntax first.

Think like this:

PROBLEM
```ruby
["ab","cd"] -> ['A','B','C','D']
```

#### STEP 1

Need characters from each string.
```
String -> many characters
```
So brain says:
```
flatMap
```

#### STEP 2

How to get chars?
```
chars()
```

#### STEP 3

chars() gives numbers.

Need character.
```
mapToObj(c -> (char)c)
```

#### STEP 4

Need uppercase.
```
map(Character::toUpperCase)
```

#### STEP 5

Need List.
```
collect(Collectors.toList())
```
