# 📘 JavaScript String Methods

## 1. What is a String?
a string is a sequence of character that represents as a text.

string is **immutable**.

#### That mean
you cannot change a character directly

```ruby
let s = "hello";
s[0] = "H";
console.log(s); // hello
```
you must create a new string

```ruby
let s = "hello";
s = "H" + s.slice(1);
console.log(s); // Hello
```

## length

Returns total number of characters.

```ruby
let s = "hello";
console.log(s.length); // 5
```

## toUpperCase() and toLowerCase()

```ruby
let s = "hello";
let a = s.toUpperCase(); // HELLO
let b = a.toLowerCase(); // hello
```

## includes()

Check if a substring exist.

```ruby
let text = "I love JavaScript";
let ispresent = text.includes("love");
let check = text.includes("has");
console.log(ispresent); // true
console.log(check); // false
```

## indexOf()
Returns the position of first occurance

```ruby
let text = "abbacdd"
console.log(text.indexOf("a"));
```

if not found returns **-1**.

## slice(start, end) ⭐ VERY IMPORTANT

extracts part of a string.

```ruby
let text = "JavaScript";
console.log(text.slice(1 , 6)); // avaSc
console.log(text.slice(5)); // cript
```

## substring()

almost same as slice but does not accept negative index;

## replace()

Replaces the first occurance.

```ruby
let s = "I like Java";
console.log(s.replace("Java","JavaScript"));
```

## split() ⭐⭐⭐ MOST IMPORTANT

convert string to Array.

```ruby
let s = "i like Java";
console.log(s.split(" ")); // [ 'i', 'like', 'Java' ]
console.log(s.split("i")); // [ '', ' l', 'ke Java' ]
```

## trim()

Removes extra spaces from first and last.

```ruby
let name = "  supriyo  ";
console.log(name.trim()); // supriyo

let fullname = "supriyo  chatterjee"
console.log(fullname.trim()); // supriyo  chatterjee
```

## charAt() and indexing

```ruby
let s = "hello";
console.log(s.charAt(1)); // e
console.log(s[1]); // e
```

## startsWith() and endsWith()

```ruby
let image = "photo.png";
console.log(image.startsWith("photo")); // true
console.log(image.startsWith("png")); // false
console.log(image.endsWith("photo")); // false
console.log(image.endsWith("png")); // true
```


