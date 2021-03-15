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

==The following piece of code shows the hierarchy that will be needed for casting, like what weicheng did in the UQsimpleFoam.H== 

`Foam::RASModels::UQkOmegaSST<Foam::IncompressibleTurbulenceModel<Foam::transportModel> >:`



The following block of code has been modified by Weicheng, and I have further changed it, see comments below

```c++
USER_SRC = /root/OpenFOAM/-v1812/src

EXE_INC = \
    -I$(USER_SRC)/TurbulenceModels/turbulenceModels/lnInclude \ //Only this line stays
    -I$(LIB_SRC)/TurbulenceModels/incompressible/lnInclude \
    -I$(USER_SRC)/transportModels \ //I have deleted this line which was originally added by Weicheng, and program compiled fine
    -I$(LIB_SRC)/transportModels \  //Also there is no transportModels directory under USER_SRC
    -I$(LIB_SRC)/transportModels/incompressible/singlePhaseTransportModel \
    -I$(LIB_SRC)/finiteVolume/lnInclude \
    -I$(LIB_SRC)/meshTools/lnInclude \
    -I$(LIB_SRC)/sampling/lnInclude


EXE_LIBS = \
    -lturbulenceModels \
    -lincompressibleTurbulenceModels \
    -lincompressibleTransportModels \
    -lfiniteVolume \
    -lmeshTools \
    -lfvOptions \
    -lsampling \
    -latmosphericModels
```



==**Very important point that I have found after numerous tests!!!!!!:**==

**Once you have ==added/replaced== the directory of the include folder for any turbulence models, e.g. UQkOmegaSST, to the `options` of UQsimpleFoam (see below for detailed info), every time you made changes inside `UQkOmegaSST.C/H` and `./Allwmake`, then you must `wmake` for the UQsimpleFoam to update. Otherwise, you may get error messages such as: `segmentation fault` or more likely `index 0 out of range` .**

+ You must put the member function you want to access under `public`
+ Go to `UEqun.H` and add `#include "UQkOmegaSST.H"`(This was later proven to be wrong and I changed this by putting it outside of `main()`)

+ Add **relative path name** of the `Include` directory, where you made your changes, e.g. UQsimpleFoam, to `/root/OpenFOAM/-v1812/applications/solvers/incompressible/UQsimpleFoam/Make/options`

  (Note I kept the old ones, i.e. prefixed with (LIB_SRC) which points to the original source files under `src` folder)

  **Why is `main()` not seeing members in `UQsimpleFoam`, although the turbulence model written in `UQsimpleFoam` has been successfully compiled? **

  

  Ans: It should notice that compilation for **solvers**, i.e. `UQsimpleFoam` using `wmake` is distinct from the compilation for **turbulence models**, i.e. `UQsimpleFoam` using `./Allwmake`. My understanding: as the executable file of `UQsimpleFoam` , i.e. `EXE = $(FOAM_USER_APPBIN)/UQsimpleFoam` is also stored under `EXE = $(FOAM_USER_APPBIN` directory in which the executable files relating to my **UQ turbulence model** is also stored. From the **UQ turbulence model** perspective, it can see **UQsimpleFoam** solver as this is specified in `turbulenceProperties` . However from **UQsimpleFoam** perspective, it cannot see the **UQkOmegaSST** model, unless we **add the relative path name of the `Include`** directory to `/root/OpenFOAM/-v1812/applications/solvers/incompressible/UQsimpleFoam/Make/options`. Then we can call the member functions defined in `UQsimpleFoam`. **In short**, those included `Include` folders for **any solvers** point to the original files under `src`, **NOT** in where you have created your own model, therefore need to specify clearly.

  

+ OpenFoam has deliberately isolated **solvers** from **turbulence models**, I guess, to prevent compilation errors and save time. Also this is efficient for people who only need to concentrate on changing one of them. 
+ A weird thing has happened later today, when I was recompiling `wmake` the `UQsimpleFoam`, an error message: `error: "namespace definition is not allowed here"`, and this issue was resolved by putting `#include "UQkOmegaSST.H"` outside the `main()`. Why this is weird is it did compile fine in the morning however did not work later afternoon. What I have done was just to changing the content in `UQkOmegaSST.C/H`, and change the names of them by `mv` and `cp` so as to test how the backup version comes out to be. Would this action affect the process of compilation? 



14. 

##### Jan 15th 2021 - pass by reference (`&`) with `const` , i.e. `const type& var_ref`  vs copy

**Very important study!**, see the code and comments below:

```c++
template<class BasicTurbulenceModel>
tmp<volSymmTensorField> UQkOmegaSST<BasicTurbulenceModel>::Perturb_UiUj
(
    const volSymmTensorField& Rij,
    const volSymmTensorField& Bij
) 
{
  
    tmp<volSymmTensorField> tPerturb_UiUj
    (
        new volSymmTensorField
        (
            IOobject
            (
                IOobject::groupName("Perturb_UiUj", this->alphaRhoPhi_.group()),
                this->runTime_.timeName(),
                this->mesh_
            ),
            this->mesh_,
            dimensionSet(0, 2, -2, 0, 0, 0, 0)
        )
    );

    volSymmTensorField& Perturb_UiUj = tPerturb_UiUj.ref();
    
    forAll(Perturb_UiUj, celli)
    {
      /* The difference between _Rij _Bij and _k and _nut is: 
           _Rij and _Bij are references that have lifetime only wihtin the forAll scope. Therefore they are
           not restricted to "const" method of Perturb_UiUj. In other words, they can either be

           1. symmTensor _Rij = Rij[celli] copy to and prone to be changed
           2. symmTensor& _Rij = Rij[celli] reference to and prone to be changed 
           3. const symmTensor _Rij = Rij[celli] copy to but fixed 
           4. const symmTensor& _Rij = Rij[celli] reference to but fixed
           and the same applies to _Bij

           On the other hand, _k and _nut are references to the members of kOmegaSSTBase class, and the "const"
           method of Perturb_UiUj must ensure these two references cannot be modified, therefore they must retain
           the following two forms

           2. const scalar _k = k_[celli] copy to and fixed
           1. const scalar& _k = k_[celli] reference to and fixed
           and the same applies to _nut 
           
           Again, Rij[celli] or k_[cellli] are deferencing and actual variable, therefore can either be copied or referenced,
           specifically var_store = Rij[celli] or var_store& = Rij[celli] */
        

        const symmTensor& _Rij = Rij[celli]; //const is not necessary in terms of programming perspective
        const symmTensor& _Bij = Bij[celli]; //const is not necessary in terms of programming perspective

        const scalar& _k = this->k_[celli]; //const is necessary in terms of programming perspective
        const scalar& _nut = this->nut_[celli]; //const is necessary in terms of programming perspective

      /* Note both Rij and Bij are const references to their corresponding variables 
      	 if they are supposed to be used DIRECTLY in below, I marked them using "". 
         the reference type must also be marked "const", i.e. const symmTensor& _Rij in the MyUQ.C
         On the other hand, if dereferencing them and assign them to variables _Rij and _Bij (
         means copying), you can either do with or without "const", like so: const symmTensor& _Rij or symmTensor& _Rij
         in the MyUQ.C/H. The reason for that is an actual variable can be referenced by "const" (promise not to change it) 
         or without "const" (no promise made to Not change it). 
         
         In short, const reference& must correspond to 
         const reference&. In contrary, for actual variables just it does not matter and it is up to the user to 
         decide if "const" reference should be used. */

        symmTensor Bij_OF;
        symmTensor UiUj_OF;
        symmTensor Sij_OF;
        
        double** newBij 	= new double* [3];
        double** newuiuj 	= new double* [3];
        double** newSij		= new double* [3];
        
        for (int j = 0; j < 3; j++)
        {
            newBij[j] 	= new double [3];
            newuiuj[j]	= new double [3];
            newSij[j]   = new double [3];
        } 
        
    //
        UQ uq(this->k_[celli], this->nut_[celli], "_Rij", celli);
        
        uq.EigenSpace(Bij_OF, UiUj_OF, Sij_OF, //directly used by OF
                      newBij, newuiuj, newSij, //in double format passed to OF
                      "_Bij", celli);//passed to MyUQ.C
        

        Perturb_UiUj[celli] = UiUj_OF;
        

        
        for (int j = 0; j < 3; j++)
        {
            delete [] newBij[j];
            delete [] newuiuj[j];
        }
        delete [] newBij;
        delete [] newuiuj;
    }
    
    return tPerturb_UiUj;
}
```



