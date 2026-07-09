# Java classes and objects

### What is a Class?

A class is a blueprint or template used to define the properties (data) and behaviors (methods) of a particular type of object.

A class does not occupy memory for actual data until an object is created.

Syntax
```
class Student {
    String name;
    int age;
    int roll;

    void study() {
        System.out.println("Studying...");
    }
}
```

### What is an Object?

An object is a real instance of a class. It contains its own copy of the class's fields and can call the methods defined in the class.

Creating an Object
```
Student s1 = new Student();

s1.name = "Rahul";
s1.age = 20;
s1.roll = 101;

s1.study();
```

### Why Do We Need Classes?

Without classes:
```
String student1Name;
int student1Age;

String student2Name;
int student2Age;
```

This becomes difficult to manage as the number of students grows.

With a class:
```
Student s1 = new Student();
Student s2 = new Student();
Student s3 = new Student();
```
One class can create thousands or even millions of objects.

State and Behavior

Every object has:
```
State (Fields)

+ name
+ age
+ roll
```
These represent the object's data.
```
Behavior (Methods)
+ study()
+ sleep()
+ play()
```
These represent the actions the object can perform.

### When Should We Create a Class?

Create a class whenever your application has an entity that has:

+ Data (properties)
+ Behavior (actions)

Examples:
```
Student
Employee
Customer
Product
Order
Ticket
User
Car
```

#### Key Points to Remember
```
Class = Blueprint
Object = Instance of a class
new creates an object in memory
One class can create many objects
Objects have state (fields) and behavior (methods)
Classes improve code reusability, maintainability, and organization
Common Interview Questions
```

#### Q1. What is a class?
A blueprint that defines the properties and behaviors of objects.

#### Q2. What is an object?
A runtime instance of a class with its own data.

#### Q3. Why do we use classes?
To represent real-world entities, avoid code duplication, and organize related data and behavior together.

#### Q4. Can one class create multiple objects?
Yes. A single class can create any number of independent objects.

#### Q5. Does a class occupy memory?
The class definition itself is loaded once by the JVM, but the instance data is allocated only when objects are created.


# Object vs Reference

+ __Object__ → The actual instance of a class that stores data. Created using new.
+ __Reference__ → A variable that stores the address of an object.

```
Student s1 = new Student();
```

+ __new Student()__ → Creates a new object in the Heap.
+ __s1__ → A reference variable that stores the object's address.

```
Stack                  Heap

s1  ------------->   Student Object
```

### One Object, Multiple References

```
Student s1 = new Student();
Student s2 = s1;

s1 ───┐
      ├────► Student Object
s2 ───┘
```

No new object is created.

s1 and s2 point to the same object.

Changing data through one reference affects the other.

```
s2.name = "Java";

System.out.println(s1.name);
```

Output:

Java


#### Creating a Separate Object
```
Student s1 = new Student();
Student s2 = new Student();
```

Two different objects are created.
Changes in one object do not affect the other.

#### Null Reference
```
Student s1 = null;
```

s1 points to nothing.

Accessing s1.name throws __NullPointerException__.

```
🧠 Memory Trick
Class      = Blueprint
Object     = Real thing
Reference  = Address of the real thing
new         = Creates a new object

One object ← Many references ✅
Many references ≠ Many objects ❌
```

# 📘 Constructors

### What is a Constructor?

A constructor is a special method that is automatically called when an object is created using the new keyword. It is mainly used to initialize an object's data.

### Why Do We Need Constructors?

Without a constructor:
```
Student s1 = new Student();

s1.name = "Rahul";
s1.age = 20;
```
You have to assign values manually after creating every object.

With a constructor:
```
Student s1 = new Student("Rahul", 20);
```
The object is initialized immediately when it is created.

### Rules of a Constructor

+ Constructor name must be the same as the class name.
+ It has no return type (not even void).
+ It is called automatically when an object is created.
+ A class can have multiple constructors (Constructor Overloading).

### Types of Constructors

#### 1. Default Constructor

If you don't write any constructor, Java automatically provides one.
```
class Student {

}
```
#### Object creation:
```
Student s = new Student();

Java internally provides:

Student() {

}
```

#### 2. No-Argument Constructor

You write it yourself.
```
class Student {

    Student() {
        System.out.println("Object Created");
    }
}

Output:

Object Created
```

#### 3. Parameterized Constructor

