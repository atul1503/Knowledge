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

21. For Android projects, the `build.gradle.kts` looks different because Android has its own build system on top of Gradle:
    ```kotlin
    plugins {
        id("com.android.application")  // For apps. Use "com.android.library" for libraries
        kotlin("android")
        kotlin("kapt")  // For annotation processing (Room, Dagger, etc.)
    }
    
    android {
        namespace = "com.example.myapp"  // Replaces applicationId in manifest
        compileSdk = 34                  // SDK version to compile against
        
        defaultConfig {
            applicationId = "com.example.myapp"  // Unique app identifier (like bundle ID)
            minSdk = 24                          // Minimum Android version supported
            targetSdk = 34                       // Target Android version (latest features)
            versionCode = 1                      // Internal version number (increment for updates)
            versionName = "1.0"                  // User-visible version string
            
            testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        }
        
        buildTypes {
            debug {
                isDebuggable = true
                applicationIdSuffix = ".debug"   // Different app ID for debug builds
                versionNameSuffix = "-DEBUG"
            }
            release {
                isMinifyEnabled = true           // Shrink code and resources
                isShrinkResources = true         // Remove unused resources
                proguardFiles(
                    getDefaultProguardFile("proguard-android-optimize.txt"),
                    "proguard-rules.pro"
                )
                signingConfig = signingConfigs.getByName("release")  // Sign for Play Store
            }
        }
        
        // For different app flavors (free vs paid, different backends, etc.)
        flavorDimensions += "version"
        productFlavors {
            create("free") {
                dimension = "version"
                applicationIdSuffix = ".free"
                versionNameSuffix = "-free"
            }
            create("paid") {
                dimension = "version"
                applicationIdSuffix = ".paid"
                versionNameSuffix = "-paid"
            }
        }
        
        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_1_8
            targetCompatibility = JavaVersion.VERSION_1_8
        }
        
        kotlinOptions {
            jvmTarget = "1.8"
        }
        
        buildFeatures {
            viewBinding = true    // Generate binding classes for layouts
            dataBinding = true    // Enable data binding
            compose = true        // Enable Jetpack Compose
        }
        
        composeOptions {
            kotlinCompilerExtensionVersion = "1.5.4"
        }
    }
    
    dependencies {
        implementation("androidx.core:core-ktx:1.12.0")
        implementation("androidx.appcompat:appcompat:1.6.1")
        implementation("com.google.android.material:material:1.9.0")
        implementation("androidx.constraintlayout:constraintlayout:2.1.4")
        
        // Jetpack Compose
        implementation(platform("androidx.compose:compose-bom:2023.10.01"))
        implementation("androidx.compose.ui:ui")
        implementation("androidx.compose.material3:material3")
        implementation("androidx.activity:activity-compose:1.8.0")
        
        // Testing
        testImplementation("junit:junit:4.13.2")
        androidTestImplementation("androidx.test.ext:junit:1.1.5")
        androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    }
    ```
    
    Key differences from regular Kotlin projects:
    * `android` block configures Android-specific settings
    * `minSdk`/`targetSdk` define Android version compatibility
    * Build types (debug/release) with different configurations
    * Product flavors for app variants
    * Special dependencies like `androidx` libraries
    * Signing configuration for Play Store releases

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

