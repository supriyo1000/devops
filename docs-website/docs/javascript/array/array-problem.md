# 1️⃣ Traversal Pattern (Foundation of Everything)

If you need to check every element its a Traversal problem.

## Find largest number.

Example 1 :
```ruby
const arr = [3,7,2,9,5];
let max = arr[0];

for (let i = 1; i < arr.length; i++) {
    if(arr[i] > max) max = arr[i];
}

console.log(max); // 9
```

Example 2 :

```ruby
const arr = [3,7,2,9,5];
const bignum = arr.reduce((acc , curr)=>{
    if(curr > acc) acc = curr;
    return acc;
}, 0)

console.log(bignum); // 9
```

# 2️⃣ Frequency Counter Pattern 🔥 (VERY VERY IMPORTANT)

Used for:

+ duplicates
+ anagram
+ occurrence count
+ most frequent element

## Find duplicates

Example 1 :
```ruby
let arr = [1,2,3,1,4,2,3];

function findDuplicates(arr){
    let seen = {};
    for( let num of arr ){
        if(seen[num]){
            seen[num] += 1;
            continue;
        }
        seen[num] = 1
    }
}

findDuplicates(arr) // { '1': 2, '2': 2, '3': 2, '4': 1 }
```

Example 2 :
```ruby
let dup = arr.reduce((acc , curr)=> {
    acc[curr]?  acc[curr] += 1 : acc[curr] = 1;
    return acc;
},{});

console.log(dup); // { '1': 2, '2': 2, '3': 2, '4': 1 }
```

## Most Frequent Element

Example 1 :

```ruby
let arr = [1,1,2,3,3,3,4];

let count = arr.reduce((acc , curr)=> {
    acc[curr]?  acc[curr] += 1 : acc[curr] = 1;
    return acc;
}, {});

let max = 0;
let elm = null;

for( let [key , value] of Object.entries(count)){
    if(value > max){
        max = value;
        elm = key;
    }
}

console.log(max); // 3
console.log(elm); // 3
```

Example 2 :

```ruby
let arr = [1,1,2,3,3,3,4,4,4,4];

let freq = {};
let max = 0;
let elm = null;

for(let num of arr){
    freq[num] = (freq[num] || 0) + 1;
    if(freq[num] > max){
        max = freq[num];
        elm = num;
    }
}

console.log(elm); // 4
console.log(max); // 4
```

## Maximum sum of 3 consecutive numbers

You only add 3 numbers at a time that are side-by-side

#### Example 1 :
```ruby
let arr = [2,1,5,1,3,2]

function maxSum(arr,k){
    let window = 0;
    for (let i = 0; i < k; i++) {
        window += arr[i]
    }
    let max = window;

    for (let i = k; i < arr.length; i++) {
        window = window - arr[i-k] + arr[i];
        max = Math.max(max, window);
    }

    return max;
}

console.log(maxSum(arr,3)); // 9
```

#### Example 2 :

```ruby
let arr = [2,1,5,1,3,2]

function maxSum(arr,k){
     let max = arr.map((_,i)=> arr.slice(i , i+k)).filter(item => item.length === k).map(sub => sub.reduce((a,b)=> a+b));
     return Math.max(...max)
}

console.log(maxSum(arr,3)); // 9
```

## Group orders by user.

#### Example :
```ruby
const orders = [
  { orderId: 1, user: "Amit",   product: "Phone",  amount: 20000 },
  { orderId: 2, user: "Rahul",  product: "Laptop", amount: 55000 },
  { orderId: 3, user: "Amit",   product: "Mouse",  amount: 500 },
  { orderId: 4, user: "Rohit",  product: "Keyboard", amount: 1500 },
  { orderId: 5, user: "Rahul",  product: "Charger", amount: 800 }
];

let group = orders.reduce((acc , curr)=>{
    if(!acc[curr.user]){
        acc[curr.user] = []
    }
    acc[curr.user].push(curr)

    return acc;
},{});

console.log(group);
```

```ruby
You are given a sorted array:

[1, 2, 3, 4, 6, 8, 9]

Find if any two numbers add up to:
target = 10


```