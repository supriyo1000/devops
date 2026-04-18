# Sliding Window

imagine an window moving over an array.

Example : [2, 1, 5, 1, 3, 2]

window size = 3;

#### 1st window
```
[2, 1, 5]
```

#### 2nd window
```
[1, 5, 1]
```

#### 3rd window
```
[5, 1, 3]
```

Slide → move right.

Instead of recalculating every time, we use previous work.

## Sliding window Idea

Instead of recalculating

```
remove left + add right
```

## FLOW (SUPER IMPORTANT)

+ expand window (right++)
+ check condition
+ if invalid (left--)
+ repeat

## When to use Sliding Window?

+ subarray/substring ?
+ continous element ?
+ max/min/lenght ?

use sliding window.

#### Example-1 : Max Sum Subarray of Size K

```ruby
let array = [2, 1, 5, 1, 3, 2];
let m = 3;

// Find max sum of any 3 elements

function maxsum(arr , k) {
    
    let sum = 0;

    for (let i = 0; i < k; i++) {
        sum = sum + arr[i];
    }

    let max = sum;

    for (let i = k; i < arr.length; i++) {
        sum = sum - arr[i - k] + arr[i];
        max = Math.max(max , sum);
    }

    return max;
}

console.log(maxsum(array , m));
```

#### Example-2 : Longest Substring Without Repeating Characters

```ruby
function uniqueSubstr(str) {
    let left = 0;
    let set = new Set();
    let maxStr = '';

    for (let right = 0; right < str.length; right++) {
        while (set.has(str[right])) {
            set.delete(str[left]);
            left++;
        }

        set.add(str[right]);

        // track longest substring
        if (set.size > maxStr.length) {
            maxStr = str.substring(left, right + 1);
        }
    }

    return maxStr;
}

console.log(uniqueSubstr("abcabcbb"));
```

#### Example-3 : Maximum Average

```ruby
let arr = [1,12,-5,-6,50,3]
let k = 4

function maxavg(arr , k){
  let sum = 0;

  for (let i = 0; i < k; i++) {
    sum += arr[i];
  }

  let max = sum;

  for (let i = k; i < arr.length; i++) {
    sum = sum - arr[i-k] + arr[i];
    max = Math.max(max,sum);
  }

  return max/3;
}

console.log(maxavg(arr , k));
```

#### Example-4 : Count Subarrays of Size K
Count how many subarrays of size k have sum > target

```ruby
let arr = [1,12,-5,-6,50,3]
let k = 3
let target = 20

function maxavg(arr , k , target){
  let sum = 0;
  let maxcount = 0;

  for (let i = 0; i < k; i++) {
    sum += arr[i];
  }

  for (let i = k; i < arr.length; i++) {
    sum = sum - arr[i-k] + arr[i];
    if(sum > target) {
      maxcount++;
    }
  }

  return maxcount;
}

console.log(maxavg(arr , k , target));
```

#### Example-5 : Minimum Length Subarray ≥ Target
👉 Find smallest subarray with sum ≥ target

let arr = [2,3,1,2,4,3 , 7]
let target = 7

function minlensubarr(arr , target){
  let left = 0;
  let sum = 0;
  let minlen = Infinity;

  for (let right = 0; right < arr.length; right++) {
    sum += arr[right];
    
    while (sum >= target ) {
      minlen = Math.min(minlen , right - left + 1 );
      sum-= arr[left];
      left++;
    }
  }

  return minlen === Infinity ? 0 : minlen;
}

console.log(minlensubarr(arr , target));




