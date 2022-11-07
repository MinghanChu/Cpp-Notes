# C++ Inheritance

### Introduction

1. Inheritance enables **a hierarchy of classes** which relates to each other.
2.  It allows **a base class** which contains common functionality, and **subclasses** which branch off the base.
3. Inheritance is useful because it avoid **code duplication**. That is blocks of code that contain basically the same thing maybe with slightly difference.
4. Therefore, the common functionalities between classes can be put into a **base** class, and create the **subclasses** from the **base** class.
5. The **subclasses** inherits **all the functionalities** in the base class.

5. Anything that is not defined inside `private` of the **base** class can be accessed by its subclass.
6. The subclasses have both the **type of itself and its parent**.
7. **IMPORTANT:** I found **when instantiating a subclass, the parent members will be constructed as well!**

`````c++
#include <iostream>
#include <vector>
#include <string>
#include <array>


class Entity
{
    public:
        float x=0, y=0;//must initialize otherwise will use whatever leftover previouly at these memory addresses
        
        void Move(float xa, float ya)
        {
            x += xa;
            y += ya;
            std::cout << "x= " << x <<" y= "<< y << std::endl;
        }

};

class player : public Entity
{
    public:
    const char* Name;
    void PrintName()
    {
        std::cout << Name << std::endl;
    }
};

int main()
{
    player user;//user is Entity! contains everything in Entity!
    user.Name = "Minghan";
    user.PrintName();
    user.Move(5,6);
    user.x = 2;
    user.y =7;
    user.Move(5,6);
    std::cin.get();
  
  //*******Output*****//
  /*Minghan
x= 5 y= 6
x= 7 y= 13*/
}
`````

