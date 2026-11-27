# LinkedHashMap

## 1. Internal Architecture

```text
                    LinkedHashMap
                          |
              ┌───────────┴───────────┐
              |                       |
              ▼                       ▼
          Hash Table             Doubly Linked List
              |                       |
       Fast Lookup              Maintains Order
              |                       |
              ▼                       ▼
        Key → Value          A ↔ B ↔ C ↔ D
              |                       |
              └───────────┬───────────┘
                          |
                          ▼
                 Predictable Iteration
```

**Internal Entry Structure:**

```text
LinkedHashMap.Entry
 ├── hash
 ├── key
 ├── value
 ├── next       → Hash bucket connection
 ├── before     → Previous entry
 └── after      → Next entry
```

## 2. What is LinkedHashMap?

LinkedHashMap is a HashMap implementation that maintains predictable iteration order using an additional doubly linked list.

It supports two ordering modes:

- Insertion Order (default)
- Access Order (based on recent access)

**Relationship:** LinkedHashMap extends HashMap.

```text
HashMap
  |
  ├── Hash Table
  ├── Hashing
  └── Fast Lookup

LinkedHashMap extends HashMap
  |
  ├── Inherits HashMap functionality
  └── Adds Doubly Linked List
```

## 3. How LinkedHashMap Works Internally

When `put(key, value)` is called:

1. Calculates the key's hash using `hashCode()`.
2. Calculates the bucket index.
3. Searches the bucket for an existing key using hash and `equals()`.
4. If the key exists, updates its value.
5. Otherwise, creates a new entry and inserts it into the hash table.
6. Connects the new entry to the doubly linked list to maintain iteration order.

Example:

```java
map.put("A", 10);
map.put("B", 20);
map.put("C", 30);
```

Internal linked list:

```text
A ↔ B ↔ C
```

Iteration returns:

```text
A → B → C
```

**Important:** Hash table handles lookup; doubly linked list maintains iteration order.

## 4. Insertion Order vs Access Order

### Insertion Order (Default)

```java
LinkedHashMap<String, Integer> map = new LinkedHashMap<>();
```

Maintains the order in which entries were inserted.

```java
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.get("A");
```

Final order:

```text
A → B → C
```

Accessing A does not change its position.

### Access Order

```java
LinkedHashMap<String, Integer> map =
        new LinkedHashMap<>(16, 0.75f, true);
```

The third constructor parameter `true` enables access order.

```java
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.get("A");
```

Before access:

```text
A → B → C
```

After accessing A:

```text
B → C → A
```

The most recently accessed entry moves to the end.

**Interview Trick:** In access-order mode, the first entry is the least recently used (LRU), and the last entry is the most recently used (MRU).

## 5. How Does LinkedHashMap Maintain Order?

Each entry contains two additional references:

- `before` → Previous entry.
- `after` → Next entry.

Example:

```text
A ↔ B ↔ C
```

When B is accessed in access-order mode:

```text
Before: A ↔ B ↔ C

After:  A ↔ C ↔ B
```

Because LinkedHashMap maintains references to previous and next entries, it can move an entry within the linked list using constant-time pointer updates.

## 6. LRU Cache Using LinkedHashMap

**LRU = Least Recently Used**

An LRU cache removes the entry that has not been accessed for the longest time when the cache exceeds its maximum capacity.

LinkedHashMap supports this using:

- `accessOrder = true`
- `removeEldestEntry()`

### Implementation

```java
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

### How It Works

Suppose:

```text
Cache Capacity = 3

A → B → C
```

Access A:

```java
cache.get("A");
```

New order:

```text
B → C → A
```

Insert D:

```java
cache.put("D", 4);
```

Cache exceeds capacity, so B is removed.

Final order:

```text
C → A → D
```

**Interview Trick:** `removeEldestEntry()` returns true when the eldest entry should be removed. In access-order mode, the eldest entry is the least recently used entry.

## 7. Time Complexity

| Operation | Average Complexity |
|---|---|
| put() | O(1) |
| get() | O(1) |
| remove() | O(1) |
| containsKey() | O(1) |
| Iteration | O(n) |

LinkedHashMap provides average O(1) lookup because it uses HashMap's hashing mechanism.

Its doubly linked list maintains order with constant-time pointer updates.

## 8. HashMap vs LinkedHashMap

| Feature | HashMap | LinkedHashMap |
|---|---|---|
| Internal Structure | Hash Table | Hash Table + Doubly Linked List |
| Ordering | No guaranteed order | Insertion / Access Order |
| Lookup | Average O(1) | Average O(1) |
| Iteration | O(n + capacity) | O(n) |
| Memory | Less | More |
| LRU Cache | Not directly suitable | Suitable |
| Thread Safety | Not thread-safe | Not thread-safe |

## 9. Important Question Tricks

1. LinkedHashMap extends HashMap.
2. Default ordering is insertion order.
3. Access order is enabled using the third constructor parameter `true`.
4. Updating an existing key does not move its position in insertion-order mode.
5. In access-order mode, accessing an existing entry moves it to the end.
6. Removing and reinserting a key places it at the end.
7. LinkedHashMap uses additional `before` and `after` references.
8. `removeEldestEntry()` is useful for implementing an LRU cache.
9. LinkedHashMap is not thread-safe by default.
10. LinkedHashMap uses more memory than HashMap because of its additional linked-list references.

## 10. Quick Revision

```text
LinkedHashMap
     |
     ├── Extends HashMap
     |
     ├── Hash Table → Fast Lookup
     |
     ├── Doubly Linked List → Maintains Order
     |
     ├── Default → Insertion Order
     |
     ├── accessOrder = true → Access Order
     |
     ├── removeEldestEntry() → LRU Cache
     |
     └── Average Lookup → O(1)
```

**Remember:**

HashMap = Fast Key-Value Lookup

LinkedHashMap = Fast Key-Value Lookup + Predictable Iteration Order

**Most Important Interview Concept:** Understand how the hash table and doubly linked list work together, especially how access-order mode enables LRU cache implementation.