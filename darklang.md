# DarkLang Programming Guide

## What is DarkLang

DarkLang is a functional programming language designed for building backend services quickly. It removes traditional deployment complexity by providing an integrated development environment where code, editor, and infrastructure work together. The key idea is **deployless** - your code runs immediately as you type without separate build or deployment steps.

## Installation and Setup

**DarkLang is web-based** - you don't install it locally. Instead, you access it through your browser at `darklang.com`. This is part of the deployless philosophy where everything runs in the cloud.

1. **Getting started**: Go to `darklang.com` and create an account
2. **Canvas creation**: Create a new "canvas" which is your workspace for a specific application
3. **Immediate execution**: Code runs instantly as you type - no build or deployment steps needed

**No traditional installation required** - just a modern web browser with internet connection.

## Type System and Execution

**DarkLang is strongly typed** with automatic type inference. This means:
- Variables have specific types (String, Int, Bool, etc.) but you rarely need to specify them
- The compiler catches type errors before your code runs
- Types are inferred from context, making code cleaner while staying safe

```darklang
let name = "Alice"        // Type: String (inferred)
let age = 25             // Type: Int (inferred)  
let score = 98.5         // Type: Float (inferred)
let isActive: Bool = true // Type: Bool (explicit, but optional)
```

**Execution model**:
- **Interpreted**: Code runs directly without creating separate executable files
- **Cloud-native**: Programs run on DarkLang's infrastructure, not your local machine
- **Real-time**: Changes take effect immediately as you edit
- **Persistent**: Your applications stay running 24/7 without you managing servers

**No executables created** - instead, your functions become live HTTP endpoints and event handlers that respond to requests immediately.

## Platform and Deployment

**Cross-platform by design** - since DarkLang runs in the browser and on cloud infrastructure:
- **Development**: Works on any OS with a modern browser (Windows, macOS, Linux)
- **Runtime**: Applications run on DarkLang's cloud infrastructure automatically
- **Access**: Your APIs and services are accessible from anywhere on the internet
- **No platform concerns**: You don't manage servers, containers, or deployment environments

**Built-in infrastructure**:
- Automatic HTTPS endpoints for your functions
- Built-in database (no setup required)
- Automatic scaling based on traffic
- Real-time error tracking and logging

```darklang
// This function automatically becomes available at:
// https://yourcanvas-yourname.builtwithdark.com/hello
fun hello(request: HttpRequest): HttpResponse =
  { 
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: "Hello from DarkLang!"
  }
```

## Basic Syntax and Variables

1. **Variables** are created with `let` and are immutable by default. DarkLang emphasizes functional programming principles.
   ```darklang
   let name = "Alice"
   let age = 25
   let isActive = true
   ```

2. **Types** are inferred automatically. DarkLang has a strong type system but you rarely need to specify types explicitly.
   ```darklang
   let message = "Hello World"    // String type inferred
   let count = 42                 // Int type inferred  
   let price = 19.99             // Float type inferred
   let items = [1, 2, 3, 4]      // List<Int> type inferred
   ```

3. **Functions** are defined with the `fun` keyword. Functions are first-class values and can be assigned to variables.
   ```darklang
   fun greet(name: String): String =
     "Hello, " + name + "!"
   
   // Function can be assigned to a variable
   let myGreeter = greet
   let result = myGreeter("Bob")  // "Hello, Bob!"
   ```

4. **Comments** use `//` for single line comments. There are no multi-line comments.
   ```darklang
   // This is a comment
   let x = 5  // Comment at end of line
   ```

## Working with Data Types

### Strings
5. **String manipulation** is done with built-in functions. Strings are immutable.
   ```darklang
   let first = "Dark"
   let second = "Lang"
   let combined = String::append(first, second)  // "DarkLang"
   let length = String::length(combined)         // 8
   let upper = String::toUppercase(combined)     // "DARKLANG"
   ```

