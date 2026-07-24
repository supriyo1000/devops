# What is Polymorphism?

Definition

The word Polymorphism comes from two Greek words:
```
Poly = Many
Morph = Forms
```
Meaning:

One interface/reference, many different forms (behaviors).

### Real-life Example

Suppose you have a remote control.
```
Remote
   ↓

TV
AC
Speaker
Projector
```
You press the Power button.

Different devices perform different actions.
```
TV        → Turns ON

AC        → Starts cooling

Speaker   → Starts playing

Projector → Starts projection

Same button.

Different behavior.
```
That is polymorphism.

Java Example

Instead of remote, let's use your bank project.
```
Account
      ↑
SavingsAccount

Account
      ↑
CurrentAccount

Account
      ↑
SalaryAccount

Now:

Account account =
        new SavingsAccount(...);

Later:

account.getAccountInfo();

If tomorrow:

Account account =
        new CurrentAccount(...);

The same statement:

account.getAccountInfo();

prints completely different information.

One variable.

Many behaviors.

That is polymorphism.
```

### What problem does polymorphism solve?

#### Imagine there were no polymorphism.

You would write:
```
SavingsAccount s =
        new SavingsAccount(...);

CurrentAccount c =
        new CurrentAccount(...);

SalaryAccount sa =
        new SalaryAccount(...);

s.getAccountInfo();

c.getAccountInfo();

sa.getAccountInfo();
```
Different variables.

Different code.

Now imagine you have:

__50__ account types

Nightmare.

#### With polymorphism:
```
Account account;

account = new SavingsAccount(...);

account.getAccountInfo();

account = new CurrentAccount(...);

account.getAccountInfo();

account = new SalaryAccount(...);

account.getAccountInfo();
```
One variable.

Same method.

Different output.

#### This is the MOST IMPORTANT sentence

Polymorphism does NOT mean:

One object becoming many objects.

It means:
```
One parent reference can refer to different child objects.
```
Example:
```
Account account;

account = new SavingsAccount();

account = new CurrentAccount();

account = new SalaryAccount();
```
The variable type never changed.

Only the object changed.

## Types of Polymorphism

There are only two.

### 1. Compile-time polymorphism

Also called

+ Static polymorphism

+ Method Overloading

Example:
```
deposit(100);

deposit(100, "Cash");

deposit(100, "UPI");
```
Java decides during compilation which method to call.

### 2. Runtime polymorphism

Also called

+ Dynamic polymorphism

+ Method Overriding

Example:
```
Account account =
        new SavingsAccount();

account.getAccountInfo();
```
Java decides during execution which method to call.



## Inheritance vs Polymorphism

Many beginners think they are the same.

They are not.

Inheritance:

Account
      ↑
SavingsAccount

defines the relationship.

Polymorphism:

Account account =
        new SavingsAccount();

uses that relationship.

Inheritance creates the family.

Polymorphism lets you treat the family members through one common type.

### Definition

Polymorphism is the ability of one reference type to represent different object types and execute different behaviors depending on the actual object at runtime.

Practice using your project

You already have:
```
Account a1 =
        BankService.createAccount(c1, "Savings");

Account a2 =
        BankService.createAccount(c2, "Current");

Account a3 =
        BankService.createAccount(c3, "Salary");

Now create:

Account[] accounts = {
        a1,
        a2,
        a3
};

Loop:

for (Account account : accounts) {
    account.getAccountInfo();
}
```
Notice:

You never wrote:
```
if (account instanceof SavingsAccount)

if (account instanceof CurrentAccount)

if (account instanceof SalaryAccount)
```
Yet Java automatically prints the correct information for each account.

That is __runtime__ polymorphism in action.

## Questions

### What is polymorphism?

The ability of one parent reference to refer to different child objects and execute different behaviors.

### How many types of polymorphism are there?

Two:

+ Compile-time polymorphism (Method Overloading)

+ Runtime polymorphism (Method Overriding)


### Does polymorphism require inheritance?

+ Compile-time polymorphism (overloading): No.

You can overload methods in the same class.

+ Runtime polymorphism (overriding): Yes.

It requires inheritance because a child class overrides a parent method.

## Compile-Time Polymorphism (Method Overloading)

### 1. What is Method Overloading?

Method overloading means:

Multiple methods in the same class have the same name and return type but different parameter lists.

Example:
```
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```
All methods are named add(), but Java can tell them apart because their parameters differ.

### 2. Why do we need overloading?

Imagine a bank application.

#### Without overloading:

```
depositCash()

depositCheque()

depositUPI()

depositCard()
```
Many method names.

#### With overloading:

```
deposit(1000);

deposit(1000, "UPI");

deposit(1000, "Cheque");

deposit(1000, "Card");
```

Same method name.

Different parameters.

This makes APIs easier to use.

### 3. How does Java decide?

Suppose:
```
deposit(1000);

Java looks at the arguments:

One int

It chooses:

deposit(int amount)

Now:

deposit(1000, "UPI");

Java sees:

int
String

So it chooses:

deposit(int amount, String mode)
```

This decision happens at compile time, so it's called compile-time polymorphism.

### 4. Rules of Method Overloading

#### Rule 1: Parameter count can change
deposit(int amount)

deposit(int amount, String mode)

✅ Valid.

#### Rule 2: Parameter types can change

deposit(int amount)

deposit(double amount)

✅ Valid.

#### Rule 3: Parameter order can change

transfer(String from, String to)

transfer(String to, int amount)

As long as the parameter types/order make the signature different.

#### 5. What is NOT overloading?

Only changing the return type
```
int add(int a, int b)

double add(int a, int b)
```

❌ Compile-time error.

Why?

Because Java decides which method to call before looking at the return value.

### 7. Overloading vs Overriding

|Method Overloading|	Method Overriding|
|------------------|---------------------|
|Same class|	Parent-child classes|
Same method name|	Same method name|
Different parameters|	Same parameters|
Compile-time|	Runtime|
Inheritance not required|	Inheritance required|