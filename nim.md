# Nim Programming Language

## Basics
1. Nim compiles to C, C++, or JavaScript. It's statically typed but has type inference so you don't always need to specify types.
2. Indentation matters like Python. No curly braces needed.
3. Variables are declared with `var` (mutable) or `let` (immutable) or `const` (compile-time constant).
   ```nim
   var name = "John"        # mutable
   let age = 25            # immutable at runtime
   const PI = 3.14159      # compile-time constant
   ```
4. Functions are declared with `proc`. The last expression is automatically returned.
   ```nim
   proc add(x, y: int): int =
     x + y  # automatically returned
   ```
5. No semicolons needed unless you put multiple statements on same line.
6. Comments start with `#` for single line or `#[` `]#` for multi-line.
7. Nim is case and underscore insensitive. So `myVar`, `my_var`, and `myvar` are all the same identifier.

## Types and Variables
1. Basic types: `int`, `float`, `string`, `char`, `bool`
2. You can specify bit size: `int8`, `int16`, `int32`, `int64`, `uint8`, etc.
3. Arrays have fixed size: `array[5, int]` means array of 5 integers.
4. Sequences are dynamic arrays: `seq[int]` is like a list in Python.
   ```nim
   var numbers: seq[int] = @[1, 2, 3, 4]  # @ creates a sequence
   numbers.add(5)  # append to sequence
   ```
5. Strings are mutable by default unlike many languages.
6. Use `$` operator to convert anything to string (like `str()` in Python).
   ```nim
   let num = 42
   echo "Number is: " & $num  # & concatenates strings
   ```

## Control Flow
1. `if` statements don't need parentheses around conditions:
   ```nim
   if age >= 18:
     echo "Adult"
   elif age >= 13:
     echo "Teenager"
   else:
     echo "Child"
   ```
2. `case` statements are like switch but more powerful:
   ```nim
   case grade
   of 'A', 'B':
     echo "Good"
   of 'C':
     echo "Average"
   else:
     echo "Needs improvement"
   ```
3. Loops: `for`, `while`, and `block` with `break`/`continue`
   ```nim
   for i in 1..10:        # range from 1 to 10
     echo i
   
   for item in @[1, 2, 3]:  # iterate over sequence
     echo item
   ```

## Procedures and Functions
1. Use `proc` for procedures (functions). Parameters and return types are optional if they can be inferred.
2. Default parameters work like Python:
   ```nim
   proc greet(name: string, greeting = "Hello"): string =
     greeting & ", " & name & "!"
   ```
3. You can have multiple return values using tuples:
   ```nim
   proc divmod(a, b: int): (int, int) =
     (a div b, a mod b)
   
   let (quotient, remainder) = divmod(10, 3)
   ```
4. `result` is a special variable that holds the return value. You can modify it instead of explicit return.
   ```nim
   proc factorial(n: int): int =
     result = 1
     for i in 1..n:
       result *= i
   ```

## Memory Management
1. Nim has garbage collection by default, but you can disable it for real-time systems.
2. Use `new()` to allocate objects on heap or just declare normally for stack allocation.
3. References are like pointers but garbage collected: `ref int` is a reference to an integer.
4. You can use manual memory management with `alloc()` and `dealloc()` if needed.

## Object-Oriented Programming
1. Objects are declared with `type` and `object`:
   ```nim
   type
     Person = object
       name: string
       age: int
   
   var p = Person(name: "Alice", age: 30)
   ```
2. Methods are procedures with first parameter being the object type:
   ```nim
   proc greet(p: Person): string =
     "Hi, I'm " & p.name
   
   echo p.greet()  # method call syntax
   ```
3. Inheritance uses `object of ParentType`:
   ```nim
   type
     Student = object of Person
       studentId: string
   ```

## Modules and Imports
1. Each `.nim` file is a module. Use `import` to include other modules.
   ```nim
   import strutils, sequtils  # import multiple modules
   import math except sqrt    # import all except sqrt
   from os import sleep       # import only sleep from os
   ```
2. Create your own modules by just creating `.nim` files and importing them.
3. Use `export` to re-export symbols from imported modules.

## Macros and Templates
1. Templates are like C macros but type-safe. They're expanded at compile time.
   ```nim
   template debug(msg: string) =
     when defined(debug):
       echo "[DEBUG] " & msg
   
   debug("This only prints in debug builds")
   ```
2. Macros are more powerful - they can manipulate the AST (Abstract Syntax Tree).
3. Use `when` for compile-time conditionals (like `#ifdef` in C).

