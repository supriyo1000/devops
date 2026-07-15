# Java Methods

## Hashset (Unique Elements and Fast lookup)

+ use when unique elements
+ you need fast check (o(1))

### Methods

+ add(x);
+ remove(x);
+ contains(x);
+ size();

#### Example- 1 : 

```ruby
import java.util.HashSet;

public class main {

    public static void main(String[] args) {
        HashSet<Integer> set = new HashSet<>();
        set.add(1);
        set.add(2);
        set.add(2);
        System.out.println(set); // [1, 2]
    }
}
```

#### Problem 1: Check Duplicates

```ruby
import java.util.HashSet;

public class main {

    static boolean hasDuplicate(int[] arr) {
        HashSet<Integer> set = new HashSet<>();
        for (int i = 0; i < arr.length; i++) {
            if (set.contains(arr[i])) {
                return true;
            }
            set.add(arr[i]);
        }

        return false;
    }

    public static void main(String[] args) {
        int[] arr = {1,2,3,2,4,5};
        System.out.println(hasDuplicate(arr));
    }
}
```

## HashMap (Frequency , key-value)

A java HashMap is a data structure from java collection framework that stores data as a key value pair.it allows to get value faster using its unique key.

HashMap is faster because it uses a technique called hashing.Internally it stores data into buckets.when we insert a key value pair it calculates the keys hash code using hashcode() method and uses it to determine the buckets where the data should be stored.

when we retrive the data from bucket java again calculate the key's hashcode and directly goes to the corresponding bucket instead of searching through all the elements one by one.

Sometimes multiple keys points to the same bucket it called collision. in older java, collision were handled using linkedlist but from java 8, if too many elements are stored in same bucket it uses red black tree which improving the time complexity from o(n) to o(log n).

+ when you need counting.
+ when you need mapping.

#### Basic Example :

```ruby
public static void main(String[] args) {
    HashMap<Integer , Integer> map = new HashMap<>();
    map.put(1, 10);
    map.put(2, 20);
    System.out.println(map); // {1=10, 2=20}
}
```

### Methods

+ map.put(key, value);
+ map.get(key);
+ map.containsKey(key);
+ map.getOrDefault(key, 0);
+ map.remove(key);

#### Problem 2: Count Frequency

```ruby
import java.util.HashMap;

public class main {

    static void countFrequency(int[] arr) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            map.put(arr[i], map.getOrDefault(arr[i] , 0) + 1);
        }

        System.out.println(map); // {1=1, 2=2, 3=1, 4=2, 5=1, 6=1}
    }

    public static void main(String[] args) {
        int[] arr = {1,2,3,5,4,2,4,6};
        countFrequency(arr);
    }
}
```

#### Problem 3: Two sum

exists a pair whose sum equals the target.

```ruby
static int[] twoSum(int[] arr, int target) {
    HashMap<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < arr.length; i++) {
        int diff = target - arr[i];

        if (map.containsKey(diff)) {
            return new int[]{map.get(diff), i};
        }

        map.put(arr[i], i);
    }

    return new int[]{-1, -1};
}
```






