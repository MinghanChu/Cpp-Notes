# C++ Member initializer

### Introduction

1. To initialize class members in the **constructor.** **Always initialize** your member variables, **otherwise** you will use whatever left in the memories!
2. **Two ways** to achieve step 1.

3. In the **constructor initializer list**, you should **initialize the member variables** **in order** as the way of declaring those **member variables**. Initializing member variables in constructors via `:`.

`````c++
#include <iostream>
#include <vector>
#include <string>
#include <array>


class Entity
{
    private:
        int m_Score;
        std::string m_Name;

    public:
        Entity()//Note to initialize Entity by creating an Entity constructor
            : m_Score(0), m_Name("constructor initializer")
        {
              //put non-trivial tasks inside the constructor scoope, not just trivial initializing variables
        }

        Entity(const std::string& name)
            : m_Name(name)
        {
              //put non-trivial tasks inside the constructor scoope, not just trivial initializing variables
        }

        const std::string& GetName() const {return m_Name;}
};



int main()
{
    Entity e0;
    std::cout << e0.GetName() << std::endl;

    Entity e1("Minghan as a parameter");
    std::cout << e1.GetName() << std::endl;

    std::cin.get();
}
`````



4. **why don't we initialize the member variables just directly inside the constructor scope?** **Ans:** In that case it is a rather trivial task, and when you have numerous member variables it becomes difficult to concentrate on what actually the constructor does.

   + But there is **actually a functional difference**! This differences applies to **classes** specifically.  See the example

     ```c++
     class Entity
     {
         private:
             int m_Score;
             std::string m_Name;
     
         public:
             Entity()//Note to initialize Entity by creating an Entity constructor
                 : m_Score(0)
             {
                   m_Name = "constructor initializer"; //Note in this way the m_Name object is constructed twice!! Once in the 
                   																	//default constructor
             }
     
             Entity(const std::string& name)
                 : m_Name(name)
             {
                   //put non-trivial tasks inside the constructor scoope, not just trivial initializing variables
             }
     
             const std::string& GetName() const {return m_Name;}
     };
     
     ...
     ```

     **Note if you do not use member initializer lists, object will be constructed twice! That is wasting the performance!** This implies after putting member variables in initializer lists, program will ignore the default constructor and just use the initialized constructor.

5. **In summary,** you should be using **member initializer lists everywhere**. It is not only a matter of style but it is a matter of **functional difference**. 

6. **Appendix:** show how to call constructor `//m_Example = Example(8);//Example object`  **Example** is a class object and can be passed as a parameter into other functions. **Example** is instantiated as `m_Example` inside the **Entity** class object. The **instance** of **Example**, `m_Example`, can be used to **1.** access the members of **Example class** and **2.** to call its constructor by `m_Example = Example(8)`.  

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <array>
   
   class Example
   {
       public:
           Example()//constructor 1 without taking parameters
           {
               std::cout << "Created Entity!" << std::endl;
           }
   
           Example(int x)//constructor 2 taking one paramter
           {
               std::cout << "Created Entity with  " << x << "!" <<std::endl;
           }
   };
   
   class Entity
   {
       private:
           int m_Score;
           std::string m_Name;
           Example m_Example; //Note you can instantiate a class object inside another class! But will also run the code!
   
       public:
           Entity()//Note to initialize Entity by creating an Entity constructor
               : m_Score(0), m_Name("constructor initializer"), m_Example(Example(8)) //equivalent to m_Example(8)
           {
               //m_Example = Example(8);//Example object
           }
   
           Entity(const std::string& name)
               : m_Name(name)
           {
           }
   
          virtual const std::string& GetName() const {return m_Name;}
   
   
   
   };
   
   
   int main()
   {
       Entity e0;//instantiate the object entity through an instance e0. 
     	e0 = Entity(); //meaning call the constructor of Entity, same as Entity e0;
    
   }
   ```

   