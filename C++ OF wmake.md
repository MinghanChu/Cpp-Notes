# C++ OF wmake

### Introduction

1. The file containing the class *definition* takes a `.C` extension. Among these `.C` files, some can be **compiled independently of other code**, e.g. the  `nc.C` file, into a binary executable library file known as **shared object library** with `.so` file extension.

2. For example, when compiling `NewApp.C` which is dependent of OpenFoam (shared object) library, `nc.H` , `nc.H` need not be recompiled, rather `NewApp.C` just calls `nc.so` at  runtime. This is known as **dynamic linking**. 

3. If a new user-defined library is created, wishes the features within this library to be available across a range of applications, **dynamic linking** at runtime can be easily done through `controdict` file.

4. when **wmake**, the compiler searches for the included header files in the following order

   * the `$WM_PROJECT_DIR/src/OpenFOAM/InInclude` directory

     this is why when you include a class (`.H file`) inside the `src/OpenFoam/InInclude` directory, e.g. `vector.H`, **wmake** will let compiler find this file.

   * a local `InInclude` directory, e.g. `NewApp/InInclude`

   * the local directory, i.e. `NewApp`

   * other directories specified explicityly in the Make/options file with the `-I` option.

5. Make/options for solvers **only**

   * other than the `OpenFOAM/InInclude` directory, **solvers** need to be linked to numerous models so that to resource them, e.g. `pisoFoam` resources the `incompressibleRANSModels`, `incompressibleLESModels`, and `incompressibleTransportModles` libraries,  `Make/options` contains the **full directory paths** to locate header files of :

     `````c++
     //Note "-I" option means to include header files and"\" to continue the EXE_INC across several lines
     EXE_INC = \
     	-I<directoryPath1> \
     	-I<directoryPath2> \
     	-I<directoryPath2> // no "\" after the final entry
     `````

     

   * also linking to **shared object library** through

     `````c++
     //Note "-L" means to link to the shared object library files in the following directory paths
     EXE_LIBS = \
     	-L<libraryPath1> \
     	-L<libraryPath2> \
       -L<libraryPath2> 
     `````



In summary, two main resources to include needed classes:

1. as long as you can find the `.H ` file in `src/OpenFoam/InInclude` directory, then just include the `.H` file in `NewApp.C` and **wmake**. The compiler will find it.
   * For example: if `tensor.H` is included in which `Tensor.H` is also included.  Note ==`.` means assign a variable as a parameter to a function: `T.inv() ` is equivalent to `inv(T)`==. Variable type classes `tensor`, `vector`, and `scalar` have nothing to do with the member functions in `tensor.H`, e.g. you `eigenVector()` and `eigenValue()` member functions do not belong to `tensor type` class, but defined in `tensor.H`.    
2. Solvers need `Make/options`, linking to`.H` files and **shared object libraries** through **dynamic linking** at runtime which can be found under the solver directory. For example, `simpleFoam`. 

​         