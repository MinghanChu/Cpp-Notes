# C++ standard static array (std::array)

### Introduction

1. Like standard **vector (std::vector) class** , **standard array class (std::array)** is part of the **standard template library** provided in `c++`.
2. `c++` provides with the **standard array class** to deal with **static arrays!**. Therefore, **Standard array is also called static array.**
3. **Static arrays do NOT grow!** That is what **static** means.
4. The **size (how many elements) and type of elements** must be defined as soon as a **static array** is created. As a result, you **cannot** change the size! (consider to use static array for implementing the **barycentric triangle** in OpenFoam).
5. **Both standard static array and normal array** are stored on **the stack**! This is different from **standard vector class** which creates and stores it data on the **Heap.**

6. **Normal arrays** will not do the **bounds checking**, while **static arrays** will do! See the example:

   `````c++
   int main()
   {
   	std::array<int, 5> array;
     
     array[6] = 2//will stop you! Because you are out of bound.
       
       int array[5];
     	array[5] = 2 //won't do anything just override data that not belong to you
   }
   `````

7. Another thing is the `std::array<int, size>` where **size** is just a **template argument** which is not stored on memory. The `size` function basically returns `5 (the template argument)` not a **size variable** stored on stack or heap (meaning only returned **variables** occupy memory). 

   *maybe can be understood as: there is an`argument` type data that does not occupy memory. Note when an **object** of a class is instantiated, inside the `int main()` it occupies on **stack memory**, it also can also be passed to a defined function as an **argument.*** See the example below: 

`````c++
#include <iostream>
#include <vector>
#include <string>
#include <array>

void PrintArray(const std::array<int, 5>& array)
{
    for (int i = 0; i < array.size(); i++)
        std::cout <<"array["<<i<<"] = " << array[i]<<std::endl;
}


int main()
{
    std::array<int, 5> data;//Note standard array class (in the standard template library)
                            //is instantiated here. Since it is a class, it enables access
                            //to it member functions! Such as "size()" asking for the how
                            //how many elements in sitting in the array!
    //data.size();  //size() is the member function of standarad array class 
    
    for (int i = 0; i < data.size(); i++)
        data[i] = i+1;                     

    PrintArray(data);

   std::cin.get();
  //*****Output*********//
 /* array[0] = 1
		array[1] = 2
		array[2] = 3
		array[3] = 4
		array[4] = 5*/
}
`````

