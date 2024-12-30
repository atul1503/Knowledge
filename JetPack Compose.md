1. You declare UI using `Composable` annotated functions.
2. Inside each composable function, you can call other composable functions to declare your UI in a declarative format.
   ```
     @Composable
     fun MyApp(){
         val context= LocalContext.current
         Column() {
             Text("Hiya sir ${State.Name.current}")
             Button(onClick = {
                 context.startActivity(Intent(context, MainActivity::class.java))
             }) {
                 Text("Go Back to first activity")
             }
         }
     }
   ```
  Here, Text() and Button is also a composable function.

3. You cannot call composable function from a non composable function. Only exception is the `setContent` inside the `onCreate` method of an activity class. That means you cannot call composable function from event handlers like onClick.

   ## How to declare global or scoped state for your app.
   
   1. Declare an object of type `ProvidableCompositionLocal<T>` which you will later use to access the state value using the `current` property of this object.
      ```
      val Name: ProvidableCompositionLocal<String> = compositionLocalOf{"Atul"}
      ```
      Here, String is the type of your state and `Atul` is its default value.
      This variable can be independant of any activity for convinience. Better declare these states in a seperate `.kt` file. Here, I declared it in a seperate `State` object as
      ```
      object State {
       val Name: ProvidableCompositionLocal<String> = compositionLocalOf{"Atul"}
      }
      ```
   
   3. Inside the `setContent` call of your `onCreate()` method, call `CompositionLocalProvider()` function, which will actually provide the value for the `ProvidableCompositionLocal<T>` object that we declared in step 1.
   
      ```
      CompositionLocalProvider(State.Name provides "Tikku" ) {
                       Greeting()
                   }
      ```
      The `Greeting` inside the lambda function is the composable function that will be called so that the `Greeting` composable function will have access to this value that is provided by `CompositionLocalProvider`. The value that `Greeting` will get as `State.Name` is `Tikku`. And no other composable is not inside the `Greeting` composable will have access to the `Tikku` value for `State.Name`. 
   
   ### I repeat, only `Greeting` and its children composables will have access to `Tikku` value, all other composables and their children will have access to the default value(i.e `Atul`).
   
   3. You can declare more composables along with the `Greeting` composable. If you want to do that, just call those composables after or before `Greeting` in the same lambda function required by `CompositionLocalProvider`.


4. Objects like `CompositionLocal` exist so that they can be accessed by full heirarchy of all composables who have access to the object. But `Provider`s exist to provide different values for the same `CompositionLocal` which is how you provide scoped and global states.
