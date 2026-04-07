# JavaScript Spread and Rest Operators

The **...** syntax are using in javascript in two distint purposes depending on context.

+ __Spread Operator__ : Expands iterable elements. 

+ __rest Operator__ : collect remaining elements in to a structure.

## 1. Spread Operator

spread operator expands elements from an array,object,string in to a individual elements.

### 1.1 Array Expansion

```ruby
const arr = [1,2,3];
console.log(...arr); // 1 2 3
```

### 1.2 Copying Arrays (Shallow Copy)

```ruby
const arr = [1,2,3];
const copy = [...arr];
copy.push(4);

console.log(...arr); // 1 2 3
console.log(...copy); // 1 2 3 4
```

### 1.3 Merging Arrays

```ruby
const arr1 = [1,2,3];
const arr2 = [4,5,6];
const merged = [...arr1 , ...arr2];

console.log(...merged);
```

### 1.4 Object Cloning and Override

```ruby
const obj = {
    oname : "abc",
    age : 18
}

const updated = {
    ...obj,
    age : 28
}

console.log(updated); // { oname: 'abc', age: 28 }
```

### 1.5 Conditional Properties

```ruby
let isAdmin = false;

const user = {
    name : "supriyo",
    ...(isAdmin && {role : "admin"})
}
console.log(user); // { name: 'supriyo' }
```

### 1.6 Nested Object Pitfall (IMPORTANT)

```ruby
const user = { name: "Supriyo", address: { city: "Kolkata" } };
const copy = {...user};
copy.address.city = "delhi";
console.log(user.address.city); // delhi
console.log(copy.address.city); // delhi
```

### 1.7 Spread with Strings

```ruby
const str = "hello";

const chars = [...str];
// ['h', 'e', 'l', 'l', 'o']
```


### 1.8 Removing Duplicates (Set + Spread)

```ruby
const nums = [1, 2, 2, 3, 3];

const unique = [...new Set(nums)];
// [1, 2, 3]
```