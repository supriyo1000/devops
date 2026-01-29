## Array.forEach()
forEach is used to perform side effetcs like loging,updating value,trigering an action and dont need to return any value.

```ruby
// Example 1: Counting errors
const apiResponses = [
  { id: 1, status: "success" },
  { id: 2, status: "error" },
  { id: 3, status: "success" },
  { id: 4, status: "error" }
];

let errorCount = 0;

apiResponses.forEach(response => {
  if (response.status === "error") {
    errorCount++;
    console.error(`Error found in request ${response.id}`);
  }
});

console.log("Total errors:", errorCount);
// Total errors: 2

```
**WHY forEach() IS PERFECT HERE**
1. we are not transforming data.
2. we do not return a new array.
3. we are performing side effects only.

```ruby
// Example 2: forEach with index
const nums = [1, 2, 3];

nums.forEach((num, index) => {
  console.log(`Index ${index} has value ${num}`);
});

```

### ❓ Why not use forEach() instead of map()?

forEach() → side effects, no return<br/>
map() → transforms data, returns new array

### ❓ Can we break out of forEach()?

❌ No<br/>
break / return won’t stop iteration<br/>
Use for...of or some() / every() instead

### ❓ Does forEach() support async/await?
❌ No (common trap)

// ❌ Wrong<br/>
arr.forEach(async item => {<br/>
&nbsp;&nbsp;await fetchData(item);<br/>
});

Use for...of instead.

### ❓ Why not use forEach() for async operations?
// ❌ Wrong<br/>
```
users.forEach(async user =>{
  await saveUser(user);
});
```
Because:<br/>

forEach() does not wait for await<br/>
errors can’t be caught properly

## Array.map()
creates a new array by applying a callback function to each element of an array.

**👉 map() must return something for every element.**

```ruby
const users = [
  { id: 1, firstName: "Supriyo", lastName: "Chatterjee" },
  { id: 2, firstName: "Amit", lastName: "Das" }
];

const formattedUsers = users.map((user)=>{
    return {
        id: user.id,
        fullName: `${user.firstname} ${user.lastname}`
    }
})
console.log(formattedUsers);

// [
    {id:1, fullName: 'Supriyo chatterjee'},
    {id:2, fullName: 'Amit Das'}
]
```

**🧠 WHY map() IS PERFECT HERE**

+ ✔ You are transforming data
+ ✔ You need a new array
+ ✔ Original data remains unchanged
+ ✔ Perfect for React rendering

### ❓ What happens if you forget return inside map?
```
[1, 2, 3].map(n => {
  n * 2;
});
```
Answer:
Returns [undefined, undefined, undefined]

### ❓ Does map() skip empty slots?
```
const arr = [1, , 3];
arr.map(x => x * 2);
```
Answer:
Yes — empty slots remain empty.

### ❓ Can map be chained?
✅ Yes<br/>
arr.map().filter().reduce();

## Array.filter()
creates a new array containing only those element that pass the test in the callback function.

Example : Return only active users
```ruby
const users = [
  { id: 1, name: "Supriyo", active: true },
  { id: 2, name: "Amit", active: false },
  { id: 3, name: "Rohit", active: true }
];

let active = users.filter(user => user.active === true);

log(active);

// [
//  { id: 1, name: "Supriyo", active: true },
//  { id: 3, name: "Rohit", active: true }
 ];
```
### ❓ What happens if callback returns non-boolean?
[0, 1, 2].filter(n=>n);

Answer:<br/>
JavaScript uses truthy / falsy values.<br/>
Result → [1, 2]

### ❓ Does filter() stop early?

❌ No — it checks every element.

### ❓ Can filter change elements?

❌ No — it only includes or excludes elements.

```ruby
const names = users.filter(user => user.active === true).map(user=> user.name);

log(names);
// ["Supriyo" , "Rohit"]
```

## Array.reduce()
Executes a reducer function on each element of the array and return a single accumulated value. Does not mutates the original array

### syntax
```ruby
arr.reduce((accumulator , current) => {
    return updatedAccumulator;
}, initialValue);
```
accumulator = memory<br/>
currentValue = current element<br/>
returns updated memory each time

example 1 (Calculate total price in a cart.)
```ruby
const prices = [100, 200, 300];
let total = prices.reduce((acc , curr)=>{
    acc += curr
    return acc;
}, 0)
log(total); // 600
```

example 2 (Count number of active users.)
```ruby
const users = [
  { name: "A", active: true },
  { name: "B", active: false },
  { name: "C", active: true }
];

let activeusers = users.reduce((acc , curr) => {
    if (curr.active) acc++;
    return acc
} , 0)
log(activeusers); // 2
```

### ❓ What happens if initialValue is NOT provided?
```ruby
let x = [1, 2, 3].reduce((acc, curr) => acc + curr);
log(x); //6
```
+ when initial value provided accumulator starts for initial value and current is array[0] eg. 1;
+ when initial value does not provide accumutaor starts from first element and current will be array[1];

```ruby
let x = [].reduce((acc, curr) => acc + curr);
console.log(x); // error

let x = [].reduce((acc, curr) => acc + curr , 0);
console.log(x); // 0
```

### ❓ Can reduce return an array?

✅ YES
```ruby
const evens = [1, 2, 3, 4].reduce((acc, n) => {
  if (n % 2 === 0) acc.push(n);
  return acc;
}, []);
log(evens); //[2,4]
```

## Array.every()
Returns true if all the elements satisfy a condion, if 1 element does not satisfy then return false.

does not mutates original array.

### Stops Early
✅ yes (as soon as one element fail)

