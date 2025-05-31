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

4a. **`guard` vs `if` - when to use which one.** Both can check conditions, but they have different purposes and strengths. Use `guard` for early exits and requirements, use `if` for branching logic.

   ```swift
   // GUARD - Use for early exits and requirements
   // Guard says "this MUST be true to continue, otherwise get out"
   
   func processOrder(items: [String]?, customerID: String?, paymentMethod: String?) {
       // Check requirements at the start - if any fail, exit early
       guard let orderItems = items, !orderItems.isEmpty else {
           print("Error: No items in order")
           return  // Must exit the function
       }
       
       guard let customer = customerID, !customer.isEmpty else {
           print("Error: No customer ID")
           return  // Must exit the function
       }
       
       guard let payment = paymentMethod else {
           print("Error: No payment method")
           return  // Must exit the function
       }
       
       // If we get here, everything is valid - continue with main logic
       print("Processing order for \(customer): \(orderItems.joined(separator: ", "))")
       print("Payment method: \(payment)")
       
       // All the variables (orderItems, customer, payment) are available here
       // because guard unwrapped them in the same scope
   }
   
   // IF - Use for branching logic and optional paths
   // If says "do this OR do that" 
   
   func displayUserInfo(user: User?) {
       if let validUser = user {
           // Do something with valid user
           print("Welcome, \(validUser.name)!")
           print("Email: \(validUser.email)")
           
           // validUser is only available inside this if block
           if validUser.isPremium {
               print("Premium features available")
           } else {
               print("Upgrade to premium for more features")
           }
       } else {
           // Do something else when user is nil
           print("Please log in to continue")
           showLoginScreen()
       }
       
       // Continue with other logic regardless of user status
       print("App version: 1.0")
   }
   
   // WHEN TO USE GUARD:
   // ✅ Checking function parameters at the start
   // ✅ Validating preconditions before doing work
   // ✅ Early exits when something is wrong
   // ✅ When you need the unwrapped value throughout the rest of the function
   
   func calculateArea(width: Double?, height: Double?) -> Double? {
       guard let w = width, w > 0 else {
           print("Invalid width")
           return nil
       }
       
       guard let h = height, h > 0 else {
           print("Invalid height") 
           return nil
       }
       
       // w and h are available for the rest of the function
       let area = w * h
       print("Calculated area: \(area)")
       return area
   }
   
   // WHEN TO USE IF:
   // ✅ When you have different paths to take
   // ✅ When nil/false is a valid scenario to handle differently
   // ✅ When you only need the unwrapped value in a small block
   // ✅ When you want to continue regardless of the condition
   
   func updateUI(user: User?) {
       if let user = user {
           nameLabel.text = user.name
           emailLabel.text = user.email
           profileImage.image = user.avatar
       } else {
           nameLabel.text = "Guest User"
           emailLabel.text = "Not logged in"
           profileImage.image = defaultAvatar
       }
       
       // Always update the timestamp, regardless of user status
       lastUpdatedLabel.text = "Updated: \(Date())"
   }
   ```

4b. **Guard with assignments and multiple conditions.** Guard can check multiple things at once and assign values that stay available after the guard statement.

   ```swift
   // MULTIPLE CONDITIONS in guard
   // All conditions must be true, or else the guard fails
   
   func createUser(name: String?, email: String?, age: Int?) {
       guard let userName = name, !userName.isEmpty,           // Check name exists and not empty
             let userEmail = email, userEmail.contains("@"),   // Check email exists and has @
             let userAge = age, userAge >= 18                  // Check age exists and >= 18
       else {
           print("Invalid user data")
           return
       }
       
       // All three variables (userName, userEmail, userAge) are now available
       // and guaranteed to be valid for the rest of the function
       let user = User(name: userName, email: userEmail, age: userAge)
       print("Created user: \(user)")
   }
   
   // GUARD with complex conditions
   func processPayment(amount: Double?, card: CreditCard?) {
       guard let paymentAmount = amount, 
             paymentAmount > 0,                    // Amount must be positive
             paymentAmount <= 10000,               // Amount must be reasonable
             let creditCard = card,                // Card must exist
             !creditCard.isExpired,                // Card must not be expired
             creditCard.hasValidNumber             // Card must have valid number
       else {
           print("Payment validation failed")
           return
       }
       
       // Process payment with validated data
       print("Processing $\(paymentAmount) on card ending in \(creditCard.lastFourDigits)")
   }
   
   // GUARD with function calls and computed properties
   func downloadFile(url: String?) {
       guard let urlString = url,
             !urlString.isEmpty,
             let validURL = URL(string: urlString),    // URL(string:) returns optional
             validURL.scheme == "https"                // Only allow HTTPS
       else {
           print("Invalid URL")
           return
       }
       
       // validURL is available and guaranteed to be a valid HTTPS URL
       print("Downloading from: \(validURL)")
       // ... download logic
   }
   
   // GUARD vs IF for the same logic - see the difference
   
   // Using GUARD - cleaner for validation
   func loginUser(username: String?, password: String?) -> Bool {
       guard let user = username, !user.isEmpty else {
           print("Username required")
           return false
       }
       
       guard let pass = password, pass.count >= 8 else {
           print("Password must be at least 8 characters")
           return false
       }
       
       // Main logic is not nested - easier to read
       print("Logging in user: \(user)")
       return authenticateUser(user, pass)
   }
   
   // Using IF - more nested and harder to read
   func loginUserWithIf(username: String?, password: String?) -> Bool {
       if let user = username, !user.isEmpty {
           if let pass = password, pass.count >= 8 {
               // Main logic is deeply nested
               print("Logging in user: \(user)")
               return authenticateUser(user, pass)
           } else {
               print("Password must be at least 8 characters")
               return false
           }
       } else {
           print("Username required")
           return false
       }
   }
   
   // GUARD with different exit strategies
   func processData(data: Data?) throws {
       // In throwing functions, you can throw instead of return
       guard let validData = data, !validData.isEmpty else {
           throw DataError.emptyData
       }
       
       // Process the data...
   }
   
   func processInLoop(items: [String?]) {
       for item in items {
           // In loops, you can use continue instead of return
           guard let validItem = item, !validItem.isEmpty else {
               print("Skipping invalid item")
               continue  // Skip to next iteration
           }
           
           print("Processing: \(validItem)")
       }
   }
   
   // COMMON PATTERNS with guard
   
   // Pattern 1: API response validation
   func handleAPIResponse(data: Data?, response: URLResponse?, error: Error?) {
       guard error == nil else {
           print("Network error: \(error!)")
           return
       }
       
       guard let responseData = data else {
           print("No data received")
           return
       }
       
       guard let httpResponse = response as? HTTPURLResponse,
             httpResponse.statusCode == 200 else {
           print("Invalid HTTP response")
           return
       }
       
       // All validations passed - process the data
       parseJSON(responseData)
   }
   
   // Pattern 2: User input validation
   func registerUser(form: RegistrationForm) {
       guard !form.email.isEmpty,
             form.email.contains("@"),
             form.email.contains(".") else {
           showError("Please enter a valid email")
           return
       }
       
       guard form.password.count >= 8,
             form.password.contains(where: { $0.isNumber }),
             form.password.contains(where: { $0.isUppercase }) else {
           showError("Password must be 8+ chars with number and uppercase")
           return
       }
       
       guard form.password == form.confirmPassword else {
           showError("Passwords don't match")
           return
       }
       
       // All validation passed
       createAccount(email: form.email, password: form.password)
   }
   
   // KEY DIFFERENCES SUMMARY:
   // 
   // GUARD:
   // - Must have 'else' clause that exits (return, throw, continue, break)
   // - Unwrapped values available in same scope after guard
   // - Best for validation and early exits
   // - Keeps main logic unindented and readable
   // - Says "this must be true to continue"
   //
   // IF:
   // - Optional 'else' clause
   // - Unwrapped values only available inside if block
   // - Best for branching logic and alternatives
   // - Can handle both success and failure cases
   // - Says "do this or do that"
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

7. **Closures are like anonymous functions.** Think of them as functions without names that you can pass around like variables. They can "capture" and use variables from the surrounding code.
   ```swift
   let numbers = [1, 2, 3, 4, 5]
   
   // Long way - explicit closure syntax
   let doubled = numbers.map({ (number: Int) -> Int in
       return number * 2
   })
   
   // Medium way - Swift can infer types
   let tripled = numbers.map({ number in
       return number * 3
   })
   
   // Short way - use $0, $1, etc. for parameters
   let quadrupled = numbers.map { $0 * 4 }
   
   // Closures can capture variables from outside
   let multiplier = 10
   let scaled = numbers.map { $0 * multiplier }  // Uses 'multiplier' from outside
   
   // Trailing closure syntax - if closure is last parameter, put it outside
   numbers.forEach { number in
       print("Number: \(number)")
   }
   
   // Multiple parameter closures
   let pairs = [(1, "one"), (2, "two"), (3, "three")]
   let sorted = pairs.sorted { $0.0 < $1.0 }  // Sort by first element
   
   // Closures that return closures (higher-order functions)
   func makeMultiplier(factor: Int) -> (Int) -> Int {
       return { number in number * factor }  // This closure captures 'factor'
   }
   
   let doubler = makeMultiplier(factor: 2)
   let result = doubler(5)  // Returns 10
   ```

8. **Arrays have powerful built-in methods** for transforming and filtering data. These are called "higher-order functions" because they take other functions (closures) as parameters.
   ```swift
   let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
   
   // filter - keeps only elements that match a condition
   let evens = numbers.filter { $0 % 2 == 0 }           // [2, 4, 6, 8, 10]
   let bigNumbers = numbers.filter { $0 > 5 }           // [6, 7, 8, 9, 10]
   
   // map - transforms each element
   let doubled = numbers.map { $0 * 2 }                 // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
   let strings = numbers.map { "Number \($0)" }         // ["Number 1", "Number 2", ...]
   
   // reduce - combines all elements into a single value
   let sum = numbers.reduce(0) { $0 + $1 }              // 55 (adds all numbers)
   let product = numbers.reduce(1) { $0 * $1 }          // 3628800 (multiplies all)
   let concatenated = numbers.reduce("") { $0 + "\($1)," }  // "1,2,3,4,5,6,7,8,9,10,"
   
   // You can chain these methods together
   let result = numbers
       .filter { $0 % 2 == 0 }      // Keep only even numbers: [2, 4, 6, 8, 10]
       .map { $0 * $0 }             // Square them: [4, 16, 36, 64, 100]
       .reduce(0, +)                // Add them up: 220
   
   // More useful array methods
   let words = ["apple", "banana", "cherry", "date"]
   
   // first and last (return optionals)
   let firstWord = words.first      // Optional("apple")
   let lastWord = words.last        // Optional("date")
   
   // contains - check if element exists
   let hasApple = words.contains("apple")  // true
   
   // sorted - returns new sorted array
   let alphabetical = words.sorted()       // ["apple", "banana", "cherry", "date"]
   let byLength = words.sorted { $0.count < $1.count }  // ["date", "apple", "banana", "cherry"]
   
   // compactMap - transforms and removes nils
   let numberStrings = ["1", "2", "hello", "4", "world"]
   let validNumbers = numberStrings.compactMap { Int($0) }  // [1, 2, 4]
   
   // forEach - do something with each element (doesn't return anything)
   numbers.forEach { print("Processing \($0)") }
   
   // Real-world example: processing user data
   struct User {
       let name: String
       let age: Int
       let isActive: Bool
   }
   
   let users = [
       User(name: "Alice", age: 25, isActive: true),
       User(name: "Bob", age: 17, isActive: false),
       User(name: "Charlie", age: 30, isActive: true),
       User(name: "Diana", age: 16, isActive: true)
   ]
   
   // Get names of active adult users
   let activeAdultNames = users
       .filter { $0.isActive && $0.age >= 18 }  // Only active adults
       .map { $0.name }                         // Get just the names
       .sorted()                                // Sort alphabetically
   // Result: ["Alice", "Charlie"]
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

10. **Properties can store values, calculate values, or watch for changes.** There are three types: stored properties (hold actual values), computed properties (calculate values on-the-fly), and property observers (run code when values change).
    ```swift
    class Temperature {
        var celsius: Double = 0.0  // Stored property - actually holds a value
        
        // Computed property - calculates value each time it's accessed
        var fahrenheit: Double {
            get { 
                // This runs when someone reads the property: temp.fahrenheit
                return celsius * 9/5 + 32 
            }
            set { 
                // This runs when someone writes to the property: temp.fahrenheit = 100
                // 'newValue' is automatically provided by Swift - it's the value being assigned
                // You could also write: set(customName) and use 'customName' instead
                celsius = (newValue - 32) * 5/9 
            }
        }
        
        // Alternative setter syntax with custom parameter name
        var kelvin: Double {
            get { 
                return celsius + 273.15 
            }
            set(newKelvinValue) {  // Custom name instead of 'newValue'
                celsius = newKelvinValue - 273.15
            }
        }
        
        // Read-only computed property (no setter)
        var description: String {
            return "\(celsius)°C (\(fahrenheit)°F)"
        }
        
        // Property observer - runs when stored property changes
        var room: String = "Living Room" {
            willSet {
                // Runs BEFORE the value changes
                // 'newValue' here is the value about to be assigned
                print("About to move from \(room) to \(newValue)")
            }
            didSet {
                // Runs AFTER the value changes
                // 'oldValue' here is the previous value
                print("Moved from \(oldValue) to \(room)")
            }
        }
        
        // You can also use custom names for willSet/didSet
        var building: String = "Main Building" {
            willSet(newBuilding) {
                print("Will move to building: \(newBuilding)")
            }
            didSet(previousBuilding) {
                print("Moved from \(previousBuilding) to \(building)")
            }
        }
    }
    
    // How it works in practice:
    let temp = Temperature()
    
    // Setting celsius directly
    temp.celsius = 25.0  // Stored property - just stores the value
    
    // Reading fahrenheit (triggers getter)
    print(temp.fahrenheit)  // Prints: 77.0 (calculated from celsius)
    
    // Setting fahrenheit (triggers setter)
    temp.fahrenheit = 100.0  // Swift automatically passes 100.0 as 'newValue' to the setter
    print(temp.celsius)      // Prints: 37.777... (calculated from fahrenheit)
    
    // Setting kelvin (triggers setter with custom parameter name)
    temp.kelvin = 300.0      // Swift passes 300.0 as 'newKelvinValue'
    print(temp.celsius)      // Prints: 26.85 (calculated from kelvin)
    
    // Property observers in action
    temp.room = "Kitchen"    // Triggers willSet, then didSet
    // Output:
    // About to move from Living Room to Kitchen
    // Moved from Living Room to Kitchen
    
    // Why 'newValue' exists:
    // When you write: temp.fahrenheit = 100.0
    // Swift automatically creates a setter call like: setFahrenheit(newValue: 100.0)
    // The 'newValue' parameter contains whatever value was assigned (100.0 in this case)
    
    // Real-world example: User profile with validation
    class UserProfile {
        private var _email: String = ""  // Private stored property
        
        var email: String {
            get { 
                return _email 
            }
            set {
                // Validate email before storing
                if newValue.contains("@") && newValue.contains(".") {
                    _email = newValue.lowercased()  // Store in lowercase
                } else {
                    print("Invalid email: \(newValue)")
                    // Don't update _email if invalid
                }
            }
        }
        
        var username: String = "" {
            didSet {
                // Automatically update display name when username changes
                if !username.isEmpty {
                    displayName = "@\(username)"
                }
            }
        }
        
        var displayName: String = "Anonymous"
    }
    
    let profile = UserProfile()
    
    // Valid email
    profile.email = "USER@EXAMPLE.COM"  // newValue = "USER@EXAMPLE.COM"
    print(profile.email)  // Prints: "user@example.com" (converted to lowercase)
    
    // Invalid email
    profile.email = "invalid-email"     // newValue = "invalid-email"
    // Prints: "Invalid email: invalid-email"
    print(profile.email)  // Still prints: "user@example.com" (unchanged)
    
    // Property observer
    profile.username = "john_doe"       // Triggers didSet
    print(profile.displayName)          // Prints: "@john_doe"
    
    // Key points about 'newValue':
    // 1. It's automatically provided by Swift in setters and willSet
    // 2. The name 'newValue' is just a convention - you can use custom names
    // 3. It contains whatever value someone tried to assign to the property
    // 4. In willSet: newValue = the value about to be set
    // 5. In didSet: oldValue = the previous value (not newValue)
    ```

