# Prototype & Prototype Chain

```ruby
const parent = {
  greet(){
    console.log("Hello");
  }
};

const child = Object.create(parent);
child.greet(); // Hello
```

#### Question
+ child does not not have a greet
+ then how it works

This is __prototype__

+ child has a internal connection with parent
+ js search child (not found) --> then search parent (found) --> stop

```ruby
const parent = {x : 1};
const child = Object.create(parent);
child.x = 2;
console.log(child.x); // 2
```

because now child has x.

so js searches child (found) --> stop

```ruby
delete child.x;
console.log(child.x); // 1
```


