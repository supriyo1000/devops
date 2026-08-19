# TreeSet

## Q1. What is TreeSet?

Answer:

TreeSet is a Set implementation that stores unique elements in sorted order. Internally, it is backed by a TreeMap, which uses a Red-Black Tree.

## Q2. Does TreeSet allow duplicates?

No.

set.add(10); // true
set.add(10); // false


## Q3. How does TreeSet detect duplicates?

This is very important.

TreeSet uses comparison.

It uses:

Comparable.compareTo()

or

Comparator.compare()

If comparison returns 0, TreeSet considers the elements duplicates.

## Q4. Does TreeSet maintain insertion order?

No.

It maintains sorted order.

HashSet       → no guaranteed order
LinkedHashSet → insertion order
TreeSet       → sorted order


## Q5. What happens if I add null to TreeSet?

With natural ordering, adding null generally throws:

NullPointerException

because TreeSet needs to compare elements.

TreeSet<Integer> set = new TreeSet<>();


set.add(null); // NullPointerException

Don't rely on TreeSet for null elements.

## Q6. Can TreeSet store custom objects?

Yes, but the objects must have a way to be ordered.

Either implement Comparable:
```
class Employee implements Comparable<Employee> {
    int id;

    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
}
```
or provide a Comparator:
```
TreeSet<Employee> set =
    new TreeSet<>(Comparator.comparingInt(e -> e.id));
```

## Q7. What is the time complexity of TreeSet?

Basic operations:
```
add()       → O(log n)
remove()    → O(log n)
contains()  → O(log n)
```
Because it uses a balanced Red-Black Tree.

## Q8. TreeSet vs HashSet?

	|HashSet	|TreeSet|
    |-----------|-------|
|Ordering	|No guaranteed order|	Sorted|
|Internal structure|	Hash table	|Red-Black Tree|
|add()	|O(1) average	|O(log n)|
|contains()	|O(1) average	|O(log n)|
|Duplicates|	No|	No|
|Custom ordering|	No	|Yes|


## Q9. What is the difference between higher() and ceiling()?

Suppose:

10 20 30 40

For 30:

higher(30)  → 40
ceiling(30) → 30

Because:

higher  → strictly greater
ceiling → greater OR equal


## Q10. What is the difference between lower() and floor()?

For 30:

lower(30) → 20
floor(30) → 30

Because:

lower → strictly smaller
floor → smaller OR equal

## 1. Internal relationship with TreeMap ⭐⭐⭐

This is the first thing to understand.

When you create:

TreeSet<Integer> set = new TreeSet<>();

internally, TreeSet uses a TreeMap.

Conceptually:

TreeSet
   |
   | internally uses
   ↓
TreeMap
   |
   | stores keys in
   ↓
Red-Black Tree

For example:
```
set.add(10);
set.add(20);
set.add(30);
```
Conceptually, TreeSet is using something similar to:

TreeMap

```
10 → PRESENT
20 → PRESENT
30 → PRESENT
```
The important point is:

TreeSet stores its elements as keys in an internal TreeMap.

The values are basically dummy/presence values.

Why is this useful?

Because TreeMap already provides:

sorted ordering
Red-Black Tree
O(log n) searching
navigation such as higher/lower

TreeSet gets these capabilities through TreeMap.

## 2. Red-Black Tree ⭐⭐⭐

A Red-Black Tree is a self-balancing binary search tree.

You don't need to memorize all the Red-Black Tree balancing rules for normal Java interviews.

Understand this:

Normal BST
```

        10
          \
           20
             \
              30
                \
                 40
```
This can become almost like a linked list.

Searching could become:

O(n)

A Red-Black Tree keeps the tree approximately balanced:
```
          20
        /    \
      10      30
                \
                 40
```
Therefore tree height remains approximately:

O(log n)

So operations such as:

add()
remove()
contains()

are:

O(log n)

### Why is TreeSet O(log n)?

TreeSet is backed by a TreeMap, which uses a self-balancing Red-Black Tree, so searching, insertion, and deletion take O(log n) time.

That's enough.

## 3. Comparable vs Comparator ⭐⭐⭐

TreeSet needs to know:

How should these elements be ordered?

There are two ways.

Comparable

The class itself defines its natural ordering.
```
class Employee implements Comparable<Employee> {

    int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
}
```
Then:
```
TreeSet<Employee> employees = new TreeSet<>();
```
TreeSet automatically uses:

compareTo()

Comparator

The ordering is provided externally.
```
TreeSet<Employee> employees =
    new TreeSet<>(Comparator.comparingInt(e -> e.id));
```
Now TreeSet uses the supplied:

### Q: When would you use Comparator instead of Comparable?

Answer:

When I need a different ordering without modifying the class, or when I need multiple possible sorting strategies.

For example:
```
Employee by ID
Employee by name
Employee by salary
```
You can have multiple Comparators.

## 4. Duplicate detection via comparison ⭐⭐⭐

This is probably the most important TreeSet trap.

With HashSet, you generally think:

equals() + hashCode()

With TreeSet, think:

compareTo() / Comparator.compare()

Example:
```
TreeSet<Integer> set = new TreeSet<>();

set.add(10);
set.add(20);

System.out.println(set.add(10));

Output:

false
```
Why?

Because comparison determines that the new 10 is equal to the existing 10.

The critical rule is:

If comparison returns 0, TreeSet considers the elements duplicates.

Example with Comparator

```
TreeSet<String> set =
    new TreeSet<>(String.CASE_INSENSITIVE_ORDER);

set.add("Java");
set.add("JAVA");
set.add("java");

System.out.println(set);

Result:

[Java]
```
Why?

The comparator considers:
```
"Java"
"JAVA"
"java"
```
equivalent for ordering.

So:

compare(...) == 0

and TreeSet doesn't add another element.

TreeSet determines uniqueness using its ordering mechanism—compareTo() or Comparator.compare(). If the comparison returns 0, it treats the elements as duplicates.