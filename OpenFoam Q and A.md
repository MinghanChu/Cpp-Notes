[toc]

# Questions and answers addressing `c++` in OpenFoam

1. 

#####  Why no output with **new variable**? Is it a OF-related problem or c++ problem?

2. 

##### How to understand `addToRunTimeSelectionTable.H`, `defineTypeNameAndDebug(fieldAverage, 0); and addToRunTimeSelectionTable(functionObject, fieldAverage, dictionary);`? E.g.

`````c++
  32 #include "addToRunTimeSelectionTable.H"
   33  
   34 // * * * * * * * * * * * * * * Static Data Members * * * * * * * * * * * * * //
   35  
   36 namespace Foam
   37 {
   38 namespace functionObjects
   39 {
   40     defineTypeNameAndDebug(fieldAverage, 0);
   41     addToRunTimeSelectionTable(functionObject, fieldAverage, dictionary);
   42 }
   43 }
`````

My understanding: the variables are registered and can be outputted. Can this explain why fields outputted from base classes, e.g. `kOmegaSSTBase.C`, **can not be shown when postprocessing in paraview?**



3. 

##### scope operator `::`:

a. cross accessing classes under different namespaces

b. defining member functions for a created template class

c. accessing variable in global scope

Q: what about OpenFoam?

Ans: ==because inheritance in both OF and SU2 creates a **two or more distinct scopes**: subclass scope vs base class scope, including template when member functions inside a template must be defined outside of the template. In that case, `::` the scope operator needs to be used.== 

**Nov 25th 2020: I have just tested `::` operator in my own code and built my understanding regarding when/why to use it (see the explanation in detail): **

`````c++
class UQ
{
	protected:
		tensor Rij[5];
		vector e[5]; 
		tensor ev[5];
		double store[5][3];//the first [] is rwo and the second [] is col. **later might be changed to double** store = new double* [];
	protected:
		double c1c;

	public:
	//***********Constructor************//
		UQ ();//Note just a brief declaration of the constructor. DON'T define here! Even if just put "UQ()	   					 //{}" will hinder the actual defining of the constructor OUTSIDE OF THE UQ CLASS SCOPE (see 						//below). So leave it simple and brief (same principle for all member functions as well), then 
  				//define clearly (one by one in order for member functions) outside of the class, and you will 
  				//need "::" operator to access the functions declared in the first place, i.e. the UQ class. 
//*********Member functions************//
		void Eigenvalue ()
		{
		
				for (int i = 0; i < 5; i++)
				{
					e[i] = eigenValues(Rij[i]);
					store[i][0] = e[i].x() ;
					store[i][1] = e[i].y() ;
					store[i][2] = e[i].z() ;
					std::sort(store[i], store[i]+3, std::greater<double>());
					Info<< "i= "<<i<<" eigenvalues= " << e[i]<< endl;
				}
			
			for (int i = 0; i < 5; i++)
			{
				for (int j = 0; j < 3; j++)
				{
					std::cout<<" store= "<< store[i][j] <<std::endl;
					
				}
				
			}
		}

		void Eigenvector ()
		{
			for (int i = 0; i < 5; i++)
			{
				ev[i] = eigenVectors(Rij[i]);	
				Info<< " eigenvectors= " << ev[i] <<endl;
			}
		}
		
};

//UQ::UQ(){...} is clearly defined in detail outside of the UQ class where only brief declarations //should be made. 
/***********THIS IS THE POPULAR STYLE THAT PROGRAMMERS FOLLOW REGARDING C++, BECAUSE I HAVE OBSERVED //BOTH SU2 AND OF STICK WITH THIS STYLE****************/
UQ::UQ(){
		
			Rij[0] = {1, -3, 3, 3, -5, 3, 6, -6, 4};
		
	
			//read in Reynold stress field
			for ( int i = 1; i < 5; i++)
			{
				Rij[i] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
				Info<<"i= "<<i-1<<" Rij= "<<Rij[i-1]<<Rij[i-1].row(1)<<nl;
				if(i==4)
				{
					Info<<"i= "<<i<<" Rij= "<<Rij[i]<<Rij[i].row(1)<<nl;
				}
			}
}
`````



