# Iterator

## 1. What is an Iterator?

An Iterator is an object used to traverse elements of a collection one by one. It provides methods such as hasNext(), next(), and remove().

### Explanation

Instead of accessing elements by index, an iterator maintains a position while traversing the collection.

```
List<String> names = List.of("A", "B", "C");

Iterator<String> it = names.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}

Flow:

A → B → C
```
Remember

Iterator = traverse collection sequentially

## 2. What does hasNext() do?

hasNext() checks whether another element is available in the collection. It returns true if another element exists, otherwise false.

Example:
```
Iterator<String> it = names.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

### Explanation

Suppose:

[A, B, C]
 ↑
iterator

hasNext():

true

Then:

it.next();

returns A.

Eventually:

[A, B, C]
         ↑

After C has been returned:

it.hasNext();

returns:

false
Important

hasNext() does not move the iterator.

hasNext() → checks
next()    → moves


## 3. What does next() do?

next() returns the next element and advances the iterator to the next position.

Example:
```
Iterator<String> it = names.iterator();


System.out.println(it.next()); // A
System.out.println(it.next()); // B
System.out.println(it.next()); // C

Think:

[A, B, C]
 ↑
next() → A


[A, B, C]
    ↑
next() → B


[A, B, C]
       ↑
next() → C
```
Important

next() both:

Returns an element
Moves the iterator forward


## 4. What happens if next() has no element?

If next() is called when no element remains, it throws NoSuchElementException.

Example:
```
List<String> names = List.of("A");

Iterator<String> it = names.iterator();

System.out.println(it.next()); // A
System.out.println(it.next()); // ❌
```
The second call throws:

NoSuchElementException

Correct approach

Always check:
```
if (it.hasNext()) {
    System.out.println(it.next());
}
```
Or:
```
while (it.hasNext()) {
    System.out.println(it.next());
}
```
Remember

No next element + next()
        ↓
NoSuchElementException


## 5. Why use Iterator.remove()?

Iterator.remove() safely removes the element that was most recently returned by next() while iterating.

Example:
```
List<String> names =
        new ArrayList<>(List.of("A", "B", "C"));

Iterator<String> it = names.iterator();

while (it.hasNext()) {
    String name = it.next();

    if (name.equals("B")) {
        it.remove();
    }
}
```

Result:

[A, C]


### Why not list.remove()?

Because you're modifying the collection directly while its iterator is active.

list.remove(name); // ❌

Instead:

it.remove();       // ✅
Important rule

remove() removes the element returned by the most recent next().

Sequence:

next()
  ↓
element returned
  ↓
remove()
  ↓
that element removed


## 6. What is fail-fast behavior?

Fail-fast behavior means an iterator attempts to detect unexpected structural modification of a collection during iteration and may throw ConcurrentModificationException.

Example:
```
List<String> names =
        new ArrayList<>(List.of("A", "B", "C"));

Iterator<String> it = names.iterator();

while (it.hasNext()) {
    String name = it.next();

    if (name.equals("B")) {
        names.remove(name); // ❌
    }
}
```

The iterator is traversing the list:

Iterator
   ↓
[A, B, C]

But the list is directly modified:

list.remove("B")

The iterator detects the unexpected structural modification and may throw:

ConcurrentModificationException

### Why "fail-fast"?

Because instead of continuing with potentially inconsistent traversal, the iterator fails quickly.
```
Unexpected modification
        ↓
Iterator detects it
        ↓
ConcurrentModificationException
```

## 7. Why can list.remove() cause ConcurrentModificationException?

Because the enhanced for-loop or iterator is traversing the collection, while list.remove() directly modifies the collection's structure. The iterator detects that the collection has changed unexpectedly.

Example:
```
for (String name : names) {
    if (name.equals("B")) {
        names.remove(name);
    }
}
```
Remember:
```
Enhanced for-loop
       ↓
Iterator internally
       ↓
list.remove()
       ↓
Collection modified independently
       ↓
ConcurrentModificationException
```

Correct approach
```
Iterator<String> it = names.iterator();

while (it.hasNext()) {
    String name = it.next();

    if (name.equals("B")) {
        it.remove();
    }
}
```


## 8. What is ListIterator?

ListIterator is a specialized iterator for List implementations that supports forward and backward traversal and also provides add() and set() operations.

Example:
```
List<String> names =
        new ArrayList<>(List.of("A", "B", "C"));

ListIterator<String> it = names.listIterator();

Forward:

while (it.hasNext()) {
    System.out.println(it.next());
}

Backward:

while (it.hasPrevious()) {
    System.out.println(it.previous());
}
```

Additional methods

```
hasPrevious()
previous()
add()
set()
remove()
```

## 9. Iterator vs ListIterator?

Iterator supports forward traversal, while ListIterator supports both forward and backward traversal and provides additional operations such as add() and set().


##  10. Why doesn't ConcurrentHashMap throw ConcurrentModificationException?

ConcurrentHashMap is designed for concurrent access. Its iterators are weakly consistent, meaning they can continue while the map is being modified instead of failing with ConcurrentModificationException merely because of those modifications.

Example:
```
ConcurrentHashMap<Integer, String> map =
        new ConcurrentHashMap<>();

