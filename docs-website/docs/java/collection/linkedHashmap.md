# LinkedHashMap


## Q1. What is LinkedHashMap?

Answer:

LinkedHashMap is a subclass of HashMap that maintains a predictable iteration order.

By default, it maintains insertion order.

Internally, it uses the hash table mechanism of HashMap for fast lookup and additionally maintains a doubly linked list to maintain the order of entries.

It can also maintain access order, which makes it useful for implementing an LRU cache.

Short interview answer:

LinkedHashMap is a HashMap implementation that maintains ordering of entries using a doubly linked list. By default, it maintains insertion order, but it can also maintain access order.

## Q2. What is the relationship between HashMap and LinkedHashMap?

Answer:

LinkedHashMap extends HashMap.

Conceptually:

HashMap
   ↑
   |
LinkedHashMap

Therefore, LinkedHashMap gets the basic hash-table functionality from HashMap.

LinkedHashMap adds additional functionality for maintaining ordering.

So:

HashMap
   ↓
Hash table
   ↓
Fast lookup


LinkedHashMap
   ↓
Hash table
+
Doubly linked list
   ↓
Ordering

Interview answer:

LinkedHashMap extends HashMap. It uses the hash table inherited from HashMap for lookup and adds a doubly linked list between entries to maintain iteration order.

## Q3. Why does LinkedHashMap maintain insertion order?

Answer:

Because it maintains an additional doubly linked list connecting all entries in the order in which they were inserted.

For example:

map.put("A", 10);
map.put("B", 20);
map.put("C", 30);

The linked list conceptually becomes:

A ↔ B ↔ C

When we iterate over the map, the entries are visited using this linked-list order.

Therefore, the output is:

A
B
C

The hash table is responsible for lookup, while the linked list is responsible for ordering.

## Q4. What data structures are used internally by LinkedHashMap?

Answer:

Two main structures are involved:

1. Hash table
2. Doubly linked list

The hash table provides fast lookup.

The doubly linked list maintains the iteration order.

Conceptually:

Hash table
    |
    ↓
Entry A
Entry B
Entry C


Linked list
    ↓
A ↔ B ↔ C

Each entry participates in both structures.

## Q5. How is a LinkedHashMap entry different from a HashMap entry?

Answer:

A normal HashMap node conceptually contains:

hash
key
value
next

A LinkedHashMap entry additionally maintains links for the doubly linked list:

hash
key
value
next
before
after

Where:

next

helps with the hash-table bucket structure, while:

before
after

maintain the linked-list ordering.

## Q6. What happens internally when put() is called?

Suppose:

map.put("A", 100);

Conceptually:

put()
 ↓
calculate hash
 ↓
find appropriate bucket
 ↓
check whether key already exists
 ↓
if not found
 ↓
create entry
 ↓
insert into hash table
 ↓
add entry to linked list

The new entry is connected to the end of the linked list.

For example:

Before:


A ↔ B


put(C)


After:


A ↔ B ↔ C


## Q7. What happens if we insert an existing key?

Suppose:

map.put("A", 10);
map.put("B", 20);
map.put("C", 30);


map.put("B", 200);

B already exists.

Therefore, Java updates the value:

B: 20 → 200

It does not create another B entry.

With normal insertion-order mode, the order remains:

A → B → C

So:

Updating an existing key does not move it in insertion-order mode.

## Q8. What happens if we remove and insert the same key again?

Suppose:

map.put("A", 10);
map.put("B", 20);
map.put("C", 30);


map.remove("B");
map.put("B", 200);

Initially:

A → B → C

After removing B:

A → C

After inserting B again:

A → C → B

Because the second put() is a new insertion.

## Q9. Why does LinkedHashMap use a doubly linked list?

Answer:

A doubly linked list allows each entry to know both:

previous entry
next entry

For example:

A ↔ B ↔ C

B knows:

before = A
after = C

This makes removing or moving an entry in the ordering list efficient.

