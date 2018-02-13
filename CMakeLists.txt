cmake_minimum_required(VERSION 3.1)
project(bgfx)

if(CMAKE_BUILD_TYPE MATCHES DEBUG OR CMAKE_BUILD_TYPE MATCHES RELWITHDEBINFO)
    target_compile_definitions(bgfx PUBLIC BGFX_CONFIG_DEBUG=${BGFX_DEBUG})
    target_compile_definitions(bimg PUBLIC BGFX_CONFIG_DEBUG=${BGFX_DEBUG})
    target_compile_definitions(bx PRIVATE BGFX_CONFIG_DEBUG=${BGFX_DEBUG}) # private, so that it won't affect target shaderc
endif()

if(APPLE)
    include_directories("bx/include/compat/osx")
endif()

# -------------------- 3rd party -----------------------
include_directories(3rdparty/khronos)

file(GLOB BGFX_3RD_PARTY
    "./bimg/src/*.cpp"
    "./bimg/3rdparty/edtaa3/*.cpp"
    "./bimg/3rdparty/etc1/*.cpp"
    "./bimg/3rdparty/etc2/*.cpp"
    "./bimg/3rdparty/iqa/source/*.cpp"
    "./bgfx/3rdparty/fcpp/*.cpp"
    "./bgfx/3rdparty/ib-compress/*.cpp"
    "./bimg/3rdparty/libsquish/*.cpp"
    "./bgfx/3rdparty/ocornut-imgui/*.cpp"
    "./bgfx/3rdparty/stb/*.cpp"
    )

add_library(bgfx3rdParty STATIC ${BGFX_3RD_PARTY})

set_target_properties(bgfx3rdParty PROPERTIES LINKER_LANGUAGE C)

target_include_directories(bgfx3rdParty PUBLIC bgfx/3rdparty/khronos)
target_include_directories(bgfx3rdParty PUBLIC bgfx/3rdparty)
target_include_directories(bgfx3rdParty PUBLIC bx/include)
target_include_directories(bgfx3rdParty PUBLIC bimg/include)
target_include_directories(bgfx3rdParty PUBLIC bimg/3rdparty)
target_include_directories(bgfx3rdParty PUBLIC bimg/3rdparty/iqa/include)

if(MINGW)
    target_include_directories(bgfx3rdParty PUBLIC bx/include/compat/mingw)

    add_definitions(-DBGFX_CONFIG_RENDERER_DIRECT3D9=0)
    add_definitions(-DBGFX_CONFIG_RENDERER_DIRECT3D11=0)
    add_definitions(-DBGFX_CONFIG_RENDERER_DIRECT3D12=0)
    add_definitions(-D__STDC_FORMAT_MACROS)
    

    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wunused-value")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdata-sections")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffunction-sections")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse2")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wunused-value")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wundef")
endif()

if(MSVC)
    add_definitions(-DBGFX_CONFIG_RENDERER_OPENGL=1)
    add_definitions(-DBGFX_CONFIG_RENDERER_DIRECT3D12=0)

	target_include_directories(bgfx3rdParty PUBLIC bx/include/compat/msvc)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	add_definitions(-D__STDC_FORMAT_MACROS)
	
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /wd4577 /wd4996")
	
	target_link_libraries(bgfx3rdParty Psapi)
endif()

# ------------------ Main library ----------------------
SET(BX_SRC "./bx/src/amalgamated.cpp")
SET(BGFX_SRC "./bgfx/src/amalgamated.cpp")

if(APPLE)
    list(APPEND BGFX_SRC
        "./bgfx/src/glcontext_nsgl.mm"
        "./bgfx/src/renderer_mtl.mm"
    )
endif()

add_library(bx STATIC ${BX_SRC})

add_library(bgfx STATIC ${BGFX_SRC})
set_target_properties(bgfx PROPERTIES LINKER_LANGUAGE C)

if(MINGW)
    target_include_directories(bx PUBLIC bx/include/compat/mingw)
endif()

if(MSVC)
    target_include_directories(bx PUBLIC bx/include/compat/msvc)
endif()

