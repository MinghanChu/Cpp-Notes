# C++ Call hidden functions in OF

See the following example



`````c++
//Example shows the nut() function is declared inside the eddyViscosity.H file. Note nut()is defined in RASModel.H which is included in most turbulence models. Therefore when the eddyviscosity.C is called inside one of these turbulence models, the eddyviscosity.C will see the definition of nut(), and return the value of nut_ as needed




//- Return the turbulence viscosity from RASModel.H because eddyvicosity.C will be called in which RASModel is included, only declaration is needed inside the eddyviscoisty.C file as follows
        virtual tmp<volScalarField> nut() const
        {
            return nut_;
        }
`````

