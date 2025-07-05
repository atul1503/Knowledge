# Rust Programming Language

## Introduction

Rust is a systems programming language that focuses on safety, speed, and concurrency. Rust prevents common programming errors like null pointer dereferences and buffer overflows by using a unique ownership system. This document explains Rust concepts with detailed examples.

## Installation

To install Rust, use the official installer:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

The `curl` command downloads and runs the Rust installer script. The `--proto` flag ensures HTTPS protocol is used. The `--tlsv1.2` flag specifies the TLS version for security. The `-sSf` flags make curl silent, show errors, and fail on server errors.

After installation, restart your terminal and verify:

```bash
rustc --version
```

The `rustc` command is the Rust compiler. The `--version` flag shows the installed version.

## Basic Syntax

### Hello World

```rust
fn main() {
    println!("Hello, world!");
}
```

- `fn` keyword declares a function
- `main` is the function name - this is the entry point of every Rust program
- `()` are parentheses for function parameters (empty here)
- `{}` are curly braces that contain the function body
- `println!` is a macro (indicated by `!`) that prints text to the console
- `"Hello, world!"` is a string literal enclosed in double quotes
- `;` semicolon ends the statement

### When to Use Curly Braces `{}`

Curly braces `{}` are used in many different contexts in Rust. Here are all the main cases:

#### 1. Function Bodies
```rust
fn my_function() {
    println!("This is inside a function body");
}
```
- `{}` contain all the code that runs when the function is called
- Everything between the braces is the function's implementation

#### 2. Struct Definitions
```rust
struct Person {
    name: String,
    age: u32,
}
```
- `{}` define the fields that belong to the struct
- Each field is listed with its name and type

#### 3. Struct Instantiation
```rust
let person = Person {
    name: String::from("Alice"),
    age: 25,
};
```
- `{}` create a new instance of the struct
- You provide values for each field inside the braces

#### 4. Enum Definitions
```rust
enum Color {
    Red,
    Green,
    Blue,
}
```
- `{}` contain all the possible variants of the enum
- Each variant is a possible value the enum can have

#### 5. Control Flow Blocks
```rust
if condition {
    println!("True case");
} else {
    println!("False case");
}

loop {
    println!("This loops forever");
    break;
}

while counter < 10 {
    counter += 1;
}

for item in collection {
    println!("Item: {}", item);
}
```
- `{}` contain the code that runs for each control flow statement
- Each block defines what happens in that case

#### 6. Match Arms
```rust
match value {
    1 => println!("One"),
    2 => println!("Two"),
    _ => println!("Other"),
}
```
- `{}` contain all the possible patterns and their actions
- Each arm specifies what to do for different values

#### 7. Module Definitions
```rust
mod my_module {
    pub fn public_function() {
        println!("This is public");
    }
}
```
- `{}` contain all the code that belongs to the module
- Functions, structs, and other items go inside

#### 8. Implementation Blocks
```rust
impl Person {
    fn new(name: String, age: u32) -> Person {
        Person { name, age }
    }
}
```
- `{}` contain methods and associated functions for a type
- All the behavior for the type is defined inside

#### 9. Trait Definitions
```rust
trait Printable {
    fn print(&self);
}
```
- `{}` contain the method signatures that types must implement
- May also contain default implementations

#### 10. Macro Definitions
```rust
macro_rules! my_macro {
    () => {
        println!("Hello from macro!");
    };
    ($name:expr) => {
        println!("Hello, {}!", $name);
    };
}
```
- The outer `{}` contain all the macro rules (the entire macro definition)
- Inside, you see pattern matching arms that look like functions but aren't
- `() => { }` is a macro arm where:
  - `()` is the pattern (what input the macro matches)
  - `=>` is the arrow that separates pattern from expansion
  - `{}` after the arrow contains the code that gets generated
- `($name:expr) => { }` is another arm that:
  - `($name:expr)` matches an expression and captures it as `$name`
  - `{}` contains the code that uses the captured `$name`
- Each arm is like a different "version" of the macro for different inputs
- The `{}` after `=>` contain the actual Rust code that replaces the macro call

Here's how this macro works when used:
```rust
fn main() {
    my_macro!();           // Uses the first arm () => { }
    my_macro!("World");    // Uses the second arm ($name:expr) => { }
}
```
- `my_macro!()` matches the `()` pattern and expands to `println!("Hello from macro!");`
- `my_macro!("World")` matches `($name:expr)` pattern, captures `"World"` as `$name`, and expands to `println!("Hello, {}!", "World");`
- The function-like syntax is actually pattern matching, not function definitions

#### Different Types of Macro Arms and Matching Patterns

Here's a comprehensive example showing various types of macro matching patterns:

```rust
macro_rules! advanced_macro {
    // 1. Literal matching - matches exact text
    (hello) => {
        println!("You said hello!");
    };
    
    // 2. Expression matching - matches any expression
    (print $expr:expr) => {
        println!("Expression value: {}", $expr);
    };
    
    // 3. Identifier matching - matches variable or function names
    // "create_var" is not a special keyword in Rust. It is just a custom pattern name for this macro arm.
    // You can use any name you want here; it is only used to match the macro call.
    (create_var $name:ident) => {
        // $name:ident means this pattern matches an identifier (like a variable or function name)
        let $name = 42;
        println!("Created variable {} with value {}", stringify!($name), $name);
    };
    
    // 4. Type matching - matches type names
    (create_vec $type:ty) => {
        let mut vec: Vec<$type> = Vec::new();
        println!("Created vector of type {}", stringify!($type));
    };
    
    // 5. Multiple parameters - matches several things
    (add $a:expr, $b:expr) => {
        println!("{} + {} = {}", $a, $b, $a + $b);
    };
    
    // 6. Repetition patterns - matches multiple items
    (sum $($num:expr),*) => {
        {
            let mut total = 0;
            $(
                total += $num;
            )*
            println!("Sum of all numbers: {}", total);
            total
        }
    };
    
    // 7. Optional patterns - matches with or without something
    (greet $name:expr $(, $title:expr)?) => {
        match stringify!($($title)?).is_empty() {
            true => println!("Hello, {}!", $name),
            false => println!("Hello, {} {}!", $($title,)? $name),
        }
    };
    
    // 8. Block matching - matches code blocks
    (if_debug $code:block) => {
        #[cfg(debug_assertions)]
        $code
    };
}
```

**Explanation of each pattern type:**

1. **Literal matching `(hello)`**: 
   - Matches exactly the word "hello"
   - No variables captured, just exact text matching

2. **Expression matching `$expr:expr`**:
   - `$expr` is the variable name to capture into
   - `:expr` means it matches any valid Rust expression
   - Can be numbers, variables, function calls, etc.

3. **Identifier matching `$name:ident`**:
   - `:ident` matches identifiers (variable names, function names)
   - `stringify!` converts the identifier to a string

4. **Type matching `$type:ty`**:
   - `:ty` matches type names like `i32`, `String`, `Vec<i32>`
   - Used when you want to work with types in macros

5. **Multiple parameters `$a:expr, $b:expr`**:
   - Comma separates multiple captures
   - Each parameter can be a different type of match

6. **Repetition patterns `$($num:expr),*`**:
   - `$( )` creates a repetition group
   - `$num:expr` is repeated for each item
   - `,*` means "separated by commas, zero or more times"
   - `*` means zero or more, `+` means one or more

7. **Optional patterns `$($title:expr)?`**:
   - `$( )?` means the pattern inside is optional
   - `?` means zero or one occurrence

8. **Block matching `$code:block`**:
   - `:block` matches code blocks `{ }`
   - Useful for conditional compilation or wrapping code

**Using these different patterns:**

```rust
fn main() {
    // Literal matching
    advanced_macro!(hello);
    
    // Expression matching
    advanced_macro!(print 5 + 3);
    advanced_macro!(print "Hello world");
    
    // Identifier matching
    advanced_macro!(create_var my_number);
    
    // Type matching
    advanced_macro!(create_vec i32);
    advanced_macro!(create_vec String);
    
    // Multiple parameters
    advanced_macro!(add 10, 20);
    
    // Repetition patterns
    advanced_macro!(sum 1, 2, 3, 4, 5);
    advanced_macro!(sum 10, 20);
    
    // Optional patterns
    advanced_macro!(greet "Alice");
    advanced_macro!(greet "Bob", "Dr.");
    
    // Block matching
    advanced_macro!(if_debug {
        println!("This only prints in debug mode");
    });
}
```

**Common pattern types summary:**
- `:expr` - expressions (values, calculations)
- `:ident` - identifiers (names)
- `:ty` - types (i32, String, etc.)
- `:block` - code blocks { }
- `:stmt` - statements
- `:pat` - patterns for matching
- `:path` - paths like `std::vec::Vec`
- `:tt` - token trees (any single token)
- `:literal` - literal values only