if(APPLE)
    set_target_properties(bgfx PROPERTIES CXX_STANDARD 98)

    find_library(COCOA_LIBRARY Cocoa)
    if(NOT COCOA_LIBRARY)
        message(FATAL_ERROR "Could not find Cocoa library")
    else()
        message(STATUS "Found Cocoa library: ${COCOA_LIBRARY}")
    endif()

    find_library(METAL_LIBRARY Metal)
    if(METAL_LIBRARY)
        message(STATUS "Found Metal library: ${METAL_LIBRARY}")
    endif()

    find_library(QUARTZ_LIBRARY QuartzCore)
    if(QUARTZ_LIBRARY)
        message(STATUS "Found Quartz library: ${QUARTZ_LIBRARY}")
    endif()

    find_package(OpenGL REQUIRED)

    list(APPEND OPENGL_LIBRARIES "${COCOA_LIBRARY}" "${METAL_LIBRARY}" "${QUARTZ_LIBRARY}")
else()
    if(NOT ANDROID)
        set(OPENGL_LIBRARIES "GL")
    endif()
endif()

if(ANDROID)
    target_link_libraries(bgfx log dl)
elseif(UNIX)
    set(DL "")
    if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
        set(DL dl)
    endif()

    find_library(X11_LIBRARY X11)
    if(NOT X11_LIBRARY)
        message(FATAL_ERROR "Could not find X11 library")
    else()
        message(STATUS "Found X11 library: ${X11_LIBRARY}")
    endif()

    target_link_libraries(bgfx ${X11_LIBRARY} ${OPENGL_LIBRARIES} pthread ${DL})
    target_link_libraries(bx pthread ${DL})
endif()

target_include_directories(bx PUBLIC bx/include)
target_include_directories(bx PUBLIC bx/3rdparty)

target_include_directories(bgfx PUBLIC bgfx/include)
target_include_directories(bgfx PUBLIC bx/include)
target_link_libraries(bgfx bgfx3rdParty bx)
    
if(APPLE)
    target_include_directories(bgfx PRIVATE ${MALLOC_INCLUDE_DIR})
endif()

# ------------------ Common library ----------------------
file(GLOB COMMON_SRC
    "./bgfx/examples/common/bounds.cpp"
    "./bgfx/examples/common/cube_atlas.cpp"
    "./bgfx/examples/common/*.cpp"
    "./bgfx/examples/common/debugdraw/*.cpp"
    "./bgfx/examples/common/entry/*.cpp"
    "./bgfx/examples/common/font/*.cpp"
    "./bgfx/examples/common/imgui/*.cpp"
    "./bgfx/examples/common/nanovg/*.cpp")
    

if(APPLE)
    list(APPEND COMMON_SRC "./bgfx/examples/common/entry/entry_osx.mm")
endif()

add_library(bgfx_common STATIC ${COMMON_SRC})
target_link_libraries(bgfx_common bgfx bx bgfx3rdParty)
set_target_properties(bgfx_common PROPERTIES LINKER_LANGUAGE C)

target_include_directories(bgfx_common PUBLIC bgfx/examples/common)
target_include_directories(bgfx_common PUBLIC bgfx/examples)

if(APPLE)
    target_include_directories(bgfx_common PRIVATE ${MALLOC_INCLUDE_DIR})
endif()







# ------------------ library for shader compiler: glslang ----------------------

set(GLSLANG_DIR "bgfx/3rdparty/glslang")

file(GLOB_RECURSE GLSLANG_SRC
    "${GLSLANG_DIR}/glslang/*.cpp"
    "${GLSLANG_DIR}/glslang/*.h"

    "${GLSLANG_DIR}/hlsl/*.cpp"
    "${GLSLANG_DIR}/hlsl/*.h"

    "${GLSLANG_DIR}/SPIRV/*.cpp"
    "${GLSLANG_DIR}/SPIRV/*.h"

    "${GLSLANG_DIR}/OGLCompilersDLL/*.cpp"
    "${GLSLANG_DIR}/OGLCompilersDLL/*.h"
)

if (MSVC)
    set(OS_EXCLUDE_FOLDER "Unix")
else()
    set(OS_EXCLUDE_FOLDER "Windows")
endif()

file(GLOB GLSL_REMOVE_FILES
    "${GLSLANG_DIR}/glslang/OSDependent/*/main.cpp"
    "${GLSLANG_DIR}/glslang/OSDependent/${OS_EXCLUDE_FOLDER}/*.cpp"
    "${GLSLANG_DIR}/glslang/OSDependent/${OS_EXCLUDE_FOLDER}/*.h"
)

foreach(remove_file ${GLSL_REMOVE_FILES})
    list(REMOVE_ITEM GLSLANG_SRC "${remove_file}")
endforeach()