This becomes especially important when LinkedHashMap is configured for access order, because accessed entries may need to be moved to the end of the list.

## Q10. What is access order?

Answer:

Access order means that entries are ordered according to when they were most recently accessed.

We enable it using:
```
LinkedHashMap<String, Integer> map =
        new LinkedHashMap<>(16, 0.75f, true);
```
The third parameter:

true

means:

accessOrder = true

Now accessing an existing entry moves it toward the end of the linked list.

## Q11. What is the difference between insertion order and access order?
Insertion order
new LinkedHashMap<>();

Order is based on insertion.

put(A)
put(B)
put(C)


A → B → C

Calling:

get(A)

does not change the order.

Access order
new LinkedHashMap<>(16, 0.75f, true);

Order is based on recent access.

Initially:

A → B → C

Then:

get(A);

becomes:

B → C → A

So the tail represents the most recently accessed entry.

## Q12. Explain this code.
```
LinkedHashMap<String, Integer> map =
        new LinkedHashMap<>(16, 0.75f, true);


map.put("A", 1);
map.put("B", 2);
map.put("C", 3);


map.get("A");
```
Answer:

Initially:

A → B → C

Because access order is enabled, get("A") moves A to the end.

Final order:

B → C → A

Therefore:

B = least recently accessed
A = most recently accessed


## Q13. Explain this sequence.

```
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);


map.get("B");
map.get("A");

with:

accessOrder = true
```

Answer:

Initially:

A → B → C

After:

get("B");

we get:

A → C → B

Then:

get("A");

moves A to the end:

C → B → A

Final order:

C → B → A

Therefore:

C = least recently used
A = most recently used
Q14. What is an LRU cache?

Answer:

LRU stands for:

Least Recently Used

An LRU cache removes the item that has not been used for the longest time when the cache reaches its capacity.

Example:

Cache capacity = 3


A → B → C

If we access A:

B → C → A

Now B is the least recently used.

If we insert D, B should be removed:

C → A → D


## Q15. Why is LinkedHashMap suitable for implementing an LRU cache?

Answer:

Because LinkedHashMap can maintain access order.

When configured with:

accessOrder = true

the linked list represents:

least recently accessed
        ↓
most recently accessed

Therefore, the first entry is the least recently used entry.

When the cache exceeds its capacity, we can remove that entry.

This makes LinkedHashMap very convenient for implementing an LRU cache.

## Q16. What is removeEldestEntry()?

Answer:

removeEldestEntry() is a method in LinkedHashMap that allows us to automatically remove the oldest/eldest entry after an insertion.

Example:
```
@Override
protected boolean removeEldestEntry(
        Map.Entry<K, V> eldest) {


    return size() > capacity;
}
```
If the map exceeds its capacity:

return true

causes the eldest entry to be removed.

In access-order mode, the eldest entry represents the least recently used entry.

## Q17. Explain an LRU cache implementation using LinkedHashMap.

```
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(16, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(
            Map.Entry<K, V> eldest) {


        return size() > capacity;
    }
}
```
Answer:

extends LinkedHashMap
class LRUCache<K, V> extends LinkedHashMap<K, V>

The cache uses LinkedHashMap functionality.

Capacity
private final int capacity;

Stores the maximum number of entries.

Constructor
super(16, 0.75f, true);

means:

initial capacity = 16
load factor = 0.75
access order = true

The important part for LRU is:

accessOrder = true
removeEldestEntry()
return size() > capacity;

Once the cache exceeds its capacity, the eldest entry is removed.

Because access order is enabled, the eldest entry is the least recently used entry.

## Q18. What happens internally during get() in access-order mode?

Suppose:

A → B → C

and:

map.get("B");

First, the hash table is used to locate B.

Then, because access order is enabled, B is moved to the end.

Result:

A → C → B

So:

B = most recently accessed

The hash table provides the lookup, while the linked list is updated to maintain access order.

## Q19. What happens internally during remove()?

Suppose:

