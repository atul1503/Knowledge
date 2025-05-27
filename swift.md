Notes:

1. Use `var` for stuff that changes, `let` for stuff that doesn't. Always use `let` if you can.
   ```swift
   let name = "John"  // Can't change this
   var age = 25       // Can change this
   ```

2. Swift figures out types automatically. You can specify them if you want.
   ```swift
   let message = "Hello"           // Swift knows it's a String
   let count: Int = 42            // You tell Swift it's an Int
   let price: Double = 19.99      // You tell Swift it's a Double
   ```

3. Optionals handle when something might not exist. Use `?` for maybe-has-value and `!` to force unwrap (be careful with this).
   ```swift
   var name: String? = "Alice"
   var age: Int? = nil
   
   // Safe way to unwrap
   if let unwrappedName = name {
       print("Name is \(unwrappedName)")
   }
   
   // Give default value if nil
   let displayName = name ?? "Unknown"
   ```

4. `guard` lets you exit early from functions. Good for checking stuff at the start.
   ```swift
   func processUser(name: String?) {
       guard let userName = name else {
           print("No name provided")
           return
       }
       print("Processing user: \(userName)")
   }
   ```

5. Put variables inside strings with `\()`.
   ```swift
   let user = "John"
   let points = 150
   let message = "User \(user) has \(points) points"
   ```

6. Functions use `func`. You can give parameters labels to make calls easier to read.
   ```swift
   func greet(person name: String, from city: String) -> String {
       return "Hello \(name) from \(city)!"
   }
   
   // Call it like this
   let greeting = greet(person: "Alice", from: "NYC")
   ```

7. Closures are like anonymous functions. They can use variables from around them.
   ```swift
   let numbers = [1, 2, 3, 4, 5]
   
   // Long way
   let doubled = numbers.map({ (number: Int) -> Int in
       return number * 2
   })
   
   // Short way
   let tripled = numbers.map { $0 * 3 }
   ```

8. Arrays have useful methods like `filter`, `map`, and `reduce`.
   ```swift
   let numbers = [1, 2, 3, 4, 5, 6]
   
   let evens = numbers.filter { $0 % 2 == 0 }     // [2, 4, 6]
   let doubled = numbers.map { $0 * 2 }           // [2, 4, 6, 8, 10, 12]
   let sum = numbers.reduce(0) { $0 + $1 }        // 21
   ```

9. Use `struct` for simple data, `class` when you need inheritance.
   ```swift
   struct Point {
       var x: Double
       var y: Double
   }
   
   class Vehicle {
       var speed: Double = 0.0
       func accelerate() {
           speed += 10
       }
   }
   ```

10. Properties can store values, calculate values, or watch for changes.
    ```swift
    class Temperature {
        var celsius: Double = 0.0
        
        // Calculated property
        var fahrenheit: Double {
            get { return celsius * 9/5 + 32 }
            set { celsius = (newValue - 32) * 5/9 }
        }
        
        // Runs when value changes
        var room: String = "Living Room" {
            didSet {
                print("Moved to \(room)")
            }
        }
    }
    ```

11. Use `init` to set up your objects when they're created.
    ```swift
    class Person {
        let name: String
        var age: Int
        
        init(name: String, age: Int) {
            self.name = name
            self.age = age
        }
        
        // Alternative way to create
        convenience init(name: String) {
            self.init(name: name, age: 0)
        }
    }
    ```

12. Protocols are like contracts. Types that adopt them must have certain things.
    ```swift
    protocol Drawable {
        func draw()
        var area: Double { get }
    }
    
    struct Circle: Drawable {
        var radius: Double
        
        func draw() {
            print("Drawing a circle")
        }
        
        var area: Double {
            return Double.pi * radius * radius
        }
    }
    ```

13. Extensions let you add new stuff to existing types.
    ```swift
    extension String {
        func removeSpaces() -> String {
            return self.replacingOccurrences(of: " ", with: "")
        }
        
        var isEmail: Bool {
            return self.contains("@") && self.contains(".")
        }
    }
    
    let email = "user@example.com"
    print(email.isEmail)  // true
    ```