#### 11. String Formatting Placeholders
```rust
println!("Hello, {}!", name);
println!("The value is: {}", value);
```
- `{}` act as placeholders in format strings
- They get replaced with the values you provide
- This is different from the other uses - here `{}` are inside strings

#### 12. Scope and Code Blocks
```rust
{
    let x = 5;
    println!("x is: {}", x);
} // x goes out of scope here
```
- `{}` create a new scope for variables
- Variables defined inside are only available within the block
- Used for controlling variable lifetimes

#### Summary of `{}` Usage
- **Code containers**: Function bodies, control flow blocks, modules
- **Data structures**: Struct definitions and instantiation, enum definitions
- **Pattern matching**: Match expressions with multiple arms
- **Implementations**: Impl blocks for methods, trait definitions
- **Macros**: Macro rule definitions and expansions
- **Formatting**: Placeholders in strings (different context)
- **Scoping**: Creating new variable scopes

To compile and run:

```bash
rustc main.rs
./main
```

The `rustc` command compiles the Rust file `main.rs` into an executable. The `./main` command runs the compiled program.

### Comments

```rust
// This is a single-line comment
/* This is a
   multi-line comment */

/// This is a documentation comment for the next item
fn example() {
    // Comments help explain code
    println!("This function demonstrates comments");
}
```

- `//` starts a single-line comment - everything after it on the same line is ignored
- `/* */` creates a multi-line comment - everything between these markers is ignored
- `///` creates documentation comments that generate documentation for the code

## Variables and Data Types

### Variables

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    
    let mut y = 10;
    println!("The value of y is: {}", y);
    y = 15;
    println!("The new value of y is: {}", y);
}
```

- `let` keyword creates a variable
- `x` is the variable name
- `=` assigns a value to the variable
- `5` is the value being assigned
- Variables are immutable (cannot be changed) by default
- `mut` keyword makes a variable mutable (can be changed)
- `{}` in the print statement is a placeholder for the variable value

### Data Types

#### Integer Types

```rust
fn main() {
    let small_number: i8 = 127;        // 8-bit signed integer
    let big_number: i64 = 1000000;     // 64-bit signed integer
    let unsigned_number: u32 = 4000;   // 32-bit unsigned integer
    
    println!("Small number: {}", small_number);
    println!("Big number: {}", big_number);
    println!("Unsigned number: {}", unsigned_number);
}
```

- `i8` is a signed 8-bit integer (can hold values from -128 to 127)
- `i64` is a signed 64-bit integer (can hold very large positive and negative numbers)
- `u32` is an unsigned 32-bit integer (can hold values from 0 to 4,294,967,295)
- `:` after variable name specifies the type explicitly
- Rust can usually infer types, but explicit types help clarity

#### Floating Point Types

```rust
fn main() {
    let price: f64 = 19.99;        // 64-bit floating point
    let weight: f32 = 2.5;         // 32-bit floating point
    
    println!("Price: ${:.2}", price);
    println!("Weight: {} kg", weight);
}
```

- `f64` is a 64-bit floating point number (double precision)
- `f32` is a 32-bit floating point number (single precision)
- `{:.2}` formats the number to 2 decimal places
- Floating point numbers can have decimal values

#### Boolean Type

```rust
fn main() {
    let is_active: bool = true;
    let is_complete: bool = false;
    
    println!("Is active: {}", is_active);
    println!("Is complete: {}", is_complete);
}
```

- `bool` type can only have two values: `true` or `false`
- Boolean values are used in conditional statements

#### Character Type

```rust
fn main() {
    let letter: char = 'A';
    let emoji: char = '😊';
    
    println!("Letter: {}", letter);
    println!("Emoji: {}", emoji);
}
```

- `char` type represents a single character
- Characters are enclosed in single quotes `'`
- Rust `char` can represent Unicode characters including emojis

#### String Types

```rust
fn main() {
    let name: &str = "Alice";              // String slice (immutable)
    let mut greeting: String = String::new(); // Mutable string
    
    greeting.push_str("Hello, ");
    greeting.push_str(name);
    
    println!("{}", greeting);
}
```

- `&str` is a string slice - a reference to a string stored elsewhere
- `String` is an owned, mutable string type
- `String::new()` creates a new empty string
- `push_str()` method adds text to the end of a string
- `::` is used to call associated functions (like static methods)

## Functions

### Basic Functions

```rust
fn greet() {
    println!("Hello from a function!");
}

fn main() {
    greet();
}
```

- Functions are defined with the `fn` keyword
- `greet()` calls the function
- Functions can be called from other functions

### Functions with Parameters

```rust
fn greet_person(name: &str) {
    println!("Hello, {}!", name);
}

fn add_numbers(a: i32, b: i32) {
    let sum = a + b;
    println!("The sum is: {}", sum);
}

fn main() {
    greet_person("Bob");
    add_numbers(5, 3);
}
```

- Parameters are defined inside parentheses `()`
- `name: &str` defines a parameter named `name` of type `&str`
- Multiple parameters are separated by commas
- Arguments are passed when calling the function

### Functions with Return Values

```rust
fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn is_even(number: i32) -> bool {
    number % 2 == 0
}

fn main() {
    let result = multiply(4, 5);
    println!("4 * 5 = {}", result);
    
    let check = is_even(10);
    println!("Is 10 even? {}", check);
}
```

- `->` indicates the return type
- The last expression in a function is returned (no semicolon needed)
- `%` is the modulo operator (returns remainder of division)
- `==` is the equality comparison operator

## Control Flow

### If Statements

```rust
fn main() {
    let number = 7;
    
    if number > 5 {
        println!("Number is greater than 5");
    } else if number < 5 {
        println!("Number is less than 5");
    } else {
        println!("Number is equal to 5");
    }
}
```

- `if` keyword starts a conditional statement
- `>` is the greater than operator
- `<` is the less than operator
- `else if` provides additional conditions
- `else` handles all other cases

### If in Assignment

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };
    
    println!("The number is: {}", number);
}
```

- `if` can be used as an expression to assign values
- Both branches must return the same type
- No semicolon after the values in the if expression

### Loops

#### Loop

```rust
fn main() {
    let mut counter = 0;
    
    loop {
        println!("Counter: {}", counter);
        counter += 1;
        
        if counter == 3 {
            break;
        }
    }
}
```

- `loop` creates an infinite loop
- `+=` is the add and assign operator (adds 1 to counter)
- `break` exits the loop
- `==` checks if two values are equal

#### While Loop

```rust
fn main() {
    let mut number = 5;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("Done!");
}
```

- `while` continues as long as the condition is true
- `!=` is the not equal operator
- `-=` is the subtract and assign operator

#### For Loop

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5];
    
    for number in numbers {
        println!("Number: {}", number);
    }
    
    for i in 0..5 {
        println!("Index: {}", i);
    }
}
```

- `for` iterates over a collection
- `numbers` is an array of integers
- `[1, 2, 3, 4, 5]` creates an array literal
- `in` keyword separates the variable from the collection
- `0..5` creates a range from 0 to 4 (5 is excluded)

## Ownership and Borrowing

### Ownership Rules

Rust's ownership system prevents memory errors. Every value has an owner, and when the owner goes out of scope, the value is dropped.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // s1 is moved to s2
    
    // println!("{}", s1); // This would cause an error
    println!("{}", s2);
}
```

- `String::from()` creates a new String from a string literal
- When `s1` is assigned to `s2`, ownership is transferred (moved)
- `s1` is no longer valid after the move
- This prevents double-free errors

### Borrowing

```rust
fn main() {
    let s1 = String::from("hello");
    let length = calculate_length(&s1);
    
    println!("The length of '{}' is {}.", s1, length);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

- `&s1` creates a reference to `s1` without taking ownership
- `&String` parameter type means the function borrows the string
- `len()` method returns the length of the string
- `usize` is an unsigned integer type for sizes
- The original owner (`s1`) can still be used after borrowing

### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");
    change_string(&mut s);
    println!("{}", s);
}

fn change_string(s: &mut String) {
    s.push_str(", world");
}
```

- `&mut s` creates a mutable reference
- `&mut String` parameter allows the function to modify the string
- Only one mutable reference can exist at a time
- This prevents data races

## Structs

### Defining and Using Structs

```rust
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() {
    let person1 = Person {
        name: String::from("Alice"),
        age: 25,
        email: String::from("alice@example.com"),
    };
    
    println!("Name: {}", person1.name);
    println!("Age: {}", person1.age);
    println!("Email: {}", person1.email);
}
```

- `struct` keyword defines a structure
- `Person` is the struct name (convention: PascalCase)
- Fields are defined with `name: type` syntax
- `{}` creates an instance of the struct
- `.` dot operator accesses struct fields

### Struct Methods

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}

fn main() {
    let rect = Rectangle::new(10, 20);
    println!("Area: {}", rect.area());
}
```