4. 

##### you can make a `vector` and `tensor` type variables array by specifying its size:

`````c++
tensor Rij[5];
vector e[5];
`````



5. 

##### In both `SU2` and `OF`, not clear instance can be found:

`````c++
//SU2 for example
#include "../../../include/numerics/turbulent/turb_sources.hpp"

CSourceBase_TurbSA::CSourceBase_TurbSA(unsigned short val_nDim,//is this line a instantiation?
                                       unsigned short val_nVar,
                                       const CConfig* config) :
  CNumerics(val_nDim, val_nVar, config),
  incompressible(config->GetKind_Regime() == INCOMPRESSIBLE),
  rotating_frame(config->GetRotating_Frame())
{
  /*--- Spalart-Allmaras closure constants ---*/

  cv1_3 = pow(7.1, 3.0);
  k2    = pow(0.41, 2.0);
  cb1   = 0.1355;
  cw2   = 0.3;
  ct3   = 1.2;
  ct4   = 0.5;
  cw3_6 = pow(2.0, 6.0);
  sigma = 2./3.;
  cb2   = 0.622;
  cb2_sigma = cb2/sigma;
  cw1 = cb1/k2+(1.0+cb2)/sigma;

  /*--- Setup the Jacobian pointer, we need to return su2double** but
   *    we know the Jacobian is 1x1 so we use this "trick" to avoid
   *    having to dynamically allocate. ---*/

  Jacobian_i = &Jacobian_Buffer;

}

CSourcePieceWise_TurbSA::CSourcePieceWise_TurbSA(unsigned short val_nDim,
                                                 unsigned short val_nVar,
                                                 const CConfig* config) :
                         CSourceBase_TurbSA(val_nDim, val_nVar, config) {

  transition = (config->GetKind_Trans_Model() == BC);
}

CNumerics::ResidualType<> CSourcePieceWise_TurbSA::ComputeResidual(const CConfig* config) {
`````



**Ans:** I think instances are not made here. It should be, in an efficient way (I do not know exactly how for now), made  inside the `int main()` or somehow through `int main()` indirectly, check it out later. Here in the above example defines the **member functions** declared in the corresponding `.h` file. When **member functions** are defined outside of the class scope, `::` operator should be used. That is why no **instantiating** can be found. Imagine there are **dozens of classes** need to be instantiated, there must be an efficient way of instantiating them, which could be hidden somewhere! **Remember** ONLY instantiate your classes inside `int main()`, no instantiations needed within **subclasses**, just call them as long as they are not marked as **private.**



6. 

##### What is the difference between declaring variables within **base class** and within **its member functions**?

*Ans:* 

+ declared inside the **base** you don't need to **actually use** it, only initializing of these variables is needed.

+ if declared inside member class, in addition to initializing the variables, some usage must be made, e.g. `for(int value:variable){std::cout<<value<<std::endl;}`

  

7. 

##### `wmake` to include `Numerics.C`

`wmake` cannot see the defined `Numerics.C` file if just has `Numerics.H` file included in its `Numerics.C` file. As the linker does not know the content written in the `Numerics.C` file even though its `Numeric.H` header file is included, because `wmake` does not tell the linker to do so. 

Ans: 

+ Add `.c` files to `files` under the **Make** directory. This tells the linker to link whatever files being included, i.e.  `#include` . `wmake` is a special rule for compiling and linking in OpenFoam. 
+ Only one of `.C` and `.H` files need to be added in the `files` under the **Make** directory. If both are added, an linker error will pop up: **multiple definitions of xxx function!!**. Also it seems to me there is no difference whether the `.H` or `.C` file is used in the `files` under the **Make**, so the reason why `.c` file is used is because, I guess, due to the programming style in OF. 