14. You can give default implementations in protocol extensions.
    ```swift
    protocol Identifiable {
        var id: String { get }
    }
    
    extension Identifiable {
        func printID() {
            print("ID: \(id)")
        }
    }
    ```

15. Handle errors with `throw`, `try`, `catch`.
    ```swift
    enum NetworkError: Error {
        case badURL
        case noConnection
        case timeout
    }
    
    func fetchData(from url: String) throws -> String {
        guard !url.isEmpty else {
            throw NetworkError.badURL
        }
        // Simulate network call
        return "Data from \(url)"
    }
    
    // How to use it
    do {
        let data = try fetchData(from: "https://api.example.com")
        print(data)
    } catch NetworkError.badURL {
        print("Invalid URL")
    } catch {
        print("Other error: \(error)")
    }
    ```

16. Swift handles memory automatically. Use `weak` to avoid retain cycles where objects hold onto each other.
    ```swift
    class Person {
        let name: String
        weak var apartment: Apartment?  // weak to break cycle
        
        init(name: String) {
            self.name = name
        }
    }
    
    class Apartment {
        let unit: String
        weak var tenant: Person?        // weak reference
        
        init(unit: String) {
            self.unit = unit
        }
    }
    ```

17. Use `[weak self]` in closures to avoid retain cycles.
    ```swift
    class ViewController {
        var name = "Controller"
        
        func setupTimer() {
            Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
                guard let self = self else { return }
                print("Timer fired in \(self.name)")
            }
        }
    }
    ```

18. Main collection types: Arrays, Sets, and Dictionaries.
    ```swift
    var fruits = ["apple", "banana", "orange"]          // Array
    var uniqueNumbers: Set = [1, 2, 3, 4, 5]          // Set
    var capitals = ["US": "Washington", "UK": "London"] // Dictionary
    
    // Safe way to get dictionary values
    let usCapital = capitals["US"] ?? "Unknown"
    ```

19. Generics let you write code that works with any type.
    ```swift
    func swap<T>(_ a: inout T, _ b: inout T) {
        let temp = a
        a = b
        b = temp
    }
    
    // Generic struct
    struct Stack<Element> {
        private var items: [Element] = []
        
        mutating func push(_ item: Element) {
            items.append(item)
        }
        
        mutating func pop() -> Element? {
            return items.popLast()
        }
    }
    ```

20. SwiftUI is the new way to build UI. UIKit is the old way.
    ```swift
    // SwiftUI example
    struct ContentView: View {
        @State private var name = ""
        
        var body: some View {
            VStack {
                TextField("Enter name", text: $name)
                Text("Hello, \(name)!")
            }
            .padding()
        }
    }
    ```

21. SwiftUI uses `@State`, `@Binding`, `@ObservedObject` to manage data.
    ```swift
    struct LoginView: View {
        @State private var username = ""
        @State private var password = ""
        @State private var isLoggedIn = false
        
        var body: some View {
            if isLoggedIn {
                Text("Welcome!")
            } else {
                LoginForm(username: $username, password: $password)
            }
        }
    }
    ```

22. Combine handles async events and data streams.
    ```swift
    import Combine
    
    class DataManager: ObservableObject {
        @Published var items: [String] = []
        private var cancellables = Set<AnyCancellable>()
        
        func fetchData() {
            URLSession.shared.dataTaskPublisher(for: url)
                .map(\.data)
                .decode(type: [String].self, decoder: JSONDecoder())
                .receive(on: DispatchQueue.main)
                .sink(
                    receiveCompletion: { _ in },
                    receiveValue: { [weak self] items in
                        self?.items = items
                    }
                )
                .store(in: &cancellables)
        }
    }
    ```