- `impl` keyword implements methods for a struct
- `&self` refers to the instance the method is called on
- `self.width` accesses the width field of the current instance
- `Rectangle::new()` is an associated function (constructor)
- `{ width, height }` uses field init shorthand when variable names match field names

## Enums

### Basic Enums

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

fn main() {
    let direction = Direction::North;
    
    match direction {
        Direction::North => println!("Going north!"),
        Direction::South => println!("Going south!"),
        Direction::East => println!("Going east!"),
        Direction::West => println!("Going west!"),
    }
}
```

- `enum` keyword defines an enumeration
- `Direction` is the enum name
- `North`, `South`, etc. are the enum variants
- `::` accesses enum variants
- `match` pattern matches against enum variants
- `=>` separates the pattern from the action

### Enums with Data

```rust
enum IpAddress {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let home = IpAddress::V4(127, 0, 0, 1);
    let loopback = IpAddress::V6(String::from("::1"));
    
    match home {
        IpAddress::V4(a, b, c, d) => {
            println!("IPv4 address: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddress::V6(addr) => {
            println!("IPv6 address: {}", addr);
        }
    }
}
```

- Enum variants can hold data
- `V4(u8, u8, u8, u8)` holds four u8 values
- `V6(String)` holds a String value
- Pattern matching extracts the data from variants
- `(a, b, c, d)` destructures the tuple data

## Error Handling

### Result Type

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file_result = File::open("hello.txt");
    
    let file = match file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => {
                println!("File not found!");
                return;
            }
            other_error => {
                println!("Error opening file: {:?}", other_error);
                return;
            }
        }
    };
    
    println!("File opened successfully!");
}
```

- `use` keyword imports modules
- `std::fs::File` is the File type from the standard library
- `File::open()` returns a `Result` type
- `Result` can be either `Ok(value)` or `Err(error)`
- `match` handles both success and error cases
- `error.kind()` gets the type of error
- `ErrorKind::NotFound` is a specific error type
- `{:?}` prints the debug representation of a value

### The ? Operator

```rust
use std::fs::File;
use std::io::Read;

fn read_file_contents() -> Result<String, std::io::Error> {
    let mut file = File::open("hello.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file_contents() {
        Ok(contents) => println!("File contents: {}", contents),
        Err(error) => println!("Error: {}", error),
    }
}
```

- `?` operator automatically handles errors
- If the operation succeeds, `?` unwraps the value
- If the operation fails, `?` returns the error early
- `Result<String, std::io::Error>` means the function returns either a String or an error
- `Ok(contents)` wraps the success value
- `read_to_string()` reads the file into a string

## Collections

### Vectors

```rust
fn main() {
    let mut numbers = Vec::new();
    numbers.push(1);
    numbers.push(2);
    numbers.push(3);
    
    println!("Numbers: {:?}", numbers);
    
    let first = &numbers[0];
    println!("First number: {}", first);
    
    for number in &numbers {
        println!("Number: {}", number);
    }
}
```

- `Vec::new()` creates a new empty vector
- `push()` method adds elements to the end
- `{:?}` prints the debug representation
- `[0]` accesses the first element (0-indexed)
- `&numbers` creates a reference to iterate without taking ownership

### Hash Maps

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Red"), 50);
    
    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
    
    match score {
        Some(s) => println!("Score: {}", s),
        None => println!("Team not found"),
    }
    
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

- `HashMap` stores key-value pairs
- `HashMap::new()` creates a new empty hash map
- `insert()` adds key-value pairs
- `get()` retrieves a value by key
- `get()` returns an `Option` type (`Some` or `None`)
- `(key, value)` destructures the pairs during iteration

## Pattern Matching

### Match Expressions

```rust
fn main() {
    let number = 13;
    
    match number {
        1 => println!("One"),
        2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
        13..=19 => println!("A teen"),
        _ => println!("Ain't special"),
    }
}
```

- `match` compares a value against patterns
- `|` means "or" in patterns
- `13..=19` is an inclusive range pattern
- `_` is a catch-all pattern that matches anything

### If Let

```rust
fn main() {
    let some_number = Some(7);
    
    if let Some(number) = some_number {
        println!("Number is: {}", number);
    } else {
        println!("No number");
    }
}
```

- `if let` is a shorter way to match one pattern
- `Some(number)` extracts the value if it exists
- More concise than a full `match` when you only care about one case

## Traits

### Defining Traits

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Article {
    title: String,
    content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}: {}", self.title, self.content)
    }
}

fn main() {
    let article = Article {
        title: String::from("Rust Programming"),
        content: String::from("Rust is a systems programming language."),
    };
    
    println!("{}", article.summarize());
}
```

- `trait` defines shared behavior
- `Summary` is the trait name
- `fn summarize(&self) -> String;` is a method signature
- `impl Summary for Article` implements the trait for a type
- `format!` macro creates formatted strings
- Traits are similar to interfaces in other languages

### Default Implementations

```rust
trait Greeting {
    fn greet(&self) -> String {
        String::from("Hello!")
    }
}

struct Person {
    name: String,
}

impl Greeting for Person {}

fn main() {
    let person = Person {
        name: String::from("Alice"),
    };
    
    println!("{}", person.greet());
}
```

- Traits can provide default implementations
- `impl Greeting for Person {}` uses the default implementation
- Types can override default implementations if needed

## Modules

### Organizing Code

```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }
    
    pub fn subtract(a: i32, b: i32) -> i32 {
        a - b
    }
}

fn main() {
    let sum = math::add(5, 3);
    let difference = math::subtract(10, 4);
    
    println!("Sum: {}", sum);
    println!("Difference: {}", difference);
}
```

- `mod` keyword defines a module
- `pub` makes functions public (accessible from outside the module)
- `math::add()` calls a function from the module
- Modules help organize related code together

### Using External Crates

```rust
// In Cargo.toml:
// [dependencies]
// rand = "0.8"

use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let random_number: i32 = rng.gen_range(1..=100);
    
    println!("Random number: {}", random_number);
}
```

- `Cargo.toml` file lists dependencies
- `use` brings items into scope
- `rand::Rng` is a trait for random number generators
- `thread_rng()` creates a random number generator
- `gen_range()` generates a number in a range

## Testing

### Unit Tests

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
        assert_eq!(add(-1, 1), 0);
    }
    
    #[test]
    fn test_add_negative() {
        assert_eq!(add(-2, -3), -5);
    }
}
```

- `#[cfg(test)]` compiles the module only when testing
- `#[test]` marks a function as a test
- `use super::*;` imports all items from the parent module
- `assert_eq!` checks if two values are equal
- Tests fail if assertions fail

### Running Tests

```bash
cargo test
```

- `cargo test` runs all tests in the project
- Shows which tests pass or fail
- Provides helpful output for debugging

## Common Patterns

### Option Type

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("Alice"))
    } else {
        None
    }
}

fn main() {
    let user = find_user(1);
    
    match user {
        Some(name) => println!("Found user: {}", name),
        None => println!("User not found"),
    }
    
    // Using unwrap_or for default values
    let user_name = find_user(2).unwrap_or(String::from("Guest"));
    println!("User name: {}", user_name);
}
```

- `Option<T>` represents a value that might not exist
- `Some(value)` wraps an existing value
- `None` represents no value
- `unwrap_or()` provides a default value if `None`

### Iterators

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    let doubled: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .collect();
    
    println!("Doubled: {:?}", doubled);
    
    let sum: i32 = numbers
        .iter()
        .sum();
    
    println!("Sum: {}", sum);
}
```

- `iter()` creates an iterator over the collection
- `map()` transforms each element
- `|x| x * 2` is a closure (anonymous function)
- `collect()` consumes the iterator and creates a new collection
- `sum()` adds all elements together

### Closures

```rust
fn main() {
    let add_one = |x| x + 1;
    let result = add_one(5);
    println!("Result: {}", result);
    
    let numbers = vec![1, 2, 3, 4, 5];
    let even_numbers: Vec<i32> = numbers
        .into_iter()
        .filter(|&x| x % 2 == 0)
        .collect();
    
    println!("Even numbers: {:?}", even_numbers);
}
```

- `|x| x + 1` is a closure that takes one parameter
- Closures can capture variables from their environment
- `filter()` keeps only elements that match a condition
- `&x` dereferences the parameter in the closure

## Memory Management

Memory management is one of Rust's most important features. Unlike languages with garbage collection (like Java or Python) or manual memory management (like C or C++), Rust uses a unique ownership system to manage memory safely and efficiently without runtime overhead.

### The Ownership System

The ownership system is Rust's approach to memory management. It prevents memory leaks, double-free errors, and use-after-free bugs at compile time.