11. **Use `init` to set up your objects when they're created.** Initializers are special functions that prepare an instance of a class or struct for use. They ensure all properties have values before the object can be used.
    ```swift
    class Person {
        let name: String        // Must be set during initialization
        var age: Int           // Must be set during initialization
        var email: String?     // Optional, can be nil
        
        // Designated initializer - the main way to create a Person
        init(name: String, age: Int) {
            self.name = name   // 'self' refers to the instance being created
            self.age = age
            self.email = nil   // Optionals can be set to nil
            
            // You can do additional setup here
            print("Created person: \(name)")
        }
        
        // Convenience initializer - alternative way to create a Person
        // The 'convenience' keyword means this initializer provides a shortcut/easier way
        // to create objects by calling the main (designated) initializer with default values
        convenience init(name: String) {
            self.init(name: name, age: 0)  // Must call designated initializer
            print("Created person with default age")
        }
        
        // Another convenience initializer - provides different defaults
        convenience init(name: String, age: Int, email: String) {
            self.init(name: name, age: age)  // Call designated initializer first
            self.email = email               // Then set additional properties
        }
        
        // Why use convenience initializers?
        // 1. They provide shortcuts for common ways to create objects
        // 2. They reduce code duplication by reusing the main initializer
        // 3. They make your API easier to use by providing sensible defaults
        // 4. They must eventually call a designated initializer (the "real" one)
        
        // IMPORTANT: 'convenience' is ONLY for initializers, not other methods or properties!
        // You cannot write: convenience func someMethod() or convenience var someProperty
    }
    
    // About default values in classes vs structs:
    
    // Classes CAN have default values for properties
    class PersonWithDefaults {
        let name: String
        var age: Int = 0                    // Default value allowed
        var email: String? = nil            // Default value allowed
        var isActive: Bool = true           // Default value allowed
        var createdAt: Date = Date()        // Default value allowed
        
        // Even with defaults, you still need an initializer for non-optional properties without defaults
        init(name: String) {
            self.name = name  // 'name' has no default, so must be set in init
            // age, email, isActive, createdAt already have defaults
        }
        
        // You can override defaults in initializers
        init(name: String, age: Int) {
            self.name = name
            self.age = age      // Override the default of 0
            // Other properties keep their defaults
        }
    }
    
    // Structs are even more flexible with defaults
    struct PersonStruct {
        let name: String
        var age: Int = 0
        var email: String? = nil
        var isActive: Bool = true
        
        // Swift automatically creates this initializer for structs:
        // init(name: String, age: Int = 0, email: String? = nil, isActive: Bool = true)
        
        // You can also add custom initializers
        init(name: String) {
            self.name = name
            // All other properties use their defaults
        }
    }
    
    // Usage examples:
    let person1 = PersonWithDefaults(name: "Alice")                    // Uses defaults
    let person2 = PersonWithDefaults(name: "Bob", age: 25)             // Overrides age default
    
    let struct1 = PersonStruct(name: "Charlie")                        // Uses defaults
    let struct2 = PersonStruct(name: "Diana", age: 30)                 // Overrides age
    let struct3 = PersonStruct(name: "Eve", age: 25, email: "eve@example.com", isActive: false)  // Override multiple
    
    // Why use convenience initializers when you have default values?
    // 1. For complex initialization logic that can't be done with simple defaults
    // 2. When you need to call other methods during initialization
    // 3. For computed or derived values
    // 4. When working with inheritance (structs don't inherit)
    
    class SmartPerson {
        let name: String
        var age: Int = 0
        var email: String?
        var username: String = ""           // Default value
        var displayName: String = ""        // Default value
        
        // Designated initializer
        init(name: String, age: Int) {
            self.name = name
            self.age = age
            // email stays nil (default)
            // username and displayName get set by setupDefaults()
            setupDefaults()
        }
        
        // Convenience initializer with complex logic
        convenience init(name: String) {
            self.init(name: name, age: 0)    // Call designated initializer
            
            // Additional setup that can't be done with simple default values
            if name.contains(" ") {
                let parts = name.split(separator: " ")
                self.username = String(parts[0]).lowercased()
            } else {
                self.username = name.lowercased()
            }
            
            self.displayName = "User: \(name)"
        }
        
        // Convenience initializer for email signup
        convenience init(email: String) {
            // Extract name from email
            let namePart = email.split(separator: "@")[0]
            self.init(name: String(namePart).capitalized)
            self.email = email
        }
        
        private func setupDefaults() {
            if username.isEmpty {
                username = name.lowercased().replacingOccurrences(of: " ", with: "_")
            }
            if displayName.isEmpty {
                displayName = name
            }
        }
    }
    
    // Different ways to create SmartPerson:
    let smart1 = SmartPerson(name: "John Doe", age: 30)        // Designated init
    let smart2 = SmartPerson(name: "Jane Smith")               // Convenience with name logic
    let smart3 = SmartPerson(email: "bob@example.com")         // Convenience from email
    
    // Key differences:
    // - Default values: Simple, static values set when property is declared
    // - Convenience initializers: Complex logic, can call methods, can derive values
    // - You can have BOTH default values AND convenience initializers
    // - 'convenience' keyword is ONLY for initializers, never for other methods or properties
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

16. **Swift handles memory automatically using ARC (Automatic Reference Counting). Use `weak` to avoid retain cycles where objects hold onto each other.** Understanding retain cycles and how `weak` breaks them is crucial for preventing memory leaks.

    **What is ARC (Automatic Reference Counting)?**
    Swift automatically manages memory by counting how many references point to each object. When the reference count reaches zero, the object is deallocated (freed from memory).
    
    ```swift
    class Person {
        let name: String
        init(name: String) { 
            self.name = name 
            print("\(name) is being created")
        }
        deinit { 
            print("\(name) is being deallocated") 
        }
    }
    
    // Reference counting in action
    var person1: Person? = Person(name: "Alice")  // Reference count: 1
    var person2 = person1                         // Reference count: 2
    person1 = nil                                 // Reference count: 1 (still alive)
    person2 = nil                                 // Reference count: 0 (deallocated)
    // Output: "Alice is being created" then "Alice is being deallocated"
    ```
    
    **What is a Retain Cycle?**
    A retain cycle happens when two or more objects hold strong references to each other, creating a circular dependency. This prevents ARC from deallocating them because their reference counts never reach zero.
    
    ```swift
    // PROBLEMATIC CODE - Creates a retain cycle
    class Parent {
        let name: String
        var child: Child?  // Strong reference to Child
        
        init(name: String) { 
            self.name = name 
            print("Parent \(name) created")
        }
        deinit { print("Parent \(name) deallocated") }
    }
    
    class Child {
        let name: String
        var parent: Parent?  // Strong reference to Parent - THIS CAUSES THE CYCLE!
        
        init(name: String) { 
            self.name = name 
            print("Child \(name) created")
        }
        deinit { print("Child \(name) deallocated") }
    }
    
    // Creating the retain cycle
    func createRetainCycle() {
        let parent = Parent(name: "John")      // Parent reference count: 1
        let child = Child(name: "Emma")        // Child reference count: 1
        
        parent.child = child                   // Child reference count: 2
        child.parent = parent                  // Parent reference count: 2
        
        // When this function ends:
        // - Local 'parent' variable goes away: Parent reference count becomes 1
        // - Local 'child' variable goes away: Child reference count becomes 1
        // - But Parent still holds Child, and Child still holds Parent
        // - Neither can be deallocated! MEMORY LEAK!
    }
    
    createRetainCycle()
    // Output: "Parent John created" and "Child Emma created"
    // Missing: No deallocation messages - they're stuck in memory forever!
    ```
    
    **How `weak` References Break the Cycle**
    A `weak` reference doesn't increase the reference count. It can become `nil` automatically when the object it points to is deallocated.
    
    ```swift
    // SOLUTION - Using weak to break the cycle
    class Parent {
        let name: String
        var child: Child?  // Strong reference to Child (Parent "owns" Child)
        
        init(name: String) { 
            self.name = name 
            print("Parent \(name) created")
        }
        deinit { print("Parent \(name) deallocated") }
    }
    
    class Child {
        let name: String
        weak var parent: Parent?  // WEAK reference to Parent - breaks the cycle!
        
        init(name: String) { 
            self.name = name 
            print("Child \(name) created")
        }
        deinit { print("Child \(name) deallocated") }
    }
    
    // No more retain cycle
    func createHealthyRelationship() {
        let parent = Parent(name: "John")      // Parent reference count: 1
        let child = Child(name: "Emma")        // Child reference count: 1
        
        parent.child = child                   // Child reference count: 2
        child.parent = parent                  // Parent reference count: STILL 1 (weak doesn't count!)
        
        // When this function ends:
        // - Local 'parent' variable goes away: Parent reference count becomes 0
        // - Parent gets deallocated immediately
        // - Parent's 'child' property goes away: Child reference count becomes 1
        // - Local 'child' variable goes away: Child reference count becomes 0
        // - Child gets deallocated
        // - Child's weak 'parent' automatically becomes nil
    }
    
    createHealthyRelationship()
    // Output: "Parent John created", "Child Emma created", 
    //         "Parent John deallocated", "Child Emma deallocated"
    ```
    
    **Real-world Example: Apartment and Tenant**
    ```swift
    class Person {
        let name: String
        weak var apartment: Apartment?  // Person doesn't "own" the apartment
        
        init(name: String) {
            self.name = name
        }
        
        deinit {
            print("\(name) is moving out")
        }
    }
    
    class Apartment {
        let unit: String
        weak var tenant: Person?        // Apartment doesn't "own" the person either
        
        init(unit: String) {
            self.unit = unit
        }
        
        deinit {
            print("Apartment \(unit) is being demolished")
        }
    }
    
    // Usage - both can be deallocated properly
    func rentApartment() {
        let person = Person(name: "Alice")
        let apartment = Apartment(unit: "4B")
        
        person.apartment = apartment    // Weak reference
        apartment.tenant = person       // Weak reference
        
        print("\(person.name) lives in apartment \(apartment.unit)")
        // Both will be deallocated when function ends
    }
    ```
    
    **When to Use `weak` vs `strong` References**
    
    Use **strong** references when:
    - One object "owns" another (parent-child relationship)
    - You want to keep the object alive
    - The relationship is hierarchical (one-way dependency)
    
    Use **weak** references when:
    - Objects have a mutual relationship but neither "owns" the other
    - You want to avoid retain cycles
    - The relationship is temporary or optional
    - One object might outlive the other
    
    ```swift
    // OWNERSHIP EXAMPLES
    
    // Strong: ViewController owns its views
    class ViewController {
        let titleLabel = UILabel()     // Strong - VC owns the label
        let submitButton = UIButton()  // Strong - VC owns the button
    }
    
    // Weak: Delegate pattern (delegate doesn't own the delegator)
    protocol DataSourceDelegate: AnyObject {
        func dataDidUpdate()
    }
    
    class DataSource {
        weak var delegate: DataSourceDelegate?  // Weak - doesn't own delegate
        
        func fetchData() {
            // ... fetch data ...
            delegate?.dataDidUpdate()  // Might be nil, that's OK
        }
    }
    
    class ViewController: DataSourceDelegate {
        let dataSource = DataSource()  // Strong - VC owns data source
        
        override func viewDidLoad() {
            dataSource.delegate = self  // Weak reference back to VC
        }
        
        func dataDidUpdate() {
            // Update UI
        }
    }
    ```
    
    **Important Notes about `weak` References:**
    
    1. **Always optionals**: `weak` references are always optionals because they can become `nil`
    2. **Automatic nil-ing**: When the referenced object is deallocated, `weak` references automatically become `nil`
    3. **Only for classes**: You can only use `weak` with class instances, not structs or enums
    4. **Performance**: `weak` references have slight overhead compared to strong references
    
    ```swift
    class Manager {
        weak var assistant: Employee?  // Must be optional
    }
    
    class Employee {
        let name: String
        init(name: String) { self.name = name }
        deinit { print("\(name) quit") }
    }
    
    let manager = Manager()
    
    // Create employee in a scope
    do {
        let employee = Employee(name: "Bob")
        manager.assistant = employee        // Weak reference
        print(manager.assistant?.name)      // Prints "Optional("Bob")"
    }  // employee goes out of scope and is deallocated
    
    print(manager.assistant?.name)          // Prints "nil" - automatically nil-ed!
    // Output: "Bob quit" (from deinit)
    ```

17. **Use `[weak self]` in closures to avoid retain cycles.** Closures can capture and hold onto objects, creating retain cycles when the object also holds onto the closure. Understanding closure capture and how `[weak self]` breaks these cycles is essential for preventing memory leaks.

    **What is Closure Capture?**
    Closures automatically "capture" variables and objects from their surrounding context so they can use them later. By default, this creates strong references to captured objects.
    
    ```swift
    class DataProcessor {
        var name = "Processor"
        
        func startProcessing() {
            // This closure captures 'self' strongly
            DispatchQueue.global().async {
                // 'self' is captured here - strong reference!
                print("Processing data in \(self.name)")
                self.finishProcessing()
            }
        }
        
        func finishProcessing() {
            print("Processing complete")
        }
        
        deinit {
            print("\(name) is being deallocated")
        }
    }
    
    // This works fine for short-lived operations
    func quickExample() {
        let processor = DataProcessor()
        processor.startProcessing()
        // processor will be deallocated after async work completes
    }
    ```
    
    **The Problem: Retain Cycles with Long-Running Closures**
    When an object holds onto a closure that also captures the object strongly, you get a retain cycle. This commonly happens with timers, notifications, and long-running async operations.
    
    ```swift
    // PROBLEMATIC CODE - Creates a retain cycle
    class ViewController {
        var name = "Controller"
        var timer: Timer?
        
        func setupTimer() {
            // ViewController holds the timer (strong reference)
            timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
                // Timer's closure captures self strongly
                // Now: ViewController -> Timer -> Closure -> ViewController (CYCLE!)
                print("Timer fired in \(self.name)")
                self.updateUI()
            }
        }
        
        func updateUI() {
            print("Updating UI...")
        }
        
        deinit {
            print("\(name) is being deallocated")
            timer?.invalidate()  // This will never run because of the retain cycle!
        }
    }
    
    // Creating the retain cycle
    func createLeakyViewController() {
        let vc = ViewController()
        vc.setupTimer()
        // When this function ends, 'vc' should be deallocated
        // But it won't be because of the retain cycle!
        // The timer keeps running forever, holding onto the ViewController
    }
    
    createLeakyViewController()
    // Missing: "Controller is being deallocated" - it's stuck in memory!
    ```
    
    **The Solution: `[weak self]` Capture List**
    A capture list with `[weak self]` tells the closure to capture `self` weakly, breaking the retain cycle.
    
    ```swift
    // SOLUTION - Using [weak self] to break the cycle
    class ViewController {
        var name = "Controller"
        var timer: Timer?
        
        func setupTimer() {
            // ViewController holds the timer (strong reference)
            timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
                // Closure captures self weakly - no retain cycle!
                // self is now optional and might be nil
                guard let self = self else { 
                    print("ViewController was deallocated, stopping timer")
                    return 
                }
                print("Timer fired in \(self.name)")
                self.updateUI()
            }
        }
        
        func updateUI() {
            print("Updating UI...")
        }
        
        deinit {
            print("\(name) is being deallocated")
            timer?.invalidate()  // This will run properly now!
        }
    }
    
    // No more retain cycle
    func createHealthyViewController() {
        let vc = ViewController()
        vc.setupTimer()
        // When this function ends, vc can be deallocated
        // The timer's closure won't prevent deallocation
    }
    
    createHealthyViewController()
    // Output: "Controller is being deallocated" - memory is freed!
    ```
    
    **Common Patterns and Examples**
    
    **1. Network Requests**
    ```swift
    class APIClient {
        var name = "APIClient"
        
        func fetchData() {
            URLSession.shared.dataTask(with: URL(string: "https://api.example.com")!) { [weak self] data, response, error in
                // Without [weak self], this closure would keep APIClient alive
                // until the network request completes
                guard let self = self else {
                    print("APIClient was deallocated before request completed")
                    return
                }
                
                if let data = data {
                    self.processData(data)
                } else {
                    self.handleError(error)
                }
            }.resume()
        }
        
        func processData(_ data: Data) {
            print("\(name) processing \(data.count) bytes")
        }
        
        func handleError(_ error: Error?) {
            print("\(name) encountered error: \(error?.localizedDescription ?? "Unknown")")
        }
        
        deinit {
            print("\(name) deallocated")
        }
    }
    ```
    
    **2. Animation Completions**
    ```swift
    class AnimationController {
        var view = UIView()
        
        func animateView() {
            UIView.animate(withDuration: 2.0, animations: {
                // Animation block doesn't need [weak self] because it's short-lived
                self.view.alpha = 0.5
            }) { [weak self] completed in
                // Completion block needs [weak self] because it might run later
                guard let self = self else { return }
                
                if completed {
                    self.onAnimationComplete()
                }
            }
        }
        
        func onAnimationComplete() {
            print("Animation finished")
        }
    }
    ```
    
    **3. Notification Observers**
    ```swift
    class NotificationHandler {
        var name = "Handler"
        
        func setupNotifications() {
            NotificationCenter.default.addObserver(
                forName: .UIApplicationDidBecomeActive,
                object: nil,
                queue: .main
            ) { [weak self] notification in
                // Without [weak self], the notification center would keep
                // this object alive until you manually remove the observer
                guard let self = self else { return }
                self.handleAppBecameActive()
            }
        }
        
        func handleAppBecameActive() {
            print("\(name) handling app activation")
        }
        
        deinit {
            print("\(name) deallocated")
            // Don't forget to remove observers!
            NotificationCenter.default.removeObserver(self)
        }
    }
    ```
    
    **4. Combine Publishers**
    ```swift
    import Combine
    
    class DataManager: ObservableObject {
        @Published var items: [String] = []
        private var cancellables = Set<AnyCancellable>()
        var name = "DataManager"
        
        func fetchData() {
            URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com")!)
                .map(\.data)
                .decode(type: [String].self, decoder: JSONDecoder())
                .receive(on: DispatchQueue.main)
                .sink(
                    receiveCompletion: { [weak self] completion in
                        guard let self = self else { return }
                        self.handleCompletion(completion)
                    },
                    receiveValue: { [weak self] items in
                        guard let self = self else { return }
                        self.items = items
                        print("\(self.name) received \(items.count) items")
                    }
                )
                .store(in: &cancellables)
        }
        
        func handleCompletion(_ completion: Subscribers.Completion<Error>) {
            switch completion {
            case .finished:
                print("\(name) finished successfully")
            case .failure(let error):
                print("\(name) failed: \(error)")
            }
        }
        
        deinit {
            print("\(name) deallocated")
        }
    }
    ```
    
    **When You DON'T Need `[weak self]`**
    
    ```swift
    class QuickOperations {
        func performQuickTask() {
            // Short-lived operations that complete quickly don't need [weak self]
            DispatchQueue.main.async {
                self.updateUI()  // OK - this runs immediately
            }
            
            // Synchronous operations don't need [weak self]
            let result = [1, 2, 3].map { value in
                return self.processValue(value)  // OK - runs synchronously
            }
        }
        
        func updateUI() { }
        func processValue(_ value: Int) -> String { return "\(value)" }
    }
    ```
    
    **Alternative: `[unowned self]`**
    Sometimes you can use `[unowned self]` instead of `[weak self]`, but it's riskier:
    
    ```swift
    class RiskyExample {
        func setupCallback() {
            someAsyncOperation { [unowned self] in
                // unowned self is NOT optional - no guard let needed
                // BUT: if self is deallocated before this runs, your app will CRASH!
                self.handleResult()
            }
        }
        
        func handleResult() { }
    }
    
    // Safer approach with [weak self]
    class SafeExample {
        func setupCallback() {
            someAsyncOperation { [weak self] in
                guard let self = self else { return }  // Gracefully handle deallocation
                self.handleResult()
            }
        }
        
        func handleResult() { }
    }
    ```
    
    **Key Rules for `[weak self]`:**
    
    1. **Use `[weak self]` when**: The closure might outlive the object, or the object holds onto the closure
    2. **Don't use it when**: The closure runs immediately and synchronously
    3. **Always use `guard let self = self else { return }`** after `[weak self]` to safely unwrap
    4. **Prefer `[weak self]` over `[unowned self]`** - it's safer and won't crash if the object is deallocated
    5. **Remember**: `self` becomes optional inside the closure when using `[weak self]`
    
    ```swift
    // Template for using [weak self]
    someAsyncOperation { [weak self] result in
        guard let self = self else { 
            print("Object was deallocated")
            return 
        }
        
        // Now you can safely use self
        self.handleResult(result)
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

27a. **Labeled statements let you break out of specific loops or switch statements.** Just like Kotlin, Swift allows you to label loops and use those labels with `break` and `continue` to control exactly which loop you want to exit or skip.

    ```swift
    // BASIC LABELED BREAK - exit specific outer loop
    // Without labels, break only exits the innermost loop
    
    let numbers = 1...10
    
    // Problem: regular break only exits inner loop
    for i in numbers {
        for j in numbers {
            if i * j == 20 {
                print("Found: \(i) * \(j) = 20")
                break  // Only breaks inner loop, outer continues
            }
        }
    }
    
    // Solution: labeled break exits the specific loop you name
    outerLoop: for i in numbers {
        for j in numbers {
            if i * j == 20 {
                print("Found: \(i) * \(j) = 20")
                break outerLoop  // Breaks out of BOTH loops
            }
        }
    }
    
    // LABELED CONTINUE - skip to next iteration of specific loop
    searchLoop: for department in departments {
        for employee in department.employees {
            if employee.name == "John" {
                print("Found John in \(department.name)")
                continue searchLoop  // Skip to next department
            }
            print("Checking \(employee.name)")
        }
    }
    
    // REAL-WORLD EXAMPLE: Finding a combination
    let options = ["up", "down", "left", "right"]
    let secretCombination = ["up", "up", "right"]
    
    combinationSearch: for option1 in options {
        for option2 in options {
            for option3 in options {
                let attempt = [option1, option2, option3]
                print("Trying: \(attempt)")
                
                if attempt == secretCombination {
                    print("Found combination: \(attempt)")
                    break combinationSearch  // Exit all three loops at once
                }
            }
        }
    }
    
    // WITHOUT LABELS - messy with extra variables
    var found = false
    for option1 in options {
        if found { break }
        for option2 in options {
            if found { break }
            for option3 in options {
                let attempt = [option1, option2, option3]
                if attempt == secretCombination {
                    print("Found: \(attempt)")
                    found = true
                    break
                }
            }
        }
    }
    
    // LABELED STATEMENTS with different control flow
    
    // Example 1: Employee search across departments
    func findEmployee(name: String, in departments: [Department]) -> Employee? {
        departmentLoop: for department in departments {
            print("Searching in \(department.name)")
            
            for employee in department.employees {
                if employee.name == name {
                    print("Found \(name) in \(department.name)")
                    return employee  // Could also use break departmentLoop
                }
            }
            
            print("Not found in \(department.name)")
        }
        
        print("Employee \(name) not found anywhere")
        return nil
    }
    
    // Example 2: Processing data with early exit conditions
    func processMatrix(matrix: [[Int]]) {
        rowLoop: for (rowIndex, row) in matrix.enumerated() {
            columnLoop: for (colIndex, value) in row.enumerated() {
                
                // Skip negative values in this row
                if value < 0 {
                    print("Negative value at [\(rowIndex)][\(colIndex)], skipping row")
                    continue rowLoop
                }
                
                // Stop everything if we find a specific value
                if value == 999 {
                    print("Found terminator value 999, stopping all processing")
                    break rowLoop
                }
                
                // Skip just this column if value is zero
                if value == 0 {
                    print("Zero at [\(rowIndex)][\(colIndex)], skipping to next column")
                    continue columnLoop
                }
                
                print("Processing value \(value) at [\(rowIndex)][\(colIndex)]")
            }
        }
    }
    
    // LABELED STATEMENTS with switch statements
    func processCommands(commands: [String]) {
        commandLoop: for command in commands {
            switch command {
            case "start":
                print("Starting process...")
                
            case "data":
                // Process data, but if we find "stop", exit everything
                dataLoop: for i in 1...100 {
                    switch i {
                    case 50:
                        print("Halfway through data processing")
                    case 75:
                        print("Found critical error, stopping all commands")
                        break commandLoop  // Exit the outer command loop
                    default:
                        print("Processing data item \(i)")
                    }
                }
                
            case "stop":
                print("Stop command received")
                break commandLoop
                
            default:
                print("Unknown command: \(command)")
            }
        }
    }
    
    // NESTED SWITCH with labels
    func analyzeAnimal(animals: [Animal]) {
        animalLoop: for animal in animals {
            switch animal.species {
            case .dog:
                switch animal.name {
                case "Lucky":
                    print("Found our dog Lucky!")
                    break animalLoop  // Found what we wanted, stop searching
                default:
                    print("Found a dog named \(animal.name), but not Lucky")
                }
                
            case .cat:
                print("Found a cat named \(animal.name)")
                
            default:
                print("Found other animal: \(animal.species)")
            }
        }
    }
    
    // MULTIPLE LABELED LOOPS for complex algorithms
    func findPath(in maze: [[Character]]) -> [(Int, Int)]? {
        var path: [(Int, Int)] = []
        
        startSearch: for startRow in 0..<maze.count {
            for startCol in 0..<maze[startRow].count {
                
                // Only start from 'S' positions
                guard maze[startRow][startCol] == "S" else { continue }
                
                path = [(startRow, startCol)]
                
                // Try to find path from this starting point
                pathSearch: for step in 1...100 {  // Max 100 steps
                    let currentPos = path.last!
                    
                    // Check all four directions
                    directionLoop: for direction in [(0,1), (1,0), (0,-1), (-1,0)] {
                        let newRow = currentPos.0 + direction.0
                        let newCol = currentPos.1 + direction.1
                        
                        // Check bounds
                        guard newRow >= 0, newRow < maze.count,
                              newCol >= 0, newCol < maze[newRow].count else {
                            continue directionLoop
                        }
                        
                        let cell = maze[newRow][newCol]
                        
                        switch cell {
                        case "E":  // Found exit
                            path.append((newRow, newCol))
                            print("Found path with \(path.count) steps")
                            return path
                            
                        case ".":  // Open path
                            if !path.contains(where: { $0.0 == newRow && $0.1 == newCol }) {
                                path.append((newRow, newCol))
                                continue pathSearch
                            }
                            
                        case "#":  // Wall
                            continue directionLoop
                            
                        default:
                            continue directionLoop
                        }
                    }
                    
                    // Dead end, try next starting position
                    continue startSearch
                }
            }
        }
        
        return nil  // No path found
    }
    
    // WHEN TO USE LABELED STATEMENTS:
    // ✅ Nested loops where you need to exit multiple levels
    // ✅ Complex search algorithms with early termination
    // ✅ State machines with nested switch statements
    // ✅ Processing hierarchical data structures
    // ✅ When you want to avoid extra boolean variables
    
    // WHEN NOT TO USE:
    // ❌ Simple single loops (regular break/continue is fine)
    // ❌ When you can restructure code to avoid nesting
    // ❌ If it makes code harder to understand
    // ❌ Overusing them - they should be rare
    
    // COMPARISON WITH KOTLIN:
    // Swift: labelName: for item in items { break labelName }
    // Kotlin: labelName@ for (item in items) { break@labelName }
    // 
    // Both languages support the same concept, just different syntax:
    // - Swift uses colon (:) after label name
    // - Kotlin uses @ before label name and @ before label in break/continue
    // - Both support labeled break and continue
    // - Both work with loops and when/switch statements
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

28a. **Use `mutating` when a method in a struct or enum needs to modify the instance itself.** Swift structs and enums are value types, so by default their methods can't change their properties. The `mutating` keyword tells Swift "this method will change the instance."

    **Why `mutating` is needed:**
    Structs and enums are value types - when you pass them around, Swift makes copies. If methods could freely modify properties, it would be confusing which copy gets changed. `mutating` makes it explicit when a method will modify the instance.
    
    ```swift
    struct BankAccount {
        var balance: Double = 0.0
        let accountNumber: String
        
        // NON-MUTATING methods - can read but not modify properties
        func getBalance() -> Double {
            return balance  // ✅ Reading is allowed
        }
        
        func getAccountInfo() -> String {
            return "Account \(accountNumber): $\(balance)"  // ✅ Reading multiple properties
        }
        
        // MUTATING methods - can modify properties
        mutating func deposit(_ amount: Double) {
            balance += amount  // ✅ Modifying balance requires mutating
        }
        
        mutating func withdraw(_ amount: Double) -> Bool {
            guard balance >= amount else { return false }
            balance -= amount  // ✅ Modifying balance requires mutating
            return true
        }
        
        // This would be an ERROR without mutating:
        // func badDeposit(_ amount: Double) {
        //     balance += amount  // ❌ Error: Cannot assign to property
        // }
    }
    
    // Usage - the variable itself must be mutable (var) to call mutating methods
    var account = BankAccount(accountNumber: "12345")  // Must be 'var', not 'let'
    
    account.deposit(100.0)        // ✅ Works - account is var, method is mutating
    account.withdraw(50.0)        // ✅ Works - account is var, method is mutating
    print(account.getBalance())   // ✅ Works - non-mutating methods work on let or var
    
    // This would be an ERROR:
    let immutableAccount = BankAccount(accountNumber: "67890")
    // immutableAccount.deposit(100.0)  // ❌ Error: Cannot use mutating member on immutable value
    ```
    
    **Mutating with Enums - Changing Cases:**
    Enums can use `mutating` to change which case they represent.
    
    ```swift
    enum PlayerState {
        case idle
        case walking(speed: Double)
        case running(speed: Double)
        case jumping
        
        // Non-mutating - just reads current state
        func getCurrentSpeed() -> Double {
            switch self {
            case .idle, .jumping:
                return 0.0
            case .walking(let speed), .running(let speed):
                return speed
            }
        }
        
        // Mutating - changes the enum case
        mutating func startWalking(speed: Double) {
            self = .walking(speed: speed)  // ✅ Changing self requires mutating
        }
        
        mutating func startRunning() {
            switch self {
            case .walking(let speed):
                self = .running(speed: speed * 2)  // Double the walking speed
            case .idle:
                self = .running(speed: 5.0)        // Default running speed
            default:
                self = .running(speed: 5.0)
            }
        }
        
        mutating func stop() {
            self = .idle  // ✅ Changing to idle state
        }
        
        mutating func jump() {
            self = .jumping  // ✅ Change to jumping, regardless of current state
        }
    }
    
    // Usage
    var player = PlayerState.idle
    player.startWalking(speed: 2.0)     // ✅ Changes to .walking(speed: 2.0)
    player.startRunning()               // ✅ Changes to .running(speed: 4.0)
    player.jump()                       // ✅ Changes to .jumping
    player.stop()                       // ✅ Changes to .idle
    ```
    
    **Mutating in Protocols:**
    When you define a protocol, you must mark methods as `mutating` if they might modify the conforming type.
    
    ```swift
    protocol Resettable {
        mutating func reset()           // Conforming types might need to modify themselves
        func getCurrentState() -> String // Non-mutating - just reads state
    }
    
    struct Counter: Resettable {
        var count: Int = 0
        
        mutating func reset() {         // Must be mutating to modify count
            count = 0
        }
        
        func getCurrentState() -> String {
            return "Count: \(count)"
        }
        
        mutating func increment() {
            count += 1
        }
    }
    
    class ReferenceCounter: Resettable {
        var count: Int = 0
        
        func reset() {                  // Classes don't need mutating (reference types)
            count = 0
        }
        
        func getCurrentState() -> String {
            return "Reference Count: \(count)"
        }
    }
    ```
    
    **Common Patterns with Mutating:**
    
    ```swift
    // Pattern 1: Collection-like structs
    struct Stack<Element> {
        private var items: [Element] = []
        
        mutating func push(_ item: Element) {
            items.append(item)          // Modifying internal array
        }
        
        mutating func pop() -> Element? {
            return items.popLast()      // Modifying internal array
        }
        
        func peek() -> Element? {       // Non-mutating - just reading
            return items.last
        }
        
        var count: Int {                // Computed property - non-mutating
            return items.count
        }
        
        var isEmpty: Bool {             // Computed property - non-mutating
            return items.isEmpty
        }
    }
    
    // Pattern 2: State machines
    struct GameState {
        var score: Int = 0
        var lives: Int = 3
        var level: Int = 1
        var isGameOver: Bool = false
        
        mutating func addPoints(_ points: Int) {
            score += points
            
            // Level up every 1000 points
            if score >= level * 1000 {
                level += 1
            }
        }
        
        mutating func loseLife() {
            lives -= 1
            if lives <= 0 {
                isGameOver = true
            }
        }
        
        mutating func resetGame() {
            score = 0
            lives = 3
            level = 1
            isGameOver = false
        }
        
        // Non-mutating helper methods
        func canContinue() -> Bool {
            return !isGameOver && lives > 0
        }
        
        func getDisplayText() -> String {
            return "Score: \(score) | Lives: \(lives) | Level: \(level)"
        }
    }
    
    // Pattern 3: Builder pattern with structs
    struct URLBuilder {
        private var scheme: String = "https"
        private var host: String = ""
        private var path: String = ""
        private var queryItems: [String: String] = [:]
        
        mutating func setScheme(_ scheme: String) {
            self.scheme = scheme
        }
        
        mutating func setHost(_ host: String) {
            self.host = host
        }
        
        mutating func setPath(_ path: String) {
            self.path = path
        }
        
        mutating func addQueryItem(key: String, value: String) {
            queryItems[key] = value
        }
        
        // Non-mutating - builds the final URL
        func build() -> URL? {
            var components = URLComponents()
            components.scheme = scheme
            components.host = host
            components.path = path
            
            if !queryItems.isEmpty {
                components.queryItems = queryItems.map { URLQueryItem(name: $0.key, value: $0.value) }
            }
            
            return components.url
        }
    }
    
    // Usage of builder pattern
    var builder = URLBuilder()
    builder.setHost("api.example.com")
    builder.setPath("/users")
    builder.addQueryItem(key: "page", value: "1")
    builder.addQueryItem(key: "limit", value: "10")
    
    if let url = builder.build() {
        print("Built URL: \(url)")  // https://api.example.com/users?page=1&limit=10
    }
    ```
    
    **When You DON'T Need Mutating:**
    
    ```swift
    struct ReadOnlyData {
        let id: String
        let createdAt: Date
        private let items: [String]
        
        // All these methods are non-mutating because they don't change anything
        func getItemCount() -> Int {
            return items.count
        }
        
        func getItem(at index: Int) -> String? {
            guard index >= 0 && index < items.count else { return nil }
            return items[index]
        }
        
        func contains(_ item: String) -> Bool {
            return items.contains(item)
        }
        
        func getAllItems() -> [String] {
            return items  // Returns a copy, doesn't modify original
        }
        
        // Even complex operations don't need mutating if they don't change the struct
        func getFilteredItems(containing text: String) -> [String] {
            return items.filter { $0.contains(text) }
        }
    }
    
    // Classes never need mutating because they're reference types
    class MutableClass {
        var value: Int = 0
        
        func increment() {              // No mutating needed for classes
            value += 1
        }
        
        func reset() {                  // No mutating needed for classes
            value = 0
        }
    }
    ```
    
    **Key Rules for Mutating:**
    
    1. **Structs and Enums only**: Classes never need `mutating` because they're reference types
    2. **Property modification**: If a method changes any stored property, it needs `mutating`
    3. **Self assignment**: If a method assigns to `self`, it needs `mutating`
    4. **Calling mutating methods**: If a method calls another `mutating` method, it must also be `mutating`
    5. **Variable requirement**: You can only call `mutating` methods on `var` instances, not `let`
    6. **Protocol conformance**: If a protocol method might modify conforming types, mark it `mutating`
    
    ```swift
    struct Example {
        var data: [Int] = []
        
        // Needs mutating - modifies stored property
        mutating func addItem(_ item: Int) {
            data.append(item)
        }
        
        // Needs mutating - calls mutating method
        mutating func addMultipleItems(_ items: [Int]) {
            for item in items {
                addItem(item)  // Calling mutating method requires this to be mutating
            }
        }
        
        // Needs mutating - assigns to self
        mutating func replaceWith(_ newData: [Int]) {
            self = Example(data: newData)  // Assigning to self requires mutating
        }
        
        // Doesn't need mutating - only reads data
        func getCount() -> Int {
            return data.count
        }
        
        // Doesn't need mutating - returns new instance without modifying self
        func withAdditionalItem(_ item: Int) -> Example {
            var copy = self
            copy.data.append(item)
            return copy
        }
    }
    ```

17a. **How closures capture variables (not just self), and what happens to their memory**

When you create a closure in Swift, it automatically "captures" any variables or objects it uses from the surrounding scope. This means the closure keeps a reference to those variables, so it can use them later—even if the original scope is gone. This is called *closure capture*.

**Key point:**
- By default, closures capture variables *strongly*. This means the closure keeps the captured object alive as long as the closure itself is alive.
- This is true for *any* object or variable the closure uses—not just `self`.

**What does this mean for memory?**
- If a closure is stored somewhere (like in a property, array, or as a callback), any objects it captures will NOT be deallocated until the closure itself is released.
- This can cause memory leaks or keep objects alive longer than you expect, especially if the closure is long-lived.

**Example: Closure capturing multiple objects**

Let's see what happens when a closure captures several objects:

```swift
class Owner {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("Owner \(name) deallocated") }
}