23. Use `async/await` for async functions instead of completion handlers.
    ```swift
    func fetchUserData(id: String) async throws -> User {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(User.self, from: data)
    }
    
    // How to call it
    Task {
        do {
            let user = try await fetchUserData(id: "123")
            print("Fetched user: \(user.name)")
        } catch {
            print("Failed to fetch user: \(error)")
        }
    }
    ```

24. `Result` type handles success and failure cleanly.
    ```swift
    func divide(_ a: Double, by b: Double) -> Result<Double, ArithmeticError> {
        guard b != 0 else {
            return .failure(.divisionByZero)
        }
        return .success(a / b)
    }
    
    // How to use it
    switch divide(10, by: 2) {
    case .success(let result):
        print("Result: \(result)")
    case .failure(let error):
        print("Error: \(error)")
    }
    ```

25. Use `enum` for related constants instead of separate variables.
    ```swift
    enum Theme {
        case light, dark, auto
        
        var backgroundColor: UIColor {
            switch self {
            case .light: return .white
            case .dark: return .black
            case .auto: return .systemBackground
            }
        }
    }
    ```

26. Use `struct` instead of `class` when you don't need inheritance.

27. `defer` runs cleanup code no matter how your function exits.
    ```swift
    func processFile() {
        let file = openFile()
        defer { closeFile(file) }  // Always runs before function returns
        
        // Process file...
        if someCondition {
            return  // defer block still executes
        }
        // More processing...
    }
    ```

28. Use enums instead of strings when possible. Compiler catches typos.
    ```swift
    enum HTTPMethod {
        case get, post, put, delete
    }
    
    // Better than using strings like "GET", "POST"
    func makeRequest(method: HTTPMethod) {
        switch method {
        case .get: // Handle GET
        case .post: // Handle POST
        // Compiler ensures all cases are handled
        }
    }
    ``` 

## Swift Package Manager (SPM) - No Xcode Needed

29. Create a new Swift project from terminal using `swift package init`.
    ```bash
    # Create executable project
    swift package init --type executable --name MyApp
    
    # Create library project  
    swift package init --type library --name MyLibrary
    
    # Create empty package
    swift package init
    ```

30. **Package.swift** is your project config file. It defines targets, products, and dependencies.
    ```swift
    // swift-tools-version: 5.8
    import PackageDescription
    
    let package = Package(
        name: "MyApp",
        products: [
            .executable(name: "MyApp", targets: ["MyApp"]),
        ],
        dependencies: [
            .package(url: "https://github.com/apple/swift-argument-parser", from: "1.0.0"),
        ],
        targets: [
            .executableTarget(
                name: "MyApp",
                dependencies: [
                    .product(name: "ArgumentParser", package: "swift-argument-parser")
                ]
            ),
            .testTarget(
                name: "MyAppTests",
                dependencies: ["MyApp"]
            ),
        ]
    )
    ```

31. **Products** are what your package builds. Can be executables or libraries.
    ```swift
    products: [
        .executable(name: "MyApp", targets: ["MyApp"]),           // Makes an app you can run
        .library(name: "MyLibrary", targets: ["MyLibrary"]),     // Makes a library others can import
    ]
    ```

32. **Targets** are groups of source files. Each target becomes a module.
    ```swift
    targets: [
        .executableTarget(name: "MyApp"),        // App target
        .target(name: "MyLibrary"),              // Library target  
        .testTarget(name: "MyAppTests"),         // Test target
    ]
    ```

33. **Dependencies** are external packages you want to use.
    ```swift
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        .package(url: "https://github.com/apple/swift-nio.git", from: "2.0.0"),
    ],
    ```

34. Basic project structure SPM creates:
    ```
    MyApp/
    ├── Package.swift           // Project config
    ├── Sources/
    │   └── MyApp/
    │       └── main.swift      // Your code goes here
    ├── Tests/
    │   └── MyAppTests/
    │       └── MyAppTests.swift
    └── .gitignore
    ```