## Compilation and Running
1. Install Nim from https://nim-lang.org or use package managers.
2. Compile with: `nim c myfile.nim` (creates executable)
3. Run directly: `nim c -r myfile.nim` (compile and run)
4. For JavaScript: `nim js myfile.nim`
5. For release builds: `nim c -d:release myfile.nim` (optimized, no debug info)
6. Check syntax without compiling: `nim check myfile.nim`

## Package Management
1. Use `nimble` (comes with Nim) for package management.
2. Install packages: `nimble install packagename`
3. Create new project: `nimble init`
4. Dependencies go in `.nimble` file (like `package.json` or `requirements.txt`).

## Nimble: Project Management in Nim

Nimble is Nim's official package manager and project tool. It helps you create, build, and manage Nim projects and their dependencies, similar to npm (JavaScript), pip (Python), or cargo (Rust).

### Creating a New Project

Nimble can create different types of projects for you:

- **Executable (default):**
  ```bash
  nimble init myapp
  # or interactively:
  nimble init
  ```
  This creates a folder `myapp/` with:
  - `myapp.nimble` (project config)
  - `src/myapp.nim` (main entrypoint)
  - `README.md`, `.gitignore`, etc.

- **Library:**
  ```bash
  nimble init mylib --type:lib
  ```
  This is for code you want others to import.

- **Empty project:**
  ```bash
  nimble init myproj --type:none
  ```
  Lets you set up everything yourself.

#### Project Structure Example
```
myapp/
├── myapp.nimble      # Project config (like package.json)
├── src/
│   └── myapp.nim     # Main entrypoint (for executables)
├── README.md
└── ...
```

### Entrypoint: Where Does My Code Start?
- For executables: The entrypoint is the `.nim` file named after your project in `src/` (e.g., `src/myapp.nim`).
- For libraries: You provide modules in `src/` that others can import.

### Installing and Managing Dependencies
- **Add a dependency:**
  ```bash
  nimble install packagename
  ```
  This downloads and builds the package, and adds it to your project if you run it inside a nimble project folder.

- **List installed packages:**
  ```bash
  nimble list --installed
  ```

- **Upgrade a package:**
  ```bash
  nimble install packagename@latest
  ```

- **Remove a package:**
  Edit your `.nimble` file and remove it from the `requires` section, then run:
  ```bash
  nimble refresh
  ```

### The .nimble File: Comprehensive Guide

The `.nimble` file is the heart of your Nim project configuration. It controls how your project is built, what dependencies it has, and how it is distributed. Here are the most common and useful fields and use cases:

#### Basic Fields
```nim
# myproject.nimble
version       = "1.0.0"                # Project version
author        = "Your Name"
description   = "A cool Nim project"
license       = "MIT"

srcDir        = "src"                   # Where your Nim source files live
bin           = @ ["myproject"]         # Executable(s) to build (for apps)
requires      = @ ["strutils >= 0.13.0", "httpbeast"]  # Dependencies
```

#### Library vs Executable
- **Executable:** Use the `bin` field to specify the main file(s) to build as executables.
- **Library:** Omit the `bin` field. Your modules in `src/` can be imported by others.

#### Linking C Libraries
If your project depends on a C library, you can specify linker options:
```nim
# Link to the math library (libm)
# Add this to your .nimble file:

# For all platforms:
passL = "-lm"

# For platform-specific linking:
when defined(windows):
  passL = "-lws2_32"   # Windows Sockets library
when defined(linux):
  passL = "-lpthread"  # POSIX threads
```
- You can also use `passC` to pass flags to the C compiler.

#### Custom Nimble Tasks
You can define custom tasks (like scripts) in your `.nimble` file:
```nim
# In your .nimble file:
task hello, "Prints Hello World":
  echo "Hello from Nimble!"
```
Run with:
```bash
nimble hello
```

#### Other Useful Fields
- `installDirs`: Control where files are installed (for libraries/tools)
- `skipDirs`: Skip certain directories when building
- `skipFiles`: Skip specific files
- `installFiles`: Extra files to install (docs, configs, etc.)
- `beforescripts`/`afterscripts`: Run commands before/after build

#### Example: Advanced .nimble File
```nim
# mytool.nimble
version       = "0.2.0"
author        = "Jane Doe"
description   = "A CLI tool with C library dependency"
license       = "Apache-2.0"

srcDir        = "src"
bin           = @ ["mytool"]
requires      = @ ["cligen >= 1.0.0"]

# Link to zlib (C compression library)
passL         = "-lz"

# Install README and config file
installFiles  = @ ["README.md", "config/default.conf"]

# Custom task
task greet, "Prints a greeting":
  echo "Greetings from mytool!"
```