#### The Three Rules of Ownership

1. **Each value in Rust has a single owner**
2. **There can only be one owner at a time**
3. **When the owner goes out of scope, the value is dropped**

#### Basic Ownership Example

```rust
fn main() {
    let s1 = String::from("hello");  // s1 owns the string
    let s2 = s1;                     // Ownership moves to s2
    
    // println!("{}", s1);           // ERROR: s1 no longer owns the string
    println!("{}", s2);              // OK: s2 owns the string
}  // s2 goes out of scope, string is dropped
```

- `String::from("hello")` allocates memory on the heap
- When `s1` is assigned to `s2`, ownership is **moved** (not copied)
- `s1` is no longer valid after the move
- When `s2` goes out of scope, the memory is automatically freed

#### Move Semantics

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // s1 is moved to s2
    
    // s1 is no longer valid
    
    let s3 = String::from("world");
    take_ownership(s3);  // s3 is moved into the function
    
    // s3 is no longer valid
}

fn take_ownership(s: String) {
    println!("I own: {}", s);
}  // s goes out of scope and is dropped
```

- Moving prevents double-free errors
- Only one variable can own a value at a time
- The owner is responsible for cleaning up the memory

#### Mutable Variables Can Be Reassigned After Moves

```rust
fn main() {
    let mut s = String::from("hello");
    println!("s: {}", s);
    
    // s loses ownership through a move
    let s2 = s;
    
    // s is no longer valid for "hello"
    // println!("s: {}", s);  // ERROR: s has been moved
    
    // BUT: s can be assigned a new value and gain ownership again
    s = String::from("world");
    println!("s: {}", s);  // OK: s now owns "world"
    
    // s can be moved again
    take_ownership(s);
    
    // And can be reassigned again
    s = String::from("rust");
    println!("s: {}", s);  // OK: s now owns "rust"
}

fn take_ownership(s: String) {
    println!("Function received: {}", s);
}  // s goes out of scope and is dropped
```

- `mut` variables can be reassigned even after losing ownership
- Each assignment gives the variable ownership of a new value
- The variable binding itself is not consumed, only the value it held
- This allows mutable variables to be reused throughout their scope

#### Understanding Variable Bindings vs Values

```rust
fn main() {
    let mut container = String::from("first");
    
    // container owns "first"
    println!("1. container: {}", container);
    
    // Move the value out of container
    let moved_value = container;
    
    // container no longer owns "first", but the binding still exists
    // println!("2. container: {}", container);  // ERROR: moved
    
    // Assign a new value to the same binding
    container = String::from("second");
    
    // container now owns "second"
    println!("3. container: {}", container);
    
    // The binding can be reused multiple times
    container = String::from("third");
    println!("4. container: {}", container);
}
```

- The **variable binding** (like `container`) is different from the **value** it holds
- When ownership is moved, the old value becomes inaccessible
- The variable binding can be assigned a new value
- Each assignment creates a new ownership relationship

#### Practical Example: Reusing Variables

```rust
fn main() {
    let mut data = Vec::new();
    
    // Build first vector
    data.push(1);
    data.push(2);
    data.push(3);
    println!("First vector: {:?}", data);
    
    // Move the vector to a function
    process_vector(data);
    
    // data is no longer valid, but can be reassigned
    data = vec![4, 5, 6];  // Create new vector
    println!("Second vector: {:?}", data);
    
    // Move again
    let final_data = data;
    
    // Reassign again
    data = vec![7, 8, 9];
    println!("Third vector: {:?}", data);
    println!("Final vector: {:?}", final_data);
}

fn process_vector(v: Vec<i32>) {
    println!("Processing: {:?}", v);
}  // v is dropped here
```

- Mutable variables can be reused for different values
- Each reassignment creates a new owned value
- This pattern is useful for resource management and data transformation

#### Clone vs Move

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Creates a deep copy
    
    println!("s1: {}, s2: {}", s1, s2);  // Both are valid
    
    // For simple types, copy happens automatically
    let x = 5;
    let y = x;  // x is copied, not moved
    
    println!("x: {}, y: {}", x, y);  // Both are valid
}
```

- `clone()` creates a deep copy of heap data
- Types that implement `Copy` trait are copied instead of moved
- Simple types like integers implement `Copy`

#### Ownership in Functions

```rust
fn main() {
    let s = String::from("hello");
    
    // s is moved into the function
    let len = calculate_length_move(s);
    
    // s is no longer valid here
    println!("Length: {}", len);
}

fn calculate_length_move(s: String) -> usize {
    let length = s.len();
    length
}  // s goes out of scope and is dropped
```

### Borrowing and References

Borrowing allows you to use a value without taking ownership of it.

#### Immutable References

```rust
fn main() {
    let s = String::from("hello");
    
    // Borrow s instead of moving it
    let len = calculate_length(&s);
    
    // s is still valid here
    println!("The length of '{}' is {}.", s, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}  // s goes out of scope, but it doesn't own the data, so nothing is dropped
```

- `&s` creates an immutable reference to `s`
- The function borrows the value but doesn't own it
- The original owner remains valid

#### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");
    
    change_string(&mut s);
    
    println!("{}", s);  // s has been modified
}

fn change_string(s: &mut String) {
    s.push_str(", world");
}
```

- `&mut s` creates a mutable reference
- The function can modify the borrowed value
- Only one mutable reference can exist at a time

#### Borrowing Rules

```rust
fn main() {
    let mut s = String::from("hello");
    
    // Multiple immutable references are OK
    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    
    // But you can't have immutable and mutable references together
    let r3 = &mut s;  // OK because r1 and r2 are no longer used
    println!("{}", r3);
}
```

**The borrowing rules:**
1. You can have either one mutable reference OR any number of immutable references
2. References must always be valid (no dangling pointers)
3. These rules prevent data races at compile time

### Stack vs Heap

```rust
fn main() {
    // Stack allocation
    let x = 5;                    // Stored on stack
    let y = [1, 2, 3, 4, 5];     // Fixed size, stored on stack
    
    // Heap allocation
    let s = String::from("hello"); // String data stored on heap
    let v = vec![1, 2, 3];        // Vector data stored on heap
    
    println!("Stack value: {}", x);
    println!("Heap string: {}", s);
}
```

**Stack characteristics:**
- Fast allocation and deallocation
- Fixed size known at compile time
- Automatic memory management (LIFO - Last In, First Out)
- Used for local variables and function parameters

**Heap characteristics:**
- Slower allocation and deallocation
- Dynamic size, can grow or shrink
- Requires explicit management (Rust handles this through ownership)
- Used for data that needs to persist beyond function scope

### Smart Pointers

Smart pointers are data structures that act like pointers but have additional capabilities and metadata. They help manage memory in more complex scenarios.

#### Box<T> - Heap Allocation

```rust
fn main() {
    let b = Box::new(5);  // Allocate an integer on the heap
    println!("b = {}", b);
    
    // Useful for large data or recursive types
    let large_array = Box::new([0; 1000]);
    
    // Box is automatically cleaned up when it goes out of scope
}
```

- `Box<T>` allocates data on the heap
- Useful for large data structures
- Required for recursive types
- Implements `Drop` trait for automatic cleanup

#### Rc<T> - Reference Counting

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(String::from("shared data"));
    
    let reference1 = Rc::clone(&data);
    let reference2 = Rc::clone(&data);
    
    println!("Reference count: {}", Rc::strong_count(&data));
    
    // Data is cleaned up when the last reference is dropped
}
```

- `Rc<T>` allows multiple owners of the same data
- Uses reference counting to track owners
- Data is cleaned up when count reaches zero
- Only for single-threaded scenarios

#### Arc<T> - Atomic Reference Counting

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(String::from("shared data"));
    
    let data_clone = Arc::clone(&data);
    
    let handle = thread::spawn(move || {
        println!("Thread: {}", data_clone);
    });
    
    println!("Main: {}", data);
    
    handle.join().unwrap();
}
```

- `Arc<T>` is like `Rc<T>` but thread-safe
- Uses atomic operations for reference counting
- Can be shared between threads
- Slightly more overhead than `Rc<T>`

### RAII (Resource Acquisition Is Initialization)

Rust uses RAII to manage all resources, not just memory.

#### Automatic Resource Management

```rust
use std::fs::File;
use std::io::Write;

fn main() {
    {
        let mut file = File::create("example.txt").unwrap();
        file.write_all(b"Hello, world!").unwrap();
    }  // File is automatically closed here
    
    // File is guaranteed to be closed, even if an error occurs
}
```

- Resources are tied to object lifetimes
- Cleanup happens automatically when objects go out of scope
- Works for files, network connections, mutexes, etc.

#### Custom Drop Implementation

```rust
struct CustomResource {
    name: String,
}

