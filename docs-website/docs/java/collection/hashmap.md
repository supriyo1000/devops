# HashMap

## 1. Internal Architecture

```
                    HashMap
                       |
                       ▼
                  Key + Value
                       |
                       ▼
                key.hashCode()
                       |
                       ▼
                  Hash Spreading
                       |
                       ▼
              Calculate Bucket Index
             (hash & (capacity - 1))
                       |
                       ▼
                Internal Array
                       |
                ┌──────┴──────┐
                ▼             ▼
           Empty Bucket    Collision
                |             |
                ▼             ▼
              Node       Linked List
                              |
                              ▼
                      Too Many Entries
                              |
                              ▼
                        Treeification
                              |
                              ▼
                       Red-Black Tree
```


## 2. How HashMap Works Internally

HashMap stores data in an internal array called a table, where each position represents a bucket.

When put(key, value) is called:
1. HashMap calls key.hashCode() to generate a hash.
2. It applies hash spreading to improve hash distribution.
3. It calculates the bucket index using (capacity - 1) & hash.
4. If the bucket is empty, it stores a new Node.
5. If the bucket already contains entries, it compares hashes and uses equals() to identify an existing key.
6. If the key already exists, its value is replaced; otherwise, a new Node is added.
7. If a bucket becomes heavily populated, it can be converted into a Red-Black Tree.


## 3. How HashMap Performs Lookup
```
map.get(key)
     |
     ▼
Calculate hash
     |
     ▼
Calculate bucket index
     |
     ▼
Access bucket directly
     |
     ▼
Compare hash
     |
     ▼
Compare using equals()
     |
     ▼
Find matching Node
     |
     ▼
Return value
```

## 4. Why Is HashMap Fast?
HashMap is fast because it uses hashing to calculate the bucket location instead of searching every entry.

#### For example:
Array:


|Index|     Data|
|-----|---------|
|  0   |    Node A|
 | 1    |   Node B|
 | 2    |   Node C|
 | 3    |   Node D|
 | 4    |   Node E|


To find Node D, HashMap calculates its bucket index and directly accesses index 3 instead of checking every entry.

- put() → O(1) average
- get() → O(1) average
- remove() → O(1) average

__Important__: Collisions can increase lookup time. Linked-list traversal can take O(n), while treeified buckets generally provide O(log n) lookup.


## 5. Important Internal Concepts

|Key (Topic)|	Value (Description / Interview Trick)|
|-----------|---------------------------------------|
|Hashing|	Converts a key into a hash value to determine its bucket.|
|Bucket|	A position in the internal array that stores entries.|
|hashCode()|	Generates an integer hash used for bucket calculation.|
|equals()|	Identifies the exact matching key inside a bucket.|
|Collision|	Different keys map to the same bucket.|
|Node|	Stores hash, key, value and next reference.|
|TreeNode|	Represents an entry in a Red-Black Tree bucket.|
|Capacity|	Number of buckets. Default initial capacity is 16.|
|Load Factor|	Default is 0.75; controls when resizing happens.|
|Threshold|	Capacity × Load Factor. Default: 16 × 0.75 = 12.|
|Resize|	Normally doubles capacity when threshold is exceeded.|
|Treeification|	Converts a heavily populated bucket into a Red-Black Tree.|
|Null|	Allows one null key and multiple null values.|
|Mutable Key|	Changing fields used by hashCode()/equals() after insertion can break lookup.|
|Thread Safety|	HashMap is not thread-safe.|

## 6. Important Question Points
- Equal objects must have the same hashCode.
- Different objects can have the same hashCode.
- HashMap uses a power-of-two capacity for efficient bucket calculation using bitwise AND.
- HashMap uses a linked structure initially and can use a Red-Black Tree for heavily populated buckets.
- Prefer immutable objects as HashMap keys.


__Remember__: HashMap = Internal Array + Hashing + Linked List + Red-Black Tree.