#### Tips
- You can have multiple `bin` entries for multi-executable projects.
- Use version constraints in `requires` for reproducible builds.
- For more, see the [Nimble documentation](https://github.com/nim-lang/nimble#nimble-packages).

### Where Are Dependencies Stored?
- When you install a dependency, nimble downloads it to a global Nimble package directory (e.g., `~/.nimble/pkgs/`).
- Your project itself does **not** get a `vendor` or `node_modules` folder by default.
- When you build or run your project, Nimble makes sure all dependencies are available on the import path.

### How to See and Access Dependencies
- **See all installed packages:**
  ```bash
  nimble list --installed
  ```
- **See dependencies for your project:**
  Check the `requires` field in your `.nimble` file.
- **Use in code:**
  Just `import` the module:
  ```nim
  import strutils, httpbeast
  ```
  Nimble makes sure these are available when you build or run your project.

### Summary Table
| Task                        | Command or File                |
|-----------------------------|-------------------------------|
| Create new project          | `nimble init`                 |
| Add dependency              | `nimble install packagename`  |
| List installed packages     | `nimble list --installed`     |
| Configure project           | Edit `.nimble` file           |
| Where are dependencies?     | `~/.nimble/pkgs/`             |
| Entrypoint for executable   | `src/<project>.nim`           |

### Tips
- You can edit the `.nimble` file directly to add or pin dependency versions.
- For reproducible builds, use version constraints in `requires`.
- To use a local package, add a path in the `requires` field: `requires = @ ["../mypkg"]`
- Nimble also supports hooks for custom build steps (see Nimble docs for advanced usage).

### Where Do Nimble Packages Come From?

- **Official Nimble Package Repository:**
  - Most packages are published to the official Nimble package index at https://nimble.directory or https://nim-lang.org/nimble.html.
  - When you run `nimble install packagename`, Nimble fetches the package from this central repository.

- **GitHub and Other Git Repositories:**
  - You can install packages directly from a Git URL:
    ```bash
    nimble install https://github.com/username/repo
    ```
  - You can also specify a branch, tag, or commit:
    ```bash
    nimble install https://github.com/username/repo@branch
    nimble install https://github.com/username/repo@v1.2.3
    nimble install https://github.com/username/repo@commit_hash
    ```

- **Local Packages:**
  - You can install a package from a local folder:
    ```bash
    nimble install /path/to/local/package
    ```

- **Searching for Packages:**
  - Browse or search for packages at https://nimble.directory
  - Or use the command line:
    ```bash
    nimble search keyword
    ```

- **Summary Table:**
| Source                | How to install                                      |
|-----------------------|-----------------------------------------------------|
| Official repository   | `nimble install packagename`                        |
| GitHub repo           | `nimble install https://github.com/user/repo`       |
| Specific branch/tag   | `nimble install https://github.com/user/repo@tag`   |
| Local folder          | `nimble install /path/to/package`                   |
| Search for packages   | `nimble search keyword` or visit nimble.directory   |

## Useful Libraries
1. `strutils` - string manipulation functions
2. `sequtils` - sequence/array utilities  
3. `json` - JSON parsing and generation
4. `httpclient` - HTTP requests
5. `asyncdispatch` - async/await support
6. `unittest` - testing framework

## Tips and Gotchas
1. Nim is whitespace-sensitive like Python, but you can use `\` for line continuation.
2. Use `discard` to ignore return values: `discard someFunction()`
3. `echo` is the print function. It automatically adds newline.
4. String interpolation uses `&` for concatenation or use `fmt` macro: `fmt"Hello {name}"`
5. Nim has no `null` - use `Option[T]` type for nullable values.
6. The compiler is very fast. Full rebuilds are usually under a second.
7. Use `--hints:off` to reduce compiler output noise.
8. Nim can interface with C libraries easily - just write the function signatures.
9. For performance-critical code, compile with `-d:danger` (removes all checks).
10. Use `nim doc myfile.nim` to generate HTML documentation from your code comments.

## Why Choose Nim?
1. **Performance**: Compiles to C, so it's as fast as C/C++.
2. **Syntax**: Clean and readable like Python.
3. **Memory safe**: Garbage collected by default, but you can go manual.
4. **Small binaries**: No runtime dependencies needed.
5. **Metaprogramming**: Powerful macro system for code generation.
6. **Interop**: Easy to use C libraries and be called from other languages.

## Interop with C and Python

### C Interop
1. Nim can call C functions directly. Just declare the function signature with `{.importc.}` pragma:
   ```nim
   proc printf(format: cstring): cint {.importc, varargs, header: "<stdio.h>".}
   printf("Hello from C: %d\n", 42)
   ```
2. Use `cstring` for C strings, `cint` for C integers, `cdouble` for C doubles, etc.
3. For C libraries, create a wrapper file or use existing ones from nimble:
   ```nim
   # Calling malloc and free from C
   proc malloc(size: csize_t): pointer {.importc, header: "<stdlib.h>".}
   proc free(p: pointer) {.importc, header: "<stdlib.h>".}
   
   let ptr = malloc(100)  # allocate 100 bytes
   # use ptr...
   free(ptr)  # don't forget to free!
   ```
4. To expose Nim functions to C, use `{.exportc.}` pragma:
   ```nim
   proc addNumbers(a, b: cint): cint {.exportc.} =
     a + b
   ```
   Then compile with: `nim c --app:staticlib mylib.nim` to create a `.a` file.
5. For structs, map them to Nim objects:
   ```nim
   type
     CPoint {.importc: "Point", header: "myheader.h".} = object
       x, y: cint
   ```

### Python Interop
1. Use `nimpy` library for Python interop. Install with: `nimble install nimpy`
2. Call Python from Nim:
   ```nim
   import nimpy
   
   let py = pyBuiltinsModule()
   let pyList = py.list([1, 2, 3, 4])
   echo pyList.len()  # prints 4
   
   # Import Python modules
   let math = pyImport("math")
   echo math.sqrt(16)  # prints 4.0
   ```
3. Convert between Nim and Python types automatically:
   ```nim
   import nimpy
   
   let pyDict = {"name": "Alice", "age": 30}.toPy()
   let nimSeq = @[1, 2, 3, 4].toPy().to(seq[int])
   ```
4. Create Python modules in Nim using `{.exportpy.}` pragma:
   ```nim
   import nimpy
   
   proc fibonacci(n: int): int {.exportpy.} =
     if n <= 1: return n
     return fibonacci(n-1) + fibonacci(n-2)
   
   # Compile with: nim c --app:lib --out:mymodule.so myfile.nim
   # Then in Python: import mymodule; print(mymodule.fibonacci(10))
   ```
5. Handle Python exceptions in Nim:
   ```nim
   import nimpy
   
   try:
     let result = pyImport("math").sqrt(-1)
   except PyException as e:
     echo "Python error: ", e.msg
   ```
6. Use Python's numpy arrays directly:
   ```nim
   import nimpy
   
   let np = pyImport("numpy")
   let arr = np.array([1, 2, 3, 4, 5])
   echo arr.sum()  # calls numpy's sum function
   ```

### Practical Tips for Interop
1. **C Libraries**: Most C libraries have Nim wrappers available on nimble. Search before writing your own.
2. **Python Performance**: Use Nim for performance-critical parts, Python for everything else.
3. **Memory Management**: Be careful with C pointers - Nim's GC won't manage them.
4. **String Conversion**: Use `$` to convert cstring to Nim string, `cstring()` for the reverse.
5. **Debugging**: Use `--debugger:native` when debugging C interop issues.
6. **Build Systems**: For complex C projects, you might need to write custom `.nims` config files.

## Common Patterns
```nim
# Error handling with try/except
try:
  let file = open("data.txt")
  # do something with file
except IOError:
  echo "File not found"
finally:
  file.close()

# Async programming
import asyncdispatch

proc fetchData(): Future[string] {.async.} =
  await sleepAsync(1000)  # simulate network delay
  return "data"

# Iterator (like Python generators)
iterator countdown(n: int): int =
  var i = n
  while i > 0:
    yield i
    dec i

for num in countdown(5):
  echo num

# C interop example
proc strlen(s: cstring): csize_t {.importc, header: "<string.h>".}
let cStr: cstring = "Hello"
echo "Length: ", strlen(cStr)

# Python interop example
import nimpy
let sys = pyImport("sys")
echo "Python version: ", sys.version
```

Remember: Nim's philosophy is "efficiency, expressiveness, elegance". It tries to be as fast as C while being as pleasant to write as Python. The learning curve is gentle if you know Python, but you get the performance benefits of a systems language. 


## 2025 Additions

1. **Improved Memory Model**: Nim 2.0 introduced a new memory model called "ARC+" that combines Automatic Reference Counting with cycle collection.
   ```nim
   type Node = ref object
     next: Node  # ARC+ automatically handles circular references
   
   proc example() =
     var head = Node()
     head.next = head  # No memory leak despite circular reference
   ```

2. **Pattern Matching**: Added first-class pattern matching similar to Rust:
   ```nim
   type Shape = object
     case kind: ShapeKind
     of Circle: radius: float
     of Rectangle: width, height: float
   
   let shape = Shape(kind: Circle, radius: 5.0)
   match shape:
     Circle(r): echo "Circle with radius ", r
     Rectangle(w, h): echo "Rectangle ", w, " x ", h
   ```

3. **Enhanced Metaprogramming**: New macro capabilities for zero-cost abstractions:
   ```nim
   macro autoDelegate(T: typedesc, field: untyped): untyped =
     # Automatically generates delegation methods
     # Added in 2025 to reduce boilerplate
   ```

4. **WebAssembly First-Class Support**: Direct WASM compilation target:
   ```nim
   # Compile directly to WASM with new backend
   nim c -d:wasm file.nim
   ```

5. **Improved Error Messages**: More helpful compiler errors with suggestions:
   ```nim
   let x: int = "hello"  # Now suggests: 
   # Error: type mismatch
   # Got: string
   # Expected: int
   # Did you mean to use: parseInt("hello")?
   ```

These features were added to make Nim more robust for large-scale development while maintaining its performance characteristics.

### NimScript and Nimble Tasks: Build Automation

NimScript is a subset of Nim that runs at build time. Nimble uses NimScript to let you automate tasks, customize builds, and write scripts directly in your `.nimble` file.

#### What is NimScript?
- NimScript is Nim code that runs in the Nimble/Nim build environment (not in your final binary).
- It lets you write logic for build steps, custom tasks, or scripting project setup.
- You can use most Nim syntax, but some features (like FFI or OS-specific code) may be limited.

#### Custom Tasks in .nimble
You can define your own tasks in the `.nimble` file. These are like custom commands you can run with Nimble.

**Example: Simple Custom Task**
```nim
# In your .nimble file:
task hello, "Prints Hello World":
  echo "Hello from Nimble!"
```
Run it with:
```bash
nimble hello
```

**Example: Task with Arguments**
```nim
task greet, "Greets a user":
  if paramCount() > 0:
    echo "Hello, ", paramStr(1), "!"
  else:
    echo "Hello, world!"
```
Run with:
```bash
nimble greet Alice
```

#### Using NimScript for Build Automation
You can use NimScript to customize the build process, copy files, generate code, or run shell commands.

**Example: Copy a file before build**
```nim
before build:
  echo "Copying config..."
  exec "cp config/default.conf build/"
```

**Example: Generate a file**
```nim
before build:
  let f = open("src/generated.nim", fmWrite)
  f.writeLine("const generatedValue = 42")
  f.close()
```

**Example: Run a shell command after build**
```nim
after build:
  exec "echo Build finished!"
```

#### Advanced NimScript Usage
- You can use `exec` to run shell commands (cross-platform: use `cp`/`mv`/`rm` on Unix, `copy`/`move`/`del` on Windows).
- Use `paramCount()` and `paramStr(i)` to access command-line arguments in tasks.
- You can import NimScript modules (like `os`, `strutils`, etc.) for more complex logic.
- You can define multiple tasks in one `.nimble` file.

**Example: Multiple Tasks**
```nim
task clean, "Removes build artifacts":
  exec "rm -rf build/"

task setup, "Sets up the project":
  exec "mkdir -p build"
  exec "cp config/default.conf build/"
```

#### Calling Other Tasks from a Task

You can call (invoke) other tasks from within a custom Nimble task using the `exec "nimble <task>"` command. This is useful for composing complex workflows from smaller, reusable tasks.

**Example: Chaining Tasks**
```nim
# In your .nimble file:
task clean, "Removes build artifacts":
  exec "rm -rf build/"

task setup, "Sets up the project":
  exec "mkdir -p build"
  exec "cp config/default.conf build/"

task reset, "Cleans and sets up the project":
  exec "nimble clean"
  exec "nimble setup"
```
Now, running:
```bash
nimble reset
```
will first run the `clean` task, then the `setup` task.

**Tips:**
- You can pass arguments to tasks as well: `exec "nimble greet Alice"`
- This works recursively, so you can build complex task chains.
- Each `exec` runs as a separate process, so environment variables may not persist between tasks.

#### More Resources
- [NimScript documentation](https://nim-lang.org/docs/nims.html)
- [Nimble tasks and scripting](https://github.com/nim-lang/nimble#custom-tasks)

