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
    fun myFunc(var myScore: Int, val myName: String): Unit{
      println("Hi")
    }
    ```
    `Unit` means void. Here, this fucntion returns void or nothing. It takes in an int variable which can be changed by the function but it also takes in a String which cannot be changed by a function. So , `var` argument means that this arg can be changed by function but `val` argument means it is a compile time error if the funtion tries to modify this arg in its body.

10. Lambda functions are just anonymous functions without a name. They are represented as:
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
  
