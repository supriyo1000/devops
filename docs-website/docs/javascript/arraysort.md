# Array.sort()

Array.sort is used to modify the array and it mutates the original array and returns the array.

### By default javascript sorts as string, not numbers.

```ruby
const arr = [10, 2, 30, 4];
arr.sort();
console.log(arr); // [10, 2, 30, 4]
```

because internally js converts numbers to string.
```
"10", "2", "30", "4"
```

JS compare character by character, not numeric value.

### How dictionary comparison works

JS compares the FIRST letter only first.

#### Look carefully :

|Value	| First Character|
|-------|----------------|
|"10"	| "1"|
|"2"	| "2"|
|"30"	| "3"|
|"4"	| "4"|

Now sorting alphabetically:
```
"1" < "2" < "3" < "4"
```

So order becomes:
```
"10", "2", "30", "4"
```

Example2 :

```ruby
const arr = [10,5, 65, 77, 30, 4];
arr.sort();
console.log(arr); // [ 10, 30, 4, 5, 65, 77 ]
```

### Custom Compare Function (The REAL sort)

To correctly sort numbers we must give a compare function.
```ruby
const arr = [10, 2, 30, 4];
arr.sort((a, b) => a - b); 
log(arr) // [2,4,10,30]
```

## How Compare Function Works (Core Concept 🧠)

JS internally checks:
```
(a, b) => result
```

|Return value	| Meaning|
|---------------|--------|
|negative	| a comes before b|
|positive	| b comes before a|
|0	| keep same order|


### Ascending Order
```
arr.sort((a,b)=>a-b);
```

### Descending Order
```
arr.sort((a,b)=>b-a);
```

## 2️⃣ Array.reverse()

Reverse the array in place and mutates the original array.
```ruby
const arr = [1,2,3,4];
arr.reverse();
console.log(arr); //[4,3,2,1]
```

>[!NOTE]
> Reverse does not sort.it just flip the position.

## 3️⃣ Array.prototype.toSorted() (Modern JavaScript ⭐)
 
toSorted() sorts without changing the original array(immutable);

```ruby
const arr = [10,5, 65, 77, 30, 4];
let newArr = arr.toSorted((a,b)=>a-b);
console.log(newArr); // [ 4, 5, 10, 30, 65, 77 ]
console.log(arr); // [10,5, 65, 77, 30, 4]
```

## 4️⃣ Array.prototype.toReversed()

It reverse but keeps the original array.

## 5️⃣ Sorting Objects (Most Important Real-World Skill 🔥)

```ruby
const users = [
  {name:"Rahul", age:25},
  {name:"Amit", age:20},
  {name:"Sohan", age:30}
];
```

### sort by age
```
users.sort((a,b)=> a.age -b.age );
log(users);

//  [
  { name: 'Amit', age: 20 },
  { name: 'Rahul', age: 25 },
  { name: 'Sohan', age: 30 }
]
```

### sort by name
```
const users = [
  {name:"Rahul", age:25},
  {name:"Amit", age:20},
  {name:"Sohan", age:30}
];

users.sort((a,b)=> a.name.localeCompare(b.name) );
console.log(users);

// [
  { name: 'Amit', age: 20 },
  { name: 'Rahul', age: 25 },
  { name: 'Sohan', age: 30 }
]
```

### Why localeCompare?

Because **"Z" < "a"** issues happen in normal comparison.

## 8️⃣ Math.min() and Math.max()

They don’t accept arrays directly.

#### ❌ Wrong :
```
Math.max([1,2,3])
```

#### ✔ Correct (spread operator):
```ruby
const arr=[1,2,3,4];

Math.max(...arr); // 4
Math.min(...arr); // 1
```

```ruby
const arr = ["banana", "apple", "cherry"];

console.log(Math.max(...arr)); // NAN
console.log(Math.min(...arr)); // NAN
```