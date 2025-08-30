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
7. Nim is case and underscore insensitive. So `myVar`, `my_var`, `myVar`, and `myvar` are all the same identifier. This means you can use any casing/underscore style consistently in your code, and it will work with libraries that use different styles.

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
     result = 1  # result is automatically initialized to the default value of the return type
     for i in 1..n:
       result *= i
   # The value of result is automatically returned at the end of the procedure
   ```
   
   The `result` variable is automatically created for any procedure that has a return type. It's initialized to the default value of that type (0 for int, "" for string, nil for ref types, etc.).

5. Nested procedures can access variables from their enclosing scope (closures):
   ```nim
   proc createCounter(): proc(): int =
     var count = 0
     proc increment(): int =
       count += 1  # accesses 'count' from outer scope
       result = count
     return increment
   
   let counter = createCounter()
   echo counter()  # prints 1
   echo counter()  # prints 2
   ```
   Nested procedures can access and modify variables from their parent procedure, creating closures that "capture" the outer scope.

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
3. Use `method` instead of `proc` for dynamic dispatch (virtual methods):
   ```nim
   type
     Shape = ref object of RootObj
     Circle = ref object of Shape
       radius: float
   
   method area(s: Shape): float {.base.} =
     # Base method that can be overridden
     0.0
   
   method area(c: Circle): float =
     # Overrides the base method
     3.14159 * c.radius * c.radius
   
   # Dynamic dispatch - calls the correct method based on actual type
   let shapes: seq[Shape] = @[Circle(radius: 5.0)]
   echo shapes[0].area()  # Calls Circle's area method
   ```
   The `{.base.}` pragma marks a method as the base method that can be overridden. Methods without this pragma override existing methods.
4. Inheritance uses `object of ParentType`:
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

## Pragmas (Compiler Directives)

Pragmas are special annotations that give instructions to the compiler about how to handle specific code. They are written in curly braces with dots: `{.pragmaname.}`.

1. **Common pragmas for procedures:**
   ```nim
   proc fastFunction() {.inline.} =
     # This procedure will be inlined for better performance
     echo "Fast!"
   
   proc noReturn() {.noreturn.} =
     # This procedure never returns (exits or raises exception)
     quit("Program terminated")
   
   proc deprecated() {.deprecated: "Use newFunction instead".} =
     # Compiler will warn when this is used
     echo "Old function"
   ```

2. **Pragmas for variables and types:**
   ```nim
   var globalVar {.threadvar.}: int  # Thread-local variable
   
   type
     PackedStruct {.packed.} = object
       # No padding between fields
       flag: bool
       value: int16
   
   var uninitializedArray: array[1000, int] {.noinit.}  # Don't zero-initialize
   ```

3. **Pragmas for imports and exports:**
   ```nim
   proc cFunction(): cint {.importc: "c_function", header: "myheader.h".}
   proc nimFunction(): int {.exportc.}  # Export to C
   ```

4. **Custom pragmas for documentation or framework integration:**
   ```nim
   proc button() {.expand: false.} =
     # Custom pragma used by UI frameworks
     # The meaning depends on the framework processing it
     discard
   ```

5. **Multiple pragmas can be combined:**
   ```nim
   proc optimizedFunction() {.inline, noSideEffect.} =
     # Both inline and marked as having no side effects
     result = 42
   ```

**Common pragma uses:**
- `{.inline.}` - Inline small functions for performance
- `{.noSideEffect.}` - Mark pure functions (no global state changes)
- `{.raises: [].}` - Specify which exceptions a procedure can raise
- `{.deprecated.}` - Mark old code that should not be used
- `{.base.}` - Mark methods that can be overridden
- Custom pragmas for domain-specific languages and frameworks

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
  This downloads and builds the package globally to `~/.nimble/pkgs/`. 
  
  **Important Note:** `nimble install` does NOT automatically update your project's `.nimble` file. The `requires` section in your `.nimble` file specifies what packages your project needs. If you want to add a new dependency to your project, you must manually edit your `.nimble` file:
  
  ```nim
  # In your myproject.nimble file:
  requires "packagename >= 1.0.0"
  ```
  
  **Installing from GitHub repositories:** You can also specify dependencies directly from GitHub repositories in your `.nimble` file:
  
  ```nim
  # In your myproject.nimble file:
  requires "https://github.com/can-lehmann/owlkettle"
  ```
  
  This line tells Nimble that your project needs the `owlkettle` package, and it should download it directly from the GitHub repository at `https://github.com/can-lehmann/owlkettle`. The `requires` keyword is used in `.nimble` files to declare what packages your project depends on. When someone runs `nimble install` in your project directory, Nimble will automatically download and install all the packages listed in the `requires` section.
  
  Then run `nimble install` (without package name) to install all dependencies listed in your `.nimble` file. This ensures your project's dependencies are properly tracked and can be installed by others who use your project.

#### When to Install Packages Separately vs. Adding to .nimble File

**Install separately with `nimble install packagename` when:**

1. **Global development tools** - Tools you use for development across multiple projects:
   ```bash
   nimble install nimlsp     # Language server for IDE support
   nimble install nimfmt     # Code formatter
   nimble install testament  # Testing tool
   ```
   These are like global utilities - you want them available everywhere but they're not part of any specific project.

2. **Experimenting or testing** - When you want to try out a package before deciding if it belongs in your project:
   ```bash
   nimble install somepackage  # Just testing it out
   # Later, if you decide to use it in your project, add it to .nimble file
   ```

3. **Command-line tools** - Packages that provide executable programs:
   ```bash
   nimble install nitch      # System information tool
   nimble install choosenim  # Nim version manager
   ```
   These are standalone programs, not libraries for your code.

**Add to .nimble file when:**

1. **Your project code imports the package** - If your Nim source code has `import packagename`, then the package must be listed in `requires`:
   ```nim
   # If your code has this:
   import jester, json, strutils
   
   # Then your .nimble file needs:
   requires "jester >= 0.5.0"
   ```

2. **Others need to build your project** - Anyone who downloads your project should be able to run `nimble install` and get all needed dependencies automatically.

3. **Reproducible builds** - Ensures everyone gets the same package versions when building your project.

**Summary:** Think of `nimble install packagename` as installing global tools, and `.nimble` file dependencies as ingredients needed to cook your specific project recipe.

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
- When you install a dependency, nimble downloads it to a global Nimble package directory (e.g., `~/.nimble/pkgs/` or `~/.nimble/pkgs2/`).
- Your project itself does **not** get a `vendor` or `node_modules` folder by default.
- When you build or run your project, Nimble makes sure all dependencies are available on the import path.

#### What's Actually Stored in the Package Directory?

**Source Code, Not Binaries:** Nimble stores the complete source code of packages, not compiled binaries. When you install a package, you get:

- **Full Nim source files** (`.nim` files) - The actual code you can read and inspect
- **Package configuration** (`.nimble` files) - The package's own dependency and build settings
- **Documentation and examples** - Any README files, docs, or example code included by the package author
- **All package subdirectories** - Complete directory structure as created by the package author

