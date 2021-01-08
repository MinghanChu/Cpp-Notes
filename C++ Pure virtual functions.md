# C++ Pure virtual functions

### Introduction (augmented version of virtual functions)

Pure virtual functions are very important and frequently encountered in OpenFoam, e.g. in `turbulenceModel.H`. 

1. **Pure virtual function** is essentially the **same as** an **abstract method or interface** (in Java or C-shop). 

2. **Pure virtual function** allows us to define a function in the **base class** that does not have an implementation, and **force** subclass to implement that function.

3. **Important!** You can only instantiate a subclass if the **pure virtual function** has **been implemented** inside the subclass! **Again declared in the base class and MUST be implemented in the subclasses!**

   Or the class was further up in the hierarchy, e.g. **A** is the subclass of another **subclass** which did implement the virtual function. 

4. **In c++**, "Interfaces" are just classes.



```c++

#include <iostream>
#include <string>

class Printable //This is called "interface" or "abstract base class" because all it has is the pure virtual function
{
    public:
        virtual std::string GetClassName() = 0;//"0" means pure virtual and implies must be 
                                               //implemented in a subclass if you want to 
                                               //instantiate that class
};

class Base : public Printable //means "Base" has inherited the "GetClassName()" pure virtual function
{
    public:
        virtual std::string GetName(){return "Minghan in Base";} 
        std::string GetClassName() override {return "Minghan in Base_Implement pure virtual";}
};

//note subclass is already the Entity that has inherited the pure virtual function from "Printable"
//if it wasn't for some reason do: "class Subclass : public Base, Printable". Note just add ", Printable"
class Subclass : public Base 
{
    private:
        std:: string m_Name;

    public:
        Subclass(const std::string& name)//name is just for being getting information for the 
            : m_Name(name) {}            //member variable m_Name

        std::string GetName() override {return m_Name;}//Note always manipulate member
                                              //variables
        std::string GetClassName() override {return "Minghan in Subclass_Implement pure virtual";}



};

void checkName(Base* v_b) // always use the class in the higher hierarchy which contains the vtable for both
                          //itself and its child class. In this case, if "subclass* v_b" is used, 
                          //then no match for the GetName() if the pointer to the "Base" class is being
                          //executed. 
{
    std::cout << v_b->GetName() << std::endl;
}

void Print(Printable* obj) //all class names (the Base and Subclass) get printing via this print function
{
    std::cout << obj->GetClassName() << std::endl;
}

int main()
{

    Base* b = new Base();//No parameter
    
    Subclass* s = new Subclass("Minghan in Subclass");

    Print(b); //check the pure virtual function
    Print(s); //check the pure virtual function
    checkName(b); //check the virtual function
    checkName(s); //check the virtual function

   
    
     

    std::cin.get();
}

/*
Minghan in Base_Implement pure virtual
Minghan in Subclass_Implement pure virtual
Minghan in Base
Minghan in Subclass
*/
```



**Summary from OpenFoam point of view**

Virtual functions are **really important** to classes and inheritance!!! Virtual functions allow us to **override methods** in subclasses. **Templated classes plus inheritance** are the essential concepts in OpenFoam environment. A good example is `turbulenceModel.H`  that contains a bunch of **pure virtual functions** that are implemented in its subclasses. 

**So what are these subclasses of `turbulenceModel.H`?** From my studies, I can list some: it should include all **turbulence models, e.g. kOmegaSST and kOmegaSSTML**, where `k()` , `omega()`, `correct()` and `elpsilon()` methods are implemented.

In `turbulenceModel.H`, ==`virtual void correct() = 0` solves the turbulence equations and corrects the turbulence viscosity==

​										  `virtual tmp<volScalarField> nu() const = 0` returns the laminar viscosity

​										  `virtual tmp<volScalarField> nut() const = 0` returns the turbulence viscosity

​										 `virtual tmp<volScalarField> nuEff() const = 0` returns the effective viscosity

​									 	`virtual tmp<volScalarField> mu() const = 0` returns the laminar dynamic viscosity

​										 `virtual tmp<volScalarField> mut() const = 0` returns the turbulence dynamic viscosity

​										 `virtual tmp<volScalarField> muEff() const = 0` returns the effective dynamic viscosity

​										==`virtual tmp<volScalarField> k() const = 0` returns the turbulence kinetic energy==

​										`virtual tmp<volScalarField> epsilon() const = 0` returns the turbulence kinetic energy dissipation rate

​									   `virtual tmp<volScalarField> omega() const = 0` returns the specific dissipation rate

​									   `virtual tmp<volSymmTensorField> R() const = 0` returns the Reynolds stress tensor	

In `transportModel.H`, `virtual void correct()=0` corrects the laminar viscosity.

​										`virtual tmp<volScalarField> nu() const = 0` returns the laminar viscosity

​										`virtual bool read() = 0` reads transportProperties dictionary

In `incompressibleTurbulenceModel.H`, `virtual tmp<volSymmTensorField> devReff() const = 0` returns the effective stress tensor including the laminar stress



In `linearViscosityStress.H`, ==`virtual void correct() = 0` solves the turbulence equations and corrects the turbulence viscosity==

​													  `virtual bool read() = 0`  re-read model coefficients if they have changed

it should be noted that in `linearViscosityStress.C`, the effective stress tensor `devRhoReff()`,  the source term for the momentum equation `divDevRhoReff(volVectorField& U)`and `divDevRhoReff(volScalarField& rho, volVectorField& U)` are returned.



In `eddyViscosity.H`, `virtual bool read() = 0` re-read coefficients if they have changed

​									  ==`virtual tmp<volScalarField> k() const = 0` return the turbulence kinetic energy==

​									  ==`virtual void correct() = 0` solves the turbulence equations and correct the turbulence viscosity==

it should be noted, in `eddyViscosity.H`, Reynolds stress tensor `R()` is defined. 