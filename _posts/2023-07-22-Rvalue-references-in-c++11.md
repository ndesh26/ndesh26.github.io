---
layout:     post
title:      Rvalue References in C++11
date:       2023-07-22 18:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [programming,cpp]
comments:   true

---
Rvalue references were introduced in the C++11 standard in September 2011, offering powerful capabilities for optimizing C++ code. Despite being a fundamental feature, rvalue references can seem daunting to C++ beginners. In this guide, we will demystify rvalue references and explore how they contribute to efficient and modern C++ programming.

<!--more-->

## C++03

### Value Categories
In C++03, each expression (not an object) has a value category, and it can either be a lvalue or a rvalue. The value categories were borrowed from C. Rvalue expressions are those that can only appear on the right-hand side of the assignment operator. The lvalues can appear on either side of the assignment operator. In C++, this definition was further enhanced. An lvalue is an expression which points to a location in memory (l stands for locator value). On the other hand, rvalues are temporaries which don’t necessarily have a place in memory.

Some examples of rvalues are 5 (in `int a = 5`), 5.0 etc. All the literals are rvalues except for string literals. The C++ standard defines the value category of the primary expressions. They decided to assign integers as rvalues as the compiler need not allocate a memory location for them; they could well be part of the instruction itself. On the hand, string literals can be arbitrarily large in size and hence we need to store them in memory. Thus the value category of string literals is lvalues. 

Rvalues mostly appear in contexts where we are performing initializations. For example:

```cpp
int a = 5;
B b = f();
```

