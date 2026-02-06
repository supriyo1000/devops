# JavaScript Array Search Methods

## Array.indexOf()

Concept :
+ Return index of an element.
+ Starts from first to last.
+ If not found then return -1.

Example :
```ruby
let fruits = ["apple", "banana", "mango"];
console.log(fruits.indexOf("mango")); // 2
console.log(fruits.indexOf("grapes")); // -1
```

## Array.lastIndexOf()

Concept :
+ Starts from last to first index.
+ Return last index of an element.
+ useful when duplicate element.

Example :
```ruby
let numbers = [10, 20, 30, 20, 40];
console.log(numbers.lastIndexOf(20)); // 3
console.log(numbers.lastIndexOf(10)); // 0
```

## Array.includes()

Concept :
+ Returns boolean
+ Best for conditions

Example :
```ruby
let colors = ["red", "green", "blue"];
console.log(colors.includes("green"));
console.log(colors.includes("black"));
```

## Array.find()

Concept :
+ Uses a condition(function).
+ Return first matching element.
+ Stops after first matching element.
+ if not found return undefined.

Example1 :
```ruby
let numbers = [5, 12, 8, 130, 44];
console.log(numbers.find(num => num > 8)); // 12
console.log(numbers.find(num => num > 400)); // undefined
```

Example2 :
```
let arr = [];
console.log(arr.find(item => item > 0)); // undefined
```

## Array.findIndex()

Concept :
+ use a condition(function).
+ It returns index not value.
+ if not found return -1.

Example :
```ruby
let numbers = [5, 12, 8, 130, 44];
console.log(numbers.findIndex(num => num > 8)); // 1
console.log(numbers.findIndex(num => num > 400)); // -1
```