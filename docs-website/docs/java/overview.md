# Java Tutorial

java is a high level, object oriented programming language. used in

+ web development(backend)
+ android apps
+ Desktop apps
+ Enterprise system

### Key features

+ Platform independant
+ Object oriented --> based on class and objects
+ Secure --> no direct memory access
+ Multithreading  --> can run multiple tasks at a time.

### How java works

+ You write code --> Hello java
+ compiler(javac) converts it into bytecode(.class)
+ JVM (java virtual machine) runs the bytecode

That's why java is platform independant.

### Java components

+ __JDK__ : java development kit (for developers --> compiler + tools)
+ __JRE__ : java runtime environment (to run java program)
+ __JVM__ : java virtual machine (executes bytecodes)

### First java code

```ruby
public class main {
    public static void main(String[] args) {
        System.out.println("Hello world");
    }
}
```

__varialbe__ : cannot declare variable with 1abc.

## Functions

+ void --> no return
+ int,double --> must return
+ static --> 

## Array

int[] arr = {1,2,3,4,5};

### Create array dynamic

int[] arr2 = new int[5];

arr2[0] = 1;
arr2[1] = 2;

#### Examle 1 :

```ruby
public class main {

    static int findsum(int[] arr){
        int sum = 0;
        for (int i = 0; i < arr.length; i++) {
            sum+= arr[i];
        }

        return sum;
    }
    public static void main(String[] args) {
        int[] array = {1,2,3,4,5};

        int result = findsum(array);
        System.out.println(result);
    }
}
```

#### Examle 2 :

```ruby
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class main {

    public static Map<Integer , Integer> maxelm(int[] array) {
        Map<Integer , Integer> map = new HashMap<>();

        for (int elm : array) {
            map.put(elm, (map.getOrDefault(elm, 0)) + 1);
        }

        return map;
    }
    public static void main(String[] args) {
        int[] arr = {1,2,3,4,2,4,5};
        Map<Integer , Integer> result = maxelm(arr);

        int maxcount = 1;
        int maxElement = -1;

        for(Map.Entry<Integer , Integer> entry : result.entrySet()) {
            if(entry.getValue() > maxcount) {
                maxcount = entry.getValue();
                maxElement = entry.getKey();
            }
        }

        System.out.println(maxElement + ":" + maxcount);
    }
}
```

#### Examle 3 :

```ruby
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class main {

    public static List<Integer> evenelm(int[] array) {
        Map<Integer,Integer> map = new HashMap<>();
        List<Integer> list = new ArrayList<>();
        
        for (int num : array) {
            map.put(num, map.getOrDefault(num , 0) +1);
        }

        for(Map.Entry<Integer,Integer> entry : map.entrySet()){
            if((entry.getValue() % 2) == 0) {
                list.add(entry.getKey());
            }
        }

        return list;
    }

    public static  void main (String[] args) {
        int[] arr = {1,2,3,4,2,4,5};
        List<Integer> result = evenelm(arr);
        System.out.println(result);
    }
}
```

## Strings

string is not primitive.
it is an object(class).

### common String operations

```ruby
String s = "hello";
System.out.println(s.length());
System.out.println(s.charAt(0));

char[] sarr = s.toCharArray();
System.out.println(sarr);
```

```ruby
String a = "hello";
String b = "hello";
System.out.println(a.equals(b));
```

### String Immutability
This creates a new object. Not modifying existing.

```ruby
String a = "hello";
String greet = a + "Supriyo";
```

+ slow loop
+ Memory heavy

## StringBuilder

Used when modifying strings frequently.

```ruby
public class main {
    public static void main(String[] args) {
        String a = "world";
        
        StringBuilder sb = new StringBuilder("hello ");
        sb.append(a);
        System.out.println(sb);
        sb.setCharAt(0, 'H');
        System.out.println(sb);
        sb.reverse();
        System.out.println(sb);
    }
}
```