map.put(1, "A");
map.put(2, "B");

for (Integer key : map.keySet()) {
    map.remove(key);
}
```
The iterator is designed to tolerate modifications.

Compare:

Normal HashMap:
```
Modification during iteration
        ↓
Fail-fast behavior
        ↓
CME may occur
```
ConcurrentHashMap:
```
Modification during iteration
        ↓
Weakly consistent iterator
        ↓
Iteration can continue
```

## 11. What is a weakly consistent iterator?

A weakly consistent iterator can safely traverse a collection while it is being modified concurrently. It does not throw ConcurrentModificationException merely because the collection changes, and it does not provide a snapshot of the collection.

This is associated with concurrent collections such as:

ConcurrentHashMap
Important point

It doesn't mean:

"The iterator always sees every modification."

Instead, modifications that happen during iteration may or may not be reflected depending on the iterator and timing.

So:
```
Weakly consistent
        ≠
Snapshot
```

12. Why does the enhanced for-loop throw ConcurrentModificationException when removing from an ArrayList?

The enhanced for-loop uses an iterator internally. If we directly modify the ArrayList while the iterator is traversing it, the collection's structural modification count changes. The iterator detects the unexpected modification and may throw ConcurrentModificationException.

Example:
```
List<Integer> numbers =
        new ArrayList<>(List.of(1, 2, 3, 4));

for (Integer n : numbers) {
    if (n == 2) {
        numbers.remove(n);
    }
}

Conceptually:

Iterator<Integer> it = numbers.iterator();

while (it.hasNext()) {
    Integer n = it.next();
    // ...
}

Then:

numbers.remove(n);
```

modifies the collection outside the iterator.

Therefore:
```
Enhanced for-loop
       ↓
Iterator
       ↓
Direct collection modification
       ↓
Structural modification detected
       ↓
ConcurrentModificationException
```

13. How do you safely remove while iterating?

There are two important approaches.

Approach 1: Iterator.remove()
```
Iterator<String> it = list.iterator();

while (it.hasNext()) {
    String value = it.next();

    if (value.equals("B")) {
        it.remove();
    }
}
```

Approach 2: removeIf()

```
list.removeIf(value -> value.equals("B"));
```
For simple removal conditions, removeIf() is often cleaner.

We can use Iterator.remove() when manually iterating, or removeIf() when we have a simple predicate-based removal condition.

## 14. Is ConcurrentModificationException related only to multiple threads?

No. ConcurrentModificationException can occur even in a single thread when a collection is structurally modified while an iterator is being used.

Single-thread example:
```
Iterator<String> it = list.iterator();

while (it.hasNext()) {
    String value = it.next();
    list.remove(value); // ❌
}
```

There is only:

Thread 1
   ↓
Iterator
   +
Direct collection modification

Still potentially:

ConcurrentModificationException
Important interview clarification

The word concurrent here doesn't necessarily mean:

multiple threads

It means that the collection is being modified independently while an iteration is in progress.

## 15. What is the difference between fail-fast and weakly consistent iterators?

Fail-fast iterators attempt to detect structural modification and may throw ConcurrentModificationException. Weakly consistent iterators, such as those used by ConcurrentHashMap, allow iteration to continue while modifications occur and do not provide a snapshot of the collection.

	
## 16. What is modCount?

modCount is an internal modification count used by many fail-fast collection implementations to track structural modifications.

Conceptually:
```
Collection
    │
    └── modCount = 5
```
Iterator is created:

expectedModCount = 5

Then the collection changes:

modCount = 6
expectedModCount = 5

Iterator detects:

6 != 5

and may throw:

ConcurrentModificationException


## 17. What is expectedModCount?

expectedModCount is the modification count that the iterator expects the collection to have while it is iterating.

Conceptually:

Collection:
modCount = 10

Iterator:
expectedModCount = 10

Everything is fine:
10 == 10

After an external modification:

Collection:
modCount = 11


Iterator:
expectedModCount = 10

Mismatch:

11 != 10

Therefore the iterator may fail fast.

## 18. Why doesn't Iterator.remove() cause CME?

Because the removal is performed through the iterator itself, allowing the iterator to update its expected modification state appropriately.

Conceptually:
```
Iterator.remove()
      ↓
Collection modified
      ↓
modCount changes
      ↓
Iterator updates expectedModCount
      ↓
Iterator remains synchronized
```
Whereas:
```
list.remove()
      ↓
modCount changes
      ↓
Iterator's expectedModCount unchanged
      ↓
Mismatch
      ↓
CME may occur
```


## 19. Can we call remove() before next()?
No.

This is invalid:

Iterator<String> it = list.iterator();


it.remove(); // ❌

You must first call:

it.next();

then:

it.remove();

Correct:

it.next();
it.remove();
Another trap

This is also invalid:

it.next();
it.remove();
it.remove(); // ❌

You normally need another next() before another remove().
```
Remember:

next()
  ↓
remove()
  ↓
next()
  ↓
remove()
```