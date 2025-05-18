## Dependency Injection using Hilt
Hilt is a dependency injection library for android. Android projects need DI just so that we don't need to handle and manage objects and offload those tasks to the DI framework.

### How to setup Hilt:
Hilt is similar to springboot and most of its work is done duirng compile time, so for that you need to add its plugin to project build.gradle file:
```
id 'com.google.dagger.hilt.android' version '2.44' apply false
```
Version can be any stable version. Note that here we are not applying the plugin, just adding it to the project and this gives choice to modules inside the project whether they use it or not.

Now, in your module's build.gradle, add this in plugin section.
```
id 'dagger.hilt.android.plugin'
```
Here we are applying this.

Now, in the dependencies section add these: 
```
implementation "com.google.dagger:hilt-android:2.44"
kapt "com.google.dagger:hilt-compiler:2.44"
```
Version can be any stable version. Kapt plugin also needs to installed as a prerequisite.
   
Dependency can be injected using public field injection or constructor injection using `@Inject` annotation.
**Field injection**:
```
class MainActivity : ComponentActivity() {

    @Inject
    lateinit var repo: MyRepository
```
Remember that since we are not creating the `MyRepository` object here so we need to declare the field as `lateinit var` as Hilt will provide us this during runtime. This field can also not be private for the same reason.

**Constructor Injection**:
```
class MyClass @Inject constructor(var dep1: Dep1, var dep2: Dep2) {
   //your class implementation
}
```
`val`,`var` or `lateinit` restrictions are not here. That depends on your usecase. 

Dependency can be of any type even primitives also. 