Here 5 is a rvalue. The return value of function `f()` is also a rvalue. An interesting example of rvalue vs lvalue is the expression `++x` and `x++`. `++x` increments the value of x by one and returns the value of x. While `x++` increments the value of x by one and returns the previous value. This indicates that we need to store the previous value of x and return that. It becomes much more apparent when we look at the function signature of these operators as given in [cppreference](https://en.cppreference.com/w/cpp/language/operator_incdec). 

```cpp
T& T::operator++();   // returns a reference to the same object
T T::operator++(int); // return by value, creating a temporary
```

When a function returns an object by value, the value category of that object is rvalue, whereas the value category of a function that returns a object by reference is treated as lvalue. This is something that confused me for a while, as I have always treated references as pointers. And in both cases, we are returning an object of same size (most compilers implement references as pointers), which is a temporary which we cannot name. However, it is defined that way in the standard to keep consistency in the language. 

### References

Let's take a step back from the previous example and see how references interact with expressions of different value categories. References behave in ways similar to pointers and most compilers implement references as pointers. But when it comes to dealing with value categories, the behaviour of pointers and references differs. We cannot initialize a reference to an object with a rvalue expression. It can only be assigned to an expression of lvalue type. The following example shows the difference:

```cpp
class A {};

A f() {
    return A();
}
A& g() {
    A& a = *new A();
    return a;
}
A* h() {
    A* a = new A();
    return a;
}

A& a = f(); // compiler error, f returns a rvalue
A& b = g(); // works
A*& c = h(); // compiler error, h returns a rvalue
```

### Temporary Lifetime Extension

However, a reference to const, `const T&` can be initialized with both const lvalues and rvalues.
```cpp
const A& a = f(); // works
const A& b = g(); // works
A* const & c = h(); // works, const A*& won't work here. Check [1] for details.
```

Something interesting is happening in case of variable `a`. We are binding a reference to a temporary value returned by the function `f()`. At the end of the statement, the temporary should have been destroyed and we would have been left with dangling reference. But that's not what happens in this case; the lifetime of the temporary is extended until the end of the lifetime of the reference that binds to it. The destructor for temporary `A` will be called when variable `a` goes out of scope. Using reference to const, we can capture both rvalues and lvalues. This allows us to write functions which can accept named variables and temporaries.

```cpp
void func1(A a) {}
void func2(A& a) {}
void func3(const A& a) {}

A& a = g();

func1(a);     // works
func1(A());   // works

func2(a);     // works
func2(A())    // compiler error, cannot bind a reference to a rvalue 

func3(a);     // works
func3((A));   // works
```
A function taking an argument by value (`func1`) can take both rvalues and lvalues. However, it creates a copy in all cases which can be very suboptimal when dealing with large size objects. A function which takes its arguments by reference (`func2`) can avoid this expensive copy but it is unable to accept rvalue inputs. A function which takes reference to const type arguments (`func3`) can accept both lvalues and rvalues and avoids the expensive copy as well. 

But it also puts certain restrictions on what we can do with function parameters. In case of lvalues, we cannot modify them inside the function due to the const property. C++ doesn't allow us to modify rvalues as these are temporaries and modifying temporaries which disappear at the end of statement could lead to serious bugs. 

## C++11

Now the landscape from C++03 is set to introduce rvalue references. Const reference give us a nice way to deal with temporaries and helps us avoid the performance penalty of copying objects in parameter passing. But we are still doing redundant work in some cases. 

```cpp
class A {
    string s;
 public:  
    A(const string& in) : s(in) {} 
};

int main() {
    string s("Lvalue");
    A a(s);
    cout << s << endl;
    A b(string("Rvalue"));
}
```

If we look at the case of variable `b`, we are first creating a new temporary object of type `string` which allocates memory on heap (ignoring the small string optimization) and stores "Rvalue" string literal in that memory area. Then we call the constructor of `A`, where the temporary string then binds to the parameter `in` (This works because we are binding a rvalue expression to a const reference). Now we are initializing the member variable `s` of `A`, which in turn will call the copy constructor of `string`. This will lead to another heap allocation and copying of the values from the temporary to the member variable. And after the constructor of `A` is finished, we will then destroy the temporary string that we created and free any memory that it allocated. 

We are paying the performance penalty for the expensive copy constructor even though we are anyway going to discard the temporary anyway. If only we could steal the memory that was allocated for the temporary, then we can avoid the expensive copy constructor. In case of initialization of variable `a`, we cannot steal the memory of that variable as we might end up using that variable further down the line (as shown with cout).

In order to have an efficient implementation, we need two new concepts:

1. A way to differentiate between value category of the object that a const reference is bound to.
2. A light way to initialize a new object from a object which is about to die. 

C++11 standard introduced the concept of rvalues references and move constructors respectively to avoid expensive copies. In this post, I will only be focusing on rvalue references. A lot of the articles focus on move constructors and how to write proper move constructors. But a lot less articles talk about rvalue references and their properties

### Rvalue References

Rvalues references are denoted by the `&&` sign. Rvalue references are just like lvalue references (normal references are referred to lvalue references from c++11 onwards) in all other aspects except for the kind of object that it binds to. As you have already guessed, they can only bind to rvalues and even modify them.
**The most important feature of rvalue references is that they themselves are lvalues** as they are a named value themselves. They behave just like a lvalue reference in their usage and as such generate code which is similar to lvalue reference. Extending our previous example:

```cpp
A&& a = f(); // works, causes lifetime extension for temporary 
A&& b = g(); // compiler error, cannot bind a lvalue to rvalue reference
A*&& c = h(); // works
```

The below example shows the similarity between the behaviour of lvalue references and rvalue references:

```cpp
void q(A& a) {
    std::cout << "Lvalue" << std::endl;
}
void q(A&& a) {
    std::cout << "Rvalue" << std::endl;
}

A&& b = g();
q(g()); // prints Lvalue
q(b);   // prints Lvalue
q(f()); // prints Rvalue
```

An rvalue reference allows us to differentiate between a temporary value that could be destroyed vs lvalue that should not be modified. From our earlier example of string if we know that the parameter passed to the constructor is a temporary string then we can steal the heap area from the temporary and we wouldn't need to allocate a new heap and copy the contents from the temporary string. The introduction of rvalue references allows us to create a something called a move constructor. It's similar to a copy constructor but instead of taking a reference to const, it takes a rvalue reference allowing us the ability to steal things from the temporary instead of making a proper copy. In some case move constructors can run significantly faster than equivalent copy constructors (when heap memory is involved).

### std::move and xvalues

Just mentioning here for the sake of completeness, C++11 also introduced a new value category called xvalues. xvalues are lvalues which are no longer required and can be treated as temporary to allow other functions to steal their values. std::move provides a way to convert from lvalues to xvalues which behave as rvalues when binding to references. For example:

```cpp
void q(A& a) {
    std::cout << "Lvalue" << std::endl;
}
void q(A&& a) {
    std::cout << "Rvalue" << std::endl;
}

A& a = g();
q(std::move(a)); // prints Rvalue
```

## Conclusion

There is still a lot of things that we can talk about, various features which build on the rvalue references like move constructors and perfect forwarding, but I will save them for later. We will only cover rvalue references for this post. 

Understanding rvalue references is crucial for C++ beginners exploring the power of C++11 and beyond. By distinguishing between lvalues and rvalues, leveraging move semantics, and using std::move effectively, developers can write more efficient and modern C++ code.

[1] In case of `h()`, `A* const&` works whereas `const A*&` gives compiler error. The first type represents a reference to a const pointer to A which is reference to a const object. However the second type is a reference to a pointer to const A and hence pointer itself is a non const object hence this is a case of const reference.
