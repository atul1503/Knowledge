# Java Programming Guide

## What is Java?

Java is a high-level, object-oriented programming language designed to be platform-independent. Java code is compiled into bytecode, which runs on the Java Virtual Machine (JVM), making it portable across different operating systems.

- **Write Once, Run Anywhere:** Java programs can run on any device with a JVM.
- **Strongly typed:** Variables must have a declared type.
- **Garbage collected:** Memory management is automatic.

## Installation and Setup

1. **Install the JDK (Java Development Kit):**
   - Download from [Oracle](https://www.oracle.com/java/technologies/downloads/) or use OpenJDK.
   - Add `java` and `javac` to your PATH.
2. **Check installation:**
   ```bash
   java -version
   javac -version
   ```
3. **Write code in any text editor or use an IDE (IntelliJ, Eclipse, VS Code).**

## Basic Syntax

- **Main method:** Entry point for Java applications.
  ```java
  public class Main {
      public static void main(String[] args) {
          System.out.println("Hello, World!");
      }
  }
  ```
- **Statements end with `;`**
- **Curly braces `{}`** define code blocks.
- **Comments:**
  - Single line: `// comment`
  - Multi-line: `/* comment */`

## Variables and Types

- **Primitive types:** `int`, `double`, `char`, `boolean`, `float`, `long`, `short`, `byte`
- **Reference types:** Objects, arrays, etc.
- **Declare variables:**
  ```java
  int age = 25;
  double price = 19.99;
  char grade = 'A';
  boolean isActive = true;
  String name = "Alice"; // String is a class
  ```

## Control Flow

- **If-else:**
  ```java
  if (age > 18) {
      System.out.println("Adult");
  } else {
      System.out.println("Minor");
  }
  ```
- **Loops:**
  - For:
    ```java
    for (int i = 0; i < 5; i++) {
        System.out.println(i);
    }
    ```
  - While:
    ```java
    int i = 0;
    while (i < 5) {
        System.out.println(i);
        i++;
    }
    ```
  - Do-while:
    ```java
    int i = 0;
    do {
        System.out.println(i);
        i++;
    } while (i < 5);
    ```

## Object-Oriented Programming (OOP)

- **Class:** Blueprint for objects.
  ```java
  public class Person {
      String name;
      int age;

      public Person(String name, int age) {
          this.name = name;
          this.age = age;
      }

      public void greet() {
          System.out.println("Hello, my name is " + name);
      }
  }
  ```
- **Object:** Instance of a class.
  ```java
  Person p = new Person("Bob", 30);
  p.greet();
  ```
- **Inheritance:**
  ```java
  public class Student extends Person {
      int grade;
      public Student(String name, int age, int grade) {
          super(name, age);
          this.grade = grade;
      }
  }
  ```
- **Interfaces:**
  ```java
  interface Drawable {
      void draw();
  }
  class Circle implements Drawable {
      public void draw() {
          System.out.println("Drawing circle");
      }
  }
  ```
- **Access Modifiers:** `public`, `private`, `protected`, (default)

## Collections

- **Array:** Fixed size.
  ```java
  int[] numbers = {1, 2, 3};
  System.out.println(numbers[0]); // 1
  ```
- **ArrayList:** Dynamic size.
  ```java
  import java.util.ArrayList;
  ArrayList<String> list = new ArrayList<>();
  list.add("apple");
  list.add("banana");
  System.out.println(list.get(0)); // apple
  ```
- **HashMap:** Key-value pairs.
  ```java
  import java.util.HashMap;
  HashMap<String, Integer> map = new HashMap<>();
  map.put("a", 1);
  map.put("b", 2);
  System.out.println(map.get("a")); // 1
  ```

## Concurrency and Non-Linear Code Execution

Java provides several ways to run code concurrently (at the same time) or in a non-linear fashion (not just top-to-bottom). This is useful for tasks like handling multiple users, background work, or speeding up computations.

### 1. Threads and Runnable
- **Thread:** Smallest unit of execution. Each thread runs independently.
- **Runnable:** Interface for code you want to run in a thread.
  ```java
  class MyTask implements Runnable {
      public void run() {
          System.out.println("Running in a thread");
      }
  }
  Thread t = new Thread(new MyTask());
  t.start(); // Starts a new thread
  ```
- You can also use lambda:
  ```java
  Thread t = new Thread(() -> System.out.println("Hello from thread"));
  t.start();
  ```

### 2. ExecutorService and Thread Pools
- **ExecutorService:** Manages a pool of threads for you. Better than creating threads manually.
  ```java
  import java.util.concurrent.*;
  ExecutorService executor = Executors.newFixedThreadPool(2);
  executor.submit(() -> System.out.println("Task 1"));
  executor.submit(() -> System.out.println("Task 2"));
  executor.shutdown();
  ```
- **Future:** Represents a result that will be available later.
  ```java
  Future<Integer> future = executor.submit(() -> 42);
  int result = future.get(); // Waits for result
  ```

### 3. CompletableFuture (Async Programming)
- For chaining tasks and non-blocking code.
  ```java
  import java.util.concurrent.CompletableFuture;
  CompletableFuture.supplyAsync(() -> {
      // long computation
      return 10;
  }).thenApply(x -> x * 2)
    .thenAccept(result -> System.out.println("Result: " + result));
  ```
- Lets you run code when previous steps finish, without blocking main thread.

### 4. Synchronization and Memory Sharing

When multiple threads access shared data, you must protect it to avoid **race conditions** (bugs from two threads changing data at the same time).

#### a. synchronized keyword
- Locks a method or block so only one thread can run it at a time.
  ```java
  public synchronized void increment() {
      count++;
  }
  // Or lock a block:
  synchronized(this) {
      // critical section
  }
  ```

#### b. volatile keyword
- Tells Java that a variable may be changed by multiple threads. Ensures all threads see the latest value.
  ```java
  private volatile boolean running = true;
  ```
- Use for simple flags, not for compound actions (like count++).

#### c. Locks and Atomic Classes
- **Lock:** More flexible than synchronized.
  ```java
  import java.util.concurrent.locks.*;
  Lock lock = new ReentrantLock();
  lock.lock();
  try {
      // critical section
  } finally {
      lock.unlock();
  }
  ```
- **Atomic classes:** For thread-safe counters and variables.
  ```java
  import java.util.concurrent.atomic.AtomicInteger;
  AtomicInteger count = new AtomicInteger(0);
  count.incrementAndGet(); // Safe increment
  ```

### 5. Best Practices for Safe Concurrency
- Minimize shared data between threads.
- Always protect shared data with synchronized, locks, or atomic classes.
- Prefer higher-level tools (ExecutorService, CompletableFuture) over manual threads.
- Avoid holding locks for long periods.
- Use thread-safe collections (e.g., `ConcurrentHashMap`).
- Test for race conditions and deadlocks.

**Common Pitfall:** Forgetting to synchronize access to shared data can cause hard-to-find bugs.

## Exception Handling

- **Try-catch:**
  ```java
  try {
      int x = 5 / 0;
  } catch (ArithmeticException e) {
      System.out.println("Error: " + e.getMessage());
  }
  ```
- **Finally:** Runs always.
  ```java
  finally {
      System.out.println("Cleanup");
  }
  ```
- **Throw:**
  ```java
  throw new IllegalArgumentException("Bad arg");
  ```

## Build Tools

- **javac:** Compile Java code.
  ```bash
  javac Main.java
  java Main
  ```
- **Maven/Gradle:** For larger projects, use these tools to manage dependencies and build process.

## Best Practices and Tips

- Use meaningful variable and method names.
- Keep classes focused (Single Responsibility Principle).
- Use interfaces for abstraction.
- Prefer composition over inheritance when possible.
- Always close resources (files, streams) in a `finally` block or use try-with-resources.
- Write unit tests for your code.
- Use JavaDocs for documentation:
  ```java
  /**
   * This is a sample method.
   * @param x input value
   * @return squared value
   */
  public int square(int x) {
      return x * x;
  }
  ```

## Common Pitfalls

- NullPointerException: Always check for null before using objects.
- ArrayIndexOutOfBoundsException: Check array bounds.
- Integer division: `5/2` is `2`, not `2.5`. Use `5.0/2` for floating point.

## Further Reading

- [Official Java Tutorials](https://docs.oracle.com/javase/tutorial/)
- [Effective Java by Joshua Bloch] 