add_library(glslang STATIC ${GLSLANG_SRC})
target_include_directories(glslang PUBLIC "${GLSLANG_DIR}")
target_compile_definitions(glslang PUBLIC ENABLE_HLSL=1)

if (MSVC)
    target_compile_options(glslang PRIVATE "/wd4005") # warning C4005: '_CRT_SECURE_NO_WARNINGS': macro redefinition
    target_compile_options(glslang PRIVATE "/wd4100") # error C4100: 'inclusionDepth' : unreferenced formal parameter
    target_compile_options(glslang PRIVATE "/wd4127") # warning C4127: conditional expression is constant
    target_compile_options(glslang PRIVATE "/wd4244") # warning C4244: '=': conversion from 'int' to 'char', possible loss of data
    target_compile_options(glslang PRIVATE "/wd4456") # warning C4456: declaration of 'feature' hides previous local declaration
    target_compile_options(glslang PRIVATE "/wd4457") # warning C4457: declaration of 'token' hides function parameter
    target_compile_options(glslang PRIVATE "/wd4458") # warning C4458: declaration of 'language' hides class member
    target_compile_options(glslang PRIVATE "/wd4702") # warning C4702: unreachable code

else()
    target_compile_options(glslang PRIVATE "-Wno-deprecated-register")
    target_compile_options(glslang PRIVATE "-Wno-ignored-qualifiers")
    target_compile_options(glslang PRIVATE "-Wno-inconsistent-missing-override")
    target_compile_options(glslang PRIVATE "-Wno-missing-field-initializers")
    target_compile_options(glslang PRIVATE "-Wno-reorder")
    target_compile_options(glslang PRIVATE "-Wno-return-type")
    target_compile_options(glslang PRIVATE "-Wno-shadow")
    target_compile_options(glslang PRIVATE "-Wno-sign-compare")
    target_compile_options(glslang PRIVATE "-Wno-undef")
    target_compile_options(glslang PRIVATE "-Wno-unknown-pragmas")
    target_compile_options(glslang PRIVATE "-Wno-unused-parameter")
    target_compile_options(glslang PRIVATE "-Wno-unused-variable")

    target_compile_options(glslang PRIVATE "-Wno-unused-but-set-variable")
endif()

# ------------------ library for shader compiler: glsl-optimizer ----------------------
set(GLSL_OPTIMIZER_DIR "bgfx/3rdparty/glsl-optimizer")

