cmake_minimum_required(VERSION 3.19)
project(whisperplot C CXX ASM)
set(CMAKE_POSITION_INDEPENDENT_CODE FALSE)

set(CMAKE_CXX_STANDARD 20)

include(FetchContent)

IF (CMAKE_BUILD_TYPE STREQUAL "Release")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Ofast -flto")
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Ofast -flto")
    set (CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -Ofast -flto")
    #set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNDEBUG")
    #set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG")
ELSE()
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0")
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0")
ENDIF()

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -g")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -g")

#set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer -fsanitize=thread")
#set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-omit-frame-pointer -fsanitize=thread")
#set (CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=thread")

#set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer -fsanitize=address -fsanitize=undefined")
#set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-omit-frame-pointer -fsanitize=address -fsanitize=undefined")
#set (CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=address -fsanitize=undefined")

FetchContent_Declare(
        cxxopts
        GIT_REPOSITORY https://github.com/jarro2783/cxxopts.git
        GIT_TAG        v2.2.1
)
FetchContent_MakeAvailable(cxxopts)

SET(BUILD_BLS_PYTHON_BINDINGS FALSE)
FetchContent_Declare(
        bls-signatures
        GIT_REPOSITORY https://github.com/computer-whisperer/bls-signatures.git
        GIT_TAG main
)
FetchContent_MakeAvailable(bls-signatures)

FetchContent_Declare(
        bitpacker
        GIT_REPOSITORY https://github.com/CrustyAuklet/bitpacker.git
)
FetchContent_MakeAvailable(bitpacker)

set(FSE_LIB chiapos_inc/FiniteStateEntropy/lib)
set(FSE_FILES
        ${FSE_LIB}/fse_compress.c
        ${FSE_LIB}/fse_decompress.c
        ${FSE_LIB}/entropy_common.c
        ${FSE_LIB}/hist.c
        inc/thread_mgr.hpp inc/explicit_templates.hpp)
add_library(fse ${FSE_FILES})

set(BLAKE3_SRC
        chiapos_inc/b3/blake3.c
        chiapos_inc/b3/blake3_portable.c
        chiapos_inc/b3/blake3_dispatch.c
        chiapos_inc/b3/blake3_avx2_x86-64_unix.S
        chiapos_inc/b3/blake3_avx512_x86-64_unix.S
        chiapos_inc/b3/blake3_sse41_x86-64_unix.S
        )

include_directories(
        inc
        chiapos_inc
        ${cxxopts_SOURCE_DIR}/include
        ${bitpacker_SOURCE_DIR}/include/bitpacker
        ${bls-signatures_SOURCE_DIR}/src
        ${bls-signatures_SOURCE_DIR}/build
        ${bls-signatures_SOURCE_DIR}/../relic-src/include
        ${bls-signatures_SOURCE_DIR}/../relic-build/include)

add_executable(whisperplot
        src/main.cpp
        src/create.cpp
        src/prove.cpp
        src/status_update.cpp
        src/encoding.cpp
        src/calculate_bucket.cpp
        chiapos_inc/chacha8.c
        ${BLAKE3_SRC})

find_package(Threads REQUIRED)
add_library(uint128 STATIC chiapos_inc/uint128_t/uint128_t.cpp)
target_include_directories(uint128 PUBLIC chiapos_inc/uint128_t)


target_link_libraries(whisperplot fse numa stdc++fs Threads::Threads bls)


add_executable(outputStreamCountTest experiments/outputStreamCountTest.cpp)
target_link_libraries(outputStreamCountTest m)