Used to initialize objects with values.
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
Usage:
```
Student s = new Student("Rahul", 20);

Output:

name = Rahul
age = 20
```

### What Happens in Memory?
```
Student s1 = new Student("Rahul", 20);
```

#### Step 1

new allocates memory in the Heap.
```
Student Object

name = null
age = 0
```

#### Step 2

Constructor runs automatically.
```
Student(String name, int age)
```

#### Step 3

Values are assigned.
```
Student Object

name = Rahul
age = 20
```

#### Step 4

Reference is returned.
```
s1 ---------> Student Object
```

### Constructor vs Method

|Constructor	|Method|
|---------------|------|
|Same name as class|	Any valid name|
|No return type	|Must have a return type (or void)|
|Called automatically	|Called manually
|Initializes object	|Performs operations|
|Runs once during object creation	|Can run many times|


### Constructor Overloading

A class can have multiple constructors with different parameters.
```
class Student {

    Student() {

    }

    Student(String name) {

    }

    Student(String name, int age) {

    }
}
```

Java chooses the correct constructor based on the arguments.

### Real-Life Example

Imagine buying a mobile phone.

Without Constructor
```
Buy phone

↓

Insert battery

↓

Set language

↓

Set time

↓

Set owner
```

Many manual steps.

With Constructor
```
Buy phone

↓

Already configured
```

The constructor performs the initial setup automatically.

### Real Project Example

Ticket Class
```
class Ticket {

    String title;
    String status;

    Ticket(String title) {
        this.title = title;
        this.status = "Open";
    }
}
```

Usage:
```
Ticket t = new Ticket("Login Issue");
```
Result:
```
title = Login Issue
status = Open
```
Every new ticket starts with status "Open" automatically.

### Interview Questions

#### Why do we use constructors?

To initialize object data during object creation.

#### Can a constructor return a value?

No. Constructors do not have a return type.

#### Can constructors be overloaded?

Yes. A class can have multiple constructors with different parameter lists.

#### Is a constructor inherited?

No.

#### When is a constructor called?

Automatically when an object is created using the new keyword.

## Constructor use cases

### 1. Assign Initial Values (Most Common)
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
Purpose:

Create an object with initial data.

### 2. Set Default Values

Suppose every new ticket should start as Open.
```
class Ticket {

    String title;
    String status;

    Ticket(String title) {
        this.title = title;
        this.status = "Open";
    }

}
```
Now
```
Ticket t = new Ticket("Login Issue");

Output

title = Login Issue
status = Open
```

Instead of remembering to write

ticket.status = "Open";

everywhere.

### 3. Validate Data

Suppose age cannot be negative.
```
class Student {

    int age;

    Student(int age) {

        if(age < 0){
            throw new IllegalArgumentException("Age cannot be negative");
        }

        this.age = age;

    }

}
```
Now
```
Student s = new Student(-10);
```
Immediately throws an exception.

The object is never created with invalid data.

### 4. Create Required Objects

Suppose a Car always has an Engine.
```
class Engine{

}

class Car{

    Engine engine;

    Car(){

        engine = new Engine();

    }

}
```
Now
```
Car c = new Car();
```
The car automatically gets an engine.

The user doesn't need to remember

car.engine = new Engine();


### 5. Open Resources

Imagine a logger.
```
class Logger{

    FileWriter writer;

    Logger(){

        writer = new FileWriter("app.log");

    }

}
```
As soon as the object is created,

the log file is opened.

Later methods can directly use it.

### 6. Database Connection (Real Projects)

Imagine
```
class DatabaseService{

    Connection connection;

    DatabaseService(){

        connection = DriverManager.getConnection(...);

    }

}
```
Now every method already has a database connection.

### 7. Load Configuration

Imagine your Helyx project.
```
class ConfigService{

    Properties properties;

    ConfigService(){

        properties = loadConfigFile();

    }

}
```
Now every method can use
```
properties.getProperty(...)
```
without loading the file again.

### 8. Dependency Injection

Very common in Spring Boot.
```
class UserService{

    UserRepository repository;

    UserService(UserRepository repository){

        this.repository = repository;

    }

}
```
Spring creates the object by calling the constructor.

### 9. Calculate Initial Values
```
class Rectangle{

    int length;
    int width;
    int area;

    Rectangle(int l,int w){

        length = l;
        width = w;

        area = l * w;

    }

}
```
Area is ready immediately.

