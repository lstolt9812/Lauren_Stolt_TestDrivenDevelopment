cmake_minimum_required(VERSION 3.23)

project(myLibrary LANGUAGES CXX VERSION 1.0.0)

# ----------------------------
# GoogleTest
# ----------------------------

include(FetchContent)

FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/heads/main.zip
)

# Required for Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(googletest)

# ----------------------------

add_subdirectory(src)
add_subdirectory(tests)