impl Drop for CustomResource {
    fn drop(&mut self) {
        println!("Dropping resource: {}", self.name);
    }
}

fn main() {
    let resource = CustomResource {
        name: String::from("My Resource"),
    };
    
    // Resource cleanup happens automatically
}  // "Dropping resource: My Resource" is printed
```

- Implement `Drop` trait for custom cleanup
- `drop()` method is called automatically
- Guaranteed to run even if panics occur

### Memory Safety Guarantees

Rust provides strong memory safety guarantees without runtime overhead.

#### Prevention of Common Bugs

```rust
fn main() {
    // 1. No null pointer dereferences
    let s: Option<String> = None;
    match s {
        Some(string) => println!("{}", string),
        None => println!("No string"),
    }
    
    // 2. No buffer overflows
    let arr = [1, 2, 3, 4, 5];
    // let x = arr[10];  // This would panic, not corrupt memory
    
    // 3. No use-after-free
    let s = String::from("hello");
    let r = &s;
    // drop(s);  // This would be a compile error
    println!("{}", r);
    
    // 4. No double-free
    let s = String::from("hello");
    // drop(s);
    // drop(s);  // This would be a compile error
}
```

#### Zero-Cost Abstractions

```rust
fn main() {
    // High-level code...
    let numbers = vec![1, 2, 3, 4, 5];
    let doubled: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .collect();
    
    // ...compiles to efficient machine code
    // No runtime overhead for safety
}
```

- Memory safety checks happen at compile time
- No runtime garbage collector
- No performance penalty for safety

### Comparison with Other Languages

#### C/C++ - Manual Memory Management

```c
// C code - manual memory management
char* get_string() {
    char* str = malloc(20);
    strcpy(str, "hello");
    return str;  // Caller must remember to free()
}

int main() {
    char* s = get_string();
    printf("%s\n", s);
    free(s);  // Easy to forget!
    return 0;
}
```

**Problems:**
- Memory leaks if `free()` is forgotten
- Double-free errors if `free()` is called twice
- Use-after-free if memory is accessed after `free()`

#### Java/Python - Garbage Collection

```java
// Java code - garbage collected
public class Example {
    public static void main(String[] args) {
        String s = new String("hello");
        // Memory is automatically cleaned up by GC
    }
}
```

**Trade-offs:**
- Automatic memory management
- Runtime overhead for garbage collection
- Unpredictable pause times
- Higher memory usage

#### Rust - Ownership System

```rust
// Rust code - ownership system
fn main() {
    let s = String::from("hello");
    // Memory is automatically cleaned up
    // No runtime overhead
    // Deterministic cleanup
}
```

**Benefits:**
- Memory safety without garbage collection
- Zero runtime overhead
- Deterministic resource cleanup
- Prevents data races

### Best Practices for Memory Management

#### Use References When Possible

```rust
fn process_string(s: &str) {  // Better: borrow instead of own
    println!("Processing: {}", s);
}

fn process_string_bad(s: String) {  // Takes ownership unnecessarily
    println!("Processing: {}", s);
}

fn main() {
    let s = String::from("hello");
    process_string(&s);     // s is still usable
    process_string(&s);     // Can call multiple times
}
```

#### Use Smart Pointers for Complex Scenarios

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    // Shared ownership with interior mutability
    let data = Rc::new(RefCell::new(String::from("shared")));
    
    let reference1 = Rc::clone(&data);
    let reference2 = Rc::clone(&data);
    
    // Modify through one reference
    reference1.borrow_mut().push_str(" data");
    
    // Access through another reference
    println!("{}", reference2.borrow());
}
```

#### Minimize Cloning

```rust
fn main() {
    let s = String::from("hello");
    
    // Good: pass by reference
    print_string(&s);
    
    // Avoid: unnecessary cloning
    // print_string_bad(s.clone());
}

fn print_string(s: &str) {
    println!("{}", s);
}
```

Rust's memory management system provides safety without sacrificing performance. The ownership system, borrowing rules, and smart pointers work together to prevent common memory bugs while maintaining zero-cost abstractions. This makes Rust ideal for system programming where both safety and performance are critical.

### Lifetimes

**What are lifetimes and why do they exist?**

Lifetimes are Rust's way of ensuring that references (pointers to data) are always valid and never point to memory that has been freed. This prevents crashes, security vulnerabilities, and unpredictable behavior that plague other systems programming languages.

#### The Problem Lifetimes Solve

In languages like C or C++, you can create "dangling pointers" - pointers that point to memory that has been freed:

```c
// This is C code that would crash
char* get_string() {
    char local_string[20] = "Hello";
    return local_string;  // DANGER: returning pointer to local variable!
}
// When this function ends, local_string is destroyed,
// but we returned a pointer to it!
```

Rust prevents this entirely through its lifetime system.

#### Understanding the Lifetime Problem with Examples

Let's see what would happen if Rust didn't have lifetimes (this code won't compile):

```rust
fn main() {
    let result;
    {
        let string1 = String::from("hello");
        result = &string1;  // ERROR: string1 doesn't live long enough
    }  // string1 goes out of scope here and is dropped
    
    println!("{}", result);  // result would be pointing to freed memory!
}
```

- `result` is declared in the outer scope
- `string1` is created in the inner scope `{}`
- We try to store a reference to `string1` in `result`
- When the inner scope ends, `string1` is destroyed
- `result` would be pointing to freed memory (dangling pointer)
- Rust prevents this at compile time

#### How Lifetimes Work

Lifetimes are annotations that tell Rust how long references should remain valid:

```rust
fn main() {
    let string1 = String::from("hello");
    let string2 = String::from("world");
    
    // Both strings live for the same duration
    let result = longest(&string1, &string2);
    println!("Longest: {}", result);
}  // Both string1 and string2 are dropped here

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

**Breaking down the lifetime annotation:**
- `<'a>` declares a lifetime parameter called `'a` (you can name it anything)
- `&'a str` means "a reference to a string that lives for at least lifetime 'a"
- The return type `&'a str` means "returns a reference that lives for lifetime 'a"
- This tells Rust: "the returned reference will be valid as long as both input references are valid"

#### Why This Function Needs Lifetime Annotations

Without lifetimes, Rust doesn't know which reference we're returning:

```rust
// This won't compile - Rust doesn't know the lifetime of the return value
fn longest_broken(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x  // Could be returning x
    } else {
        y  // Or could be returning y
    }
}
```

With lifetimes, we tell Rust: "the returned reference has the same lifetime as the input references":

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

#### Different Lifetime Scenarios

**Scenario 1: References with different lifetimes**

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    
    {
        let string2 = String::from("short");
        result = longest(&string1, &string2);
        println!("Longest: {}", result);  // OK - used while both are valid
    }  // string2 is dropped here
    
    // println!("{}", result);  // ERROR: string2 is gone, so result is invalid
}
```

- `string1` lives for the entire main function
- `string2` only lives within the inner scope
- The lifetime `'a` is the shorter of the two (string2's lifetime)
- `result` can only be used while both strings are alive

**Scenario 2: One reference outlives the other**

```rust
fn main() {
    let string1 = String::from("long string is long");
    
    {
        let string2 = String::from("short");
        let result = longest(&string1, &string2);
        println!("Longest: {}", result);  // OK - both are alive here
    }  // string2 and result are dropped here
    
    // string1 is still alive, but result is gone
}
```

#### Lifetime Elision (When You Don't Need to Write Lifetimes)

Rust can often infer lifetimes automatically. These three functions are equivalent:

```rust
// 1. Explicit lifetime (what the compiler sees)
fn first_word_explicit<'a>(s: &'a str) -> &'a str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

// 2. Elided lifetime (what you can write)
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

// 3. They work the same way
fn main() {
    let sentence = String::from("hello world");
    let word = first_word(&sentence);
    println!("First word: {}", word);
}
```

**Lifetime elision rules:**
1. Each parameter gets its own lifetime
2. If there's exactly one input lifetime, it's assigned to all outputs
3. If there's a `&self` parameter, its lifetime is assigned to all outputs

#### Lifetimes in Structs

Structs can hold references, but they need lifetime annotations:

```rust
// This struct holds a reference to a string
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    
    let excerpt = ImportantExcerpt {
        part: first_sentence,
    };
    
    println!("Excerpt: {}", excerpt.part);
}  // excerpt and novel are both dropped here
```

- `ImportantExcerpt<'a>` means the struct needs a lifetime parameter
- `part: &'a str` means the reference inside must live for lifetime `'a`
- The struct cannot outlive the data it references

#### Multiple Lifetimes

Sometimes you need multiple lifetime parameters:

```rust
fn compare_strings<'a, 'b>(x: &'a str, y: &'b str) -> bool {
    x.len() > y.len()
}

fn main() {
    let string1 = String::from("long");
    {
        let string2 = String::from("short");
        let is_longer = compare_strings(&string1, &string2);
        println!("Is longer: {}", is_longer);
    }
}
```

