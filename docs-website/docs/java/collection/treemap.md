# TreeMap — Internal Working & Important Concepts

## 1. What is TreeMap?

`TreeMap` is a Java `Map` implementation that stores entries in **sorted order of keys**.

- Internal data structure: Red-Black Tree.
- Sorting: Natural ordering (`Comparable`) or custom (`Comparator`).
- `put()`, `get()`, `remove()`: O(log n).
- Supports navigation and range-based operations.

## 2. Internal Architecture

```text
                  TreeMap
                     |
                Red-Black Tree
                     |
                    20
                   /  \
                 10    30
                      /  \
                     25   40

          Each node stores:
          - key
          - value
          - parent
          - left child
          - right child
          - color (RED / BLACK)
```

A Red-Black Tree is a self-balancing Binary Search Tree.

**Why O(log n)?**

The tree maintains approximate balance, keeping its height proportional to `log n`.

Therefore:

| Operation | Complexity |
|---|---|
| put() | O(log n) |
| get() | O(log n) |
| remove() | O(log n) |
| firstKey() / lastKey() | O(log n) |
| containsKey() | O(log n) |

## 3. How does TreeMap sort keys?

TreeMap sorts **keys, not values**.

```java
TreeMap<Integer, String> map = new TreeMap<>();

map.put(30, "Apple");
map.put(10, "Mango");
map.put(20, "Banana");

System.out.println(map);
```

Output:

```text
{10=Mango, 20=Banana, 30=Apple}
```

## 4. Comparable vs Comparator

TreeMap needs a way to compare keys.

**Comparable — Natural ordering**

The key class defines its own ordering.

```java
class Employee implements Comparable<Employee> {

    private int id;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
}
```

**Comparator — Custom ordering**

Ordering is provided externally.

```java
TreeMap<Employee, String> map =
    new TreeMap<>(Comparator.comparing(Employee::getSalary));
```

Remember:

```text
Comparable
    → Natural ordering
    → compareTo()

Comparator
    → Custom ordering
    → compare()
```

## 5. Important: What happens when Comparator returns 0?

TreeMap treats two keys as equivalent when their comparison returns `0`.

```java
TreeMap<String, Integer> map =
    new TreeMap<>(Comparator.comparingInt(String::length));

map.put("Java", 1);
map.put("Code", 2);

System.out.println(map);
```

Output:

```text
{Java=2}
```

Both keys have the same length, so the second insertion replaces the existing value.

**Interview trick:** TreeMap uses `compareTo()` or `Comparator.compare()` to determine key ordering and equivalence. It does not use `equals()` to locate keys.

Make sure your comparison logic is consistent with `equals()` when you want normal Map semantics.

## 6. Duplicate Keys and Values

```java
map.put(10, "A");
map.put(10, "B");  // Replaces previous value

map.put(20, "B");  // Duplicate values are allowed
```

- Duplicate keys: Not allowed.
- Duplicate values: Allowed.
- Same key inserted again: Previous value is replaced.

## 7. Null Handling

With natural ordering:

```java
TreeMap<Integer, String> map = new TreeMap<>();

map.put(null, "A");  // NullPointerException
map.put(10, null);   // Allowed
```

- Null keys: Not supported with natural ordering.
- Null values: Allowed.
- A custom comparator can support null keys if it explicitly handles them.

## 8. Important Navigation Methods

TreeMap implements `NavigableMap`, providing methods to find nearby keys.

Example keys:

```text
10    20    30    40
```

| Method | Meaning | For key 25 |
|---|---|---|
| lowerKey() | Greatest key `<` given key | 20 |
| floorKey() | Greatest key `<=` given key | 20 |
| higherKey() | Smallest key `>` given key | 30 |
| ceilingKey() | Smallest key `>=` given key | 30 |

Remember:

```text
lower    → <
floor    → <=
higher   → >
ceiling  → >=
```

Example:

```java
TreeMap<Integer, String> map = new TreeMap<>();

map.put(10, "A");
map.put(20, "B");
map.put(30, "C");
map.put(40, "D");

System.out.println(map.lowerKey(20));   // 10
System.out.println(map.floorKey(20));   // 20
System.out.println(map.higherKey(20));  // 30
System.out.println(map.ceilingKey(20)); // 20
```

## 9. Other Important Methods

```java
map.firstKey();              // Smallest key
map.lastKey();               // Largest key

map.pollFirstEntry();        // Retrieves and removes first entry
map.pollLastEntry();         // Retrieves and removes last entry

map.headMap(30);             // Keys < 30
map.tailMap(20);             // Keys >= 20
map.subMap(10, 30);          // Keys >= 10 and < 30
```

Range methods return backed views, not independent copies. Changes to the view can affect the original map.

## 10. TreeMap vs HashMap vs LinkedHashMap

| Feature | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Internal structure | Hash table | Hash table + linked list | Red-Black Tree |
| Ordering | No guaranteed order | Insertion/access order | Sorted by key |
| Average get | O(1) | O(1) | O(log n) |
| put | Average O(1) | Average O(1) | O(log n) |
| Navigation | Limited | Limited | Rich navigation |
| Null key | One allowed | One allowed | Generally not with natural ordering |

**Selection rule:**

- `HashMap` → Fast lookup.
- `LinkedHashMap` → Maintain insertion/access order.
- `TreeMap` → Sorted keys and range/navigation operations.

## 11. Is TreeMap Thread-Safe?

No. TreeMap is not thread-safe by default.

For concurrent sorted-map requirements, use:

```java
ConcurrentSkipListMap<Integer, String> map =
    new ConcurrentSkipListMap<>();
```

It provides concurrent access and sorted-key navigation.

## 12. Important Question Points

1. TreeMap internally uses a Red-Black Tree.
2. Its keys are always maintained in sorted order.
3. Its basic operations take O(log n).
4. It implements `NavigableMap` and `SortedMap`.
5. It uses Comparable or Comparator for key ordering.
6. A comparison result of `0` means TreeMap treats keys as equivalent.
7. Duplicate keys replace existing values; duplicate values are allowed.
8. Natural ordering does not support null keys.
9. TreeMap is not thread-safe.
10. TreeMap maintains sorted order, not insertion order.

## Quick Revision

```text
TreeMap
   ↓
Red-Black Tree
   ↓
Sorted keys
   ↓
O(log n) operations
   ↓
Comparable / Comparator
   ↓
NavigableMap methods
   ↓
lowerKey(), floorKey()
higherKey(), ceilingKey()
   ↓
Range operations
   ↓
headMap(), tailMap(), subMap()
```

**Remember:** Use TreeMap when sorted keys, nearest-key lookup, or range-based operations matter more than constant-time average lookup.