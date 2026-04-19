# Hashing

hashing means 
```
store data so i can find it fast
```

#### Set and Map method is used to store data.

Example :
arr = [3, 2, 4]
target = 6

```
**Step 1** :

current = 3
need = 6 - 3 = 3

👉 “Did I already see 3?” → ❌ no
👉 store 3

**Step 2** :

current = 2
need = 6 - 2 = 4

👉 seen 4? ❌
👉 store 2

**Step 3** :

current = 4
need = 6 - 4 = 2

👉 seen 2? ✅ YES

👉 ANSWER FOUND
```

### ❓ Problem :
```
"aabbc"
Find first character that appears only once
```

solve manually :

1. a -> 2 times
2. b -> 2 times
3. c -> 1 times (found)

why map? because we need to store count.

### 🔥 Example 1: Two Sum (UNSORTED)

```
❓ Problem
let arr = [7, 11, 15, 2]
let target = 9

function twosum(arr , target){
  let map = new Map();

  for (let i = 0; i < arr.length; i++) {

    let need = target - arr[i];

    if(map.has(need)){
      return [map.get(need) , i];
    }

    map.set(arr[i] , i);
  }
}

console.log(twosum(arr , target));
```

#### Example 2: Count Frequency

```ruby
❓ Problem
"aabbc"
Find first character that appears only once

let str = "aabbc";

function countfreq(str) {
  let map = new Map();

  for (let char of str) {
    map.set(char , ((map.get(char) || 0) + 1))
  }

  return map;
}

console.log(countfreq(str));
```

#### 🔥 Example 3: First Non-Repeating Character
❓ Problem
"aabbcdd"

👉 Output: c

```ruby
function nonRepeat(str) {
  const map = new Map();

  for (let char of str) {
    map.set(char , (map.get(char) || 0) + 1);
  }

  for(let char of str) {
    if (map.get(char) === 1) {
      return char;
    }
  }
}

console.log(nonRepeat(str));
```

#### 🔥 Example 4: Contains Duplicate
❓ Problem
[1,2,3,1]

```ruby
let arr = [1,2,3,5,1,2];

function duplicate(arr){
  let set = new Set();

  for (let elm of arr) {
    if(set.has(elm)){
      return true;
    }
    set.add(elm);
  }

  return false;
}

console.log(duplicate(arr));
```

#### 🔥 Example 5: Longest Consecutive Sequence
❓ Problem
[100,4,200,1,3,2]

👉 Output: 4 → (1,2,3,4)

```ruby
function longestConsequence(arr){
  let set = new Set();

  for (let elm of arr) {
    if(!set.has(elm)) set.add(elm);
  }

  console.log(set);

  let sortedarr = [...set].sort((a,b) => a-b);
  console.log(sortedarr);

  let count = 1;
  let max = 1;

  for (let i = 1; i < sortedarr.length; i++) {
    if(sortedarr[i] === sortedarr[i-1] + 1) {
      count++;
    }

    max = Math.max(max,count);
  }

  console.log(max);
}

longestConsequence(Input2);
```