35. Build and run your project from terminal:
    ```bash
    # Build the project
    swift build
    
    # Run the executable
    swift run
    
    # Run specific executable if you have multiple
    swift run MyApp
    
    # Run tests
    swift test
    
    # Build for release (optimized)
    swift build -c release
    ```

36. Add dependencies by editing Package.swift, then run `swift package resolve`:
    ```bash
    # After adding dependencies to Package.swift
    swift package resolve
    
    # Update dependencies to latest versions
    swift package update
    ```

37. **Modules** are what you import. Each target becomes a module.
    ```swift
    // If you have a target named "Networking"
    import Networking
    
    // System modules
    import Foundation
    import SwiftUI
    ```

38. Create multiple targets for different parts of your app:
    ```swift
    targets: [
        .executableTarget(name: "MyApp", dependencies: ["Networking", "Database"]),
        .target(name: "Networking"),     // Network related code
        .target(name: "Database"),       // Database related code
        .testTarget(name: "MyAppTests", dependencies: ["MyApp"]),
    ]
    ```

39. Organize code in folders under Sources:
    ```
    Sources/
    ├── MyApp/
    │   ├── main.swift
    │   └── AppLogic.swift
    ├── Networking/
    │   ├── APIClient.swift
    │   └── NetworkError.swift
    └── Database/
        ├── DatabaseManager.swift
        └── Models.swift
    ```

40. **Platform requirements** - specify which platforms your package supports:
    ```swift
    let package = Package(
        name: "MyApp",
        platforms: [
            .macOS(.v12),       // Requires macOS 12+
            .iOS(.v15),         // Requires iOS 15+
            .watchOS(.v8),      // Requires watchOS 8+
            .tvOS(.v15),        // Requires tvOS 15+
        ],
        // ... rest of package
    )
    ```

41. **Local dependencies** - use packages from your file system:
    ```swift
    dependencies: [
        .package(path: "../MyLocalPackage"),        // Relative path
        .package(path: "/Users/me/MyPackage"),      // Absolute path
    ]
    ```

42. **Version requirements** for dependencies:
    ```swift
    dependencies: [
        .package(url: "https://github.com/user/repo.git", from: "1.0.0"),           // 1.0.0 or higher
        .package(url: "https://github.com/user/repo.git", "1.0.0"..<"2.0.0"),      // Between 1.0.0 and 2.0.0
        .package(url: "https://github.com/user/repo.git", exact: "1.5.0"),         // Exactly 1.5.0
        .package(url: "https://github.com/user/repo.git", branch: "main"),         // Latest from main branch
        .package(url: "https://github.com/user/repo.git", revision: "abc123"),     // Specific commit
    ]
    ```

43. Use VS Code or any editor for Swift development. Install Swift extension for VS Code for syntax highlighting and basic IntelliSense.

## Interfacing Swift with Other Languages

44. Swift can call C code directly. Just import the C library and use it.
    ```swift
    import Foundation  // Has tons of C functions
    
    // Using C math functions
    let result = sqrt(16.0)  // C function from math.h
    let randomNum = arc4random()  // C function
    ```

45. **Bridging headers** let you use C/Objective-C code in Swift projects. Create a file ending in `-Bridging-Header.h`.
    ```c
    // MyApp-Bridging-Header.h
    #include "my_c_library.h"
    #include "SomeObjectiveCClass.h"
    ```
    
    Then set it in your build settings or Package.swift:
    ```swift
    .target(
        name: "MyApp",
        publicHeadersPath: "include",
        cSettings: [
            .headerSearchPath("path/to/headers")
        ]
    )
    ```

46. **Module maps** let you import C libraries as modules. Create a `module.modulemap` file:
    ```
    module MyCLibrary {
        header "my_library.h"
        link "my_library"
        export *
    }
    ```
    
    Then import it like:
    ```swift
    import MyCLibrary
    ```

