# Array Methods

### array.push()

=> add element at the end of array and mutates the original array.

### array.pop()

=> Remove last element from array and mutates original array.

```ruby
let arr = [1, 2, 3];

arr.push(4);   // add at end
console.log(arr); // [1,2,3,4]

arr.pop();    // remove last
console.log(arr); // [1,2,3]

```

### array.shift()

=> Remove first element from array

Mutates original array:
✅ Yes

### array.unshift(1)

=> add element at the first of array 
Mutates original array:
✅ Yes

```ruby
let arr = [2, 3];
let k = arr.shift();
let j = arr.unshift(1);

console.log(k) => [3]
console.log(j) => [1,2,3]
```

### array.join()

+ 👉 Converts array → string
+ 👉 Original array NOT changed

```ruby
let fruits = ["apple", "banana", "mango"];

let result = fruits.join(",");

console.log(typeof(result)); => string
console.log(result); => apple,banana,mango
console.log(fruits); => [ 'apple', 'banana', 'mango' ]
```

### array.concat() 

+ 👉 Merge arrays
+ 👉 Does NOT modify original arrays and Returns a new array by combining existing arrays.

**example-1**
```ruby
let a = [1, 2];
let b = [3, 4];

let c = a.concat(b);
console.log(c); => [ 1, 2, 3, 4 ]
console.log(a); => [ 1, 2]
console.log(b); => [ 3, 4 ]
```
**example-2**
```ruby
let arr1 = [1];
let arr2 = [2];
let arr3 = [3];

let merged = arr1.concat(arr2, arr3);
console.log(merged);
// [1,2,3]

```
### Array.slice()

+ 📌 slice(start, end)
+ 👉 end not included
+ 👉 original unchanged
To extract part of an array without mutation.

```ruby
let arr = [10, 20, 30, 40];

let part = arr.slice(1, 3);
console.log(part); // [20,30]
console.log(arr);  // original unchanged

```

```ruby
let arr = [1, 2, 3, 4, 5];

console.log(arr.slice(2)); => end included
// [3,4,5]

```

```ruby
let arr = [1, 2, 3];

let copy = arr.slice(); // [1,2,3]
copy.push(4);

console.log(arr);  // [1,2,3]
console.log(copy); // [1,2,3,4]

```

### Array.splice()

To add, remove, or replace elements in-place.
=> splice modifies original array

```ruby
//example 1
const arr = [1, 2, 3];
arr.splice(1,1,99);
log(arr) => [1,99,3]
```

```ruby
//example 2
const fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2, 0, "Lemon", "Kiwi");
log(fruits) => ["Banana", "Orange", "Lemon", "Kiwi", "Apple", "Mango"]
```

|slice|splice|
|-----|------|
|does not mutates the original array|mutates the original array|
|extract part of an array| add,remove,delete elements from array|