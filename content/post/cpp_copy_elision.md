---
title: "(Guaranteed) Copy elision in C++11/14/17"
summary: "Using modern CMake to test private C/C++ implementation files without exposing them to the library user."
draft: true
date: "2021-02-05T00:00:00Z"
authors:
  - philippjung
tags:
  - C++
  - C++20
  - Copy-Elision
  - RVO
  - NRVO
---
Regardless of what you think of Herb Sutter's [Almost Always Auto (AAA)](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/) style, there are situations where using auto helps not repeating oneself.
For example, when using `std::make_unique<>`[^1] and `std::make_shared<>`.
```cpp
auto some_ptr = std::make_unique<SomeClass>();
```
But wait, isn't there a copy happening here, or at least a move?
According to the C++11 standard compilers are allowed to elide such copy and moves.
Meaning, it can construct the object on the right directly in the stack space of the left operant.
In our example, `std::unique_ptr<SomeClass>` could be directly constructed at the memory location of `some_ptr`.
However, prior to C++14 compilers are not required to do so, i.e., the expression on the right must be copy or moveable, even thought the copy and move assignment operators may never be called.


[^1]: requires C++14 upwards