### Lists
6. **Lists** hold multiple values of the same type. Lists are immutable - operations return new lists.
   ```darklang
   let numbers = [1, 2, 3, 4, 5]
   let firstItem = List::head(numbers)           // Some(1) - returns Option type
   let restItems = List::tail(numbers)           // [2, 3, 4, 5]
   let newList = List::append(numbers, [6, 7])   // [1, 2, 3, 4, 5, 6, 7]
   let doubled = List::map(numbers, fun x -> x * 2)  // [2, 4, 6, 8, 10]
   ```

### Dictionaries (Records)
7. **Dictionaries** store key-value pairs. They're called Records in DarkLang and use curly braces.
   ```darklang
   let user = {
     name: "Alice",
     age: 30,
     email: "alice@example.com"
   }
   
   let userName = user.name              // Access field with dot notation
   let userAge = user["age"]             // Access field with bracket notation
   ```

## Option Types and Error Handling

8. **Option types** handle values that might not exist. They replace null/undefined from other languages.
   ```darklang
   // Some() wraps a value, None represents no value
   let maybeValue = Some(42)
   let noValue = None
   
   // Pattern matching to handle Options
   match maybeValue with
   | Some(value) -> "Got value: " + toString(value)
   | None -> "No value found"
   ```

9. **Result types** handle operations that might fail. They replace exceptions.
   ```darklang
   // Ok() wraps success value, Error() wraps error value  
   fun divide(a: Int, b: Int): Result<Int, String> =
     if b == 0 then
       Error("Cannot divide by zero")
     else
       Ok(a / b)
   
   let result = divide(10, 2)
   match result with
   | Ok(value) -> "Result: " + toString(value)
   | Error(msg) -> "Error: " + msg
   ```

## Pattern Matching

10. **Pattern matching** with `match` is the primary way to handle different cases. It's like switch statements but more powerful.
    ```darklang
    fun processNumber(n: Int): String =
      match n with
      | 0 -> "Zero"
      | 1 -> "One" 
      | x when x > 100 -> "Big number"
      | x when x < 0 -> "Negative"
      | _ -> "Some other number"
    ```

11. **Destructuring** in pattern matching lets you extract values from complex data structures.
    ```darklang
    let point = { x: 10, y: 20 }
    
    match point with
    | { x: 0, y: 0 } -> "Origin"
    | { x: x, y: 0 } -> "On X-axis at " + toString(x)
    | { x: 0, y: y } -> "On Y-axis at " + toString(y)
    | { x: x, y: y } -> "Point at (" + toString(x) + ", " + toString(y) + ")"
    ```

## HTTP Handlers and Web Services

12. **HTTP handlers** are functions that respond to web requests. They're the main way to create APIs in DarkLang.
    ```darklang
    // Basic HTTP handler - responds to GET /hello
    fun hello(request: HttpRequest): HttpResponse =
      { 
        statusCode: 200,
        headers: { "Content-Type": "application/json" },
        body: "{ \"message\": \"Hello World\" }"
      }
    ```

13. **Request parameters** are accessed through the request object. URL parameters, query strings, and body data are available.
    ```darklang
    // Handler for GET /users/:id
    fun getUser(request: HttpRequest): HttpResponse =
      let userId = request.pathParams["id"]
      let user = Database::findUser(userId)
      match user with
      | Some(userData) -> 
          { 
            statusCode: 200,
            headers: { "Content-Type": "application/json" },
            body: JSON::serialize(userData)
          }
      | None -> 
          { 
            statusCode: 404,
            headers: { "Content-Type": "application/json" },
            body: "{ \"error\": \"User not found\" }"
          }
    ```

14. **POST handlers** can access request body data. DarkLang automatically parses JSON bodies.
    ```darklang
    // Handler for POST /users
    fun createUser(request: HttpRequest): HttpResponse =
      match JSON::parse(request.body) with
      | Ok(userData) ->
          let newUser = Database::insertUser(userData)
          { 
            statusCode: 201,
            headers: { "Content-Type": "application/json" },
            body: JSON::serialize(newUser)
          }
      | Error(parseError) ->
          { 
            statusCode: 400,
            headers: { "Content-Type": "application/json" },
            body: "{ \"error\": \"Invalid JSON\" }"
          }
    ```