8. 

##### Why `using namespace Foam;` not working inside `Numerics.H`?

Ans: You must include, i.e. `#include` , a `.H` file that is already defined in `Foam namespace` so that the **Foam** namespace library is used. For example, `#include tensor.H`. In my final implementation, these `.H` files defined in **Foam namespace** are already included and should not be a problem. 

[How to use qualified and directive namespace](https://docs.microsoft.com/en-us/cpp/cpp/namespaces-cpp?view=msvc-160#:~:text=A%20namespace%20is%20a%20declarative,code%20base%20includes%20multiple%20libraries.)



9. 

##### **void function** 

void function will update whatever types of variables already declared and passed as parameters in the void function. So you can call **void** functions to update these variables, and then use the updated variables in later calculations. 

```c++
 static void EigenRecomposition(double** A_ij, double** Eig_Vec, double* Eig_Val, unsigned short n); //declared in Numerics.H
 
 	EigenRecomposition(m_newA_ij, m_New_Eig_Vec, m_Eig_Val, 3); //call the EigenRecomposition function in MyUQ.C then m_newA_ij will be updated and returned according to its declared type. You can use the updated m_newA_ij right after the call, e.g.
 	
 		for (x = 0; x < 3; x++)
	{
		for (y = 0; y < 3; y++)
		{
			Info<<"m_newA_ij"<<"["<< x <<"]"<<"["<<y<<"]= "<< m_newA_ij[x][y]<<endl; 
		}
	}
```



10. 

##### what is the difference between `symmTensorField` and `symmTensor` in OF?

Ans: it seems to me that any types with `Field` suffix must be used in turbulence models, e.g. I am not allowed to use `symmTensorField` in `MyUQ.C` . However mathematically, both of them refer to symmetric tensors and does not generate type confliction. `symmTensorField`  must contain a loop over all cells, and inside each cell a **symmTensor ** variable is stored, therefore any variable defined as **symmTensor** can be stored into the variable defined as **symmTensorField**, which is a collection of **symmTensor** variables. Similarly,**TensorField** is a collection **tensor** variables.



**Instantiating a type in OF:**

`symmTensor Rij_OF(1, 2, 3, 4, 5, 6);`  which is equivalent to `symmTensor Rij_OF = {1, 2, 3, 4, 5, 6};`or if you want to declare `Rij_OF` in a `.H` file, then in the `.H` file *do* `symmTensor Rij_OF;`, and in its corresponding `.C` file *do* `Rij_OF = {1, 2, 3, 4, 5, 6};`. Note if you try to define like this: `Rij_OF(1, 2, 3, 4, 5, 6)` in the `.C` file, an error will be generated: `no call to the Rij_OF...`. This has something to do with **instantiation** in `c++`. Here in this particular example, `symmTensor` is a class **intrinsic to** OF. Therefore `symmTensor Rij_OF;` is to create an **instance** of the **symmTensor class**, and **Rij_OF** is the created **object/instance** which can be used to access any defined member functions of the **symmTensor class.** An **object** can be passed to any functions as a parameter. 



11. 

##### **How to convert `field` to `Matrix`**

In **OpenFoam**, you can convert `fields` to the corresponding `Matrix` by using the `fvc::div` function or similar forms **provided** by OpenFoam.

**Important observations of c++ in terms of OpenFoam**

**Jan 5th 2021:** There is a complex inheritance existing in OpenFoam, and really need further deep studies. But now some conclusions can be drawn here from testing and investigation:

+ there are bunch of already written functions that are dedicated to **returning** flow fields, and the **parent (or subclass)** class which contains most of those functions that are written in `pure virtual, e.g. =0` is in located in `turbulenceModel.H`. The actual definitions are defined in its subclasses, 
+ `fvm::div(phi)`



12. 

##### Jan 6th 2021 - Understanding iterating process in OpenFoam

From testing and investigation, inside the `main()` the non-trivial user-defined field-type variables, e.g. `turbulence->R()`,  will be passed to the `UEqn.H` file in the **2nd iteration**. In other words, the results calculated at the **new time step** will be used.  This is because `R()` will only be calculated when `correct()` is called, i.e.

```c++

    Info<< "\nStarting time loop\n" << endl;

    while (simple.loop())
    {
        Info<< "Time = " << runTime.timeName() << nl << endl;	

        // --- Pressure-velocity SIMPLE corrector
        {
            #include "UEqn.H" //R() inside UEqn.H retains the non-trivial results at the new time step
            #include "pEqn.H"
        }


        laminarTransport.correct();
        turbulence->correct();//after this line R() is calculated

        Info<<"R()_outer in UQsimpleFOam= "<<turbulence->R()<<endl;//as R() is calculated from the line above, here R() retains 
      																													//the results at the current time step
        runTime.write();

        runTime.printExecutionTime(Info);

    }
```





13. 

##### Jan 7th 2021 - different ways to define fields

* when defining a new field in a `.C` file, e.g. `volScalarField CDkOmega(.....);` the memory of this variable will be freed up when reaching `}`. Therefore, declare this field variable in `.H` file, and output this field via `IOobject` if longer life time is required. In short, for a variable having the life time cross the whole compilation, declare it in `.H` file and output this geometric field vis `IOobject`.  Then you can manipulate this variable by `turbulence->f()` in the `main()` in the upper hierarchy, note `f()` is already defined function by OpenFoam (not user defined). 

##### Jan 12th 2021 - Struggling and findings

+ there exists differences between variables defined with **constructor in OpenFoam** and the ones defined as **member functions** marked as "`new`" in OpenFoam. The fact is newly created constructor variables are no longer used for being manipulated via `fvc::div()`, but `new` variables under created member functions can. Not actually know why and have to study more later.

##### Jan 13th 2021 - How to access created functions in `UQkOmegaSST` from `main()` (Summary after talking to Weicheng)

+ You must put the member function you want to access under `public`
+ Go to `UEqun.H` and add `#include "UQkOmegaSST.H"`

+ Add **relative path name** of the `Include` directory, where you made your changes, e.g. UQsimpleFoam, to `/root/OpenFOAM/-v1812/applications/solvers/incompressible/UQsimpleFoam/Make/options`

  (Note I kept the old ones, i.e. prefixed with (LIB_SRC) which points to the original source files under `src` folder)

  **Why is `main()` not seeing members in `UQsimpleFoam`, although the turbulence model written in `UQsimpleFoam` has been successfully compiled? **

  

  Ans: It should notice that compilation for **solvers**, i.e. `UQsimpleFoam` using `wmake` is distinct from the compilation for **turbulence models**, i.e. `UQsimpleFoam` using `./Allwmake`. My understanding: as the executable file of `UQsimpleFoam` , i.e. `EXE = $(FOAM_USER_APPBIN)/UQsimpleFoam` is also stored under `EXE = $(FOAM_USER_APPBIN` directory in which the executable files relating to my **UQ turbulence model** is also stored. From the **UQ turbulence model** perspective, it can see **UQsimpleFoam** solver as this is specified in `turbulenceProperties` . However from **UQsimpleFoam** perspective, it cannot see the **UQkOmegaSST** model, unless we **add the relative path name of the `Include`** directory to `/root/OpenFOAM/-v1812/applications/solvers/incompressible/UQsimpleFoam/Make/options`. Then we can call the member functions defined in `UQsimpleFoam`. 

  

+ OpenFoam has deliberately isolated **solvers** from **turbulence models**, I guess, to prevent compilation errors and save time. Also this is efficient for people who only need to concentrate on changing one of them. 