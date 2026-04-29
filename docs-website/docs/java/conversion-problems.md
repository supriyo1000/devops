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