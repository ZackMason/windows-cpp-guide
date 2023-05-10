# windows-cpp-guide
Collection of thoughts on productive windows workflows, **specifically for cpp game development**, but non game devs might find this useful 

This page is very early in development (May 8, 2023)

Everything on this page is just my **personal opinion**. It's important that you find your own workflow that is most productive and best suits your goals, it might look something like this, or it might not.

# Important

Before continuing I am putting this here.

For the love of god please turn on your compiler warnings when programming in C/C++, you can do this using -Wall -Werror -Wconversion if using clang, or /W4 /WX on MSVC. Feel free to disable things like unused variables and functions, and push/pop disable warnings inside of dependencies if needed, but all other warning should be enabled.

A large majority of the bugs that I see people asking for help with would be caught if compiler warnings were properly set up. Yes it's a little annoying at first and takes some getting used to, but in the long run this will save you lots of time and pain.

Turning it on for the first time in projects that are well in to development can be daunting, but it's really not too bad, and you will catch lots of mistakes. Best to use this from day one though.

# Table Of Contents
- [Why C++](#why-c++)
- [Tools](#tools)
- [Libraries](#libraries)
- [VSCode](#vs-code)
- [Compiling](#compiling)
- [Static Databases](#static-databases)
- [Vulkan Resources](#vulkan-resources)
- [Hot Takes](#hot-takes)

# Why C++
Modern C++ is perfect for game development due to a number of reasons, such as strong type system with compile time execution, robust library support, console support, and more.

# Tools

## Highly Recommended
- [visual studio](https://visualstudio.microsoft.com/) - compiler
- [clang](https://clang.llvm.org/) - compiler, msvc is generally more up to date, but good to have
- [vscode](https://code.visualstudio.com/) - ide
- [RemedyBG](https://remedybg.itch.io/remedybg) - amazing debugger, only on windows, costs ~30$, worth every penny
- [cmake](https://cmake.org/) - build system
- [conan](https://conan.io/) - package manager
- [emscripten](https://emscripten.org/) - wasm compiler
- [RenderDoc](https://renderdoc.org/) - if doing graphics this is required, use after 5-10 minutes of being stuck
- [cmder](https://cmder.app/) - console emulator with git and linux commands
- [ninja](https://ninja-build.org/) - if using cmake

## Okay
- [milton](https://www.miltonpaint.com/) - infinite drawing board - Sketching problems is very underrated, especially in game dev

# Libraries

## Highly Recommended
- [fmt](https://github.com/fmtlib/fmt) - formatted printing
- [sdl](https://github.com/libsdl-org/SDL) - cross platform graphics / window management
- [stb](https://github.com/nothings/stb) - png, text, etc
- [vulkan](https://www.vulkan.org/) - graphics api
- [directx](https://learn.microsoft.com/en-us/windows/win32/directx) - graphics api
- [PhysX 5.0](https://github.com/NVIDIA-Omniverse/PhysX) - physics engine - BSD3

## Okay
- [opengl](https://learnopengl.com/) - graphics engine
- [smfl](https://www.sfml-dev.org/) - cross platform graphics (no wasm, thus bad for jams)
- [raylib](https://www.raylib.com/) - cross platform graphics (has wasm, don't like personally)
- [assimp](https://github.com/assimp/assimp) - annoying and bloat but gets the job done

## Bad
- [bullet](https://github.com/bulletphysics/bullet3) - just look at how complicated implementing a character controller is. Sure its easy to make a few boxes, but PhysX is just better in every way. Also the documentation is very bad compared to PhysX

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

```batch
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

```CMake
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


add_subdirectory(GameEngine)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

file(GLOB_RECURSE src_files 
    ${PROJECT_SOURCE_DIR}/src/*.c*
)

include_directories(include)
include_directories(GameEngine/include)
add_executable(game ${src_files})

// embed full path for development, it makes everything easier
target_compile_definitions(game PUBLIC GAME_ASSETS_PATH="${CMAKE_CURRENT_SOURCE_DIR}/assets/")
// but remember to change to relative for release
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
    "C:/Users/z/Documents/GitHub/PhysX"
)

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


# Hot Game Reload

When working on a large game project, it is very important to be able to make changes to the code while the game is running. Here is a basic example of how to do that. This is a condensed version of hot reloading for windows that comes from Casey Muratori's Handmade Hero Series, a linux or wasm implementation is also possible.

```C++
// app_interface.hpp

// included by both the game (dll) and platform (exe)

struct app_input_t {
    ...
};

struct app_memory_t {
    std::byte*  memory;
    size_t      size;

    bool        running{true};

    app_input_t input;

    // extend with app config info
};

#define no_mangle extern "C"
#define export_dll __declspec(dllexport)
#define export_fn(rt_type) no_mangle export_dll rt_type __cdecl

using app_callback_t = void(__cdecl *)(app_memory_t*);
struct app_dll_t {
    void* dll{nullptr};

    app_callback_t on_init{nullptr};
    app_callback_t on_deinit{nullptr};

    app_callback_t on_update{nullptr};
    app_callback_t on_render{nullptr};

    // these hooks are sometimes useful
    app_callback_t on_unload{nullptr};
    app_callback_t on_reload{nullptr};
};
```

``` C++
// app_platform.cpp 

#include "app_interface.hpp"

no_mangle void __cdecl 
default_game_callback(app_memory_t* app) {
    // no op
}

int main() {
    app_memory_t app_memory {
        .memory = VirtualAlloc(APP_SIZE, ...),
        .size = APP_SIZE,
    };

    app_dll_t app_dll {
        .dll = LoadLibraryA("game.dll");
    };
    load_game_callbacks(&app_dll);

    while (app_memory.running) {
        check_and_reload_dll(&app_dll);

        platform_input(&app_memory.input);

        (app_dll.on_update ? app_dll.on_update : default_game_callback)(&app_memory);
        (app_dll.on_render ? app_dll.on_render : default_game_callback)(&app_memory);

    }
}
```

``` C++
// Game.cpp

#include "app_interface.hpp"

struct game_t {

};

game_t* get_game(app_memory_t* app_memory) {
    return static_cast<game_t*>(app_memory->memory);
}

export_fn(void) 
app_on_init(app_memory_t* app_memory) {
    game_t* game = get_game(app_memory);
    new (game) game_t; // construct your game at the start of the memory block, build from there.
}

export_fn(void) 
app_on_update(app_memory_t* app_memory) {
    game_t* game = get_game(app_memory);

    // do game
}
```

# Static Databases

Often times in game development it's useful to have static game data, such as for levels and game entities.
One common solution for storing this data is to use a file format such as json or yaml, then parse the files at runtime. This works fine, however we can leverage C++'s type system and aggregate constructors in order to create a much better solution. 

```C++
namespace game::db {

struct entity_def_t {
    entity_type type{entity_type::environment};
    string_t type_name{};

    struct gfx_t {
        string_t mesh_name{};
        gfx::material_t material{};
        string_t albedo_tex{};
        string_t normal_tex{};

        string_t animations{};
    } gfx;

    std::optional<character_stats_t> stats{};
    std::optional<base_weapon_t> weapon{};

    struct physics_body_t {
        u32 flags{PhysicsEntityFlags_None};
        physics::collider_shape_type shape{physics::collider_shape_type::NONE};
        union {
            struct box_t {
                v3f size{};
            } box;
            struct sphere_t {
                f32 radius;
            } sphere;
            struct capsule_t {
                f32 radius;
                f32 height;
            } capsule;
        } shape_def;
    };
    std::optional<physics_body_t> physics{};

    struct particle_emitter_t {
        u32 count{};
        f32 rate{};
        v3f vel{};
        v3f acl{};
    };
    std::optional<particle_emitter_t> emitter{};

    struct child_t {
        const entity_def_t* entity{0};
        v3f                 offset{0.0f};
    };
    child_t children[10]{};
};


#define DB_ENTRY static constexpr entity_def_t

namespace misc {

// only fill out what you need
DB_ENTRY
teapot {
    // unlike json, we can add comments anywhere also!
    .type = entity_type::environment,
    .type_name = "Teapot",
    .gfx = {
        .mesh_name = "assets/models/utah-teapot.obj",
        .material = gfx::material_t::metal(gfx::color::v4::light_gray),
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Dynamic,
    
        .shape = physics::collider_shape_type::CONVEX,
    },
};

DB_ENTRY
door {
    .type = entity_type::environment,
    .type_name = "door",
    .gfx = {
        .mesh_name = "door",
        .material = gfx::material_t::plastic(gfx::color::v4::light_gray),
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Trigger,
        .shape = physics::collider_shape_type::BOX,
        .shape_def = {
            .box = {
                .size = v3f{1.0f},
            },
        },
    },
};

}; //namespace misc

namespace rooms {

DB_ENTRY
room_0 {
    .type = entity_type::environment,
    .type_name = "room_0",
    .gfx = {
        .mesh_name = "assets/models/rooms/room_0.obj",
        .material = gfx::material_t::metal(gfx::color::v4::light_gray),
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Static,
        .shape = physics::collider_shape_type::TRIMESH,
    },
};

DB_ENTRY
room_01 {
    .type = entity_type::environment,
    .type_name = "room_01",
    .gfx = {
        .mesh_name = "assets/models/rooms/room_01.obj",
        .material = gfx::material_t::metal(gfx::color::v4::light_gray),
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Static,
        .shape = physics::collider_shape_type::TRIMESH,
    },
};

DB_ENTRY
map_01 {
    .type = entity_type::environment,
    .type_name = "map_01",
    .gfx = {
        .mesh_name = "assets/models/map_01.obj",
        .material = gfx::material_t::metal(gfx::color::v4::light_gray),
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Static,
        .shape = physics::collider_shape_type::TRIMESH,
    },
};

}; //namespace rooms

namespace items {

}; //namespace items

namespace weapons {

DB_ENTRY
pistol {
    .type = entity_type::weapon,
    .type_name = "pistol",
    .gfx = {
        .mesh_name = "pistol",
        .material = gfx::material_t::metal(gfx::color::v4::dark_gray),
    },
    .weapon = wep::create_pistol(),
};

}; // namespace weapons

namespace characters {

DB_ENTRY
assassin {
    .type = entity_type::player,
    .type_name = "assassin",
    .gfx = {
        .mesh_name = "assets/models/capsule.obj",
    },
    .stats = character_stats_t {
        .health = {
            80
        },
        .movement = {
            .move_speed = 1.3f,
        },
    },
    .physics = entity_def_t::physics_body_t {
        .flags = PhysicsEntityFlags_Character,
        .shape = physics::collider_shape_type::CAPSULE,
        .shape_def = {
            .capsule = {
                .radius = 1.0f,
                .height = 3.0f,
            },
        },
    },
    .children = {
        {
            .entity = &weapons::rifle,
            .offset = v3f{0.0f},
        },
        {
            .entity = &weapons::pistol,
            .offset = v3f{0.0f},
        },
    },
};

}; // namespace characters

}; // namespace game::db

...

void spawn_level() {
    using game::db;

    spawn_player(character::assassin);

    spawn(room:room_01);

    ...
}

```

Using a structure such as this allows for static type checking, the ability to call constexpr engine functions, store const pointers, and even create hierarchies. Combining unions/variants and optionals allows for storing a large variety of data, that is fairly easy to reflect on later when it comes to instantiating these "Prefabs". If you ensure that your struct is mem copyable, you can even write and load the structs as binary straight to disk. One thing you might find annoying is that fields need to be listed in order, but I find that this helps keeps all of the data consistent and easy to read. Another thing that you could do is load these symbols from a dll, perhaps for modding.

# Vulkan Resources

- [vkguide.dev](https://vkguide.dev/) - Very good, focuses on building a usable engine, with GPU driven rendering
- [Sascha Willems](https://github.com/SaschaWillems/Vulkan) - The goat himself
- [vulkan tutorial](https://vulkan-tutorial.com/) - extremely basic, but a good start.

# Hot Takes
- linux sucks, but that's why we're here, not a hot take
- null terminated strings suck
- thus all str* functions suck and should be banned from use, use mem* instead
- no really, they are terrible, stop using them, always store the size
- malloc sucks, use an allocation pattern that supports many different many allocation backends
- free at the end of a program (and arguably always) is bloat, OS will free resources faster, yes even graphics resources.
- cpp is a nice language when used right (ie not by me)
- memory management should always happen at the highest possible level, nodes allocating and managing other nodes is a nightmare of life time spaghetti. Always try to do allocations at least one level up the call stack.
- stl has a few good parts - span, string_view, array, bitset, tuple, variant
- stl has a lot of bad parts - vector, map, set, string, function
- writing code that is easy to step through in a debugger is vital, functional programming and OOP greatly hinders this.