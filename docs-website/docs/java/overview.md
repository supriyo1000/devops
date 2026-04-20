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

