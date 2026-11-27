# Iterator

## 1. What is an Iterator?

Iterator is an interface in Java used to traverse elements of a Collection one by one without directly accessing elements by index.

**Main methods:**
- `hasNext()` → Checks whether another element exists.
- `next()` → Returns the next element and advances the iterator.
- `remove()` → Removes the last element returned by `next()`.

### Example

```java
List<String> names = new ArrayList<>(
        List.of("A", "B", "C")
);

Iterator<String> it = names.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

Output:
```text
A
B
C
```

## 2. Internal Working of Iterator

```text
             Collection
                  |
                  ▼
             iterator()
                  |
                  ▼
              Iterator
                  |
                  ▼
              hasNext()
                  |
           ┌──────┴──────┐
           |             |
          true          false
           |             |
           ▼             ▼
         next()         Stop
           |
           ▼
     Return Element
           |
           ▼
    Move to Next Element
           |
           ▼
       Repeat Loop
```

**Explanation:**

1. `iterator()` creates an iterator for the collection.
2. The iterator maintains traversal state, such as its current position.
3. `hasNext()` checks whether another element is available.
4. `next()` returns the next element and advances the iterator.
5. Traversal continues until `hasNext()` returns false.

**Remember:** `hasNext()` checks; `next()` returns and advances.

## 3. What happens if next() has no element?

Calling `next()` when no elements remain throws `NoSuchElementException`.

```java
Iterator<String> it = List.of("A").iterator();

System.out.println(it.next()); // A
System.out.println(it.next()); // NoSuchElementException
```

Always check `hasNext()` before calling `next()`.

## 4. What is Iterator.remove()?

`Iterator.remove()` removes the element most recently returned by `next()`.

### Example: Safe Removal

```java
List<String> names = new ArrayList<>(
        List.of("A", "B", "C")
);

Iterator<String> it = names.iterator();

while (it.hasNext()) {

    String name = it.next();

    if (name.equals("B")) {
        it.remove();
    }
}

System.out.println(names);
```

Output:
```text
[A, C]
```

**Important:** Use `it.remove()` instead of directly calling `list.remove()` while traversing through the iterator.

## 5. What is Fail-Fast Behavior?

Fail-fast behavior means an iterator attempts to detect unexpected structural modifications to a collection during iteration and may throw `ConcurrentModificationException` (CME).

### Example

```java
List<String> names = new ArrayList<>(
        List.of("A", "B", "C")
);

Iterator<String> it = names.iterator();

while (it.hasNext()) {

    String name = it.next();

    if (name.equals("B")) {
        names.remove(name); // Unsafe
    }
}
```

This can throw:

```text
ConcurrentModificationException
```

### Internal Working

```text
Iterator Created
      |
      ▼
Collection modCount = 3
      |
      ▼
Iterator Stores expectedModCount = 3
      |
      ▼
Collection Modified Directly
      |
      ▼
modCount = 4
      |
      ▼
Iterator Checks Modification
      |
      ▼
4 != 3
      |
      ▼
ConcurrentModificationException
```

**Remember:** Fail-fast is a best-effort bug-detection mechanism, not a thread-safety guarantee.

## 6. What is modCount and expectedModCount?

| Key | Description |
|---|---|
| modCount | Internal counter that tracks structural modifications in many collection implementations. |
| expectedModCount | Modification count that the iterator expects while traversing. |

Example:

```text
Collection:
modCount = 5

Iterator Created:
expectedModCount = 5

External Structural Modification:

modCount = 6
expectedModCount = 5

Mismatch:
6 != 5

Result:
ConcurrentModificationException may occur
```

**Why doesn't Iterator.remove() cause CME?**

Because the iterator performs the removal and updates its expected modification count accordingly.

```text
Iterator.remove()
       |
       ▼
Collection Updated
       |
       ▼
modCount Changes
       |
       ▼
expectedModCount Updated
       |
       ▼
Iterator Continues
```

## 7. Iterator vs Enhanced For-Loop

The enhanced for-loop uses an iterator internally when traversing an Iterable collection.

```java
for (String name : names) {
    System.out.println(name);
}
```

Conceptually:

```java
Iterator<String> it = names.iterator();