- `'a` and `'b` are different lifetimes
- The parameters can have different lifetimes
- The return type `bool` is not a reference, so no lifetime needed

#### Static Lifetime

The `'static` lifetime means the reference lives for the entire program:

```rust
fn main() {
    let s: &'static str = "I live for the entire program";
    println!("{}", s);
}
```

- String literals have `'static` lifetime
- They're stored in the program's binary
- Available for the entire program execution

#### Common Lifetime Patterns

**Pattern 1: Returning one of the input references**

```rust
fn choose_first<'a>(x: &'a str, y: &str) -> &'a str {
    x  // Always return the first one
}
```

**Pattern 2: Creating new data (no lifetime needed)**

```rust
fn create_greeting(name: &str) -> String {
    format!("Hello, {}!", name)  // Returns owned String, not reference
}
```

**Pattern 3: Methods on structs**

```rust
struct Person<'a> {
    name: &'a str,
}

impl<'a> Person<'a> {
    fn get_name(&self) -> &str {
        self.name  // Lifetime is inferred from &self
    }
}
```

#### Why Lifetimes Are Important

1. **Memory Safety**: Prevents dangling pointers and use-after-free bugs
2. **No Runtime Cost**: All checking happens at compile time
3. **No Garbage Collector**: Manual memory management without the dangers
4. **Predictable Performance**: No unexpected pauses for garbage collection
5. **Catch Bugs Early**: Memory errors found at compile time, not runtime

#### Summary

- **Lifetimes ensure references are always valid**
- **They prevent dangling pointers and memory bugs**
- **Rust infers most lifetimes automatically**
- **You only need to write them when Rust can't figure them out**
- **They have zero runtime cost - all checking is at compile time**
- **They're what makes Rust memory-safe without garbage collection**

The key insight is: lifetimes are not about managing memory directly, but about ensuring that references (pointers) always point to valid memory. This is what makes Rust both safe and fast.

## Advanced Features

### Generics

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Point<T> {
        Point { x, y }
    }
}

fn main() {
    let integer_point = Point::new(5, 10);
    let float_point = Point::new(1.0, 4.0);
    
    println!("Integer point: ({}, {})", integer_point.x, integer_point.y);
    println!("Float point: ({}, {})", float_point.x, float_point.y);
}
```

- `<T>` declares a generic type parameter
- `T` can be any type
- Same code works with different types
- Generics enable code reuse without sacrificing performance

### Macros

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
    ($name:expr) => {
        println!("Hello, {}!", $name);
    };
}

fn main() {
    say_hello!();
    say_hello!("World");
}
```

- `macro_rules!` defines a macro
- `() => {}` is a macro arm for no parameters
- `($name:expr) => {}` takes an expression parameter
- `$name` uses the captured parameter
- Macros generate code at compile time

## Best Practices

### Error Handling

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file("config.txt") {
        Ok(contents) => println!("File contents: {}", contents),
        Err(error) => eprintln!("Error reading file: {}", error),
    }
}
```

- Always handle errors explicitly
- Use `Result` type for operations that can fail
- `?` operator for error propagation
- `eprintln!` prints to stderr instead of stdout

### Documentation

```rust
/// Calculates the area of a circle
/// 
/// # Arguments
/// 
/// * `radius` - The radius of the circle
/// 
/// # Examples
/// 
/// ```
/// let area = circle_area(5.0);
/// assert_eq!(area, 78.53981633974483);
/// ```
fn circle_area(radius: f64) -> f64 {
    std::f64::consts::PI * radius * radius
}
```

- Use `///` for documentation comments
- Include examples in documentation
- `cargo doc` generates HTML documentation
- Good documentation helps other developers

## Cargo and Project Management

Cargo is Rust's package manager and build system. It handles project creation, dependency management, building, testing, and publishing. Cargo makes Rust development much easier by automating common tasks.

### Creating a New Project

#### Creating a Binary Project (Executable)

```bash
cargo new hello_world
cd hello_world
```

The `cargo new` command creates a new project. `hello_world` is the project name. `cd` changes into the project directory.

This creates the following structure:
```
hello_world/
├── Cargo.toml
├── src/
│   └── main.rs
└── .gitignore
```

- `Cargo.toml` is the project configuration file
- `src/main.rs` is the main source file for executables
- `.gitignore` ignores build artifacts in version control

#### Creating a Library Project

```bash
cargo new my_library --lib
cd my_library
```

The `--lib` flag creates a library project instead of an executable.

This creates:
```
my_library/
├── Cargo.toml
├── src/
│   └── lib.rs
└── .gitignore
```

- `src/lib.rs` is the main source file for libraries
- Libraries are code that other programs can use

### Understanding Cargo.toml

The `Cargo.toml` file contains project metadata and dependencies:

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
```

- `[package]` section contains project information
- `name` is the project name
- `version` follows semantic versioning (major.minor.patch)
- `edition` specifies which Rust edition to use
- `[dependencies]` section lists external packages

### Building and Running Projects

#### Building a Project

```bash
cargo build
```

The `cargo build` command compiles the project. It creates a `target/debug/` directory with the compiled executable.

#### Running a Project

```bash
cargo run
```

The `cargo run` command builds and runs the project in one step. This is faster than separate build and run commands.

#### Building for Release

```bash
cargo build --release
```

The `--release` flag creates an optimized build. The executable goes in `target/release/`. Release builds are slower to compile but run faster.

### Managing Dependencies

#### Adding Dependencies

To add a dependency, edit `Cargo.toml`:

```toml
[dependencies]
rand = "0.8"
serde = { version = "1.0", features = ["derive"] }
```

- `rand = "0.8"` adds the rand crate version 0.8.x
- `serde = { version = "1.0", features = ["derive"] }` adds serde with specific features
- `features` are optional parts of a crate you can enable

#### Using Dependencies in Code

```rust
use rand::Rng;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let mut rng = rand::thread_rng();
    let random_number = rng.gen_range(1..=100);
    
    let person = Person {
        name: String::from("Alice"),
        age: random_number,
    };
    
    println!("Random person: {} is {} years old", person.name, person.age);
}
```

- `use` brings external crate items into scope
- `#[derive(Serialize, Deserialize)]` automatically implements traits
- External crates work just like built-in Rust code

#### Updating Dependencies

```bash
cargo update
```

The `cargo update` command updates dependencies to their latest compatible versions. It respects version constraints in `Cargo.toml`.

### Workspaces

Workspaces allow you to manage multiple related packages together.

#### Creating a Workspace

Create a `Cargo.toml` file in the root directory:

```toml
[workspace]
members = [
    "my_app",
    "my_library",
    "shared_utils",
]
```

- `[workspace]` defines a workspace
- `members` lists all packages in the workspace
- Each member has its own `Cargo.toml` file

#### Workspace Structure

```
my_workspace/
├── Cargo.toml          # Workspace configuration
├── my_app/
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
├── my_library/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── shared_utils/
    ├── Cargo.toml
    └── src/
        └── lib.rs
```

#### Using Workspace Dependencies

In `my_app/Cargo.toml`:

```toml
[dependencies]
my_library = { path = "../my_library" }
shared_utils = { path = "../shared_utils" }
```

- `path` specifies the location of local dependencies
- Workspace members can depend on each other

### Modules and Code Organization

#### Single File Modules

```rust
// In src/main.rs or src/lib.rs
mod utils {
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }
    
    pub fn multiply(a: i32, b: i32) -> i32 {
        a * b
    }
}

fn main() {
    let sum = utils::add(5, 3);
    let product = utils::multiply(4, 6);
    
    println!("Sum: {}, Product: {}", sum, product);
}
```

- `mod utils` declares a module named `utils`
- `pub` makes functions public (accessible from outside the module)
- `utils::add()` calls a function from the module

#### Separate File Modules

Create `src/utils.rs`:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn multiply(a: i32, b: i32) -> i32 {
    a * b
}
```

In `src/main.rs`:

```rust
mod utils;

fn main() {
    let sum = utils::add(5, 3);
    let product = utils::multiply(4, 6);
    
    println!("Sum: {}, Product: {}", sum, product);
}
```

- `mod utils;` tells Rust to look for `utils.rs` file
- The module functions are defined in the separate file

#### Directory Modules

Create `src/math/` directory with `src/math/mod.rs`:

```rust
pub mod basic;
pub mod advanced;
```

Create `src/math/basic.rs`:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn subtract(a: i32, b: i32) -> i32 {
    a - b
}
```

Create `src/math/advanced.rs`:

```rust
pub fn power(base: i32, exponent: u32) -> i32 {
    base.pow(exponent)
}

pub fn factorial(n: u32) -> u32 {
    match n {
        0 | 1 => 1,
        _ => n * factorial(n - 1),
    }
}
```

