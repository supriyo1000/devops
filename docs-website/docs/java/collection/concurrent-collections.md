# Concurrent Collections — Internal Working & Important Concepts

## 1. What are Concurrent Collections?

Concurrent collections are thread-safe collection implementations designed for multiple threads accessing shared data.

They help prevent data corruption and provide concurrency-specific behavior without requiring developers to synchronize every operation manually.

```text
                  Concurrent Collections
                           |
          +----------------+----------------+
          |                |                |
    ConcurrentMap        List             Queue
          |                |                |
   ConcurrentHashMap  CopyOnWriteArrayList  BlockingQueue
                                               |
                               +---------------+---------------+
                               |               |               |
                         ArrayBlockingQueue  LinkedBlockingQueue  SynchronousQueue
```

## 2. ConcurrentHashMap

### What is it?

`ConcurrentHashMap` is a thread-safe implementation of `Map` that allows multiple threads to access and update the map concurrently.

Unlike `Hashtable`, it does not use one global lock for every operation.

### Internal Architecture

In modern Java implementations, ConcurrentHashMap uses:

- Hash table (array of bins).
- CAS (Compare-And-Set) for certain updates.
- `synchronized` locking on individual bins when necessary.
- Linked lists or Red-Black Trees for collision handling.

```text
             ConcurrentHashMap
                     |
                Hash Table
                     |
          +----------+----------+
          |          |          |
        Bin 0      Bin 1      Bin 2
          |          |          |
        Node       Node       Node
          |                     |
        Node                 TreeBin
```

Different bins can often be updated concurrently.

**Important:** Modern ConcurrentHashMap does not use the old Java 7 segment-based locking architecture.

### Example

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentMapDemo {

    public static void main(String[] args) {

        ConcurrentHashMap<Integer, String> map =
                new ConcurrentHashMap<>();

        map.put(1, "Java");
        map.put(2, "Spring");

        map.putIfAbsent(1, "Python");

        System.out.println(map.get(1)); // Java

        map.compute(2, (key, value) -> value + " Boot");

        System.out.println(map);
    }
}
```

### Important Methods

```java
map.putIfAbsent(1, "Java");

map.compute(1, (key, value) -> value + " Updated");

map.computeIfAbsent(2, key -> "Spring");

map.computeIfPresent(1, (key, value) -> value + " Boot");

map.merge(1, " New", String::concat);
```

These methods provide atomic operations for their respective mapping updates.

### Important Interview Points

- Thread-safe without locking the entire map for every operation.
- Does not allow null keys or null values.
- Iterators are weakly consistent, not fail-fast.
- Supports concurrent reads and updates.
- `size()` is not necessarily an instantaneous snapshot during concurrent modifications.
- Use `computeIfAbsent()` for atomic creation of a missing mapping.

**When to use:** Shared key-value storage accessed by multiple threads.

---

## 3. CopyOnWriteArrayList

### What is it?

`CopyOnWriteArrayList` is a thread-safe list optimized for situations where reads are frequent and modifications are relatively rare.

Its main idea:

**Every modification creates a new copy of the underlying array.**

### Internal Working

Suppose the current list is:

```text
Original Array
+-----+-----+-----+
|  A  |  B  |  C  |
+-----+-----+-----+
```

When adding `D`:

```text
Step 1: Create a new array

+-----+-----+-----+-----+
|  A  |  B  |  C  |  D  |
+-----+-----+-----+-----+

Step 2: Replace the internal array reference

Old Array → New Array
```

Readers can continue reading the old array while the new array is being prepared.

### Example

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteDemo {

    public static void main(String[] args) {

        CopyOnWriteArrayList<String> list =
                new CopyOnWriteArrayList<>();

        list.add("Java");
        list.add("Spring");
        list.add("SQL");

        for (String item : list) {
            System.out.println(item);

            if (item.equals("Java")) {
                list.add("Hibernate");
            }
        }

        System.out.println(list);
    }
}
```

The iterator uses a snapshot of the array created when the iterator was constructed.

Therefore, `"Hibernate"` is added to the list but is not seen by the current iterator.

### Important Points

- Thread-safe.
- Reads are generally fast and do not require locking.
- Every write creates a new array.
- Iterators are snapshot-based.
- Iterator does not support `remove()`.
- High memory and copying cost when modifications are frequent.

**When to use:** Read-heavy data such as listener lists, configuration lists, and frequently read application settings.

---

## 4. BlockingQueue

### What is it?

