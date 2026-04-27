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

## Deep Copy

Rule to follow :

+ if it is primitive then copy directly
+ if object/array then creates a new one and go deeper.

#### Example 1 :

```ruby

const user = {
  name: "supriyo",
  age :28,
  location: {
    dis : "kolkata",
    pin : 700060,
    area : ["behala" , "parnashree"]
  }
}


function deepcopy(obj) {
  if(obj === null || typeof obj !== 'object') {
    return obj;
  }

  let copy = Array.isArray(obj) ? [] : {};

  for(let key in obj){
    copy[key] = deepcopy(obj[key]);
  }

  return copy;
}

let copy = deepcopy(user);

copy.location.pin = 700038

console.log(copy);

console.log(user);
```

```ruby
let copy2 = structuredClone(user)
copy2.location.pin = 700051
copy2.location.area.push("ajanta");
console.log(copy2);
console.log(user);
```

```ruby
function deepcopyMap(obj , map = new WeakMap()) {
    if(obj === null || typeof obj !== 'object') return obj;
    if(map.get(obj)) return map.get(obj);

    let copy = Array.isArray(obj) ? [] : {};
    map.set(obj , copy);

    for (let key in obj) {
      copy[key] = deepcopyMap(obj[key] , map);
    }

    return copy;
}

let copy3 = deepcopyMap(user);

copy3.location.area.push("haldia");

console.log(copy3);
console.log(user);
```


