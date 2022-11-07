# C++ New Operator (overloading the new operator if wanted)

### Introduction

1. In `c++` , you need to always care about **memory** , **address**, **performance**, **optimization**, etc.
2. There are so many other alternative languages, like `java`, with which you do not need to care about everything. `c++` cares about everything!
3. **New operator** is very important in `c++`.
4. **The main purpose of new operator is to allocate memory on the heap specifically.**
5. The way of using **new** is: you write **new** together with the **data type** you want. For example, `int* a = new int ` , the **data type** could also be **char** and **class**. The reason to declare the **data type** with **new** is to determine the necessary **size in bytes**. For example, `new int` requires **4 bytes** of memory being allocated. 
6. Then program will ask the **C standard library** to give **4 bytes of memory**! 

7. Then it will find a **contiguous block of 4 bytes memory** (in a roll) for us! Once done, it will **return a pointer to that memory address** so that we can read and write data from and to that memory!

   **My understanding:** a pointer is like a key to a vehicle. Once we receive a vehicle key we know we have a car in which some number of people could occupy. The actual number depends on the type of car you have received: SUV for 7 people or a sedan for 4 people. This is like the **int type for 4 bytes and char type for 1 byte.**

8. **Remember**: **New operator** takes time! Look these steps above. Basically there is **time taken** to **find a block of memory** that is **big enough** as we requested, e.g. **4 bytes**.

9. When **new used with class type**, e.g. `ClassName* e = new ClassName()`, note `ClassName()` or `ClassName` is the **constructor** which is **being called** with **new**.

10. You must use **delete** with **new operator** to free up the memory **manually**.

    `delete e;`

    `delete[] e` if e is an array!

11. placement new (decide where the memory comes from instead of just calling the constructor): 

    `int* b = new int[50]`

    `ClassName* e = new(b) ClassName()` **here new(b) is specifying the memory address which is b which is allocated 200 bytes in this particular example (ensure it is big enough to hold your class)**

