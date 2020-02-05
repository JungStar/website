---
title: "Using modern CMake to test private C++ implementation files"
summary: "Using modern CMake to test private C/C++ implementation files without exposing them to the library user."
draft: false
date: "2020-02-05T00:00:00Z"
authors: 
  - philippjung
tags:
  - C++
  - C 
  - CMake
  - Linux
---
## The Problem
When writing C++ code, I often find myself wanting to write tests for private implementation files. Consider a small example project with the following folder structure:
```
<component>
├── include
│   └── src.h
├── private_include
│   └── private_impl.h
├── src
│   ├── src.cpp
│   └── private_impl.cpp
├── test
│   ├── test.t.cpp
│   └── CMakeLists.txt
└── CMakeLists.txt
```
Where the top-level `CMakeLists.txt` could look something like this:

```cmake
add_library(<component_target> SHARED src/src.cpp src/private_impl.cpp)
target_include_directories (<component_target> PRIVATE include)
```
and the `test/CMakeLists.txt` like this:

```cmake
add_executable(<test_target> test.t.cpp)
target_link_libraries(<test_target> PUBLIC <component_target>)
```
When writing tests in `test/test.t.cpp`, how can we access private implementations from the `private_include` directory?

### The "pitfall" solution
I have seen people do things, like including the relevant source file in their sources.
```cmake
# This is the wrong thing to do
SET (SOURCES 
      ../src/private_impl.cpp
      test.t.cpp)
```
This is a bad solution for several reasons.
* First, the implementation in `src/private_impl.cpp` is used outside the original component, e.g., link dependencies may no longer be satisfied. 
* Second, oftentimes there exists some logic (within CMake or otherwise) that sets context (like macros) depending on the current component. With the "patch" from before we now test `private_impl.cpp` in another context than it is normally used in.

### A better solution
Luckily, modern CMake has our back. We can explicitly model the depency on the private implementation of `<component>`, since we already specify all required information within the top level `CMakeLists.txt`. We simply need to access this information within our test environment.
This can be done using CMakes property system, i.e., we can get the necessary properties from `<component_target>` and reuse them for `<test_target>`:
```cmake 
add_executable(<test_target> test.t.cpp)
target_link_libraries(<test_target> PRIVATE <component_target>)

get_target_property (private_include_dirs <test_target> INCLUDE_DIRECTORIES)
get_target_property (private_link_libraries <test_target> LINK_LIBRARIES)

target_include_directories (<test_target> PRIVATE ${private_include_dirs})
target_link_libraries (<test_target> PRIVATE ${private_link_libraries})
```

This way, we make sure that we use the content of `private_impl.cpp` in the right context. We furthermore explicitly express the link dependency between our `<test_target>` and the corresponding `<component_target>`.

## Symbol Visibility (-fvisibility=hidden)[^1] 
If you are hiding ELF symbols by default (which you should), this method has the drawback that symbols from `private_impl.h` you are testing need to be marked as exported, so the linker can find them when linking `<test_target>`. This is done by specifying 
> `__attribute__ ((visibility ("default")))` 

for the symbol in question. Both GCC and Clang support this syntax.
As a side note, with modern CMake versions > 3.0.2, you can auto generate a cross platform export definition using `GenerateExportHeader`[^2]

See my other post on symbol visibility if you want to dive deeper into this topic.

[^1]: https://gcc.gnu.org/wiki/Visibility
[^2]: https://cmake.org/cmake/help/v3.0/module/GenerateExportHeader.html