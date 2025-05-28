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
