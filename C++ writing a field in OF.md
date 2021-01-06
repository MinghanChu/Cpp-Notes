# Nov 12 2020 - Summary of writing a field_One 

1. When creating a new `IOobject` inside a constructor, you need to declare it in the corresponding H file
2. When creating a new `IOobject` inside a ==constructor (if not inside constructor no warnings)==  of a base class, warnings being generated whiling compiling indicating that you are prone to any influences on the child classes, e.g. LESmodel, that inherent from this base class. So please make copy of the base class and give it a new name and use it as your customized base class. I have tested the changes I made to `kOmegaSSTBase.C` will not only affect the `kOmegaSST.C` but also affect, for example, the `kOmegaSSTLM.C`
3. Inside `IOobject` , the argument `NO_READ` means no input file is needed (Tested in T3A)
4. When creating a `IOobject`, if dimension of a variable is **not** specified you will be asked to create a file of initial condition for this variable inside **0** directory
5. **IMPORTANT**: in addition to *3* and *4*, you should always write a `void` function to be called for any variable to be outputted, **NEVER Return** (DO NOT KNOW WHY always return wrong values maybe related to references) your outputted variable, see the example below:

`````c++
//more code above 
omegaInf_
    (
        dimensioned<scalar>::lookupOrAddToDict
        (
            "omegaInf",
            this->coeffDict_,
            omega_.dimensions(),
            0
        )
    ),

TestRij_//uisng void functions without a return, field type must be declared here! As well as in .H file!
        (
                IOobject
                (
                    IOobject::groupName("TestRij_", alphaRhoPhi.group()),
                    this->runTime_.timeName(),//runTime is an object of Time class
                    this->mesh_,//mesh is an object of polyMesh or fvMesh class (geometricField must be 																								//declared in .H file)
                    IOobject::NO_READ,
                    IOobject::AUTO_WRITE
                ),
                this->mesh_,
                //dimensionedScalar(dimless, Zero)
                dimensionSet(0, 2, -2, 0, 0, 0, 0)
                //dimless
        )
     
     {
    bound(k_, this->kMin_);
    bound(omega_, this->omegaMin_);

    setDecayControl(this->coeffDict_);
}

/*template<class BasicTurbulenceModel>
tmp<volScalarField> kOmegaSSTBase<BasicTurbulenceModel>::ReynoldsStress() const
{
        
        //Info<< "Minghan in kOmegaSSTBase" << endl;
      
            
          
          return this->k_;
           
        //return ((2.0/3.0)*I)*this->k_ - (this->nut_)*dev(twoSymm(fvc::grad(this->U_)));
             
}*/ //This block of code is commentted out, as you can see "return" a value will result in errors 

//Inside kOmegaSSTBase.C file
template<class BasicTurbulenceModel>
void kOmegaSSTBase<BasicTurbulenceModel>::R_ij()//Note void function is used!
{
 
    //   tmp<volSymmTensorField> intermediate = kOmegaSSTBase<BasicTurbulenceModel>::devRhoReff();
   
    //TestRij_ = kOmegaSSTBase<BasicTurbulenceModel>::devRhoReff();
    TestRij_=((2.0/3.0)*I)*this->k_ - (this->nut_)*dev(twoSymm(fvc::grad(this->U_)));
    Info<<"Just test TestRij_"<<endl;
}

//In kOmegaSSTBase.H file, we need to clarify
volSymmTensorField::Internal TestRij_;//Note both the variable that is returned and the variable like "TestRij_" calculated in a void function need to be declared in a .H file. One exception is the kind of variable defined with a "New" operator indicating Heap memory is used without "return", see the example below
virtual void R_ij();
`````

`````c++
//Example
//There are a bunch of examples with "new-defined" variables with return at the end of the code block in the kOmegaSSTLM.C file. No declarations are needed for those variables in the .H file. This might be because these variables are defined as "new" and allocated on heap memory.

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

    return tReThetac;// return here!
}
`````

### In summary

