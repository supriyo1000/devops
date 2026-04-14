# JavaScript Asynchronous Programming — Layer 1 (Foundation)

## 1. Synchronous Execution

js executes codes line by line, in a single thread.

```
console.log("A");
console.log("B");
console.log("C");
```

This will print

```
A
B
C
```

+ Each line waits for the previous to be complete.
+ Blocking behaviour.

### 🤯 Now real problem

What if something takes time?

Example:

```ruby
function slow() {
    const start = Date.now();
    while (Date.now() - start < 3000) {}
}

console.log("start");
slow();
console.log("end");
```

__Output__

```
start
wait for 3 sec for slow();
end
```

+ ui frezzes
+ button not clickable
+ app hangs

so it needs a solution.

## 2. Asynchronous Execution

To handle asynchronus operation javascript uses :

+ Call stack
+ Web API
+ callback queue (macrotask)
+ microtask queue
+ Event loop

```ruby
console.log("start");

setTimeout(()=>{
    console.log("async task");
},0);

console.log("end");
```

__Output__

```
start
end
async task
```

#### Step by step

1. js sees setTimeOut.
2. js says i dont handle it. send to __browser/node__(called __web Api__) and timer runs there.
3. meanwhile js continue executing next line.

### Call Stack

tracks function execution.

```ruby
function one(){
    console.log("one running");
    two();
}

function two(){
    console.log("two running");
    three();
}

function three(){
    console.log("hello world");
}

one();
```

__Output__

```
one running
two running
hello world
```

## 3. Web APIs (Browser / Node APIs)

These are externals api provided by the environment, not by javascript itself.

Examples :

+ setTimeOut
+ setIntervals
+ DOM events
+ fetch api

## 4. Callback Functions

callback function is a function that is passed as an argument to another function and executes later.

```ruby
function greet (name , callback) {
    console.log("hi", name);
    callback();
}

function saybye() {
    console.log("bye!!!");
}

greet("supriyo" , saybye);
```

### Async Callback Example

```ruby
console.log("start");

setTimeout(()=>{
    console.log("inside callback");
},3000);

console.log("end");
```

__Output__

```
Start
End
Inside callback
```

## 5. Callback Hell (IMPORTANT)
Problem

Callback hell is a situation where multiple nested callbacks make code difficult to read and maintain.

```ruby
setTimeout(()=>{
    console.log("step 1");

    setTimeout(()=>{
        console.log("step 2");

        setTimeout(()=>{
            console.log("step 3");
        }, 1000);
    },1000);
},1000);
```
+ hard to debug.
+ callback hell is the reason **promises** are introduced.

## key mental model

+ Javascript executes synchronusly bydefault.
+ Asychronus are handled by :
    + web apis (browser/node).
    + callback queue.
+ callbacks are bridge between sync and async.

## Common mistakes

+ #### Thinking setTimeout runs immediately.
    + 0ms is also not not immediate.

+ #### Confusing async with parallelism
    + js is single threaded not fully parallel.


## ✅ 1. What is the difference between synchronous and asynchronous code?

+ synchronus code executes line by line, each operation waits for the previous one to complete.
+ asychronus code allows certain task(like api calls , timers) to run in the background without blocking the main thread.

## ✅ 2. How does JavaScript handle async operations if it is single-threaded?

javascript delegates async task to web apis and uses the event loop to execute callbacks later without blocking the main thread.


### Callback Queue

Stores callback from :
+ setTimeout
+ setInterval

### Microtask Queue

+ promise.then

## 6. Event Loop

Event loop is a mechanism in javascript that manage the execution of asynchronus code.js is single-threaded so event loop ensure that after executing the synchronus code, excute the processed queue. first microtask then macrotask.so non-blocking behaviour is achived. 

Continously check **is call stack empty?"

if yes then :

+ Executes all microtask.
+ Then Executes one callback from queue

## Execution Flow Diagram

```ruby
        ┌───────────────┐
        │   Call Stack  │
        └──────┬────────┘
               │
               ▼
        ┌───────────────┐
        │   Event Loop  │
        └──────┬────────┘
               │
   ┌───────────┴───────────┐
   ▼                       ▼
Microtask Queue     Callback Queue
(Promises)          (setTimeout)
```

#### Example

```ruby
console.log("A");

setTimeout(()=>{
    console.log("B");
},2000);

console.log("C");
```

### Execution Steps
```
console.log("A"); -> call stack -> prints "A".

setTimeout --> sent to web api 

console.log("C"); --> print "C"

Timer complete. --> callback goes to callback queue

Event loop --> moves callback to stack

print "B"
```

## 7. Microtask vs Macrotask (CRITICAL)

Microtask and Macrotask are two types of task queue in javascript.

+ __Microtask__ : High priority task that executes immediately after synchronus code. like promise(then , catch) , async/await.

+ __Macrotask__ : low priority task that executes after all synchronus code and microtask are complete. like setTimeout, setInterval, DOM events.

Example :

```ruby
console.log("Start");

setTimeout(()=> console.log("B") ,2000);

Promise.resolve().then(()=> console.log("C"));

console.log("D");
```

### Step-by-Step Execution

```
Start --> printed
setTimeout --> sent to web api --> later to callback queue
promise --> Goes to microtask queue
D --> printed
```

now call stack is empty :
1. Event loop runs all microtask first.
2. Then callback queue

print :

```
C
B
```

### Advanced Example

### ❓ Why do Promises execute before setTimeout ?

Because promise callbacks are placed in the microtask queue, while timeout are placed in the macrotask queue. so javascript always execute macrotask after all microtask are complete.

```ruby
setTimeout(() => console.log("1"), 100);

Promise.resolve().then(() => console.log("2"));

setTimeout(() => console.log("3"), 0);

Promise.resolve().then(() => console.log("4"));
```

Output :

```
2
4
3
1
```

### Nested Microtask Example

```ruby
console.log("A");

setTimeout(()=>{
    setTimeout(()=>{
        console.log("B");
    })

    console.log("C");
},2000);

Promise.resolve().then(()=>{
    Promise.resolve().then(()=> console.log("D"));
    console.log("E");
})

console.log("End");
```

Output :

```
A
End
E
D
C
B
```

### ❓ Does setTimeout(fn, 0) execute immediately?

No. setTimeout are placed in the macrotask queue. so javascript always execute

1. synchronus code
2. all microtask queue
3. macrotask queue