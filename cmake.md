# CMake

CMake is a tool to generate build scripts.
A *build script* helps you compile and link your project codes automatically 
(for example, a *Makefile* is a build script).
CMake helps you generate such build scripts.
The original purpose of CMake is to be platform independent.
For example, the same CMake file can generate scripts for GNU make, clang and MSVC.

There is a growing population of IDEs supporting CMake.
E.g., KDevelop and CLion.

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

set(warnings "-Wall")   # set warning options
set(misc "-mavx2 -m64 -std=c++11")  # set other miscellaneous options, e.g., architecture extension, language standard ...

if(NOT CONFIGURED_ONCE)
#   Set compiler flags 
#   shared by all build types
    set(CMAKE_CXX_FLAGS "${warnings} ${misc}"
        CACHE STRING "Flags used by the compiler during all build types." FORCE)
#   Set compiler flags for different build types (debug/release/relwithdebinfo)
#   mainly with different debug option and optimization levels
    set(CMAKE_CXX_FLAGS_DEBUG "-ggdb3 -O1 -D NPREFETCH" 
        CACHE STRING "Flags used by the complier during debug build type." FORCE)
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -D NDEBUG -D NPREFETCH" 
        CACHE STRING "Flags used by the compiler during release build type." FORCE)
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -ggdb3 -D NDEBUG -D NPREFETCH" 
        CACHE STRING "Flags used by the compiler during relwithdebinfo." FORCE)
#   Set default build type as debug
    if(NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE Debug CACHE STRING "Default build type: Debug." FORCE)
    endif()
#   Note: you must use FORCE when setting built-in variables. Because they exist in cache even before the first configuration.
endif()


#############################
##  Add source files
#############################
# Add non-system include directories where the compiler will search for headers
# Variables like CMAKE_CURRENT_SOURCE_DIR are CMake's built-in variables
include_directories("${CMAKE_CURRENT_SOURCE_DIR}"  "${CMAKE_CURRENT_SOURCE_DIR}/include")

# Add sub-directories
# Each sub-directory should contain a CMakeLists.txt for building targets in that directory
add_subdirectory(src)
add_subdirectory(experiments)

#########################################
##  Set up test 
##  1. GoogleTest library is used here
##  2. Emulate 'make check' and 'make check-build'
##  from autotools. 
##  Reference: http://www.cmake.org/Wiki/CMakeEmulateMakeCheck
#########################################
enable_testing()
SET(testlog "tests/tests.log")  # Also redirect the test output to a log file
add_custom_target(check-build)  # 'check-build' only builds but not runs the tests.
add_custom_target(check 
    # a couple of options can be passed. See http://www.cmake.org/cmake/help/v2.8.8/ctest.html
    COMMAND ${CMAKE_CTEST_COMMAND}  
        --verbose --output-on-failure 
        --output-log ${testlog}
)
add_dependencies(check check-build)
add_subdirectory(gtest-1.7.0 EXCLUDE_FROM_ALL)  # Exclude test targets from ALL
add_subdirectory(tests EXCLUDE_FROM_ALL)    # Same as above

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
# Set compiler version:
CC=gcc-4.9 CXX=g++-4.9 cmake ..
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
Each \*.cpp file contains a `main` function and each file is compiled into an **executable**.

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

## Using Google Test

GoogleTest comes along with its own CMake file that builds its own library.
My approach: simply copy the whole GoogleTest folder into your project directory,
then make it a sub-directory of your project. (A quick intro of GoogleTest: http://www.ibm.com/developerworks/aix/library/au-googletestingframework.html)

Notice the line `add_subdirectory(gtest-1.7.0)` in the root CMake file above.

`project_root/test/CMakeLists.txt`
---
The `test` folder contains a set of test cases that uses the GoogleTest framework.

```cmake
# Remember to include the GoogleTest's headers
# the variables like gtest_SOURCE_DIR are defined by gtest's CMake file
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

list(APPEND test_list
        avx-utility_test
        bitvector_block_test
        bitvector_iterator_test
        bitvector_test
        byteslice_column_block_test
        column_test
    )

# For each test file, add it to an executable and add it to dependencies of 'check-build'
foreach(tt ${test_list})
    add_executable(${tt}  "${tt}.cpp")
    target_link_libraries(${tt} byteslice-core gtest gtest_main)
    add_test(NAME ${tt} COMMAND ${tt} --gtest_color=yes)
    add_dependencies(check-build ${tt})
endforeach()
```
