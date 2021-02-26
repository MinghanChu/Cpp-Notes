# C++ Strings

### Introduction

1. String is closely tied to arrays. **String** basically is an **array** of characters!
2. **string** is a group of characters, e.g. letters, symbols, basically **text**.
3. Need to understand how **characters** work, e.g. letters, symbols and numbers.
4. `char` type, `1` byte english character.

5. `''` single quote means character, `""` means `char` pointer!

6. `0` mean `null` to end a string array.
7. `std::string` is sort of equivalent to `char*`. But `std::string` is a class and therefore has its member functions, e.g. `.size()`.



**char**

```c++
#include <iostream>
#include <string> //to print string type variable must include this header file

int main()
{
  const char* name = "Minghan"; //must add "const" (read-only) 
  
  std::cout << name <<std::endl; //this will print out the whole characters stored at its location pointed by the name "pointer"
  std::cout << *name <<std::endl; //if dereference name only the character corresponding to the first location will be printed,   																	//i.e. "M"
  
  std::cin.get();
}
```



**string literals**

```c++
#include <iostream>
#include <string>

// \0 means string ends here
// e.g. "Min\0ghan", only "Min" is identified. \0 means null.

int main()
{
  const char name[8] = "Min\0han";//first of all, arrary size no longer smaller than 8, because we have 8 characters, plus one 
  																//implicit \0 in the end. Second of all, \0 means stop at here, therefore only "Min" will be
  																//printed.
  const char* name = "Minghan"; //you cannot write code like this (even without "const", its undefined behavior)
  
  char name[] = "Minghan";
  name[2] = 'e'; //yes you can write now! Space is not counted as a character! e.g. "Minghan is" then location 8 is "i" not 		   									//a space.
}


```

