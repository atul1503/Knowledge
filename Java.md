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

## OOP Concepts and Design Patterns in Java

### Core OOP Concepts

1. **Encapsulation**
   - Hiding internal state and requiring all interaction to be performed through methods.
   - Use `private` fields and `public` getters/setters.
   ```java
   public class Account {
       private double balance;
       public double getBalance() { return balance; }
       public void deposit(double amount) { balance += amount; }
   }
   ```

2. **Inheritance**
   - Allows a class to inherit fields and methods from another class.
   - Use `extends` keyword.
   ```java
   class Animal { void speak() { System.out.println("Animal sound"); } }
   class Dog extends Animal { void speak() { System.out.println("Woof"); } }
   ```

3. **Polymorphism**
   - One interface, many implementations. Enables using a superclass reference for subclass objects.
   - Achieved via method overriding and interfaces.
   ```java
   Animal a = new Dog();
   a.speak(); // Prints "Woof"
   ```

4. **Abstraction**
   - Hiding complex implementation details and showing only the necessary features.
   - Use `abstract` classes and interfaces.
   ```java
   abstract class Shape {
       abstract double area();
   }
   class Square extends Shape {
       double side;
       Square(double s) { side = s; }
       double area() { return side * side; }
   }
   ```

5. **Interfaces and Abstract Classes**
   - **Interface:** Only method signatures, no implementation (until Java 8 default methods).
   - **Abstract class:** Can have both abstract and concrete methods.
   ```java
   interface Flyable { void fly(); }
   abstract class Bird implements Flyable {
       public abstract void fly();
       public void eat() { System.out.println("Eating"); }
   }
   ```

6. **Method Overloading and Overriding**
   - **Overloading:** Same method name, different parameters (compile-time polymorphism).
   - **Overriding:** Subclass provides specific implementation for a superclass method (runtime polymorphism).
   ```java
   class MathUtil {
       int add(int a, int b) { return a + b; }
       double add(double a, double b) { return a + b; } // Overloading
   }
   class Parent { void show() { System.out.println("Parent"); } }
   class Child extends Parent { void show() { System.out.println("Child"); } } // Overriding
   ```

7. **Other OOP Features in Java**
   - **final:** Prevents inheritance or modification.
     ```java
     final class Constants { static final double PI = 3.14; }
     ```
   - **static:** Belongs to the class, not instances.
     ```java
     class Counter { static int count = 0; }
     ```
   - **Inner Classes:** Classes defined within another class.
     ```java
     class Outer {
         class Inner { void hello() { System.out.println("Hi"); } }
     }
     ```
   - **Anonymous Classes:** Inline implementation of interfaces or abstract classes.
     ```java
     Runnable r = new Runnable() { public void run() { System.out.println("Run"); } };
     ```
   - **Default and Static Methods in Interfaces (Java 8+):**
     ```java
     interface MyInterface {
         default void hello() { System.out.println("Hello"); }
         static void hi() { System.out.println("Hi"); }
     }
     ```

### Common Design Patterns in Java

1. **Singleton Pattern**
   - Ensures only one instance of a class exists.
   ```java
   class Singleton {
       private static Singleton instance;
       private Singleton() {}
       public static Singleton getInstance() {
           if (instance == null) instance = new Singleton();
           return instance;
       }
   }
   ```

2. **Factory Pattern**
   - Creates objects without specifying the exact class.
   ```java
   interface Animal { void speak(); }
   class Dog implements Animal { public void speak() { System.out.println("Woof"); } }
   class Cat implements Animal { public void speak() { System.out.println("Meow"); } }
   class AnimalFactory {
       public static Animal getAnimal(String type) {
           if (type.equals("dog")) return new Dog();
           else return new Cat();
       }
   }
   // Usage: Animal a = AnimalFactory.getAnimal("dog");
   ```

3. **Observer Pattern**
   - One-to-many dependency. When one object changes, all dependents are notified.
   ```java
   import java.util.*;
   interface Observer { void update(String msg); }
   class Subject {
       private List<Observer> observers = new ArrayList<>();
       public void addObserver(Observer o) { observers.add(o); }
       public void notifyAll(String msg) {
           for (Observer o : observers) o.update(msg);
       }
   }
   class User implements Observer {
       public void update(String msg) { System.out.println("Received: " + msg); }
   }
   // Usage: Subject s = new Subject(); s.addObserver(new User()); s.notifyAll("Hello");
   ```

4. **Strategy Pattern**
   - Selects an algorithm at runtime.
   ```java
   interface SortStrategy { void sort(int[] arr); }
   class BubbleSort implements SortStrategy {
       public void sort(int[] arr) { /* bubble sort */ }
   }
   class QuickSort implements SortStrategy {
       public void sort(int[] arr) { /* quick sort */ }
   }
   class Sorter {
       private SortStrategy strategy;
       public Sorter(SortStrategy s) { strategy = s; }
       public void sort(int[] arr) { strategy.sort(arr); }
   }
   // Usage: Sorter s = new Sorter(new QuickSort()); s.sort(arr);
   ```

5. **Decorator Pattern**
   - Adds new behavior to objects dynamically.
   ```java
   interface Coffee { String getDescription(); }
   class SimpleCoffee implements Coffee { public String getDescription() { return "Simple Coffee"; } }
   class MilkDecorator implements Coffee {
       private Coffee coffee;
       public MilkDecorator(Coffee c) { coffee = c; }
       public String getDescription() { return coffee.getDescription() + ", Milk"; }
   }
   // Usage: Coffee c = new MilkDecorator(new SimpleCoffee());
   ```

6. **Adapter Pattern**
   - Allows incompatible interfaces to work together.
   ```java
   interface Target { void request(); }
   class Adaptee { void specificRequest() { System.out.println("Specific"); } }
   class Adapter implements Target {
       private Adaptee adaptee;
       public Adapter(Adaptee a) { adaptee = a; }
       public void request() { adaptee.specificRequest(); }
   }
   // Usage: Target t = new Adapter(new Adaptee()); t.request();
   ```

7. **Command Pattern**
   - Encapsulates a request as an object.
   ```java
   interface Command { void execute(); }
   class LightOnCommand implements Command {
       public void execute() { System.out.println("Light On"); }
   }
   class Remote {
       private Command command;
       public void setCommand(Command c) { command = c; }
       public void pressButton() { command.execute(); }
   }
   // Usage: Remote r = new Remote(); r.setCommand(new LightOnCommand()); r.pressButton();
   ```

8. **Other Patterns**
   - **Builder:** For complex object construction.
   - **Prototype:** Cloning objects.
   - **Facade:** Simplifies complex subsystems.
   - **Proxy:** Controls access to another object.

**Tip:** Use design patterns to solve common problems in a reusable way. Don't overuse them—choose patterns that fit your problem.

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