A ↔ B ↔ C

and:

map.remove("B");

B must be removed from both structures.

Hash table

Remove B from its bucket.

Linked list

Reconnect:

A ↔ C

So B is no longer part of either structure.

## Q20. Does LinkedHashMap provide O(1) lookup?

Answer:

Yes, average-case lookup is:

O(1)

because LinkedHashMap is based on the hash-table mechanism of HashMap.

For example:

map.get("A");

uses the hash of "A" to locate the appropriate bucket.

The linked-list maintenance in access-order mode also involves constant-time pointer updates.

Therefore, typical operations are:
```
get()     → O(1) average
put()     → O(1) average
remove()  → O(1) average
containsKey() → O(1) average
```

## Q21. What is the time complexity of iterating over LinkedHashMap?

Answer:

Iteration is:

O(n)

where n is the number of entries.

An important advantage is that iteration follows the linked-list order.

So LinkedHashMap doesn't need to depend on the arbitrary arrangement of hash buckets to determine the iteration order.

## Q22. Why does LinkedHashMap use more memory than HashMap?

Answer:

Because each entry needs additional references for maintaining the doubly linked list.

Conceptually:

HashMap entry:
```
hash
key
value
next
```
LinkedHashMap additionally needs:
```
before
after
```
Therefore:

LinkedHashMap
    ↓
HashMap memory
+
linked-list references

So it has additional memory overhead.

## Q23. Compare HashMap and LinkedHashMap.

I would use HashMap when I only need fast key-value lookup and don't care about iteration order. I would use LinkedHashMap when I need predictable iteration order or access-order behavior, such as an LRU cache.

## Q25. Why can't we simply use HashMap for an LRU cache?

Answer:

Because an LRU cache needs to know:

which entry was used least recently

HashMap doesn't maintain access order.

It gives us fast lookup:

key → value

but doesn't naturally maintain:

least recently used
        ↓
most recently used

LinkedHashMap can provide this using:

accessOrder = true

Therefore, it is much more suitable for an LRU cache.

## Q26. Does updating an existing key change its position?

This depends on the mode.

Insertion-order mode

```
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.put("B", 200);
```
Order remains:

A → B → C

The value changes:

B: 2 → 200

but its position doesn't change.

Access-order mode

An operation that accesses an existing mapping can cause it to move to the end.

So in access-order mode, an existing mapping can change position when it is accessed.

## Q27. Is LinkedHashMap thread-safe?

Answer:

No.

LinkedHashMap is not thread-safe by default.

If multiple threads modify the same LinkedHashMap concurrently, synchronization may be required.

For example:

Thread 1
     ↓
Thread 2 → LinkedHashMap
     ↑
Thread 3

can cause concurrency problems if access is not properly synchronized.

For concurrent applications, you need to choose an appropriate thread-safe/concurrent design based on the requirements.

## Q28. What is the biggest advantage of LinkedHashMap?

Answer:

Its main advantage is that it combines:

HashMap's fast lookup
+
predictable ordering

So we get approximately:

O(1) average lookup

while also maintaining insertion or access order.

Q29. What is the biggest disadvantage?

Answer:

The main disadvantages are:

Additional memory usage because of linked-list references.
Slightly more overhead than HashMap.
It is not thread-safe by default.
If ordering isn't required, a normal HashMap may be simpler and more memory-efficient.


## Q30. Why can't HashMap simply iterate through buckets in insertion order?

Answer:

Because bucket placement is determined by the key's hash, not by insertion time.

Suppose we insert:

A
B
C

The insertion order is:

A → B → C

But their hash values might place them into:

bucket 1 → B
bucket 4 → A
bucket 7 → C

The bucket arrangement doesn't contain enough information to tell us reliably:

A was inserted first
B was inserted second
C was inserted third

Therefore, LinkedHashMap maintains an additional linked list:

A ↔ B ↔ C

to explicitly preserve the required order.

This is one of the most important questions to understand.