example 1 (form validation)
```ruby
const inputs = [
  { value: "john", valid: true },
  { value: "doe", valid: true (if false)},
  { value: "email", valid: true }
];
isvalid = inputs.every(item => item.valid);
log(isvalid); // true // if false return false
```

### ❓ What does every() return for an empty array?
```
[].every(x => x > 0);
```

✅ Returns true

📌 Reason:

“All elements satisfy the condition” is vacuously true when there are no elements.

## Array.some()
Returns true if atleast 1 element satisfy a condition.

### Stops Early
✅ yes (as soon as 1 element satisfy condition)

example 1 (Permission check)

```ruby
const userRoles = ["user", "editor"];
let hasPermission = userRoles.some(user => user === "admin");
log(hasPermisiion); //false
```

### ❓ What does some() return for an empty array?
```ruby
[].some(x => x > 0);
```
✅ Returns false

|Array.every() | Array.some()|
|--------------|-------------|
|return true if all elements pass condition| return true if 1 element pass the condition|

```ruby
const arr = [0, 1, 2];

arr.some(x => x); // true (as soon as it get 1 return true)
arr.every(x => x); // false (as soon as it get 0 return false)
```
javascript falsy values:
+ 0
+ " "
+ null
+ nan
+ undefined
+ []
+ {}

javascript truthy values:

+ 1
+ rest..

```ruby
const arr = [];

arr.some(x => x);  // false (there is no element to test)
arr.every(x => x); // true (“All elements satisfy the condition” is vacuously true when there are no elements.)
```

## Array.flatMap()
this methods first maps all the elements of an array then creating a new array by flatenning the array by one level.

```ruby
const users = [
  { name: "A", skills: ["JS", "React"] },
  { name: "B", skills: ["Node", "Docker"] }
];

let allSkills = users.flatMap(user => user.skills);
log(allSkills); // [ 'JS', 'React', 'Node', 'Docker' ]
```

```ruby
let x = [1, [2, [3]]].flatMap(x => x);
console.log(x); // [1,2,[3]]
```
```ruby
let x = [[[]]].flatMap(x => x);
console.log(x); // [[]]
```
👉 flatMap() flattens ONLY one level.

### ❓ How is flatMap() different from map().flat()?

Answer:

flatMap() → one pass (faster)

map().flat() → two passes

## ❓ Does flatMap() skip empty slots?
```ruby
[1, , 3].flatMap(x => [x]);

// [1,3]
```
✅ Yes — empty slots are skipped.

### ❓ result?
```ruby
[1, 2].flatMap(x => [x, x * 10]);
// [1 , 10 , 2 , 20]
```

## Array.keys()
Returns an iterator containing the keys (indexes) of an array. when you want only indexes.

### Return Value

✅ Iterator (NOT an array)

```ruby
const steps = ["Login", "Verify", "Dashboard"];

for(const index of steps.keys()){
    log(index);
}
// 0
// 1
// 2
```
```ruby
for (const value of steps.keys()) {
  console.log(value); // this is index, not value
}
// 0
// 1
// 2
```
```ruby
const steps = [];

for(const index of steps.keys()){
    console.log(index);
} // nothing
```
### ❓ Does keys() skip empty slots?
```ruby
[ , , "a"].keys();
// 0
// 1
// 2
```
✅ No — it includes all indexes.

### ❓ Difference between keys() and Object.keys(array)?

+ keys() → iterator
+ Object.keys() → array of string indexes

## Array.entries()
Returns an iterator containing the key,value pair (index , value) of an array. when you want [key,value] pair.

```ruby
const steps = ["Login", "Verify", "Dashboard"];

for(const index of steps.entries()){
    console.log(index);
}
// [ 0, 'Login' ]
// [ 1, 'Verify' ]
// [ 2, 'Dashboard' ]
```
```ruby
const tech = ["JS", "React", "Node"];

for (const item of tech.entries()) {
  console.log(item.index); // ❌ undefined
}
```
👉 Each item is an array: [index, value]

### ❓ What does this return?
```ruby
[..."abc"].entries();

Answer:
👉 Iterator of:

[0, "a"], [1, "b"], [2, "c"]
```

### ❓ What does this return?
```ruby
[...["x", "y"].keys()];
//0 1
```

```ruby
[...["x", "y"].entries()];
// [ 0, 'x' ] [ 1, 'y' ]
```

## Array.Spread()
To expand elements of an array into another array.it does not mutates original array.

```ruby
const original = [1, 2, 3];
const copy = [...original];
copy.push(4);
log(original) // [1,2,3]
log(copy) // [1,2,3,4]
```

```ruby
const frontend = ["JS", "React"];
const backend = ["Node", "Docker"];
const fullstack = [...frontend , ...backend];
log(fullstack) // ["JS", "React", "Node", "Docker"]
```

```ruby
const a = [{ x: 1 }];
const b = [...a];
b.push({ x: 2});

log(a) // [{x:1}]
log(b) // [{ x:1 }, {x:2}]
```
push modifies array structure. so a and b are different array now.

```ruby
const a = [{ x: 1 }];
const b = [...a];
b[0].x = 9;

log(a) // [{x:9}]
log(b) // [{ x:9 }]
```
spreads supports shallow copy. not deep copy.

## Array.rest()
to collect multiple values into single array in functional arguments.

```ruby
function sum(...numbers){
    let total = numbers.reduce((acc , curr)=>{
        acc+= curr
        return acc;
    },0);

    return total;
}
const a = [1,2,3,4];
const b = [5,6];

console.log(sum(...a)); // 10
console.log(sum(...a, ...b, 7)); // 28
console.log(sum(...b)); // 11
```
