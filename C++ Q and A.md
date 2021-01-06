# C++ Questions and Answers

1. I just found **you can use different names for the parameters when declaring functions and that when defining the functions. Only rule to hold: ensure you stick with **the same type for these parameters being passed into the functions.**

   ```c++
   class Numerics
   {
   	 virtual void EigenDecomposition(double** m_A_ij, double** m_Eig_Vec, double* m_Eig_Val, unsigned short n);//note different 
     	//names with m_ preceded compared to the its definition below
   }
   void Numerics::EigenDecomposition(double** A_ij, double** Eig_Vec, double* Eig_Val, unsigned short n)
   {
   	//definitions
   }
   ```

2. When calling the function, **no** **type** need to be declared! That has something to do with **lvalue**, but left to the future to dig deeper. However, you must declare them clearly. The following example shows declarations for **2D arrays**, see how detail this should be!

   ```c++
   //***************** Parent class Numerics.H Start **********************************************//
   class Numerics
   {
      protected://The following declarations are defined in the base and are actually used in the subclass
           double** m_A_ij;
           double* m_Eig_Val;
           double** m_Eig_Vec;
           double** m_newA_ij;
           double** m_New_Eig_Vec;
       
       public:
           //constructor
           Numerics();
           
           //destructor
           virtual ~Numerics();
           
           //member function (note m_Aij, m_Eig_Vec and m_Eig_Val could be any names!)
           virtual void EigenDecomposition(double** A_ij, double** Eig_Vec, double* Eig_Val, unsigned short n);
                   
   };
   //***************** End **********************************************//
   
   
   //***************** Parent class Numerics.C Start **********************************************//
   //member functions
   Numerics::Numerics() 
   {
                  //std::cout <<"m_Aij= "<< m_Aij <<" m_Eig_Val= "<< m_Eig_Val <<" m_Eig_Vec= "<<m_Eig_Vec<<"\n";
                   //std::cout<<"m_newA_ij= "<<m_newA_ij<<" m_New_Eig_Vec= "<<m_New_Eig_Vec<<std::endl;
   
                   m_A_ij = new double* [3];
                   m_Eig_Val = new double [3];
                   m_newA_ij = new double* [3];
                   m_Eig_Vec = new double* [3];
                   m_New_Eig_Vec = new double* [3];
                   for (unsigned short i = 0; i < 3; i++)
                   {
                       m_A_ij[i] = new double [3];
                       m_Eig_Val[i] = 0;
                       m_newA_ij[i] = new double [3];
                       m_Eig_Vec[i] = new double [3];
                       m_New_Eig_Vec[i] = new double [3];
                   }
   }
   ```

3. **Observed fact - parameter name vs variable name** 

   Within a same scope variable name should be same as its corresponding (sits at the same location) parameter name, e.g.

   ```c++
   void UQ::EigenSpace (symmTensor& m_newBij_OF, symmTensor& m_newuiuj_OF, double** m_newB_ij, double** m_MeanPerturbedRSM, 
   					 symmTensor& _Bij, unsigned short& _celli)
   					 {
   					 		PerturbedAij(m_newB_ij, m_MeanPerturbedRSM);//note m_newB_ij and m_MeanPerturbedRSM can be found in the 																													        //"EigenSpace" declaration parameters
   					 }
   
   ```

   When calling a function **inside a different scope** you may use a different parameter name as long as these parameters are put in the same locations, e.g.

   ```c++
   void UQ::EigenSpace (symmTensor& m_newBij_OF, symmTensor& m_newuiuj_OF, double** m_newB_ij, double** m_MeanPerturbedRSM, 
   					 symmTensor& _Bij, unsigned short& _celli)
   					 {
   					 		PerturbedAij(m_newB_ij, m_MeanPerturbedRSM);//Note here you may use different names other than "m_newB_ij" and 				   //"m_MeanPerturbedRSM" however this is not recommended because it is error prone and you also need to define new 						//variables that occupy new memories, so always keep the parameter same name as its corresponding variable name
   					 }
   					 
   					 void UQ::PerturbedAij(double** m_newB_ij, double** m_MeanPerturbedRSM)
              {
                ...
              }
   ```

   4. a. Inside a **same translation unit**, one member function can see variables defined in another member function (meaning seeing across different scopes). However, if a variable must be used **by first calling** a function defined in a **different translation unit**, you must pass the wanted variable by reference (by calling the particular function) inside the current scope where you are using this variable. 

      b. **Important:** 

      passing by reference: calling the function inside a **same translation unit**, `e.g. void(var_1, var_2` corresponds to its actual definition `void (type& name_1, type& name_2)`, you can have your variable name differ from the names used in its actual definitions. 

      passing by pointer: calling the function inside a **same translation unit,** `e.g. void (var_1, var_2)` corresponds to its actual definition `void (type** var_1, type** var_2)` , consistent names in terms of both actual name and location must be used .

      

      Similarities: inside a **same translation unit and inside one member function** always ensure consistent names used 

      ```c++
      //declaration
      int** m_var_1 = new int* m_var_1*[3];
        for (int i = 0; i < 3; i++)
          m_var_1[i] = i+1;
      
      int m_ref_1 = 2;
      
      {
        void (m_var_1, m_ref_1) //note when calling the void function, variable must stick to the declared names 
      }
      
      //void member function is defined 
      void member(int** var_1, int& ref_1)//inside definition variable may differ from the declared names however always keep 
        //using the consistent names to prevent errors
      {
      	var_1 = 1; //same name as passing by reference or by pointer
      	ref_1 = 2;
      }
      ```

      

      Differences: inside **same translation unit** but cross different member functions, passing by pointer must have same names (parameter and body) while passing by reference could have different names (parameter and body), check it out later in detail. 

      The rule of sum is: always keep using the consistent names!!

      Recall: **passing by reference** is an **alias** for a declared variable and occupies no memory.  **ONLY** declare a variable once for passing by reference, and note always **declaration precedes** passing the declared variables by reference. Because passing by reference can only occur when  a variable exists. 

      



















