# C++ Raw arrays (Pointer)

### Introduction

1. **Pointer** is the base of how arrays work in `c++`.
2. **Arrays** is a collection of elements in a particular order. 
3. In `c++`, an **array** is usually used to represent a **collection of a bunch of variables in a roll**, usually **the same** **type**.

4. Why we need array? Why do we just create whole bunch of variables? Ans: that is very messy and error prone, not only that most of cases unfeasible! **Arrays** can very effectively group those **whole bunch of variables** into a **data set**. 
5. Therefore, in short, **array** is a useful tool to deal with **many variables**. We use an array to contain **many**, for example `int`, variables in one **array variable**.

6. we give a **single name (it is like an object being instantiated)**  to a created array, with that we can refer to all the variables created with the array!

7. **Array** stores data **contiguously or in a roll**, allocating memories one after another. The storage depends on the **data type:** for example `int` occupies `4 bytes` and `char` for `1 byte`. This means if you have an array that contains `5` integers, each integer   occupies `4 bytes`, and therefore `20 bytes` in total. 

8. **Array** is a **pointer** pointing at the beginning of the array!

   `````c++
   int Rij[2]; //declare an int array
   int* ptr = Rij;//compile fine
   `````

9. Again **pointer is an integer** and has **nothing to do with types**, but just affect the **memory (bytes)** taken to contain data. In other words, when **a pointer** is created, the elements can be accessed according to the **bytes assigned** to the specific type **preceding the created pointer**. For example, if `int` type **pointer** is defined, **pointer** will access these elements in `x4 bytes`fashion, while if a`char` **pointer** is defined, then access elements in `x 1 byte` fashion. See the example:

   `````c++
   int Rij[2]; //declare an int array
   int* ptr = Rij;//compile fine memory address of the array(pointer) is copied to ptr
   
   *(ptr+2) = 6;//+2 means 2 location (2x4 bytes) to access the 3rd elements
   
   //we can cast ptr to char type
     *(int*)((char*)ptr + 8) = 6 //Note pointer (ptr) could be any type, only difference is the memory 
       													//taken to access the elements. In this case, char* ptr takes 1 byte
       													//each time to access elements, therefore +8. Since 6 is an integer
       													//holding 4 bytes memory, cast to int* type, and finally dereference to
       													//get integer 6
   
   
   `````

   See the complete example:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <array>
   
   
   
   
   int main()
   {
       //*********Create an array on stack*********//
       int Rij[5];//instantiate an array or declare an array on stack will be
                  //destroyed when go out of the scope at "}"
       int* ptr = Rij;//copy the memory address of Rij to ptr
       for (int i = 0; i<5; i++)//loop to assign values
           Rij[i] = 2;
   
       Rij[2] = 5;// element number 2 means the 3rd element is set to 5
       *(ptr+2) = 6;//Note ptr points at the beginning of the array
                    //+2 means the second element location (accessing by 2x4=8 bytes)
                    //at the 3rd element!
   
       //****To better understand how pointer works******//
       *(int*)((char*)ptr + 8) = 6; //cast a char pointer (1 byte memory),
                                    //then must +8,finally cast back to int type to contain 
                                    //the 4 byte integer 6, then deference to get the integer
       
   
       std::cin.get();
   }
   `````

10. Allocating array on **stack**, there is no a direct way to check its size. But can be somehow determined using the example below:

    `````c++
    #include <iostream>
    #include <vector>
    #include <string>
    #include <array>
    
    
    
    int main()
    {
    
        //*********Create an array on stack*********//
        int Rij[5];//instantiate an array or declare an array on stack will be
                   //destroyed when go out of the scope at "}"
        int count = sizeof(Rij) / sizeof(int); //sizeof() function calculates the memory taken up
                                              //e.g. 4 bytes for int and 5x4 for Rij array
                                              //therefore size = total bytes/bytes for each element
    
                    
        std::cout<<"Int memory = "<<sizeof(int)<<" Count Rij elements = "<<count<<std::endl;
    
    
        std::cin.get();
    }
    `````

    In the above example, the method is not trustworthy. So switch to `std::array` if possible because it has `.size()` to give you the array size directly. You can still use to set size for **raw array** though

    `````c++
    static const int size = 5;//using static operator
    int array[size];
    
    for (int i = 0; i < size; i++)
    	array[i] = 4;
    `````

    

11. When **allocating  memory on heap** using `new` operator, compiler jumps from the **stack** to **heap**, so try to avoid `new` unless needed.

12. **Weird observation:**

    `````c++
      int* RijNew = new int[5];//memory allocated on heap
    
        int count_entity = sizeof(RijNew);//this is 8 bytes Note: weird thing! no matter what type 
                                          //pointer I am using, the size of pinter always gives 8 bytes!!!!
    
    int Rij[5];//instantiate an array or declare an array on stack will be
                   //destroyed when go out of the scope at "}"
        int count = sizeof(Rij) / sizeof(int); //20 bytes / 4 bytes = 5 elements
    `````

    

    **Complete code**

    `````c++
    #include <iostream>
    #include <vector>
    #include <string>
    #include <array>
    
    
    class Entity
    {
    public:
        int* RijNew = new int[5];//memory allocated on heap
    
        int count_entity = sizeof(RijNew);//this is 8 bytes Note: weird thing! no matter what type 
                                          //pointer I am using, the size of pinter always gives 8 bytes!!!!
    
      
    
        Entity()
        {
            for (int i = 0; i < 5; i++)
                RijNew[i] = 4;
                  std::cout <<"RijNew array memory = "<<count_entity<<std::endl;
        }
    };
    
    int main()
    {
    
        //*********Create an array on stack*********//
        int Rij[5];//instantiate an array or declare an array on stack will be
                   //destroyed when go out of the scope at "}"
        int count = sizeof(Rij) / sizeof(int); //sizeof() function calculates the memory taken up
                                              //e.g. 4 bytes for int and 5x4 for Rij array
                                              //therefore size = total bytes/bytes for each element
    
                    
        int* ptr = Rij;//copy the memory address of Rij to ptr
    
        for (int i = 0; i<5; i++)//loop to assign values
            Rij[i] = 2;
    
        Rij[2] = 5;// element number 2 means the 3rd element is set to 5
        *(ptr+2) = 6;//Note ptr points at the beginning of the array
                     //+2 means the second element location (accessing by 2x4=8 bytes)
                     //at the 3rd element!
    
        //****To better understand how pointer works******//
        *(int*)((char*)ptr + 8) = 6; //cast a char pointer (1 byte memory),
                                     //then must +8,finally cast back to int type to contain 
                                     //the 4 byte integer 6, then deference to get the integer
    
        //**********Create an array on heap************//
    
        Entity e;
        
     		std::cout<<"Int memory = "<<sizeof(int);
        std::cout<<" Rij memory = "<<sizeof(Rij)<<" Count Rij elements = "<<count;
        std::cout<<" RijNew memory = "<<sizeof(e.RijNew)<<std::endl;
    
        delete[] e.RijNew;
    
    
        std::cin.get();
    }
    `````

    