In `src/main.rs`:

```rust
mod math;

fn main() {
    let sum = math::basic::add(5, 3);
    let power = math::advanced::power(2, 3);
    let fact = math::advanced::factorial(5);
    
    println!("Sum: {}, Power: {}, Factorial: {}", sum, power, fact);
}
```

- `mod.rs` defines the module structure
- `pub mod basic;` makes the submodule public
- `math::basic::add()` accesses nested module functions

### Building Libraries

#### Library Structure

A library project has this structure:

```
my_library/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── utils.rs
│   └── models/
│       ├── mod.rs
│       ├── user.rs
│       └── product.rs
├── tests/
│   └── integration_tests.rs
└── examples/
    └── basic_usage.rs
```

#### src/lib.rs

```rust
pub mod utils;
pub mod models;

pub use models::user::User;
pub use models::product::Product;

pub fn library_version() -> &'static str {
    "1.0.0"
}
```

- `pub mod` exposes modules to library users
- `pub use` re-exports items for easier access
- `pub fn` exposes functions to library users

#### Publishing Configuration

In `Cargo.toml`:

```toml
[package]
name = "my_awesome_library"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
description = "A library that does awesome things"
license = "MIT OR Apache-2.0"
repository = "https://github.com/yourusername/my_awesome_library"
homepage = "https://github.com/yourusername/my_awesome_library"
documentation = "https://docs.rs/my_awesome_library"
readme = "README.md"
keywords = ["awesome", "library", "rust"]
categories = ["development-tools"]

[dependencies]
```

- `authors` lists the package authors
- `description` describes what the library does
- `license` specifies the license type
- `repository` points to the source code
- `keywords` help users find the library
- `categories` classify the library type

### Testing

#### Unit Tests

```rust
// In src/lib.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
        assert_eq!(add(-1, 1), 0);
        assert_eq!(add(0, 0), 0);
    }
    
    #[test]
    fn test_add_negative() {
        assert_eq!(add(-2, -3), -5);
    }
}
```

#### Integration Tests

Create `tests/integration_tests.rs`:

```rust
use my_library::add;

#[test]
fn test_library_integration() {
    let result = add(10, 20);
    assert_eq!(result, 30);
}
```

#### Running Tests

```bash
cargo test
```

- Runs all unit tests and integration tests
- Shows which tests pass or fail

```bash
cargo test test_add
```

- Runs only tests matching the name pattern

### Documentation

#### Writing Documentation

```rust
/// Adds two numbers together
/// 
/// # Arguments
/// 
/// * `a` - The first number
/// * `b` - The second number
/// 
/// # Examples
/// 
/// ```
/// use my_library::add;
/// 
/// let result = add(5, 3);
/// assert_eq!(result, 8);
/// ```
/// 
/// # Panics
/// 
/// This function does not panic.
/// 
/// # Errors
/// 
/// This function does not return errors.
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

#### Generating Documentation

```bash
cargo doc
```

- Generates HTML documentation from doc comments
- Creates documentation in `target/doc/`

```bash
cargo doc --open
```

- Generates and opens documentation in your browser

### Publishing to crates.io

#### Preparing for Publishing

1. Create account on crates.io
2. Generate API token
3. Configure authentication:

```bash
cargo login your_api_token_here
```

#### Publishing

```bash
cargo publish
```

- Uploads your crate to crates.io
- Makes it available for others to use

#### Versioning

```bash
cargo publish --dry-run
```

- Tests the publishing process without actually publishing
- Useful for checking everything is correct

### Common Cargo Commands

#### Development Commands

```bash
cargo new project_name          # Create new project
cargo new --lib lib_name        # Create new library
cargo build                     # Build project
cargo run                       # Build and run
cargo check                     # Check code without building
cargo test                      # Run tests
cargo doc                       # Generate documentation
```

#### Dependency Management

```bash
cargo update                    # Update dependencies
cargo tree                      # Show dependency tree
cargo outdated                  # Show outdated dependencies (requires cargo-outdated)
```

#### Maintenance Commands

```bash
cargo clean                     # Remove build artifacts
cargo fmt                       # Format code (requires rustfmt)
cargo clippy                    # Lint code (requires clippy)
```

### Best Practices

#### Project Structure

```
my_project/
├── Cargo.toml
├── README.md
├── src/
│   ├── lib.rs or main.rs
│   ├── utils.rs
│   └── models/
│       ├── mod.rs
│       └── user.rs
├── tests/
│   └── integration_tests.rs
├── examples/
│   └── basic_usage.rs
└── docs/
    └── user_guide.md
```

#### Dependency Management

- Use specific version numbers for important dependencies
- Group related dependencies together
- Use `dev-dependencies` for testing and development tools
- Keep dependencies up to date

#### Code Organization

- Use modules to organize related functionality
- Keep modules focused on single responsibilities
- Use `pub` carefully to control the public API
- Write comprehensive documentation
- Include examples in documentation

Cargo simplifies Rust development by handling the complex tasks of building, testing, and managing dependencies. It follows conventions that make Rust projects consistent and easy to work with.

## Interoperability with C and Other Languages

Rust has excellent support for interoperating with C and other languages. This makes it easy to integrate Rust into existing projects, use existing C libraries, or create libraries that can be called from other languages.

### Foreign Function Interface (FFI)

FFI allows Rust to call functions written in other languages (primarily C) and allows other languages to call Rust functions.

#### Calling C Functions from Rust

```rust
// Declare external C functions
extern "C" {
    fn abs(input: i32) -> i32;
    fn sqrt(input: f64) -> f64;
    fn strlen(s: *const i8) -> usize;
}

fn main() {
    let x = -42;
    let result = unsafe { abs(x) };
    println!("abs({}) = {}", x, result);
    
    let num = 16.0;
    let result = unsafe { sqrt(num) };
    println!("sqrt({}) = {}", num, result);
}
```

- `extern "C"` declares functions from C libraries
- `abs`, `sqrt`, `strlen` are standard C library functions
- `unsafe` blocks are required when calling C functions
- C functions don't provide Rust's safety guarantees

#### Using C Libraries with Cargo

Create a `build.rs` file to link C libraries:

```rust
// build.rs
fn main() {
    println!("cargo:rustc-link-lib=m");  // Link math library
    println!("cargo:rustc-link-search=native=/usr/lib");
}
```

In `Cargo.toml`:

```toml
[build-dependencies]
cc = "1.0"
```

Example using a custom C library:

```rust
// src/lib.rs
extern "C" {
    fn custom_add(a: i32, b: i32) -> i32;
}

pub fn safe_add(a: i32, b: i32) -> i32 {
    unsafe { custom_add(a, b) }
}
```

```c
// custom.c
int custom_add(int a, int b) {
    return a + b;
}
```

```rust
// build.rs
use cc;

fn main() {
    cc::Build::new()
        .file("custom.c")
        .compile("libcustom.a");
}
```

#### Working with C Data Types

Rust provides types that correspond to C types:

```rust
use std::os::raw::{c_char, c_int, c_double, c_void};
use std::ffi::{CStr, CString};

extern "C" {
    fn c_function(
        number: c_int,
        text: *const c_char,
        data: *mut c_void,
        size: usize,
    ) -> c_int;
}

fn main() {
    let number = 42;
    let text = CString::new("Hello from Rust").unwrap();
    let mut buffer = vec![0u8; 100];
    
    let result = unsafe {
        c_function(
            number,
            text.as_ptr(),
            buffer.as_mut_ptr() as *mut c_void,
            buffer.len(),
        )
    };
    
    println!("C function returned: {}", result);
}
```

- `c_int`, `c_char`, etc. are C-compatible types
- `CString` converts Rust strings to C strings
- `CStr` converts C strings to Rust strings
- Raw pointers (`*const`, `*mut`) interface with C pointers

#### String Handling Between Rust and C

```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

extern "C" {
    fn process_string(s: *const c_char) -> *mut c_char;
    fn free_string(s: *mut c_char);
}

fn main() {
    // Rust string to C string
    let rust_string = "Hello, C!";
    let c_string = CString::new(rust_string).unwrap();
    
    unsafe {
        // Call C function
        let result_ptr = process_string(c_string.as_ptr());
        
        // Convert C string back to Rust string
        let result_cstr = CStr::from_ptr(result_ptr);
        let result_string = result_cstr.to_str().unwrap();
        
        println!("Result from C: {}", result_string);
        
        // Free the C-allocated string
        free_string(result_ptr);
    }
}
```

- `CString::new()` creates a null-terminated C string
- `CStr::from_ptr()` wraps a C string pointer
- Always free C-allocated memory properly
- Handle encoding issues between Rust UTF-8 and C strings

### Creating C-Compatible Libraries

You can create Rust libraries that can be called from C and other languages.

