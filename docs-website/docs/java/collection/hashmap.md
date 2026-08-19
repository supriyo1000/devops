
# Hashmap

### Hashmap internal architechture

```
                 HashMap
                    │
                    ▼
              Key + Value
                    │
                    ▼
              key.hashCode()
                    │
                    ▼
                 hash
                    │
                    ▼
             Bucket index
                    │
                    ▼
                Bucket
                    │
             ┌──────┴──────┐
             │             │
        No collision    Collision
             │             │
             ▼             ▼
           Node       Linked structure
                           │
                           ▼
                    Too many entries
                           │
                           ▼
                     Treeification
                           │
                           ▼
                      Red-Black Tree
```

### Hashmap when lookup

```
map.get(key)
     ↓
hashCode()
     ↓
find bucket
     ↓
compare hash
     ↓
equals()
     ↓
find exact key
     ↓
return value
```

### 1. What is HashMap?

HashMap is a collection in Java that stores data as key-value pairs. It uses hashing internally to determine where a key-value pair should be stored, which allows very fast insertion and lookup on average.

### 2. Why is HashMap fast?

HashMap is fast because it uses hashing to calculate a bucket location for a key, so it can usually access the required entry directly instead of searching through all elements.

put() → O(1)
get() → O(1)
remove() → O(1)

### 3. What is a bucket?

A bucket is a position in HashMap's internal array where entries are stored. Multiple entries can exist in the same bucket when hash collisions occur.

### 4. What is hashing?

Hashing is the process of converting a key into a hash value, which HashMap uses to determine the bucket where the entry should be stored.

### 5. What is hashCode()?

hashCode() is a method that returns an integer hash value for an object. HashMap uses the key's hash code to determine which bucket should contain the entry.

### 6. What is a collision?

A collision occurs when two different keys are mapped to the same bucket.

### 7. How does HashMap handle collision?

HashMap handles collisions by storing multiple entries in the same bucket. Initially they can be stored as a linked list, and when the bucket becomes sufficiently large, Java can convert the structure into a red-black tree to improve lookup performance.

### 8. Why are equals() and hashCode() important?

HashMap uses hashCode() to locate the bucket and equals() to identify the exact key inside that bucket. The equals-hashCode contract requires that equal objects must have the same hash code.

### 9. What is capacity?

Capacity is the number of buckets available in the HashMap's internal table.

### 10. What is load factor?

The default load factor is 0.75 because it provides a good balance between memory usage and collision performance. A higher load factor saves memory but increases collisions, while a lower load factor reduces collisions but uses more memory and causes more resizing.

### 11. What is threshold?

Threshold is the number of entries that HashMap allows before resizing. It is generally calculated as capacity multiplied by load factor.

### 12. How is threshold calculated?

The basic formula:
```
threshold = capacity × load factor
```
Example:
```
Capacity = 16
Load factor = 0.75


Threshold = 16 × 0.75
          = 12
```
Another example:
```
Capacity = 32
Load factor = 0.75


Threshold = 32 × 0.75
          = 24
```
Another:
```
Capacity = 64
Load factor = 0.75


Threshold = 64 × 0.75
          = 48
```

### 14. What happens during resize?

When HashMap reaches its threshold, it resizes the internal table, usually doubling the capacity. Existing entries are redistributed into the new table according to the new capacity.

### 15. Why does capacity grow 16 → 32 → 64?

HashMap's capacity normally grows by doubling.
```
16
 ↓ ×2
32
 ↓ ×2
64
 ↓ ×2
128
 ↓ ×2
256
```
Why?

Because doubling allows HashMap to grow efficiently while keeping the bucket-index calculation efficient.

It also works particularly well with the power-of-two design.

### 16. Why does HashMap use power-of-two capacity?

This is a deeper interview question.

HashMap uses capacities like:

16
32
64
128
256

rather than:

10
20
30
40

One major reason is efficient bucket-index calculation.

Conceptually, HashMap can calculate the index using:

(hash & (capacity - 1))

instead of an expensive modulo operation:

hash % capacity

For a power of two:

capacity = 16


capacity - 1 = 15

So:

hash & 15

can efficiently produce an index from:

0 → 15
Interview answer

