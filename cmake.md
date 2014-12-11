# CMake

Basically, there should be a `CMakeLists.txt` file (don't miss the **s** in the file name) 
in your project's root directory.
There should also be a `CMakeLists.txt` file in every sub-directory 
that contains source files to be compiled
(either into object files or executables).

## Root CMake file and sub-directory CMake file

`project_root/CMakeLists.txt`
---
```cmake
# Put this file in the root directory of your project
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project ("Project_name")

#############################
##  Set up compiler
############################

set(warnings "-Wall")
set(archs "-mavx2 -m64 -std=c++11")

if(NOT CONFIGURED_ONCE)
#   Set warning/architecture/standard or other compiler flags 
#   shared by all build types (debug/release/relwithdebinfo)
    set(CMAKE_CXX_FLAGS "${warnings} ${archs}"
            CACHE STRING "Flags used by the compiler during all build types." FORCE)
#   Set compiler flags for different build types
#   mainly regarding debug option and optimization levels
    set(CMAKE_CXX_FLAGS_DEBUG "-ggdb3 -O1")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -D NDEBUG")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -ggdb3 -D NDEBUG")
endif()


#############################
##  Add source files
############################
# Add non-system include directories where the compiler will search for headers
# Variables like CMAKE_CURRENT_SOURCE_DIR are CMake's built-in variables
include_directories("${CMAKE_CURRENT_SOURCE_DIR}"  "${CMAKE_CURRENT_SOURCE_DIR}/include")

# Add sub-directories
# Each sub-directory should contain a CMakeLists.txt for building targets in that directory
add_subdirectory(src)
add_subdirectory(experiments)

#############################
##  Set up test
############################
option(test "Build all tests." ON)
option(autoplay "Auto play test immediately after build." ON)
if(test)
    message(STATUS "Tests will be built.")
    enable_testing()
    add_subdirectory(gtest-1.7.0)
    add_subdirectory(tests)
    add_test(NAME all-test COMMAND all-test)
endif()

set(CONFIGURED_ONCE TRUE CACHE INTERNAL "A flag showing that CMake has configured at least once.")

```

### Generating build script

Out-of-source building are usually used with CMake. 
That is, we create a separate fold named build/release/debug/etc. 
and put all built files in it.

```bash
mkdir build
cd build
cmake ..  # this will generate a set of folders and a build script (e.g., makefile)
make      # build
# Alternatives:
cmake -DCMAKE_BUILD_TYPE=debug ..
cmake -DCMAKE_BUILD_TYPE=relase -Dtest=OFF ..
VERBOSE=1 make
```

`project_root/src/CMakeLists.txt`
---
The `src` directory contains a bunch of **class definitions**.
I compile these classes into object files and group them as a (static) library.

```cmake
# foo_src is a list of *.cpp files in the this directory
file(GLOB foo_src RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "*.cpp")
# CMake will compile them into a library libfoo.a
add_library(foo STATIC ${foo_src})
```

`project_root/experiment/CMakeLists.txt`
---
The `experiment` directory contains a set of experiment programs.
Each *.cpp file contains a `main` function and each file is compiled into an **executable**.

```cmake
# My experiments need to use another library called pcm
# In this example, pcm is obtained as a pre-compiled library (*.a)
include_directories("${CMAKE_SOURCE_DIR}/pcm" "${CMAKE_CURRENT_SOURCE_DIR}")
file(GLOB experiment_src RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "*.cpp")
# Add an executable for each *.cpp file
foreach(srcfile ${experiment_src})
    string(REPLACE ".cpp" "" exp ${srcfile})
    add_executable(${exp} ${srcfile})
#   Linking my classes definition and pcm
    target_link_libraries(${exp} foo ${CMAKE_SOURCE_DIR}/pcm/pcm-lib.a)
endforeach()
```
