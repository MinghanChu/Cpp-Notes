# C++ linker

### Introduction

1. **Linking** is the process triggered when files (translation units) are bing compiled.
2. Recall each file is compiled into separate **object files (.o files)** as **a translation unit** and they have no relation to each other without **linking**, meaning these files cannot interact!
3. The **main purpose of linking**: if we decide to **split our program** into **multiple c++** files, we need a way to link those files together into **one program!**
4. Even if you write your entire program **into one file**, your application still need to know where your **entry point or your main function** is!
5. Note an **entry point** does not necessarily have to be **a main function!** But normally is.

5. When you actually run your **application**, your **C runtime library** will jump to your **main function** and start executing code from there.

6. You need to know whether it is a **compiling error or a linking error!**

7. One way to debug **linking error** is to comment out any **referenced function(s)** that keep generating the error "undefined reference to xxx function". The **logic behind** is this: once a problematic function is commented out, the **linker will skip** that function! If I deliberately make a mistake "Logr" instead of "Log", then a **linking** error will pop up. To avoid this **linking error,** you can just comment out the **reference to the Log function：**

   `````c++
   //in one translation unit (.cpp file)
   void Log(const char* message);
   
   
   int Multiply(int a, int b)
   {
       //Log("Multiply");//comment it out to avoid linking error
       return a * b;
   }
   
   int main()
   {
       std::cout<< Multiply(5, 8) <<std::endl;
       
       std::cin.get();
   }
   
   `````

   `````c++
   //The Log function is defined in a second translation unit (.cpp file)
   #include <iostream>
   
   
   void Logr(const char* message)
   {
   
       std::cout << "Mnghan is inside log" << std::endl;
   }
   `````

   

8. Compared to the above example, if instead you leave the `Log` function uncommented, however comment the **reference to the Multiply()** inside the `main()` function, you will still get the **linking error**, because linker assumes that this **Log function** would be referenced in other translation units. Therefore you must mark the **Log** function as **static**, meaning **only visible to the current translation unit** to avoid this problem:

   `````c++
   .....
   
   static int Multiply(int a, int b)//mark as static
   {
       Log("Multiply");//leave it here
       return a * b;
   }
   
   int main()
   {
       //std::cout<< Multiply(5, 8) <<std::endl;//instead comment out the reference to the Multiply function in which Log function is referenced
       
       std::cin.get();
   }
   
   
   `````

   **In summary**: if you do not want to comment out the problematic function, you must mark it as **static**, **even though** this function is **not referenced** inside the current translation unit!

9. Another linking error:

   **always be consistent** (exactly same): same type and number of parameters must be used, otherwise you will get a linking error.

   ````c++
   int Log(const char* message, int level)//two parameters and int type function
   {
   	std::cout << message << std::endl;
   }
   ````

   `````c++
   void Log(const char* message);//void type function and one parameter 
   `````

   

10. Another linking error:

    Duplicate names

    ````c++
    //one translation unit(.cpp)
    #include <iostream>
    #include "Log.h"//copy the Log function 1st time here
    
    void InitLog()
    {
        Log("Initialized Log");// log is referenced here
    }
    ````

    `````c++
    //second translation unit (Log.h)
    #include <iostream>
    
    void Log(const char* message)
    {
    
        std::cout << "Mnghan is inside log" << std::endl;
    }
    `````

    `````c++
    //main translation unit (.cpp)
    #include <iostream>
    #include "Log.h" //copy the log function the second time therefore linking error! Which Log function should the linker look //for.
    
    int Multiply(int a, int b)
    {
        Log("Multiply");//Log is referenced here
        return a * b;
    }
    
    int main()
    {
        std::cout<< Multiply(5, 8) <<std::endl;
        
        std::cin.get();
    }
    
    `````

11. How to fix the problem in 10, we have several **options:**

    a. mark the **Log function** in `.h` file as **static**, meaning **linking internally in other two translation units (.cpp files)**. In other words, when the `.h` **Log** function is included in **other two translation units**, only linked internally, or **only visible** to that particular translation unit! **They have their own version Log functions and not visible to other object files.**

    `````c++
    //second translation unit (.h)
    #include <iostream>
    
    static void Log(const char* message)//mark as static
    {
    
        std::cout << "Mnghan is inside log" << std::endl;
    }
    `````

    b. to make the log **inline**, meaning only take the actual body of the **Log** function defined in the `.h` file, and replace the **reference to the Log** with the body.

    `````c++
    //second translation unit (.h)
    #include <iostream>
    
    inline void Log(const char* message)//mark as static
    {
    
        std::cout << "Mnghan is inside log" << std::endl;
    }
    `````

    c. This seems the most common way of using `.h` file to avoid **duplication errors:** move the definition of **Log function** into one of these translation units. Because we want to prevent including the **definition of the Log function** multiple times in different translation units. **The rule of thumb:**

    **only put declarations in .h file (note declarations can be included multiple times in different translation units), put their corresponding definitions into another .cpp file**. Then you can reference those defined functions inside your `main()`  which ensures the each function is defined only once!

    `````c++
    // main function 1st translation unit (.cpp file)
    #include <iostream>
    #include "Log.h"
    
    
    /*void Log(const char* message);//declaration for compiler to see the 
                                  //"Log" function actually exists. Note
                                  //this is ONLY put here to let compiler 
                                  //believe that there exists a function called
                                  //"Log", the fact whether or not there really
                                  //exists a "Log" function is unknown, and this
                                  //will be left to linker to check it out! In short,
                                  //we can cheat compiler!*/
    
    
    int Multiply(int a, int b)
    {
        Log("Multiply");//here we reference the "Log" function, and linker will go find the 
                        //definition of the "Log" function
        return a * b;
    }
    
    int main()
    {
        std::cout<< Multiply(5, 8) <<std::endl;
        InitLog();
        
    
        
        std::cin.get();
    }
    
    `````

     `````c++
    //definition file 2nd translation unit (.cpp file)
    #include <iostream>
    #include "Log.h"
    
    
    void InitLog()
    {
        Log("Initialized Log");
    }
    
    
    void Log(const char* message)//definitions put in .cpp file
    {
    
        std::cout << message << std::endl;
    }
     `````

    `````c++
    //declaration file 3rd translation unit (.h file)
    #pragma once
    
    #include <iostream>
    
    void Log(const char* message);//just leave the declaration in the .h file
    void InitLog();
    `````

    

    

