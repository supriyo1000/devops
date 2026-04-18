# Two Pointer

## 1. What is pointer?

Array
```
index : [0,1,2,3,4];
Array : [1,2,3,4,5];
```

A pointer = just a variable holding an index

```
let i = 0; --> points to 1
let j = 4; --> points to 5
```

## 2. What is two pointer?

Now two pointer is 

```
let left = 0 ; points to 1;
let right = array.length() - 1; points to 5;
```

### Move pointer

It means :
+ left++
+ right--


## Flow

Every two pointer problems follow this

+ Take two pointers
+ compare something
+ Based on condition move pointer
+ Repeat until they meet

### Situation

+ too big? right--
+ too small? left++

## How to detect two pointer?

+ Comparaing start and end
+ Finding pairs(2 numbers)
+ working on sorted array

#### Example-1

__Palindrome__

```ruby
function isPalindrome(str){
  let left = 0;
  let right = str.length -1;

  while (left !== right) {
    if(str[left] !== str[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

console.log(isPalindrome("racecar")); // true
console.log(isPalindrome("madam"));  // true
console.log(isPalindrome("hello")); // false
```

#### Example-2

__Sum__

```ruby
function sum(arr , target){
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {
    if((arr[left] + arr[right]) < target ){
      left++;
    }
    else if((arr[left] + arr[right]) > target){
      right--;
    } else {
      return [arr[left] , arr[right]]
    }
  }

  return null;
}

console.log(sum([1,2,3,4,6] , 3));
```

#### Example 3: Move Zeroes (Same Direction)

```ruby
function movezero(arr) {
  let j = 0;

  for (let i = 0; i < arr.length; i++) {
    if(arr[i] !== 0) {
      [arr[i] , arr[j]] = [arr[j] , arr[i]];
      j++;
    }
  }

  return arr;
}

console.log(movezero([5,0, 1,0,0, 0, 3, 12]));
```

#### Example 4: Reverse String

```ruby
function reverseString (str) {
  str = str.split("");
  
  let left = 0;
  let right = str.length - 1;

  while (left < right) {
    [str[left] , str[right]] = [str[right] , str[left]];
    left++;
    right--;
  }

  return str.join("");
  
}

console.log(reverseString("sensex"));
```
