# Abstraction in Java (Simple Notes)

### What is Abstraction?

#### Definition:

Abstraction is the process of hiding the implementation details and showing only the essential behavior.

In simple words:

Tell "what" to do, not "how" to do it.

### Real-life Example

Think of an ATM.
```
You insert the card.

You enter the PIN.

You withdraw money.
```
Do you know:

How the bank verifies the PIN?
How the database is updated?
How the cash machine communicates with the server?

No.

You only know:

Withdraw Money

The internal implementation is hidden.

This is abstraction.

### Bank Project Example

Suppose you have:
```
Account
    ↑

SavingsAccount
CurrentAccount
SalaryAccount
```
Every account should calculate interest.

But can the parent Account class calculate it?
```
class Account {

    void calculateInterest() {

    }

}

Question:

How much interest?

Savings → 5%

Current → 0%

Salary → 2%

The parent doesn't know.
```
So instead of writing incorrect logic:

void calculateInterest(){

}

we write:
```
abstract void calculateInterest();
```
Meaning:

Every account must calculate interest,

but I don't know how.

Each child will decide.

### What is an Abstract Class?

An abstract class is an incomplete class.

It provides a __blueprint__.

Example:
```
public abstract class Account {

}

You cannot create its object.

Account a = new Account();
```
❌ Compile-time Error

Why?

Because an account is only a concept.

The real object is:
```
SavingsAccount

CurrentAccount

SalaryAccount
```

### What is an Abstract Method?

An abstract method has:

+ No body
+ No implementation

Example:
```
public abstract void calculateInterest();
```

Notice:

No {}

No code.

It only says:

Every child must implement this.

#### Child Class

```
class SavingsAccount extends Account {

    @Override
    public void calculateInterest() {

        System.out.println("5% Interest");

    }

}
```
Another child:
```
class CurrentAccount extends Account {

    @Override
    public void calculateInterest() {

        System.out.println("No Interest");

    }

}
```
Each child decides its own implementation.

### Why do we need abstraction?

Without abstraction:
```
class Account {

    void calculateInterest() {

        // What should I write?

    }

}
```
There is no common implementation.

#### With abstraction:
```
abstract class Account {

    abstract void calculateInterest();

}
```
Now every child is forced to implement it correctly.

### Difference between Normal Method and Abstract Method

#### Normal method:
```
public void deposit(){

    balance += amount;

}
```
Already implemented.

Child classes can use it directly.

#### Abstract method:

public abstract void calculateInterest();

No implementation.

Every child must write its own version.

### Rules

#### Rule 1

Abstract class cannot be instantiated.

Account a = new Account();

❌

#### Rule 2

Child class must implement every abstract method.

Otherwise:

Compile-time Error

#### Rule 3

Abstract class can have:

+ Variables
+ Constructors
+ Normal methods
+ Abstract methods

Example:
```
abstract class Account {

    String accountNumber;

    Account(){

    }

    void deposit(){

    }

    abstract void calculateInterest();

}
```
Perfectly valid.

### Definition

Abstraction is the process of hiding implementation details and exposing only the essential behavior. In Java, abstraction is achieved using abstract classes and interfaces.

### Easy Trick to Remember

Whenever you write a parent class, ask yourself:

"Can the parent really implement this method?"

If the answer is No, make it abstract.

Example:
```
deposit()                ✅ Parent knows how.

getAccountInfo()         ✅ Parent knows how.

calculateInterest()      ❌ Parent doesn't know.

withdraw()               ❓ Depends on business rules.
```

### One-Line Summary
```
Inheritance says: "IS-A relationship."

Polymorphism says: "One parent reference, many child behaviors."

Abstraction says: "The parent defines what must be done, and the child decides how to do it."
```

# Interface in Java (Easy Notes)

### First, why do we need interfaces?

Let's use your bank project again.

Suppose your bank has different payment methods.
```
UPI
Credit Card
Net Banking
Wallet
```
Every payment method can make a payment.