**Example of what you'll find:**
```
~/.nimble/pkgs2/
├── jester-0.5.0/           # Web framework package
│   ├── jester.nim          # Main module source code
│   ├── jester.nimble       # Package configuration
│   ├── src/                # Source directory
│   │   ├── jester/         # Package modules
│   │   │   ├── core.nim    # Core functionality
│   │   │   └── utils.nim   # Utility functions
│   ├── examples/           # Example programs
│   └── README.md           # Documentation
└── chronicles-0.10.3/      # Logging package
    ├── chronicles.nim      # Main module source code
    ├── chronicles.nimble   # Package configuration
    └── chronicles/         # Package implementation
        ├── log.nim         # Logging implementation
        └── formatters.nim  # Log formatting code
```

**Executables:** If a package provides command-line tools, Nimble compiles those and puts the resulting binary executables in `~/.nimble/bin/`. This directory is usually added to your system's PATH so you can run these tools from anywhere.

**Why Source Code?** Storing source code instead of binaries means:
- You can inspect and understand what the package does
- Nim can optimize the code when compiling your project
- The package works across different architectures and operating systems
- You can modify package code if needed (though this isn't recommended)

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

### Default Nimble Tasks (Built-in Commands)

Nimble provides several built-in tasks that are available in every project, even if you don't define any custom tasks in your `.nimble` file. These commands help you manage your project without any additional configuration:

**Project Build and Management:**
- **`nimble build`** - Compiles your project based on the configuration in your `.nimble` file. Creates an executable from the `bin` field or compiles all modules.
- **`nimble run`** - Builds and immediately runs your project. Equivalent to `nimble build` followed by running the executable. You can pass arguments to your program using `nimble run -- arg1 arg2`.
- **`nimble install`** - Installs your project system-wide, making it available as a command or library for other projects.
- **`nimble test`** - Runs the project's tests. Looks for and executes test files (usually in a `tests/` directory).
- **`nimble clean`** - Removes build artifacts and temporary files created during compilation.

**Package Management:**
- **`nimble install <packagename>`** - Installs a package from the Nimble repository or other sources (GitHub, local path).
- **`nimble uninstall <packagename>`** - Removes an installed package from your system.
- **`nimble list`** - Shows all available packages in the Nimble repository.
- **`nimble list --installed`** - Lists all packages currently installed on your system.
- **`nimble search <keyword>`** - Searches for packages in the Nimble repository that match the keyword.
- **`nimble refresh`** - Updates the local package list cache from the Nimble repository.

**Project Creation and Information:**
- **`nimble init`** - Creates a new Nim project with basic structure and `.nimble` file.
- **`nimble dump`** - Shows detailed information about your project's configuration and dependencies.
- **`nimble tasks`** - Lists all custom tasks defined in your project's `.nimble` file.

**Publishing and Distribution:**
- **`nimble publish`** - Publishes your project as a new package to the Nimble directory (requires account setup).
- **`nimble check`** - Validates your project's `.nimble` file for correctness before publishing.

**Help and Information:**
- **`nimble --help`** - Shows general help and list of all available commands.
- **`nimble --help <command>`** - Shows detailed help for a specific command (e.g., `nimble --help build`).
- **`nimble --version`** - Displays the version of Nimble currently installed.

**Example Usage:**
```bash
# Build your project
nimble build

# Build and run your project
nimble run

# Run your project with arguments
nimble run -- --verbose input.txt

# Run tests
nimble test

# Install a dependency
nimble install jester

# See what custom tasks are available
nimble tasks

# Get help on a specific command
nimble --help install

# Clean build artifacts
nimble clean
```

**Important Notes:**
- These tasks work automatically based on your `.nimble` file configuration
- You don't need to define these tasks yourself - they're built into Nimble
- Some tasks (like `test` and `build`) require proper project structure and configuration
- You can override these default behaviors by defining custom tasks with the same names in your `.nimble` file

### Summary Table
| Task                        | Command or File                |
|-----------------------------|-------------------------------|
| Create new project          | `nimble init`                 |
| Install package globally    | `nimble install packagename`  |
| Add dependency to project   | Edit `.nimble` file manually   |
| Install project dependencies| `nimble install` (no package name) |
| List installed packages     | `nimble list --installed`     |
| Configure project           | Edit `.nimble` file           |
| Where are dependencies?     | `~/.nimble/pkgs/`             |
| Entrypoint for executable   | `src/<project>.nim`           |

### Tips
- You can edit the `.nimble` file directly to add or pin dependency versions.
- For reproducible builds, use version constraints in `requires`.
- To use a local package, add a path in the `requires` field: `requires "../mypkg"`
- Nimble also supports hooks for custom build steps (see Nimble docs for advanced usage).

### Where Do Nimble Packages Come From?

- **Official Nimble Package Repository:**
  - Most packages are published to the official Nimble package index at https://nimble.directory or https://nim-lang.org/nimble.html.
  - When you run `nimble install packagename`, Nimble fetches the package from this central repository.

- **GitHub and Other Git Repositories:**
  - You can specify packages directly from GitHub repositories in your project's `.nimble` file:
    ```nim
    # In your .nimble file:
    requires "https://github.com/username/repo"
    ```
  - You can also specify a specific branch, tag, or commit by adding it after a # symbol:
    ```nim
    # In your .nimble file:
    requires "https://github.com/username/repo#branch"
    requires "https://github.com/username/repo#v1.2.3"
    requires "https://github.com/username/repo#commit_hash"
    ```

- **Local Packages:**
  - You can specify a local package in your `.nimble` file:
    ```nim
    # In your .nimble file:
    requires "../path/to/local/package"
    ```

- **Searching for Packages:**
  - Browse or search for packages at https://nimble.directory
  - Or use the command line:
    ```bash
    nimble search keyword
    ```

- **Summary Table:**
| Source                | How to specify in .nimble file                      |
|-----------------------|-----------------------------------------------------|
| Official repository   | `requires "packagename >= 1.0.0"`                  |
| GitHub repo           | `requires "https://github.com/user/repo"`          |
| Specific branch/tag   | `requires "https://github.com/user/repo#tag"`      |
| Local folder          | `requires "../path/to/local/package"`              |
| Search for packages   | `nimble search keyword` or visit nimble.directory   |

## Useful Libraries

Nim's standard library is comprehensive, and there are many third-party packages available through Nimble:

### Standard Library (Built-in)
1. `strutils` - string manipulation functions (split, join, replace, etc.)
2. `sequtils` - sequence/array utilities (map, filter, fold, etc.)
3. `json` - JSON parsing and generation
4. `httpclient` - HTTP requests and responses
5. `asyncdispatch` - async/await support for non-blocking I/O
6. `unittest` - testing framework for writing unit tests
7. `os` - operating system interface (file paths, environment variables)
8. `tables` - hash tables and dictionaries
9. `sets` - hash sets for unique collections
10. `times` - date and time handling
11. `re` - regular expressions
12. `math` - mathematical functions
13. `random` - random number generation
14. `logging` - logging facilities
15. `uri` - URL parsing and manipulation

### Popular Third-Party Libraries (install with `nimble install`)

**Web Development:**
- `jester` - lightweight web framework (like Flask)
- `karax` - single-page application framework
- `prologue` - full-featured web framework (like Django)

**Database:**
- `db_sqlite` - SQLite database support (standard library)
- `db_postgres` - PostgreSQL support (standard library)
- `redis` - Redis client
- `norm` - ORM (Object-Relational Mapping)

**Networking:**
- `websocket` - WebSocket client and server
- `smtp` - email sending
- `net` - TCP/UDP networking (standard library)

**Parsing and Serialization:**
- `yaml` - YAML parsing
- `xmlparser` - XML parsing (standard library)
- `parsecfg` - configuration file parsing (standard library)

**GUI (Most Stable Options):**
- `nigui` - cross-platform native GUI toolkit (Windows, Linux, macOS). Simple API, good for desktop apps
- `gintro` - high-level GTK3/GTK4 bindings (Linux/Windows/macOS). Very mature, comprehensive widget set
- `wnim` - Windows-only GUI framework with native Windows look and feel. Stable for Windows desktop apps
- `nimqml` - Qt QML bindings for modern, dynamic interfaces. Good for cross-platform apps with Qt
- `nimx` - cross-platform GUI framework (desktop, mobile, web). Lightweight but less mature than others

**Graphics and Games (Most Stable Options):**
- `opengl` - OpenGL bindings for 3D graphics. Very stable, widely used
- `sdl2` - SDL2 bindings for games and multimedia. Battle-tested, excellent for 2D/3D games
- `raylib` - Raylib game development library. Simple API, great for beginners and prototypes
- `glfw` - GLFW bindings for OpenGL window management. Stable foundation for OpenGL apps
- `cairo` - Cairo graphics library bindings for 2D vector graphics. Very stable for drawing and graphics

**Utilities:**
- `cligen` - command-line argument parsing
- `chronicles` - structured logging
- `nimpy` - Python interop
- `regex` - advanced regular expressions

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

## Error Handling Patterns

Nim provides several ways to handle errors. Here are the most common patterns developers use:

### 1. Exceptions (Traditional)
```nim
proc readFile(filename: string): string =
  if not fileExists(filename):
    raise newException(IOError, "File not found: " & filename)
  # read file content here
  result = readFile(filename)

try:
  let content = readFile("data.txt")
  echo content
except IOError as e:
  echo "Error: ", e.msg
except Exception as e:
  echo "Unexpected error: ", e.msg
```

### 2. Result Types (Modern, Preferred)
```nim
import std/options

type
  Result[T, E] = object
    case isOk: bool
    of true: value: T
    of false: error: E

proc parseInt(s: string): Result[int, string] =
  try:
    Result[int, string](isOk: true, value: parseInt(s))
  except ValueError:
    Result[int, string](isOk: false, error: "Invalid integer: " & s)

# Usage
let parseResult = parseInt("42")
if parseResult.isOk:
  echo "Number: ", parseResult.value
else:
  echo "Error: ", parseResult.error
```

### 3. Option Types for Nullable Values
```nim
import std/options

proc findUser(id: int): Option[string] =
  if id == 1:
    some("Alice")  # some() wraps a value in an Option
  else:
    none(string)   # none() represents "no value"

# Usage
let user = findUser(1)
if user.isSome:
  echo "Found user: ", user.get()
else:
  echo "User not found"

# Or use pattern matching
case user:
of Some(name): echo "Hello, ", name
of None: echo "No user found"
```

### 4. Custom Error Types
```nim
type
  AppError = object of CatchableError
  NetworkError = object of AppError
  DatabaseError = object of AppError

proc fetchData(): string =
  # This will raise a NetworkError
  raise newException(NetworkError, "Connection failed")

try:
  let data = fetchData()
except NetworkError as e:
  echo "Network issue: ", e.msg
except DatabaseError as e:
  echo "Database issue: ", e.msg
except AppError as e:
  echo "App error: ", e.msg
```

## Generics (Templates for Types)

Generics allow you to write code that works with different types. They're like templates in C++ or generics in Java/C#.

### Basic Generic Procedures
```nim
proc swap[T](a, b: var T) =
  let temp = a
  a = b
  b = temp

var x = 10
var y = 20
swap(x, y)  # T is inferred as int
echo x, " ", y  # prints "20 10"

var s1 = "hello"
var s2 = "world"
swap(s1, s2)  # T is inferred as string
echo s1, " ", s2  # prints "world hello"
```

### Generic Types
```nim
type
  Stack[T] = object
    items: seq[T]

proc push[T](stack: var Stack[T], item: T) =
  stack.items.add(item)

proc pop[T](stack: var Stack[T]): T =
  if stack.items.len > 0:
    result = stack.items[^1]  # ^1 means last element
    stack.items.setLen(stack.items.len - 1)
  else:
    raise newException(IndexDefect, "Stack is empty")

proc isEmpty[T](stack: Stack[T]): bool =
  stack.items.len == 0

# Usage
var intStack = Stack[int]()
intStack.push(1)
intStack.push(2)
echo intStack.pop()  # prints 2

var stringStack = Stack[string]()
stringStack.push("hello")
stringStack.push("world")
echo stringStack.pop()  # prints "world"
```

### Generic Constraints
```nim
# Only allow types that support the + operator
proc add[T: SomeNumber](a, b: T): T =
  a + b

# Only allow types that are ordinal (can be enumerated)
proc next[T: Ordinal](x: T): T =
  succ(x)

# Multiple constraints
proc compare[T: Ordinal and SomeNumber](a, b: T): int =
  if a < b: -1
  elif a > b: 1
  else: 0
```

## Logging

Nim has a built-in logging module that developers commonly use:

```nim
import std/logging

# Create a logger that writes to console
var logger = newConsoleLogger()
addHandler(logger)

# Different log levels
debug("This is debug info")    # Only shows in debug builds
info("Application started")
warn("This is a warning")
error("Something went wrong")
fatal("Critical error!")

# Custom formatting
var fileLogger = newFileLogger("app.log", fmtStr = "$date $time - $levelname: $msg")
addHandler(fileLogger)

# Log with different levels
setLogFilter(lvlInfo)  # Only log info and above
info("This will be logged")
debug("This will be ignored")
```

## Testing Framework

Nim includes a unittest module for writing tests:

```nim
import unittest

# Test suite
suite "Math operations":
  test "addition":
    check(2 + 2 == 4)
    check(5 + 3 == 8)
  
  test "division":
    check(10 div 2 == 5)
    expect(DivByZeroDefect):
      discard 10 div 0  # This should raise an exception
  
  test "floating point":
    check(1.0 + 2.0 == 3.0)
    # For floating point comparisons with tolerance
    check(abs(0.1 + 0.2 - 0.3) < 0.0001)

# Run tests with: nim c -r test_file.nim
```

## Iterators (Custom Loops)

Iterators are like Python generators - they yield values one at a time:

```nim
# Simple iterator
iterator countdown(n: int): int =
  var i = n
  while i > 0:
    yield i
    dec i

for num in countdown(5):
  echo num  # prints 5, 4, 3, 2, 1

# Iterator with multiple values
iterator enumerate[T](s: seq[T]): tuple[index: int, value: T] =
  for i, item in s:
    yield (i, item)

let names = @["Alice", "Bob", "Carol"]
for i, name in enumerate(names):
  echo i, ": ", name
```

## Operator Overloading

You can define custom operators or overload existing ones:

```nim
type
  Vector2 = object
    x, y: float

# Overload the + operator
proc `+`(a, b: Vector2): Vector2 =
  Vector2(x: a.x + b.x, y: a.y + b.y)

# Overload the * operator for scalar multiplication
proc `*`(v: Vector2, scalar: float): Vector2 =
  Vector2(x: v.x * scalar, y: v.y * scalar)

# Define a custom operator
proc `**`(base: float, exponent: float): float =
  # Custom power operator
  pow(base, exponent)

# Usage
let v1 = Vector2(x: 1.0, y: 2.0)
let v2 = Vector2(x: 3.0, y: 4.0)
let v3 = v1 + v2  # Vector2(x: 4.0, y: 6.0)
let v4 = v1 * 2.0  # Vector2(x: 2.0, y: 4.0)
let result = 2.0 ** 3.0  # 8.0
```

## Async/Await Programming

Nim has excellent support for asynchronous programming, similar to Python's asyncio or JavaScript's async/await:

```nim
import asyncdispatch, httpclient

# Basic async procedure
proc fetchData(url: string): Future[string] {.async.} =
  let client = newAsyncHttpClient()
  try:
    let response = await client.get(url)
    result = await response.body
  finally:
    client.close()

# Async procedure that uses other async procedures
proc processUrls(urls: seq[string]): Future[seq[string]] {.async.} =
  result = @[]
  for url in urls:
    let data = await fetchData(url)
    result.add(data)

# Run async code
proc main() {.async.} =
  let urls = @["http://example.com", "http://google.com"]
  let results = await processUrls(urls)
  for i, result in results:
    echo "URL ", i, ": ", result.len, " characters"

# Start the async event loop
waitFor main()
```

### Async Utilities
```nim
import asyncdispatch, times

# Sleep without blocking
proc delayedHello(): Future[void] {.async.} =
  await sleepAsync(1000)  # Sleep for 1 second
  echo "Hello after delay!"

# Run multiple async operations concurrently
proc concurrent(): Future[void] {.async.} =
  let futures = @[
    delayedHello(),
    delayedHello(),
    delayedHello()
  ]
  await all(futures)  # Wait for all to complete

# Timeout for async operations
proc withTimeout(): Future[void] {.async.} =
  try:
    await sleepAsync(5000).withTimeout(2000)  # 2 second timeout
  except TimeoutError:
    echo "Operation timed out!"
```

## String Formatting and Manipulation

Nim provides several ways to work with strings:

```nim
import strutils, strformat

# Basic string operations
let name = "Alice"
let age = 30

# String interpolation with strformat
let message = fmt"Hello, {name}! You are {age} years old."
echo message

# String interpolation with & operator
let message2 = "Hello, " & name & "! You are " & $age & " years old."

# String formatting with % operator
let message3 = "Hello, $1! You are $2 years old." % [name, $age]

# Multi-line strings
let multiline = """
This is a multi-line string.
It preserves line breaks.
Very useful for templates.
"""

# String manipulation
let text = "  Hello World  "
echo text.strip()           # "Hello World" (remove whitespace)
echo text.toUpper()         # "  HELLO WORLD  "
echo text.replace("World", "Nim")  # "  Hello Nim  "
echo text.split()           # @["Hello", "World"]

# String checking
echo "Hello".startsWith("He")  # true
echo "World".endsWith("ld")    # true
echo "test" in "this is a test"  # true

# String joining
let words = @["Hello", "beautiful", "world"]
echo words.join(" ")  # "Hello beautiful world"
```

## File I/O Operations

```nim
import std/os, std/streams

# Reading files
proc readTextFile(filename: string): string =
  try:
    result = readFile(filename)
  except IOError as e:
    echo "Error reading file: ", e.msg
    result = ""

# Writing files
proc writeTextFile(filename: string, content: string) =
  try:
    writeFile(filename, content)
  except IOError as e:
    echo "Error writing file: ", e.msg

# Line-by-line reading
proc processFileLines(filename: string) =
  try:
    for line in lines(filename):
      echo "Line: ", line
  except IOError as e:
    echo "Error reading file: ", e.msg

# Working with directories
proc listFiles(directory: string) =
  try:
    for kind, path in walkDir(directory):
      case kind:
      of pcFile:
        echo "File: ", path
      of pcDir:
        echo "Directory: ", path
      of pcLinkToFile, pcLinkToDir:
        echo "Link: ", path
  except OSError as e:
    echo "Error accessing directory: ", e.msg

# File streams for large files
proc processLargeFile(filename: string) =
  let fileStream = newFileStream(filename, fmRead)
  if fileStream != nil:
    try:
      var line: string
      while fileStream.readLine(line):
        # Process line without loading entire file into memory
        echo "Processing: ", line
    finally:
      fileStream.close()
```

## JSON Handling

```nim
import json, options

# Creating JSON
let jsonData = %* {
  "name": "Alice",
  "age": 30,
  "hobbies": ["reading", "swimming"],
  "address": {
    "street": "123 Main St",
    "city": "Anytown"
  }
}

echo jsonData.pretty()  # Pretty print JSON

# Parsing JSON from string
let jsonString = """{"name": "Bob", "age": 25}"""
let parsed = parseJson(jsonString)
echo parsed["name"].getStr()  # "Bob"
echo parsed["age"].getInt()   # 25

# Safe JSON access
if parsed.hasKey("email"):
  echo "Email: ", parsed["email"].getStr()
else:
  echo "No email found"

# Converting to Nim objects
type
  Person = object
    name: string
    age: int
    email: Option[string]

proc toPerson(json: JsonNode): Person =
  Person(
    name: json["name"].getStr(),
    age: json["age"].getInt(),
    email: if json.hasKey("email"): some(json["email"].getStr()) else: none(string)
  )

let person = toPerson(parsed)
echo person.name, " is ", person.age, " years old"
```

## Cross-Compilation

Nim makes it easy to compile for different platforms:

```nim
# Compile for Windows (from any platform)
# nim c --cpu:amd64 --os:windows myapp.nim

# Compile for Linux
# nim c --cpu:amd64 --os:linux myapp.nim

# Compile for macOS
# nim c --cpu:amd64 --os:macosx myapp.nim

# Compile for ARM (like Raspberry Pi)
# nim c --cpu:arm --os:linux myapp.nim

# Compile for WebAssembly
# nim c -d:emscripten myapp.nim
```

You can also create conditional compilation:
```nim
when defined(windows):
  proc platformSpecific() =
    echo "Running on Windows"
elif defined(linux):
  proc platformSpecific() =
    echo "Running on Linux"
elif defined(macosx):
  proc platformSpecific() =
    echo "Running on macOS"
else:
  proc platformSpecific() =
    echo "Running on unknown platform"
```

## Performance Optimization Tips

```nim
# 1. Use --opt:speed for optimized builds
# nim c --opt:speed myapp.nim

# 2. Use --gc:arc for deterministic memory management
# nim c --gc:arc myapp.nim

# 3. Use {.inline.} for small, frequently called procedures
proc fastMath(x: float): float {.inline.} =
  x * x + 2.0 * x + 1.0

# 4. Use {.noinit.} to avoid zero-initialization
var bigArray: array[1000000, int] {.noinit.}

# 5. Use views for zero-copy string operations
proc processString(s: openArray[char]) =
  # Process string without copying
  for c in s:
    echo c

# 6. Use compile-time evaluation
const
  fibonacci10 = block:
    var a, b = 1
    for i in 1..8:
      let temp = a + b
      a = b
      b = temp
    b

# 7. Use packed objects for memory efficiency
type
  PackedData {.packed.} = object
    flag: bool
    value: int16
    # No padding between fields

# 8. Profile with --profiler:on
# nim c --profiler:on myapp.nim
```

## Debugging Techniques

```nim
# 1. Use echo for basic debugging
proc debugExample() =
  let x = 42
  echo "Debug: x = ", x
  debugEcho "This goes to stderr"

# 2. Use assert for runtime checks
proc divide(a, b: int): int =
  assert b != 0, "Division by zero"
  a div b

# 3. Use static assert for compile-time checks
static:
  assert sizeof(int) == 8, "This code requires 64-bit integers"

# 4. Use when for conditional compilation
when defined(debug):
  proc debug(msg: string) =
    echo "[DEBUG] ", msg
else:
  proc debug(msg: string) =
    discard  # Do nothing in release builds

# 5. Use --stackTrace:on for better error messages
# nim c --stackTrace:on myapp.nim

# 6. Use --lineTrace:on for line-by-line tracing
# nim c --lineTrace:on myapp.nim

# 7. Use gdb or lldb for native debugging
# nim c --debugger:native myapp.nim
```

## Functional Programming Features

```nim
import sequtils, sugar

# Higher-order functions
let numbers = @[1, 2, 3, 4, 5]

# Map: transform each element
let doubled = numbers.map(x => x * 2)
echo doubled  # @[2, 4, 6, 8, 10]

# Filter: keep elements that match condition
let evens = numbers.filter(x => x mod 2 == 0)
echo evens  # @[2, 4]

# Reduce: combine all elements into one value
let sum = numbers.foldl(a + b)
echo sum  # 15

# Chain operations
let result = numbers
  .filter(x => x > 2)
  .map(x => x * x)
  .foldl(a + b)
echo result  # 50 (3² + 4² + 5² = 9 + 16 + 25)

# Custom higher-order functions
proc applyTwice[T](fn: T -> T, value: T): T =
  fn(fn(value))

let double = (x: int) => x * 2
echo applyTwice(double, 5)  # 20

# Partial application
proc add(x, y: int): int = x + y
let addFive = (y: int) => add(5, y)
echo addFive(10)  # 15

# Currying
proc multiply(x: int): int -> int =
  result = (y: int) => x * y

let multiplyByThree = multiply(3)
echo multiplyByThree(7)  # 21
```

## Common Data Structures

```nim
# Hash tables (dictionaries)
import tables

var userAges = initTable[string, int]()
userAges["Alice"] = 30
userAges["Bob"] = 25

# Or use table literals
var scores = {"Alice": 100, "Bob": 85, "Carol": 92}.toTable()

# Check if key exists
if "Alice" in scores:
  echo "Alice's score: ", scores["Alice"]

# Iterate over table
for name, score in scores:
  echo name, ": ", score

# Sets
import sets

var uniqueNumbers = initHashSet[int]()
uniqueNumbers.incl(1)
uniqueNumbers.incl(2)
uniqueNumbers.incl(1)  # Duplicate, won't be added

echo uniqueNumbers.len  # 2
echo 1 in uniqueNumbers  # true

# Queues and deques
import deques

var queue = initDeque[string]()
queue.addLast("first")
queue.addLast("second")
queue.addFirst("zero")

echo queue.popFirst()  # "zero"
echo queue.popLast()   # "second"
```

## Development Environment Setup

### Installing Nim
```bash
# Method 1: Using choosenim (recommended)
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
choosenim stable  # Install latest stable version

# Method 2: Package managers
# macOS with Homebrew
brew install nim

# Ubuntu/Debian
sudo apt-get install nim

# Windows with Chocolatey
choco install nim

# Arch Linux
sudo pacman -S nim
```

### IDE and Editor Support
**VS Code (Most Popular):**
- Install "Nim" extension by nimsaem
- Provides syntax highlighting, IntelliSense, and debugging
- Configure with nim-lsp for best experience

**Vim/Neovim:**
- Install nim.vim plugin
- Use with coc.vim or nvim-lsp for language server support

**Sublime Text:**
- Install "Nim" package via Package Control

**Emacs:**
- Use nim-mode package

**JetBrains IDEs:**
- Install Nim plugin (community-developed)

### Essential Tools
```bash
# Install nimlsp for IDE support
nimble install nimlsp

# Install nimfmt for code formatting
nimble install nimfmt

# Install testament for testing
nimble install testament

# Install nim-gdb for debugging
nimble install nim-gdb
```

### Project Structure Best Practices
```
myproject/
├── myproject.nimble       # Project configuration
├── src/
│   ├── myproject.nim      # Main module
│   ├── myproject/
│   │   ├── config.nim     # Configuration
│   │   ├── utils.nim      # Utilities
│   │   └── models.nim     # Data models
├── tests/
│   ├── test_config.nim    # Unit tests
│   ├── test_utils.nim
│   └── test_models.nim
├── docs/                  # Documentation
├── examples/              # Example usage
└── README.md
```

### Configuration Files
**nim.cfg** (project-specific compiler settings):
```ini
# Enable runtime checks in debug builds
--checks:on
--stackTrace:on
--lineTrace:on

# Optimize for release builds
--opt:speed
--gc:arc
```

**config.nims** (NimScript configuration):
```nim
# Configure search paths
switch("path", "$projectDir/src")

# Custom tasks
task test, "Run tests":
  exec "nim c -r tests/test_all.nim"

task docs, "Generate documentation":
  exec "nim doc --project --out:docs src/myproject.nim"
```

### Debugging Setup
```bash
# Compile with debugging symbols
nim c --debugger:native --debugInfo myapp.nim

# Use with GDB
gdb ./myapp

# Or with LLDB on macOS
lldb ./myapp
```

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

### Full Example: Multiple External Dependencies and Build Task

Here's an example of a Nimble project with more than three external dependencies and a custom build task.

#### 1. Project Structure
```
myapp/
├── myapp.nimble
├── src/
│   └── myapp.nim
└── README.md
```

#### 2. myapp.nimble
```nim
# myapp.nimble
version       = "0.1.0"
author        = "Your Name"
description   = "Demo app with multiple dependencies and build task"
license       = "MIT"

srcDir        = "src"
bin           = @ ["myapp"]
requires      = @ [
  "strutils >= 0.13.0",   # String utilities
  "httpbeast >= 0.2.0",   # HTTP server
  "jsony >= 0.2.0",       # JSON parsing
  "osproc >= 1.0.0"       # OS process utilities
]

# Custom build task
task buildapp, "Builds the app using nimble":
  exec "nimble build -y"
```
- `requires` lists four external dependencies from the Nimble repository.

#### 3. src/myapp.nim
```nim
import strutils, jsony, osproc

echo "Uppercase: ", "hello world".toUpper()

let jsonStr = "{""name"": ""Nimble"", ""year"": 2025}"
let data = jsonStr.fromJson(Table[string, JsonNode])
echo "Parsed JSON: ", data["name"].getStr()

echo "Current process ID: ", getCurrentProcessId()
```

#### 4. Build the App Using the Task
From the project root, run:
```bash
nimble buildapp
```
This will:
- Ensure all dependencies are installed
- Build the app using the custom task
- Output the executable (e.g., `myapp` or `myapp.exe`)

#### Notes
- You can add as many dependencies as needed in the `requires` field.
- The code sample uses at least two dependencies (`strutils`, `jsony`, and `osproc`).
- You can define more tasks for testing, cleaning, etc.

## NimScript: Comprehensive Guide

NimScript is much more than just Nimble tasks. It's a full scripting environment that allows you to write system scripts, configuration files, and automation tools using Nim syntax.

### What is NimScript?

NimScript is a subset of Nim that runs at compile-time or as a standalone script interpreter. It's designed for:
- Build automation and configuration
- System scripting (like Python or Bash scripts)
- Cross-platform automation
- CI/CD pipelines
- Package management
- Code generation

### Standalone NimScript Files

You can create standalone `.nims` files that run like traditional scripts:

**hello.nims**
```nim
#!/usr/bin/env nim script
# The shebang line makes it executable on Unix systems

echo "Hello from NimScript!"
echo "Arguments: "
for i in 1..paramCount():
  echo "  ", i, ": ", paramStr(i)
```

Run with:
```bash
nim script hello.nims arg1 arg2
# or make it executable and run directly:
chmod +x hello.nims
./hello.nims arg1 arg2
```

### File and Directory Operations

NimScript provides extensive file system operations:

```nim
import os, strutils, times

# Create directories
if not dirExists("build"):
  createDir("build")
  echo "Created build directory"

# Copy files
if fileExists("config.json"):
  copyFile("config.json", "build/config.json")
  echo "Copied config file"

# List files with filtering
echo "Source files:"
for file in walkDirRec("src"):
  if file.endsWith(".nim"):
    echo "  ", file

# File information
let info = getFileInfo("script.nims")
echo "Script size: ", info.size, " bytes"
echo "Modified: ", info.lastWriteTime.format("yyyy-MM-dd HH:mm:ss")

# Environment variables
echo "HOME: ", getEnv("HOME", "not set")
echo "PATH: ", getEnv("PATH")[0..50] & "..."

# Cross-platform paths
let configPath = getConfigDir() / "myapp" / "config.json"
echo "Config path: ", configPath
```

### System Integration and Process Management

```nim
import osproc, strutils

# Execute shell commands and capture output
let (output, exitCode) = execCmdEx("git status --porcelain")
if exitCode == 0:
  if output.strip() == "":
    echo "Git working directory is clean"
  else:
    echo "Git has uncommitted changes:"
    echo output
else:
  echo "Git command failed"

# Run commands with real-time output
echo "Running tests..."
let testProcess = startProcess("nim", args=["c", "-r", "tests/test_all.nim"])
let testOutput = testProcess.outputStream.readAll()
testProcess.close()
echo testOutput

# Platform-specific commands
when defined(windows):
  exec "dir"
else:
  exec "ls -la"

# Check if programs exist
if findExe("git") != "":
  echo "Git is available"
else:
  echo "Git not found in PATH"
```

### Configuration Management

NimScript is excellent for configuration files:

**config.nims**
```nim
# Project configuration script
import os, strutils

# Set compiler flags based on environment
when defined(debug):
  switch("stackTrace", "on")
  switch("lineTrace", "on")
  switch("checks", "on")
  echo "Debug mode enabled"
else:
  switch("opt", "speed")
  switch("gc", "arc")
  echo "Release mode enabled"

# Platform-specific settings
when defined(windows):
  switch("passL", "-lws2_32")  # Windows Sockets
elif defined(macosx):
  switch("passL", "-framework CoreFoundation")
elif defined(linux):
  switch("passL", "-lpthread")

# Custom paths
switch("path", "src")
switch("path", "libs")

# Version information
const
  version = "1.0.0"
  gitHash = staticExec("git rev-parse --short HEAD").strip()
  buildDate = staticExec("date").strip()

switch("define", "version:" & version)
switch("define", "gitHash:" & gitHash)
switch("define", "buildDate:" & buildDate)
```

### Advanced Build Automation

**build.nims**
```nim
import os, strutils, times

# Build configuration
const
  srcDir = "src"
  binDir = "bin"
  testDir = "tests"

# Helper procedures
proc runCmd(cmd: string) =
  echo "Running: ", cmd
  let exitCode = execShellCmd(cmd)
  if exitCode != 0:
    echo "Command failed with exit code: ", exitCode
    quit(1)

proc buildTarget(target: string, flags: string = "") =
  let outputPath = binDir / target
  if not dirExists(binDir):
    createDir(binDir)
  
  let cmd = "nim c " & flags & " -o:" & outputPath & " " & srcDir / target & ".nim"
  runCmd(cmd)

# Tasks (can be called from command line)
task clean, "Clean build artifacts":
  if dirExists(binDir):
    removeDir(binDir)
    echo "Cleaned build directory"

task build, "Build the application":
  buildTarget("myapp", "--opt:speed")
  echo "Build completed successfully"

task debug, "Build debug version":
  buildTarget("myapp", "--stackTrace:on --lineTrace:on")
  echo "Debug build completed"

task test, "Run tests":
  for testFile in walkFiles(testDir / "test_*.nim"):
    echo "Running test: ", testFile
    runCmd("nim c -r " & testFile)

task release, "Build release version":
  buildTarget("myapp", "--opt:speed --gc:arc -d:release")
  echo "Release build completed"
  
  # Create distribution package
  let distDir = "dist"
  if dirExists(distDir):
    removeDir(distDir)
  createDir(distDir)
  
  when defined(windows):
    copyFile(binDir / "myapp.exe", distDir / "myapp.exe")
  else:
    copyFile(binDir / "myapp", distDir / "myapp")
  
  copyFile("README.md", distDir / "README.md")
  echo "Distribution package created in ", distDir

task install, "Install the application":
  when defined(windows):
    let installDir = getEnv("PROGRAMFILES") / "MyApp"
  else:
    let installDir = "/usr/local/bin"
  
  echo "Installing to: ", installDir
  # Implementation depends on your needs
```

### Package Management Scripts

**setup.nims**
```nim
# Project setup script
import os, osproc, strutils

proc installDependency(package: string, version: string = "") =
  let cmd = if version != "": 
    "nimble install " & package & "@" & version
  else:
    "nimble install " & package
  
  echo "Installing: ", package
  let exitCode = execShellCmd(cmd)
  if exitCode != 0:
    echo "Failed to install: ", package
    quit(1)

proc checkNimVersion() =
  let (output, exitCode) = execCmdEx("nim --version")
  if exitCode != 0:
    echo "Nim is not installed or not in PATH"
    quit(1)
  
  let firstLine = output.splitLines()[0]
  echo "Found: ", firstLine

proc main() =
  echo "Setting up development environment..."
  
  # Check prerequisites
  checkNimVersion()
  
  # Install dependencies
  let dependencies = [
    ("jester", "0.5.0"),
    ("norm", ""),
    ("chronicles", ""),
    ("yaml", "")
  ]
  
  for (pkg, ver) in dependencies:
    installDependency(pkg, ver)
  
  # Create project structure
  let dirs = ["src", "tests", "docs", "examples"]
  for dir in dirs:
    if not dirExists(dir):
      createDir(dir)
      echo "Created directory: ", dir
  
  # Create initial files
  if not fileExists("src/main.nim"):
    writeFile("src/main.nim", """
echo "Hello, World!"
""")
    echo "Created src/main.nim"
  
  echo "Setup completed successfully!"

main()
```

### CI/CD Integration

**ci.nims**
```nim
# Continuous Integration script
import os, osproc, strutils, times

proc runTests(): bool =
  echo "Running test suite..."
  let startTime = now()
  
  for testFile in walkFiles("tests/test_*.nim"):
    echo "Running: ", testFile
    let exitCode = execShellCmd("nim c -r " & testFile)
    if exitCode != 0:
      echo "Test failed: ", testFile
      return false
  
  let duration = now() - startTime
  echo "All tests passed in ", duration.inMilliseconds, "ms"
  return true

proc checkCodeStyle(): bool =
  echo "Checking code style..."
  let exitCode = execShellCmd("nimfmt --check src/")
  if exitCode != 0:
    echo "Code style check failed"
    return false
  
  echo "Code style check passed"
  return true

proc buildDocumentation() =
  echo "Building documentation..."
  if not dirExists("docs"):
    createDir("docs")
  
  let exitCode = execShellCmd("nim doc --project --out:docs src/main.nim")
  if exitCode != 0:
    echo "Documentation build failed"
    quit(1)
  
  echo "Documentation built successfully"

proc main() =
  echo "Starting CI pipeline..."
  
  if not checkCodeStyle():
    quit(1)
  
  if not runTests():
    quit(1)
  
  buildDocumentation()
  
  echo "CI pipeline completed successfully!"

main()
```

### Cross-Platform Scripting

**deploy.nims**
```nim
# Deployment script that works on Windows, macOS, and Linux
import os, strutils, osproc

proc detectOS(): string =
  when defined(windows):
    return "windows"
  elif defined(macosx):
    return "macos"
  elif defined(linux):
    return "linux"
  else:
    return "unknown"

proc getPlatformExe(name: string): string =
  when defined(windows):
    return name & ".exe"
  else:
    return name

proc deployTo(target: string) =
  let platform = detectOS()
  let exeName = getPlatformExe("myapp")
  
  echo "Deploying to ", target, " on ", platform
  
  case target:
  of "local":
    let homeDir = getHomeDir()
    let binDir = homeDir / "bin"
    if not dirExists(binDir):
      createDir(binDir)
    copyFile("bin" / exeName, binDir / exeName)
    echo "Deployed to: ", binDir / exeName
    
  of "system":
    when defined(windows):
      let systemDir = getEnv("SYSTEMROOT") / "System32"
      copyFile("bin" / exeName, systemDir / exeName)
    else:
      let systemDir = "/usr/local/bin"
      copyFile("bin" / exeName, systemDir / exeName)
    echo "Deployed to system directory"
    
  of "docker":
    runCmd("docker build -t myapp .")
    echo "Docker image built"
    
  else:
    echo "Unknown deployment target: ", target
    quit(1)

proc runCmd(cmd: string) =
  echo "Running: ", cmd
  let exitCode = execShellCmd(cmd)
  if exitCode != 0:
    echo "Command failed"
    quit(1)

# Main deployment logic
if paramCount() == 0:
  echo "Usage: nim script deploy.nims <target>"
  echo "Targets: local, system, docker"
  quit(1)

let target = paramStr(1)
deployTo(target)
```

### Environment and Configuration Detection

**env.nims**
```nim
# Environment detection and configuration
import os, strutils, osproc

proc detectEnvironment(): string =
  # Check for common CI environment variables
  if getEnv("CI") != "":
    return "ci"
  elif getEnv("GITHUB_ACTIONS") != "":
    return "github_actions"
  elif getEnv("GITLAB_CI") != "":
    return "gitlab_ci"
  elif getEnv("DEVELOPMENT") != "":
    return "development"
  else:
    return "production"

proc getSystemInfo() =
  echo "System Information:"
  echo "  OS: ", detectOS()
  echo "  Environment: ", detectEnvironment()
  echo "  CPU Count: ", countProcessors()
  echo "  Current Dir: ", getCurrentDir()
  echo "  Home Dir: ", getHomeDir()
  echo "  Temp Dir: ", getTempDir()
  
  # Git information
  if findExe("git") != "":
    let (branch, _) = execCmdEx("git branch --show-current")
    let (hash, _) = execCmdEx("git rev-parse --short HEAD")
    echo "  Git Branch: ", branch.strip()
    echo "  Git Hash: ", hash.strip()

proc configureForEnvironment() =
  let env = detectEnvironment()
  
  case env:
  of "ci":
    echo "Configuring for CI environment"
    putEnv("NIM_OPTS", "--hints:off --verbosity:0")
  of "development":
    echo "Configuring for development"
    putEnv("NIM_OPTS", "--stackTrace:on --lineTrace:on")
  of "production":
    echo "Configuring for production"
    putEnv("NIM_OPTS", "--opt:speed --gc:arc")
  else:
    echo "Using default configuration"

# Main execution
getSystemInfo()
configureForEnvironment()
```

### Tips for Effective NimScript Usage

1. **Use .nims files for reusable scripts** - They can be version controlled and shared across projects
2. **Leverage cross-platform compatibility** - NimScript handles path separators and platform differences
3. **Error handling** - Always check return codes and handle failures gracefully
4. **Modular design** - Break large scripts into smaller, focused procedures
5. **Documentation** - Comment your scripts well, especially complex build logic
6. **Testing** - Test your scripts on different platforms and environments
7. **Version control** - Keep build scripts in version control alongside your code

### Common NimScript Patterns

```nim
# Pattern 1: Safe command execution
proc safeExec(cmd: string): bool =
  echo "Executing: ", cmd
  let exitCode = execShellCmd(cmd)
  if exitCode != 0:
    echo "Command failed with exit code: ", exitCode
    return false
  return true

# Pattern 2: File operations with error handling
proc safeCopyFile(src, dst: string): bool =
  try:
    copyFile(src, dst)
    echo "Copied: ", src, " -> ", dst
    return true
  except:
    echo "Failed to copy: ", src, " -> ", dst
    return false

# Pattern 3: Environment-based configuration
proc getConfig(key: string, default: string = ""): string =
  let envValue = getEnv(key)
  if envValue != "":
    return envValue
  return default

# Pattern 4: Conditional execution
proc runIfExists(exe: string, args: string = ""): bool =
  if findExe(exe) != "":
    return safeExec(exe & " " & args)
  else:
    echo "Executable not found: ", exe
    return false
```

NimScript is a powerful tool that bridges the gap between simple build scripts and complex automation systems. It provides the expressiveness of Nim with the practicality of scripting languages, making it ideal for modern development workflows.

## Documentation Generation for Nimble Projects

Documentation generation means creating readable HTML web pages from special comments in your code. Nim reads these comments and creates a website that others can browse to understand your project.

### Writing Documentation Comments

Use `##` (double hash) to write documentation comments in your code:

```nim
## This module provides math utilities for calculations.

proc add*(x, y: int): int =
  ## Adds two integers together and returns the result.
  ## 
  ## Arguments:
  ## - x: The first number to add
  ## - y: The second number to add
  ## 
  ## Example:
  ## ```nim
  ## let result = add(5, 3)
  ## echo result  # prints 8
  ## ```
  result = x + y
```

The `##` comments explain what your functions and modules do. The `*` after function names makes them public (available to other modules).

### Generating Documentation for Specific Files

Sometimes you want to document only certain files instead of the whole project:

**Single File Documentation:**
```bash
# Generate docs for one specific file
nim doc --out:docs/utils.html src/utils.nim
```

**Multiple Specific Files:**
```bash
# Generate docs for specific files only
nim doc --out:docs src/utils.nim src/models.nim src/config.nim
```

This approach is useful when:
- You want to document only public modules
- Your project has many internal files you don't want to expose
- You're documenting a specific part of a large project

**Document All Files in Project (Including Subdirectories):**

If you want to generate documentation for every .nim file in your project, including files in subdirectories that might not be imported by your main module:

```bash
# Find all .nim files and generate docs for each
find ./ -name "*.nim" -exec nim doc --project --out:docs {} \;
```

**Explanation of each part:**
- `find` - the command that searches for files
- `./` - the directory to search in (starts from current directory, searches all subdirectories)
- `-name "*.nim"` - only find files whose names end with .nim
- `-exec` - run a command for each file that was found
- `nim doc` - the Nim documentation generator command
- `--project` - generate documentation for the entire project including all imported modules
- `--out:docs` - save output to docs folder (Nim will create appropriate file names)
- `{}` - placeholder that gets replaced with each found file path
- `\;` - marks the end of the -exec command (semicolon must be escaped with backslash)

**Example of what happens:**
If the command finds `./src/utils.nim` and `./src/data/models.nim`, it runs:
```bash
nim doc --project --out:docs ./src/utils.nim
nim doc --project --out:docs ./src/data/models.nim
```

Or using a Nimble task that works on all platforms:

```nim
# In myproject.nimble
task docs_all, "Generate docs for every file in the project":
  if not dirExists("docs"):
    mkDir("docs")
  
  # Generate docs for all .nim files recursively
  for file in walkDirRec("src"):
    if file.endsWith(".nim"):
      let outputFile = "docs" / file.replace("src/", "").replace(".nim", ".html")
      let outputDir = outputFile.parentDir()
      if not dirExists(outputDir):
        mkDir(outputDir)
      exec "nim doc --out:" & outputFile & " " & file
  
  echo "Documentation generated for all files"
```

This approach:
- Finds every .nim file in your src directory and subdirectories
- Generates documentation for each file individually
- Preserves the directory structure in the docs folder
- Documents files even if they're not imported by your main module

### Generating Documentation for Whole Project

The most important technique is generating documentation for your entire project at once. This creates HTML files for all modules in your project.

**Basic Project Documentation:**
```bash
# Generate docs for entire project
nim doc --project --out:docs src/myproject.nim
```

This command:
- `--project` means "document this file AND all files it imports"
- Starts from your main file (`src/myproject.nim`)
- Follows all `import` statements to find other files
- Creates HTML files for each module
- Creates an index page linking all modules

**Example result:**
```
myproject/
├── src/
│   ├── myproject.nim     # Main module
│   ├── utils.nim         # Utility functions
│   └── models.nim        # Data types
└── docs/                 # Generated documentation
    ├── myproject.html    # Main page
    ├── utils.html        # Utils documentation
    ├── models.html       # Models documentation
    └── theindex.html     # Index of all modules
```

### Using Nimble Tasks for Project Documentation

Create a Nimble task to easily regenerate documentation for your whole project:

```nim
# In myproject.nimble
task docs, "Generate project documentation":
  exec "nim doc --project --index:on --out:docs src/myproject.nim"
```

Run with:
```bash
nimble docs
```

**Enhanced Project Documentation Task:**
```nim
# In myproject.nimble
task docs, "Generate complete project documentation":
  # Create docs directory
  if not dirExists("docs"):
    mkDir("docs")
  
  # Generate documentation with search index
  exec "nim doc --project --index:on --out:docs src/myproject.nim"
  
  # Copy README to docs folder
  if fileExists("README.md"):
    exec "cp README.md docs/"
  
  echo "Project documentation generated in docs/"
  echo "Open docs/myproject.html in your browser"
```

This task:
- `--index:on` creates a searchable index of all functions and types
- Copies your README file to the documentation folder
- Gives helpful messages about where to find the documentation

### Complete Project Documentation Workflow

For serious projects, use this complete workflow:

```nim
# In myproject.nimble
task clean_docs, "Remove old documentation":
  if dirExists("docs"):
    rmDir("docs")
  echo "Documentation cleaned"

task build_docs, "Build complete project documentation":
  # Clean old documentation
  exec "nimble clean_docs"
  
  # Create fresh documentation
  mkDir("docs")
  exec "nim doc --project --index:on --out:docs src/myproject.nim"
  
  # Copy additional project files
  if fileExists("README.md"):
    exec "cp README.md docs/"
  if dirExists("examples"):
    exec "cp -r examples docs/"
  
  echo "Complete project documentation built!"

task serve_docs, "View documentation in browser":
  if not dirExists("docs"):
    exec "nimble build_docs"
  
  echo "Starting documentation server at http://localhost:8888"
  echo "The search button will work properly when served through this server"
  exec "http-server docs -p 8888"
```

**Using the workflow:**
```bash
# Generate complete project documentation
nimble build_docs

# View documentation in web browser
nimble serve_docs

# Clean old documentation
nimble clean_docs
```

### Documentation Best Practices for Projects

1. **Document your main module first** - This becomes the entry point for your documentation
2. **Use `*` to make functions public** - Only public functions appear in documentation
3. **Import all modules in main** - This ensures all modules get documented
4. **Include examples** - Show how to use your project's main features
5. **Keep documentation up to date** - Regenerate docs when you change code

**Example of well-documented project structure:**
```nim
# src/myproject.nim (main module)
## MyProject provides tools for data processing.
## 
## This is the main entry point for the project.
## Import this module to access all functionality.
## 
## Example:
## ```nim
## import myproject
## let result = processData("input.txt")
## echo result
## ```

import myproject/utils, myproject/models

proc processData*(filename: string): string =
  ## Main function to process data files.
  ## This combines utilities and models to process input.
  # implementation here
```

This approach creates comprehensive documentation that covers your entire project, making it easy for others to understand and use your code.

### Important Note: Search Button in Generated Documentation

When you generate documentation with `nim doc`, the HTML files include a search button that allows you to search through all symbols in your package. However, **the search button is broken when you open the HTML files directly in your browser** (using `file://` URLs). This happens because the search functionality requires the `dochack.js` file that gets generated with `nim doc`, but browsers block local JavaScript files for security reasons.

**Solution: Use a Real Web Server**

To make the search button work properly, you need to serve the documentation through a real web server instead of opening the files directly. Here are two simple solutions:

**Option 1: Using Node.js http-server (Recommended)**
```bash
# Install http-server globally (only need to do this once)
npm install -g http-server

# Navigate to your docs directory and start the server
cd docs
http-server . -p 8888

# Or start the server from your project root
http-server docs -p 8888
```

Then open `http://localhost:8888` in your browser. The search button will now work perfectly - you can type in any symbol name from your whole package and it will find and highlight matches.

**Option 2: Using Python's built-in server**
```bash
# Navigate to your docs directory
cd docs
python3 -m http.server 8888

# Or from project root
python3 -m http.server 8888 --directory docs
```

Then open `http://localhost:8888` in your browser.

**What the Search Button Does When Working:**
- You can search for any function name, type name, variable name, or module name from your entire project
- It provides instant search results as you type
- It highlights matching text and shows you exactly where each symbol is defined
- It works across all modules in your project documentation

This search functionality is very powerful for navigating large codebases and makes your documentation much more user-friendly.