24. For multiplatform projects, Kotlin can run on JVM, Android, iOS, JavaScript, and Native platforms. This means you can share code between different platforms while still having platform-specific code when needed:
    ```kotlin
    plugins {
        kotlin("multiplatform")
        id("com.android.library")  // If you want Android support
    }
    
    kotlin {
        // Define target platforms
        jvm {
            jvmToolchain(8)
            withJava()  // Include Java sources
        }
        
        android {
            compilations.all {
                kotlinOptions {
                    jvmTarget = "1.8"
                }
            }
        }
        
        js(IR) {
            browser {
                commonWebpackConfig {
                    cssSupport {
                        enabled.set(true)
                    }
                }
            }
            nodejs()
        }
        
        // iOS targets
        iosX64()
        iosArm64()
        iosSimulatorArm64()
        
        // Native targets (optional)
        linuxX64()
        mingwX64()
        macosX64()
        
        sourceSets {
            // Common code that works on all platforms
            val commonMain by getting {
                dependencies {
                    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
                    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
                    implementation("io.ktor:ktor-client-core:2.3.5")
                }
            }
            
            val commonTest by getting {
                dependencies {
                    implementation(kotlin("test"))
                }
            }
            
            // JVM-specific code
            val jvmMain by getting {
                dependencies {
                    implementation("io.ktor:ktor-client-cio:2.3.5")  // JVM HTTP client
                }
            }
            
            // Android-specific code
            val androidMain by getting {
                dependencies {
                    implementation("androidx.core:core-ktx:1.12.0")
                    implementation("io.ktor:ktor-client-android:2.3.5")  // Android HTTP client
                }
            }
            
            // JavaScript-specific code
            val jsMain by getting {
                dependencies {
                    implementation("io.ktor:ktor-client-js:2.3.5")  // JS HTTP client
                }
            }
            
            // iOS-specific code (shared between all iOS targets)
            val iosMain by creating {
                dependsOn(commonMain)
                dependencies {
                    implementation("io.ktor:ktor-client-darwin:2.3.5")  // iOS HTTP client
                }
            }
            
            // Connect iOS targets to shared iOS source set
            val iosX64Main by getting { dependsOn(iosMain) }
            val iosArm64Main by getting { dependsOn(iosMain) }
            val iosSimulatorArm64Main by getting { dependsOn(iosMain) }
        }
    }
    
    android {
        namespace = "com.example.shared"
        compileSdk = 34
        defaultConfig {
            minSdk = 24
        }
    }
    ```
    
    Project structure for multiplatform:
    ```
    src/
    ├── commonMain/kotlin/          # Code that works everywhere
    │   └── com/example/
    │       ├── ApiClient.kt        # Shared business logic
    │       └── DataModel.kt        # Shared data classes
    ├── commonTest/kotlin/          # Tests for common code
    ├── jvmMain/kotlin/             # JVM-specific code
    │   └── com/example/
    │       └── DatabaseHelper.kt   # JVM database access
    ├── androidMain/kotlin/         # Android-specific code
    │   └── com/example/
    │       └── AndroidUtils.kt     # Android UI helpers
    ├── jsMain/kotlin/              # JavaScript-specific code
    │   └── com/example/
    │       └── DomUtils.kt         # DOM manipulation
    └── iosMain/kotlin/             # iOS-specific code
        └── com/example/
            └── IOSUtils.kt         # iOS native interop
    ```
    
    Using `expect` and `actual` for platform-specific implementations:
    ```kotlin
    // In commonMain - declare what you expect each platform to provide
    expect class Platform() {
        val name: String
    }
    
    expect fun getCurrentTimestamp(): Long
    
    // In jvmMain - provide JVM implementation
    actual class Platform {
        actual val name: String = "JVM"
    }
    
    actual fun getCurrentTimestamp(): Long = System.currentTimeMillis()
    
    // In jsMain - provide JavaScript implementation
    actual class Platform {
        actual val name: String = "JavaScript"
    }
    
    actual fun getCurrentTimestamp(): Long = js("Date.now()") as Long
    
    // In iosMain - provide iOS implementation
    actual class Platform {
        actual val name: String = "iOS"
    }
    
    actual fun getCurrentTimestamp(): Long = NSDate().timeIntervalSince1970.toLong() * 1000
    ```
    
    Common multiplatform use cases:
    * **Shared business logic**: API clients, data models, validation
    * **Mobile apps**: Share logic between Android and iOS
    * **Full-stack**: Share models between server (JVM) and client (JS)
    * **Libraries**: Create libraries that work on multiple platforms
    
    Build commands for multiplatform:
    ```bash
    ./gradlew build                    # Build all targets
    ./gradlew jvmTest                  # Test JVM target only
    ./gradlew jsTest                   # Test JavaScript target only
    ./gradlew iosX64Test               # Test iOS simulator
    ./gradlew publishToMavenLocal      # Publish to local Maven repo
    ```
    
    Benefits of multiplatform:
    * Write business logic once, use everywhere
    * Type-safe sharing between platforms
    * Platform-specific optimizations when needed
    * Gradual adoption - start with small shared modules

25. Gradle wrapper (`gradlew`) is included in projects so you don't need to install Gradle globally. It downloads the right version automatically.