class Pet {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("Pet \(name) deallocated") }
}

var owner: Owner? = Owner(name: "Alice")   // Create Owner object
var pet: Pet? = Pet(name: "Fluffy")        // Create Pet object

// Create a closure that captures both owner and pet
let closure = {
    // The closure uses both objects, so it captures them
    print("Owner: \(owner!.name), Pet: \(pet!.name)")
}

owner = nil   // Try to release Owner
pet = nil     // Try to release Pet
// Both objects are still alive, because 'closure' holds strong references

closure()     // Prints: Owner: Alice, Pet: Fluffy
// Only when 'closure' is released will Owner and Pet be deallocated
```

**Line-by-line explanation:**
- `class Owner` and `class Pet`: Simple classes with a name and a deinit to show when they're deallocated.
- `var owner: Owner? = ...` and `var pet: Pet? = ...`: Create two objects.
- `let closure = { ... }`: The closure uses both `owner` and `pet`, so it captures them *strongly*.
- `owner = nil` and `pet = nil`: Set the original variables to nil. But the objects are NOT deallocated, because the closure still holds them.
- `closure()`: The closure can still use both objects.
- Only when the closure itself is released (e.g., goes out of scope or is set to nil) will the captured objects be deallocated.

**How to avoid keeping objects alive?**
- If you want the closure to NOT keep a strong reference to an object, you can use a *capture list* with `weak` or `unowned`, just like you do with `self`.
- This works for ANY object, not just `self`.

**Example: Using [weak owner, weak pet] in a closure**

```swift
var owner: Owner? = Owner(name: "Alice")
var pet: Pet? = Pet(name: "Fluffy")