47. Swift automatically converts C types to Swift types:
    ```c
    // C function
    int add_numbers(int a, int b);
    char* get_string(void);
    ```
    
    ```swift
    // Swift usage
    let result = add_numbers(5, 3)  // Int
    let str = String(cString: get_string())  // Convert C string to Swift String
    ```

48. **Using C libraries** in SPM - add them as system libraries:
    ```swift
    .target(
        name: "MyApp",
        dependencies: [],
        linkerSettings: [
            .linkedLibrary("sqlite3"),  // Link against libsqlite3
            .linkedLibrary("curl"),     // Link against libcurl
        ]
    )
    ```

49. **C++ interop** is limited but possible through C wrappers. Create C wrapper functions:
    ```cpp
    // wrapper.cpp
    extern "C" {
        int cpp_add(int a, int b) {
            MyCppClass obj;
            return obj.add(a, b);
        }
    }
    ```
    
    ```c
    // wrapper.h
    #ifdef __cplusplus
    extern "C" {
    #endif
    
    int cpp_add(int a, int b);
    
    #ifdef __cplusplus
    }
    #endif
    ```

50. **Objective-C** works seamlessly with Swift (on Apple platforms):
    ```objc
    // SomeClass.h
    @interface SomeClass : NSObject
    - (NSString *)getName;
    - (void)setName:(NSString *)name;
    @end
    ```
    
    ```swift
    // Swift usage
    let obj = SomeClass()
    obj.setName("Hello")
    let name = obj.getName()
    ```

51. **Calling Swift from C** is harder. You need to expose Swift functions with `@_cdecl`:
    ```swift
    @_cdecl("swift_add")
    public func swiftAdd(a: Int32, b: Int32) -> Int32 {
        return a + b
    }
    ```
    
    Then compile to a library:
    ```bash
    swiftc -emit-library -emit-module MySwiftCode.swift -o libmyswift.dylib
    ```

52. **Common C interop patterns**:
    ```swift
    // C pointers become UnsafePointer in Swift
    func c_function(data: UnsafePointer<UInt8>, length: Int)
    
    // C arrays
    var buffer = [UInt8](repeating: 0, count: 1024)
    buffer.withUnsafeBufferPointer { ptr in
        c_function(ptr.baseAddress!, ptr.count)
    }
    
    // C strings
    let cString = "hello world".withCString { $0 }
    ```

53. **Memory management** when interfacing with C:
    ```swift
    // Swift manages memory automatically, but with C you might need manual management
    let ptr = malloc(1024)
    defer { free(ptr) }  // Always free C memory
    
    // Or use Swift's memory management
    let buffer = UnsafeMutableBufferPointer<UInt8>.allocate(capacity: 1024)
    defer { buffer.deallocate() }
    ```

54. **Working with C structs**:
    ```c
    // C struct
    typedef struct {
        int x;
        int y;
        char name[50];
    } Point;
    ```
    
    ```swift
    // Swift can use it directly
    var point = Point()
    point.x = 10
    point.y = 20
    // For C strings in structs, use tuple access
    ```

55. **Build flags** for C/C++ integration in SPM:
    ```swift
    .target(
        name: "MyApp",
        cSettings: [
            .define("MY_DEFINE"),                    // -DMY_DEFINE
            .headerSearchPath("include"),            // -Iinclude
            .unsafeFlags(["-O3"]),                  // Custom compiler flags
        ],
        cxxSettings: [
            .define("CPP_DEFINE"),
            .headerSearchPath("cpp_headers"),
        ],
        linkerSettings: [
            .linkedLibrary("m"),                     // Link math library
            .linkedFramework("Foundation"),          // Link framework (macOS/iOS)
            .unsafeFlags(["-L/usr/local/lib"]),     // Custom linker flags
        ]
    )
    ```

56. **Cross-platform considerations**: 
    - C interop works on all Swift platforms (Linux, Windows, macOS)
    - Objective-C only works on Apple platforms
    - Some C libraries might not be available on all platforms
    - Use `#if os(macOS)` conditionals when needed