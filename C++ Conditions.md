# C++ Conditions and branches

### Introduction

1. When/why do we need **conditions/branching**?

   In certain times we want to evaluate **a certain condition** and decide what code we want to execute as **the result of that evaluation.**

2. If the **evaluation** is **true,** jump to a part of the source code, if **false** jump to another part of the source code. (or you can say **branching** to the right section of the **CPU instructions**.)
3. An application is running along with its **modules**, and all get loaded into **memory**.

4. Basically a **condition** tells computer to jump to a **certain part of memory** and start executing from there.
5. A lot **optimized code** will try to avoid **if statement/branching**. Because it slows the program!

6. ```c++
   x == 5 //"==" operator will return true or false in c++
   
     //"==" is the operator that is overloaded into c STANDARD LIBRARY
     //meaning "==" will compare two already exisiting integers, if they are equal return true
   ```

7. **Boolean** is basically integer: **0** for **false** and **1 or other integers** for **true**. 

8. So `if(0)` the **if statement** will Not be performed, otherwise `if(1 or 2..)`, the **if statement** will be performed.

   ```c++
   if(1)//equivalent to if(true)
   {
   	std::cout<<Inside if statement<<std::endl;
   }
   ```

9. **if statement** with **pointer**

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <array>
   
   
   static void Log(const std::string& greetings) 
   {
       std::cout<<greetings<<std::endl;
   }
   
   int main()
   {
       const char* ptr = "Minghan";//here the pointer ptr is not equal to zero or nullptr
                                   //char* makes string "Minghan" a memory address. It is like
                                   //int* ptr = &variable where &variable is a memory address
       int x = 5;
   
       /* Note both if statements are executed if both of them are true statements
       To avoid printing two true statements you must use "else if" instead. In that
       case, only the first true statement will be executed, even if the "else if" statement is 
       also true */
       if (ptr)
           Log(ptr);//& requires memory address therefore pointer is used
       else if (ptr == "Minghan")//"Minghan" here is a memory address and will not be printed out!
           Log("Ptr is minghan!");  
       if (ptr == "Minghan")//"Minghan" here is a memory address
           Log("Ptr is minghan!");
       else 
           Log("Ptr is null!");
   
       bool comparisonResult = x == 5;//x==5 returns a true or false
       if(comparisonResult)
       {
           Log("Hello World");//jump to the memory of Log function and execute 
       }
       std::cin.get();
   
   }
   ```

   

10. **else if** is two statements!!, **not** a **key word combination** which consists of **else** and **if**. Check this out:

    ```c++
    else if (condition)
    	std::cout<<Hello World!<<std::endl;
    	
    	//which is equivalent to 
    	
    	else 
    	{
    		if (condition)
    			std::cout<<Hello World!<<std::endl;
    	}
    ```

    