let closure: () -> Void = { [weak owner, weak pet] in
    // owner and pet are now optionals inside the closure
    if let owner = owner, let pet = pet {
        print("Owner: \(owner.name), Pet: \(pet.name)")
    } else {
        print("Owner or pet is gone")
    }
}

owner = nil   // Now Owner can be deallocated
pet = nil     // Now Pet can be deallocated

closure()     // Prints: Owner or pet is gone
```

**Line-by-line explanation:**
- `[weak owner, weak pet]`: The closure captures both variables weakly. Now, the closure does NOT keep them alive.
- Inside the closure, `owner` and `pet` are optionals (they might be nil if deallocated).
- The closure checks if both are still alive before using them.
- When you set `owner = nil` and `pet = nil`, the objects are deallocated immediately (if nothing else holds them).
- When you call the closure, it prints "Owner or pet is gone" because both are nil.

**Summary:**
- Closures capture *all* variables they use, not just `self`.
- By default, this is a strong reference, which keeps the object alive.
- To avoid memory leaks or unwanted retention, use `[weak ...]` or `[unowned ...]` in the closure's capture list for *any* object you don't want to keep alive.
- Always check for nil if you use `weak` (since the object might be gone).
- Use `unowned` only if you are sure the object will always be alive when the closure runs (otherwise, your app will crash).

**When to worry about this:**
- If your closure is stored somewhere (property, array, callback, etc.) and might outlive the objects it uses, always consider how it captures those objects.
- If the closure is short-lived (runs immediately), you usually don't need to worry.

**Template for capturing multiple objects weakly:**
```swift
let closure = { [weak obj1, weak obj2] in
    guard let obj1 = obj1, let obj2 = obj2 else { return }
    // Use obj1 and obj2 safely
}
```

## Swift Package Manager (SPM) - No Xcode Needed

29. Create a new Swift project from terminal using `swift package init`. This creates all the files and folders you need.
    ```bash
    # Create executable project (makes an app you can run)
    swift package init --type executable --name MyApp
    
    # Create library project (makes code others can import)
    swift package init --type library --name MyLibrary
    
    # Create empty package (you decide what to put in it)
    swift package init
    
    # You can also create in a specific directory
    mkdir MyProject && cd MyProject
    swift package init --type executable
    ```
    
    After running this, you get a complete project structure:
    ```
    MyApp/
    ├── Package.swift          // Project configuration
    ├── README.md             // Documentation
    ├── .gitignore           // Git ignore file
    ├── Sources/
    │   └── MyApp/
    │       └── main.swift    // Your app's entry point
    └── Tests/
        └── MyAppTests/
            └── MyAppTests.swift  // Your tests
    ```

30. **Package.swift** is your project config file. It's like a recipe that tells Swift how to build your project.
    ```swift
    // swift-tools-version: 5.8  // Minimum Swift version needed
    import PackageDescription
    
    let package = Package(
        name: "MyApp",                    // Name of your package
        platforms: [.macOS(.v12)],        // Which platforms it supports (optional)
        products: [
            // Products are what gets built - executables or libraries
            .executable(name: "MyApp", targets: ["MyApp"]),
        ],
        dependencies: [
            // External packages your project uses
            .package(url: "https://github.com/apple/swift-argument-parser", from: "1.0.0"),
            .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        ],
        targets: [
            // Targets are groups of source files
            .executableTarget(
                name: "MyApp",            // This target builds the main app
                dependencies: [
                    .product(name: "ArgumentParser", package: "swift-argument-parser"),
                    .product(name: "Vapor", package: "vapor"),
                ]
            ),
            .testTarget(
                name: "MyAppTests",       // This target builds the tests
                dependencies: ["MyApp"]   // Tests depend on the main app
            ),
        ]
    )
    ```
    
    Each part does something specific:
    - **name**: What your package is called
    - **platforms**: Which OS versions you support (iOS 14+, macOS 11+, etc.)
    - **products**: What gets built (apps, libraries)
    - **dependencies**: External code you want to use  
    - **targets**: Groups of your source files

31. **Products** are what your package builds - the end result people can use.
    ```swift
    products: [
        // Executable product - makes a command-line app
        .executable(name: "MyApp", targets: ["MyApp"]),
        
        // Library product - makes code others can import
        .library(name: "MyLibrary", targets: ["MyLibrary"]),
        
        // You can have multiple products from one package
        .executable(name: "AdminTool", targets: ["AdminTool"]),
        .library(name: "SharedUtils", targets: ["Utils", "Helpers"]),
    ]
    ```
    
    **Executable** products:
    - Can be run from command line: `swift run MyApp`
    - Have a `main.swift` file or `@main` struct
    - Good for CLI tools, servers, scripts
    
    **Library** products:
    - Other packages can import them: `import MyLibrary`
    - Can't be run directly - they're code for other code
    - Good for reusable components, frameworks
    
    One package can make multiple products. For example, a web framework might have:
    - Library for the framework itself
    - Executable for code generation tools

32. **Targets** are groups of source files. Each target becomes a module you can import.
    ```swift
    targets: [
        // Executable target - has a main.swift or @main
        .executableTarget(
            name: "MyApp",
            dependencies: ["Networking", "Database"],  // Uses other targets
            path: "Sources/MyApp",                     // Where the source files are (optional)
            exclude: ["README.md", "old_code/"],       // Files to ignore (optional)
            sources: ["main.swift", "AppLogic.swift"], // Specific files (optional)
            resources: [.copy("config.json")]          // Include data files (optional)
        ),
        
        // Regular target - library code
        .target(
            name: "Networking",
            dependencies: [
                .product(name: "AsyncHTTPClient", package: "async-http-client")
            ]
        ),
        
        // Another target
        .target(name: "Database"),
        
        // Test target - tests your code
        .testTarget(
            name: "MyAppTests",
            dependencies: ["MyApp", "Networking"],     // What it tests
            resources: [.copy("test_data.json")]       // Test data files
        ),
    ]
    ```
    
    Target types:
    - **executableTarget**: Makes an executable, needs main.swift or @main
    - **target**: Makes a library/module, no main.swift  
    - **testTarget**: Contains tests, runs with `swift test`
    
    Each target gets its own folder under Sources/ with the same name.

33. **Dependencies** are external packages you want to use. SPM downloads and builds them for you.
    ```swift
    dependencies: [
        // GitHub packages - most common
        .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        .package(url: "https://github.com/apple/swift-nio.git", from: "2.0.0"),
        
        // You can use different version requirements
        .package(url: "https://github.com/user/repo.git", .upToNextMajor(from: "1.0.0")),  // 1.x.x
        .package(url: "https://github.com/user/repo.git", .upToNextMinor(from: "1.2.0")), // 1.2.x
        .package(url: "https://github.com/user/repo.git", exact: "2.1.5"),                // Exactly 2.1.5
        .package(url: "https://github.com/user/repo.git", "1.0.0"..<"3.0.0"),            // Between versions
        
        // Development versions
        .package(url: "https://github.com/user/repo.git", branch: "main"),               // Latest from main
        .package(url: "https://github.com/user/repo.git", revision: "abc123def"),        // Specific commit
        
        // Local packages (for development)
        .package(path: "../MyLocalPackage"),           // Relative path
        .package(path: "/Users/me/Code/MyPackage"),    // Absolute path
    ],
    ```
    
    After adding dependencies, you use them in targets:
    ```swift
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "Vapor", package: "vapor"),        // Use specific product
            .product(name: "NIO", package: "swift-nio"),
            "SomeLocalTarget",                                // Use target from same package
        ]
    )
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

