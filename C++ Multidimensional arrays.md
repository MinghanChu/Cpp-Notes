# C++ Multidimensional arrays

### Introduction

1. Again array is a pointer! Pointer to the **beginning** of an array.

2. **Multidimensional arrays** is an array of arrays, e.g. a 2D array is an array of arrays, and a 3D array is an array of array of arrays, or pointers of arrays.

3. Basically a 2D array is **a collection of arrays**.

4. `int**` is a pointer to an **integer pointer**, **not** to **an actual integer**.

5. Example: `int** a2d = new int*[50]`, here `int*[50]` indicates allocating room for 50 **int pointers!**. Note not actual integers. More explanation:

   `````c++
   int* array = new int[50]; //only two things to consider: 1. a pointer is returned, and 2. 4 bytes for integer type 
   													//here 50 * 4 bytes = 200 bytes in total
   													//Therefore no integers!! int is only the representative of size!!
   int** a2d = new int*[50];  //50 pointers returned to the int pointers of arrays
   int*** a3d = new int**[50]; //50 pointers returned to the pointers of the int pointers of arrays
   `````

6. To deal with Multidimensional arrays, we must loop through **50** times of **a2d** array (50 pointers or memory locations of int pointers of arrays)

   ````c++
   
   int** a2d = new int*[50];//corresponding to outer most index, e.g. a2d[x][y] then x is the outer most index ()
                             //in other words must be limited to 5
                             //and the inner most y is a free index
   
   
   for (int i = 0; i < 50; i++)
   {
     a2d[i] = new int[50]; //i = 50 pointers (outer arrays) to the int pointers of arrays with [50] elements
   }
   
   
   
   
   //**************************************************************************************//
   //***********Just take 3d array as a good example to help understanding*****************//
   //**************************************************************************************//
   int*** a3d = new int**[50];//a p
   for (int i = 0; i < 50; i++)
   {
     a3d[i] = new int*[50];
    //*************************
     for (int j = 0; j < 50; j++)
     {
       a3d[i][j] = new int[50];// a3d[i] means dereferencing the first part and then a3d[i][j] dereferencing the 2nd part
     }
     //*************************
   }
     
   //equivalent to
   ....
     a3d[i] = new int*[50];//a pointer to a pointer to a pointer
     for (int j = 0; j < 50; j++)
     {
    //*************************
       int** ptr = a3d[i];
       ptr[j] = new int[50];//allocating actual array of integers
    //*************************
     }
   ````

7. Deleting Multidimensional arrays

   `````c++
   //for 2d array
   int** a2d = new int*[50];
   for (int i = 0; i < 50; i++)
   {
     a2d[i] = new int[50]; //50 pointers to the int pointers of arrays with [50] elements
   }
   
   for (int i = 0; i < 50; i++)//start from deleting the outer most
   	delete[] a2d[i];
   delete[] a2d; //detete last 1D array
   `````

8. **In summary, when dealing with multidimensional arrays, some useful rules apply:**

   + `int^n a2d = new int^(n-1)[50];`this declares the **outermost** (leftmost in terms of "[]" operator) pointers, and **`n`** represents the **number of "*"**

   + `````c++
     for (int i = 0; i < 50; i++)
     {
       a2d[i] = new int^n-2[50]; //pointers are assigned starting from the outermost to the inner ones (except the most inner array!) and n-2 represents the number of "*". Note that "new" operator already accounts for one "*", therefore the total number of sets for pointers: (n-2)+1 which also stands for the outer most pointers. For example, a 2D array n=2, then n-2=2-2=0, with only the "new" operator left, representing the total number of sets (as well as the outer most) pointers is 1. One step further from that is the most inner array.
     }
     `````

     

`````c++
#include <iostream>
#include <vector>
#include <string>
#include <array>



int main()
{

 
                             
int** a = new int[4];
   /* int** a2d = new int*[50];  //allocating 200 bytes of memory, i.e. the size of int is 4 bytes
    for (int i = 0; i < 50; i++)
    {
        a2d[i] = new int[50];//essentially allocated 50 arrays, each memory address of these arrays is stored 
                            //inside the a2d array
         delete[] a2d[i];//delete integer
    }
    delete[] a2d;//delete pointer 
    a2d[0][0] = 0; //the first 0 is index (pointer) and the second 0 is my actual first integer
    a2d[0][1] = 0; //my second integer
    a2d[0][2] = 0; //my third integer*/
    
    
    
    //**********Example of an equivalent way of representing a 2D array through a 1D array************//
    
   int** a2d = new int*[5];
   for (int i = 0; i < 5; i++)
   {
       a2d[i] = new int[5];
   }
   
    for (int y = 0; y < 5; y++)
    {
        for (int x = 0; x < 5; x++)//inner loop is my actual integer arrays
        {
            a2d[x][y] = 2;
            std::cout <<"x= "<< x <<" y= "<< y <<" a2d= "<<a2d[x][y]<<std::endl;
        }
        
    }

    int* array = new int[5 * 5];
    for (int y = 0; y < 5; y++)
    {
        for (int x = 0; x < 5; x++)//inner loop is my actual integer arrays
        {
            array[x + y * 5] = 2;//every time the y increments it jumps 5 elements forward which is equivalent to 
                                 //dropping 1 row down
        
            //std::cout <<"x= "<< x <<" y= "<< y <<" array= "<<array[x]<<std::endl;
        }
        int count = sizeof(array);
       
    }
    std::cin.get();

}
`````