In situations where there is a dependency that you want hilt to manage but your project code is using it as a third party dependency(meanign that you didn't implement the code for it), then just like spring boot(where you could create configuration class), you can create a hilt module to define or provide multiple dependencies of such type:
```
@Module
@InstallIn(SingletonComponent::class.java)
class DepProvider

   @Provides
   fun provideDep1(dep1 : Dependency1) {
      return Dependency1()
   }
   
```
So essentially, just like in spring boot we had bean factory functions in configuration classes, we have a hilt module which has dependency factory methods like this. Hilt uses this class to create dependencies when needed.
`@Module` signals the same to hilt that is a hilt module which has functions which provide dependencies.

#### What is `@InstallIn` here?
Hilt creates and maintains several containers of dependencies internally and each of those containers have dependencies that have a common visibility. For eg, there is container for dependencies that are visible across the whole application, there is a container which contains dependencies which have visibility only across the activity it is associated with. So, in short there are containers for each of the android components including services,fragments etc.
Here, with `SingletonComponent::class.java` arg, you are telling Hilt that all dependencies of this hilt module will have visibility across the whole app. 
There are other Component classes like this, one is there for Activity, there is one for Fragment etc. Google it.

#### Are containers same as scopes here?
The containers like `SingletonComponent::class.java` are not a way to define scopes. Since a DI framework needs to know the scope or lifecycle of a dependency, we need to declare scopes for them. Hilt also has concept of scopes and you have two options:
1. Either don't declare scopes for a dependency which will mean that whenever you ask for this dependency, Hilt will create a new instance of it. Old will still be there as long as the scope to which it is tied to doesnot die.
2. Declare a scope. Scopes are also, like containers, of type activity, fragment, viewmodel etc. Google them. When their scoped components die the dependencies will also be put for garbage collection. Meaning that if you have provided a dependency with activity scope and hilt injects that in an activity then when the activity dies, your dependency also dies.

Components and scopes need to be on same level. For eg, if Dependency A is scoped as activity scope then it should also be in ActivityComponent, otherwise you will get compile time error.

#### Scopes declaration example:
```
@ActivityScoped
class MyAnalyticsTracker @Inject constructor(...)


@Module
@InstallIn(ActivityComponent::class)
object MyActivityModule {

    @Provides
    fun provideSomething(): Something = ...
}
```
I know its unrelated but a good example.


#### How to declare multiple dependencies of same type.
Using `@Qualifier` annotations. Simply create your own qualifier annotation with retention policy of binary(annotation will be till compile time).
And tag either your `@Provides` function or  `@Inject` constrcutor or field with this annotation. Now wherver this dependency is required as arg of a constructor or field, also tag it there with this annotation. Like this:
```
//Declaring seperate annotation for tagging.
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BaseUrl

// another annotation declared to for any other dependency
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ThirdPartyUrl


//module which provides the depednency
@Module
@InstallIn(SingletonComponent::class)
object ProviderModule {

    @Provides
    @BaseUrl
    fun provideBaseUrl() = "https://www.google.com"

}

//component which requires a string dependency.
class MyRepository
@Inject
constructor(@BaseUrl val baseUrl: String) {
    fun getData(): String {
        return baseUrl
    }
}
// tagged a string arg as the annotation to signal hilt that it should come which one we are talking about.


@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    @Inject
    lateinit var repo: MyRepository

```


### How to use a hilt view model in a composable
Viewmodel handle business logic, you can call viewmodels directly inside a composable instead of passing viewmodels via composable args and its better to avoid getting viewmodel handle from an activity since your composable can be called from any activity.

For this, hilt requires to declare a viewmodel as hilt view model, by using `@HiltViewModel` like this:
```
@HiltViewModel
class MyViewModel @Inject constructor(repo: MyRepository) : ViewModel() {
    val mystate= MutableStateFlow(repo.getData())
}
```
In the composable, you can directly get instance of the viewmodel like this:
```
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    val vm: MyViewModel= hiltViewModel()
    Text(
        text = "Hello ${vm.mystate.collectAsState().value}!",
        modifier = modifier
    )
}
```

### Hilt Tips:
1. Composables should not interact or get Hilt dependencies directly, they should only have a handle to the viewmodel and ensure that all complex operations of using hilt dependencies for complex business logic should be done in view model only and the composable should just call the viewmodel code to get them done.

## General Tips

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

## Material theme and colors
1. Material theme has `colors` arg.
2. This arg needs to provide `primary`,`secondary`,`onPrimary`,`onSeocndary` colors.
   * `primary`: color of background more often used for many widgets.
   * `secondary`: color of background but less often used.
   * `onPrimary`: text color to use when `primary` is the background.
   * `onSecondary`: text color to use when `secondary` is the background.

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

## Important points about positioning
### `Column`s
1. To align children, use `horizontalAlignment` property to align them horizontally. Meaning defining how will flow left, or right or center.
2. To put space between children, use `verticalArrangement = Arrangement.spacedBy(16.dp)` and specify exact number to specify how much space should be there between two children.

### `Row`s
1. To align children, use `verticalAlignment` property to align them vertically. Meaning, defining how will children flow: left, or right or center.
2. To put space between children, use `horizontalArrangement = Arrangement.spacedBy(16.dp)` and specify exact number to specify how much space should be there between two children.

### `Box`s
1. Only children of the box can have alignment, the complete box cannot have alignment. So use `modifier.align()` to align a box's child inside box.
2. Arrangement or space between children is not straightforward, use either absolute positioning or stack.

### Use `LocalConfiguration.current.screenWidthDp` to get the width of screen. 
Using this you can size 95% of the things so that you dont have to worry about different phone sizes.


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

## How to create background services/tasks that run on the background

1. First create a worker class for your job.
   * This class should be a subclass of `CoroutineWorker` class. This will have a `doWork()` method which you have to override to define what you want your background task to do.
   * This worker class is the class which represents the task that you want to be done.
   * `doWork()` method  body need to be a `withContext(Dispatchers.IO)` function call. This function call accepts a lambda in which you write the code to do your task.
2. Now, there is some point in your application where you tell the android OS to schedule/start your worker. This can be done in `onCreate()` method of your activity.
3. Wherever, you put the code to tell android OS to start your worker, we will call that `SchedPlace`.
4. In the `SchedPlace`, you have to tell android how often you need to run your worker. For that you have to create a work request. There are two types of work request:
   * `OneTimeWorkRequest` which is run only once by android.
   * `PeriodicWorkRequest` which runs forever on intervals that you specify.
5. Creating a work request means specifying constraints about in what conditions should it run and also specify at what interval it should run(in case of `PeriodicWorkRequest`)
6. Work request is created using a work request builder. Based on the type of work request you have to type of builders.
7. Builder also needs to know the class of your worker so that it will run it.
8. After building the worker request with a builder, you enqueue it into a queue of `WorkManager`. The queue is also of two types corresponding to the type of requests.
9. In the enqueue method of the `WorkManager`, you also need to specify a policy to tell work manager to do something if there are multiple workers with same identity(since you also specify the name,tag of the worker when building and enqueing the work request). By providing a correct predefined policy, you tell work manager to either replace the exsiting worker with this new one or let it run or update etc.

Enough with the conceptual theory, here is a simple worker:
```
class NotificationWorker(private val context: Context,params: WorkerParameters): CoroutineWorker(context,params){
   override suspend fun doWork(): Result = withContext(Dispatchers.IO) {
   //do something
   } 
}
```
1. The `WorkerParamters` are actually there if you want to pass some arguments to the worker while scheduling.

Here is how you would build work request:
```
val constraints= Constraints.Builder()
.setRequiredNetworkType(NetworkType.NOT_REQUIRED)
.build()

val workerRequest= PeriodicWorkRequestBuilder<NotificationWorker>(
            15, TimeUnit.MINUTES,
            5, TimeUnit.MINUTES
)
.setConstraints(constraints)
.addTag("RoutineNotificationWorkerTag")
.build()

```
1. You can see we first create a constraints with its builder to specify that on any network type you can go ahead and run any worker on which this constraint is attached. You can create multiple rules in the same constraint.
2. Next, we are building a `PeriodicWorkRequest` with `PeriodicWorkRequestBuilder` which takes your worker class as its generic constructor arg essentially, which will tell the workmanager to invoke `doWork()` of this class when its time to run the work.
3. The `15,TimeUnit.MINUTES`(called the interval time) is being used to convey that work manager should run it every 15 minutes since it is a `PeriodicWorkRequest`. That 5 minutes(called the flex time) is telling work manager  that you are allowed to run even in 10-15 minute interval. Which can it can run either in 10,11,12,13,14 minute intervals. It is the choice of workmanager. You can decrease it to force work manager to run it closer and closer to the interval time(here, 15 minutes). **You can't set interval time less than 15 minutes**
4. Adding tag (like `.addTag("RoutineNotificationWorkerTag")`) is important to identify workers in the workmanager later to cancel or delete them.
5. We finally `build()` to get the work request object.

Here, is how you work manager to enqueue this work request:
```
WorkManager.getInstance(this).enqueueUniquePeriodicWork("RoutineNotificationWorker",ExistingPeriodicWorkPolicy.CANCEL_AND_REENQUEUE,workerRequest)
```
1. `enqueueUniquePeriodicWork` on the work manager instance is used to enqueue your work request in this periodic work queue.
2. `RoutineNotificationWorker` is just a name we are giving to our worker.
3. `ExsitingPeriodicWorkPolicy` is used to tell work manager to what to do if it sees multiple workers with same name. Here we are asking it to cancel past ones and use this one.
4. In the last arg, we provide our work request object.
