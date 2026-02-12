# Array Loops

## for loop

a for loop repeats code a fixed number of times.

Syntax : 

```ruby
for( initialization; condition; update ){
    // code
}
```

## Reverse Loop (Very Important)

Example :

```ruby
let array = [1, 2, 3];

for( let i = array.length - 1; i >= 0; i-- ){
    console.log(array[i]);
}

// 3
// 2
// 1
```

### using const

```ruby
for( const i = arr.length - 1; i >= 0; i-- ){
    console.log(arr[i]);
}

// Error : Assignment to constant variable.
```

## while Loop — The Condition Loop

Runs until a condition becomes false.

Syntax:

```ruby
while(condition){
    // code
}
```

Example 1 :

```ruby
let arr = [0 ,1, 2, 3, 4];

let sum = 0;
let i = 0;

while (i < arr.length) {
    sum+= arr[i];
    i++;
}

console.log(sum); // 10
```

Example 2 :

```ruby
let arr = [1, 2, 3, 4, 6, 8, 9, 11];

function hasPair(arr, target) {

    let left = 0;
    let right = arr.length - 1;

    while (left < right) {

        let sum = arr[left] + arr[right];

        if (sum === target) {
            return [arr[left], arr[right]]
        }

        else if (sum < target) {
            left++;
        }

        else {
            right--;
        }
    }

    return null;
}

console.log(hasPair(arr , 9));
```

## do...while loop

do...while loops runs the code atleast once then check the condition.

```
do {
    // runs the code first
} while (condition)
```

| while | dowhile |
|-------|---------|
| first check the condition then run the code. | run the code first then check the condition. |

Example 1 :

```ruby
let correctotp = 123;
let userotp;

do {
    userotp = prompt("enter otp");
} while (correctotp !== userotp);

console.log("login successfully!!");
```

Example 2 :

```ruby
let choice;

do {
    console.log("1. withdraw");
    console.log("2. balance");
    console.log("3. Exit");
    
    choice = Number(prompt("Enter choice.."));
    
} while( choice !== 3 );

console.log("Thanks for using ATM");
```

Example 3 :

```ruby
function connectserver(){
    let attempt = 0;
    let success = false;

    do {
        attempt++;
        console.log("Trying to connect server", attempt);
        let x = Math.random();
        console.log(x);
        
        success = x > 1;

        if(success){
            console.log("server connected");
        } else {
            console.log("server not connected");
        }
        
    } while (!success && attempt < 3);
}

connectserver();
```