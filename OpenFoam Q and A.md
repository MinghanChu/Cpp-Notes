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



==**Very important point that I have found after numerous testing!!!!!!:**==

**Once you have ==added/replaced== the directory of the include folder for any turbulence models, e.g. UQkOmegaSST, to the `options` of UQsimpleFoam (see below for detailed info), every time you made changes inside `UQkOmegaSST.C/H` and `./Allwmake`, then you must `wmake` for the UQsimpleFoam to update. Otherwise, you may get error messages such as: `segmentation fault` or more likely `index 0 out of range` .**

+ You must put the member function you want to access under `public`
+ Go to `UEqun.H` and add `#include "UQkOmegaSST.H"`

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







```c++
/*---------------------------------------------------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     |
    \\  /    A nd           | Copyright (C) 2016-2017 OpenFOAM Foundation
     \\/     M anipulation  |
-------------------------------------------------------------------------------
License
    This file is part of OpenFOAM.

    OpenFOAM is free software: you can redistribute it and/or modify it
    under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    OpenFOAM is distributed in the hope that it will be useful, but WITHOUT
    ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
    FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
    for more details.

    You should have received a copy of the GNU General Public License
    along with OpenFOAM.  If not, see <http://www.gnu.org/licenses/>.

\*---------------------------------------------------------------------------*/

#include "kOmegaSSTLM.H"

// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

namespace Foam
{
namespace RASModels
{

// * * * * * * * * * * * * Protected Member Functions  * * * * * * * * * * * //

template<class BasicTurbulenceModel>
tmp<volScalarField> kOmegaSSTLM<BasicTurbulenceModel>::F1
(
    const volScalarField& CDkOmega
) const
{
    const volScalarField Ry(this->y_*sqrt(this->k_)/this->nu());
    const volScalarField F3(exp(-pow(Ry/120.0, 8)));

    Info<< "Fuji is important" << endl;
    return max(kOmegaSST<BasicTurbulenceModel>::F1(CDkOmega), F3);
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::Pk
(
    const volScalarField::Internal& G
) const
{
    return gammaIntEff_*kOmegaSST<BasicTurbulenceModel>::Pk(G);
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::epsilonByk
(
    const volScalarField& F1,
    const volTensorField& gradU
) const
{
    return
        min(max(gammaIntEff_, scalar(0.1)), scalar(1))
       *kOmegaSST<BasicTurbulenceModel>::epsilonByk(F1, gradU);
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::Fthetat
(
    const volScalarField::Internal& Us,
    const volScalarField::Internal& Omega,
    const volScalarField::Internal& nu
) const
{
    const volScalarField::Internal& omega = this->omega_();
    const volScalarField::Internal& y = this->y_();

    dimensionedScalar deltaMin("deltaMin", dimLength, SMALL);
    volScalarField::Internal delta
    (
        max(375*Omega*nu*ReThetat_()*y/sqr(Us), deltaMin)
    );

    const volScalarField::Internal ReOmega(sqr(y)*omega/nu);
    const volScalarField::Internal Fwake(exp(-sqr(ReOmega/1e5)));

    return tmp<volScalarField::Internal>
    (
        new volScalarField::Internal
        (
            IOobject::groupName("Fthetat", this->alphaRhoPhi_.group()),
            min
            (
                max
                (
                    Fwake*exp(-pow4((y/delta))),
                    (1 - sqr((gammaInt_() - 1.0/ce2_)/(1 - 1.0/ce2_)))
                ),
                scalar(1)
            )
        )
    );
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal>
kOmegaSSTLM<BasicTurbulenceModel>::ReThetac() const
{
    tmp<volScalarField::Internal> tReThetac
    (
        new volScalarField::Internal
        (
            IOobject
            (
                IOobject::groupName("ReThetac", this->alphaRhoPhi_.group()),
                this->runTime_.timeName(),
                this->mesh_
            ),
            this->mesh_,
            dimless
        )
    );
    volScalarField::Internal& ReThetac = tReThetac.ref();

    forAll(ReThetac, celli)
    {
        const scalar ReThetat = ReThetat_[celli];

        ReThetac[celli] =
            ReThetat <= 1870
          ?
            ReThetat
          - 396.035e-2
          + 120.656e-4*ReThetat
          - 868.230e-6*sqr(ReThetat)
          + 696.506e-9*pow3(ReThetat)
          - 174.105e-12*pow4(ReThetat)
          :
            ReThetat - 593.11 - 0.482*(ReThetat - 1870);
    }

    return tReThetac;
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::Flength
(
    const volScalarField::Internal& nu
) const
{
    tmp<volScalarField::Internal> tFlength
    (
        new volScalarField::Internal
        (
            IOobject
            (
                IOobject::groupName("Flength", this->alphaRhoPhi_.group()),
                this->runTime_.timeName(),
                this->mesh_,
                IOobject::NO_READ,//Added by Minghan
                IOobject::AUTO_WRITE//Added by Minghan No output in time internal folders!
            ),
            this->mesh_,
            dimless
        )
    );
    volScalarField::Internal& Flength = tFlength.ref();

    const volScalarField::Internal& omega = this->omega_();
    const volScalarField::Internal& y = this->y_();

    forAll(ReThetat_, celli)
    {
        const scalar ReThetat = ReThetat_[celli];

        if (ReThetat < 400)
        {
            Flength[celli] =
                398.189e-1
              - 119.270e-4*ReThetat
              - 132.567e-6*sqr(ReThetat);
        }
        else if (ReThetat < 596)
        {
            Flength[celli] =
                263.404
              - 123.939e-2*ReThetat
              + 194.548e-5*sqr(ReThetat)
              - 101.695e-8*pow3(ReThetat);
        }
        else if (ReThetat < 1200)
        {
            Flength[celli] = 0.5 - 3e-4*(ReThetat - 596);
        }
        else
        {
            Flength[celli] = 0.3188;
        }

        const scalar Fsublayer =
            exp(-sqr(sqr(y[celli])*omega[celli]/(200*nu[celli])));

        Flength[celli] = Flength[celli]*(1 - Fsublayer) + 40*Fsublayer;
    }

    return tFlength;
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::ReThetat0
(
    const volScalarField::Internal& Us,
    const volScalarField::Internal& dUsds,
    const volScalarField::Internal& nu
) const
{
    tmp<volScalarField::Internal> tReThetat0
    (
        new volScalarField::Internal
        (
            IOobject
            (
                IOobject::groupName("ReThetat0", this->alphaRhoPhi_.group()),
                this->runTime_.timeName(),
                this->mesh_
            ),
            this->mesh_,
            dimless
        )
    );
    volScalarField::Internal& ReThetat0 = tReThetat0.ref();

    const volScalarField& k = this->k_;

    label maxIter = 0;

    forAll(ReThetat0, celli)
    {
        const scalar Tu
        (
            max(100*sqrt((2.0/3.0)*k[celli])/Us[celli], scalar(0.027))
        );

        // Initialize lambda to zero.
        // If lambda were cached between time-steps convergence would be faster
        // starting from the previous time-step value.
        scalar lambda = 0;

        scalar lambdaErr;
        scalar thetat;
        label iter = 0;

        do
        {
            // Previous iteration lambda for convergence test
            const scalar lambda0 = lambda;

            if (Tu <= 1.3)
            {
                const scalar Flambda =
                    dUsds[celli] <= 0
                  ?
                    1
                  - (
                     - 12.986*lambda
                     - 123.66*sqr(lambda)
                     - 405.689*pow3(lambda)
                    )*exp(-pow(Tu/1.5, 1.5))
                  :
                    1
                  + 0.275*(1 - exp(-35*lambda))
                   *exp(-Tu/0.5);

                thetat =
                    (1173.51 - 589.428*Tu + 0.2196/sqr(Tu))
                   *Flambda*nu[celli]
                   /Us[celli];
            }
            else
            {
                const scalar Flambda =
                    dUsds[celli] <= 0
                  ?
                    1
                  - (
                      -12.986*lambda
                      -123.66*sqr(lambda)
                      -405.689*pow3(lambda)
                    )*exp(-pow(Tu/1.5, 1.5))
                  :
                    1
                  + 0.275*(1 - exp(-35*lambda))
                   *exp(-2*Tu);

                thetat =
                    331.50*pow((Tu - 0.5658), -0.671)
                   *Flambda*nu[celli]/Us[celli];
            }

            lambda = sqr(thetat)/nu[celli]*dUsds[celli];
            lambda = max(min(lambda, 0.1), -0.1);

            lambdaErr = mag(lambda - lambda0);

            maxIter = max(maxIter, ++iter);

        } while (lambdaErr > lambdaErr_);

        ReThetat0[celli] = max(thetat*Us[celli]/nu[celli], scalar(20));
    }

    if (maxIter > maxLambdaIter_)
    {
        WarningInFunction
            << "Number of lambda iterations exceeds maxLambdaIter("
            << maxLambdaIter_ << ')'<< endl;
    }

    return tReThetat0;
}


template<class BasicTurbulenceModel>
tmp<volScalarField::Internal> kOmegaSSTLM<BasicTurbulenceModel>::Fonset
(
    const volScalarField::Internal& Rev,
    const volScalarField::Internal& ReThetac,
    const volScalarField::Internal& RT
) const
{
    const volScalarField::Internal Fonset1(Rev/(2.193*ReThetac));

    const volScalarField::Internal Fonset2
    (
        min(max(Fonset1, pow4(Fonset1)), scalar(2))
    );

    const volScalarField::Internal Fonset3(max(1 - pow3(RT/2.5), scalar(0)));

    return tmp<volScalarField::Internal>
    (
        new volScalarField::Internal
        (
            IOobject::groupName("Fonset", this->alphaRhoPhi_.group()),
            max(Fonset2 - Fonset3, scalar(0))
        )
    );
}


// * * * * * * * * * * * * * * * * Constructors  * * * * * * * * * * * * * * //

template<class BasicTurbulenceModel>
kOmegaSSTLM<BasicTurbulenceModel>::kOmegaSSTLM
(
    const alphaField& alpha,
    const rhoField& rho,
    const volVectorField& U,
    const surfaceScalarField& alphaRhoPhi,
    const surfaceScalarField& phi,
    const transportModel& transport,
    const word& propertiesName,
    const word& type
)
:
    kOmegaSST<BasicTurbulenceModel>
    (
        alpha,
        rho,
        U,
        alphaRhoPhi,
        phi,
        transport,
        propertiesName,
        typeName
    ),

    ca1_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "ca1",
            this->coeffDict_,
            2
        )
    ),
    ca2_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "ca2",
            this->coeffDict_,
            0.06
        )
    ),
    ce1_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "ce1",
            this->coeffDict_,
            1
        )
    ),
    ce2_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "ce2",
            this->coeffDict_,
            50
        )
    ),
    cThetat_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "cThetat",
            this->coeffDict_,
            0.03
        )
    ),
    sigmaThetat_
    (
        dimensionedScalar::lookupOrAddToDict
        (
            "sigmaThetat",
            this->coeffDict_,
            2
        )
    ),
    lambdaErr_
    (
        this->coeffDict_.lookupOrDefault("lambdaErr", 1e-6)
    ),
    maxLambdaIter_
    (
        this->coeffDict_.lookupOrDefault("maxLambdaIter", 10)
    ),
    deltaU_("deltaU", dimVelocity, SMALL),

    ReThetat_
    (
        IOobject
        (
            IOobject::groupName("ReThetat", alphaRhoPhi.group()),
            this->runTime_.timeName(),
            this->mesh_,
            IOobject::MUST_READ,
            IOobject::AUTO_WRITE
        ),
        this->mesh_
    ),

    gammaInt_
    (
        IOobject
        (
            IOobject::groupName("gammaInt", alphaRhoPhi.group()),
            this->runTime_.timeName(),
            this->mesh_,
            IOobject::MUST_READ,
            IOobject::AUTO_WRITE
        ),
        this->mesh_
    ),

    gammaIntEff_
    (
        IOobject
        (
            IOobject::groupName("gammaIntEff", alphaRhoPhi.group()),
            this->runTime_.timeName(),
            this->mesh_,
            IOobject::NO_READ,//Added by minghan: no_read means initial value (file) of gammaInteff in 0 is needed!
            IOobject::AUTO_WRITE//will write the field of gammaIntEff_ at control time step
        ),
        this->mesh_,
        dimensionedScalar(dimless, Zero)//Also note since no read is used, dimesion needs to be specified
    )
{
    if (type == typeName)
    {
        this->printCoeffs(type);
    }
}


// * * * * * * * * * * * * * * * Member Functions  * * * * * * * * * * * * * //

template<class BasicTurbulenceModel>
bool kOmegaSSTLM<BasicTurbulenceModel>::read()
{
    if (kOmegaSST<BasicTurbulenceModel>::read())
    {
        ca1_.readIfPresent(this->coeffDict());
        ca2_.readIfPresent(this->coeffDict());
        ce1_.readIfPresent(this->coeffDict());
        ce2_.readIfPresent(this->coeffDict());
        sigmaThetat_.readIfPresent(this->coeffDict());
        cThetat_.readIfPresent(this->coeffDict());
        this->coeffDict().readIfPresent("lambdaErr", lambdaErr_);
        this->coeffDict().readIfPresent("maxLambdaIter", maxLambdaIter_);

        return true;
    }
    else
    {
        return false;
    }
}


template<class BasicTurbulenceModel>
void kOmegaSSTLM<BasicTurbulenceModel>::correctReThetatGammaInt()
{
    // Local references
    const alphaField& alpha = this->alpha_;
    const rhoField& rho = this->rho_;
    const surfaceScalarField& alphaRhoPhi = this->alphaRhoPhi_;
    const volVectorField& U = this->U_;
    const volScalarField& k = this->k_;
    const volScalarField& omega = this->omega_;
    const tmp<volScalarField> tnu = this->nu();
    const volScalarField::Internal& nu = tnu()();
    const volScalarField::Internal& y = this->y_();
    fv::options& fvOptions(fv::options::New(this->mesh_));

    // Fields derived from the velocity gradient
    tmp<volTensorField> tgradU = fvc::grad(U);
    const volScalarField::Internal Omega(sqrt(2*magSqr(skew(tgradU()()))));
    const volScalarField::Internal S(sqrt(2*magSqr(symm(tgradU()()))));
    const volScalarField::Internal Us(max(mag(U()), deltaU_));
    const volScalarField::Internal dUsds((U() & (U() & tgradU()()))/sqr(Us));
    tgradU.clear();

    const volScalarField::Internal Fthetat(this->Fthetat(Us, Omega, nu));

    {
        const volScalarField::Internal t(500*nu/sqr(Us));
        const volScalarField::Internal Pthetat
        (
            alpha()*rho()*(cThetat_/t)*(1 - Fthetat)
        );

        // Transition onset momentum-thickness Reynolds number equation
        tmp<fvScalarMatrix> ReThetatEqn
        (
            fvm::ddt(alpha, rho, ReThetat_)
          + fvm::div(alphaRhoPhi, ReThetat_)
          - fvm::laplacian(alpha*rho*DReThetatEff(), ReThetat_)
         ==
            Pthetat*ReThetat0(Us, dUsds, nu) - fvm::Sp(Pthetat, ReThetat_)
          + fvOptions(alpha, rho, ReThetat_)
        );

        ReThetatEqn.ref().relax();
        fvOptions.constrain(ReThetatEqn.ref());
        solve(ReThetatEqn);
        fvOptions.correct(ReThetat_);
        bound(ReThetat_, 0);
    }

    const volScalarField::Internal ReThetac(this->ReThetac());
    const volScalarField::Internal Rev(sqr(y)*S/nu);
    const volScalarField::Internal RT(k()/(nu*omega()));

    {
        const volScalarField::Internal Pgamma
        (
            alpha()*rho()
           *ca1_*Flength(nu)*S*sqrt(gammaInt_()*Fonset(Rev, ReThetac, RT))
        );

        const volScalarField::Internal Fturb(exp(-pow4(0.25*RT)));

        const volScalarField::Internal Egamma
        (
            alpha()*rho()*ca2_*Omega*Fturb*gammaInt_()
        );

        // Intermittency equation
        tmp<fvScalarMatrix> gammaIntEqn
        (
            fvm::ddt(alpha, rho, gammaInt_)
          + fvm::div(alphaRhoPhi, gammaInt_)
          - fvm::laplacian(alpha*rho*DgammaIntEff(), gammaInt_)
        ==
            Pgamma - fvm::Sp(ce1_*Pgamma, gammaInt_)
          + Egamma - fvm::Sp(ce2_*Egamma, gammaInt_)
          + fvOptions(alpha, rho, gammaInt_)
        );

        gammaIntEqn.ref().relax();
        fvOptions.constrain(gammaIntEqn.ref());
        solve(gammaIntEqn);
        fvOptions.correct(gammaInt_);
        bound(gammaInt_, 0);
    }

    const volScalarField::Internal Freattach(exp(-pow4(RT/20.0)));
    const volScalarField::Internal gammaSep
    (
        min(2*max(Rev/(3.235*ReThetac) - 1, scalar(0))*Freattach, scalar(2))
       *Fthetat
    );

    gammaIntEff_ = max(gammaInt_(), gammaSep);
}


template<class BasicTurbulenceModel>
void kOmegaSSTLM<BasicTurbulenceModel>::correct()
{
    if (!this->turbulence_)
    {
        return;
    }

    // Correct k and omega
    kOmegaSST<BasicTurbulenceModel>::correct();

    // Correct ReThetat and gammaInt
    correctReThetatGammaInt();
}


// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

} // End namespace RASModels
} // End namespace Foam

// ************************************************************************* //

```

