# CMake

## What is CMake ?

CMake is a tool used to generate *build scripts* (of course, along with a couple of other features).

## What is a build script?

A *build script* helps you compile, link and install your project.
For example, a *Makefile* is a commonly used kind of build script.

## Advantages of CMake

+ **It's platform independent**. For example, while MSVC uses slashes for options (/O2), GCC uses dashes (-O2). CMake takes care of such issues for you.
+ **It has a consistent and easy-to-comprehend syntax**. You only have to master one set of syntax of CMake, and the syntax is highly readable and self-explanatory like `target_link_libraries(tg lib1 lib2 ...)`. In older build systems such as GNU Autotools, users need to master different syntax for Autoconf and Automake, and possibly along with some knowledge about M4 macro. That syntax also appears to be clumsy sometimes. **A build system should make your life easier, not more complicated.**
+ **Growing IDE supporting**. Eclipse, KDevelop and CLion are on a growing path of supporting CMake.

## CMake in a typical project

Basically, there should be a `CMakeLists.txt` file (don't miss the **s** in the file name, and case-sensitive) 
in your project's root directory.
There should also be a `CMakeLists.txt` file in every sub-directory 
that contains source files to be compiled
(into object files, libraries or executables).

### `project_root/CMakeLists.txt`

```cmake
# Put this file in the root directory of your project
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project ("Project_name")

#############################
##  Set up compiler
############################

set(warnings "-Wall")   # set warning options
set(misc "-mavx2 -m64 -std=c++11")  # set other miscellaneous options, e.g., architecture extension, language standard ...

# Set default build type as debug
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Default build type: Debug." FORCE)
endif()

if(NOT CONFIGURED_ONCE)
#   Set compiler flags shared by all build types
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

#   Note: you must use FORCE when setting built-in variables. Because they exist in cache even before the first configuration.
endif()


#############################
##  Add source files
#############################
# Variables like CMAKE_SOURCE_DIR are CMake's built-in variables
include_directories("${CMAKE_SOURCE_DIR}/src")

# Add sub-directories
# Each sub-directory should contain a CMakeLists.txt for building targets in that directory
add_subdirectory(src)
add_subdirectory(experiments)

#########################################
##  Set up test 
##  1. GoogleTest library is assumed here
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
    COMMENT "Test log is written to ${testlog}"
)
add_dependencies(check check-build)
add_subdirectory(gtest-1.7.0 EXCLUDE_FROM_ALL)  # Exclude test targets from ALL
add_subdirectory(tests EXCLUDE_FROM_ALL)    # Same as above

set(CONFIGURED_ONCE TRUE CACHE INTERNAL "A flag showing that CMake has configured at least once.")

```

### Generating build script and building

In the following, we assume out-of-source building (similar to VPATH builds to GNU Automake). 
It helps us keep a clean source tree.
And when something gets messed up, you can simply remove the whole build tree.

```bash
mkdir build
cd build
cmake ..  # this will generate a set of folders and a build script (e.g., Makefile)
make      # build
# Alternatives:
cmake -DCMAKE_BUILD_TYPE=relase ..
make VERBOSE=1
# Set compiler version:
CC=gcc-4.9 CXX=g++-4.9 cmake ..
```

### `project_root/src/CMakeLists.txt`

The `src/` directory contains a bunch of **source files**.
I compile these classes into object files and group them as a (static) library.

```cmake
# Define a list 'demo_sources' for the source files of the library 'demo'
# It's not necessary to name it 'xxx_sources'.
# But doing so may be comfortable for those who move from GNU Autotools.
list(APPEND demo_sources
        a.cpp
        b.cpp
        c.cpp
    )

add_library(demo STATIC ${demo_sources})
```

### `project_root/experiments/CMakeLists.txt`

The `experiments/` directory contains a set of experiment programs.
Each .cpp file contains a `main` function and each file is compiled into an **executable**.

```cmake
list(APPEND experiments
    expm1
    expm2
    )

foreach(ee ${experiments})
    add_executable(${ee} "${ee}.cpp")
    target_link_libraries(${ee} demo)
endforeach()
```

### Using Google Test

GoogleTest comes along with its own CMake file that builds its own library.
My approach: simply copy the whole GoogleTest folder into your project directory,
then make it a sub-directory of your project. (A quick intro of GoogleTest: http://www.ibm.com/developerworks/aix/library/au-googletestingframework.html)

Notice the line `add_subdirectory(gtest-1.7.0)` in the root CMake file above.


### `project_root/tests/CMakeLists.txt`

The `tests/` folder contains a set of test cases that uses the GoogleTest framework.

```cmake
# Remember to include the GoogleTest's headers
# the variables like gtest_SOURCE_DIR are defined by gtest's CMake file
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

list(APPEND test_list
        test1
        test2
    )

foreach(tt ${test_list})
    add_executable(${tt}  "${tt}.cpp")
    target_link_libraries(${tt} demo gtest gtest_main)
    add_test(NAME ${tt} COMMAND ${tt} --gtest_color=yes)    # you may append other gtest options here
    add_dependencies(check-build ${tt})
endforeach()
```
