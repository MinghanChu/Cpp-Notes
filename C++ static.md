# C++ static

### Introduction

1. **Static** ==outside== of a **class/struct**

   + **linkage** to the variable **declared to be static** is internal, meaning only **visible to the translation unit** in which the **static variable** is defined.

   + Several examples

     ````c++
     //main.cpp file
     #include <iostream>
     
     int s_Variable = 10;//global variable
     
     int main()
     {
     
         std::cout << s_Variable << std::endl;
         std::cin.get();
     }
     
     //test.cpp file
     static int s_Variable = 5;//will only be linked internally to this 
                                   //current translation unit
     //*****Output****//
     //10
      
     //compile fine because the global static int s_Variable is not visible to the s_Variable in main.cpp
     ````

     `````c++
     //main.cpp file
     #include <iostream>
     
     extern int s_Variable;//external linkage
     
     
     int main()
     {
     
         std::cout << s_Variable << std::endl;
         std::cin.get();
     }
     
     //test.cpp file
     int s_Variable = 5;
     
     //*****Output****//
     //5
     
     //compile fine because the extern ensure only the s_Variable in test.cpp read
     `````

     `````c++
     //main.cpp file
     #include <iostream>
     
     int s_Variable=10;//external linkage
     void ()
     {}
     int main()
     {
     
         std::cout << s_Variable << std::endl;
         std::cin.get();
     }
     
     //test.cpp file
     int s_Variable = 5;
     
     static void()//ensure no duplicated void function
     {}
     //linkage error due to duplicating s_Variable
     `````

     

2. **Static** ==inside== of a **class/struct**（key words: global, namespace, lifetime）

   **The purpose of this:** when you want to share a piece of information inside a class with all instances of the class.

   + The **the static variable** is going to **share the memory with all instances** of the class. This means no matter how many instances/objects you have created for that class/struct, there will be **only one** instance for that **static variable**.

   + If **one** of those instances changed the **static variable**, it is going to reflect that changes **across all instances!**(all instances are pointing to the **same** memory!)

   + Therefore it is meaningless to refer to a **static variable** through a class instance (e.g.`class_name::s_Var` like accessing a variable in a namespace called `class_name`).

   + **A static method** does **not** have access to class instances!

   + **A static method** can be called without class instances!

   + **Inside a static method** you **cannot** write a code to refer to a class instance, because **no such an instance** to refer to!

   + **Static variables** inside a class/struct are not their member, therefore we have to declare **static variables** outside of the class/struct (global).

     

     `````c++
     #include <iostream>
     struct Entity
     {
         static int x, y;//static member variables
     
         static void Print()//static member function
         {
             std::cout<< x<<", "<< y << std::endl;
         }
     };
     
     
     int Entity::x;//declared in global indicating lifetime is for full program 
     int Entity::y;
     
     int main()
     {
         Entity e; //instance
         Entity::x = 2;
         Entity::y = 3;
     
         Entity e1;
         Entity::x = 5;
         Entity::y = 8;//no meaning to set different values for x and y because only one shared class 											//instance is visible to static variables
     
         Entity::x = 7;//access static member variable inside a class
         Entity::y = 4;
         Entity::Print();
         Entity::Print();//accessing static member function inside a class
     
     
         std::cin.get();
     }
     
     `````

   + Also, **static methods** cannot access **non-static variables!** Why? Because a static method **dose not** have a class instance. How class works: **every non-static method** coded into a class always gets the **instance** of the current class as a parameter! ==There is no such a thing called class they are just functions with a hidden parameter.== Therefore a static method **does not** get that hidden parameter (or class instance). So a **static method** is equivalent to if you write a class outside of a class (**lifetime of the entire program**). But still a **static method** is restricted to the the rules of class (**scope is limited to that class**), e.g. you cannot manually/deliberately pass the **created class instance** as a parameter into the **static method.** See the following example.

   + In short, **static methods** **only use** the class name as its **namespace**, but is essentially defined **in global**, only seeing **static variables in the class** which are actually **defined in global**. ==non-static symbols do not work well with static symbols!==

     `````c++
     //you will get error message saying no x and y are defined, the reason is above two points.
     #include <iostream>
     struct Entity
     {
         int x, y;//changed to non-static member variables
     
         static void Print()//static member function inside the Entity class is not taking the class 
           								 //instance as a parameter, therefore an error is generated
         {
             std::cout<< x<<", "<< y << std::endl;
         }
     };
     
     static void Print(Entity e)//if we put the static method outside of the class and manually pass the 													//class instance into the print() function, it works
     {
         std::cout<< e.x<<", "<< e.y << std::endl;
     }
     
     //int Entity::x;//declare only static variables!
     //int Entity::y;
     
     int main()
     {
       Entity e; //instance
         e.x = 2;
         e.y = 3;
     
         Entity e1;//instance1
         e1.x = 5;
         e1.y = 8;
     
         Entity::Print();
         Entity::Print();//accessing static member function inside a class
     
     
         std::cin.get();
     }
     `````

   

3. **local static variable** has the **lifetime of the entire program** however its **scope** is inside that function where it is declared in, e.g. inside a function, inside a if statement, basically anywhere.

   `````c++
   
   #include <iostream>
   
   
   
   void Function()
   {
       static int i = 0;//local static variable has its lifetime of the entire program! 
                        //only visible to this Function() scope, meaning you cannot
                        //change i in other scopes! value of i is updated and fixed everytime
     									 //without being freed up coming to the end of the void function scope!
       i++;
       std::cout << i << std::endl;
   }*/
   
   int Entity::x;//declare only static variables!
   int Entity::y;
   
   int main()
   {
   	Function();
   	Function();
   	Function();
   	Function();
   	Function();
   
       std::cin.get();
     //******Output******//
     /*1
       2
     	3
     	4
     	5*/
   }
   `````

   



