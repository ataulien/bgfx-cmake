cmake_minimum_required(VERSION 3.1)
project(bgfx)

# -------------------- 3rd party -----------------------
include_directories(3rdparty/khronos)

file(GLOB BGFX_3RD_PARTY
    "bgfx/3rdparty/edtaa3/*.cpp"
    "bgfx/3rdparty/etc1/*.cpp"
    "bgfx/3rdparty/etc2/*.cpp"
    "bgfx/3rdparty/fcpp/*.cpp"
    "bgfx/3rdparty/ib-compress/*.cpp"
    "bgfx/3rdparty/libsquish/*.cpp"
    "bgfx/3rdparty/ocornut-imgui/*.cpp"
    "bgfx/3rdparty/stb/*.cpp"
    )

add_library(bgfx3rdParty STATIC ${BGFX_3RD_PARTY})
set_target_properties(bgfx3rdParty PROPERTIES LINKER_LANGUAGE C)
    
# ------------------ Main library ----------------------
file(GLOB BGFX_SRC
    "bgfx/src/amalgamated.cpp")
    
include_directories(bgfx/include)
include_directories(bx/include)
include_directories(bgfx/3rdparty/khronos)
include_directories(bgfx/3rdparty)
include_directories(bgfx/examples/common)
        
    
add_library(bgfx STATIC ${BGFX_SRC})
set_target_properties(bgfx PROPERTIES LINKER_LANGUAGE C)

if(UNIX)
    target_link_libraries(bgfx X11 GL pthread dl)
endif()

# ------------------ Common library ----------------------
file(GLOB COMMON_SRC
    "bgfx/examples/common/*.cpp"
    "bgfx/examples/common/debugdraw/*.cpp"
    "bgfx/examples/common/entry/*.cpp"
    "bgfx/examples/common/font/*.cpp"
    "bgfx/examples/common/imgui/*.cpp"
    "bgfx/examples/common/nanovg/*.cpp")
    
add_library(bgfx_common STATIC ${COMMON_SRC})
set_target_properties(bgfx_common PROPERTIES LINKER_LANGUAGE C)

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
  
