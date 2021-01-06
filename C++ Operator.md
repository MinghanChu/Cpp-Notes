# C++ Operator

### Introduction

1. **Operator overloading:** essentially creating, i.e. means giving a new meaning to, adding parameter to. You are allowed to **define or change** the behaviour of the operator in the program! (Note this feature is not supported in `Java` or partly supported in `C#`).

2. Basically an **operator** is just a **function**. Make expressions look more readable. See the example below:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   
   struct Vector
   {
       float x, y;
   
       Vector(float x, float y)//constructor
           : x(x), y(y) {}
   
       Vector Add(const Vector& other) const//copy constructor
       {
           return Vector(x+other.x, y+other.y);
       }
   
       Vector operator+(const Vector& other) const
       {
           return Add(other);
       }
   
        Vector Multiply(const Vector& other) const//copy constructor
       {
           return Vector(x*other.x, y*other.y);
       }
   
       Vector operator*(const Vector& other) const//copy constructor
       {
           return Multiply(other);
       }
   
       bool operator==(const Vector& other) const
       {
           return x == other.x && y == other.y;
       }
   
         bool operator!=(const Vector& other) const
       {
           return !(*this == other);//*this to the object x and y (initialized values)
       }
   };
   
   std::ostream& operator<<(std::ostream& stream, const Vector& other)
   {
       stream << other.x <<","<<other.y;
       return stream;
   }
   
   int main()
   {
       Vector position(4.0f, 4.0f);//Instantiating an object automatically calls constructor
       Vector speed(0.5f, 1.5f);  //and initializing the member variables which are used as
       Vector powerup(1.1f, 1.1f);//building blocks to create different functionalities
   
    //note "speed" as a constructor can be copied to the allocated constructor memory inside 
    //the constructor scope
       Vector result1 = position.Add(speed.Multiply(powerup));//Note once objects are initialized
     																												 //parameters need not be shown
       Vector result2 = position + speed * powerup;
      
       std::cout <<result2<<std::endl; //"<<" operator is customized to print array elements otherwise
     		 															//will not print
   
       if(result1 == result2)
       {
           std::cout<<"result1=result2"<<std::endl;
       }
   
       std::cin.get();
   }
   `````

   

*Remark: it seems after an existing operator being customized, it has the combined functionalities consisting of both its intrinsic functionality as well as the newly customized ones. I need to refresh and update on this understanding later.*