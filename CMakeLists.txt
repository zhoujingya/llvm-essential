cmake_minimum_required(VERSION 3.22)
project(compiler-essential
        VERSION 1.0.0)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_LINKER mold)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(add-subdir-recursive)

find_package(LLVM REQUIRED)
include(AddLLVM)
message("LLVM version: " ${LLVM_PACKAGE_VERSION})
message("LLVM include directory: " ${LLVM_INCLUDE_DIRS})
message("LLVM library directory: " ${LLVM_LIBRARY_DIRS})
message("LLVM install path: " ${LLVM_INSTALL_PREFIX})
set(LLVM_DIR ${LLVM_INSTALL_PREFIX})
add_definitions(${LLVM_DEFINITIONS})

# Find clang path
# Solution1: set search pat
set (search_paths
 ${LLVM_INSTALL_PREFIX}
 ${LLVM_INSTALL_PREFIX}/lib/cmake
 ${LLVM_INSTALL_PREFIX}/lib/cmake/llvm
 ${LLVM_INSTALL_PREFIX}/lib/cmake/clang
 ${LLVM_INSTALL_PREFIX}/share/clang/cmake
 ${LLVM_INSTALL_PREFIX}/share/llvm/cmake
)
# find_package(LLVM REQUIRED CONFIG
#  PATHS ${search_paths}
#  NO_DEFAULT_PATH)
# Solution 2:
find_package(Clang REQUIRED CONFIG
    PATHS "${search_paths}")

# Set runtime bin/lib output path
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Very very important when importing LLVM library
if(NOT LLVM_ENABLE_RTTI)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
endif()

# Import gtest
find_package(GTest)
if( NOT GTest_FOUND)
  include(FetchContent)
  FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/03597a01ee50ed33e9dfd640b249b4be3799d395.zip
  )
  # For Windows: Prevent overriding the parent project's compiler/linker settings
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
  FetchContent_MakeAvailable(googletest)
endif()
include(GoogleTest)

enable_testing()
add_subdirectory(src)
add_subdirectory(test)
