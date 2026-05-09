# 🚀 20 Interview-Level Java Conversion Problems

### 🟢 LEVEL 1 (Fundamentals – must be instant)

```ruby
1.

int[] → List<Integer>

[1,2,3] → [1,2,3]
```
```ruby
2.

List<Integer> → int[]

[1,2,3] → [1,2,3]
```
```ruby
3.

String[] → List<String>

["a","b","c"] → List
```
```ruby
4.

List<String> → String[]
```
```ruby
5.

int[] → String[]

[1,2,3] → ["1","2","3"]
```

### 🟡 LEVEL 2 (Transformation – real interview start)

```ruby
6.

String[] → int[]

["1","2","3"] → [1,2,3]
```
```ruby
7.

List<String> → List<Integer>
```
```ruby
8.

int[] → List<String>

[1,2,3] → ["1","2","3"]
```
```ruby
9.

List<Integer> → List<String>
```
```ruby
10.

String → List<Character>

"hello" → ['h','e','l','l','o']
```


### 🔴 LEVEL 3 (Filtering + Mapping combined)

```ruby
11.

int[] → List of even numbers

[1,2,3,4] → [2,4]
```
```ruby
12.

List<String> → uppercase List

["a","b"] → ["A","B"]
```
```ruby
13.

String[] → List<String> (length > 3 only)

["hi","hello","hey"] → ["hello"]
```
```ruby
14.

int[] → square → List

[1,2,3] → [1,4,9]
```

### 🔥 LEVEL 4 (Grouping / Map Conversion – very important)

```ruby
15.

String[] → frequency map

["a","b","a"] → {a:2, b:1}
```
```ruby
16.

List<Integer> → Map (number → square)

[1,2,3] → {1:1, 2:4, 3:9}
```
```ruby
17.

List<String> → Map (length → list of words)

["hi","hey","hello"] → {2:[hi], 3:[hey], 5:[hello]}
```

### ⚡ LEVEL 5 (Advanced / Interview Killer)

```ruby
18.

List<String> → comma-separated String

["a","b","c"] → "a,b,c"
```
```ruby
19.

String → frequency map (characters)

"aabcc" → {a:2, b:1, c:2}
```
```ruby
20.

List<List<Integer>> → flatten → List<Integer>

[[1,2],[3,4]] → [1,2,3,4]
```

## 🚀 Next Interview-Level Java Conversion Problems

### 🟢 LEVEL 1 (Core Conversions – must be easy)

```ruby
21.

char[] → List<Character>

['a','b','c'] → ['a','b','c']
```
```ruby
22.
List<Character> → char[]

['a','b'] → ['a','b']
```
```ruby
23.

String → char[]

"hello" → ['h','e','l','l','o']
```
```ruby
24.

char[] → String

['h','i'] → "hi"
```
```ruby
25.

int[] → Set<Integer>

[1,2,2,3] → [1,2,3]
```

### 🟡 LEVEL 2 (Transformation + Collect)

```ruby
26.

List<Integer> → Set<String>

[1,2,2] → ["1","2"]
```
```ruby
27.

List<String> → Set<Integer>

["1","2","2"] → [1,2]
```
```ruby
28.

int[] → List<Integer> (only > 2)

[1,2,3,4] → [3,4]
```
```ruby
29.

List<String> → List<Integer> (string length)

["hi","hello"] → [2,5]
```
```ruby
30.

String[] → List<String> (reverse each word)

["ab","cd"] → ["ba","dc"]
```

### 🔴 LEVEL 3 (Map + Collect – real interview zone)

```ruby
31.

List<String> → Map<String, Integer> (word → length)

["hi","hello"] → {hi:2, hello:5}
```
```ruby
32.

List<Integer> → Map<Boolean, List<Integer>> (even/odd grouping)

[1,2,3,4] → {true:[2,4], false:[1,3]}
```
```ruby
33.

String → Map<Character, List<Integer>> (char → positions)

"aba" → {a:[0,2], b:[1]}
```
```ruby
34.

List<String> → Map<Character, List<String>> (group by first letter)

["apple","bat","ball"] → {a:[apple], b:[bat,ball]}
```
```ruby
35.

int[] → Map<Integer, Long> (frequency count)

[1,2,2,3] → {1:1,2:2,3:1}
```


### 🔥 LEVEL 4 (Advanced collect + transformation)

```ruby
36.

List<String> → single String (reverse order join)

["a","b","c"] → "c,b,a"
```
```ruby
37.

List<Integer> → sum of squares

[1,2,3] → 14
```
```ruby
38.

String[] → longest string

["hi","hello","hey"] → "hello"
```
```ruby
39.

List<String> → count of words starting with vowel

["apple","bat","orange"] → 2
```
```ruby
40.

int[] → average of even numbers

[1,2,4,5] → 3.0
```


### ⚡ LEVEL 5 (Stack + String + Stream combo)

```ruby
41.

String → remove duplicates (keep order)

"abac" → "abc"
```
```ruby
42.

String → check palindrome using Stack

"madam" → true
```
```ruby
43.

String[] → flatten characters → List<Character>

["ab","cd"] → ['a','b','c','d']
```
```ruby
44.

List<String> → List<Character> (all chars uppercase)

["ab","cd"] → ['A','B','C','D']
```
```ruby
45.

String → compress (basic run-length)

"aabbbc" → "a2b3c1"
```