+ **both variables calculated in ==a void function== or calculated ==then returned== must be declared in .H file and results can be ==outputted==**
+ **Whenever you are using a variable you must declare the type for it first!**
+ **For those  "==NEW-operator==" defined variables returned however NOT being declared (==It did was declared inside the .C file within its block of code: e.g. `tmp<volScalarField::Internal> tReThetac`==), e.g. the example above,  results ==cannot== be outputted. I think this is because "NEW" operator is used, and variables are allocated on heap memory.** 
+ **[When to use Void or Value-Returning Functions](https://www.cs.fsu.edu/~cop3014p/lectures/ch7/index.html):**
  1. Void Function: when must return more than one value or modify any of the caller's arguments
  2. Void Function: when must perform I/O
  3. Void Function: when in doubt, can recode any value-returning function as void function by adding additional outgoing parameter
  4. Value-returning Function: when only **one** value returned

1. I have noticed that  the cross-access to functions defined in different translation units (.cpp files) under the same namespace **foam** is allowed. Therefore I can access `R()` and `devRhoReff()`  functions inside `kOmegaSSTBase.C` file which are originally defined in `eddyViscosity.C` and `linearViscousStress.C` files, see example below

2. I have test the following code, and they gave same outputs (call a function with returned fields and get it stored in a variable)

   `````c++
   template<class BasicTurbulenceModel>
   tmp<volScalarField> kOmegaSSTBase<BasicTurbulenceModel>::ktest()
   {
   
        //   Info<<kOmegaSSTBase<BasicTurbulenceModel>::R()<<endl;
        Info<<kOmegaSSTBase<BasicTurbulenceModel>::devRhoReff()<<endl;//get output from log file
       tmp<volSymmTensorField> tdev = kOmegaSSTBase<BasicTurbulenceModel>::devRhoReff();
       Info<<tdev<<endl;//the above line assign devRhoReff() to the variable tdev and gave same results 
           tmp<volScalarField> ktest_ = 
             (
                 
             this->k_
             );
           return ktest_; 
   
   }
   `````

3. **IMPORTANT OBSERVATIONS:** continued from 1...

   + It is ==normal== to access the function defined in outer scope via the base class, you can also try to access using the parameter (or changing the parameter name if you want: see detailed explanation in the example below), and eventually get the same results:

     `````c++
     
     template<class BasicTurbulenceModel>
     void UQkOmegaSST<BasicTurbulenceModel>::UQ_R_ij()
     {
         
         //   tmp<volSymmTensorField> intermediate = kOmegaSSTBase<BasicTurbulenceModel>::devRhoReff();
        
         TestRij_UQ_ = kOmegaSSTBase<eddyViscosity<RASModel<BasicTurbulenceModel>>>::R();//Note this is the most COMMON way of accessing in OF and "kOmegaSSTBase<eddyViscosity<RASModel<BasicTurbulenceModel>>>" is just the base class of UQkOmegaSST, check the inheritance in UQkOmegaSST.H file
       
       //********The following are alternative ways of doing this***************
         //TestRij_ = UQkOmegaSST<BasicEddyViscosityModel>::R();//Will give the same result
         //TestRij_ = BasicEddyViscosityModel::R();//For example if you have redefined "BasicEdyViscosityModel" as the new parameter name (keep consistancy and replace all "BasicTurblenceModel" to "BasicEddyViscosityModel"!) will end up with the same result
         //TestRij_ = BasicTurbulenceModel::R(); //again will give the same result
     }
     `````

   + This piece of information is **rather important**! I have noticed that the base classes, e.g. "kOmegaSSTBase.C", **CANNOT** be used to write out recognizable fields for postprocessing, meaning even though the user-defined fields are outputted ok into each time folder while running they still do not appear using paraview or Techplot. This might be because these base class files are not registered or some reason through "addToRunTimeSelectionTable" (check functionObjects OpenFoam). ==Therefore always define output fields in subclass .cpp files, e.g. UQkOmegaSST.C== then these user defined fields will appear in paraview or techplot.

4. Next steps:

   + check if outputted results are correct
   + test ==EigenvalueDecomposition== 
   + and implement the ==barycentricTriangle== coordinates.

   + go review const
   + and &
   + return