But how they make the payment is different.
```
UPI:

Validate UPI ID
Send payment
```
```
Card:

Validate card
Check CVV
Send payment
```
```
Wallet:

Check wallet balance
Deduct amount
```
Although the implementation differs, they all have one __common__ behavior:
```
pay()
```
Instead of making them inherit from one another (which doesn't make sense), we define a contract.

### What is an Interface?

An interface is a contract.

It says:

"If you implement me, you must provide these methods."

It doesn't say how.


### Java Syntax
```
public interface Payment {

    void pay(double amount);

}
```
Notice:

No class
No extends

Just a method declaration

This defines a contract.

### Implementing an Interface

```
public class UpiPayment implements Payment {

    @Override
    public void pay(double amount) {

        System.out.println("Paid using UPI");

    }

}
```

Another class:
```
public class CardPayment implements Payment {

    @Override
    public void pay(double amount) {

        System.out.println("Paid using Card");

    }

}
```
Another:
```
public class WalletPayment implements Payment {

    @Override
    public void pay(double amount) {

        System.out.println("Paid using Wallet");

    }

}
```
Each class implements the contract in its own way.

### Using Polymorphism with Interfaces

```
Payment payment = new UpiPayment();

payment.pay(1000);
```
Later:
```
payment = new CardPayment();

payment.pay(1000);
```
Same reference.

Different behavior.

Exactly like runtime polymorphism.

### Why not use an abstract class?

Good question.

Suppose you already have:
```
class Account {

}

Now you create:

class SavingsAccount extends Account {

}
```
Later, you also want SavingsAccount to support online payments.

If Payment were another class:
```
class Payment {

}

You might want:

class SavingsAccount extends Account, Payment
```

❌ Java doesn't allow a class to extend two classes.

This is where interfaces help.

class SavingsAccount extends Account implements Payment

✅ Valid.

A class can extend one class but implement multiple interfaces.

### Multiple Interfaces

```
interface Payment {

    void pay();

}
interface Printable {

    void print();

}
class SavingsAccount
        extends Account
        implements Payment, Printable {

    @Override
    public void pay() {

    }

    @Override
    public void print() {

    }

}
```
This is Java's way of achieving multiple inheritance of behavior.

### extends vs implements

#### For classes:

class Dog extends Animal

#### For interfaces:

class Dog implements Animal


|Abstract Class	| Interface|
|---------------|----------|
|Represents a base class with shared implementation|	Represents a contract|
|extends|	implements|
|Can have constructors|	No constructors|
|Can have instance variables|	No instance state (constants only)|
|Can have implemented methods|	Can declare methods and also have default/static methods (Java 8+)|
|Single inheritance|	Multiple interfaces can be implemented|


### Real Spring Boot Example

You've probably seen this:
```
public interface UserRepository {

}
```
Then:
```
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```
You don't implement methods like:
```
save()

findById()

delete()

findAll()
```
Spring Data JPA provides the implementation.

You only agree to the contract by extending the interface.

That's why interfaces are everywhere in Spring Boot.

### Bank Project Example

Imagine this interface:
```
public interface Payment {

    void pay(double amount);

}

Then:

class UpiPayment implements Payment
class CardPayment implements Payment
class WalletPayment implements Payment
```
Your payment service could use:
```
Payment payment;

payment = new UpiPayment();
payment.pay(1000);

payment = new CardPayment();
payment.pay(1000);
```

The service doesn't care which payment type it receives. It only knows that every Payment can pay().

## Questions

### What is an interface?

An interface is a contract that defines methods a class must implement.

### Can we create an object of an interface?

No.

Payment p = new Payment();

❌ Not allowed.

### Can a class implement multiple interfaces?

Yes.

class A implements X, Y, Z


### Why do we use interfaces?

To define a common contract.
To achieve loose coupling.
To support polymorphism.
To allow multiple inheritance of behavior.

```
Easy Way to Remember
Class
-----
Can do the work.

Abstract Class
--------------
Knows some work.
Leaves some work for children.

Interface
----------
Knows no work.
Only defines the contract.
```