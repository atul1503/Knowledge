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
