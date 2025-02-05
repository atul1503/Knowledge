Notes:
1. Use Thread() from multithreading library or ThreadPoolExecutor() to do multiple io tasks at the same time. IO task means sending network requests, writing to file or writing to database.
2. For task other than IO tasks you can use Process() from multiprocessing library to execute those tasks paralllely. But remember that it will create sperate process and sperate memory for the processes. If you want to send messages or communicate you need to use Queue() also from this library.
3. For cron type of jobs use celery beat which allows you to runs jobs at specific times.
4. There is also django-crontab which can also do the shceudling of job but it runs on the same process where django server runs.
5. Python has GIL which means at a time for any cpu related task that means for any non io task, it will run only task at a time. If you want to avoid that use multiprocessing library but you don't get to share memory.
6. dir(obj) is used to see attrirbutes and methods of class or object.
7. isinstance(obj,classname) to check if the obj is instance of classname.
8. issubclass(childclass,parentclass) to chec if childclass is instance of parent class.
9. To create a package in python just create a folder with the package name and put __init__.py file in the folder. it will become a package.
10. __name__ is a variable that is used to access the name of the current module.
11. You can do
    ```
      if __name__=="__main__":
          main()
    ```
12. In python decorators are implemented as a function which has a nested function declaration that returns another function.
    ```
        def decorator_Annotation():
            def new_func(func):
                #Do somehting
                func()
                #Do something
            return new_func
        @decorator_Annotation
        def front()
    ```
    Now, this front will be replaced by new_func but will be called front only. This is the case for only decorators that don't accept args.
    For decorators that accept args, we declare the decorator with two functions nested inside it. The argument of the decorator function gets the argument to the decorator and the second level function gets the function reference and the innermost fucntion gets the arguments of the decorat-ed fucntion. Yeah i know its crap. Thank Guido for that.

13. `from concorrunt.futures import ThreadPoolExecutor` object is imported. Use this with `with` and this has a map() method which takes a fucntion to execute on seprate threads and an array of args to execute like `[(arg11,arg12),(arg21,arg22)]`. The object accepts a max_workers arg as integer to put that many threads but use only for io tasks. the `map()` returns a an array of results that each thread has executed. This is better than `Threads` from threading since they don't need to be managed as we are using with. 
14. Iterables implement __iter__ method but iterators implement __next__ and __iter__ method. next is to get the next element of the collection, iter is to get the iterator itself.
15. generator are same as iterators but except that they yield a value and does not return it. It uses a function instead of class to work its way. Uses less memory as yield values only when asked and doesnot maintain any collection in memory.
16. python has numberic types(int,float,complex,bool), None, sequence types(list, array,tuple,str, range, set,frozenset) and mapping types(dict).
