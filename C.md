## Compilation process
```
1. Preprocessor
2. Compiler
3. Assembler
4. Linker
```

* `Preprocessor` processes macros and `#include`s `#define`s and puts all the content of header files in your C file where you `#include`ed the header file. This generates `.i` file. Thie `.i` file is just a big version of your `.c` file. It includes all the external library type and function definitions.
* `Compiler` compiles `.i` file to assembly language. This creates `.s` files.
* `Assembler` generates machine code from `.s` files. This creates `.o` files also known as object files.
* `Linker` links all the `.o` files with each other and creates a single executable file which you can directly run. In Linux, you may also have to give execute permission to the executable.

* Command to execute:
```
gcc my_program.c my_other_file.c -o my_executable_name
```
This will generate an executable.

* To only generate object file without linking:
```
gcc -c my_program.c my_other_file.c
```
This will generate object files(.o) for all the .c files. Note that you can't use -o option since you can't name all object files with the same name. Only if you were using a single .c file you can put -o here.
To make it an executable use gcc to link these object files:
```
gcc my_program.o my_other_file.o -o my_executable_name
```
This generates an executable.

* You can also generate shared library(.so) object files also, which are like external libraries that you don't have in your executable. It is only loaded or referenced during runtime if those .so files are already present in the system. This is used for mainly standard libraries like `stdlib`. This is like having `DLL`s in windows. This is also like having standard library of python already present in the system where you want to run your python program. So you dont have to manually download those python files from internet and keep it in the same directory where your program is just so that your program can reference those python files to execute your program. They should be present in system seperately.
* You can also use static library(.a) for dependency management. They are exactly like external jars from java. Basically, .a file is an archive file containing other object files. This you can use to include external libraries.
* For using an external library, you need to have both its header files and its object files when you are compiling your code.
   * Header files are required only for the preprocessor to put the type definition of the external code into its output `.i` file. This `.i` file is used by the compiler to compile the code.
   * Object files contain the actual implementation of types and functions declared in header files.
* If you want to create your own C library, you just:
   * Generate object files.
   * Create a header file that includes the declarations of all functions and types that you want to expose from your code.
   * Distribute your header files and object files.
   * Ask the users to put the header files either in the current directory or the default directory where preprocessor looks for header files.
   * Ask the users to put the object files either in the current directory or the defualt directory where linker looks for object files. 
   Your library users will download the header files and object files and use them during compilation.
   *  If you have shared your object files as `.o` files, your users need to put those file path like this:
      ```
      gcc user_program.c my_lib_object_file1.o my_lib_object_file1.o
      ```
      This is obviously more impractical but it works.
   * In practical purpose, you will put all your object files in `.a` archive or as `.so` files. And then tell your linker to link it with current program:
     ```
      gcc user_program.c -L /path/to/.a/or/.so/library/parent/folder -lmyLib
     ```
  

## Tips
* Avoid using `goto`.
* Instead of doing `struct myStruct myVar;` everytime you can do
```
typedef struct myStruct { int x,int y } Point;
Point myVar;
```


## Noteworthy
* You only provide declaration in header file of those functions or variables that you want to expose from this header file.  Meaning that anybody can use those functions that you have declared in header files by simply including them in their program.
* Declaration means just the signature of function or variable and sometimes initialization also. Definition means defining what that function does and what value a variable has.
* You need to put `extern` keyword for variable declarations in header files to tell the preprocessor that it is defined in some .c file.
* Function declarations don't need `extern` keyword.
* Dereferencing means accessing the object pointed by pointer.
* Use `.` with structs or unions only when you have the actual instance of a struct or union. If you have pointer to the struct then use `->` because `pointer_name.val` will try to access `val` member of `pointer_name` but we want to access `val` of the instance of the structure not the pointer.
* `*ptrName` deferences the pointer. This means that we get the actual object which the `ptrName` is pointing to.
* `(*ptrName).val` is same as `ptrName->val` as we first dereferencing the pointer to get the actual object and then accessing the `val` member of the object.
* variable name can be only 31 characters long. Can start with letter or underscore but not number.
* We can do pointer arithmatic to refer to memory with relation to the address of pointer. `*(ptr+5)` will refer to 6 integer element stored in 5 places ahead of the memory location pointed by `ptr`. So `ptr+5` is basically calcualting the address of 5th integer from ptr's location. And the `*()` is dereferencing it to access the value stored in the address.
* To get the address of an obect use `&` which called `address of` operator.
* Instances of any type is called as objects of that type even though `C` is not an OOPs language.
* Any line starting with `#` is a code for preprocessor to be interpreted.
* Binary operator require 2 operands, Unary operators require 1 operand and ternary requires 3 operands.
* You declare a variable as `volatile` only if that variable's value may be changed by other programs or threads also. This keyword allows the compiler to generate machine code where they always do operations on later values. Meaning that reads are more frequent to get its exact values.
* `[]` and `*` change meaning when used in declaration statements than their usual meaning in expressions. For eg, `arr[]` in declarations , will mean that arr is an array and its size is put inside the `[]`. But in expressions `[]` helps to refer in pointer arithmetic and array element access. `*` in declarations is used to mark variables as pointer types but in expressions `*` is used to deference a pointer.
* `typedef myTypeName newTypeName` will allow you to use `newTypeName` like a predefined type in declarations. `myTypeName` can be struct,union,int,double etc.
* Use `sizeof` operator to get the size of an object or type.
* `goto` is used to jump to different labels in the same function. You can define labels like this:
```
goto myLabel;
printf("\nThis will be skipped");
myLabel:
{
  printf("inside myLabel");
}
```
`goto` is good for error handling and stuff. You can also use it as a better `break` since you can end multiple nested lopps with a `goto`.

* `struct`s and `union`s are used to define user defined types.
* `struct` or `union` declarations are same as other declaration statements:
`struct myStruct { int x, int y } myVar;` is similar to `int myVar`.
