# this Keyword

### What is this?

this is a reference variable that refers to the current object.

### Why Do We Need this?

Suppose you have:
```
class Student {

    String name;
    int age;

    Student(String name, int age) {

        name = name;
        age = age;

    }
}
```
Looks correct?

No.

Both __name__ and __age__ on the left refer to the constructor parameters, not the object's fields.

After creating:
```
Student s = new Student("Rahul", 20);
```
The object contains:
```
name = null
age = 0
```
because you assigned the parameters to themselves.

Using this

```
class Student {

    String name;
    int age;

    Student(String name, int age) {

        this.name = name;
        this.age = age;

    }
}
```
Now Java understands:

```
this.name → object's field
name → constructor parameter
```

### Memory View
```
Student s = new Student("Rahul", 20);
```

When the constructor executes:

```
Current Object

name = null
age = 0

↓

this

Inside the constructor:

this.name = name;

means

Current Object.name = constructor parameter name

Result:

Current Object

name = Rahul
age = 20
```


### this in Methods

```
class User {

    String name;

    User( String name) {
        this.name = name;
    }

    void printName() {

        System.out.println(this.name);

    }
}

Student s = new Student("Rahul");

s.printName();

Output

Rahul
```

Here this refers to s.

### What Does this Refer To?

```
Student s1 = new Student("Rahul");
Student s2 = new Student("Amit");

When:

s1.printName();

Inside the method

this → s1

When:

s2.printName();

Inside the method

this → s2
```

this always refers to the object that called the method.

## Common Uses of this

### 1. Differentiate fields from parameters ✅

Most common use.
```
this.name = name;
```

### 2. Call another constructor

```
class Student {

    String name;
    int age;

    Student() {

        this("Unknown", 18);

    }

    Student(String name, int age) {

        this.name = name;
        this.age = age;

    }

}

Creating

new Student();

actually calls the second constructor.

This is called constructor chaining.

3. Pass Current Object
class Student {

    void register() {

        School.addStudent(this);

    }

}

this passes the current object.

4. Return Current Object

Useful in method chaining.

class Calculator {

    Calculator add() {

        return this;

    }

}

Now

calculator.add().add().add();

is possible.

Interview Questions
Why do we use this?

To refer to the current object.

Is this a keyword?

Yes.

Can we use this in a static method?

No.

static void test(){

    System.out.println(this);
}

Compile Error.

Why?

Because static methods belong to the class, not to any particular object. Since there is no current object, there is no this.

Can we assign this?

No.

this = anotherObject;

Not allowed.

Real Project Example

Imagine your ticket system.

class Ticket {

    String title;
    String status;

    Ticket(String title) {

        this.title = title;
        this.status = "Open";

    }

}

Every time you create

Ticket t = new Ticket("Login Issue");

this.title refers to that specific ticket object, not any other ticket.