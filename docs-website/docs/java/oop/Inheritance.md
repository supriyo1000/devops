# Inheritance

### 1. What is inheritance?

Inheritance is an OOP feature in Java that allows one class to acquire the properties and methods of another class.

It helps us reuse code and avoid writing the same code multiple times.

Example:
```
class Account {

    int balance;

    void deposit() {
        System.out.println("Money deposited");
    }
}

class SavingsAccount extends Account {

}
```

Here, SavingsAccount automatically gets the balance variable and deposit() method from Account.

### 2. What does extends mean?

The extends keyword is used to create inheritance in Java.

Syntax:
```
class Child extends Parent {

}

Example:

class Dog extends Animal {

}
```
It means:

Dog inherits from Animal.
Dog can use the methods and variables of Animal.


### 3. What is a parent class?

The class whose properties and methods are inherited by another class is called the parent class (also called superclass or base class).

Example:
```
class Vehicle {

}
```
Here, Vehicle is the parent class.

### 4. What is a child class?

The class that inherits from another class is called the child class (also called subclass or derived class).

Example:
```
class Bike extends Vehicle {

}
```
Here, Bike is the child class.

### 5. What is super()?

super() is used to call the constructor of the parent class.

Example:
```
class Animal {

    Animal() {
        System.out.println("Animal constructor");
    }
}

class Dog extends Animal {

    Dog() {
        super();

        System.out.println("Dog constructor");
    }
}

Output:

Animal constructor
Dog constructor
```

If you do not write __super()__, Java __automatically__ inserts it as the first line.

### 6. What is constructor chaining?

Constructor chaining means calling one constructor from another constructor.

In inheritance, when we create an object of the child class, the __parent__ constructor __executes__ first and then the child constructor.

Example:
```
class A {

    A() {
        System.out.println("A");
    }
}

class B extends A {

    B() {
        System.out.println("B");
    }
}
B obj = new B();

Output:

A
B
```

Flow:
```
Object creation
        ↓
Parent constructor
        ↓
Child constructor
```

### 7. What is an IS-A relationship?

Inheritance represents an IS-A relationship.

Examples:
```
Dog IS-A Animal

Bike IS-A Vehicle

SavingsAccount IS-A Account
```
Incorrect examples:
```
Engine IS-A Car ❌

Wheel IS-A Bike ❌
```
These are __HAS-A__ relationships.

## Step 7: Important Questions

### 1. Why do we use inheritance?

We use inheritance to:

+ Reuse code.
+ Reduce duplicate code.
+ Improve maintainability.
+ Represent real-world relationships.
+ Support polymorphism.

Without inheritance:
```
class SavingsAccount {

    void deposit() {}

    void withdraw() {}
}

class CurrentAccount {

    void deposit() {}

    void withdraw() {}
}
```
The same code is __repeated__.

With inheritance:
```
class Account {

    void deposit() {}

    void withdraw() {}
}

class SavingsAccount extends Account {

}

class CurrentAccount extends Account {

}
```

### 2. Difference between parent and child classes

|Parent Class|	Child Class|
|------------|-------------|
|Base class|	Derived class|
|Gives properties and methods|	Inherits properties and methods|
|Created first|	Depends on parent|
|Cannot access child-specific members|	Can access parent members|

Example:
```
class Vehicle {

}

class Car extends Vehicle {

}

Vehicle → Parent.

Car → Child.
```


### 3. Can a child access private variables?

__No__.

A child class cannot directly access private variables of the parent class.

Example:
```
class Animal {

    private int age;
}

class Dog extends Animal {

    void test() {

        System.out.println(age); // Error
    }
}
```
To __access__ them, use getters and setters:
```
class Animal {

    private int age;

    public int getAge() {
        return age;
    }
}
```

### 4. Difference between this and super

|this|	super|
|----|-------|
|Refers to the current object|	Refers to the parent object|
|Calls current class constructor|	Calls parent constructor|
|Accesses current class variables|	Accesses parent variables|

Example:
```
class Animal {

    String color = "White";
}

class Dog extends Animal {

    String color = "Black";

    void print() {

        System.out.println(this.color);

        System.out.println(super.color);
    }
}

Output:

Black
White
```

### 5. Can Java inherit multiple classes?

__No__.

Java does not support multiple inheritance with classes.

