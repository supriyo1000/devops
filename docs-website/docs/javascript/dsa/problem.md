# 🚀 20 JavaScript Interview Questions


### 🟢 LEVEL 1 (Core Fundamentals)

```ruby
1. Type Conversion

Convert:

"123" → number
```
```ruby
2. Boolean Conversion

What will be the output?

Boolean("")
Boolean("0")
Boolean(0)
Boolean([])
```
```ruby
3. Implicit Coercion

Explain output:

"5" - 2
"5" + 2
true + 1
false + "1"
```
```ruby
4. Array → String

Convert:

[1, 2, 3] → "1,2,3"
```
```ruby
5. String → Array

Convert:

"hello" → ["h","e","l","l","o"]
```


### 🟡 LEVEL 2 (map / filter / reduce Basics)

```ruby
6. map()

Double all numbers:

[1,2,3,4] → [2,4,6,8]
```
```ruby
7. filter()

Get even numbers:

[1,2,3,4,5,6] → [2,4,6]
```
```ruby
8. reduce()

Find sum:

[1,2,3,4] → 10
```
```ruby
9. reduce() – max

Find max number:

[10, 5, 20, 8]
```
```ruby
10. map + filter
[1,2,3,4,5]

👉 Double only even numbers → [4,8]
```


### 🟠 LEVEL 3 (Intermediate Logic)

```ruby
11. Flatten Array
[1, [2,3], [4,[5]]] → [1,2,3,4,5]
```

```ruby
12. Count Occurrences (reduce)
["a","b","a","c","b","a"]
→ {a:3, b:2, c:1}
```

```ruby
13. Remove Duplicates
[1,2,2,3,4,4]
→ [1,2,3,4]
```

```ruby
14. Object → Array
{a:1, b:2}
→ [["a",1],["b",2]]
```

```ruby
15. Array → Object
[["a",1],["b",2]]
→ {a:1, b:2}
```


### 🔵 LEVEL 4 (Async / Promise Basics)

```ruby
16. Promise Creation

Create a promise that resolves after 2 seconds with "done"
```

```ruby
17. Promise Chaining

What is output?

Promise.resolve(5)
  .then(x => x * 2)
  .then(x => x + 1)
  .then(console.log);
```

```ruby
18. async/await

Convert this to async/await:

fetchData().then(res => console.log(res));
```

```ruby
19. Parallel Execution

Run 3 promises in parallel and get all results
```

```ruby
20. Error Handling

Handle error using async/await:

await fetchData()
```


## 🚀 20 Advanced JavaScript Problems (Interview Level)


### 🔥 LEVEL 1 — Event Loop & Async Behavior

```ruby
1. Event Loop Order

What is the output?

console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

```ruby
2. Microtask vs Macrotask
setTimeout(() => console.log("timeout"));

Promise.resolve()
  .then(() => console.log("promise1"))
  .then(() => console.log("promise2"));

console.log("end");
```

```ruby
3. Async/Await Execution
async function test() {
  console.log("1");
  await Promise.resolve();
  console.log("2");
}
test();
console.log("3");
```

```ruby
4. Nested Promises
Promise.resolve()
  .then(() => {
    console.log("A");
    return Promise.resolve();
  })
  .then(() => console.log("B"));

console.log("C");
```


### ⚡ LEVEL 2 — Closures & Scope

```ruby
5. Closure Trap
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}

👉 Fix it to print: 0 1 2
```

```ruby
6. Private Variable

Create a function:

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2

👉 Without exposing the internal variable
```

```ruby
7. Function Currying

Convert:

add(1,2,3)

👉 Into:

add(1)(2)(3)
```

```ruby
8. Memoization

Write a memoized version of:

function slowFn(n) { ... }
```


### 🧠 LEVEL 3 — Advanced Array Transformations

```ruby
9. Custom map()

Implement your own:

Array.prototype.myMap
```

```ruby
10. Custom reduce()

Implement:

Array.prototype.myReduce
```

```ruby
11. Group By (Important)
[
 {age:20}, 
 {age:30}, 
 {age:20}
]

👉 Output:

{
 20: [...],
 30: [...]
}
```

```ruby
12. Deep Flatten (Recursive)
[1,[2,[3,[4]]]]
→ [1,2,3,4]
```


### 🔄 LEVEL 4 — Promises Deep Dive

```ruby
13. Promise.all Polyfill

Implement:

myPromiseAll([p1, p2, p3])
```

```ruby
14. Promise.race Polyfill
```

```ruby
15. Retry Promise

Retry API call 3 times before failing
```

```ruby
16. Limit Concurrency (🔥 very important)

Run 10 async tasks but only 3 at a time
```


### 🧩 LEVEL 5 — Real Interview Traps

```ruby
17. This Binding
const obj = {
  name: "JS",
  getName: function() {
    return this.name;
  }
};

const fn = obj.getName;
console.log(fn());
```

```ruby
18. Object Reference
let a = {x:1};
let b = a;
b.x = 5;
console.log(a.x);
```

```ruby
19. Shallow vs Deep Copy

Explain difference and implement deep clone
```

```ruby
20. Tricky Equality
console.log([] == ![]);
```


### 💀 BONUS — HARDCORE (Only for Top 1%)

```ruby
🔥 Debounce Implementation
debounce(fn, delay)
```

```ruby
🔥 Throttle Implementation
```

```ruby
🔥 Build your own Promise (core concept)
```

```ruby
🔥 Implement EventEmitter
```