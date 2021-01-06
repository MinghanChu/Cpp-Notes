# C++ const

### Introduction

1. **Const** is like a "promise" you give that something is not going to change. But you can **break** that **promise!**

2. When you want to **keep around** a number in your program, use **const**

   ```c++
   const int MAX_AGE = 23;
   
   //MAX_AGE = 50;//you cannot do this because "MAX_AGE" is a const variable
   ```

3. **Case 1:** you cannot modify the **content** of a pointer (memory address):

   ```c++
   const int MAX_AGE = 90;
   
   const int* a = new int; // means the content of the memory address (pointer) of variable a is fixed. But you can change the 
   												//pointer of a to point a different memory address, see following 
   
   //const int* a = new int is equivalent to int const* a = new int int
     
   a = (int*)&MAX_AGE; //pointer of a is pointing to memory address of MAX_AGE now
   
   //*a = 2; // I cannot deferencen the pointer and change the value of a
   
   
   std::cout << *a << std::endl; //I can still read the content of a
   ```

   

4. **Case 2**: you can modify the **content** of a pointer but you **cannot** change the **pointer**:

   ```c++
   const int MAX_AGE = 90;
   
   int* const a = new int; //pointer a is fixed
   
   //a = (int*)&MAX_AGE;//you cannot reassign a different memory address to pointer a
   ```

   

5. so **case 3** would be you **cannot** change both the **content** and the **pointer** 

```c++
const int* const a = new int;
```

6. **Case 4: const** with classes and methods:

   ```c++
   class Entity
   {
   	private:
   		int m_X, m_Y;
   	public:
   		int GetX() const //this means the method cannot modify the class members: m_X and m_Y
   		{
   			//m_X = 2; //you cannot do this
         return m_X;
   		}
     
     //only mark the methods as const if you do NOT want to change the class!!!
     
     	void SetX(int x) //do not mark SetX as const because you want to modify the class member m_X!!
       {
       	m_X = x;
       }
   }
   ```

   * more restrictions:

   ```c++
   class Entity
   {
   	private:
   		int* m_X, *m_Y; //you must stick * next to each variable to make it a pointer
   	public:
   		const int* const GetX() const //means the returned pointer and the content of this pointer cannot be modified
         														//also it promises that the method cannot modify the class members
   		{
         return m_X;//m_X is a pointer so use int*
   		}
     
   ...
   }
   ```

7. Application

   ```c++
   class Entity
   {
   	private:
   		int m_X, m_Y;
   	public:
   		int GetX() const //this means the method cannot modify the class members: m_X and m_Y
   		{
   			//m_X = 2; //you cannot do this
         return m_X;
   		}
     
     //only mark the methods as const if you do NOT want to change the class!!!
     
     	void SetX(int x) //do not mark SetX as const because you want to modify the class member m_X!!
       {
       	m_X = x;
       }
   }
   
   void PrintEntity(const Entity& e)//with references you are the contents
     															 //const Entity will call any const functions, e.g. int GetX() const, if int GetX() is used
     															 //you are not allowed to do so, because no promise that the class Entity will not be changed
   {
     //e = Entity()//you cannot do this now with const
     std::cout <<e.GetX() <<std::endl;
   }
   
   int main()
   {
     Entity e; 
   }
   ```

8. you can mark the variable within the **const** methods as `mutable int var;` if you really want to change that variable within a const method.

