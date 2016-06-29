cmake_minimum_required(VERSION 3.1)
project(bgfx)

if(CMAKE_BUILD_TYPE MATCHES DEBUG)
	add_definitions(-DBGFX_CONFIG_DEBUG=1)
endif()

# -------------------- 3rd party -----------------------
include_directories(3rdparty/khronos)

file(GLOB BGFX_3RD_PARTY
    "./bgfx/3rdparty/edtaa3/*.cpp"
    "./bgfx/3rdparty/etc1/*.cpp"
    "./bgfx/3rdparty/etc2/*.cpp"
    "./bgfx/3rdparty/fcpp/*.cpp"
    "./bgfx/3rdparty/ib-compress/*.cpp"
    "./bgfx/3rdparty/libsquish/*.cpp"
    "./bgfx/3rdparty/ocornut-imgui/*.cpp"
    "./bgfx/3rdparty/stb/*.cpp"
    )

add_library(bgfx3rdParty STATIC ${BGFX_3RD_PARTY})

set_target_properties(bgfx3rdParty PROPERTIES LINKER_LANGUAGE C)

target_include_directories(bgfx3rdParty PUBLIC bgfx/3rdparty/khronos)
target_include_directories(bgfx3rdParty PUBLIC bgfx/3rdparty)
target_include_directories(bgfx3rdParty PUBLIC bx/include)

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
file(GLOB BGFX_SRC
    "./bgfx/src/amalgamated.cpp")
    

add_library(bgfx STATIC ${BGFX_SRC})
set_target_properties(bgfx PROPERTIES LINKER_LANGUAGE C)

if(UNIX)
    target_link_libraries(bgfx X11 GL pthread dl)
endif()


target_include_directories(bgfx PUBLIC bgfx/include)
target_include_directories(bgfx PUBLIC bx/include)
target_link_libraries(bgfx bgfx3rdParty)
    

# ------------------ Common library ----------------------
file(GLOB COMMON_SRC
    "./bgfx/examples/common/*.cpp"
    "./bgfx/examples/common/debugdraw/*.cpp"
    "./bgfx/examples/common/entry/*.cpp"
    "./bgfx/examples/common/font/*.cpp"
    "./bgfx/examples/common/imgui/*.cpp"
    "./bgfx/examples/common/nanovg/*.cpp")
    
add_library(bgfx_common STATIC ${COMMON_SRC})
target_link_libraries(bgfx_common bgfx bgfx3rdParty)
set_target_properties(bgfx_common PROPERTIES LINKER_LANGUAGE C)

target_include_directories(bgfx_common PUBLIC bgfx/examples/common)
target_include_directories(bgfx_common PUBLIC bgfx/examples)

# ---------------------- Examples ------------------------
set(BGFX_LIBS bgfx bgfx_common bgfx3rdParty)

add_executable(ex_00_helloworld "bgfx/examples/00-helloworld/helloworld.cpp")
target_link_libraries(ex_00_helloworld ${BGFX_LIBS})

file(GLOB EX_09_HDR_SRC "bgfx/examples/09-hdr/*.cpp")
add_executable(ex_09_hdr ${EX_09_HDR_SRC})
target_link_libraries(ex_09_hdr ${BGFX_LIBS})



    

# ----------------- Example run-script -------------------
CONFIGURE_FILE(
    ${CMAKE_CURRENT_SOURCE_DIR}/runex.sh.in
    ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/runex.sh
)

# Mark as executable
file(COPY ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/runex.sh
  DESTINATION ${CMAKE_BINARY_DIR}
  FILE_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ
  GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
  