This is not allowed:
```
class A {

}

class B {

}

class C extends A, B {

}
```
Because it creates __ambiguity__ (Diamond Problem).

However, Java supports multiple __inheritance__ using __interfaces__:
```
interface A {

}

interface B {

}

class C implements A, B {

}
```

### Question: Why doesn't Java support multiple inheritance?

Answer:

Java does not support multiple inheritance with classes to avoid ambiguity and the diamond problem, but it supports multiple inheritance using interfaces.

## Method Overriding

### 1. What is Method Overriding?

Method overriding means that the child class provides its own implementation of a method that already exists in the parent class.

Example
```
class Animal {

    void sound() {
        System.out.println("Animal makes sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}
```
Here:

Parent class: __Animal__
Child class: __Dog__

__sound()__ in Dog __overrides__ sound() in Animal.


### 2. Why do we need method overriding?

Suppose every animal makes a different sound.

Without overriding:
```
class Animal {

    void sound() {
        System.out.println("Animal makes sound");
    }
}
```
Every animal would produce the same output.

With overriding:
```
class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {

    @Override
    void sound() {
        System.out.println("Meow");
    }
}
```
Now each class has its own behavior.

### 3. Create objects and test

public class Main {

    public static void main(String[] args) {

        Dog dog = new Dog();

        Cat cat = new Cat();

        dog.sound();

        cat.sound();
    }
}

Output:

Dog barks
Meow
``

### 4. Rules of method overriding

__Rule 1__: Method name must be the same

✅ Correct:
```
class Animal {

    void sound() {

    }
}

class Dog extends Animal {

    void sound() {

    }
}
```
❌ Wrong:
```
class Dog extends Animal {

    void bark() {

    }
}
```

__Rule 2__: Parameters must be the same

✅ Correct:
```
void eat(int amount)

❌ Wrong:

void eat()
```

__Rule 3__: Return type must be compatible

✅ Correct:
```
class Animal {

    String getName() {
        return "Animal";
    }
}

class Dog extends Animal {

    String getName() {
        return "Dog";
    }
}
```

__Rule 4__: Cannot override private methods

```
class Animal {

    private void sleep() {

    }
}
```
Dog cannot override sleep() because private methods are not inherited.

__Rule 5__: Cannot override final methods
```
class Animal {

    final void walk() {

    }
}

This gives a compile-time error:

class Dog extends Animal {

    void walk() {

    }
}
```

### 5. What does @Override do?
```
@Override
void sound() {
    System.out.println("Bark");
}
```
@Override tells Java:

"I am overriding a parent method."

It helps the compiler catch mistakes.

Example:
```
@Override
void sounds() {

}
```
Error:

Method does not __override__ method from __superclass__

Without __@Override__, Java would not warn you.

## 6. questions

### Why do we use method overriding?

+ To achieve runtime polymorphism.
+ To provide specific behavior in child classes.
+ To customize parent methods.


### What is the difference between overloading and overriding?

|Overloading|	Overriding|
|-----------|-------------|
|Same class|	Parent-child class|
|Same method name|	Same method name|
|Different parameters|	Same parameters|
|Compile time|	Runtime|


### Why should we use @Override?

Because it helps the compiler detect errors and confirms that the method is actually overriding a parent method.


## Upcasting

### 1. What is upcasting?

Upcasting means storing a child object inside a parent reference variable.

Syntax:
```
Parent p = new Child();
```
Example:
```
class Animal {

    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {

    void bark() {
        System.out.println("Dog is barking");
    }
}
```
Now create the object:
```
Dog dog = new Dog();
```

But Java also allows:
```
Animal animal = new Dog();
```
This is called upcasting.

### 2. Why is Java allowing this?

Because:

Dog __IS-A__ Animal

A dog is an animal, so Java allows us to store a Dog object in an Animal variable.

But:
```
Dog dog = new Animal();
```
❌ Not allowed.

Because:

Animal __IS NOT__ A Dog


### 3. What actually happens in memory?

When you write:
```
Animal animal = new Dog();
```
Java creates:
```
Reference Variable      Object

animal  ------------->   Dog object
(Type Animal)            (Type Dog)
```
Two things exist:

Reference type → __Animal__
Actual object type → __Dog__

This difference is extremely important.

### 4. What methods can you access?

Example:
```
class Animal {

    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {

    void bark() {
        System.out.println("Bark");
    }
}

Now:

Animal animal = new Dog();

animal.eat();
```
✅ Allowed.