#### Creating a Dynamic Library

```rust
// src/lib.rs
use std::os::raw::c_int;

#[no_mangle]
pub extern "C" fn add(a: c_int, b: c_int) -> c_int {
    a + b
}

#[no_mangle]
pub extern "C" fn multiply(a: c_int, b: c_int) -> c_int {
    a * b
}

#[no_mangle]
pub extern "C" fn fibonacci(n: c_int) -> c_int {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

```toml
# Cargo.toml
[package]
name = "mathlib"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]
```

- `#[no_mangle]` prevents name mangling so C can find the function
- `pub extern "C"` makes the function callable from C
- `cdylib` creates a C-compatible dynamic library

#### Creating a Header File

```c
// mathlib.h
#ifndef MATHLIB_H
#define MATHLIB_H

extern int add(int a, int b);
extern int multiply(int a, int b);
extern int fibonacci(int n);

#endif
```

#### Using the Library from C

```c
// main.c
#include <stdio.h>
#include "mathlib.h"

int main() {
    int result = add(5, 3);
    printf("5 + 3 = %d\n", result);
    
    result = multiply(4, 6);
    printf("4 * 6 = %d\n", result);
    
    result = fibonacci(10);
    printf("fibonacci(10) = %d\n", result);
    
    return 0;
}
```

Compile and link:

```bash
# Build the Rust library
cargo build --release

# Compile and link the C program
gcc -o main main.c -L./target/release -lmathlib
```

#### Working with Complex Data Structures

```rust
use std::os::raw::{c_char, c_int};
use std::ffi::{CStr, CString};

#[repr(C)]
pub struct Person {
    name: *mut c_char,
    age: c_int,
}

#[no_mangle]
pub extern "C" fn create_person(name: *const c_char, age: c_int) -> *mut Person {
    let c_str = unsafe { CStr::from_ptr(name) };
    let rust_str = c_str.to_str().unwrap();
    let owned_name = CString::new(rust_str).unwrap();
    
    let person = Person {
        name: owned_name.into_raw(),
        age,
    };
    
    Box::into_raw(Box::new(person))
}

#[no_mangle]
pub extern "C" fn get_person_name(person: *const Person) -> *const c_char {
    let person = unsafe { &*person };
    person.name
}

#[no_mangle]
pub extern "C" fn get_person_age(person: *const Person) -> c_int {
    let person = unsafe { &*person };
    person.age
}

#[no_mangle]
pub extern "C" fn free_person(person: *mut Person) {
    if !person.is_null() {
        unsafe {
            let person = Box::from_raw(person);
            CString::from_raw(person.name);
        }
    }
}
```

- `#[repr(C)]` ensures C-compatible memory layout
- `Box::into_raw()` converts to raw pointer for C
- `Box::from_raw()` converts back to owned value for cleanup
- Always provide cleanup functions for complex data

### Interfacing with Other Languages

#### Python Integration

Using PyO3 to create Python extensions:

```toml
# Cargo.toml
[dependencies]
pyo3 = { version = "0.17", features = ["extension-module"] }

[lib]
crate-type = ["cdylib"]
```

```rust
// src/lib.rs
use pyo3::prelude::*;

#[pyfunction]
fn add(a: usize, b: usize) -> PyResult<usize> {
    Ok(a + b)
}

#[pyfunction]
fn process_list(list: Vec<i32>) -> PyResult<Vec<i32>> {
    Ok(list.iter().map(|x| x * 2).collect())
}

#[pymodule]
fn mymodule(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(add, m)?)?;
    m.add_function(wrap_pyfunction!(process_list, m)?)?;
    Ok(())
}
```

```python
# test.py
import mymodule

result = mymodule.add(5, 3)
print(f"5 + 3 = {result}")

numbers = [1, 2, 3, 4, 5]
doubled = mymodule.process_list(numbers)
print(f"Doubled: {doubled}")
```

#### JavaScript Integration (WebAssembly)

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

```toml
# Cargo.toml
[dependencies]
wasm-bindgen = "0.2"

[lib]
crate-type = ["cdylib"]
```

```javascript
// index.js
import { greet, fibonacci } from './pkg/mylib.js';

console.log(greet('WebAssembly'));
console.log(`fibonacci(10) = ${fibonacci(10)}`);
```

### Safety Considerations

#### Unsafe Code Guidelines

```rust
use std::slice;

// BAD: Potential undefined behavior
unsafe fn bad_example(ptr: *const i32, len: usize) -> i32 {
    let slice = slice::from_raw_parts(ptr, len);
    slice.iter().sum()
}

// GOOD: Proper safety checks
unsafe fn good_example(ptr: *const i32, len: usize) -> Option<i32> {
    if ptr.is_null() || len == 0 {
        return None;
    }
    
    let slice = slice::from_raw_parts(ptr, len);
    Some(slice.iter().sum())
}

// BETTER: Safe wrapper
fn safe_sum(data: &[i32]) -> i32 {
    data.iter().sum()
}
```

#### Memory Management Across FFI

```rust
use std::ffi::CString;
use std::os::raw::c_char;

// Rust allocates, C frees
#[no_mangle]
pub extern "C" fn rust_allocate_string() -> *mut c_char {
    let s = CString::new("Hello from Rust").unwrap();
    s.into_raw()
}

// C allocates, Rust frees
#[no_mangle]
pub extern "C" fn rust_free_string(s: *mut c_char) {
    if !s.is_null() {
        unsafe { CString::from_raw(s) };
    }
}

// Always document ownership rules
/// Caller must free the returned string with rust_free_string
#[no_mangle]
pub extern "C" fn get_message() -> *mut c_char {
    let msg = CString::new("Remember to free me!").unwrap();
    msg.into_raw()
}
```

### Best Practices

#### Error Handling in FFI

```rust
use std::os::raw::c_int;

// Return codes for error handling
const SUCCESS: c_int = 0;
const ERROR_NULL_POINTER: c_int = -1;
const ERROR_INVALID_INPUT: c_int = -2;

#[no_mangle]
pub extern "C" fn safe_divide(a: c_int, b: c_int, result: *mut c_int) -> c_int {
    if result.is_null() {
        return ERROR_NULL_POINTER;
    }
    
    if b == 0 {
        return ERROR_INVALID_INPUT;
    }
    
    unsafe {
        *result = a / b;
    }
    
    SUCCESS
}
```

#### Documentation and Testing

```rust
/// Adds two integers safely
/// 
/// # Safety
/// 
/// This function is safe to call from C code.
/// 
/// # Arguments
/// 
/// * `a` - First integer
/// * `b` - Second integer
/// 
/// # Returns
/// 
/// The sum of `a` and `b`
#[no_mangle]
pub extern "C" fn add_safe(a: i32, b: i32) -> i32 {
    a.saturating_add(b)
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add_safe() {
        assert_eq!(add_safe(2, 3), 5);
        assert_eq!(add_safe(i32::MAX, 1), i32::MAX);
    }
}
```

### Tools and Utilities

#### Bindgen for Automatic Bindings

```toml
# Cargo.toml
[build-dependencies]
bindgen = "0.60"
```

```rust
// build.rs
use bindgen;

fn main() {
    let bindings = bindgen::Builder::default()
        .header("wrapper.h")
        .parse_callbacks(Box::new(bindgen::CargoCallbacks))
        .generate()
        .expect("Unable to generate bindings");
    
    let out_path = std::path::PathBuf::from(std::env::var("OUT_DIR").unwrap());
    bindings
        .write_to_file(out_path.join("bindings.rs"))
        .expect("Couldn't write bindings!");
}
```

#### cbindgen for C Header Generation

```toml
# Cargo.toml
[build-dependencies]
cbindgen = "0.20"
```

```rust
// build.rs
use cbindgen;

fn main() {
    let crate_dir = std::env::var("CARGO_MANIFEST_DIR").unwrap();
    
    cbindgen::Builder::new()
        .with_crate(crate_dir)
        .generate()
        .expect("Unable to generate bindings")
        .write_to_file("target/mylib.h");
}
```

Rust's FFI capabilities make it an excellent choice for integrating with existing systems, creating high-performance libraries for other languages, and gradually migrating legacy code to Rust. The combination of safety and performance makes Rust particularly valuable for creating reliable system interfaces.

## Conclusion

Rust is a powerful systems programming language that provides memory safety without garbage collection. Its ownership system, type system, and error handling make it excellent for building reliable and efficient software. The examples in this document demonstrate the fundamental concepts needed to start programming in Rust.

Key takeaways:
- Ownership prevents memory errors
- Pattern matching provides powerful control flow
- Traits enable code reuse and abstraction
- The compiler helps catch errors early
- Explicit error handling makes programs more robust

Continue practicing with these concepts and gradually explore more advanced features as you become comfortable with the basics. 