35. Build and run your project from terminal. SPM handles all the compilation.
    ```bash
    # Build the project (debug mode by default)
    swift build
    
    # Build for release (optimized, takes longer)
    swift build -c release
    
    # Run the executable (builds first if needed)
    swift run                    # Runs the default executable
    swift run MyApp              # Runs specific executable if you have multiple
    swift run MyApp arg1 arg2    # Pass arguments to your app
    
    # Run tests
    swift test                   # Runs all tests
    swift test --filter MyAppTests.testSomething  # Run specific test
    swift test --parallel        # Run tests in parallel (faster)
    
    # Clean build artifacts
    swift package clean
    
    # Show package info
    swift package describe       # Shows package structure
    swift package show-dependencies  # Shows dependency tree
    
    # Generate Xcode project (if you want to use Xcode)
    swift package generate-xcodeproj
    ```
    
    Build outputs go to `.build/debug/` or `.build/release/`:
    ```bash
    # You can run the built executable directly
    ./.build/debug/MyApp
    
    # Or the release version
    ./.build/release/MyApp
    ```

36. Add dependencies by editing Package.swift, then let SPM download them:
    ```bash
    # After adding dependencies to Package.swift, resolve them
    swift package resolve          # Downloads and locks dependency versions
    
    # Update dependencies to latest allowed versions
    swift package update           # Updates within your version constraints
    swift package update MyPackage # Update just one dependency
    
    # Reset dependencies (delete and re-download)
    swift package reset
    
    # See what versions are being used
    swift package show-dependencies
    ```
    
    SPM creates these files to track dependencies:
    ```
    Package.resolved      # Exact versions being used (commit this to git)
    .build/               # Build artifacts (don't commit this)
    ```
    
    **Package.resolved** is important - it locks exact versions so everyone gets the same dependencies:
    ```json
    {
      "version": 2,
      "dependencies": [
        {
          "identity": "vapor",
          "kind": "remoteSourceControl",
          "location": "https://github.com/vapor/vapor.git",
          "state": {
            "revision": "abc123...",
            "version": "4.65.2"
          }
        }
      ]
    }
    ```

37. **Modules** are what you import. Each target becomes a module you can use in other code.
    ```swift
    // Import your own targets/modules
    import Networking     // If you have a target named "Networking"
    import Database       // If you have a target named "Database"
    
    // Import external package products
    import Vapor          // From vapor dependency
    import ArgumentParser // From swift-argument-parser dependency
    
    // System modules (built into Swift/platform)
    import Foundation     // Basic types, networking, file system
    import SwiftUI        // UI framework (Apple platforms)
    import Dispatch       // Grand Central Dispatch
    ```
    
    Module visibility rules:
    ```swift
    // In your Networking target
    public class APIClient {        // Available to other modules
        internal func setup() {}    // Only available within Networking module
        private func validate() {}  // Only available within this file
    }
    
    // In your main app
    import Networking
    let client = APIClient()        // Works - it's public
    client.setup()                  // Error - setup() is internal
    ```
    
    **public**: Available to anyone who imports your module
    **internal**: Available only within the same module (default)
    **private**: Available only within the same file

38. Create multiple targets to organize your code into logical pieces:
    ```swift
    targets: [
        // Main app uses other targets
        .executableTarget(
            name: "MyApp", 
            dependencies: ["Networking", "Database", "Utils"]
        ),
        
        // Networking module - handles API calls
        .target(
            name: "Networking",
            dependencies: [
                .product(name: "AsyncHTTPClient", package: "async-http-client")
            ]
        ),
        
        // Database module - handles data storage
        .target(
            name: "Database",
            dependencies: [
                .product(name: "SQLite", package: "sqlite.swift")
            ]
        ),
        
        // Utility functions used by multiple modules
        .target(name: "Utils"),
        
        // Tests for each module
        .testTarget(name: "NetworkingTests", dependencies: ["Networking"]),
        .testTarget(name: "DatabaseTests", dependencies: ["Database"]),
        .testTarget(name: "MyAppTests", dependencies: ["MyApp"]),
    ]
    ```
    
    Benefits of multiple targets:
    - **Separation**: Each target has a clear purpose
    - **Reusability**: Other projects can import just what they need
    - **Testing**: Test each component separately
    - **Build time**: Only rebuild what changed

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

## Swift Package Manager vs Java Build Systems (Maven/Gradle)

**If you're coming from Java, here's how Swift Package Manager concepts map to what you already know:**

44. **Package.swift is like pom.xml or build.gradle** - it's your project configuration file that defines everything about your project.
    ```swift
    // Package.swift (Swift)
    let package = Package(
        name: "MyApp",                    // Like <artifactId> in Maven
        platforms: [.macOS(.v12)],        // Like <properties><maven.compiler.target>
        products: [                       // What gets built (like Maven goals)
            .executable(name: "MyApp", targets: ["MyApp"]),
            .library(name: "Utils", targets: ["Utils"]),
        ],
        dependencies: [                   // Like <dependencies> in Maven
            .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        ],
        targets: [                        // Like <modules> in Maven multi-module projects
            .executableTarget(name: "MyApp", dependencies: ["Utils"]),
            .target(name: "Utils"),
        ]
    )
    ```
    
    ```xml
    <!-- pom.xml (Maven equivalent) -->
    <project>
        <artifactId>MyApp</artifactId>
        <properties>
            <maven.compiler.target>17</maven.compiler.target>
        </properties>
        <dependencies>
            <dependency>
                <groupId>com.vapor</groupId>
                <artifactId>vapor</artifactId>
                <version>4.0.0</version>
            </dependency>
        </dependencies>
        <modules>
            <module>MyApp</module>
            <module>Utils</module>
        </modules>
    </project>
    ```

45. **Products are like Maven artifacts** - they're what gets built and can be used by others.
    ```swift
    // Swift Package Manager
    products: [
        .executable(name: "MyApp", targets: ["MyApp"]),        // Like jar with main class
        .library(name: "MyLibrary", targets: ["MyLibrary"]),   // Like regular jar library
        .library(name: "Utils", type: .static, targets: ["Utils"]), // Like jar with dependencies included
    ]
    ```
    
    **Java equivalent:**
    - **Executable product** = JAR with main class (can run with `java -jar myapp.jar`)
    - **Library product** = Regular JAR that other projects can depend on
    - **Static library** = Fat JAR with all dependencies bundled (like Maven shade plugin)
    
    ```xml
    <!-- Maven: Creating executable JAR -->
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    
    <!-- Maven: Creating library JAR (default) -->
    <packaging>jar</packaging>
    
    <!-- Maven: Creating fat JAR -->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
    </plugin>
    ```

46. **Targets are like Maven modules or Gradle subprojects** - they're separate pieces of your codebase that can depend on each other.
    ```swift
    // Swift Package Manager targets
    targets: [
        .executableTarget(
            name: "MyApp",                    // Main application
            dependencies: ["Database", "API"] // Uses other targets
        ),
        .target(
            name: "Database",                 // Database layer
            dependencies: [
                .product(name: "SQLite", package: "sqlite.swift")
            ]
        ),
        .target(name: "API"),                 // API layer
        .testTarget(
            name: "MyAppTests",               // Test module
            dependencies: ["MyApp"]
        ),
    ]
    ```
    
    **Java Maven equivalent:**
    ```xml
    <!-- Parent pom.xml -->
    <modules>
        <module>myapp-main</module>      <!-- Main application -->
        <module>myapp-database</module>  <!-- Database layer -->
        <module>myapp-api</module>       <!-- API layer -->
    </modules>
    
    <!-- myapp-main/pom.xml -->
    <dependencies>
        <dependency>
            <groupId>com.mycompany</groupId>
            <artifactId>myapp-database</artifactId>  <!-- Uses other modules -->
        </dependency>
        <dependency>
            <groupId>com.mycompany</groupId>
            <artifactId>myapp-api</artifactId>
        </dependency>
    </dependencies>
    
    <!-- myapp-database/pom.xml -->
    <dependencies>
        <dependency>
            <groupId>org.xerial</groupId>
            <artifactId>sqlite-jdbc</artifactId>    <!-- External dependency -->
        </dependency>
    </dependencies>
    ```

47. **Directory structure comparison** - Swift is simpler and more standardized.
    ```
    Swift Package Manager:
    MyApp/
    ├── Package.swift              // Project config (like pom.xml)
    |   ├── Sources/                   // All source code goes here
    |   │   ├── MyApp/                 // Main app target
    |   │   │   └── main.swift
    |   ├── Database/              // Database target
    |   │   └── DatabaseManager.swift
    |   └── API/                   // API target
    |       └── APIClient.swift
    └── Tests/                     // All tests go here
        ├── MyAppTests/
        ├── DatabaseTests/
        └── APITests/
    
    Java Maven:
    MyApp/
    ├── pom.xml                    // Project config
    ├── myapp-main/                // Main module
    │   ├── pom.xml
    │   └── src/
    │       ├── main/java/         // Source code
    │       └── test/java/         // Tests
    ├── myapp-database/            // Database module
    │   ├── pom.xml
    │   └── src/
    │       ├── main/java/
    │       └── test/java/
    └── myapp-api/                 // API module
        ├── pom.xml
        └── src/
            ├── main/java/
            └── test/java/
    ```

48. **Dependency management comparison** - both use semantic versioning but syntax differs.
    ```swift
    // Swift Package Manager
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),      // 4.0.0+
        .package(url: "https://github.com/user/repo.git", "1.0.0"..<"2.0.0"),   // 1.x.x
        .package(url: "https://github.com/user/repo.git", exact: "1.5.0"),      // Exactly 1.5.0
        .package(url: "https://github.com/user/repo.git", branch: "main"),      // Latest from branch
        .package(path: "../MyLocalPackage"),                                    // Local dependency
    ]
    ```
    
    ```xml
    <!-- Maven -->
    <dependencies>
        <dependency>
            <groupId>com.vapor</groupId>
            <artifactId>vapor</artifactId>
            <version>[4.0.0,)</version>        <!-- 4.0.0+ -->
        </dependency>
        <dependency>
            <groupId>com.user</groupId>
            <artifactId>repo</artifactId>
            <version>[1.0.0,2.0.0)</version>   <!-- 1.x.x -->
        </dependency>
        <dependency>
            <groupId>com.user</groupId>
            <artifactId>repo</artifactId>
            <version>1.5.0</version>           <!-- Exactly 1.5.0 -->
        </dependency>
        <dependency>
            <groupId>com.mycompany</groupId>
            <artifactId>MyLocalPackage</artifactId>
            <version>1.0-SNAPSHOT</version>    <!-- Local/snapshot -->
        </dependency>
    </dependencies>
    ```

