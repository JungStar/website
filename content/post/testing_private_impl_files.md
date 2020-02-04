---
title: "Using modern CMake to test private C++ implementation files"
summary: "Using modern CMake to test private C/C++ implementation files without exposing them to the library user."
draft: true
date: "2020-01-01T00:00:00Z"
authors: ["philippjung"]
categories:
  - "programming"
tags:
  - C++
  - C 
  - CMake
  - Linux
---
## The Problem
When writing C++ code, I often find myself in a situation where I want to write tests for private implementation files. Consider a small example project with the following folder structure:
```
component
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
When writing tests in `test/test.t.cpp`, how can we access private implementions from `private_include` setting this directory as public include dir?

### The "pitfall" solution
I have seen people do things, like including the relevant source file in their sources.
```cmake
# This is the wrong thing to do
SET (SOURCES 
      ../src/private_impl.cpp
      test.t.cpp)
```
Why is this a bad solution? Well, first, the implementation in `src/private_impl.cpp` is out of context, e.g., link dependencies may no longer be satisfied. 
Second, oftentimes there exists some logic (within CMake or otherwise) that sets context (like macros) depending on the current component. With the "patch" from before we now test `private_impl.cpp` in another context than it is used in the rest of our codebase.

### A better solution
Luckily, modern CMake has our back. The relevant parts of the component level `CMakeLists.txt` could look something like this:
```cmake
target_include_directories (<component_target> PUBLIC include)
target_include_directories (<component_target> PRIVATE private_include)

target_link_libraries(<component_target> PRIVATE <private_link_dependency>)
```
Meaning, we already specify all required information, we simply need to access it within our test environment.
This can be done using CMakes property system, i.e., we ca get the necessary properties from `<component_target>` and reuse them for `<test_target>`:
```cmake 
get_target_property (private_include_dirs <test_target> INCLUDE_DIRECTORIES)
get_target_property (private_link_libraries <test_target> LINK_LIBRARIES)

target_include_directories (<test_target> PRIVATE ${private_include_dirs})
target_link_libraries (<test_target> PRIVATE ${private_link_libraries})
```
You probably also want to link against the interface part of `<component_target>`
```cmake
target_link_libraries (<test_target> PRIVATE <component_target>)
```

This way, we make sure that we use the content of `private_impl.cpp` in the right context. We furthermore explicitly express the link dependency between our `<test_target>` and the corresponding `<component_target>`.

## Symbol Visibility (-fvisibility=hidden)[^1] 
If you are hidding ELF symbols by default like me, this method has the drawback that symbols from `private_impl.h` you are testing need to be marked as exported, so the linker can find them when linking the `<test_target>`. This is done by specifying 
> `__attribute__ ((visibility ("default")))` 

for the symbol in question, both gcc and clang support this syntax.
As a side note, with modern CMake versions > 3.0.2, you can auto generate a cross platform export definition using `GenerateExportHeader`[^2]

See my other post on symbol visibility if you want to dive deeper into this issue.

[^1]: https://gcc.gnu.org/wiki/Visibility
[^2]: https://cmake.org/cmake/help/v3.0/module/GenerateExportHeader.html