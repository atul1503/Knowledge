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
