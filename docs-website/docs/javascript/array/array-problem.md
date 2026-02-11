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
