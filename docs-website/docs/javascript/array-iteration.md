### Array.forEach()
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

### Array.map()
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