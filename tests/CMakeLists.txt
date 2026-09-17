include(GoogleTest)

add_executable(myLibrary_unittests)
target_sources(myLibrary_unittests
    PRIVATE
        mylibrary_unittests.cpp
)
target_compile_features(myLibrary_unittests PRIVATE cxx_std_20)
target_include_directories(myLibrary_unittests
    PRIVATE
        "${GTEST_INCLUDE_DIRS}"
)
target_link_libraries(myLibrary_unittests
    GTest::gtest_main
    myLibrary
)
