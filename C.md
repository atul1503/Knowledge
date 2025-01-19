## Compilation process
```
1. Preprocessor
2. Compiler
3. Assembler
4. Linker
```

* `Preprocessor` processes macros and `#include`s `#define`s and puts all the content of header files in your C file where you `#include`ed the header file. This generates `.i` file.
* `Compiler` compiles `.i` file to assembly language. This creates `.s` files.
* `Assembler` generates machine code from `.s` files. This creates `.o` files.
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

## Tips
* You only provide declaration in header file of those functions or variables that you want to expose from this header file.  Meaning that anybody can use those functions that you have declared in header files by simply including them in their program.
* Declaration means just the signature of function or variable. Definition means defining what that function does and what value a variable has.
* You need to put `extern` keyword for variable declarations in header files to tell the preprocessor that it is defined in some .c file.
* Function declarations don't need `extern` keyword.
