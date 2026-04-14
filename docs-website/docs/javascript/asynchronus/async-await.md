# JavaScript Async / Await

Async/Await is a syntactic sugar over promises. it makes asynchronus code look synchronus.

## 1. async Function

Async functions always returns a promise

```ruby
async function greet (){
    return "Hello"
}

greet().then(console.log); // Hello
```

## 2. await Keyword

await pauses xecution until the promise resolves.

```ruby
function getData() {
    console.log("Inside");
    return new Promise((resolve, reject) => {
        setTimeout(()=> resolve("Data received"), 300);
    })
}

async function fetchData() {
    console.log("Start");
    const res = await getData();
    console.log(res);
    console.log("End");
}

fetchData();
```

Output :

```
Start
Inside
Data received
End
```

## 3. Using try/catch

```ruby
async function example() {
    try {
        let data = await Promise.reject("Error occured");
        console.log(data);
    } catch (error) {
        console.log(error);
    }
}

example() // Error occured
```

#### Without try/catch

example().catch(console.log);

### ❌ Sequential (Slow)

```
await example1();
await example2();
```

### ✅ Parallel (Fast)

```
Promise.all([
    example1(),
    example2()
])
```

Example :

```ruby
async function example1() {
    try {
        let data = await Promise.resolve("Hello");
        console.log(data);
    } catch (error) {
        console.log(error);
    }
}

async function example2() {
    try {
        let data = await Promise.resolve("world");
        console.log(data);
    } catch (error) {
        console.log(error);
    }
}

await example1();
await example2();

Promise.all([
    example1(),
    example2()
])
```

## 4. Microtask Behavior

```ruby
async function test() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}

console.log("Start");
test();
console.log("End");
```

Output :
```
Start
A
End
B
```

### 5. ❓ Does an async function always return a Promise?

Yes, an async function always returns a Promise, even if you return a normal value.

🧪 Example
```
async function test() {
  return 10;
}
```

### 6. ❓ Difference between async/await and Promise.then?

+ __async/await__ : synchronus like code, sequential by default, easier error handling with try/catch.
+ __Promise.then__ : chain based syntax, error handled using .catch().
