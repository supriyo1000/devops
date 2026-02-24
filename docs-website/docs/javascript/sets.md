# Javascript Set

Set is a collection of unique values only.
it automatically removes duplicates.

### Example (real understanding)

#### Array:
```
[1, 2, 2, 3, 4, 4, 5]
```

#### Set:
```
{1, 2, 3, 4, 5}
```

## Creating a Set

```ruby
let set = new Set();

set.add(1);
set.add(2);
set.add(2);
set.add(3);

console.log(set); // Set(3) { 1, 2, 3 }

// check 2 is only stored once.
```

## Create directly from array ⭐⭐⭐

```ruby
let arr = [1,2,2,3,4,4,5];

let unique = new Set(arr);
console.log(unique); // Set(5) { 1, 2, 3, 4, 5 }
```

## Why Set Exists (REAL REASON)

#### Problem :

Checking if a value exist in a array :

```
arr.includes(5);
```

Time complexity :
```
O(n)  ← slow for big data
```

Set :

```
set.has(5);
```

Time Complexity :

```
O(1) <= instant lookup
```

## Core Set Methods (You MUST know)

### 1️⃣ add(value)

Adds element.
```ruby
let s = new Set();
s.add(10);
s.add(20);
```

### 2️⃣ has(value) ⭐ MOST IMPORTANT

```ruby
let arr = [1,2,2,3,4,4,5];

let set = new Set(arr);
console.log(set.has(5)); // true
console.log(set.has(7)); // false
```

### 3️⃣ delete(value)

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set(arr);

set.delete(4);
console.log(set); // Set(4) { 1, 2, 3, 5 }
```

### 4️⃣ size

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set(arr);
console.log(set.size); // 5
```

### 5️⃣ clear()

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set(arr);
set.clear(); // Set(0) {}
```

## Converting Set ↔ Array (VERY IMPORTANT)

### Set → Array

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set(arr);
let array = [...set];
console.log(array); // [1,2,3,4,5]
```

### Array → Set

```
let set = new Set(arr);
```

This line alone is a famous interview trick:

```ruby
let unique = [...new Set(arr)];
console.log(unique); // // [1,2,3,4,5]
```

(Remove duplicates in 1 line)

## Looping a Set

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set(arr);

for ( let num of set) {
    console.log(num);
}
```

## Where Sets are Used (VERY IMPORTANT)

### 1. Remove duplicates

```ruby
let arr = [1,2,2,3,4,4,5];
let unique = [...new Set(arr)];
console.log(unique);
```

### 2. Detect duplicates

```ruby
let arr = [1,2,2,3,4,4,5];
let set = new Set();

for(let num of arr) {
    if(set.has(num)) console.log(true);
    set.add(num);
} // true true
```

### 3. Two Sum using Set

Array:
```
[2,7,11,15], target = 9
```