26. Common Gradle gotchas:
    * Use `./gradlew` on Mac/Linux, `gradlew.bat` on Windows
    * If build fails, try `./gradlew clean build`
    * Gradle caches everything, sometimes you need to clear cache with `./gradlew clean`
    * Dependencies can conflict - use `./gradlew dependencies` to debug

27. Gradle build phases: initialization → configuration → execution. Your build scripts run during configuration phase.

28. You can create custom tasks:
    ```kotlin
    tasks.register("hello") {
        doLast {
            println("Hello from custom task!")
        }
    }
    ```
    Run with: `./gradlew hello`

29. For faster builds, add these to `gradle.properties`:
    ```properties
    org.gradle.daemon=true
    org.gradle.parallel=true
    org.gradle.caching=true
    kotlin.code.style=official
    ```

Remember: Gradle is powerful but can be confusing. Start with a simple `build.gradle.kts`, add dependencies as needed, and use `./gradlew build` to compile. Most of the time that's all you need.

## Generating HTML Documentation with Dokka

30. Dokka is a documentation generation tool for Kotlin projects. It reads your code comments and generates beautiful HTML documentation websites. Think of it like Javadoc for Java but designed specifically for Kotlin.

31. To use Dokka, first add the Dokka plugin to your `build.gradle.kts` file:
    ```kotlin
    plugins {
        kotlin("jvm") version "1.9.10"
        id("org.jetbrains.dokka") version "1.9.10"  // Add this line
    }
    ```
    The `id("org.jetbrains.dokka")` tells Gradle to use the Dokka plugin. The version `"1.9.10"` specifies which version of Dokka to use - make sure it matches or is compatible with your Kotlin version.

32. For multi-module projects, add Dokka to each module that needs documentation:
    ```kotlin
    // In each module's build.gradle.kts
    plugins {
        kotlin("jvm")
        id("org.jetbrains.dokka")
    }
    ```

33. To generate HTML documentation, run this command in your terminal:
    ```bash
    ./gradlew dokkaHtml
    ```
    The `dokkaHtml` is a Gradle task that the Dokka plugin creates automatically. When you run this command, Dokka will read all your Kotlin files and generate HTML documentation.

34. The generated HTML documentation will be located in specific folders:
    ```
    yourProjectName/build/dokka/html/           # For single-module projects
    yourProjectName/yourModule/build/dokka/html/  # For multi-module projects
    ```
    
    For example, if your project is called "MyApp" with a module called "core", the docs will be at:
    ```
    MyApp/core/build/dokka/html/index.html
    ```
    
    The `build` folder is where Gradle puts all compiled and generated files. The `dokka` folder contains Dokka's output, and `html` contains the website files. Open `index.html` in your web browser to view the documentation.

35. Dokka only generates documentation for your project code, not for external libraries you depend on. This means if your code uses classes from dependencies like OkHttp or Gson, those won't have clickable links in your documentation. However, there are exceptions for well-known frameworks:
    
    * **Android**: Types like `Activity`, `ViewModel`, `Fragment`, `View`, etc. will have working links to Android documentation
    * **Kotlin Standard Library**: Types like `String`, `List`, `Map` will link to Kotlin docs
    * **Java Standard Library**: Types like `ArrayList`, `HashMap` will link to Java docs
    
    Example: If your function returns an `Activity`, the generated docs will link to Android's official Activity documentation. But if it returns a `OkHttpClient`, there won't be a link to OkHttp's documentation.

36. To write good documentation comments that Dokka can use, follow this format:
    ```kotlin
    /**
     * Calculates the total price including tax for a list of items.
     * 
     * This function takes a list of item prices and applies the given tax rate
     * to calculate the final total that a customer needs to pay.
     * 
     * @param prices A list of individual item prices in dollars
     * @param taxRate The tax percentage as a decimal (0.1 for 10% tax)
     * @return The total price including tax, rounded to 2 decimal places
     * 
     * @throws IllegalArgumentException if taxRate is negative
     * 
     * @sample
     * ```
     * val itemPrices = listOf(10.99, 25.50, 5.99)
     * val total = calculateTotal(itemPrices, 0.08)  // 8% tax
     * println(total)  // Prints: 45.94
     * ```
     */
    fun calculateTotal(prices: List<Double>, taxRate: Double): Double {
        require(taxRate >= 0) { "Tax rate cannot be negative" }
        val subtotal = prices.sum()
        return (subtotal * (1 + taxRate)).round(2)
    }
    ```
    
    Documentation comment structure:
    * `/**` starts a documentation comment (three asterisks, not two)
    * First paragraph is a brief description
    * `@param` explains what each parameter does
    * `@return` explains what the function returns
    * `@throws` explains when exceptions are thrown
    * `@sample` provides example usage code
    * `*/` ends the documentation comment

