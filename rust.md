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

- Stack: Fast, fixed size, automatically managed
- Heap: Slower, dynamic size, manually managed (but Rust handles this)
- Simple types like integers go on the stack
- Complex types like String and Vec use the heap

### Lifetimes

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string");
    let string2 = String::from("short");
    
    let result = longest(&string1, &string2);
    println!("Longest string: {}", result);
}
```

- `<'a>` declares a lifetime parameter
- `&'a str` means the reference lives for lifetime 'a
- Lifetimes ensure references are valid
- The compiler usually infers lifetimes automatically

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

## Conclusion

Rust is a powerful systems programming language that provides memory safety without garbage collection. Its ownership system, type system, and error handling make it excellent for building reliable and efficient software. The examples in this document demonstrate the fundamental concepts needed to start programming in Rust.

Key takeaways:
- Ownership prevents memory errors
- Pattern matching provides powerful control flow
- Traits enable code reuse and abstraction
- The compiler helps catch errors early
- Explicit error handling makes programs more robust

Continue practicing with these concepts and gradually explore more advanced features as you become comfortable with the basics. 