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