49. **Build commands comparison** - Swift commands are simpler and more consistent.
    ```bash
    # Swift Package Manager
    swift build                    # Compile (like mvn compile)
    swift build -c release        # Release build (like mvn package)
    swift run                      # Run main executable (like mvn exec:java)
    swift test                     # Run tests (like mvn test)
    swift package clean            # Clean build artifacts (like mvn clean)
    swift package resolve         # Download dependencies (like mvn dependency:resolve)
    swift package update           # Update dependencies (like mvn versions:use-latest-versions)
    
    # Maven
    mvn compile                    # Compile source code
    mvn package                    # Create JAR/WAR
    mvn exec:java -Dexec.mainClass="com.example.Main"  # Run main class
    mvn test                       # Run tests
    mvn clean                      # Clean build artifacts
    mvn dependency:resolve         # Download dependencies
    mvn versions:use-latest-versions  # Update dependencies
    
    # Gradle
    ./gradlew build               # Build project
    ./gradlew run                 # Run application
    ./gradlew test                # Run tests
    ./gradlew clean               # Clean build
    ```

50. **Module system comparison** - Swift modules are simpler than Java modules.
    ```swift
    // Swift: Each target becomes a module automatically
    // In Database target:
    public class DatabaseManager {     // Available to other modules
        internal func setup() {}       // Only within Database module
        private func validate() {}     // Only within this file
    }
    
    // In main app:
    import Database                    // Import the module
    let db = DatabaseManager()        // Use public classes
    ```
    
    ```java
    // Java: Need explicit module-info.java (Java 9+)
    // In database module (module-info.java):
    module myapp.database {
        exports com.myapp.database;    // Make package available
        requires java.sql;             // Depend on other modules
    }
    
    // In main app module:
    module myapp.main {
        requires myapp.database;       // Import the module
    }
    
    // In main app code:
    import com.myapp.database.DatabaseManager;  // Import specific classes
    DatabaseManager db = new DatabaseManager();
    ```

51. **Testing comparison** - Swift has built-in testing, Java needs frameworks.
    ```swift
    // Swift: Built-in XCTest framework
    import XCTest
    @testable import MyApp         // @testable gives access to internal members
    
    final class MyAppTests: XCTestCase {
        func testExample() {
            XCTAssertEqual(add(2, 3), 5)
        }
    }
    ```
    
    ```java
    // Java: Need JUnit dependency
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>
    
    // Java test code:
    import org.junit.jupiter.api.Test;
    import static org.junit.jupiter.api.Assertions.assertEquals;
    
    class MyAppTest {
        @Test
        void testExample() {
            assertEquals(5, add(2, 3));
        }
    }
    ```

52. **Key differences summary:**
    
    | Aspect | Swift Package Manager | Java (Maven/Gradle) |
    |--------|----------------------|---------------------|
    | **Config file** | Package.swift (code) | pom.xml/build.gradle (XML/DSL) |
    | **Directory structure** | Fixed, simple | Flexible, more complex |
    | **Modules** | Targets = modules | Explicit module system |
    | **Dependencies** | Git URLs, semantic versioning | Central repositories, coordinates |
    | **Build outputs** | Products (executable/library) | Artifacts (JAR/WAR/EAR) |
    | **Testing** | Built-in XCTest | External frameworks (JUnit) |
    | **IDE integration** | Generate Xcode project | Native IDE support |
    | **Platforms** | Apple + Linux | Cross-platform JVM |
    
    **When to use each:**
    - **Swift Package Manager**: Apple ecosystem, command-line tools, simple projects
    - **Maven/Gradle**: Enterprise Java, complex builds, extensive plugin ecosystem

## Interfacing Swift with Other Languages

44. Swift can call C code directly. Just import the C library and use it. Foundation already imports lots of C functions.
    ```swift
    import Foundation  // Has tons of C functions like malloc, free, strcpy, etc.
    
    // Using C math functions - these come from math.h
    let result = sqrt(16.0)  // Returns 4.0, same as C's sqrt()
    let randomNum = arc4random()  // C random function, returns UInt32
    let power = pow(2.0, 3.0)  // 2 to the power of 3 = 8.0
    
    // C string functions work too
    let cStr = strdup("Hello")  // Duplicates a C string
    defer { free(cStr) }  // Always free malloc'd memory
    ```

45. **Bridging headers** let you use your own C/Objective-C code in Swift projects. This is like telling Swift "hey, these C functions exist".
    ```c
    // MyApp-Bridging-Header.h - put this in your project root
    #include "my_c_library.h"      // Your custom C library
    #include "SomeObjectiveCClass.h"  // Objective-C class you want to use
    
    // You can also declare C functions directly here
    int my_c_function(int x, int y);
    void process_data(char* buffer, int size);
    ```
    
    In Xcode, set the bridging header path in Build Settings. For SPM, it's trickier - you usually put headers in `include/` folder:
    ```swift
    .target(
        name: "MyApp",
        publicHeadersPath: "include",  // Headers go in Sources/MyApp/include/
        cSettings: [
            .headerSearchPath("include"),  // Tell compiler where to find headers
            .headerSearchPath("/usr/local/include"),  // System header paths
        ]
    )
    ```
    
    Project structure would look like:
    ```
    Sources/
    └── MyApp/
        ├── main.swift
        ├── include/
        │   ├── my_c_library.h
        │   └── module.modulemap
        └── my_c_library.c
    ```

46. **Module maps** let you import C libraries as modules instead of using bridging headers. This is cleaner for libraries.
    ```
    // module.modulemap - put this in your include/ folder
    module MyCLibrary {
        header "my_library.h"        // Main header file to expose
        header "helper_functions.h"  // You can have multiple headers
        link "my_library"            // Link against libmy_library.a or .so
        export *                     // Export everything from headers
    }
    
    // For system libraries
    module SQLite {
        header "/usr/include/sqlite3.h"
        link "sqlite3"
        export *
    }
    ```
    
    Then import it in Swift like any other module:
    ```swift
    import MyCLibrary
    import SQLite
    
    // Now you can use C functions directly
    let result = my_c_function(10, 20)  // From MyCLibrary
    let db = sqlite3_open("test.db")    // From SQLite module
    ```

47. Swift automatically converts C types to Swift types, but you need to know the mappings:
    ```c
    // C functions in your library
    int add_numbers(int a, int b);           // int becomes Int32
    unsigned int get_count(void);            // unsigned int becomes UInt32
    long get_big_number(void);               // long becomes Int
    float get_price(void);                   // float becomes Float
    double get_precise_value(void);          // double becomes Double
    char* get_string(void);                  // char* becomes UnsafePointer<CChar>
    void process_buffer(char* data, int len); // void returns Void (nothing)
    bool is_valid(void);                     // bool becomes Bool (if you include stdbool.h)
    ```
    
    ```swift
    // Swift usage - types are automatically converted
    let result = add_numbers(5, 3)           // Returns Int32, not Int!
    let count = get_count()                  // Returns UInt32
    let bigNum = get_big_number()            // Returns Int
    let price = get_price()                  // Returns Float
    let precise = get_precise_value()        // Returns Double
    
    // C strings need manual conversion
    let cStringPtr = get_string()            // UnsafePointer<CChar>
    let swiftString = String(cString: cStringPtr)  // Convert to Swift String
    
    // Calling functions with buffers
    let data = "Hello World"
    data.withCString { cString in
        process_buffer(cString, Int32(data.count))  // Need to cast count to Int32
    }
    
    let valid = is_valid()                   // Returns Bool
    ```

48. **Using C libraries** in SPM - tell the linker which libraries to link against at build time:
    ```swift
    .target(
        name: "MyApp",
        dependencies: [],
        linkerSettings: [
            .linkedLibrary("sqlite3"),      // Links libsqlite3.dylib (macOS) or libsqlite3.so (Linux)
            .linkedLibrary("curl"),         // Links libcurl for HTTP requests
            .linkedLibrary("z"),            // Links zlib for compression
            .linkedLibrary("m"),            // Links math library (usually needed on Linux)
            .linkedFramework("Foundation"), // Links frameworks on Apple platforms
        ]
    )
    ```
    
    On Linux you might need to install dev packages first:
    ```bash
    # Ubuntu/Debian
    sudo apt-get install libsqlite3-dev libcurl4-openssl-dev zlib1g-dev
    
    # CentOS/RHEL
    sudo yum install sqlite-devel libcurl-devel zlib-devel
    ```
    
    Then in your Swift code:
    ```swift
    import SQLite3  // If you made a module map
    // or just use the functions directly if they're in a bridging header
    
    var db: OpaquePointer?
    let result = sqlite3_open("database.db", &db)  // C function
    if result == SQLITE_OK {
        print("Database opened successfully")
        sqlite3_close(db)
    }
    ```

49. **C++ interop** is limited because Swift can't directly call C++. You need to create C wrapper functions that call your C++ code:
    ```cpp
    // MyCppLibrary.cpp - your existing C++ code
    class Calculator {
    public:
        int add(int a, int b) { return a + b; }
        double multiply(double x, double y) { return x * y; }
        std::string getName() { return "Calculator v1.0"; }
    };
    
    // wrapper.cpp - C functions that call C++ code
    #include "MyCppLibrary.cpp"
    extern "C" {
        // Wrapper for the add function
        int cpp_add(int a, int b) {
            Calculator calc;
            return calc.add(a, b);
        }
        
        // Wrapper for multiply function
        double cpp_multiply(double x, double y) {
            Calculator calc;
            return calc.multiply(x, y);
        }
        
        // Wrapper for string function - returns C string
        const char* cpp_get_name() {
            Calculator calc;
            static std::string name = calc.getName();  // Keep it alive
            return name.c_str();
        }
    }
    ```
    
    ```c
    // wrapper.h - C header that Swift can understand
    #ifdef __cplusplus
    extern "C" {
    #endif
    
    int cpp_add(int a, int b);
    double cpp_multiply(double x, double y);
    const char* cpp_get_name(void);
    
    #ifdef __cplusplus
    }
    #endif
    ```
    
    Then in Swift:
    ```swift
    // Swift code using the C wrappers
    let sum = cpp_add(5, 3)                          // Calls C++, gets Int32
    let product = cpp_multiply(2.5, 4.0)             // Calls C++, gets Double
    let name = String(cString: cpp_get_name())       // Convert C string to Swift
    ```

50. **Objective-C** works seamlessly with Swift on Apple platforms. No wrappers needed, just include the headers:
    ```objc
    // SomeClass.h
    @interface SomeClass : NSObject
    @property (nonatomic, strong) NSString *name;  // Property becomes Swift property
    @property (nonatomic, assign) NSInteger count; // NSInteger becomes Int
    
    - (NSString *)getName;                          // Method becomes func getName() -> String
    - (void)setName:(NSString *)name;              // Method becomes func setName(_ name: String)
    - (void)processArray:(NSArray *)items;         // NSArray becomes [Any]
    - (void)processNumbers:(NSArray<NSNumber *> *)numbers; // Becomes [NSNumber]
    @end
    
    // SomeClass.m
    @implementation SomeClass
    - (NSString *)getName {
        return self.name ?: @"Unknown";
    }
    
    - (void)setName:(NSString *)name {
        self.name = name;
    }
    
    - (void)processArray:(NSArray *)items {
        NSLog(@"Processing %ld items", items.count);
    }
    
    - (void)processNumbers:(NSArray<NSNumber *> *)numbers {
        for (NSNumber *num in numbers) {
            NSLog(@"Number: %@", num);
        }
    }
    @end
    ```
    
    ```swift
    // Swift usage - types are automatically bridged
    let obj = SomeClass()
    
    // Property access
    obj.name = "Hello World"        // Calls setter automatically
    let currentName = obj.name      // Calls getter, might be nil
    
    // Method calls
    obj.setName("Hello")            // Explicit setter call
    let name = obj.getName()        // Returns String, never nil due to ?: in ObjC
    
    // Array bridging
    let mixedArray: [Any] = ["string", 42, true]
    obj.processArray(mixedArray)    // Swift Array becomes NSArray
    
    let numbers = [1, 2, 3, 4, 5].map { NSNumber(value: $0) }
    obj.processNumbers(numbers)     // [NSNumber] works directly
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

52. **Common C interop patterns** for working with pointers and memory:
    ```swift
    // C function that takes a buffer
    // void process_data(unsigned char* data, int length);
    
    // Pattern 1: Swift Array to C buffer
    var buffer = [UInt8](repeating: 0, count: 1024)  // Create Swift array
    buffer.withUnsafeBufferPointer { ptr in
        // ptr.baseAddress is UnsafePointer<UInt8>
        // ptr.count is Int
        process_data(ptr.baseAddress!, Int32(ptr.count))  // Call C function
    }
    
    // Pattern 2: Swift String to C string
    let swiftString = "hello world"
    swiftString.withCString { cString in
        // cString is UnsafePointer<CChar> (same as char* in C)
        some_c_function(cString)
    }
    
    // Pattern 3: Getting data back from C
    // If C function allocates memory: char* get_data(int* size);
    var size: Int32 = 0
    let cData = get_data(&size)  // C allocates memory
    defer { free(cData) }        // Always free C memory!
    
    // Convert to Swift array
    let swiftArray = Array(UnsafeBufferPointer(start: cData, count: Int(size)))
    
    // Pattern 4: Passing Swift closure to C (as function pointer)
    // C function: void set_callback(void (*callback)(int));
    let callback: @convention(c) (Int32) -> Void = { value in
        print("C called back with: \(value)")
    }
    set_callback(callback)
    ```

53. **Memory management** when interfacing with C - be careful who owns what:
    ```swift
    // Rule 1: If C allocates, you must free
    let ptr = malloc(1024)           // C's malloc
    defer { free(ptr) }              // Must use C's free
    
    // Rule 2: If you allocate with Swift, let Swift handle it
    let buffer = UnsafeMutableBufferPointer<UInt8>.allocate(capacity: 1024)
    defer { buffer.deallocate() }    // Swift deallocates
    
    // Rule 3: If C function returns malloc'd memory, you must free it
    // char* get_error_message(void);  // C function that returns malloc'd string
    let errorPtr = get_error_message()
    if errorPtr != nil {
        let error = String(cString: errorPtr!)
        free(errorPtr)               // Must free what C malloc'd
        print("Error: \(error)")
    }
    
    // Rule 4: If C just returns a pointer to its own memory, don't free it
    // const char* get_version(void);  // Returns pointer to static string
    let versionPtr = get_version()   // Don't free this!
    let version = String(cString: versionPtr!)
    
    // Rule 5: When passing Swift data to C, keep Swift object alive
    class DataHolder {
        var data = [UInt8](repeating: 42, count: 1000)
        
        func processWithC() {
            data.withUnsafeBufferPointer { ptr in
                // C function that stores the pointer for later use
                c_function_that_stores_pointer(ptr.baseAddress!)
                // BAD: If this function returns and DataHolder gets freed,
                // the C code will have a dangling pointer!
            }
        }
    }
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

