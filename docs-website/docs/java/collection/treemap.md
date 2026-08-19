# TreeMap

## What is TreeMap?

TreeMap is a Map implementation that stores entries sorted according to the keys. Internally, it uses a Red-Black Tree, so operations like put, get, and remove take O(log n) time. It also provides navigation methods such as firstKey, lastKey, higherKey, and lowerKey.

## 2. How is TreeMap different from HashMap?
|Feature	|HashMap	|TreeMap|
|-----------|-----------|-------|
|Ordering	|No guaranteed |sorted order(Sorted by key)|
|Internal structure	|Hash table	|Red-Black Tree|
|put()	|Average O(1)	|O(log n)|
|get()	|Average O(1)	|O(log n)|
|remove()	|Average O(1)	|O(log n)|
|Navigation	|Limited	|Rich navigation methods|
|null key	|Allows one	|Generally doesn't allow null key with natural ordering|

Interview point:

Use HashMap when fast lookup is the priority. Use TreeMap when you need keys to remain sorted or need range/navigation operations.

## 3. Why is TreeMap O(log n)?

Because it uses a self-balancing Red-Black Tree.

The tree remains approximately balanced, so searching/inserting/removing requires traversing roughly the height of the tree.

Therefore:

Search → O(log n)
Insert → O(log n)
Delete → O(log n)

You don't need to explain the rotation/recoloring algorithm unless specifically asked.

## 4. Does TreeMap sort keys or values?

Keys.

TreeMap<Integer, String> map = new TreeMap<>();


map.put(30, "Apple");
map.put(10, "Mango");
map.put(20, "Banana");

Result:

{10=Mango, 20=Banana, 30=Apple}

The keys are sorted.

The values don't determine the ordering.

## 5. How does TreeMap know how to sort custom objects?

Either:

The key implements Comparable, or
You provide a Comparator.

Example:

TreeMap<Employee, String> map =
        new TreeMap<>(Comparator.comparing(Employee::getSalary));

Now employees are ordered according to salary.

## 6. What is the difference between Comparable and Comparator in TreeMap?
Comparable

The class itself defines its natural ordering.

class Employee implements Comparable<Employee> {


    @Override
    public int compareTo(Employee other) {
        return this.id - other.id;
    }
}
Comparator

The ordering is provided externally.

TreeMap<Employee, String> map =
        new TreeMap<>(Comparator.comparing(Employee::getName));

Think:

Comparable
    ↓
Natural ordering


Comparator
    ↓
Custom ordering
## 7. Can TreeMap have duplicate keys?

No.

Like every Map:

map.put(10, "A");
map.put(10, "B");

Result:

{10=B}

The second value replaces the first.

## 8. Can TreeMap have duplicate values?

Yes.

map.put(10, "A");
map.put(20, "A");

Valid:

{10=A, 20=A}

Map uniqueness applies to keys, not values.

## 9. Can TreeMap have null keys?

This is a common interview question.

With the natural ordering, a TreeMap does not allow a null key because it needs to compare keys.

TreeMap<Integer, String> map = new TreeMap<>();


map.put(null, "A");

This results in NullPointerException.

Values can be null:

map.put(10, null);
## 10. What happens if Comparator returns 0?

Very important.

TreeMap uses the comparison result to determine whether two keys are considered equivalent.

For example:

TreeMap<String, Integer> map =
        new TreeMap<>(Comparator.comparingInt(String::length));


map.put("Java", 1);
map.put("Code", 2);

Both strings have length 4.

Comparator returns:

0

So TreeMap considers them the same key.

Result:

{Java=2}

This is an important difference from simply thinking about equals().

## 11. Does TreeMap use equals() to compare keys?

The important practical answer is:

TreeMap uses compareTo() or the supplied Comparator to determine key ordering and key equivalence.

So for TreeMap, your comparison logic is extremely important.

## 12. What are higherKey() and lowerKey()?
higherKey(key)

Returns the smallest key strictly greater than the given key.

lowerKey(key)

Returns the largest key strictly smaller than the given key.

Example:

10  20  30  40

For:

higherKey(20)

→ 30

For:

lowerKey(20)

→ 10

## 13. What are ceilingKey() and floorKey()?

These are also worth knowing.

ceilingKey()

Returns the smallest key greater than or equal to the given key.

floorKey()

Returns the largest key less than or equal to the given key.

Example:

10  20  30  40

For 25:

lowerKey(25)   → 20
floorKey(25)   → 20


higherKey(25)  → 30
ceilingKey(25) → 30

For 20:

lowerKey(20)   → 10
floorKey(20)   → 20


higherKey(20)  → 30
ceilingKey(20) → 20
Easy way to remember
lower     → <
floor     → <=


higher    → >
ceiling   → >=


## 14. When should you use TreeMap?

Use TreeMap when you need:

sorted keys
minimum/maximum key
nearest lower key
nearest higher key
range-based operations
ordered traversal

Example:

Find the nearest available price greater than a user's budget.

TreeMap is a strong choice.

## 15. Is TreeMap thread-safe?

No.

TreeMap is not synchronized/thread-safe by default.

For concurrent sorted-map requirements, you should know about:

ConcurrentSkipListMap

We'll cover that later.

## 16. Does TreeMap maintain insertion order?

No.

It maintains key-sorted order.

map.put(30, "C");
map.put(10, "A");
map.put(20, "B");

Output:

10, 20, 30

Not:

30, 10, 20

If you need insertion order:

LinkedHashMap