15. 

##### Jan 15th 2021 - OpenFoam styles for dealing with `xxxFields` type variables and analyze how to store and output them

+ `tmp<>`  is a wrapper class that allows **large local objects** to be returned from functions/methods **without being copied**. It allows a program to **quickly clear a large object out of memory**, which can be used to reduce the **peak memory**. **OpenFoam uses `tmp<>` everywhere, however `tmp<>` is NOT needed at all, it is chosen to just reduce the effort and increases the efficiency of your program.** In other words, `tmp<>` is a choice but not a must.

+ `tmp<>` ensures a large dataset (e.g. 10 million element geometric tensor field) to **survive the change in scope**. Because if dealing with large objects, passing by reference won't work because the data will still be going out of scope. (This might be a cause of why I was getting extreme values ). You can delete a large object **in the middle of an algorithm** to reduce **peak memory**. 

+ **Important!!! -> How to use it?** To return a large object using `tmp`, simply enclose the return type with `tmp<>`.

  ```c++
  tmp<largeClass> myFunc() 
      {
          tmp<largeClass> tReturnMe(new largeClass);
          largeClass& returnMe = tReturnMe.ref();//use ref() to unwrap the object
          // Somehow fill the data in returnMe
          return tReturnMe;
      }
  
  tReturnMe.clear(); //use the object as usual when it comes time to throw away the data use object.clear()
  									 //I have realized that .clear() in today's version OpenFoam should probably be written into somewhere else 
  										//and .clear() will be accessed when program reaches "}"
  ```

  

16. 

##### Jan 17th 2021 - Important findings！！！`When to use var_() and var_` for member variables

+ after several hours investigation: `var_()` is dedicated to `volScalarField::internal`, e.g.

  ```c++
  template<class BasicTurbulenceModel>
     77 tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::Fthetat // internal
     78 (
     79     const volScalarField::Internal& Us, //internal
     80     const volScalarField::Internal& Omega,//internal
     81     const volScalarField::Internal& nu //internal
     82 ) const
     83 {
     84     const volScalarField::Internal& omega = this->omega_(); //internal
     85     const volScalarField::Internal& y = this->y_();//internal
     86  
     87     dimensionedScalar deltaMin("deltaMin", dimLength, SMALL);
     88     volScalarField::Internal delta //internal
     89     (
     90         max(375*Omega*nu*ReThetat_()*y/sqr(Us), deltaMin)//Rethetat_() corresponds to internal
     91     );
     92  
     93     const volScalarField::Internal ReOmega(sqr(y)*omega/nu); //internal
     94     const volScalarField::Internal Fwake(exp(-sqr(ReOmega/1e5)));//internal
     95  
     96     return tmp<volScalarField::Internal>//internal
     97     (
     98         new volScalarField::Internal// internal
     99         (
    100             IOobject::groupName("Fthetat", this->alphaRhoPhi_.group()),
    101             min
    102             (
    103                 max
    104                 (
    105                     Fwake*exp(-pow4((y/delta))),
    106                     (1 - sqr((gammaInt_() - 1.0/ce2_)/(1 - 1.0/ce2_)))//gammaInt_() corresponds to internal
    107                 ),
    108                 scalar(1)
    109             )
    110         )
    111     );
    112 }
  ```

+ Otherwise just use `var_`, e.g.

  ```c++
  template<class BasicEddyViscosityModel>
     41 tmp<volScalarField> kOmegaSSTBase<BasicEddyViscosityModel>::F1
     42 (
     43     const volScalarField& CDkOmega
     44 ) const
     45 {
     46     tmp<volScalarField> CDkOmegaPlus = max
     47     (
     48         CDkOmega,
     49         dimensionedScalar("1.0e-10", dimless/sqr(dimTime), 1.0e-10)
     50     );
     51  
     52     tmp<volScalarField> arg1 = min
     53     (
     54         min
     55         (
     56             max
     57             (
     58                 (scalar(1)/betaStar_)*sqrt(k_)/(omega_*y_),//Just k_, y_, omega_
     59                 scalar(500)*(this->mu()/this->rho_)/(sqr(y_)*omega_)//note mu() is pure virtual function
     60             ),
     61             (4*alphaOmega2_)*k_/(CDkOmegaPlus*sqr(y_))
     62         ),
     63         scalar(10)
     64     );
     65  
     66     return tanh(pow4(arg1));
     67 }
  ```



17. 

##### Jan 17th 2021 - Assign values to a variable: `type var(function())` vs `type var = function(arg1, ..)`

```c++
//In kOmegaSSTLM.C

  543     const volScalarField::Internal Fthetat(this->Fthetat(Us, Omega, nu)); //Fthetat is a newly created variable and is 		                        																						//initialized by calling Fthetat(Us, Omega, nu)  which is the                                                                     //member function defined in kOmegaSSTLM.C
  544  
  545     {
  546         const volScalarField::Internal t(500*nu/sqr(Us));
  547         const volScalarField::Internal Pthetat
  548         (
  549             alpha()*rho()*(cThetat_/t)*(1 - Fthetat)
  550         );
  551  
  552       ...
  568     }
```

