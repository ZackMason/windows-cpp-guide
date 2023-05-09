# windows-cpp-guide
Collection of thoughts on productive windows workflows, specifically for cpp game development

# Table Of Contents

# Tools

## Highly Recommended
- visual studio - compiler
- vscode - ide
- cmake - build system
- conan - package manager
- emscripten - wasm compiler
- RemedyBG - amazing debugger, only on windows
- RenderDoc - if doing graphics this is required, use after 5-10 minutes of being stuck

# Libraries

## Highly Recommened
- fmt - formated printing
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

# VS Code Extensions

## Highly Recommened
- emacs keybinds - model editing is bad, lifting hands for arrows is bad, this solves both, what more can you say
- glsl color picker 
- markdown table formater - not as good as emacs' but does the job

# Compiling

Personally I find using cl (the compiler that visual studio uses in the background) directly from the command line more pleasent.

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









