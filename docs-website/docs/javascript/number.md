# Number methods

## 1. What is a Number in JavaScript?
javascript has only one numeric type that is number.

no int , float , double separately

#### internaly js uses

**64 bit floating point**

Thats why 
```
0.1 + 0.2 = 0.30000000000000004
```

## 2. Number Properties

### Number.MAX_VALUE

largest possible number :

```ruby
console.log(Number.MAX_VALUE); // 1.7976931348623157e+308
```

### Number.MIN_VALUE

Smallest positive number.

```ruby
console.log(Number.MIN_VALUE); // 5e-324
```

### Number.POSITIVE_INFINITY and NEGATIVE_INFINITY

```ruby
console.log(1/0); // Infinity
console.log(-1/0); // -Infinity
```

### Number.NaN

means not a number

```ruby
let x = "hello" * 5;
console.log(x); // NaN
```

#### important interview point

```
console.log(NaN === NaN); // false
```

#### Correct way

```ruby
let x = "hello" * 5;
console.log(Number.isNaN(x)); // true
```

## 3. Converting to Number ⭐ VERY IMPORTANT

### Number()

```ruby
console.log(Number("12")); // 12
console.log(Number("12.5")); // 12.5
console.log(Number("abc")); // NaN
```

### parseInt()

Extracts integer

```ruby
console.log(parseInt("123px")); // 123
console.log(parseInt("10.9")); // 10
```

### parseFloat()

Extracts decimal

```ruby
console.log(parseFloat("11.75kg")); // 11.75
console.log(parseFloat("12")); // 12
```

## 4. Checking Numbers

### isNaN()
```
console.log(isNaN("abc")); // true
```

### Number.isInteger()
```
console.log(Number.isInteger("12.9")); // false
console.log(Number.isInteger("abc")); // false
```

### Number.isFinite()

```
console.log(Number.isFinite(10)); // true
console.log(Number.isFinite(Infinity)); // false
```

## 5. Number Methods

### toFixed(n) ⭐

control decimal places

```ruby
console.log(123.9876544.toFixed(3)); // 123.988
console.log(23.018345.toFixed(3)); // 23.018
```

### toPrecision(n)

controls total number count

```ruby
let num = 123.456;
console.log(num.toPrecision(4)); // 123.4
console.log(num.toPrecision(3)); // 123
```

### toString()

convert number to string

```ruby
let num = 123;
console.log(num.toString()); // "123"
console.log(typeof(num.toString())); // string
```

## 6. Math Object (MOST IMPORTANT PART)

### Math.round()

```ruby
console.log(Math.round(4.6)); // 5
console.log(Math.round(4.4)); // 4
```

### Math.floor()

Always down

```ruby
console.log(Math.floor(4.6)); // 4
```

### Math.ceil()

Always up

```ruby
console.log(Math.ceil(4.6)); // 5
console.log(Math.ceil(4.1)); // 5
```

### Math.trunc()

Removes decimal

```ruby
console.log(Math.trunc(5.6)); // 5
console.log(Math.trunc(4.1)); // 4
```

### Math.abs()

Absolute value

```ruby
console.log(Math.abs(-10));
console.log(Math.abs(-3.1));
```

### Math.max() / Math.min()

```ruby
console.log(Math.max(1,2,3)); // 3
console.log(Math.min(2,5,1)); // 1
```

#### Array case 

```ruby
let num1 = [1,2,5];
let num2 = [5,4,2];
console.log(Math.max(...num1)); // 5
console.log(Math.min(...num2)); // 2
```

### Math.random() ⭐⭐⭐

Random number 0–1.

```ruby
console.log(Math.random());// 0.688765433356
```

#### for Dice

```ruby
console.log(Math.floor((Math.random() * 6) + 1)); // 4
```

### Math.pow()
To get base ^ exponent

```ruby
Math.pow(2,3); // 8
```

```ruby
Math.pow(3 , 3) // 27
```

### Math.sqrt()

To get square root

```ruby
console.log(Math.sqrt(25)); // 5
console.log(Math.sqrt(20)); // 4.472345
```