file(GLOB_RECURSE GLSL_OPTIMIZER_SRC
    "${GLSL_OPTIMIZER_DIR}/src/mesa/*.c"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/*.cpp"
    "${GLSL_OPTIMIZER_DIR}/src/mesa/*.h"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/*.c"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/*.cpp"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/*.h"
    "${GLSL_OPTIMIZER_DIR}/src/util/*.c"
    "${GLSL_OPTIMIZER_DIR}/src/util/*.h"
)

file(GLOB_RECURSE GLSL_OPTIMIZER_REMOVE_FILES
    "${GLSL_OPTIMIZER_DIR}/src/glsl/glcpp/glcpp.c"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/glcpp/tests/*"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/glcpp/*.l"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/glcpp/*.y"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/ir_set_program_inouts.cpp"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/main.cpp"
    "${GLSL_OPTIMIZER_DIR}/src/glsl/builtin_stubs.cpp"
)

foreach(remove_file ${GLSL_OPTIMIZER_REMOVE_FILES})
    list(REMOVE_ITEM GLSL_OPTIMIZER_SRC "${remove_file}")
endforeach()

add_library(glsl-optimizer STATIC ${GLSL_OPTIMIZER_SRC})

target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/src")
target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/include")
target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/src/mesa")
target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/src/mapi")
target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/src/glsl")

if(MSVC)
    target_include_directories(glsl-optimizer PUBLIC "${GLSL_OPTIMIZER_DIR}/src/glsl/msvc")

    target_compile_definitions(glsl-optimizer PRIVATE
            __STDC__
            __STDC_VERSION__=199901L
            strdup=_strdup
            alloca=_alloca
            isascii=__isascii
    )

    target_compile_options(glsl-optimizer PRIVATE "/wd4100") # error C4100: '' : unreferenced formal parameter
    target_compile_options(glsl-optimizer PRIVATE "/wd4127") # warning C4127: conditional expression is constant
    target_compile_options(glsl-optimizer PRIVATE "/wd4132") # warning C4132: 'deleted_key_value': const object should be initialized
    target_compile_options(glsl-optimizer PRIVATE "/wd4189") # warning C4189: 'interface_type': local variable is initialized but not referenced
    target_compile_options(glsl-optimizer PRIVATE "/wd4204") # warning C4204: nonstandard extension used: non-constant aggregate initializer
    target_compile_options(glsl-optimizer PRIVATE "/wd4244") # warning C4244: '=': conversion from 'const flex_int32_t' to 'YY_CHAR', possible loss of data
    target_compile_options(glsl-optimizer PRIVATE "/wd4389") # warning C4389: '!=': signed/unsigned mismatch
    target_compile_options(glsl-optimizer PRIVATE "/wd4245") # warning C4245: 'return': conversion from 'int' to 'unsigned int', signed/unsigned mismatch
    target_compile_options(glsl-optimizer PRIVATE "/wd4701") # warning C4701: potentially uninitialized local variable 'lower' used
    target_compile_options(glsl-optimizer PRIVATE "/wd4702") # warning C4702: unreachable code
    target_compile_options(glsl-optimizer PRIVATE "/wd4706") # warning C4706: assignment within conditional expression
    target_compile_options(glsl-optimizer PRIVATE "/wd4996") # warning C4996: 'strdup': The POSIX name for this item is deprecated. Instead, use the ISO C++ conformant name: _strdup.
else()
    target_compile_options(glsl-optimizer PRIVATE "-fno-strict-aliasing") # glsl-optimizer has bugs if strict aliasing is used.
    target_compile_options(glsl-optimizer PRIVATE "-Wno-unused-parameter")
    target_compile_options(glsl-optimizer PRIVATE "-Wno-shadow") # glsl-optimizer is full of -Wshadow warnings ignore it.
endif()

# ------------------ library for shader compiler: fcpp ----------------------

set(FCPP_DIR "bgfx/3rdparty/fcpp")

file(GLOB FCPP_SRC
    "${FCPP_DIR}/*.h"
    "${FCPP_DIR}/cpp*.c"
)

add_library(fcpp STATIC ${FCPP_SRC})

target_compile_definitions(fcpp PRIVATE
		NINCLUDE=64
		NWORK=65536
		NBUFF=65536
		OLD_PREPROCESSOR=0
)

if(MSVC)
    target_compile_options(fcpp PRIVATE "/wd4055") # warning C4055: 'type cast': from data pointer 'void *' to function pointer 'void (__cdecl *)(char *,void *)'
    target_compile_options(fcpp PRIVATE "/wd4244") # warning C4244: '=': conversion from 'const flex_int32_t' to 'YY_CHAR', possible loss of data
    target_compile_options(fcpp PRIVATE "/wd4701") # warning C4701: potentially uninitialized local variable 'lower' used
    target_compile_options(fcpp PRIVATE "/wd4706") # warning C4706: assignment within conditional expression
else()
    target_compile_options(fcpp PRIVATE "-Wno-implicit-fallthrough")
endif()

# ------------------ shader compiler: shaderc ----------------------
file(GLOB SHADERC_SRC
    "bgfx/tools/shaderc/*.cpp"
    "bgfx/tools/shaderc/*.h"
    "bgfx/src/vertexdecl.*"
    "bgfx/src/shader_spirv.*"
)

add_executable(shaderc ${SHADERC_SRC})

target_include_directories(shaderc PUBLIC "bx/include")
target_include_directories(shaderc PUBLIC "bimg/include")
target_include_directories(shaderc PUBLIC "bgfx/include")
target_include_directories(shaderc PUBLIC "bgfx/3rdparty/dxsdk/include")
target_include_directories(shaderc PUBLIC "${FCPP_DIR}")
target_include_directories(shaderc PUBLIC "bgfx/3rdparty/glslang/glslang/Public")
target_include_directories(shaderc PUBLIC "bgfx/3rdparty/glslang/glslang/Include")
target_include_directories(shaderc PUBLIC "bgfx/3rdparty/glslang")
target_include_directories(shaderc PUBLIC "${GLSL_OPTIMIZER}/include")
target_include_directories(shaderc PUBLIC "${GLSL_OPTIMIZER}/src/glsl")

target_link_libraries(shaderc bx fcpp glslang glsl-optimizer)

if(MSVC)
    target_include_directories(shaderc PUBLIC "${GLSL_OPTIMIZER}/include/c99")
    target_link_libraries(shaderc psapi)
else()
    target_link_libraries(shaderc pthread)
endif()
