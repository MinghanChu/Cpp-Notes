# C++ Visibility

### Introduction

1. Visibility deals with how visible of **certain** members and methods of a class actually are. It prevents users to call certain functions that they should not be calling!
2. Basically **visible** means: who can see, who can call, who can use.
3. **Important**: visibility has **no effect** on how your program runs and **no effect** on the performance of your program.

4. **Visibility** exists in c++ to help programmer **write better code** or organize your code. 

5. **Three** visibility modifiers in c++: **private, protected, and public.**

6. The **default visibility modifier** for a **class** will be **private**. Compared to **struct**, the default visibility modifier is **public**.
7. Why don't we make everything public? *Ans:* because it is a **BAD style** for people to **maintain, understand, and maybe extend** the code. **In short, it help not only ourselves but other people.** 
8. For example, **marked as private** informs both you and others that please do not try to modify that block of code from **other classes**. You should only be **accessing it internally** inside this class.
9. In other words, we are only allowed to touch only the **public stuff** which is the **proper usage** of this class, e.g. just call public functions.
10. **Private members:**

````c++
//for private only whatever is inside the class scope can access, meaning read and write. Unless declared as a friend.

#include <iostream>
#include <string>

class Entity
{
  private:
  	int X, Y;
  	void Print(){}
  public:
  	Entity()//constructor is created to initialize the X private member variable
    {
      X = 0;//access within the class no problem
      Print(); //access print() function within the class no problem
    }
};

class Player : public Entity
{
 	public:
  	Player()
    {
      //X = 2;//note inside the subclass "Player" you still CANNOT access the private member variable 		     						   //defined in the base class
      //Print();// NO you CANNOT do that with private methods either
    }
};

int main()
{
  Entity e;
  //e.X = 2;//CANNOT do that because X is a private variable
  
  std::cin.get();
}

````



8. **Protected member:**

   `````c++
   //for private only whatever is inside the class scope can access, meaning read and write. Unless declared as a friend.
   
   #include <iostream>
   #include <string>
   
   class Entity
   {
     protected:
     	int X, Y;
     	void Print(){}
     public:
     	Entity()//constructor is created to initialize the X private member variable
       {
         X = 0;//access within the class no problem
         Print(); //access print() function within the class no problem
       }
   };
   
   class Player : public Entity
   {
    	public:
     	Player()
       {
         X = 2;//subclass can access the members of the base class if declared as "protected" 		     			
         Print();// same as protected methods
       }
   };
   
   int main()
   {
     Entity e;
     //e.X = 2;//Still not visible to main() even though defined as protected
     
     std::cin.get();
   }
   
   
   `````

   9. **Public members:** you can access for all: inside main, inside subclass and base class.