## Database Operations

15. **Database queries** use the built-in Database module. DarkLang provides a key-value store by default.
    ```darklang
    // Store data
    let user = { id: "123", name: "Alice", email: "alice@example.com" }
    Database::set("user_123", user)
    
    // Retrieve data
    let retrievedUser = Database::get("user_123")  // Returns Option<Any>
    
    // Check if key exists
    let userExists = Database::exists("user_123")  // Returns Bool
    
    // Delete data
    Database::delete("user_123")
    ```

16. **Database collections** help organize related data. You can query across collections.
    ```darklang
    // Store in a collection
    Database::setInCollection("users", "123", user)
    
    // Get all from collection
    let allUsers = Database::getCollection("users")  // Returns List<Any>
    
    // Find specific items in collection
    let activeUsers = Database::queryCollection("users", fun user ->
      user.isActive == true
    )
    ```

## Working with External APIs

17. **HTTP client** requests use the HTTP module. You can make requests to external services.
    ```darklang
    // GET request
    let response = HTTP::get("https://api.example.com/users")
    match response with
    | Ok(data) -> JSON::parse(data.body)
    | Error(httpError) -> Error("API request failed")
    
    // POST request with body
    let postData = { name: "New User", email: "user@example.com" }
    let postResponse = HTTP::post(
      "https://api.example.com/users",
      JSON::serialize(postData),
      { "Content-Type": "application/json" }
    )
    ```

18. **Headers and authentication** can be added to HTTP requests.
    ```darklang
    let headers = {
      "Authorization": "Bearer " + apiToken,
      "Content-Type": "application/json",
      "User-Agent": "DarkLang-App/1.0"
    }
    
    let authenticatedResponse = HTTP::get(
      "https://api.protected.com/data",
      headers
    )
    ```

## Higher-Order Functions

19. **Map, filter, and reduce** are essential functional programming operations for working with lists.
    ```darklang
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    // Map: transform each element
    let doubled = List::map(numbers, fun x -> x * 2)  // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
    
    // Filter: keep elements that match condition
    let evens = List::filter(numbers, fun x -> x % 2 == 0)  // [2, 4, 6, 8, 10]
    
    // Reduce: combine all elements into single value
    let sum = List::fold(numbers, 0, fun acc x -> acc + x)  // 55
    ```

20. **Function composition** and **partial application** help build complex operations from simple ones.
    ```darklang
    // Function composition with pipe operator
    let processNumbers = fun numbers ->
      numbers
      |> List::filter(fun x -> x > 0)      // Keep positive numbers
      |> List::map(fun x -> x * x)         // Square each number  
      |> List::fold(0, fun acc x -> acc + x) // Sum all squares
    
    let result = processNumbers([-2, -1, 0, 1, 2, 3])  // 14 (1 + 4 + 9)
    ```

## Event Handling and Background Tasks

21. **Event handlers** respond to external events like webhooks, scheduled tasks, or database changes.
    ```darklang
    // Scheduled event - runs every hour
    fun hourlyCleanup(event: ScheduledEvent): Unit =
      let oldRecords = Database::queryCollection("logs", fun log ->
        Date::isOlderThan(log.timestamp, Date::hoursAgo(24))
      )
      List::iter(oldRecords, fun record ->
        Database::deleteFromCollection("logs", record.id)
      )
    ```

22. **Webhook handlers** process incoming webhook events from external services.
    ```darklang
    // Webhook for payment notifications
    fun paymentWebhook(request: HttpRequest): HttpResponse =
      match JSON::parse(request.body) with
      | Ok(payload) ->
          let paymentId = payload.payment_id
          let status = payload.status
          Database::updateInCollection("payments", paymentId, { status: status })
          { statusCode: 200, headers: {}, body: "OK" }
      | Error(_) ->
          { statusCode: 400, headers: {}, body: "Invalid payload" }
    ```

## Error Handling Best Practices

