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

## Tips
* You only provide declaration in header file of those functions or variables that you want to expose from this header file.  Meaning that anybody can use those functions that you have declared in header files by simply including them in their program.
* Declaration means just the signature of function or variable. Definition means defining what that function does and what value a variable has.
* You need to put `extern` keyword for variable declarations in header files to tell the preprocessor that it is defined in some .c file.
* Function declarations don't need `extern` keyword.
