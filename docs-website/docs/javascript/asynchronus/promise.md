# JavaScript Promises (Core Async System)

A promise executes an asynchronus operation with 3 states, pending, fullfilled and rejected.it allows chaining using .then() and handle errors using .catch().promise are executed in the microtask queue.

## 1. Why promise exist ?

before promise async code uses callback

```
getUser(id, (user) => { 
    getOrders(user.id, (orders) => { 
        getPayment(orders[0], (payment) => { 
            console.log(payment); 
        }); 
    }); 
});

__Problem__ :

+ calback hell
+ hard to read and debug

promise solve this by making async code chainable and managable.

Promise has 3 states :

1. pending
2. fulfilled
3. rejected

### Creating a promise

```ruby
3. Creating a Promise
const myPromise = new Promise((resolve, reject) => {
  let success = true;

  if (success) {
    resolve("Data received");
  } else {
    reject("Error occurred");
  }
});
```

## 2. Consuming a Promise

```
mypromise.then((result)=>{
    console.log(result);
})

mypromise.catch((error)=> {
    console.error(error);
});

mypromise.finally(()=>{
    console.log("Completed");
})
```

## 3. Promise Chaining (IMPORTANT)

Promise chaining is the process to execute multiple asynchronus operation sequentially, using .then().

```ruby
function step1(){
    return Promise.resolve(10);
}

function step2(data){
    return Promise.resolve(data * 2);
}

function step3(data){
    return Promise.resolve(data + 10);
}

step1()
.then(step2)
.then(step3)
.then((result)=>{
    console.log(result);
})
```

### Error Propagation

```ruby
Promise.resolve()
.then(() => {
    throw "error in step 1";
}).then(()=>{
    console.log("skipped");
}).catch((error)=>{
    console.log(error);
}) // error in step 1
```

## 4. Promise Static Methods

```ruby
Promise.all([
    Promise.resolve(1),
    Promise.resolve(2),
    Promise.resolve(3)
]).then(console.log)  // [1,2,3]
```

+ runs in parallel
+ fail if any promise fail

### 5. ❓ Difference between resolve and reject?

+ __resolve__ : called when operation is successfull, moves promise to fullfilled state, triggers .then().

+ __reject__ : called when operation fail, moves promise to rejected state, triggers .catch().

### 6. ❓ Difference between Promise.all and Promise.race?

+ __Promise.all__ : waits for all promise to be complete and it fails if any promise fail.

+ __promise.race__ : triggers setteled promise, resolve/reject which ever finish first.

```ruby
Promise.race([
    new Promise(res => setTimeout(()=>{res("fast")},2000)),
    new Promise(res => setTimeout(()=>{res("slow")},3000))
]).then(console.log); // fast
```

### 7. ❓ Why do Promises use the Microtask Queue?
because they need to execute as fast as synchronus code complete.

this ensures :
+ faster execution
+ consistent async behaviour

