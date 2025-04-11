1. You declare UI using `@Composable` annotated functions. These functions are called composables. These are similar to widgets in flutter.
2. `LaunchedEffect` allows to run side effects. In its lambda, you need to call `launch()` with a dispatcher to un your code in different threads.  
3. Inside each composable function, you can call other composable functions to declare your UI in a declarative format.
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

For each activity, you can keep seperate view models.

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


## How to have simple value animations

To animate most types of values, you can use `animate*AsState` function. Here, `*` can be replaced by `Color`,`Int`,`Double` etc. More of these types are there. Search when you need them.

This function returns a state object of type that is specified by that `*`. You can put this value to any place where you use. For eg, if you animate color then put this value in `Modifier.background(color=color.value)`.

Here, `color` is the object created by `animateColorAsState` function.

For example, see this code: 
```
var targetColor by remember {
        mutableStateOf(Color.Red)
    }
    val color= animateColorAsState(targetValue = targetColor, animationSpec = tween(durationMillis = 2000) )

    Box(modifier = Modifier.background(color = color.value)){
        Button(onClick = {
            if(targetColor == Color.Red){
                targetColor=Color.Blue
            }
            else{
                targetColor=Color.Red
            }
        }) {
            Text(text = "Click me")
        }
    }
```
Here, `color` variable is the Animated State object of type `Color`. 
In animation, 
* There are two values, A and B.
* A is a value that is trying to become equal to B, but not immediately.
* A eventually becomes equal to B.
* Here, `B is targetValue` and `A is color.value`
* `color.value` is trying to become equal to `targetValue`.
* A is always trying to become equal to B. If you change B, A will also start to become equal to this new B.
* We cannot control what should be A's value at any time, but we can change B anytime if we declare B as a state variable.

Here, in the above example, we are changing B in the onClick event handler.

 
How long does it take A to become B, that we can define by providing `animationSpec` arg of `animate*AsState`. This takes an object called `tween` which describes how A becomes B.
**Remember** The first `targetValue=` you provide becomes the initial value of the animation. That is why you need the `targetValue=` as a state instead of a hardcoded value.

## How to control animations globally or centrally

Since it is pointless in most cases to control the animation state directly since you can control the target state itself.
So, to control the animation centrally or globally, you can put animation target states in a viewModel itself. And reference those target states in the `targetValue` arg itself.
Like this:
```
@Composable
fun AnimateHolder(animateViewModel: AnimateViewModel){
    val offset=animateOffsetAsState(targetValue = animateViewModel.b1TargetOffset.value, animationSpec = tween(durationMillis = 6000))
    Box(modifier = Modifier
        .offset(x = offset.value.x.dp, y = offset.value.y.dp)
        .background(color = Color.Green)
        .size(80.dp))

}
```
Here as you can see, we are referencing the value of `MutableState<Offset>` in the `targetValue` arg as `animateViewModel.b1TargetOffset.value`.`animateViewModel` is the viewmodel.`b1TargetOffset` is the target state. This `MutableState<Offset>` is declared in a view model. Since this is a mutable state and is stored in a view model we can access and change it anywhere we want.
I would say its best if you have two seperate view models for animation and data and pass each of these models to every composable in your activity.

## How to setup database in jetpack compose
In android ,for database, you need `Room` which is the ORM for sqlite. Room can directly create database and perform crud and queries.

To setup room, you need to have these changes in the `build.gradle` in your project module.
* Add this plugin to handle annotations in kotlin, as kotlin doesnot handle annotations.
  ```
  plugins {
   // other plugins
   id 'kotlin-kapt'
  }
  ```
* You need these dependencies for room:
```
def room_version = "2.5.0"

implementation "androidx.room:room-runtime:$room_version"
kapt "androidx.room:room-compiler:$room_version"
implementation "androidx.room:room-ktx:$room_version"
// other dependencies
```

* For defining entities, declare them as data class with `@Entity` annotation:
```
@Entity(tableName = "Person")
data class Person(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val age: Int
)
```

* Create a interface with `Dao` annotation and annotate each method with different operation annotations:
```
@Dao
interface PersonDao {
    @Insert
    fun insertPerson(person: Person): Long

    @Delete
    fun deletePerson(person: Person)

    @Update
    fun updatePerson(person: Person)

    @Query("SELECT * FROM Person WHERE id = :uid")
    fun getPerson(uid: Int): Person
}

```
`@Insert`: for inserting. Method needs entity object.
`@Delete`: for deleting. Method needs entity object.
`@Update`: for updating. Methods need entity object.
`Query`: for performing custom query which includes, select, delete ,update etc. You can pass any arg which can be accessed in the query with `:nameOfArg`. 
All of these methods don't need return types.

Dao means data access object. You should get this object to perform operations on database. Room uses the method signature to automatically implement class during runtime for this interface.

* Create the database abstract class: 
```
@Database(entities = [Person::class], version = 1)
abstract class AppDatabase: RoomDatabase(){
    abstract fun personDao(): PersonDao
}
```

An instance of this class when created represents the database. This abstract class will be implemented by Room during runtime.

@Database annotation defines what entities you have for this database. This is required to specify which entity should be part of this database.
This class needs a method signature to get the dao assicated with this database. This also means that you can get the dao object from object of this class using this method to perform operations on this database.

