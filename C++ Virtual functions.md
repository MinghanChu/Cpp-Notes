# c++ Virtual functions

### Introduction

1. **Virtual functions** allows overriding methods in subclasses.
2. If **Base class: A** and **Subclass: B**. A method created in **class A** is marked as **virtual**, overriding that method in **class B** to get it do a different task is allowed!
3. When dealing with the concept of **polymorphism**:  if referring to the **subclass** as if it was not **base class**, this will run into problems! In that case, we need **virtual functions**.
4. **Virtual functions** introduce **dynamic dispatch** which compiles and implements in **vtable**.
5. **vtable** contains the **mapping** for all the **virtual functions** in the **base class**. So it can map them to the correct **overriden** function at **runtime**.
6. **vtable** requires additional memory to be stored in to **dispatch** to the **correct function**. It includes a **member pointer** in the base class pointing to the **vtable**. And the program will always go to that **vtable** to check which function should be mapped to every time when the **virtual function** is called. This is the cost! But very small impact, so use **virtual functions!**

`````c++

#include <iostream>
#include <string>

class Base
{
    public:
        virtual std::string GetName() {return "Base class";}
        //std::string GetName() {return "Base class";}
};

class Subclass : public Base
{
    private:
        std::string m_Name;//member variable

    public:
        Subclass(const std::string& name)//name is just for being getting information for the 
            : m_Name(name) {}            //member variable m_Name

        std::string GetName() {return m_Name;}//Note always manipulate member
                                              //variables
};

void checkName(Base* v_b)//instantiating object of Base which is also the subclass (subclass is base!)
{
    std::cout << v_b->GetName() << std::endl;
}

int main()
{
    Base* b = new Base();//constructor
    checkName(b);//passing instance as a parameter into void checkName function

    std::cout << b->GetName() <<std::endl;//accessing its member function
    Subclass* s = new Subclass("Subclass");//parameter
    checkName(s);
    std::cout << s->GetName() << std::endl;//accessing its member function


    Base* base = s;// assigning the pointer of subclass to the base pointer, 
                   //or referring my subclass as the base class
    std::cout << base->GetName()<<std::endl;//will print "Baseclass" 
                                            //we want to print "Subclass" because subclass
                                            //is part of the Baseclass, and compiler should
                                            //identify which is which! In this case we need 
                                            //virtual.
    std::cin.get();
  //*****output*****//
  /*Base class
		Base class
		Subclass
		Subclass
		Subclass*/
}

`````

