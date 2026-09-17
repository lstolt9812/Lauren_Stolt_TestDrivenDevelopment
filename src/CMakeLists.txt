add_library(myLibrary STATIC)
target_sources(myLibrary
    PRIVATE
        myLibrary.cpp
    PUBLIC
        myLibrary.hpp
)
target_compile_features(myLibrary PRIVATE cxx_std_20)

target_include_directories(myLibrary
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
)

