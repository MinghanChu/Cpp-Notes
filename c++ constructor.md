# C++ constructor

#### Introduction

1. Constructor is a special type of method (function) which runs every time we **instantiate** an object. **It will not run if you do not create an instance of a class. **The **main purpose** of constructor: to initialize your class to ensure all memory are properly initialized.
2. Constructor is a special member function and **cannot be accessed by** `.` like those regular member functions do.
3. A constructor is automatically called **whenever a new object of this class** is created, allowing the class to **initialize member variables or allocate storage**.
4. you can have **more than one** constructor created.
5. ``c++``  provides **default** constructor ``constructor_name{}`` , however unlike ``java`` , which automatically initializes `int`  and `float` variables by setting to zero, ``c++`` does not do that! Meaning you will **use whatever leftover previously in the currently occupied memory address.**
6. From `3`, we need to create **constructor(s)** with variables properly initialized
7. Also other types of constructor: copy constructor and move constructor
8. ***weicheng, why does not Typora provide colouring functionality for users to change text colour? without always sticking with black and white format***



***example 1***

`````c++
class Entity
{
  public:
  	float X, Y;//member variables declared
  
  //Defualt constructor looks like
  Entity()
  {}
  
  
  //constructor1
  Entity()//constructor must have the same name as the class (
  {
    X = 0.0f;//initializing variable X and Y inside constructor and set to zeros
    Y = 0.0f;
  }
  
  //parameterized constructor2
  Entity(float x, float y)//here x and y are "parameter" (be passed from a different scope(s))
  {
    X = x;//Note this means I am assigning parameter x and y to member variable X and Y
    Y = y;
    
  }
  void Print()
  {
    std::cout << X << "," << Y << std::endl;
  }
};

int main()
{
  Entity E;
  Entity e(10.0f, 5.0f); //instantiate your class by creating an instance/object "e".							 
  std::cout << e.X <<std::endl;//will print 10
  std::cout << E.X << std::endl;//will print 0	
  e.Print();//will print 10,5
  E.Print();//will print 0,0
  
  //Summary: once different constructors instantiated, they function accordingly based on their own dinstinct definition when being called!
  std::cin.get();
  
}
`````



**example 2**

`````c++
//note Static methods can be called without a class instance. Inside a static method you cannot write code to refer to class instance. Or a static method does NOT have a class instance. Static variable has the global functionality (same as namespace which is global)
class Log
{
  private:
  	Log(){}//prevent creating an instances from people
  public:
  	static void Write()
    {
      
    }
};

int main()
{
  //Because static methods do not have a class instance, always do not want people to create an instance, and only want people to use Write() this way, like accessing a namespace of "Log". Therefore put Log(){} inside private
  Log::Write();
   
}
`````



**example3 - continued from example2**

`````c++
//An alternative way to prevent creating an instance

class Log
{
  //private:
  	//Log(){}//prevent creating an instances from people
  public:
  	Log() = delete; //since c++ supplies a default constructor and therefore we just need to delete it
  	
  	static void Write()
    {
      
    }
};

int main()
{
  Log::Write();
  //any instantiation like: Log try; is no longer allowed 
}
`````