## Creating Windows UI Applications with Swift

57. **Setting up Swift for Windows development.** You need Swift for Windows and a way to create UI. The easiest approach is using Win32 API through C interop.
    ```bash
    # Install Swift for Windows from swift.org
    # Download and install Visual Studio Build Tools or Visual Studio Community
    # Make sure you have Windows SDK installed
    
    # Verify Swift installation
    swift --version
    
    # Create a new project directory
    mkdir MyWindowsApp
    cd MyWindowsApp
    swift package init --type executable
    ```

58. **Basic Package.swift setup for Windows UI.** Configure your project to link with Windows system libraries.
    ```swift
    // swift-tools-version: 5.9
    import PackageDescription
    
    let package = Package(
        name: "MyWindowsApp",
        platforms: [.macOS(.v10_15), .iOS(.v13), .windows(.v10)],  // Support Windows 10+
        targets: [
            .executableTarget(
                name: "MyWindowsApp",
                linkerSettings: [
                    .linkedLibrary("user32", .when(platforms: [.windows])),    // Windows user interface
                    .linkedLibrary("kernel32", .when(platforms: [.windows])),  // Windows kernel functions
                    .linkedLibrary("gdi32", .when(platforms: [.windows])),     // Graphics device interface
                    .linkedLibrary("shell32", .when(platforms: [.windows])),   // Windows shell functions
                ]
            ),
        ]
    )
    ```

59. **Windows API imports and basic setup.** Import necessary Windows headers and define basic types.
    ```swift
    // Sources/MyWindowsApp/WindowsAPI.swift
    #if os(Windows)
    import WinSDK  // This imports Windows SDK headers
    
    // Common Windows types and constants
    typealias HWND = WinSDK.HWND
    typealias HINSTANCE = WinSDK.HINSTANCE
    typealias WPARAM = WinSDK.WPARAM
    typealias LPARAM = WinSDK.LPARAM
    typealias LRESULT = WinSDK.LRESULT
    
    // Window class name - must be unique
    let WINDOW_CLASS_NAME = "MySwiftWindowClass"
    
    // Window procedure function type
    typealias WNDPROC = @convention(c) (HWND?, UINT, WPARAM, LPARAM) -> LRESULT
    #endif
    ```

60. **Creating a basic window class and window procedure.** The window procedure handles all messages sent to your window.
    ```swift
    #if os(Windows)
    // Window procedure - handles all window messages
    let windowProc: WNDPROC = { hwnd, uMsg, wParam, lParam in
        switch uMsg {
        case UINT(WM_DESTROY):                    // User closed the window
            PostQuitMessage(0)                    // Tell Windows to quit the app
            return 0
            
        case UINT(WM_PAINT):                      // Window needs to be redrawn
            var ps = PAINTSTRUCT()
            let hdc = BeginPaint(hwnd, &ps)       // Start painting
            
            // Draw some text
            let text = "Hello from Swift on Windows!"
            let rect = ps.rcPaint                 // Area that needs repainting
            DrawTextA(hdc, text, Int32(text.count), UnsafeMutablePointer(mutating: &rect), 
                     UINT(DT_CENTER | DT_VCENTER | DT_SINGLELINE))
            
            EndPaint(hwnd, &ps)                   // Finish painting
            return 0
            
        case UINT(WM_COMMAND):                    // Button clicks, menu selections
            let commandId = LOWORD(wParam)        // Extract command ID
            switch commandId {
            case 1001:                            // Our button ID
                MessageBoxA(hwnd, "Button clicked!", "Swift App", UINT(MB_OK))
                return 0
            default:
                break
            }
            
        default:
            break
        }
        
        // Let Windows handle messages we don't process
        return DefWindowProcA(hwnd, uMsg, wParam, lParam)
    }
    
    // Register the window class
    func registerWindowClass(hInstance: HINSTANCE) -> Bool {
        var wc = WNDCLASSA()
        wc.lpfnWndProc = windowProc              // Our window procedure
        wc.hInstance = hInstance                 // Application instance
        wc.lpszClassName = WINDOW_CLASS_NAME     // Class name
        wc.hCursor = LoadCursorA(nil, IDC_ARROW) // Default arrow cursor
        wc.hbrBackground = HBRUSH(COLOR_WINDOW + 1)  // Default window background
        wc.style = UINT(CS_HREDRAW | CS_VREDRAW) // Redraw when resized
        
        return RegisterClassA(&wc) != 0          // Returns true if successful
    }
    #endif
    ```

61. **Creating and showing the main window.** Set up the window with title, size, and position.
    ```swift
    #if os(Windows)
    func createMainWindow(hInstance: HINSTANCE) -> HWND? {
        // Create the window
        let hwnd = CreateWindowExA(
            0,                                    // Extended window style
            WINDOW_CLASS_NAME,                    // Window class name
            "My Swift Windows App",               // Window title
            DWORD(WS_OVERLAPPEDWINDOW),          // Window style (standard window)
            CW_USEDEFAULT,                       // X position (let Windows decide)
            CW_USEDEFAULT,                       // Y position (let Windows decide)
            800,                                 // Width in pixels
            600,                                 // Height in pixels
            nil,                                 // Parent window (none)
            nil,                                 // Menu (none)
            hInstance,                           // Application instance
            nil                                  // Additional data (none)
        )
        
        guard hwnd != nil else {
            print("Failed to create window")
            return nil
        }
        
        // Show the window
        ShowWindow(hwnd, SW_SHOW)                // Make window visible
        UpdateWindow(hwnd)                       // Force immediate redraw
        
        return hwnd
    }
    #endif
    ```

62. **Adding controls like buttons and text fields.** Create child windows that act as UI controls.
    ```swift
    #if os(Windows)
    func createControls(parentHwnd: HWND, hInstance: HINSTANCE) {
        // Create a button
        let buttonHwnd = CreateWindowExA(
            0,                                    // Extended style
            "BUTTON",                            // Control class (built-in button class)
            "Click Me!",                         // Button text
            DWORD(WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON),  // Button style
            50,                                  // X position relative to parent
            50,                                  // Y position relative to parent
            100,                                 // Width
            30,                                  // Height
            parentHwnd,                          // Parent window
            HMENU(1001),                         // Control ID (used in WM_COMMAND)
            hInstance,                           // Application instance
            nil                                  // Additional data
        )
        
        // Create a text input field
        let editHwnd = CreateWindowExA(
            WS_EX_CLIENTEDGE,                    // Sunken border style
            "EDIT",                              // Edit control class
            "Type here...",                      // Initial text
            DWORD(WS_CHILD | WS_VISIBLE | WS_BORDER | ES_LEFT),  // Edit control style
            50,                                  // X position
            100,                                 // Y position
            200,                                 // Width
            25,                                  // Height
            parentHwnd,                          // Parent window
            HMENU(1002),                         // Control ID
            hInstance,                           // Application instance
            nil                                  // Additional data
        )
        
        // Create a label
        let labelHwnd = CreateWindowExA(
            0,                                   // No extended style
            "STATIC",                            // Static text control
            "Enter your name:",                  // Label text
            DWORD(WS_CHILD | WS_VISIBLE | SS_LEFT),  // Static control style
            50,                                  // X position
            140,                                 // Y position
            150,                                 // Width
            20,                                  // Height
            parentHwnd,                          // Parent window
            HMENU(1003),                         // Control ID
            hInstance,                           // Application instance
            nil                                  // Additional data
        )
    }
    #endif
    ```

63. **Main application loop and message handling.** Process Windows messages to keep the app responsive.
    ```swift
    #if os(Windows)
    func runMessageLoop() {
        var msg = MSG()
        
        // Main message loop
        while true {
            let result = GetMessageA(&msg, nil, 0, 0)  // Get next message
            
            if result == 0 {                     // WM_QUIT received
                print("Application exiting...")
                break
            } else if result == -1 {             // Error occurred
                print("Error in message loop")
                break
            }
            
            // Process the message
            TranslateMessage(&msg)               // Translate keyboard input
            DispatchMessageA(&msg)               // Send to window procedure
        }
    }
    #endif
    ```

64. **Complete main.swift file.** Put it all together in a working Windows application.
    ```swift
    // Sources/MyWindowsApp/main.swift
    #if os(Windows)
    import WinSDK
    
    func main() {
        // Get application instance handle
        let hInstance = GetModuleHandleA(nil)
        guard hInstance != nil else {
            print("Failed to get module handle")
            return
        }
        
        // Register our window class
        guard registerWindowClass(hInstance: hInstance!) else {
            print("Failed to register window class")
            return
        }
        
        // Create the main window
        guard let hwnd = createMainWindow(hInstance: hInstance!) else {
            print("Failed to create main window")
            return
        }
        
        // Create controls (buttons, text fields, etc.)
        createControls(parentHwnd: hwnd, hInstance: hInstance!)
        
        // Run the message loop
        runMessageLoop()
    }
    
    // Start the application
    main()
    
    #else
    print("This application is designed for Windows only")
    #endif
    ```

65. **Building and running your Windows app.** Compile and test your Swift Windows application.
    ```bash
    # Build the application
    swift build -c release
    
    # Run the application
    .build\release\MyWindowsApp.exe
    
    # Or build and run in one step
    swift run
    
    # For debugging, build without optimization
    swift build
    .build\debug\MyWindowsApp.exe
    ```

66. **Handling more advanced UI features.** Add menus, icons, and better event handling.
    ```swift
    #if os(Windows)
    // Create a menu bar
    func createMenuBar(hwnd: HWND) {
        let hMenu = CreateMenu()                 // Main menu bar
        let hFileMenu = CreatePopupMenu()        // File submenu
        
        // Add items to File menu
        AppendMenuA(hFileMenu, MF_STRING, 2001, "New")
        AppendMenuA(hFileMenu, MF_STRING, 2002, "Open")
        AppendMenuA(hFileMenu, MF_SEPARATOR, 0, nil)  // Separator line
        AppendMenuA(hFileMenu, MF_STRING, 2003, "Exit")
        
        // Add File menu to menu bar
        AppendMenuA(hMenu, MF_POPUP, UINT_PTR(hFileMenu), "File")
        
        // Attach menu to window
        SetMenu(hwnd, hMenu)
    }
    
    // Enhanced window procedure with menu handling
    let enhancedWindowProc: WNDPROC = { hwnd, uMsg, wParam, lParam in
        switch uMsg {
        case UINT(WM_CREATE):                    // Window is being created
            createMenuBar(hwnd: hwnd!)           // Add menu when window is created
            return 0
            
        case UINT(WM_COMMAND):
            let commandId = LOWORD(wParam)
            switch commandId {
            case 2001:  // New
                MessageBoxA(hwnd, "New file selected", "Menu", UINT(MB_OK))
            case 2002:  // Open
                MessageBoxA(hwnd, "Open file selected", "Menu", UINT(MB_OK))
            case 2003:  // Exit
                PostQuitMessage(0)
            default:
                break
            }
            return 0
            
        case UINT(WM_SIZE):                      // Window was resized
            let width = LOWORD(lParam)           // New width
            let height = HIWORD(lParam)          // New height
            print("Window resized to \(width)x\(height)")
            return 0
            
        case UINT(WM_DESTROY):
            PostQuitMessage(0)
            return 0
            
        default:
            break
        }
        
        return DefWindowProcA(hwnd, uMsg, wParam, lParam)
    }
    #endif
    ```

67. **Error handling and debugging tips.** Common issues and how to solve them.
    ```swift
    #if os(Windows)
    // Get detailed error information
    func getLastErrorMessage() -> String {
        let errorCode = GetLastError()
        var buffer = [CHAR](repeating: 0, count: 256)
        
        let length = FormatMessageA(
            DWORD(FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS),
            nil,                                 // Message source
            errorCode,                           // Error code
            0,                                   // Language ID (default)
            &buffer,                             // Output buffer
            DWORD(buffer.count),                 // Buffer size
            nil                                  // Arguments
        )
        
        if length > 0 {
            return String(cString: buffer)
        } else {
            return "Unknown error (code: \(errorCode))"
        }
    }
    
    // Safe window creation with error checking
    func createWindowSafely(hInstance: HINSTANCE) -> HWND? {
        let hwnd = CreateWindowExA(
            0, WINDOW_CLASS_NAME, "My App",
            DWORD(WS_OVERLAPPEDWINDOW),
            CW_USEDEFAULT, CW_USEDEFAULT, 800, 600,
            nil, nil, hInstance, nil
        )
        
        if hwnd == nil {
            print("CreateWindowExA failed: \(getLastErrorMessage())")
            return nil
        }
        
        return hwnd
    }
    
    // Common debugging techniques
    func debugWindowInfo(hwnd: HWND) {
        var rect = RECT()
        GetWindowRect(hwnd, &rect)               // Get window position and size
        print("Window rect: (\(rect.left), \(rect.top), \(rect.right), \(rect.bottom))")
        
        var className = [CHAR](repeating: 0, count: 256)
        GetClassNameA(hwnd, &className, Int32(className.count))
        print("Window class: \(String(cString: className))")
        
        let isVisible = IsWindowVisible(hwnd)
        print("Window visible: \(isVisible != 0)")
    }
    #endif
    ```

68. **Tips for Windows Swift development:**
    - Always check return values from Windows API calls
    - Use `#if os(Windows)` to make your code cross-platform
    - Windows API uses different calling conventions - stick to `@convention(c)`
    - Memory management: Windows API often returns handles, not pointers
    - Use Windows SDK documentation for detailed API reference
    - Test on actual Windows machines, not just emulators

## Swift Package Manager: Why targets have dependencies, and what products really are

### Why do targets have their own `dependencies` array, even with a top-level `dependencies` section?

- **Top-level `dependencies`:**
  This section in `Package.swift` lists all the _external packages_ (from GitHub, local path, etc.) your project can use. Think of it as your "available libraries" shelf.

  ```swift
  dependencies: [
      .package(url: "https://github.com/apple/swift-argument-parser", from: "1.0.0"),
      .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
  ]
  ```