"HashMap uses power-of-two capacities because it allows efficient bucket index calculation using bitwise operations. Instead of using modulo, it can use hash & (capacity - 1)."

Don't overcomplicate this

For most interviews, remember:

Power of 2
   ↓
Efficient index calculation
   ↓
Better distribution

### 18. What are Node and TreeNode?

These are implementation classes inside HashMap.

A normal HashMap entry is represented by a Node.

Conceptually:

Node
 ├── hash
 ├── key
 ├── value
 └── next

For example:

map.put("A", 100);

internally HashMap needs something conceptually like:

Node
key   = "A"
value = 100
hash  = ...
next  = ...

The next reference allows nodes to form a linked structure when collisions occur.

TreeNode

When a bucket is treeified, HashMap uses TreeNode.

Conceptually it contains references necessary for a tree structure:

TreeNode
 ├── key
 ├── value
 ├── hash
 ├── left
 ├── right
 └── parent

The actual implementation also maintains red-black tree information.

Interview answer

"Node represents a normal HashMap entry and is used in the bucket's linked structure. TreeNode is used when a heavily-collided bucket is converted into a red-black tree."

Remember
Normal bucket
     ↓
Node


Heavy collision
     ↓
TreeNode


### 19. What is the average complexity of HashMap?

The commonly quoted average complexities are:

Operation	Average
put()	O(1)
get()	O(1)
remove()	O(1)
containsKey()	O(1)

But there is an important detail.

Because collisions can occur, the operation isn't magically always O(1).

Historically, a heavily-collided bucket could behave like:

O(n)

With treeification in modern Java:

Linked structure → O(n)
Tree structure   → O(log n)


Interview answer

"HashMap provides O(1) average-time complexity for put, get and remove. In the presence of many collisions, performance can degrade, but Java 8 introduced treeification so heavily-collided buckets can achieve O(log n) lookup."

### 20. What is the mutable-key problem?

This is a very good interview question because it tests whether you actually understand HashMap.

Consider:

class Employee {
    int id;


    Employee(int id) {
        this.id = id;
    }


    @Override
    public int hashCode() {
        return id;
    }


    @Override
    public boolean equals(Object obj) {
        ...
    }
}

Now:

Employee e = new Employee(10);


HashMap<Employee, String> map = new HashMap<>();


map.put(e, "John");

At this point:

id = 10

HashMap calculates the hash and puts the entry into a bucket.

Now suppose you change:

e.id = 20;

The object's hash code changes:

Before:
hashCode() → based on 10


After:
hashCode() → based on 20

But the entry is still physically sitting in the bucket chosen using the old hash.

Now:

map.get(e);

HashMap calculates the new hash.

It looks in the bucket corresponding to the new hash.

But the entry is still in the old bucket.

So you may get:

null

even though the entry appears to be inside the map.

This is why mutable keys are dangerous.

Keys should generally be immutable.

Good examples:

String
Integer
Long
Interview answer

"A mutable-key problem occurs when fields used by equals() or hashCode() are changed after the object is inserted into a HashMap. The object's hash may then point to a different bucket than where the entry was originally stored, causing get() or remove() to fail. Therefore, HashMap keys should preferably be immutable."

Remember this flow
put(key)
   ↓
hash based on key
   ↓
bucket A


key changes
   ↓
hash changes
   ↓
get() searches bucket B
   ↓
entry is still in bucket A
   ↓
NOT FOUND

This is one of the best HashMap concepts to understand deeply.

### 21. How does HashMap handle null?

Java's HashMap allows:

One null key
Multiple null values

For example:

HashMap<String, Integer> map = new HashMap<>();


map.put(null, 100);
map.put("A", null);
map.put("B", null);

This is valid.

You can have:

null → 100
A    → null
B    → null
Why only one null key?

A map cannot have duplicate keys.

If you do:

map.put(null, 100);
map.put(null, 200);

the second value replaces the first:

null → 200
Important implementation detail

HashMap handles the null key specially because you cannot call:

null.hashCode()

So HashMap has special handling for a null key.

Interview answer

"HashMap allows one null key and multiple null values. Since a null key cannot have a hashCode, HashMap handles the null key specially. If we insert another null key, it replaces the previous value."
