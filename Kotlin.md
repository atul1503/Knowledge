1. You can create everything in one file. Whether it be multiple functions, or multiple public or private classes under a single `.kt` file.
2. You can declare a static class with `object` keyword and use it directly without creating any object.
3. All classes are `final` by default. If you want to subclass them then use `open` keyword as `open class Animal`
4. `;` is not necessary. Only necessary if you write code in same line.
5. `val` means variable is immutable. Meaning that whatever object it was pointing at the first time , it will point to it only till its deleted.
   ```
    val myvar = Intent() // need to put before the variable name.
   ```      
6. `var` mean variable can point to multiple objects in its lifecycle.
7. No ned to declare types everytime. If you declare a variable to point to some object and if that objects' type is known then no need to specify type of variable.
       For example.
     ```
        var myvar = Intent(context,myActivity)
     ```
     This means that myvar is pointing to an `Intent` object and since its type is known now that means throughout the lifecycle of `myvar` it will point to an object of `Intent` type only.

8. Even if you want to put type of variable then do it like this:
   ```
   var myScore: Int = 5;
   ```
9. Also functions are define with `fun` keyword.
    ```
    fun myFunc(myScore: Int,myName: String): Unit{
      println("Hi")
    }
    ```
    `Unit` means void. Here, this function returns void or nothing. It takes in an int variable but it also takes in a String.

10. Lambda functions are just anonymous functions. They are represented as:
    ```
    val myFunc : () -> Unit = { println("Hi) } 
    ``` 
11. The last expression in a function is returned by the function so there is no need of `return` keyword.
    ```
    val myFunc: () -> Int = { 5 } // 5 is returned.
    ```
12. Also, functions can take in lambda functions as arg, usually they are last args to a function:

   ```
   func myFunc(myval: Int,callthis: () -> Unit): Unit //myfunc takes an integer and a lambda function
   ```

   So to call this, you do this:
   
   ```
   myFunc(5) { //some shit }
   ```
   So this `{ //someshit }` is a lambda function that you are passing to `myFunc`

13. lambda functions sometimes may not have a return type declared. It is inferred then.
    ```
    val lambda= { x: Int, y: Int -> x + y }
    ```
14. Or sometimes they may have the types for lambda declared:
    ```
    val lambda: (String, Int) -> Double = { str, num -> 2.0 }
    ```
14. Label are put on functions before the `fun` keyword.
    ```
    myl@ fun myfunc(x: Int): Int {
        return@myl x * 2
    }
    ```
    This is true even for anonymous functions since even anonymous functions have `fun` keyword.
15. But for lambda functions, labels are declared before the `{}`.
    ```
    val lambda=  lamba@{ x: Int, y: Int -> return@lamba x + y }
    ```
16. You can pass lambdas to functions like :
    ```
      myFunc(someInt: Int, {  print(message)  })
    ```
      Here, closure of `message` is taken and used. The caller, `myFunc` may call this lambda anytime in its body and that time the value of `message` will be the value at the time of lambda definition.

## Working with coroutines

`Coroutine`s help you to execute lambda functions in different threads. But instead of specifying or creating threads of diffrent types, you specify `Dispatchers` type while creating a `CoroutineScope` or `launch`ing an already created coroutine.

This is how you create a `Coroutine`:
```
val scope = CoroutineScope(Dispatchers.IO) 
```
This creates a `CoroutineScope` which means the coroutine that will be launched later, will use this scope. Scope here means that all the variables and objects accessible in the current code(where this scope is created) will also be accessible in the coroutine code(the lambda function that will be called in the coroutine or thread). 

The `Dispatchers.IO` here means that the coroutine code will only be called in threads which are I/O related threads and different from the main thread. There are 3 or 4 types of `Dispatcher`s.

Now, to specify the lambda function for you coroutine, do this:
```
scope.launch {
\\ this is your lambda function for the coroutine which will run in a seperate thread
}
```

So, the `launch` function requires a lambda function which will be called on seperate thread. The `launch` function also accepts a `Dispatcher` argument which specifies the type of dispatcher to use. This is only if you want to use different dispatcher for your code than the default. The default is the dispatcher you specified while creating the `CoroutineScope`. 

* The IO dispatcher and other types of dispatcher run on different thread than the main thread so they dont block the main thread. But if by mistake or on purpose you are using the main dispatcher, the functions you can call in your lambda can only be `suspend` functions.

* `suspend` functions are functions that can pause their execution and resume it later.

* `launch` function returns a `Job` object which symbolizes a coroutine object. You can cancel this `Job` whenever you want and do other stuff also on it.


In jetpack compose, you would write like this:
```
val scope=CoroutineScope(Dispatchers.IO)
val state= remember { mutableStateOf(55) }

Button(onClick= {
   scope.launch {
      state.value++
      println("${state.value}")
}
 }) { Text("Click me") }
```

As you can see that the variables declared in the scope where the `scope` `CoroutineScope` object was created, are also accessible in the lambda function passed to the launch method as part of closure.


* In kotlin or whatever programming language which has concept of closures, the closure stores the memory address of the variable itself so that we can reference them directly and correctly in callbacks and lambda functions. This also means that we are reading their actual values and not their stale values. If the variable value is changed, that is also reflected in the callback or lambda function.