- **Target-level `dependencies**:**
  Each target (your app, a library, a test suite, etc.) specifies _which_ of those dependencies (and which other targets in your package) it actually uses. This is like saying, "This module needs these libraries and these other modules to work."

  ```swift
  .target(
      name: "MyApp",
      dependencies: [
          .product(name: "ArgumentParser", package: "swift-argument-parser"),
          "MyLibrary" // another target in your package
      ]
  )
  ```

  - This keeps your build fast and clean: Only the code you actually use gets built and linked for each target.
  - **Analogy:**
    - Top-level `dependencies` = all the books in your library
    - Target-level `dependencies` = the books you actually cite in your paper

### Are products just a way to build targets (modules)?

- **Products** in SPM are the _outputs_ your package offers to the outside world. They tell Swift (and other packages) what you want to make available for use.

  ```swift
  products: [
      .executable(name: "MyApp", targets: ["MyApp"]),
      .library(name: "MyLibrary", targets: ["MyLibrary"]),
  ]
  ```

  - **Executable product:**
    - Builds a command-line tool or app from one or more targets (must have a `main.swift` or `@main`).
    - Can be run with `swift run MyApp`.

  - **Library product:**
    - Builds a module (or set of modules) that other packages can import.
    - Can be used with `import MyLibrary` in other Swift code.

- **Targets** are the actual code modules (folders under `Sources/`) that get built.
  - You can have many targets, but only the ones listed in `products` are visible to the outside world.
  - **Analogy:**
    - Targets = chapters in your book
    - Products = the finished book you publish (could be just one chapter, or a collection)

#### In summary:
- **Top-level `dependencies**:** All possible external packages you might use.
- **Target `dependencies**:** Which packages/targets each module actually uses.
- **Products:** The final things you build and share (executables or libraries), made from one or more targets.

## When do you need multiple targets to build a product?

You need multiple targets for a product when your final output (executable or library) is made up of several separate modules, each with its own code, dependencies, or responsibilities. This is common for:

- **Code organization:** Keeping networking, database, and UI logic in separate modules.
- **Reusability:** Sharing utility code between different products or apps.
- **Testing:** Having test targets that depend on your main code targets.
- **Conditional builds:** Including/excluding features for different platforms or configurations.

### Example: A web server library with shared utilities

Suppose you want to build a library product called `WebServerKit` that includes:
- Core server logic (`WebServerCore` target)
- HTTP utilities (`HTTPUtils` target)
- Logging utilities (`LoggingUtils` target)
- The product should expose all of these as a single library.

**Package.swift:**
```swift
products: [
    .library(
        name: "WebServerKit",
        targets: ["WebServerCore", "HTTPUtils", "LoggingUtils"]
    ),
],
targets: [
    .target(
        name: "WebServerCore",
        dependencies: ["HTTPUtils", "LoggingUtils"]
    ),
    .target(
        name: "HTTPUtils"
    ),
    .target(
        name: "LoggingUtils"
    ),
    .testTarget(
        name: "WebServerCoreTests",
        dependencies: ["WebServerCore"]
    ),
]
```

- The `WebServerKit` product is built from **three targets**: `WebServerCore`, `HTTPUtils`, and `LoggingUtils`.
- Users who add `WebServerKit` as a dependency can access all the public APIs from all three modules.

**Directory structure:**
```
Sources/
├── WebServerCore/
│   └── Server.swift
├── HTTPUtils/
│   └── HTTPParser.swift
└── LoggingUtils/
    └── Logger.swift
Tests/
└── WebServerCoreTests/
    └── ServerTests.swift
```

**Summary:**
- Use multiple targets in a product when you want to combine several modules into a single library or executable, for better code organization, reusability, and modularity.

## Why do we keep providing `name:` everywhere in Package.swift?

- **`name:` in the package declaration**
  - This is the name of your whole package (the project as a whole).
    ```swift
    let package = Package(
        name: "MyApp",
        // ...
    )
    ```
  - This is mostly for humans and for tools to identify your package.

- **`name:` in products and targets**
  - Every product and every target also needs a `name:`.
    ```swift
    .library(name: "MyLibrary", targets: ["MyLibrary"])
    .target(name: "MyLibrary")
    ```
  - **Why?**
    - You can have multiple products and multiple targets in one package, and their names might not match the package name.
    - You might want to build several libraries or executables from the same package, each with a different name.
    - Targets can be reused in different products, so each must have a unique name.
  - **Analogy:**
    - Package name = the name of your book series
    - Product name = the name of a specific book in the series
    - Target name = the name of a chapter or section in a book

## Do each module/target have their own folder inside `Sources/`? Is that mandatory?

- **Yes, by convention and for SPM to work smoothly:**
  - Each target (module) should have its own folder inside `Sources/` (for code) or `Tests/` (for tests).
  - The folder name should match the target name you declare in `Package.swift`.
  ```
  Sources/
  ├── MyApp/         // Target: MyApp
  ├── Utils/         // Target: Utils
  └── Networking/    // Target: Networking
  ```

- **Is it mandatory?**
  - For most use cases: **Yes.**
    - SPM expects each target to have its own folder named exactly as the target.
    - If you want to use a different folder name, you must specify the `path:` property in the target declaration:
      ```swift
      .target(
          name: "Networking",
          path: "Sources/NetCode" // Folder is NetCode, but target is Networking
      )
      ```
    - But the default and simplest approach is to match folder and target names.

## Do you use the folder name as the target name in the package file?

- **By default, yes.**
  - If you don't specify a `path:`, SPM looks for a folder in `Sources/` (or `Tests/`) with the same name as the target.
  - The target name in `Package.swift` should match the folder name for everything to "just work."

---

### Summary Table

| What         | Where is `name:` used?         | What does it mean?                | Folder required?      |
|--------------|-------------------------------|-----------------------------------|-----------------------|
| Package      | `name:` in `Package(...)`     | Name of the whole package         | No folder needed      |
| Product      | `name:` in `.library/.executable` | Name of the built output (lib/app) | No folder needed      |
| Target       | `name:` in `.target/.testTarget`  | Name of the module/code group     | Yes, folder in Sources/ or specify `path:` |

## How to add a dependency or create a target in Swift Package Manager (without Xcode)

You can manage your Swift package entirely from the terminal and a text editor—no Xcode required!

### 1. Add a dependency (library/package)
- Open your `Package.swift` file in a text editor.
- Find the `dependencies:` array and add your new package.

**Example:** Add Alamofire HTTP networking library:
```swift
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.7.0"),
    // ...other dependencies
],
```

- In the target that needs Alamofire, add it to the `dependencies:` array:
```swift
.target(
    name: "MyApp",
    dependencies: [
        .product(name: "Alamofire", package: "Alamofire")
    ]
)
```

- Save the file, then run in your terminal:
```bash
swift package resolve
```
This downloads and prepares the dependency.

---

### 2. Create a new target (module)
- In your project folder, create a new directory under `Sources/` with the name of your new target:
```bash
mkdir Sources/MyNewModule
# Add a Swift file to the new module
nano Sources/MyNewModule/MyNewModule.swift
```

- In `Package.swift`, add a new target entry:
```swift
.target(
    name: "MyNewModule"
    // Optionally, add dependencies: [...]
)
```

- If you want to use this new module in another target, add it to that target's dependencies:
```swift
.target(
    name: "MyApp",
    dependencies: [
        "MyNewModule"
    ]
)
```

- Save the file. Now you can use `import MyNewModule` in your code.

---

### 3. Build and test your changes
```bash
swift build
swift test
```

---

### Summary Table

| Task                | What to do (no Xcode)                                                                 |
|---------------------|--------------------------------------------------------------------------------------|
| Add dependency      | Edit `Package.swift` → add to `dependencies:` → run `swift package resolve`          |
| Create new target   | Make folder in `Sources/` → add `.swift` file → edit `Package.swift` → add `.target` |
| Use new target      | Add target name to another target's `dependencies:` in `Package.swift`               |
| Build/test          | Run `swift build` and `swift test` in terminal                                       |

---

### Example: Adding a dependency and a target

```swift
// Package.swift
let package = Package(
    name: "MyApp",
    products: [
        .executable(name: "MyApp", targets: ["MyApp"]),
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.7.0"),
    ],
    targets: [
        .target(
            name: "MyApp",
            dependencies: [
                "MyNewModule",
                .product(name: "Alamofire", package: "Alamofire")
            ]
        ),
        .target(
            name: "MyNewModule"
        ),
    ]
)
```

## Swift Project Folder Structure: What goes where?

A typical Swift Package Manager (SPM) project looks like this:

```
MyApp/
├── Package.swift         # Project configuration file
├── Sources/              # All your code (organized by target/module)
│   ├── MyApp/            # Main target/module
│   │   ├── main.swift
│   │   └── ...           # Other Swift files
│   └── Utils/            # Another target/module (optional)
│       └── Helper.swift
├── Tests/                # All your test code (organized by test target)
│   └── MyAppTests/
│       └── MyAppTests.swift
├── Resources/            # (Optional) Shared resources for the package
├── .build/               # Build artifacts (created by SPM, not in git)
│   └── debug/            # Debug build output
│   └── release/          # Release build output
├── Package.resolved      # Locked dependency versions
└── README.md             # Project documentation
```

### What does each folder contain?

- **Package.swift**  
  The manifest file that describes your package, its dependencies, targets, products, etc.

- **Sources/**  
  Contains all your source code, organized by target/module.  
  Each subfolder is a target (module) and should match the target name in `Package.swift`.

- **Tests/**  
  Contains all your test code, organized by test target.  
  Each subfolder is a test target and should match the test target name in `Package.swift`.

- **Resources/** (optional)  
  You can put shared resources here, but SPM expects resources for a target to be inside that target's folder (see below).

- **.build/**  
  Created by SPM when you build. Contains all build artifacts, including:
  - **debug/**: Debug build output (executables, libraries, intermediate files)
  - **release/**: Release build output (optimized binaries)
  - **other internal folders**: Caches, dependency checkouts, etc.
  - **You should NOT commit `.build/` to git.**

- **Package.resolved**  
  Records the exact versions of dependencies used. Should be committed to git.

- **README.md**  
  Your project's documentation.

---

### Where do resources go?

- **Target-specific resources:**  
  If you want to include resources (images, JSON, config files, etc.) in your app or library, put them in a `Resources/` folder inside the target's folder:

  ```
  Sources/
  └── MyApp/
      ├── main.swift
      ├── Resources/
      │   ├── config.json
      │   └── logo.png
  ```

  In `Package.swift`, declare them in the target:
  ```swift
  .target(
      name: "MyApp",
      resources: [
          .process("Resources")
      ]
  )
  ```

- **After building:**  
  - For executables: Resources are bundled alongside the binary in the `.build/` output directory.
  - For libraries: Resources are bundled in a way that importing packages can access them.
  - **Resources are NOT embedded inside the binary itself** (unlike some other languages). They are packaged in a resources directory next to the binary.

---

### Where does the binary go after building?

- **After you run `swift build`:**
  - The built executable or library is placed in:
    - `.build/debug/` for debug builds
    - `.build/release/` for release builds

  **Example:**
  ```
  .build/debug/MyApp         # Your executable (if you have an executable target)
  .build/debug/libMyLibrary.a  # Your static library (if you have a library target)
  .build/debug/MyApp.resources/ # Resources directory for your target
  ```

---

### Summary Table

| Folder/File         | What it contains / is for                                 |
|---------------------|----------------------------------------------------------|
| Package.swift       | Project configuration and manifest                       |
| Sources/            | All source code, organized by target/module              |
| Tests/              | All test code, organized by test target                  |
| Resources/          | (Optional) Shared resources (not standard for SPM)       |
| Sources/Target/Resources/ | Resources for a specific target                    |
| .build/             | Build artifacts (binaries, resources, caches)            |
| .build/debug/       | Debug build output (executables, libraries, resources)   |
| .build/release/     | Release build output (optimized)                         |
| Package.resolved    | Locked dependency versions                               |
| README.md           | Project documentation                                    |

---

### Key Points

- Each target/module has its own folder in `Sources/`.
- Resources for a target go in a `Resources/` folder inside that target's folder.
- After building, binaries and resources are placed in `.build/debug/` or `.build/release/`.
- Resources are not embedded in the binary, but are bundled alongside it in a `.resources/` directory.

## What file types are produced by Swift Package Manager?

### Executables

- **On macOS and Linux:**  
  Executables are **plain binaries** (no `.exe` extension).
  - Example:
    ```
    .build/debug/MyApp      # Executable, run with ./MyApp
    ```
- **On Windows:**  
  Executables have the `.exe` extension.
  - Example:
    ```
    .build/debug/MyApp.exe
    ```

### Libraries

- **Static libraries:**  
  - File extension: `.a` (on all platforms)
  - Example:
    ```
    .build/debug/libMyLibrary.a
    ```
  - These are archives of object files, linked at compile time.

- **Dynamic/shared libraries:**  
  - File extension depends on platform:
    - **macOS:** `.dylib`
    - **Linux:** `.so`
    - **Windows:** `.dll`
  - Example:
    ```
    .build/debug/libMyLibrary.dylib   # macOS
    .build/debug/libMyLibrary.so      # Linux
    .build/debug/MyLibrary.dll        # Windows
    ```
  - These are loaded at runtime.

- **Object files:**  
  - File extension: `.o`
  - These are intermediate files created during compilation, not usually distributed or used directly.

### Summary Table

| Output Type         | macOS         | Linux         | Windows      | Notes                                 |
|---------------------|---------------|---------------|--------------|---------------------------------------|
| Executable          | (no extension)| (no extension)| `.exe`       | Run with `./MyApp` or `MyApp.exe`     |
| Static library      | `.a`          | `.a`          | `.a`         | Linked at compile time                |
| Dynamic library     | `.dylib`      | `.so`         | `.dll`       | Loaded at runtime                     |
| Object file         | `.o`          | `.o`          | `.o`         | Intermediate, not usually distributed |

---

### Key Points

- **Executables:**  
  No extension on macOS/Linux, `.exe` on Windows.
- **Static libraries:**  
  Always `.a` (all platforms).
- **Dynamic/shared libraries:**  
  `.dylib` (macOS), `.so` (Linux), `.dll` (Windows).
- **Object files:**  
  `.o` (all platforms), not usually needed by end users.

---

### How to control what gets built?

- The type of output (executable, static library, dynamic library) is determined by your `products` in `Package.swift`:
  ```swift
  products: [
      .executable(name: "MyApp", targets: ["MyApp"]),
      .library(name: "MyLibrary", targets: ["MyLibrary"]), // Default is static
      .library(name: "MyDynamicLib", type: .dynamic, targets: ["MyDynamicLib"]),
  ]
  ```
