# Advanced Questions

## 1. 🔥 QUESTION 1 (Classic)

```ruby
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

#### Output :

```
A
D
C
B
```

## 2. 🔥 QUESTION 2 (Nested Promise)

```ruby
Promise.resolve().then(() => {
  console.log("A");
  Promise.resolve().then(() => console.log("B"));
});

console.log("C");
```

#### Output :

```
C
A
B
```

## 3. 🔥 QUESTION 3 (await + Promise)

```ruby
async function test() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}

console.log("C");
test();
console.log("D");
```

#### Output :

```
C
D
A
B
```

## 4. 🔥 QUESTION 4 (Multiple awaits)

```ruby
async function test() {
  console.log("1");
  await Promise.resolve();
  console.log("2");
  await Promise.resolve();
  console.log("3");
}

test();
console.log("4");
```

#### Output :

```
1
4
2
3
```

## 5. 🔥 QUESTION 5 (setTimeout vs Promise chain)

```ruby
setTimeout(() => console.log("Timeout"), 0);

Promise.resolve()
  .then(() => {
    console.log("P1");
  })
  .then(() => {
    console.log("P2");
  });
```

#### Output :

```
P1
P2
Timeout
```

## 6. 🔥 QUESTION 6 (IMPORTANT EDGE CASE)

```ruby
Promise.resolve().then(() => {
  console.log("A");
  setTimeout(() => console.log("B"), 0);
});

console.log("C");
```

#### Output :

```
C
A
B
```

## 7. 🔥 QUESTION 7 (INTERVIEW KILLER)

```ruby
console.log("Start");

setTimeout(() => {
  console.log("Timeout 1");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise 1");
  setTimeout(() => console.log("Timeout 2"), 0);
});

Promise.resolve().then(() => console.log("Promise 2"));

console.log("End");
```

#### Output :

```
Start
End
Promise 1
Promise 2
Timeout 1
Timeout 2
```

## 8. 🔥 QUESTION 8 (CHAIN + ERROR)

```ruby
Promise.resolve()
  .then(() => {
    throw "Error!";
  })
  .then(() => console.log("A"))
  .catch(() => console.log("B"))
  .then(() => console.log("C"));
```

#### Output :

```
B
C
```

## 9. 🔥 QUESTION 9 (VERY TRICKY)

```ruby
async function test() {
  console.log("A");
  await null;
  console.log("B");
}

Promise.resolve().then(() => console.log("C"));

test();

console.log("D");
```

#### Output :
await null = await Promise.resolve(null)

```
A
D
C
B
```

## 10. 🔥 QUESTION 10 (ULTRA IMPORTANT)

```ruby
setTimeout(() => console.log("Timeout"), 0);

(async () => {
  console.log("Async Start");
  await Promise.resolve();
  console.log("Async End");
})();

Promise.resolve().then(() => console.log("Promise"));

console.log("End");
```

#### Output :

```
Async Start
End
Async End
Promise
Timeout
```

