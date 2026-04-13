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