* You can use labels in kotlin to decide which loop/function to `continue`,`break` or `return`. Labels are declared like this `mylable@ { return@mylable }`. Here, although this is a very simplest example, but inside nested functions, you can return to a specific function if you label that function and then use that label in the return statement. Here, the lambda marked with `mylable@` has been returned using `return@mylabel`. Same is the case for break and continue. Make sure you declare a label just before the function or loop which will allow kotlin to mark those loops and functions with those labels.  

## Gradle Builds for Kotlin

17. Gradle is the build tool for Kotlin projects. It handles dependencies, compilation, and packaging. Think of it like `npm` for JavaScript or `pip` for Python but more powerful.

18. Basic `build.gradle.kts` file structure for a Kotlin project:
    ```kotlin
    plugins {
        kotlin("jvm") version "1.9.10"
        application
    }
    
    group = "com.example"
    version = "1.0-SNAPSHOT"
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        implementation("org.jetbrains.kotlin:kotlin-stdlib")
        testImplementation("org.junit.jupiter:junit-jupiter:5.9.2")
    }
    
    application {
        mainClass.set("MainKt")  // Your main function file
    }
    ```

19. Common Gradle commands you'll use:
    ```bash
    ./gradlew build          # Compile and build the project
    ./gradlew run            # Run the application
    ./gradlew test           # Run tests
    ./gradlew clean          # Clean build artifacts
    ./gradlew jar            # Create a JAR file
    ./gradlew dependencies   # Show all dependencies
    ```

20. Adding dependencies is simple. Just put them in the `dependencies` block:
    ```kotlin
    dependencies {
        implementation("com.squareup.okhttp3:okhttp:4.11.0")      // HTTP client
        implementation("com.google.code.gson:gson:2.10.1")        // JSON parsing
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")  // Coroutines
        
        testImplementation("org.junit.jupiter:junit-jupiter:5.9.2")
        testImplementation("io.mockk:mockk:1.13.7")               // Mocking for tests
    }
    ```

21. For Android projects, the `build.gradle.kts` looks different:
    ```kotlin
    plugins {
        id("com.android.application")
        kotlin("android")
    }
    
    android {
        compileSdk = 34
        
        defaultConfig {
            applicationId = "com.example.myapp"
            minSdk = 24
            targetSdk = 34
            versionCode = 1
            versionName = "1.0"
        }
        
        buildTypes {
            release {
                isMinifyEnabled = false
                proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"))
            }
        }
    }
    
    dependencies {
        implementation("androidx.core:core-ktx:1.12.0")
        implementation("androidx.appcompat:appcompat:1.6.1")
        implementation("com.google.android.material:material:1.9.0")
    }
    ```

22. Gradle has different dependency types:
    ```kotlin
    dependencies {
        implementation("lib")     // Available at compile and runtime
        api("lib")               // Like implementation but exposes to consumers
        compileOnly("lib")       // Only available at compile time
        runtimeOnly("lib")       // Only available at runtime
        testImplementation("lib") // Only for tests
    }
    ```

23. You can create multiple source sets for different environments:
    ```kotlin
    sourceSets {
        main {
            kotlin.srcDirs("src/main/kotlin")
        }
        test {
            kotlin.srcDirs("src/test/kotlin")
        }
    }
    ```

24. For multiplatform projects (Kotlin can run on JVM, Android, iOS, JS, Native):
    ```kotlin
    plugins {
        kotlin("multiplatform")
    }
    
    kotlin {
        jvm()
        js(IR) {
            browser()
            nodejs()
        }
        
        sourceSets {
            val commonMain by getting {
                dependencies {
                    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
                }
            }
            val jvmMain by getting
            val jsMain by getting
        }
    }
    ```

25. Gradle wrapper (`gradlew`) is included in projects so you don't need to install Gradle globally. It downloads the right version automatically.

26. Common Gradle gotchas:
    * Use `./gradlew` on Mac/Linux, `gradlew.bat` on Windows
    * If build fails, try `./gradlew clean build`
    * Gradle caches everything, sometimes you need to clear cache with `./gradlew clean`
    * Dependencies can conflict - use `./gradlew dependencies` to debug

27. For publishing libraries, add publishing plugin:
    ```kotlin
    plugins {
        kotlin("jvm")
        `maven-publish`
    }
    
    publishing {
        publications {
            create<MavenPublication>("maven") {
                from(components["java"])
            }
        }
    }
    ```

28. Gradle build phases: initialization → configuration → execution. Your build scripts run during configuration phase.

29. You can create custom tasks:
    ```kotlin
    tasks.register("hello") {
        doLast {
            println("Hello from custom task!")
        }
    }
    ```
    Run with: `./gradlew hello`

30. For faster builds, add these to `gradle.properties`:
    ```properties
    org.gradle.daemon=true
    org.gradle.parallel=true
    org.gradle.caching=true
    kotlin.code.style=official
    ```

Remember: Gradle is powerful but can be confusing. Start with a simple `build.gradle.kts`, add dependencies as needed, and use `./gradlew build` to compile. Most of the time that's all you need.