+ Question: why ` GbyNu0 = GbyNu(GbyNu0, F23(), S2());` in the `kOmegaSSTBase.C` does not directly use `F23` and `S2`  but `F23()` and `S2` instead. Do not deliberately spend time studying this, try to build understanding along the way of my research. 



18. 

##### Jan 18th 2021 - The way/style of writing member functions in OpenFoam, and how are those member functions used

**In** `kOmegaSSTBase.C`

| Protected member functions                                   |                         Where called                         |
| ------------------------------------------------------------ | :----------------------------------------------------------: |
| `F1()`                                                       |     In `correct()`, `F1()` is used to calculate  `Susp`      |
| `F2()`                                                       |            In `F23()` (protected member function)            |
| `F23()`                                                      |            In `correct()` for initializing `F23`             |
| `Pk(const volScalarField::Internal& G) const`                |                        In `correct()`                        |
| `epsilonByk(const volScalarField& F1, const volTensorField& gradU)` |                        In `correct()`                        |
| `GbyNu(const volScalarField::Internal& GbyNu0, const volScalarField::Internal& F2, const volScalarField::Internal& S2) const` |                        In `correct()`                        |
| **In summary**                                               | **Member functions can be called in either `correct()` or other member functions** |
| **`This`** pointer **ONLY** deals with **members**, i.e. **class member variables or functions** |                                                              |



| Special member functions                      |                                                              |
| --------------------------------------------- | :----------------------------------------------------------: |
| `correct()`                                   | `correct()` function not only appears in `kOmegaSSTBase` but in other related turbulence models, e.g. `kepsilon` and `kOmegaSSTLM`. This `correct()` function is the **key** to call all member functions that will **return** (not `void()`) values back to the `correct()` function to use. `correct()` will finally call `correctNut(S2)` so that to correct **eddy viscosity** and to trigger another iteration inside the `simpleFoam`. |
| `correctNut(S2)` and`correctNut()`            | `void` functions accompanied with `correct()` (need more study) |
| `Pk(const volScalarField::Internal& G) const` | `this->k_()` and `this->omega_()`  instead of `k_()` and `omega_()` are used. This may be related to the fact that `Pk` has been repeatedly overridden through `subclasses`, e.g. `kOmegaSSTLM`. |



19. 

##### Jan 19th 2021 - How to add $\Delta R_{i j}$ to `simpleFoam`?

| Studies                                                      |                         Explanation                          |
| ------------------------------------------------------------ | :----------------------------------------------------------: |
| override `divDevRhoReff(volVectorField& U) const` in `UQkOmegaSST.C` |     return the **source term** for the momentum equation     |
| overriding in `UQkOmegaSST.C` -> 1. call the overridden function in `UQkOmegaSST.C` (original definition of function `divDevRhoReff` in `linearViscousStress.C` is skipped); therefore 2. new member variables, e.g. `k_`and`U_`, that are used to calculate `UiUj_` and `Bij_`  will be assigned using the values calculated in the last time step. In other words: `k_` at step 1 will use the value of `k` at step 0, and `k_`at step 2 will be based on the `k` in step 1, etc. | As the result,  perturbation will start from the **1st** step |

**Important to note:** **must define the full definition of baseline Reynolds stress and its anisotropy form** inside the **overridden**`divDevRhoReff(volVectorField& U) const` function which will then be passed to the `Perturb_UiUj(Rij, Bij)` to get the **perturbed Reynolds stress**. If directly call any member functions of `UQkOmegaSST` model  inside the **overridden function** without defining the **Reynolds stress and anisotropy** inside the function (==correct() function will be called after the overridden`divDevRhoReff`function is called==), then no data (extreme values) will be passed to the `Perturb_UiUj` function and program crashes. **This result was obtained after numerous tests, so pay attention to it!!!!!** 



20. 

##### Jan 23rd 2021 - Resources that might be useful to correctly add $\Delta R_{i j}$

+ add a new `virtal tmp<fvVectoMatrix> divDevRhoReff()` function in `IncompressibleTurbulenceModel.C/H`
+ add the definition of the `divDevRhoReff()` function in `linearViscousStress.H` file, which for my case is the perturbed Reynolds stress
+ In `createFields.H` file, add needed fields (e.g. parameters passed by reference to the `divDevRhoReff()` function), and initiate these fields by creating a new `.H` file. This might solve my problem of repeatedly crashing. 