37. For classes, document the class purpose and important properties:
    ```kotlin
    /**
     * Represents a user account in the application.
     * 
     * This class stores user information and provides methods to update
     * user preferences and validate account status.
     * 
     * @property userId Unique identifier for the user account
     * @property email User's email address, used for login
     * @property isActive Whether the account is currently active
     * 
     * @constructor Creates a new user account with the given details
     */
    class UserAccount(
        val userId: String,
        val email: String,
        var isActive: Boolean = true
    ) {
        /**
         * Updates the user's email address.
         * 
         * @param newEmail The new email address to set
         * @throws IllegalArgumentException if email format is invalid
         */
        fun updateEmail(newEmail: String) {
            // Implementation here
        }
    }
    ```

38. You can customize Dokka's output by configuring it in your `build.gradle.kts`:
    ```kotlin
    tasks.dokkaHtml.configure {
        dokkaSourceSets {
            named("main") {
                moduleName.set("My Awesome Library")                    // Custom module name
                moduleDescription.set("A library that does amazing things")  // Module description
                
                // Add external documentation links
                externalDocumentationLink {
                    url.set(URL("https://docs.oracle.com/javase/8/docs/api/"))
                    packageListUrl.set(URL("https://docs.oracle.com/javase/8/docs/api/package-list"))
                }
                
                // Include source code links
                sourceLink {
                    localDirectory.set(file("src/main/kotlin"))
                    remoteUrl.set(URL("https://github.com/yourname/yourproject/tree/main/src/main/kotlin"))
                    remoteLineSuffix.set("#L")
                }
            }
        }
    }
    ```
    
    Configuration options explained:
    * `moduleName.set()` changes the title shown in the documentation
    * `moduleDescription.set()` adds a description at the top of the docs
    * `externalDocumentationLink` adds links to external library documentation
    * `sourceLink` adds "Source" links that point to your code on GitHub

39. For multi-module projects, you can generate combined documentation:
    ```kotlin
    // In root build.gradle.kts
    plugins {
        id("org.jetbrains.dokka")
    }
    
    tasks.dokkaHtmlMultiModule.configure {
        moduleName.set("My Complete Project")
        includes.from("README.md")  // Include project README in docs
    }
    ```
    
    Then run:
    ```bash
    ./gradlew dokkaHtmlMultiModule
    ```
    
    This creates a single documentation site that includes all your modules, with a navigation menu to switch between them.

40. Common Dokka commands:
    ```bash
    ./gradlew dokkaHtml              # Generate HTML docs for current module
    ./gradlew dokkaHtmlMultiModule   # Generate combined docs for all modules
    ./gradlew clean dokkaHtml        # Clean and regenerate docs
    ```

41. If the standard `dokkaHtml` or `dokkaMultiProjectHtml` tasks don't work (maybe due to version conflicts or plugin issues), you can use an alternative task:
    ```bash
    ./gradlew dokkaGeneratePublicationHtml
    ```
    
    The `dokkaGeneratePublicationHtml` is another Gradle task provided by Dokka that generates HTML documentation for each module in their respective build directories. This task is sometimes more reliable when the standard tasks fail.
    
    After the documentation generation completes, this task will automatically print a clickable link to the `index.html` file in your terminal. You can click this link or copy-paste it into your browser to view the generated documentation. The link will look something like:
    ```
    file:///path/to/yourProjectName/yourModule/build/dokka/html/index.html
    ```

42. Tips for better documentation:
     * Write clear, simple descriptions that explain the "why", not just the "what"
     * Include examples in `@sample` tags - they're very helpful for users
     * Document parameters and return values thoroughly
     * Use `@see` tags to reference related functions or classes
     * Update documentation when you change code - outdated docs are worse than no docs

Remember: Good documentation makes your code easier to use and maintain. Dokka makes it easy to create professional-looking documentation that stays up-to-date with your code.
