# C++ functions

#### **Introduction**

1. Function is a block of code written to perform tasks. In `class` functions are called **Methods**.
2. Functions are **primarily** used to prevent code duplication instead of writing the same code multiple times.
3. Functions can take input and output a type value.
4. Traditionally a function can only return **one** type! It can specifically return one variable. But we `c++` can do this!
5. In `c++` we do not have **nested functions** but we can call a function inside another function.

#### **function pointer**

6. function pointer can assign a function to a variable, or pass a function to other functions as a parameter. Example:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   void HelloMinghan()
   {
       std::cout << "Hello Minghan Never give up" << std::endl;
   }
   
   int main()
   {
      
       void(*function)() = HelloMinghan;//assign function "HelloMinghan" to "function" pointer variable
       
   
       function();//print "Hello Minghan Never give up"
       function();//print "Hello Minghan Never give up"
       HelloMinghan();//print "Hello Minghan Never give up"
       HelloMinghan();//print "Hello Minghan Never give up"
   
   
   std::cin.get();
   
   }
   `````

7. An alternative cleaner way of step 6:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   void HelloMinghan()
   {
       std::cout << "Hello Minghan Never give up" << std::endl;
   }
   
   int main()
   {
       typedef void(*helloMinghanFunction)();//create an alias for a void function named "helloMinghanFunction" through deferencing the function pointer of "helloMinghanFunction"
     	  helloMinghanFunction test = HelloMinghan;// assign function "HelloMinghan" to a variable called "test" preceded with the declared function type
       
       	test();//print "Hello Minghan Never give up"
   
   std::cin.get();
   
   }
   `````

8. Add parameter to function:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   void HelloMinghan(int x)
   {
       std::cout << "Hello Minghan Never give up for "<< x << std::endl;
   }
   int Multiply(int a, int b)
   {
        return a*b;
   }
   
   int main()
   {
       typedef void(*helloMinghanFunction)(int);//compared to step 7, a int parameter is included
       void(*function)(int) = HelloMinghan;//compared to step 7, a int parameter is included
       
       helloMinghanFunction test = HelloMinghan;
       test(4);
       function(44);
       function(444);
       HelloMinghan(4);
       HelloMinghan(4444);
   //******Output****************
   /*Hello Minghan Never give up for 4
   Hello Minghan Never give up for 44
   Hello Minghan Never give up for 444
   Hello Minghan Never give up for 4
   Hello Minghan Never give up for 4444
   std::cin.get();*/
   
   
   }
   `````

9. **An useful example using function pointer:**

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   void PrintValue(int value)
   {
       std::cout << "Vector integer value :"<< value << std::endl;
   }
   
   void ForEach(const std::vector<int>& values, void(*func)(int))
   {
       for (int val : values)
           func(val);
   }
   
   int main()
   {
       std::vector<int> values = {1, 2, 3, 4, 5, 6};
       //ForEach means: ForEach function into which we can pass a vector of integers and each of these integers
       //inside the vector the "PrintValue" function will be executed
       ForEach(values, PrintValue);//passing the function "PrintVlaue" to ForEach function as a parameter
     
   std::cin.get(); 
     
   //******Output****************
   /*
   Vector integer value :1
   Vector integer value :2
   Vector integer value :3
   Vector integer value :4
   Vector integer value :5
   Vector integer value :6*/
   }
   
   
   `````

   