* Create a static object of this database in your code where you want to access the database:
```
object DatabaseProvider {
    @Volatile
    private var instance: AppDatabase? = null
    public fun getDatabase(context: Context): AppDatabase {
        return instance ?: synchronized(this) {
            val db = Room.databaseBuilder(
                context.applicationContext,
                AppDatabase::class.java,
                "person_database"
            ).build()
            instance = db
            db
        }
    }
}
```

This is an `object` since you only need one object of it in your entire application and can use it anywhere in your application.

`@Volatile` annotation is used in multi threaded programs in kotlin to make sure that whatever variable or object has this annotation, its latest value will always be updated to all the threads that use it. This is made sure by language itself. We just need to annoate the variable with this.

`getDatabase` method will get the database object.
`Room.databaseBuilder()` creates the database object. It uses the context of the application so that the database can be stored in the application internal storage.
It also needs the class object of the database class we defined earlier to get the information about the database like what interface or abstract classes to implement and create assocated objects from them.
Lastly, it also needs the name of the dabase file that will generate for this. Here, `person_database`.
Calling `build()` starts all this process of creation.
`synchronized` keyword on a method is used to make sure that only thread can call this function at a time.
`B = A ?: C()` this ensure that  set value of `B` to `A` if `A` is not null. If `A` is null, then execute `C()` function and set the value of `B` to what `C()` returns.

All this is to ensure that in a multi threaded application, only one instance of the database is created and used.

## How to use keys in jetpack

Keys are used to identify composables uniquely.
For that, just wrap each UI portion with `key(keyValue) { }`.
Like:
```
for(ex in Exercises) {
   key(ex.name) {
      Text(ex.name)
      Text(ex.steps)
   }
}
```
Now compose will treat each key block content will be treated as one and this will help in avoiding multiple exercises of same name.

**What happens if you don't use keys:**
1. Multiple UI portions will have same values.
2. If you edit one portion, other portions may also get modified.

So its better to use keys.


## How to use coroutines or perform long running background tasks
1. Use `lifecycleScope.launch()` to create coroutines that will live as long as activity or fragment is not destroyed. This is like `CoroutineScope` in kotlin but   you don't have to explicitely cancel the coroutines created from this. Canceling and other types of management will be taken care by the acitivty or fragment itself.
2. Use `GlobalScope.launch()` to create coroutines that will as long as the application itself. You don't need to manage thier canceling or management.
3. Use `rememberCoroutineScope` to create scope that will be managed by the composable. Should be used only inside composables. Will live as long as the composable is not destroyed(i.e composable is not removed from ui tree).
4. You can use `CoroutineScope` but remember you have to call `cancel()` on it when you have no use of it. Once you call `cancel()` on it ,all the jobs created from this scope will be cancelled. So, you have to manage its lifecycle manually.


For any scopes, its better to provide the `launch()` with `Dispatchers.IO` or other non-main dispatcher since your task may block the main thread.


## How to form relationships in room databases

Room does not allow defining relationships inside entity classes. To define a relation, you have to create a data class:

```
data class UserAndPost {

   @Embedded
   val user: User,

   @Relation( parentColumn="username", entityColumn="PostOwnerUsername" )
   val posts: List<Post>

}
```

Here, `username` is a field in `User` entity class and `PostOwnerUsername` is a field in `Post` entity class. 
So this means that with `@Relation` annotation, we are relating the fields of both entities togather, thereby establishing a relation.


Regarding why we use `@Embedded` as `User` is because if this data class type(`UserAndPost`) is used as return type in any DAO query, then that means, you have to just define the query to get only the `User` object and then using the `Relation` defined above, Room is intelligent enough to also fetch the associated `List<Post>` associated to the user object.

For eg, here is a DAO query: 
```
interface UserDao {
   @Transaction
   @Query("SELECT * FROM User WHERE username= :username")
   abstract fun getUserAndTheirPosts(String username): UserAndPost
}
```

Since the query is asking for `User` entity to be fetched but declaring `UserAndPost` as the return type of the query, then that sends a signal to Room to first fetch `User` objects and then execute another query to fetch the `Post`s related to this `User`. And then room will combine the `User` fields and that `User`'s associated list of `Post` fields as fields of `UserAndPost` object that you will finally get as the result of executing this query.

This means that the final `UserAndPost` object that you will get as a result of the query, will let you do this:
```
val user = userAndPostObject.user.username
val posts: List<Post> = userAndPostObject.posts.postText
```

Don’t be confused about why the `parentColumn` argument in the `@Relation` annotation refers to a field from the `User` entity (like `username`), and not a field from the `Post` entity (like `PostOwnerUsername`).

This is because the `User` field in `UserAndPost` is annotated with `@Embedded`, which tells Room that `User` is the parent entity.

As a result, the DAO query must select from the `User` table, and Room will then use the mapping defined in `@Relation` to automatically fetch and match the related Post rows based on the specified columns.

You have to use `@Transaction` annotation for these query since internally it is calling 2 queries and we want those queries to be a part of single transaction.

*Remember that DAO interface is not tied to any enitity so that in a single DAO interface you can define queries for all enitities that you have in the DB. But it's not recommended to do that.*