23. **Always handle Option and Result types** explicitly. DarkLang forces you to deal with potential failures.
    ```darklang
    fun safeUserLookup(userId: String): Result<User, String> =
      match Database::get("user_" + userId) with
      | Some(userData) ->
          match JSON::parse(userData) with
          | Ok(user) -> Ok(user)
          | Error(_) -> Error("Invalid user data in database")
      | None -> Error("User not found")
    ```

24. **Chain operations safely** using Result and Option combinators.
    ```darklang
    fun getUserEmail(userId: String): Result<String, String> =
      Database::get("user_" + userId)
      |> Result::fromOption("User not found")
      |> Result::andThen(JSON::parse)
      |> Result::map(fun user -> user.email)
      |> Result::mapError(fun _ -> "Failed to get user email")
    ```

## Testing Your Code

25. **Unit tests** use the `test` keyword. Tests run automatically when you save your code.
    ```darklang
    test "greet function works correctly" =
      let result = greet("World")
      Expect::equal(result, "Hello, World!")
    
    test "divide by zero returns error" =
      let result = divide(10, 0)
      match result with
      | Error(_) -> Expect::pass()
      | Ok(_) -> Expect::fail("Expected error for division by zero")
    ```

26. **HTTP handler testing** can simulate requests and check responses.
    ```darklang
    test "hello endpoint returns 200" =
      let mockRequest = { 
        method: "GET", 
        path: "/hello", 
        headers: {},
        body: "",
        pathParams: {},
        queryParams: {}
      }
      let response = hello(mockRequest)
      Expect::equal(response.statusCode, 200)
    ```

## Performance Considerations

27. **Avoid unnecessary data copying**. Lists and records are immutable but DarkLang optimizes common operations.
    ```darklang
    // Good: Use built-in functions that are optimized
    let processedData = 
      largeList
      |> List::filter(isValid)
      |> List::map(transform)
      |> List::take(100)
    
    // Avoid: Multiple separate operations that copy data
    let filtered = List::filter(largeList, isValid)
    let mapped = List::map(filtered, transform)  
    let limited = List::take(mapped, 100)
    ```

28. **Database operations** should be batched when possible. Each database call has overhead.
    ```darklang
    // Good: Batch operations
    let userIds = ["1", "2", "3", "4", "5"]
    let users = Database::getBatch(userIds)
    
    // Avoid: Individual database calls in a loop
    let users = List::map(userIds, fun id -> 
      Database::get("user_" + id)  // Multiple DB calls
    )
    ```

## Common Patterns and Idioms

29. **The Option pattern** for handling missing data safely throughout your application.
    ```darklang
    fun processUserAction(userId: String, action: String): Result<String, String> =
      let userOpt = Database::get("user_" + userId)
      let user = Option::withDefault(userOpt, { name: "Anonymous", permissions: [] })
      
      if List::contains(user.permissions, action) then
        Ok("Action " + action + " completed for " + user.name)
      else
        Error("User does not have permission for " + action)
    ```

30. **Resource cleanup** pattern for managing external resources like API connections.
    ```darklang
    fun withDatabaseConnection(operation: DatabaseConnection -> Result<T, String>): Result<T, String> =
      let connection = Database::connect()
      let result = operation(connection)
      Database::close(connection)
      result
    
    // Usage
    let userData = withDatabaseConnection(fun conn ->
      Database::query(conn, "SELECT * FROM users WHERE active = true")
    )
    ```

**Key DarkLang Principles:**
- **Deployless**: Code runs immediately without build steps
- **Functional**: Immutable data, pure functions, and explicit error handling
- **Simple**: Minimal syntax with powerful built-in operations
- **Integrated**: Editor, runtime, and infrastructure work together
- **Safe**: Strong types and explicit error handling prevent common bugs

**Getting Started Tips:**
1. Start with simple HTTP handlers to understand the request/response flow
2. Use pattern matching extensively - it's the idiomatic way to handle different cases
3. Always handle Option and Result types explicitly
4. Test your functions as you write them using the built-in test framework
5. Keep functions small and focused on single responsibilities