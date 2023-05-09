# windows-cpp-guide
Collection of thoughts on productive windows workflows, specifically for cpp game development

This page is very early in development (May 9, 2023)

Before continuing I am putting this here.

For the love of god please turn on your compiler warnings when programming in C/C++
using -Wall -Werror -Wconversion if using clang, or /W4 /WX on MSVC.
Feel free to disable things like unused variables and functions, and push/pop disable warnings inside of dependencies if needed, but all other warning should be fixed.

The large majority of bugs that I see people asking would be caught if compiler warnings were properly set up

# Table Of Contents
- [Tools](#tools)
- [Libraries](#libraries)
- [VSCode](#vs-code)
- [Compiling](#compiling)

# Tools

## Highly Recommended
- visual studio - compiler
- clang - compiler, msvc is generally more up to date, but good to have
- vscode - ide
- cmake - build system
- ninja - if using cmake
- conan - package manager
- emscripten - wasm compiler
- RemedyBG - amazing debugger, only on windows
- RenderDoc - if doing graphics this is required, use after 5-10 minutes of being stuck
- Milton - infinite drawing board - Sketching problems is very underrated, especially in game dev

# Libraries

## Highly Recommended
- fmt - formatted printing
- sdl - cross platform graphics / window management
- stb - png, text
- vulkan - graphics api
- directx - graphics api
- PhysX - physics engine

## Okay
- opengl - graphics engine
- smfl - cross platform graphics (no wasm, thus bad for jams)
- raylib - cross platform graphics (has wasm, don't like personally)
- assimp - annoying and bloat but gets the job done

## Bad
- Bullet - just look at how complicated implementing a character controller is. Sure its easy to make a few boxes, but physx is just better in every way.

# VS Code 

## Extensions

### Highly Recommended
- emacs keybinds - model editing is bad, lifting hands for arrows is bad, this solves both, what more can you say
- glsl color picker 
- markdown table formatter - not as good as emacs' but does the job

# Compiling

## CL (Visual Studio CLI)
Personally I find using cl (the compiler that visual studio uses in the background) directly from the command line more pleasant.

here what that might look like with a batch script

```
@echo off

rem MSVC build script
rem need to run from Visual Studio Command Prompt
rem or C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvars64.bat
rem in order to use cl


if not exist src echo "no source"
if not exist build mkdir build

pushd build

set Compiler=cl
set CompilerFlags=-nologo -FC -WX -W4 -wd4100 -wd4201 -wd4702 -wd4701 -wd4189 /std:c++20 /EHsc
set SDLLinkFlags=/LIBPATH:..\lib\release SDL2.lib
set LinkFlags=-opt:ref user32.lib gdi32.lib shell32.lib %SDLLinkFlags%

set OptimizationFlagsRelease=/DNDEBUG /O2 /fp:fast /arch:AVX2 /openmp 
set OptimizationFlagsDebug=/DEBUG:full /Zi /O0 /fp:fast /arch:AVX2 /openmp

set OptimizationFlags=%OptimizationFlagsRelease%
if "%~1"=="debug" (
    set OptimizationFlags=%OptimizationFlagsDebug%
    shift
)
if "%~1"=="release" (
    set OptimizationFlags=%OptimizationFlagsRelease%
    shift
)

set BuildFile=%1
shift
rem shift doesnt affect %* so I'm hard coding it
set Arg1=%~1
set Arg2=%~2
set Arg3=%~3

set IncludeFlags=/I ..\include

%Compiler% %Arg1% %Arg2% %Arg3% %OptimizationFlags% %IncludeFlags% %CompilerFlags% ..\%BuildFile% /link %LinkFlags%

popd build

```


## CMAKE

A good option if you have a lot of dependencies is to use cmake, ninja, and conan. I have an old script that initializes projects with a cmake and optional conan setup [here](https://github.com/ZackMason/cpp_init).

You will need to learn how each of these tools work. learning cmake is unfortunately not very easy in my experience, there is a lot of dated info. Reading through the script linked above will give you a starting idea.

Heres what a cmake script might look like 

```
cmake_minimum_required(VERSION 3.8.12)

project(game C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_C_STANDARD 11)

SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR})
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR})

if ( CMAKE_COMPILER_IS_GNUCC )
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wall -Wextra")
endif()
if ( MSVC )
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} /W4 /WX /wd4100 /wd4201")
endif()


add_subdirectory(ActuallyAGameEngine)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

file(GLOB_RECURSE src_files 
    ${PROJECT_SOURCE_DIR}/src/*.c*
)

add_compile_options(/bigobj)

include_directories(include)
include_directories(ActuallyAGameEngine/include)
add_executable(game ${src_files})

target_compile_definitions(game PUBLIC GAME_ASSETS_PATH="${CMAKE_CURRENT_SOURCE_DIR}/assets/")
#target_compile_definitions(game PUBLIC GAME_ASSETS_PATH="./assets/") # for release


file(GLOB_RECURSE test_files 
    ${PROJECT_SOURCE_DIR}/src/*.c*
    ${PROJECT_SOURCE_DIR}/tests/*.c*
)
list(REMOVE_ITEM test_files ${PROJECT_SOURCE_DIR}/src/main.c)
list(REMOVE_ITEM test_files ${PROJECT_SOURCE_DIR}/src/main.cpp)

add_executable(tests ${test_files})
target_compile_definitions(tests PUBLIC GAME_ASSETS_PATH="${CMAKE_CURRENT_SOURCE_DIR}/assets/")
target_link_libraries(tests ${CONAN_LIBS} Engine)
target_link_libraries(game ${CONAN_LIBS} Engine)

set(PHYSX_ROOT_WIN
    "C:/Users/zack/Documents/GitHub/PhysX"
)

# https://github.com/nyers33/minimal_glfw_physx/blob/main/src/CMakeLists.txt
set(PHSYX_LIBS
	"PhysX_64.lib"
	"PhysXCommon_64.lib"
	"PhysXCooking_64.lib"
	"PhysXFoundation_64.lib"

	"PhysXPvdSDK_static_64.lib"
	"PhysXVehicle_static_64.lib"
	"PhysXExtensions_static_64.lib"
	"PhysXCharacterKinematic_static_64.lib"
)

set(PHSYX_LIBS_ROOT_DIR "${PHYSX_ROOT_WIN}/physx/bin/win.x86_64.vc142.mt")

include_directories("${PHYSX_ROOT_WIN}/physx/include" )
include_directories("${PHYSX_ROOT_WIN}/pxshared/include" )

foreach(lib ${PHSYX_LIBS})
    target_link_libraries(game optimized "${PHSYX_LIBS_ROOT_DIR}/checked/${lib}")
    target_link_libraries(tests optimized "${PHSYX_LIBS_ROOT_DIR}/checked/${lib}")
    target_link_libraries(game debug "${PHSYX_LIBS_ROOT_DIR}/debug/${lib}")
    target_link_libraries(tests debug "${PHSYX_LIBS_ROOT_DIR}/debug/${lib}")
endforeach()
```





