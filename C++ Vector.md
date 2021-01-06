#### C++ dynamic array (std::Vector) *maybe better be called array list*

#### Introduction

1. In `c++`, a **vector** class is defined in the `std` namespace. This implies there are probably a bunch of different classes defined in `std` namespace, and when accessing one of these classes, `::` operator must be used, i.e. `std::vector`. *This is like the way how OpenFoam works: inside the **Foam** namespace are defined numerous different classes, and `::` operator must be used in order to access one*.  

2. **standard template library**: a library filled with containers that contain data. The whole library is templated. The data type that containers contains is determined by the programmer.
3. We are going to rewrite the class structures that exist in `c++` in order to **optimize** them and make them **faster than** the ones in the **standard template library**. For example,==In OpenFoam, `Info` is used to replace `cout`. More importantly OpenFoam provides its own vector.h! Where eigenvalue and eigenvector can be calculated.==
4. A **set size** is not given when a **vector or dynamic array** is created. You can give it a set size, but basically a set size is not given.
5. Dynamic array wants to let users keep giving elements to the created array (no enforced limitations) and the **size grows**! Yes you are having an array that **resizes**! How does this work? The answer is when you are pushing more elements exceeding the size of vector, **it basically creates a new array in memory** that is bigger (having more storage) than the first one, then copy everything to there, and delete the other one!
6. When deciding whether to use **vector objects** or **vector pointers on heap**, correspondingly  `Vector<type>` or `Vector<type*>` . ==Can this help explain why in OpenFoam new-operated defined variable on heap is defined pointers and therefore were not outputted?== It is optimal to store **vector objects** instead of **vector pointers**. 
7. Storing **vector objects** give `inline` memory. Dynamic arrays or vector give **contiguous memory in a roll** in contrary to fragmented in memory. So for example: inline memory gives `x y z x y z x y z` in a roll one after the other. This **is optimal** if you want to manipulate them: read, change, set, etc.

8. Storing **vector pointers**, the actual memory stay intact, and pointers is held to that allocated memory. When resize happens memory address (integers) will be copied to the actual data stored in various places in memory.
9. **In summary objects concentrate on storage and pointer concentrates on memory address.**

`````c++
#include <iostream>
#include <vector>
#include <string>


using namespace std;

struct Vertex
{
    float x, y, z;
};

std::ostream& operator<<(std::ostream& stream, const Vertex& vertex)
{
    stream << vertex.x <<","<< vertex.y<<","<<vertex.z;
    return stream;
}

void ShowArray(std::vector<Vertex>& ArrayRef)//always passing by &. If no changes to the array is allowed, 																								//then const std::vector<Vertex>& ArrayRef
{
    int plus = 1;
    for (int i=0; i < ArrayRef.size(); i++ )
      {
        ArrayRef[i] = {4,4,static_cast<float>(i+plus)};// why float? because the class type is float
        std::cout<< "At i= "<< i << " Array element is: "<<ArrayRef[i] << std::endl;
      }  
      

}

int main()
{

    //static array with fixed number of elements therefore we cannot enter
    //as many elements as we want! See the examples below
    //Vertex vertices[5];
    //Vertex* vertices = new Vertex[5];

    //Therefore we use dynamic array or vector
    std::vector<Vertex> vertices; //vector object, size = 0 
    vertices.push_back({1, 2, 3});//size = 1 (1 array element could contain many entries)
    vertices.push_back({4, 5, 6});//size = 2
    ShowArray(vertices);

    for (int i = 0; i < vertices.size(); i++)//size = 2 meaning two arrays
    std::cout << vertices[i] << std::endl;// for i = 1 array = {1,2,3} 
                                          // for i = 2 array = {4,5,6}

     for (Vertex& v : vertices)//cleaner way to represent the above for loop
     std::cout << v << std::endl;  //ampersand ensures not copying the data but
                                   //passing the memory address      

     vertices.erase(vertices.begin()+1);// just erase 2nd (index 1) 
                                        //array element (beginning +1) 
     vertices.clear(); //clear the vertices: size changes from 2 to 0   
                                                          
    std::cin.get();
  //**********Output********//
  /*
  At i= 0 Array element is: 4,4,1
	At i= 1 Array element is: 4,4,2
	4,4,1
	4,4,2
	4,4,1
	4,4,2*/
}
`````





10. **Optimizing to avoid copying using vector** 

