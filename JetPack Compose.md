1. You declare UI using `@Composable` annotated functions. These functions are called composables. These are similar to widgets in flutter.
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
  Here, `Text` and `Button` is also a composable function.

3. You cannot call composable function from a non composable function. Only exception is the `setContent` inside the `onCreate` method of an activity class. That means you cannot call composable function from event handlers like onClick.

## How to declare readonly global or scoped state for your app.
   
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


5. Objects like `CompositionLocal` exist so that they can be accessed by full heirarchy of all composables who have access to the object. But `Provider`s exist to provide different values for the same `CompositionLocal` which is how you provide scoped and global states.

## How to create mutable state for your activity

1. Declare a `ViewModel` class seperately and put data for your state in it.
   ```
      class StateViewModel: ViewModel() {
       private val _name= mutableStateOf("Tikku")
       private val _age= mutableStateOf(25)
   
       var name=_name
       var age=_age
   
       fun updateName(newName: String){
           _name.value=newName
           name=_name
       }
   
       fun updateAge(newAge: Int){
           _age.value=newAge
           age=_age
       }
   }
   ```
   Here, name and age are the data that is stored in this view model.
   We have two `val` variables declared for both. One type of variables `_age` and `_name` which are objects of type `MutableState<T>` which actually store data.
   And there are other two `var` variables, which are `age` and `name` which are just copies of the `MutableState<T>` objects explained previously.

   We will use the `var` variables to read the `MutableState<T>` objects in our composables.

   We also have defined functions inside the viewModel which when called update the respective state variables.

   So, in our viewmodel we define the state variables, and then we define the functions which will change them. That's it.

   `MutableState<T>` objects are special type of objects that observe which composables read them and when they are updated, they re-compose those composables. That is why `MutableState<T>` objects are called observables in reactive programming. They implement `Observable` interface.


2. Inside the `setContent` of an activity, just create an object of your `ViewModel` class and pass this object to each composable in the heirarchy.

   ```
   setContent {

            val viewModel=StateViewModel()
            MyApp(viewModel)
                
   }
   ```
   Keep passing this view model object to every child composable to have them access to the viewModel.
   ```
   @Composable
        fun MyApp(viewModel: StateViewModel) {
            Column {
                Greeting(viewModel)

                SecondGreeting(viewModel)
            }

   }
   ```
   With this `ViewModel` object, they can access the value of `MutableState<T>` objects that are part of it, like this:
   ```
   @Composable
   fun SecondGreeting(viewModel: StateViewModel) {
       println("rendered second greeting")
       Text("bye ${viewModel.age.value}")
       Button(onClick = {
           viewModel.age.value+=1
       }) {
   
           Text("Add age")
       }
   }

   ```
   Here, you can see that the composables are able to view the value of the `_age` mutable state object with `viewModel.age.value` as `age` is just a variable that is pointing to `_age` object itself.
   The `_age` mutable state object views this that `SecondGreeting` composable reads its value, so whenever `_age.value` is changed, it re-composes the `SecondGreeting` composable.

## How to perform basic layouts

### For custom positioning of children inside a parent, use `Box` and put its children inside it.
Each UI composable has a `modifier` constructor arg which you can use to modify its ui. But this modifer applies modification based on the order of modifications applied. This order matters a lot. For eg, set position and size first then modify background and borders otherwise it might lead to very unexpected behaviours.
For eg:
```
Box(modifier = Modifier.size(width = 400.dp, height = 400.dp).background(color = Color.Blue)){
        Text(text = "1", modifier = Modifier.offset(x=100.dp,y=100.dp).background(color = Color.Red).size(80.dp))
        Text(text = "2", modifier = Modifier.offset(x=300.dp,y=100.dp))
}
```
Here, for the `Box` and the `Text` composables inside it have their `modifier` set and since each modification returns the same modifier you can chain modifications and expect that the modifications are applied in the same order that you declare. For the `Box` here, size is set first, then background color is set. 

You can use `offset` to postion children wherever you like. Here offset will allow you to offset that much from its original localtion. For eg, here, first `Text` is moved 100 dp to the right and 100 dp down from its original position. 

The original position of all composables is exactly at the top left corner of its parent.


### For simple positioning

By simple positioning, I mean, put children at predefined places like bottom-left,bottom-right,top-left,top-right,center,top-center,bottom-center.
This is done using `align()` method on modifier. Align accepts all these positions as:
```
Text(text = "2", modifier = Modifier.align(Alignment.TopEnd))
```
This is for top-right.
* For top left, you have `Alignment.TopStart`
* For bottom left, you have `Alignment.BottomLeft` 
* ...etc


6. Since composables can be modified with modifiers and modifiers can be different for different order of modifications, you can do really complex things.

For eg, since, there is no simple way to position child from the top right corner of the parent, using the power of order of modifications, you can align the child to the top right corner of the parent and then apply offset using negative x axis value:
```
Text(text = "1", modifier = Modifier.align(Alignment.TopEnd).offset(x=-100.dp,y=200.dp).background(color = Color.Red))
```