`BlockingQueue` is an interface in `java.util.concurrent` used for thread-safe communication between producers and consumers.

Its key feature is blocking behavior.

- If the queue is empty, `take()` waits until an element becomes available.
- If the queue is full, `put()` waits until space becomes available.

### Producer-Consumer Architecture

```text
       Producer Thread
              |
              | put()
              v
       +--------------+
       | BlockingQueue|
       +--------------+
              |
              | take()
              v
       Consumer Thread
```

### Important Methods

| Method | Behavior |
|---|---|
| add(e) | Inserts or throws exception if full |
| offer(e) | Inserts or returns false if full |
| put(e) | Waits until space is available |
| offer(e, time, unit) | Waits for limited time |
| remove() | Removes or throws exception if empty |
| poll() | Removes or returns null if empty |
| take() | Waits until an element is available |
| poll(time, unit) | Waits for limited time |

### Practical Example

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ProducerConsumerDemo {

    public static void main(String[] args) {

        BlockingQueue<Integer> queue =
                new ArrayBlockingQueue<>(3);

        Thread producer = new Thread(() -> {

            try {
                for (int i = 1; i <= 5; i++) {
                    queue.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {

            try {
                for (int i = 1; i <= 5; i++) {
                    Integer value = queue.take();
                    System.out.println("Consumed: " + value);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
    }
}
```

The producer waits if the queue becomes full. The consumer waits if the queue becomes empty.

**Important:** `BlockingQueue` is an interface, not a concrete collection implementation.

---

## 5. ArrayBlockingQueue

### What is it?

`ArrayBlockingQueue` is a bounded, FIFO blocking queue backed by a fixed-size array.

Bounded means its capacity is fixed when the queue is created.

### Internal Architecture

```text
       ArrayBlockingQueue (capacity = 4)

       +-----+-----+-----+-----+
       |  A  |  B  |  C  |     |
       +-----+-----+-----+-----+
          ^                 ^
        Take              Put
        Index             Index
```

It uses:

- Fixed-size array.
- Circular buffer.
- Lock and condition variables for blocking coordination.

### Example

```java
import java.util.concurrent.ArrayBlockingQueue;

public class ArrayQueueDemo {

    public static void main(String[] args) throws InterruptedException {

        ArrayBlockingQueue<Integer> queue =
                new ArrayBlockingQueue<>(3);

        queue.put(10);
        queue.put(20);
        queue.put(30);

        System.out.println(queue);

        System.out.println(queue.take()); // 10
        System.out.println(queue.take()); // 20
    }
}
```

### Important Points

- Fixed capacity.
- FIFO ordering.
- Does not allow null elements.
- Uses an array internally.
- Supports optional fairness configuration.

```java
ArrayBlockingQueue<Integer> queue =
        new ArrayBlockingQueue<>(10, true);
```

`true` enables fair access to the queue's waiting threads, generally favoring the longest-waiting thread.

**When to use:** When you need predictable, fixed memory usage and bounded buffering.

---

## 6. LinkedBlockingQueue

### What is it?

`LinkedBlockingQueue` is a FIFO blocking queue backed by linked nodes.

It can be bounded or effectively unbounded.

### Internal Architecture

```text
       LinkedBlockingQueue

       Head
        |
        v
     +------+     +------+     +------+
     |  A   | --> |  B   | --> |  C   |
     +------+     +------+     +------+
                                   |
                                  Tail
```

Each node stores an element and a reference to the next node.

### Example

```java
import java.util.concurrent.LinkedBlockingQueue;

public class LinkedQueueDemo {

    public static void main(String[] args) throws InterruptedException {

        LinkedBlockingQueue<String> queue =
                new LinkedBlockingQueue<>(3);

        queue.put("Java");
        queue.put("Spring");
        queue.put("SQL");

        System.out.println(queue.take()); // Java
    }
}
```

### Important Points

- FIFO ordering.
- Uses linked nodes internally.
- Can have a specified capacity.
- Default constructor has a theoretical capacity of `Integer.MAX_VALUE`.
- Does not allow null elements.
- Uses separate put and take locks with an atomic count in its standard implementation.

**When to use:** When you need a producer-consumer queue with linked-node storage and potentially higher throughput under suitable workloads.

---

## 7. SynchronousQueue

### What is it?

`SynchronousQueue` is a special blocking queue that does not store elements.

It has effectively zero capacity.

Every insertion must wait for a corresponding removal, and every removal must wait for a corresponding insertion.

### Internal Working

```text
        Producer Thread
               |
               | put("Java")
               |
               v
        SynchronousQueue
               |
               | Direct handoff
               v
        Consumer Thread
               |
               | take()
               v
           Receives Java
```

There is no element buffer.

### Example

```java
import java.util.concurrent.SynchronousQueue;

public class SynchronousQueueDemo {

    public static void main(String[] args) {

        SynchronousQueue<String> queue =
                new SynchronousQueue<>();

        Thread producer = new Thread(() -> {
            try {
                queue.put("Java");
                System.out.println("Transferred Java");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {
            try {
                String value = queue.take();
                System.out.println("Received: " + value);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        consumer.start();
        producer.start();
    }
}
```

The producer's `put()` completes only when a consumer receives the element.

### Important Points

- Zero capacity.
- No element storage.
- Direct thread-to-thread handoff.
- `put()` waits for a consumer.
- `take()` waits for a producer.
- Supports optional fairness configuration.

**When to use:** Direct task handoff between threads, including certain thread-pool execution strategies.

---

## 8. Compare the Three BlockingQueue Implementations

| Feature | ArrayBlockingQueue | LinkedBlockingQueue | SynchronousQueue |
|---|---|---|---|
| Internal structure | Array | Linked nodes | Handoff mechanism |
| Capacity | Fixed | Bounded or effectively unbounded | Zero |
| FIFO | Yes | Yes | Handoff, no stored order |
| Stores elements | Yes | Yes | No |
| Memory | Fixed array | Allocates nodes | No element buffer |
| Main purpose | Bounded buffering | Producer-consumer buffering | Direct handoff |

### Easy way to remember

```text
ArrayBlockingQueue
    → Fixed array
    → Fixed capacity

LinkedBlockingQueue
    → Linked nodes
    → Configurable capacity

SynchronousQueue
    → No storage
    → Direct handoff
```

---

## 9. Concurrent Collections vs Synchronized Collections

| Feature | Synchronized Collection | Concurrent Collection |
|---|---|---|
| Locking | Often one shared lock | Specialized concurrency mechanisms |
| Concurrent access | More contention possible | Designed for concurrent access |
| Iteration | Often requires external synchronization | Depends on collection's iterator semantics |
| Examples | Collections.synchronizedMap() | ConcurrentHashMap |

**Important:** Thread-safe does not automatically mean every multi-step operation is atomic.

For example:

```java
if (!map.containsKey(1)) {
    map.put(1, "Java");
}
```

This check-then-act sequence is not atomic.

Prefer:

```java
map.putIfAbsent(1, "Java");
```

---

## 10. Important Interview Points

1. ConcurrentHashMap supports concurrent access without one global lock for every operation.
2. Modern ConcurrentHashMap uses CAS and bin-level synchronization where needed.
3. ConcurrentHashMap does not allow null keys or values.
4. CopyOnWriteArrayList copies its underlying array on modifications.
5. CopyOnWriteArrayList iterators use snapshots.
6. BlockingQueue supports blocking insertion and removal.
7. `put()` waits for available capacity; `take()` waits for an available element.
8. ArrayBlockingQueue uses a fixed-size circular array.
9. LinkedBlockingQueue uses linked nodes and can be bounded or effectively unbounded.
10. SynchronousQueue has zero capacity and performs direct handoff.
11. Concurrent collections have different iterator consistency guarantees; do not assume they are all fail-fast or snapshot-based.
12. Thread-safe individual methods do not guarantee atomicity across multiple method calls.

## Quick Revision

```text
ConcurrentHashMap
    → Concurrent key-value storage
    → CAS + bin-level synchronization

CopyOnWriteArrayList
    → Read-heavy list
    → Copy array on write
    → Snapshot iterator

BlockingQueue
    → Producer-consumer communication
    → put() and take() can block

ArrayBlockingQueue
    → Fixed array
    → Fixed capacity

LinkedBlockingQueue
    → Linked nodes
    → Bounded or effectively unbounded

SynchronousQueue
    → Zero capacity
    → Direct handoff
```

**Final selection rule:**

- Need concurrent Map operations → `ConcurrentHashMap`.
- Need a read-heavy thread-safe List → `CopyOnWriteArrayList`.
- Need producer-consumer communication → `BlockingQueue`.
- Need fixed-capacity buffering → `ArrayBlockingQueue`.
- Need linked-node buffering → `LinkedBlockingQueue`.
- Need direct thread-to-thread transfer → `SynchronousQueue`.