Because eat() __exists__ in Animal.

But:
```
animal.bark();

❌ Error.
```
Compiler says:

Cannot find symbol bark()

#### Why?

Because the reference type is __Animal__, and Animal does __not have__ bark().

### 5. Important rule

Java checks methods in two stages.

#### Compile time

Java checks:

"What methods does the reference type have?"

Reference type:

Animal animal

Java only sees:

eat()


#### Runtime

Java checks:

"What object was actually created?"

Object:

new Dog()

This is where method overriding becomes important.

### 6. Example with overriding

```
class Animal {

    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```
Now:
```
Animal animal = new Dog();

animal.sound();
```

Output:

__Bark__

Because Java looks at:

Compile time → Is sound() present in Animal? ✅
Runtime → Which object is created? Dog ✅

So Java calls:

Dog.sound()

This behavior is called __dynamic method dispatch__.

### 7. Why do we need upcasting?

Imagine:

Without upcasting:
```
Dog dog = new Dog();

Cat cat = new Cat();

Cow cow = new Cow();

You need separate code for everything.

With upcasting:

Animal[] animals = {
    new Dog(),
    new Cat(),
    new Cow()
};
```
Now you can process all animals together.


## 8. questions

### What is upcasting?

Upcasting is the process of assigning a child object to a parent reference variable.

Example:
```
Animal animal = new Dog();
```

### Is upcasting explicit or implicit?

Upcasting is automatic (implicit).

Animal animal = new Dog();

No cast is needed.

### Which methods are accessible after upcasting?

Only methods available in the parent class reference are accessible.

### Does upcasting create a new object?

No.

Animal animal = new Dog();

creates only one Dog object.

## Dynamic Method Dispatch (Runtime Polymorphism)

#### 1. First, look at this code
```
class Animal {

    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {

    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}

Now:

Animal a1 = new Dog();

Animal a2 = new Cat();

a1.sound();

a2.sound();

Output:

Dog barks

Cat meows
```

#### 2. But wait...

The reference types are:

Animal a1
Animal a2

So why doesn't Java call:

Animal.sound() ?

Because Java __decides__ which overridden method to execute at runtime.

This process is called __dynamic method dispatch__.

### 3. Definition

Dynamic method dispatch is the mechanism by which Java determines, at runtime, which overridden method should be called based on the actual object type.

### 4. Compile time vs Runtime

Consider:

Animal animal = new Dog();

animal.sound();

#### Compile time

Java checks:

Does Animal have sound()?

Answer:

Yes

Compilation succeeds.

#### Runtime

Java checks:

Which object was created?

Answer:

Dog

So Java executes:

Dog.sound();


### 5. Memory diagram
```
Reference Variable              Object

Animal animal -----------> Dog object
```
Important:

Reference type = Animal
Actual object type = Dog

Java chooses the method using the actual object type, not the reference type.

### 6. Why is this useful?

Without dynamic method dispatch:

Dog dog = new Dog();
Cat cat = new Cat();

dog.sound();
cat.sound();

You must handle every type separately.

With dynamic dispatch:
```
Animal[] animals = {
    new Dog(),
    new Cat(),
    new Cow()
};

for (Animal a : animals) {
    a.sound();
}

Output:

Dog barks
Cat meows
Cow moos
```
One loop works for every animal.

This is why frameworks and large applications depend on polymorphism.

### 7. Dynamic dispatch happens only for overridden methods

Example:
```
class Animal {

    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {

    void bark() {
        System.out.println("Bark");
    }
}
Animal animal = new Dog();

animal.eat();   // OK

animal.bark();  // Error
```
Why?

Because:

At compile time Java only sees:

Animal animal

and Animal does not contain:

bark()


### 8. Interview definition

Dynamic method dispatch is the process by which Java resolves calls to overridden methods at runtime based on the actual object type rather than the reference type.

## 9. Very important question


### What is the difference between compile-time polymorphism and runtime polymorphism?

|Compile Time|	Runtime|
|------------|---------|
|Method overloading|	Method overriding|
|Decision at compile time|	Decision at runtime|
|Faster|	Slightly slower|
|Same class|	Parent-child class|


## instanceof

### 1. What is instanceof?

The instanceof operator checks whether an object belongs to a particular class or not.

