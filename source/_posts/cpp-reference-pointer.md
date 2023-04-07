---
title: "Comprehensive Guide to C++ Reference and Pointer"
tags:
  - c++
categories:
  - learning log
date: 2020-08-13
description: Learn how to effectively use C++ pointers and references in our comprehensive guide. From understanding the basics to mastering advanced techniques, our expert-written tutorial will help you write efficient and error-free code. Whether you're a beginner or an experienced programmer, join our community and take your C++ skills to the next level!
---
  
## Basics

**De-referencing**

add * before a pointer to expose the object’s value it's pointing

- `varN`, varN’s value
- `ptrN`, varN’s address
- `*ptrN`, de-reference ptrN, varN’s value
- (`&ptrN`, ptrN’s address, which has little meaning)

**Symbol to access a member of a class or class object (instance)**

- `a->b`: **b** is the member of the object that pointer **a** refers to
- `a.b`: **b** is the member of the object, or the reference of an object **a**
- `a::b`: **b** is the member of the class, or namespace **a**

## Declaration

### Pointer Declaration

Syntax: `varType* varName`

Declaration
```cpp
//declare a c++ pointer to an integer
int* ptrx; 
```

Initialization
```cpp
// a pointer pointing to address of varN, varN type should be int
int varN = 9;
int* ptrN;
ptrN = &varN;
```

short-hand:
```cpp
int* ptrN = &varN;  
```
    

### Reference Declaration

Syntax: `varType& varName`

```cpp
int main()
{
    int& invalidRef;   // error: references must be initialized

    int x = 5;
    int& ref = x; // okay: reference to int is bound to int variable

    return 0;
}
```

When a reference is initialized with an object (or function), we say it is **bound** to that object (or function). The process by which such a reference is bound is called **reference binding**

references must be bound to a *modifiable variable*

## Function Pass Argument

> Both of these methods are prevented copying the variable, and capable of modifying the variable value that is passed in; if a variable is expensive to copy, e.g. struct, class
> 

### By Reference

- Modify the variable, can’t pass in an un-modifiable variable like `const`
    
    Syntax `varType& varName`
    
    ```cpp
    void printValue(int& y)
    {
        std::cout << y << '\n';
    }
    ```
    
- Prevent modifying the variable, while having the benefit of not making a copy
    
    Syntax `const varType& varName`
    
    ```cpp
    void printValue(const int& y)
    {
        std::cout << y << '\n';
    }
    ```
    

### By Address (via pointer)

Syntax `varType* varName`, to access the variable, de-reference using `*varName`
    
```cpp
void printByAddress(const std::string* ptr) // The function parameter is a pointer that holds the address of str
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}
```
    

## Function Return Argument

### Variable Pointer

works almost identically to return by reference, except a pointer to an object is returned instead of a reference to an object

The major advantage of return by address over return by reference is that we can have the function return `nullptr` if there is no valid object to return

### Variable Reference

> Never return a reference to a local variable or some such, because it won't be there to be referenced.
> 

Syntax `varType& funName(args)`


## References

[Stack Overflow - When do I use a dot, arrow, or double colon to refer to members of a class in C++?](https://stackoverflow.com/questions/4984600/when-do-i-use-a-dot-arrow-or-double-colon-to-refer-to-members-of-a-class-in-c)

[Runestone Academy - Pointers](https://runestone.academy/ns/books/published/cpp4python/AtomicData/AtomicData.html#pointers)

[Learn C++ - Lvalue reference](https://www.learncpp.com/cpp-tutorial/lvalue-references/)

[Learn C++ - Return by reference and return by address](https://www.learncpp.com/cpp-tutorial/return-by-reference-and-return-by-address/)

[Learn C++ - Pass by lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-lvalue-reference/)

[Learn C++ - Pass by address](https://www.learncpp.com/cpp-tutorial/pass-by-address/)

