# Javascript Objects

Objects are reference types

```
let a = {x:1 , y:2};
let b = a;
```

both a , b have same object because i do not copy the value , i copied only the reference.

so :

```
b.x = 5;
console.log(a.x) // 5;
```

## Shallow Copy

```
let a = {x:1};
let b = {...a};
b.x = 30;
console.log(a.x) // 1
```

because {...a} creates a new object.

#### Trap :

```ruby

const obj = {
    users : {
        name : "supriyo"
    }
}

const copy = {...obj};
copy.users.name = "komal"
console.log(obj.users.name) // komal
```

because shallow copy only copies the first level value not nested value.


## Object keys

```ruby
const obj = {};
obj[1] = "one";
obj["1"] = "two";
console.log(obj); // { '1': 'two' }
```

why because?
+  object keys are always string or symbol
+ so 1 becomes "1"

```ruby
const obj = {};
obj[1] = "one";
obj["2"] = "two";
console.log(obj); // { '1': 'one', '2': 'two' }
```

### Object properties

```ruby
const obj = {
  "name" : "supriyo",
  "users age" : 28
};

console.log(obj.name); // supriyo
console.log(obj["name"]); // supriyo
console.log(obj["users age"]); // 28
// console.log(obj.users age); // error
```

## Dynamic keys

```ruby
const key1 = "name";
const key2 = "age";

const obj = {
  [key1] : "supriyo",
  [key2] : 28
};

console.log(obj.name); // supriyo
console.log(obj.age); // 28
```

## Object as functions container

```ruby
const obj = {
  "name" : "supriyo",
  greet(){
    console.log("Hello " + this.name);
  }
};

obj.greet(); // Hello supriyo


const obj2 = {
  "name" : "supriyo",
  greet : ()=> {
    console.log("Hello " + this.name);
  }
};

obj2.greet(); // Hello undefined
```

Arrow function does not bind this.

## Add / delete properties

```ruby
const obj = {
  "name" : "supriyo"
};

obj.age = 28;

delete obj.name;
console.log(obj); // { age: 28 }
```

## Check property exists

```ruby
const obj = {
  "name" : "supriyo"
};

obj.age = 28;

console.log("name" in obj); // true
console.log(obj.hasOwnProperty("age")); // true
```

#### Difference 

+ in checks prototype also
+ hasOwnProperty checks own properties

## Iteration (important)

```ruby
const obj = {
  a : 1,
  b : 2
};

console.log(Object.keys(obj)); // [ 'a', 'b' ]

console.log(Object.entries(obj)); // [ [ 'a', 1 ], [ 'b', 2 ] ]

for (let key in Object.keys(obj)) {
  console.log(key); // 0 -> 1
}

for(let keyval in Object.entries(obj)){
  console.log(keyval); // 0 -> 1
}
```