while (it.hasNext()) {
    String name = it.next();
    System.out.println(name);
}
```

**Interview Trick:** Directly removing elements from the collection inside an enhanced for-loop can cause CME.

## 8. How to Safely Remove Elements During Iteration?

Two common approaches:

### Approach 1: Iterator.remove()

```java
Iterator<String> it = names.iterator();

while (it.hasNext()) {

    String name = it.next();

    if (name.equals("B")) {
        it.remove();
    }
}
```

### Approach 2: removeIf()

```java
names.removeIf(name -> name.equals("B"));
```

`removeIf()` is cleaner for simple condition-based removal.

## 9. What is ListIterator?

ListIterator is a specialized iterator for List implementations that supports both forward and backward traversal.

It also supports adding, replacing, and removing elements during traversal.

**Important methods:**

| Method | Description |
|---|---|
| hasNext() | Checks next element |
| next() | Returns next element |
| hasPrevious() | Checks previous element |
| previous() | Returns previous element |
| add() | Adds an element |
| set() | Replaces the last returned element |
| remove() | Removes the last returned element |

### Example

```java
List<String> names = new ArrayList<>(
        List.of("A", "B", "C")
);

ListIterator<String> it = names.listIterator();

// Forward traversal
while (it.hasNext()) {
    System.out.println(it.next());
}

// Backward traversal
while (it.hasPrevious()) {
    System.out.println(it.previous());
}
```

### Iterator vs ListIterator

| Iterator | ListIterator |
|---|---|
| Forward traversal | Forward and backward traversal |
| Works with Collection | Works with List |
| hasNext(), next() | Additional previous(), add(), set() |
| Supports remove() | Supports remove() |

## 10. What is a Weakly Consistent Iterator?

A weakly consistent iterator allows traversal while a concurrent collection is being modified.

Unlike fail-fast iterators, it does not throw `ConcurrentModificationException` merely because of concurrent modifications.

Example: `ConcurrentHashMap`.

```java
ConcurrentHashMap<Integer, String> map =
        new ConcurrentHashMap<>();

map.put(1, "A");
map.put(2, "B");
map.put(3, "C");

for (Integer key : map.keySet()) {

    System.out.println(key);

    map.remove(key);
}
```

The iterator can continue while entries are removed.

**Important:**

- It does not guarantee that every concurrent modification will be visible.
- It is not a snapshot of the collection.
- It does not guarantee a fixed view of the collection during traversal.

```text
Fail-Fast Iterator
       |
       ▼
Unexpected Structural Modification
       |
       ▼
CME May Occur


Weakly Consistent Iterator
       |
       ▼
Concurrent Modification
       |
       ▼
Iteration Can Continue
```

## 11. Important Question Tricks

1. `hasNext()` does not advance the iterator; `next()` does.
2. Calling `next()` without another element throws `NoSuchElementException`.
3. `remove()` removes the last element returned by `next()`.
4. Calling `remove()` before `next()` throws `IllegalStateException`.
5. Calling `remove()` twice without another `next()` also throws `IllegalStateException`.
6. CME can occur even in a single-threaded application.
7. Enhanced for-loops use iterators internally for Iterable collections.
8. `Iterator.remove()` is the safe removal mechanism during iterator traversal.
9. `removeIf()` is useful for condition-based removal.
10. `ListIterator` supports backward traversal.
11. `ConcurrentHashMap` uses weakly consistent iterators.
12. Fail-fast behavior is best-effort, not guaranteed.

## 12. Quick Revision

```text
Iterator
   |
   ├── hasNext() → Check next element
   |
   ├── next() → Return and advance
   |
   ├── remove() → Remove last returned element
   |
   ├── Fail-Fast → CME may occur
   |
   ├── modCount → Tracks structural modifications
   |
   ├── expectedModCount → Iterator's expected count
   |
   ├── ListIterator → Forward + Backward
   |
   └── Weakly Consistent → ConcurrentHashMap
```

**Remember:**

- Iterator → Sequential traversal.
- Iterator.remove() → Safe removal during traversal.
- Fail-Fast → Detects unexpected structural modification.
- ListIterator → Bidirectional traversal.
- Weakly Consistent → Allows concurrent modification during iteration.

**Most Important Interview Concept:** Understand how `modCount` and `expectedModCount` work, why direct collection modification can cause CME, and why `Iterator.remove()` avoids that problem.