All detailed information can be found at: [rans-uncertainty](https://github.com/cics-nd/rans-uncertainty/blob/master/sdd-rans/TurbulenceModels/turbulenceModels/linearViscousStress/linearViscousStress.C)



21. 

##### Jan 24th 2021 - Struggling findings regarding `wmake`!!

In order to let the solver `UQsimpleFoam` see `MyUQ.C/H` located under the `UQkOmegaSST` directory, the following actions must be done:

```c++
USER_SRC = /root/OpenFOAM/-v1812/src

EXE_INC = \
    -I$(USER_SRC)/TurbulenceModels/turbulenceModels/lnInclude \ //MyUQ class and UQkOmegaSST can be seen 
    -I$(USER_SRC)/TurbulenceModels/incompressible/lnInclude \ //programm can see changes in linearviscousstress class. I have 																																//found it is easy to put MyUQ.C/H and Numerics.C/H under the turbulenceModels directory from which the header files of MyUQ.H and Numerics.H will be updated and put into the lnclude folder in turbulenceModels directory. 
    -I$(LIB_SRC)/transportModels \
    -I$(LIB_SRC)/transportModels/incompressible/singlePhaseTransportModel \
    -I$(LIB_SRC)/finiteVolume/lnInclude \
    -I$(LIB_SRC)/meshTools/lnInclude \
    -I$(LIB_SRC)/sampling/lnInclude


EXE_LIBS = \
    -lcompressibleTurbulenceModels \ //resolve the "Undefined reference to vtable MyUQ.C/H"
    -lturbulenceModels \
    -lincompressibleTurbulenceModels \
    -lincompressibleTransportModels \
    -lfiniteVolume \
    -lmeshTools \
```



22. 

##### Jan 25th 2020 - Polymorphism regarding turbulence models in OpenFoam

`BasicTurbulenceModel`-> `RASModel` -> `eddyViscosity` ->`SpalartAllmaras`

​																								  ->`kEpsilon`

​																								  ->`kOmegaSSTBase`-> `kOmegaSST` -> `kOmegaSSTLM`

`BasicTurbulenceModel`-> `kOmegaSST`->`kOmegaSSTLM`

`BasicTurbulenceModel`-> `linearViscousStress`-> `eddyVicosity`

`BasicEddyViscosityModel`->`kOmegaSSTBase`

`turbulenceModel`->`incompressibleTurbulenceModel`

`turbulenceModel`->`fvMesh`

`TurbulenceModel`->`IncompressibleTurbulence`



23. 

##### Jan 25th 2020 - Add a new function in higher hierarchy classes, e.g. `linearViscousStress`

My understanding: `turbulence->` pointer only deals with several higher hierarchy classes, e.g. `incompressibleTurbulenceModel.H` and `turbulenceModel.H` where you can find  a group of **pure virtual functions, (virtual fun() = 0).** This ensures those functions where those pure virtual functions are implemented can be called in solvers, e.g. `simpleFoam`, through `turbulence->`. Certainly, you also need to ensure `-I` header files in the correct directory and `-l` libraries in the correct directory are provided in the `Make` file of the solver. This explains why an error (`No member function`) will be generated if I want to call a newly created function in one of those turbulence models, e.g. `UQkOmegaSST`.  This can be broken by `static_casting`:

```c++
+ fvc::div( 
          ( 
              static_cast<
                RASModels::UQkOmegaSST<
                    IncompressibleTurbulenceModel<transportModel>
                >
              &> (turbulence())
          ).delta_test()

        )
```





**How to create a new function in `linearViscousStress`**. 

after declaring and defining your function inside the `linearViscousStress.C/H`, you need to also declare it in its parent classes, which enables `turbulence->` in `UQsimpleFoam` solver.

In `IncompressibleTurbulenceModel.H`

```c++
     //- Return the source term for the momentum equation
        virtual tmp<fvVectorMatrix> divDevReff //this 
        (
           
            volSymmTensorField& Rij,
            volSymmTensorField& Bij,
            volVectorField& U
        ) const;       

//- Return the delta Rij contributing to the source term for the momentum equation using UQ
        virtual tmp<fvVectorMatrix> divDevRhoReff
        (
            //const volScalarField& rho,
            volSymmTensorField& Rij,
            volSymmTensorField& Bij,
            volVectorField& U
        ) const;
```

In `IncompressibleTurbulenceModel.C`

```c++
template<class TransportModel>
Foam::tmp<Foam::fvVectorMatrix>
Foam::IncompressibleTurbulenceModel<TransportModel>::divDevReff
(
       volSymmTensorField& Rij,
       volSymmTensorField& Bij,
       volVectorField& U
) const
{
    Info<< "Uh oh" <<'\n'<<endl;
    return divDevRhoReff(Rij, Bij, U);

}


template<class TransportModel>
Foam::tmp<Foam::fvVectorMatrix>
Foam::IncompressibleTurbulenceModel<TransportModel>::divDevRhoReff //divDevRhoReff is a newly created function therefore marked as "NotImplemented", return divDevReff(arg1, arg2,arg3) which is defined above, which then calls divDevRhoReff(arg1, arg2, arg3) which is defined in linearViscousStress.C. 
(
       volSymmTensorField& Rij,
       volSymmTensorField& Bij,
       volVectorField& U
) const
{
    NotImplemented;

    return divDevReff(Rij, Bij, U);
}
```

In `incompressibleTurbulenceModel.H`

```c++
   //- Return the delta Rij contributing to the source term for the momentum equation using UQ
        virtual tmp<fvVectorMatrix> divDevReff //add pure virtual function of devDevReff which is discussed above NotImplemented
        (
            //const volScalarField& rho,
            volSymmTensorField& Rij,
            volSymmTensorField& Bij,
            volVectorField& U
        ) const = 0; 
```



In `linearViscousStress.C`

```c++
template<class BasicTurbulenceModel>
Foam::linearViscousStress<BasicTurbulenceModel>::linearViscousStress
(
    const word& modelName,
    const alphaField& alpha,
    const rhoField& rho,
    const volVectorField& U,
    const surfaceScalarField& alphaRhoPhi,
    const surfaceScalarField& phi,
    const transportModel& transport,
    const word& propertiesName
)
:
    BasicTurbulenceModel
    (
        modelName,
        alpha,
        rho,
        U,
        alphaRhoPhi,
        phi,
        transport,
        propertiesName
    ),
        perturb_UiUj_
        (
            IOobject
            (
                IOobject::groupName("perturb_UiUj_", alphaRhoPhi.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,
                IOobject::AUTO_WRITE
            ),
            this->mesh_,
            //dimless
            dimensionSet(0, 2, -2, 0, 0, 0, 0)
        ),

        delta_UiUj_
        (
            IOobject
            (
                IOobject::groupName("delta_UiUj_", alphaRhoPhi.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,
                IOobject::AUTO_WRITE
            ),
            this->mesh_,
            //dimless
            dimensionSet(0, 2, -2, 0, 0, 0, 0)
        ),

        newDelta_UiUj_
        (
            IOobject
            (
                IOobject::groupName("newDeltaUiUj_", alphaRhoPhi.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,
                IOobject::AUTO_WRITE
            ),
            this->mesh_,
            //dimless
            dimensionSet(0, 2, -2, 0, 0, 0, 0)
        ),

        perturb_Sij_
        (
            IOobject
            (
                IOobject::groupName("perturb_Sij_", alphaRhoPhi.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,
                IOobject::AUTO_WRITE
            ),
            this->mesh_,
            //dimless
            dimensionSet(0, 0, -1, 0, 0, 0, 0)
        ),

        perturb_Bij_
        (
            IOobject
            (
                IOobject::groupName("perturb_Bij_", alphaRhoPhi.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,
                IOobject::AUTO_WRITE
            ),
            this->mesh_,
            //dimless
            dimensionSet(0, 0, 0, 0, 0, 0, 0)
        )
        
{}

//- Return the delta Rij contributing to the source term for the momentum equation using UQ
template<class BasicTurbulenceModel> 
Foam::tmp<Foam::fvVectorMatrix>
Foam::linearViscousStress<BasicTurbulenceModel>::divDevRhoReff
(
    //const volScalarField& rho,
    volSymmTensorField& UiUj,
    volSymmTensorField& Bij,
    volVectorField& U
) const
{

    Info<<"**** You are in linearViscousStress.C - number 2 ****"<<'\n'<<endl;

    //Info<<"linearViscousStress U= "<< U<<endl;

    label timeIndex = this->mesh_.time().timeIndex();
    label startTime = this->runTime_.startTimeIndex();
    Info<<"timeIndex= "<<timeIndex<<" startTime= "<<startTime<<endl;
    
    //Only do a forward iteration on the FIRST timestep
    if(timeIndex - 1 == startTime && startTime != 0){

        Info<<"0 timestep is NOT being used!"<<endl;
    }
    else if(timeIndex - 1 == startTime && startTime == 0){

        Info << "[WARNING] 0 timestep is being used!" << endl;
        return
        (
        - fvc::div((this->alpha_*this->rho_*this->nuEff())*dev2(T(fvc::grad(U))))//deviatoric Reynolds stress
        - fvm::laplacian(this->alpha_*this->rho_*this->nuEff(), U)//molecular diffusion for newtonian fluids, nuEff must be used
        //+ fvc::div(this->newDelta_UiUj_)  //Note the extra perturbation term is commentted out and hence it returns the original (baseline) form 
        //for the source term of momentum equation. This is due to the fact that complex eigenvalues will be got at 0 time step which 
        //causes computation errors. This piece of code is to initiate the program using the baseline form and peturbation involves starting at
        //the 2nd time step.
        );
    }
    else{

        Info<<"****Program is running !!!****"<<endl;
    }
    
    // for (Foam::label i=0; i<(list).size(); ++i) this is the shortcut for "forAll" macro
    // mesh_.C() works same as mesh_.V() and any one of member variables defined, e.g. perturb_Sij_,
    // as long as they retain the same size or number of cells. Note mesh_.C() represents the 
    // whole field which involves all cells. If only "mesh_" is used, ambiguous size as an error
    // will be produced. This implies "mesh" does not contain the infomation for number of elements.
    forAll(this->mesh_.C(), celli)
    {
        symmTensor Bij_OF;
        symmTensor UiUj_OF;
        symmTensor Sij_OF;
       
        const volScalarField k(this->k());
        const volScalarField nut(this->nut());

        const symmTensor& _Rij = UiUj[celli]; //const is not necessary in terms of programming perspective
        const symmTensor& _Bij = Bij[celli]; //const is not necessary in terms of programming perspective

        //const method of Perturb_UiUj ensures members of class not changed
        const scalar& _k = k[celli]; //const is necessary in terms of programming perspective (member of class kOmegaSSTBase)
        const scalar& _nut = nut[celli]; //const is necessary in terms of programming perspective (member of class kOmegaSSTBase)

        /*if (celli < 6)
        {
            Info<<"In linearViscousStress k= "<<_k<<endl;

        }*/

        UQ uq(_k, _nut, _Rij, celli);
        
        uq.EigenSpace(Bij_OF, UiUj_OF, Sij_OF, //directly used by OF
                      //newBij, newuiuj, newSij, //in double format passed to OF
                      _Bij, celli);//passed to MyUQ.C
        /* Already tested Sij_OF gives the perturbed strain rate (for the current time step). 
           Coming out of linearViscousStress.C gives perturbed U and P fields. If the original
           expression is used to calculate the production term of k and omega, i.e. the perturbed
           U* field at current time step will be used. Since the U* field is partly based on the 
           old field of nut and therefore cannot be used to compute the perturbed production term
           for k and omega. As the result, we use perturbed Sij instead to get perturbed production term
           of k and omega  */
        
        //if (celli < 6)
        //{
        //    Info<<"linearViscousStress Sij_OF= "<< Sij_OF<<endl;
        //}
        
        this->perturb_UiUj_[celli]      = UiUj_OF;
        this->perturb_Sij_[celli]       = Sij_OF;
        this->perturb_Bij_[celli]       = Bij_OF;
        this->delta_UiUj_[celli]        = UiUj_OF - _Rij;
        
    }
        /* For incompressible flow testing showed there is no difference if rho 
           and alpha are multiplied, i.e. delta_UiUj_ = this->alpha_*this->rho_*(this->delta_UiUj_)
           however Info<<this->alpha_; or Info<<this->rho_; does not work to print out these results
           , should figure out later. */
        
        this->newDelta_UiUj_ = this->alpha_*this->rho_*(this->delta_UiUj_);
        //Info<<"linearViscousStress perturb_Bij_"<<this->perturb_Bij_<<endl;
    
    Info<< nl;

    Info<< "**** In linearViscousStress right after forAll loop where flow quantites have perturbed ****"<<endl;
    return
    (
      - fvc::div((this->alpha_*this->rho_*this->nuEff())*dev2(T(fvc::grad(U))))//deviatoric Reynolds stress
      - fvm::laplacian(this->alpha_*this->rho_*this->nuEff(), U)//molecular diffusion for newtonian fluids, nuEff must be used
      + fvc::div(this->newDelta_UiUj_)  
    );
}
```



`linearViscousStress.H`

```c++
   /* Added by Minghan Jan 29th 2021 - declare new member variables */
        mutable volSymmTensorField perturb_UiUj_;
        mutable volSymmTensorField delta_UiUj_;
        mutable volSymmTensorField newDelta_UiUj_;
        mutable volSymmTensorField perturb_Sij_;
        mutable volSymmTensorField perturb_Bij_;
        
        
             /* Added by Minghan Chu Jan 25th 2021 - Return the delta Rij contributing to 
        the source term for the momentum equation using UQ */
        virtual tmp<fvVectorMatrix> divDevRhoReff
        (
            //const volScalarField& rho,
            volSymmTensorField& UiUj,
            volSymmTensorField& Bij,
            volVectorField& U
        ) const;
```









24. 

##### Jan 26th 2021 - Struggling findings!! Understanding `Make/files` and `Make/options` 

**`Make/files vs Make/options`:** **header files** of classes in the `turbulenceModels` directory will be included in all other classes (e.g. `incompressibleTurbulenceModel`)  under the`TurbulenceModels` directory by  ` -I../turbulenceModels/lnInclude`. Also, by including all header files through ` -I../turbulenceModels/lnInclude` does not mean you have to use them all while being compiled, it is dedicated to providing a pool of (more than enough) classes and ensure they actually exist when any of them is being used. Header files `.H` that include the names and functions of the class need to be included **at the beginning** of any piece of code using the class. Then the **top level class , e.g.`.c`** can resource any needed classes via these **included `.H`** files by **searching recursively**. Meanwhile, you also must ensure the any of these classes that are being searched have their functions **compiled into** designated`bin` directory, known as `.so` shared library. **In summary:** `Make/files` ensure actual classes included by `.H` files are compiled  into the designated `bin` folder, and `Make/options` ensure all needed `.H` files and correct libraries, known as `-I` and `-l` , are included.

**Need to ponder:** It should be noted that **compilation went fine if** `./Allwmake` under `TurbulenceModels` directory after a **new function** (e.g. `diDevRhoReff(Rij, Bij, U)`) is written into the `linearViscousStress class` that is located in the further hierarchy than those **RASModels**: e.g.`UQkOmegaSST` . On the other hand, if there is no regarding declarations addressing the newly added function defined in the `linearViscousStress class` included in the `IncompressibleTurbulenceModel class`, an **linking error** will be generated: `no matching function for call to ‘Foam::IncompressibleTurbulenceModel<Foam::transportModel>::divDevRhoReff(Foam::volSymmTensorField&, Foam::volSymmTensorField&, Foam::volVectorField&)’`. This means there exists a relationship between the `IncompressibleTurbulenceModel class` and the `linearViscousStress class`. **Summary:** `turbulence` pointer dereferences the functions in `IncompressibleTurbulenceModel` which is the parent class of the`linearViscousStress` class. 



25. 

##### Jan 28th 2021 and Feb 9th 2021 - UQkOmegaSST compilation logic(==Refer to notes Jan 29th 2021==)

`createFields`(reading baseline results)->`UEqn.H` and `RABCal.H`->`linearViscousStress.C`(perturb the baseline results and calculate the **perturbed momentum source term**)->back to `UEqn.H`(calculate perturbed **U** and **P** fields)->`laminarTransport.correct()`(need to study later)->`turbulence->correct()`( use **perturbed Sij** to get perturbed production term of **k** and **Omega** and eventually **perturbed k and omega** fields  )->**Next iteration**(will use perturbed fields of **k, Omega, U, nut, P** as the baseline results, i.e. work in the same way as **createFields.H**.)



**Feb 9th 2021**

+ In `UQsimpleFoam.H`, do this

  ```c++
  #include "fvCFD.H"
  #include "singlePhaseTransportModel.H"
  #include "turbulentTransportModel.H"
  #include "simpleControl.H"
  #include "fvOptions.H"
  //#include "UQkOmegaSST.H"
  //#include "MyUQ.H"
  // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
  
  int main(int argc, char *argv[])
  {    
      argList::addNote
      (
          "Steady-state solver for incompressible, turbulent flows."        
      );    
  
      #include "postProcess.H"
  
      #include "addCheckCaseOptions.H"
      #include "setRootCaseLists.H"
      #include "createTime.H"
      #include "createMesh.H"
      #include "createControl.H"
      #include "createFields.H"
      #include "initContinuityErrs.H"
  
    
      turbulence->validate();
      Info<< "Minghan is very handsome in root  directory <4" << nl <<endl;
      
      // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
  
      Info<< "\nStarting time loop\n" << endl;
  
      while (simple.loop())
      {
          Info<< "Time = " << runTime.timeName() << nl << endl;	
  
          // --- Pressure-velocity SIMPLE corrector
          {
              #include "RABCal.H" //Added by Minghan Chu Jan 25th 2021
              #include "UEqn.H"
              #include "pEqn.H"
          }
          Info<< nl;
          Info<< "**** In UQsimpleFoam right before calling laminarTransport.correct() - number 3 ****"<<endl;
          Info<< nl;
          laminarTransport.correct();
          Info<< "**** Right outside of laminarTransport.correct() ****"<<endl; 
          Info<< nl;
          Info<< "**** In UQsimpleFoam right before calling turbulence->correct() - number 4 ****"<<endl;
          turbulence->correct();
          Info<< nl;
          Info<< "**** Right outside of turbulence->correct() ****"<<endl; 
          Info<< nl;
      
  
          runTime.write();
  
          runTime.printExecutionTime(Info);
          
      }
  
  
      Info<< "End\n" << endl;
      return 0;
  }
  
  
  // ************************************************************************* //
  
  ```

  

+ In `UQsimpleFoam.H` , `RABCal.H` is added: initializing baseline results for **strain rate (Sij)**, **Reynolds stress tensor (UiUj)**,  **deviatoric anisotropy (Aij)**, and **normalized anisotropy tensor Bij**

  ```c++
  tmp<volTensorField> tgradU (fvc::grad(U));
  
  UGrad = tgradU();
  
  /* already tested strain rate (Sij) = dev(symm(gradiant(U))) = 
  1/2(dUi/dxj + dUj/dxi) - 1/3(duk/dxk) Note: must include "dev()" 
  in OpenFoam for strain rate calculation! Otherwise you will get 
  wrong results! */
  Sij = dev(symm(tgradU()));
  
  UiUj = turbulence->R();
       
  Aij = turbulence->R() - ((2.0/3.0)*I)*turbulence->k();
  
  //Info<<"Aij= "<<Aij<<" "<<"k= "<<turbulence->k()<<endl;
  
  dimensionedScalar smal = dimensionedScalar("smal", dimensionSet(0, 2, -2, 0, 0, 0, 0), SMALL); //SMALL = 1e-15
  
  tmp<volScalarField> tmp_k (turbulence->k());
  
  //Info<<"tmp_k= "<<tmp_k<<"max(tmp_k, smal)= "<<max(tmp_k, smal)<<endl; //have tested as long as tmp_k = 0, the small value will be used (1e-15).
                                                                          //Otherwise, max(tmp_k, smal) will give the same result as that of turbulence->k().
  
  Bij = Aij/(2 * max(tmp_k, smal));
  
  /******* Added by Minghan Chu Feb 9th 2021 (obsolete) ********/
  //Bij = Aij/(2 * turbulence->k()+ smal); //This is an obsolete line because smal, which is a small number, is added throughout the entire domain. 
                                           //Even if the number is very small.
  
  //Bij = Aij/(2 * turbulence->k()); //A mathematical error is generated when zero boundary condition is used, e.g. for T3A simulation on the surface of plate k=0. This results in
                                     //dividing by zero. Therefore max() is required to avoid this issue.
  /********************* End ***********************
  ```

+ In `UEqn.H`, modifications are made:

  ```c++
      // Momentum predictor
  
      MRF.correctBoundaryVelocity(U);
  
       //Info<<"R()_inner in UQsimpleFOam= "<<turbulence->R()<<endl;
       /* Jan 5th 2020 - Minghan Chu modified the momentum equation, i.e. fvc::div(turbulence->R()).
          Note that "turbulence->R()" is on "volSymmTensorField" type, and will be converted
          to "fvVectorMatrix" through "fvc::div()"  */
  
      Info<<"You are in UEqn.H - number 1"<<endl;
      //Info<<"UEqn k()= "<<turbulence->k()<<endl;
      //Info<<"UEqn Bij= "<<Bij<<endl;
      //Info<<"UEqn Sij= "<<Sij<<endl;
  
  
      tmp<fvVectorMatrix> tUEqn
      (
          fvm::div(phi, U)
        + MRF.DDt(U) //this is not unsteady term check the pimpleFoam to see what an unsteady term is
        //+ turbulence->divDevReff(U)
        + turbulence->divDevRhoReff(UiUj, Bij, U)
        // + fvc::div(turbulence->ReynoldsStress())
        /*+ fvc::div( 
            ( 
                static_cast<
                  RASModels::UQkOmegaSST<
                      IncompressibleTurbulenceModel<transportModel>
                  >
                &> (turbulence())
            ).delta_test(UiUj, Bij, U)
  
          )*/
       // + fvc::div(turbulence->R()) 
       //+ fvc::div(turbulence->devRhoReff())
       ==
          fvOptions(U)
      );
      fvVectorMatrix& UEqn = tUEqn.ref();
  
      UEqn.relax();
  
      fvOptions.constrain(UEqn);
  
      if (simple.momentumPredictor())
      {
          solve(UEqn == -fvc::grad(p));
  
          fvOptions.correct(U);
      }
  
  ```

  

26. 

##### Feb 8th 2021 - Why program crashed for T3A at Time = 0? And how to fix this problem?

+ **Recall**: To prevent the calculation error, such that "**complex eigenvalues detected for tensor**", **NO** perturbation is enforced at **Time = 0**. Instead the original form that calculates the source term of momentum equation should be used:

  ```c++
       return
          (
          - fvc::div((this->alpha_*this->rho_*this->nuEff())*dev2(T(fvc::grad(U))))//deviatoric Reynolds stress
          - fvm::laplacian(this->alpha_*this->rho_*this->nuEff(), U)//molecular diffusion for newtonian fluids, nuEff in use 
          );
  ```

+ This solution **works well** if the boundary condition for `k(turbulence kinetic energy)` at the bottom wall is **NOT** 0. Otherwise, like what T3A was set at time=0, program crashes in `RABCal.H` when processing  `Bij = Aij/(2 * turbulence->k())`. Therefore, this issue can be avoided by just changing the boundary condition to a non-zero value. But is this scientifically acceptable? Maybe it is better to skip the first time step. 



27. 

##### Feb 10th 2021 - Steps to implement my UQ framework in OpenFoam

1. Implement the source code in `linearViscousStress.C/H` , see 23.
2. Put `MyUQ` (contains `MyUQ.C/H` and `Numerics.C/H`) under the directory: `/root/OpenFOAM/-v1812/src/TurbulenceModels/turbulenceModels`

3. In `UQkOmegaSST.C` within `correct()`add: `volScalarField::Internal GbyNu0((tgradU() && dev(2*(this->perturb_Sij_))));` meanwhile get `volScalarField::Internal GbyNu0((tgradU() && dev(twoSymm(tgradU()))));`(original form) commented out.

4. Then compile `UQ` class into `libincompressibleTurbulenceModels.so` and `libturbulenceModels.so` by putting the following two lines in `Make/files` files of `/root/OpenFOAM/-v1812/src/TurbulenceModels/turbulenceModels/Make` and `/root/OpenFOAM/-v1812/src/TurbulenceModels/incompressible/Make`(see **c++/OF messages and fixing for more explanation**): 

   ```c++
   /root/OpenFOAM/-v1812/src/TurbulenceModels/turbulenceModels/MyUQ/MyUQ.C
   /root/OpenFOAM/-v1812/src/TurbulenceModels/turbulenceModels/MyUQ/Numerics.C
   ```

5. Now modify the `UQsimpleFoam` solver, see 21.

6. Also initializing certain flow variables, see 25.

**Recap: When to refer to 27? When you are setting up your new simulation case in a new version or unmodified OpenFoam, you should follow the steps above.**



28. 

##### Feb 10th 2021 - How to easily switch between baseline and perturbed results?

It is very easy to switch between the **Baseline kOmegaSST** and the **perturbed kOmegaSST** versions. However you will definitely forget how to follow such easy steps months or years from now, as a reminder I provide the detailed steps below:

1. check in `UQsimpleFoam` that `#include "RABCal.H"` is added.

2. check in `UEqn.H`, `turbulence->divDevRhoReff(UiUj, Bij, U)` is used.

3. You do not need to do anything in `linearViscousStress.C/H`, but you could check if the running program is accessing it by reading the `Info` added.

4. Note you do not need to change anything in `UQkOmegaSST.C/H` either, but you could also check that `volScalarField::Internal GbyNu0((tgradU() && dev(2*(this->perturb_Sij_)))); ` is used.

   + **case 1 - baseline results**

     to get baseline results, just ensure `uq_delta_b = 0.0`;

   + **case 2 - perturbed results**

     to get perturbed results, just assign bigger-than-zero values to `uq_delta_b`, e.g. 0.1.

   

   **Observations:**

   My guess got proof, and in short three points:

   + **`volScalarField::Internal GbyNu0((tgradU() && dev(twoSymm(tgradU())))); `** instead of `volScalarField::Internal GbyNu0((tgradU() && dev(2*(this->perturb_Sij_))));` is used to produce **baseline results**. This results in updated **U field** at current time step, and will in turn be used to calculate production of k, i.e. **pk**. 
   + The first bullet point will produce different results then if `volScalarField::Internal GbyNu0((tgradU() && dev(2*(this->perturb_Sij_))));`is used, which **follows steps 1-4.** Note the earlier time step the iteration is at the larger the difference should be noticed. 
   + Fortunately, the difference observed in bullet point 2 will diminish to an **extremely small** number, e.g. `<1 thousandths`. This indicates **steps 1-4 are acceptable** to produce **both perturbed and baseline results.** 

   

##### Feb 19th 2021 - Following Feb 10th 2021 in detail what is the difference using the two different methods to produce the baseline results?

refer to Notes on Feb 16th 2021



##### Feb 25th 2021 - what are`ksource()` and `fvOptions`?

defined in `kepsilon.C` which is the k per time interval interior of a cell. `fvOptions` are advanced manipulations to the source terms, e.g. adding sink terms with specified zones. Check out the details on official website.



##### Feb 25th 2021 - compare the difference between `<uiuj>` and `<Sij>`

refer to Notes on Feb 25th 2021



##### March 4th 2021 - OpenFoam units

For an incompressible flow, all the equations are divided with density. For example, wall shear stress in OpenFoam is in the units:

**Pa/r = N/m2/kg/m3 = kg*m/s2/m2/kg/m3 = m2/s2, where r is density**.



##### March 13th 2021 - Machine learning (Weicheng and Minghan)

```

Weicheng
还是那句话，有好的data当然会有帮助！但是机器学习本质是如何高效利用所有已知的东西，包括数据和一些理论（比如f=ma，牛顿三定律，PV=nRT），这包括两层意思：

1. 如何把所有已知的数据发挥到极致
2. 如何把所有已知的知识发挥到极致
   不同的方法在这两点上各有优势。bayesian相对于neural networks在2上更有优势而已。
   
   即使“没有”数据，也可以讨论如何获得数据；事实上很少有真正没有数据的情况。顶多是获得数据难，没有好数据较难而已。公式本身就可以变成数据，譬如通过monte carlo 方法。
   
   我们对数学的理解，往往从定义开始，似乎是自上而下。但是很多机器学习的问题，我们学习的过程是自下而上。
   
   Minghan
  我昨天想了一下，可能learning完后面implement是可以的。

我用的模型是trainsition（转捩）模型，他有一个condition是：如果检测的transition，开启transition处理模式，

而这个transition mechanisms只发生在那个我们用learning处理的zone里面。也就是说可以learning后把结果并入transtion condition，及检测到调用，检测不到（trasition没有发生）保持原状，
```



##### March 15th 2021 - C++ Polymorphism in OpenFoam

Using an real example to explain polymorphism in Openfoam:

In `UQsimpleFoam`, `turbulence->correct()` instantiates the selected class, e.g. `UQkOmegaSSTLM`. This means any **pure virtual functions** being used will be overridden if redefined in the`UQkOmegaSSTLM` class. In this particular example, `Pk(G)` is redefined in the `UQkOmegaSSTLM` class, `return gammaIntEff_*UQkOmegaSST<BasicTurbulenceModel>::Pk(G);`. And `UQkOmegaSST<BasicTurbulenceModel>::Pk(G);`  forces the linker to search where `Pk(G)` is defined by providing the "path" to it.  Note if `this->Pk(G)` is used, you will encounter an infinite loop! Because as mentioned earlier, `turbulence->correct()`  tells `this->` to point to the current instance, i.e. `UQkOmegaSSTLM`. 

In short, for any redefined **(pure) virtual functions** in the current instantiated class, `this->` always points to these functions in this instantiated class. For any **virtual functions** that are NOT redefined in the current instantiated class, `this->` will point to the right ones based on polymorphism, e.g. in its parent class. Note child class **is** parent class, which contains everything in the parent class and some additional stuff! In this example, `UQkOmegaSST<BasicTurbulenceModel>::Pk(G)` will call the `Pk(G)` defined in `kOmegaSSTBase` class, which is the parent class of both `UQkOmegaSST` and `UQkOmegaSSTLM`. 







##### C++/OF error messages and fixing

+ `undefined symbol`: usually you forgot to comment out declarations (in `.H` file) that you have commented out in its corresponding `.C` file. **The second possible error is:** forgot to include the newly added class into the **correct** `Make/files` . "**Correct**" refers to possible classes that would call the new `UQ class`. As mentioned earlier if you call a new class in `linearViscousStress` , e.g `MyUQ`, you will need to include the directory to `MyUQ.C` and `Numerics.C` to the `Make/files` of both `turbulenceModels` and `incompressible` directories. This ensures the `class UQ`  is compiled into these `Make/files` and therefore can be seen by them. 

  **Why do I need to compile `UQ` class into `libincompressibleTurbulenceModels.os` if `UQ class` is only called in `linearViscousStress` and already compiled into `libturbulenceModles.so`?** This is because **inheritance** requires `linearViscousStress class` to be used in numerous classes rather than just `UQkOmegaSST` that only belongs to `turbulenceModels` (compiled into `libturbulenceModels.os`). To let all other models that are inherited from the higher class of `IncompressibleTurbulenceModels` see `UQ class` included in the `linearViscousStress class`, we need to compile `UQ class` into `libincompressibleTurbulenceModels.os` by including the directory to `MyUQ.C and Numerics.C`  to the `Make/fiels` of `incompressible` directory. Go to explanation for `.H` and `.os` to build better understanding.

  

  *** Strikethrough is applied to the following content simply because it (my understanding) was not true! So far have updated my understanding:**

  1. After creating a new functionality (MyUQ.C) and get it included in an existing class by its header file . The full list of `.C` source files **must** be included in the `Make/files` file. 

  2. 1 is discussed following the bullet point above. 1 ensures that **binary executable library** of `UQ` class is added to the **shared object library** of `libturbulenceModels.so` and `libincompressibleTurbulenceModels.so`. 

  3. **Why does not the following method work? Or compile separately by just adding the library `-L$(FOAM_USER_LIBBIN)/MyUQ`** to the `Make/files` files of `incompressibleTurbulenceModels` and `turbulenceModels`, without doing 1?

     ==I am not 100% sure now, but from my tests and observations some conclusions might be drawn: `MyUQ` folder is put under the directory being compiled! Not in an already compiled state waiting to be called, see other `-l`libraries through dynamic linking, where all `.os`libraries are sitting under `opt`. Maybe try put it under `opt` directory later and test.==

  ~~**An alternative way to solve this problem:** to make your `UQ class` more agile and therefore more easily be used in other different classes, you can create a **a new folder named MyUQ under the turbulenceModels directory**, create its Make files with **"options" left empty** and "files" like below~~:

  ```
  MyUQ.C
  Numerics.C
  
  LIB = $(FOAM_USER_LIBBIN)/libMyUQ
  ```

  Maybe need to modify this in order to successfully compile MyUQ.so into a separately export LD_LIBRARY_PATH=/path/to/the/folder/that/stores/your/uq.so/:$LD_LIBRARY_PATH from weicheng Mar 11th 2021

  `cat /opt/OpenFOAM/setImage_v1812.sh 
  source /opt/OpenFOAM/OpenFOAM-v1812/etc/bashrc
  export LD_LIBRARY_PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/ParaView-5.6.0/lib/mesa:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/ParaView-5.6.0/lib/paraview-5.6/plugins:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/qt-5.9.0/lib:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=$WM_THIRD_PARTY_DIR/platforms/linux64/zlib-1.2.11/lib:$LD_LIBRARY_PATH
  export PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/qt-5.9.0/bin:$PATH
  export QT_PLUGIN_PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/qt-5.9.0/plugins`

  

  ~~Under the **turbulenceModels** directory ensures MyUQ header files go to the `lnclude` folder for **turbulenceModels**, and will in turn be included in `incompressibleTurbulenceModels` and `compressibleTurbulenceModels`. If you want to put MyUQ header files outside of **turbulenceModels** directory, you must specify its path to the header files in "options"~~.

  `$FOAM_USER_LIBBIN = /root/OpenFOAM/-v1812/platforms/linux64GccDPInt32Debug/lib`

+ `type& var(this->fun())` **No `&` ** which can be compared to `type& var1 = var2(this->fun())` which can have `&`

+ ==`wmakeLnInclude -u turbulenceModels`== can be used to solve `no matching call to function error`, which is a **linking error**. This command will update the **old linkages with the new ones!!** I have spent more than 6 hours today (**Jan 24th 2021**) to fix this error. 





