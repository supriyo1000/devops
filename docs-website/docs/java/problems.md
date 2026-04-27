## ✅ 1. Two Sum

#### Problem:
Given an array and a target, return indices of two numbers that add up to the target.

Input:
nums = [2,7,11,15], target = 9

Output:
[0,1]

## ✅ 2. Move Zeroes

#### Problem:
Move all 0s to the end without changing order of others.

Input:
[0,1,0,3,12]

Output:
[1,3,12,0,0]

## ✅ 3. Rotate Array

#### Problem:
Rotate array to the right by k steps.

Input:
nums = [1,2,3,4,5,6,7], k = 3

Output:
[5,6,7,1,2,3,4]

## ✅ 4. Maximum Subarray (Kadane's Algorithm)

#### Problem:
Find contiguous subarray with maximum sum.

Input:
[-2,1,-3,4,-1,2,1,-5,4]

Output:
6
(Explanation: [4,-1,2,1])

## ✅ 5. Longest Substring Without Repeating Characters

Input:
"abcabcbb"

Output:
3
("abc")

## ✅ 6. Valid Palindrome

#### Problem:
Check if string is palindrome (ignore spaces & symbols).

Input:
"A man, a plan, a canal: Panama"

Output:
true

## ✅ 7. Group Anagrams

Input:
["eat","tea","tan","ate","nat","bat"]

Output:
[["eat","tea","ate"],["tan","nat"],["bat"]]

## ✅ 8. First Non-Repeating Character

Input:
"aabbcdd"

Output:
"c"

## ✅ 9. Frequency Counter

#### Problem:
Count frequency of each element.

Input:
[1,2,2,3,1,1]

Output:
{1:3, 2:2, 3:1}

## ✅ 10. Valid Parentheses

Input:
"()[]{}"

Output:
true

Input:
"(]"

Output:
false

## ✅ 11. Next Greater Element

#### Problem:
For each element, find next greater element on right.

Input:
[2,1,2,4,3]

Output:
[4,2,4,-1,-1]

## ✅ 12. Binary Search

#### Problem:
Find index of target in sorted array.

Input:
nums = [-1,0,3,5,9,12], target = 9

Output:
4

## ✅ 13. Merge Intervals

Input:
[[1,3],[2,6],[8,10],[15,18]]

Output:
[[1,6],[8,10],[15,18]]

## ✅ 14. Deep Copy vs Shallow Copy

#### Problem:
Create deep copy of object/array.

Input (JS example):

obj = {a:1, b:{c:2}}
copy = deepCopy(obj)
copy.b.c = 10

Output:
Original remains:

{a:1, b:{c:2}}
## ✅ 15. Implement map / filter / reduce

### 🟢 MAP Problems (Transformation)

```
1. Double numbers
Input:  [1,2,3,4]
Output: [2,4,6,8]
```
```
2. Square numbers
Input:  [2,3,4]
Output: [4,9,16]
```
```
3. Convert to string
Input:  [1,2,3]
Output: ["1","2","3"]
```
```
4. Add 10 to each
Input:  [5,10,15]
Output: [15,20,25]
```
```
5. Extract property
Input:  [{name:"a"}, {name:"b"}]
Output: ["a","b"]
```

### 🟡 FILTER Problems (Selection)

👉 Think: “keep only valid ones”

```
6. Even numbers
Input:  [1,2,3,4,5]
Output: [2,4]
```
```
7. Numbers > 10
Input:  [5,12,8,20]
Output: [12,20]
```
```
8. Remove null/undefined
Input:  [1,null,2,undefined,3]
Output: [1,2,3]
```
```
9. Words longer than 3 chars
Input:  ["hi","hello","hey"]
Output: ["hello"]
```
```
10. Filter objects
Input:  [{age:10},{age:20}]
Output: [{age:20}]
```

### 🔴 REDUCE Problems (Aggregation)

👉 Think: “combine into one result”

```
11. Sum of array
Input:  [1,2,3,4]
Output: 10
```
```
12. Multiply all numbers
Input:  [1,2,3,4]
Output: 24
```
```
13. Count elements
Input:  ["a","b","a"]
Output: {a:2, b:1}
```
```
14. Find max
Input:  [5,10,2]
Output: 10
```
```
15. Flatten array
Input:  [[1,2],[3,4]]
Output: [1,2,3,4]
```