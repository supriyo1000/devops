# Javascript Destructuring

## What is Destructuring?
Destructuring is a taking values out of array and objects and putting them into variables.

Instead of ugly javascript code :

```ruby
const user = {
  name: "Supriyo",
  age: 24,
  city: "Kolkata"
};

const name = user.name;
const age = user.age;
const city = user.city;
```

We can do 

```ruby
const user = {
  name: "Supriyo",
  age: 24,
  city: "Kolkata"
};

const {name,age,city} = user;

console.log(name); // Supriyo
console.log(age); // 24
console.log(city); // Kolkata
```

## 2️⃣ Array Destructuring

Old JS

```ruby
const fruits = ["apple", "banana", "mango"];

const first = fruits[0];
const second = fruits[1];
const third = fruits[2];
```

Modern javascript

```ruby
const fruits = ["apple", "banana", "mango"];

const [first , second , third] = fruits;
console.log(first); // "apple"
console.log(second); // "banana"
console.log(third); // "mango"
```

### Skipping Values

```ruby
const numbers = [10, 20, 30, 40];

const [a , , c] = numbers;
console.log(a , c); // 10 30
```

### Default Values

```ruby
const colors = ["red"];

const [primary , secondary = "blue"] = colors;

console.log(primary); // "red"
console.log(secondary); // "blue"
```

### Swapping Variables (VERY IMPORTANT INTERVIEW QUESTION)

Before destructuring:

```ruby
let a = 5;
let b = 10;

let temp = a;
a = b;
b = temp;

console.log(a , b); // 10 5
```

Modern JS :

```ruby
let a = 5;
let b = 10;

[a , b] = [ b, a]

console.log(a , b); // 10 5
```

## 3️⃣ Object Destructuring (MOST IMPORTANT)

```ruby
const user = {
  name: "Supriyo",
  age: 24,
  city: "Kolkata"
};

const { name, age } = user;

console.log(name); // "Supriyo"
console.log(age); // 24
```

### Renaming Variables
Very very important in React & APIs.

```ruby
const user = {
  name: "Supriyo",
  age: 24
};

let {name : userName , age : myAge} = user;
console.log(userName , myAge); // Supriyo 24
```

### Default Value in Object

```ruby
const user = {
  name: "Supriyo",
  age: 24
};

const { name , age , city = "Kolkata"} = user;

console.log(name , age , city); // Supriyo 24 Kolkata
```

### Nested Object Destructuring (🔥 Very Important)

```ruby
const user = {
  name: "Supriyo",
  address: {
    city: "Kolkata",
    pin: 700001
  }
};

const { name , address : {city , pin}} = user;

console.log(name , city , pin); // Supriyo Kolkata 700001
```

## 4️⃣ Destructuring in Functions (REAL WORLD USE)

Without destructuring

```ruby
const user = {
  name: "Supriyo",
  address: "Kolkata"
};

function getUser(user) {
    console.log(user.name);
    console.log(user.address);
}
```

With destructuring

```ruby
const user = {
  name: "Supriyo",
  address: "Kolkata"
};

function getUser({name , address}) {
    console.log(name);
    console.log(address);
}

getUser(user);

// Supriyo
// Kolkata
```

## 5️⃣ Destructuring in API Responses (Super Practical)

```ruby
const response = {
  status: 200,
  data: {
    token: "abcd123",
    email: "supriyo@gmail.com"
  }
};

const {data : {token , email}} = response;
console.log(token , email);
```

## 6️⃣ Rest Operator with Destructuring

Example 1 :

```ruby
const user = {
  name: "Supriyo",
  age: 24,
  city: "Kolkata"
};

const { name , ...rest} = user;
console.log(rest); // { age: 24, city: 'Kolkata' }
```

Example 2 :

```ruby
const response = {
  status: 200,
  data: {
    token: "abcd123",
    email: "supriyo@gmail.com"
  }
};

const { status , data : {...rest}} = response;
console.log(rest); // { token: 'abcd123', email: 'supriyo@gmail.com' }
```