Syntax:
```
object instanceof ClassName

It returns:

true
false
```

### 2. Simple example
```
class Animal {

}

class Dog extends Animal {

}

public class Main {

    public static void main(String[] args) {

        Dog dog = new Dog();

        System.out.println(dog instanceof Dog);

        System.out.println(dog instanceof Animal);
    }
}
```

Output:

true

true

Why?

Because:

Dog __IS-A__ Dog

Dog __IS-A__ Animal


### 3. Example with upcasting
```
class Animal {

}

class Dog extends Animal {

}

Animal animal = new Dog();

System.out.println(animal instanceof Dog);

System.out.println(animal instanceof Animal);
```

Output:

true

true

Even though the reference type is:

__Animal animal__

the actual object is:

__new Dog()__

and instanceof checks the __actual__ object type.

### 4. Example with multiple child classes
```
class Animal {

}

class Dog extends Animal {

}

class Cat extends Animal {

}

Animal animal = new Dog();

System.out.println(animal instanceof Dog);

System.out.println(animal instanceof Cat);

System.out.println(animal instanceof Animal);
```

Output:

true

false

true


### 5. Why do we use instanceof?

Suppose:

Animal animal = new Dog();

You want to call:

bark();

But:

animal.bark();

#### ❌ Compile-time error.

You can first check:
```
if (animal instanceof Dog) {

    Dog dog = (Dog) animal;

    dog.bark();
}
```

#### What is happening here:

This line:

Dog dog = (Dog) animal;

is called __downcasting__.

#### Upcasting:

Animal animal = new Dog();

#### Downcasting:

Dog dog = (Dog) animal;

Always check with __instanceof__ before downcasting.

### 7. What happens if you don't check?

Animal animal = new Cat();

Dog dog = (Dog) animal;

Runtime error:

#### ClassCastException

Because:

Cat __IS NOT__ A Dog


## 8. questions

### What is instanceof?

instanceof is an operator that checks whether an object belongs to a particular class or interface.

### What is downcasting?

Converting a parent reference back to a child reference.

Example:
```
Animal animal = new Dog();

Dog dog = (Dog) animal;
```

### Why do we use instanceof before downcasting?

To avoid __ClassCastException__.

### 9. Real-world analogy

Imagine:

Animal animal = new Dog();

You know the object is "some animal", but you don't know which animal.

instanceof is like asking:

"Are you a Dog?"

If yes:

Use Dog features.

## final

### 1. What is final?

The final keyword means:

"This cannot be changed."

You can use final with:

+ Variables
+ Methods
+ Classes

### 2. final method

A final method cannot be overridden by a child class.

Example:
```
class Animal {

    final void sleep() {
        System.out.println("Animal is sleeping");
    }
}

class Dog extends Animal {

    @Override
    void sleep() {    // Error
        System.out.println("Dog is sleeping");
    }
}
```
Java gives an error because sleep() is final.


### 3. Why use a final method?

Suppose your bank application has a method:
```
class Account {

    final void calculateTax() {
        System.out.println("Tax calculated");
    }
}
```
The bank does not want any child class to change the tax calculation logic.

So:
```
class SavingsAccount extends Account {

}
```
can use the method but cannot override it.

### 4. final class

A final class cannot be inherited.

Example:
```
final class Vehicle {

}

class Bike extends Vehicle {

}
```
❌ Compile-time error.

Because:

Cannot inherit from __final__ Vehicle


### 5. Real-world example

Java's __String__ class is final.

String name = "Supriyo";

You cannot do:
```
class MyString extends String {

}
```
because String is final.

Why?

+ Security
+ Immutability
+ Prevent accidental modifications


### 6. final variable (quick overview)
```
final int AGE = 25;

AGE = 30;
```

❌ Error.

Once assigned, a final variable cannot be __changed__.

By convention, constants are written in uppercase:

final double PI = 3.14;


## 7. questions


### What is final?

The final keyword restricts modification.

+ Final variable → Cannot change value.
+ Final method → Cannot override.
+ Final class → Cannot inherit.

### Can we override a final method?

No.

### Can we inherit a final class?

No.

### Why is the String class final?

To provide __security__ and __immutability__.

```
What should you learn next?

This is the best order:

Encapsulation (quick revision)
Polymorphism
Abstraction
Interface
abstract class vs interface
Composition ("HAS-A" relationship)
```