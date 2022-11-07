# C++ copy and copy constructor

### Introduction

1. copy data, memory, primitive data from one place to another. Two copy of them are created. When you set `a=b` , you are **always copying** the value b to a. Two **separate variables** with **two different memory addresses**.

   `````c++
   //Example1
   
   int a = 5;//primitive data int a is declared at one memory address
   int b = a;//int b is declared at another (different) memory address
   
   b = 7;//a still remains at 5!
   
   `````

   

2. **Copying takes time!** So we need avoid unnecessary copying when possible. 

3. To know when copying is needed and when copying should be avoided.

4. **Shallow copy (direct copy)** will copy both the pointer pointing to the memory address and the memory, which generates errors sometime when two variables (supposed to be two separate variables) end up with the **same** pointer pointing at the same memory address for them. In that case, we need **Deep copy** to make the copied variable have its **own unique memory address** without touching the first variable.

5. We use **copy constructor** to achieve step 4. We **do not** want to copy the **pointer** instead we want to copy the **memory** at the **pointer**.

6. A **copy constructor** is dedicated to being called for the copied variable, assigning the first created variable to the copied variable with the **same** type for them!

7. `c++` by default provides the **copy constructor**. See the definition:

   `````c++
   class class_name
   {
     private
       char* variable1;
     	unsigned int variable2;
     class_name(const class_name& other ) //copy constructor takes the same name as the class name
     : variable1(other.variable1), variable2(other.variable2)//copy the shallow copy of other into these
       								//member variables see better explanation in the following example
   }
   `````

8. If we do not want to allow copy, we do

   `````c++
   class_name(const class_name& other )=delete;
   `````

   

9. **Always** pass your **object** by `const` `&`(reference)! Otherwise you will every time execute the **copy constructor**, allocating the memory on the heap, copy to that memory, and eventually free it up through `delete`. That is waste of time! So always use **passing by const &**, `const` because it avoids changing the original data. See the detailed example that covers all above statements:

   `````c++
   #include <iostream>
   #include <vector>
   #include <string>
   
   class String//string is made up of an array of characters!
   {
       private:
           char* m_Buffer;//for char type each character occupies 1 byte memory
           unsigned int m_Size;
       public:
           String(const char* string)//constructor
           {
               m_Size = strlen(string);//get the size of string
               m_Buffer = new char[m_Size+1];//declare the size of char
   
               //for (int i = 0; i < m_Size; i++)//Equivalent to memcpy below
                 //  m_Buffer[i] = string[i];
   
                   memcpy(m_Buffer, string, m_Size+1);
                   m_Buffer[m_Size] = 0;//manually add termination
           }
   
           //copy constructor
           String(const String& other)//"other" means the other same type variable we want to copy from
               : m_Size(other.m_Size)//shallow copy the other m_Size varaible to THIS m_Size 
           {
               std::cout<<"copied inside copy constructor!"<<std::endl;
               m_Buffer = new char[m_Size+1];
               memcpy(m_Buffer, other.m_Buffer,m_Size+1);//memcpy(destination, source, size)
           }
           ~String()//destructor
           {
               delete[] m_Buffer;//delete allocated memory address through new
           }
   
           char& operator[](unsigned int index)//It acts as a member function of this class
           {
               return m_Buffer[index];
           }
   
           friend std::ostream& operator<<(std::ostream& stream, const String& string_output);//declaration copied here to access
   };
   
   std::ostream& operator<<(std::ostream& stream, const String& string_output)//Instantiate object string_output
   {
       stream << string_output.m_Buffer;
       return stream;
   }
   
   void PrintName(const String& string_output)
   {
       //String copyname = string_output;//In case we want to copy. Note copy is allowed unless 
                                       //you do not modify the original data!
       std::cout << string_output << std::endl;
   }
   
   int main()
   {
       //String name = "Minghan pays tribute to Shusheng";//An alternative way 
       String name("Minghan pays tribute to Jiaxian");//constructor automatically being called 
                                                     //while the object "name" of the "String" 
                                                     //class is created. Allowing the class to 
                                                     //initializing variables or allocate storage
       //name.String("Test here");//Note constructor cannot be accessed through "." operator!
       
       String copyname = name;//copy the variables of "char* m_Buffer" and "unsigned int m_Size" to
                              //the "copyname string variable" However, the memory address of "char* m_Buffer"
                              //is same! for two string variables. Therefore generate a problem at "}" (end
                              //of the scope) when destructor is called, and end up with deleting
                              //the "m_Buffer" memory twice where the memory has already been freed!
                              //you cannot free it again!
       
       copyname[24] = 'C'; //"[]" operator is not originall defined in the "String" type, therefore 
     											//need to be defined.
       
       PrintName(name);
       PrintName(copyname);
       //std::cout << name << std::endl;
       //std::cout << copyname << std::endl;//copy the memory address
   
      std::cin.get();
     //********Output********//
   /*copied inside copy constructor!
   Minghan pays tribute to Jiaxian
   Minghan pays tribute to Ciaxian*/
   }
   
   `````

   

10. From the above example: 

* **object** can pass variables declared in the corresponding class as parameters to the member functions, including constructors, in that class. E.g. `name("blablabla")`because `char* m_Buffer` is declared as one of the variables in `String` class.
* `copyname[24]` directly manipulates one of declared `char*`variable because `operator[]` was defined so (check it in the code). Therefore `copyname[24]` is just like `copyname.m_function()`, while in our case `operator []` is equivalent to `.m_function`(check the explanation in code). How about the `int m_Size`? Ans: `int m_Size` is the result of `char m_Buffer`, therefore is indirectly affected. 