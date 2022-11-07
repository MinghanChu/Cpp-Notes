# C++ Pointer and reference

### Introduction

#### Pointer

1. All the code written telling the computer what to do get loaded into **memory** when an application is being launched 
2. **Pointer** is a beautiful tool to manage and manipulate that memory

3. A pointer (memory address) is an **integer (number)** that stores a memory address
4. A pointer has nothing to do with types! In other words, types do not change what the pointer is! which is just a **memory address**

#### Reference

3. There is nothing you can do with references that you cannot do with pointer!

4. Reference is **NOT** a real variable, it is a reference. When a reference is declared, it needs to be **immediately assigned** to a variable!

5. Unlike pointer that a **new pointer variable** can be created and then set it to a null pointer or a memory address of a created variable, you cannot do that with reference. Because reference has to **reference an already existing variable**. See example below for distinguishing between them (**==note pointer UNLIKE references can be repeatedly used to store multiple memory 	addresses==**).

6. References **are not typical variables** that occupy memory. They are actually **aliases** to the variables being referenced! So you cannot change what is originally being referenced. See the example (This is different from what pointer does which can be created to manipulate multiple variables):

   `````c++
   int& ref; //wrong
   int& ref = a; //correct
   `````

   `````c++
   int a = 5;
   int b = 8;
   
   int& ref = a;
   ref = b;//to assign the value of b to a (ref:alias of a) instead of referencing b
   `````

   



`````c++
//passing by pointer
#include <iostream>
#include <vector>
#include <cstring>
#define LOG(x) std::cout << x << std::endl

void Increment(int* value)
{
    (*value)++;//dereference first and increase the value stored at this memory address, otherwise you will increase the pointer first and dereference 
}

int main()
{
 
    int a = 5;
    Increment(&a);
    LOG(a);
    std::cin.get();
   
}
`````



`````c++
//passing by reference
#include <iostream>
#include <vector>
#include <cstring>
#define LOG(x) std::cout << x << std::endl

void Increment(int& value)
{
    value++;//whenever you can do with reference then do it because reference makes your code cleaner
}

int main()
{
 
    int a = 5;
    Increment(a);
    LOG(a);
    std::cin.get();
   
}
`````



`````c++
//difference between pointer and reference
#include <iostream>
#include <vector>
#include <cstring>
#define LOG(x) std::cout << x << std::endl

int main()
{
	int a = 5;
  int b = 8;
  
  int* ref = &a;//pointer varaible "ref" storing the memory address of variable a
  							//**note pointer UNLIKE references can be repeatedly used to store multiple memory 	addresses**
	*ref = 2;//assign a new value of 2 to the memory address of a
	ref = &b;//assign the memory address of variable b (note repeatedly used) to the pointer ref
	*ref = 1;//deference ref to change the value of b from 8 to 1
  
  LOG(a);//print 2
  LOG(b);//print 1


}

`````

