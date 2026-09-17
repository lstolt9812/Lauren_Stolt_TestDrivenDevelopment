
# CMake Shared Library with GoogleTest

> **Note:** The indentation and formatting of the CMake calls in this file are intended to make the file easier to read and understand. They are not required by CMake.


This example creates:

- A shared library
- An executable containing GoogleTest unit tests
- Installation rules for the library and headers
- CMake package configuration files that make the library easier for other CMake projects to use

---

## 1. Specify the Minimum CMake Version

Specifies the minimum CMake version required to understand this file.

```cmake
cmake_minimum_required(VERSION 3.23)
```

---

## 2. Define the Project

Define the project name, language, and version.

The project name can be accessed with the variable `${PROJECT_NAME}`.

`VERSION` specifies the version of the library using the format:

```text
MAJOR.MINOR.PATCH
```

```cmake
project(myLibrary LANGUAGES CXX VERSION 1.0.0)
```

---

## 3. Generate a Version Header

The following creates a header file containing the library version information.

This header file can then be included by source files that need access to the version numbers.

```cmake
configure_file(myLibraryConfig.h.in myLibraryConfig.h)
```

### `myLibraryConfig.h.in`

`myLibraryConfig.h.in` must be created by us and placed in the source directory.

It should contain:

```c
#define myLibrary_VERSION_MAJOR @myLibrary_VERSION_MAJOR@
#define myLibrary_VERSION_MINOR @myLibrary_VERSION_MINOR@
#define myLibrary_VERSION_PATCH @myLibrary_VERSION_PATCH@
```

> **Important:** Make sure to remove the `#` characters from the beginning of the lines if they were included as comments in an example.

CMake replaces the `@VARIABLE@` expressions with the corresponding project version values when it generates `myLibraryConfig.h`.

---

## 4. Configure Shared Libraries on Windows

The following code is used when creating a shared library on Windows.

It automatically adds the necessary export declarations for classes and functions.

```cmake
if (MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()
```

---

## 5. Setup GoogleTest

### Fetch googletest
```cmake
include(FetchContent)

FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/heads/main.zip
)
```
### Required for Windows
```cmake
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
```
### Make googeltest available
```cmake
FetchContent_MakeAvailable(googletest)
```
---

## 6. Create the Shared Library

This defines a shared library target.

The general form is:

```cmake
add_library("library_name" type)
```

Additional information can be added to the target using the various `target_*` functions.

For a static library, use `STATIC` instead of `SHARED`.

```cmake
add_library(myLibrary SHARED)
```

---

## 7. Specify the Library Sources

The following sets the source files for the library.

The identifiers `PRIVATE`, `INTERFACE`, and `PUBLIC` specify the visibility of the source files.

### Visibility

* **PRIVATE** — Sources are only used by the current target.
* **INTERFACE** — Sources are not used by the current target but are used by targets that depend on it.
* **PUBLIC** — Sources are used by both the current target and targets that depend on it.

`FILE_SET HEADERS` defines a set of files named `HEADERS`.

Currently, `HEADERS` is the only valid choice for the file-set name.

`BASE_DIRS ${PROJECT_SOURCE_DIR}` is used to change the absolute path of header files in the file set to a relative path by removing `${PROJECT_SOURCE_DIR}` from the beginning of each file path.

`FILES` specifies the files included in the file set.

For example:

```text
/home/userName/projectName/subdirectory/mylibrary.hpp
```

can become:

```text
/cmake/install/prefix/include/subdirectory/mylibrary.hpp
```

when installed.

```cmake
target_sources(myLibrary
    PRIVATE
        myLibrary.cpp
    PUBLIC
        FILE_SET HEADERS
            BASE_DIRS ${PROJECT_SOURCE_DIR}
            FILES myLibrary.hpp
)
```

---

## 8. Link Library Dependencies

If this library depended on another library, the following would be needed.

This specifies the additional libraries that this library needs to be linked with.

```cmake
target_link_libraries(myLibrary
    PRIVATE
        someDependedOnLibrary
)
```

For this example, the command is commented out because the library does not have any additional dependencies.

---

## 9. Add the Build Directory as an Include Directory

The following adds the build directory as an include directory.

This is done in this example so that the source code can access the generated `myLibraryConfig.h` file containing the version information.

```cmake
target_include_directories(myLibrary
    PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}
)
```

---

## 10. Create the Unit Test Executable

Define an executable with the given name.

```cmake
add_executable(myLibrary_unittests)
```

---

## 11. Specify the Unit Test Source

This sets the source file for the executable.

```cmake
target_sources(
    myLibrary_unittests
    PRIVATE
        mylibrary_unittests.cpp
)
```

---

## 12. Specify GoogleTest Include Directories

This sets the directories where CMake should look for include files.

`${GTEST_INCLUDE_DIRS}` is defined by the `find_package(GTest)` call.

`PRIVATE` indicates that these include directories are only needed by this executable target.

```cmake
target_include_directories(
    myLibrary_unittests
    PRIVATE
        "${GTEST_INCLUDE_DIRS}"
)
```

---

## 13. Link the Unit Tests

This specifies the libraries that need to be linked with the unit test executable.

```cmake
target_link_libraries(myLibrary_unittests
    PRIVATE
        GTest::gtest_main 
        myLibrary
)
```

The unit test executable therefore links against:

* `GTest::gtest_main`
* `myLibrary`

---

# Installing the Library

The next section is about installing the library and making it easier for other projects to use.

Installation is performed with:

```bash
cmake --install
```

The `CMAKE_INSTALL_PREFIX` variable determines the installation location by default, but this can be overridden by specifying the installation destination.

In an IDE, you can also build the installation target.

---

## 14. Install the Library and Headers

The following `install(TARGETS)` and `install(EXPORT)` commands work together to install:

* The `myLibrary` target
* Its headers
* A CMake file that helps other projects use `myLibrary`

The `EXPORT` defines an export named `myLibraryTargets` for the `myLibrary` target.

The following installation locations are used:

| Item             | Installation Directory |
| ---------------- | ---------------------- |
| Header files     | `include`              |
| Shared libraries | `lib`                  |
| Static libraries | `lib`                  |
| Executables      | `bin`                  |

```cmake
install(TARGETS myLibrary
    EXPORT myLibraryTargets
    FILE_SET HEADERS
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)
```

---

# CMake Package Configuration

The following section creates CMake files that make it easier for other projects to use the library.

---

## 15. Install the CMake Target Export

`install(EXPORT)` creates a CMake file that makes it easier for other CMake projects to use our library.

### `FILE`

Specifies the name of the CMake file:

```text
myLibraryTargets.cmake
```

The file should follow the naming convention:

```text
<filename>Targets.cmake
```

### `NAMESPACE`

Defines the library namespace.

The library can then be used with `target_link_libraries()` using the namespace prefix:

```cmake
target_link_libraries(myexe
    PRIVATE
        myLibrary::myLibrary
)
```

### `DESTINATION`

Specifies where the `Targets.cmake` file will be installed.

```cmake
install(EXPORT myLibraryTargets
    FILE myLibraryTargets.cmake
    NAMESPACE myLibrary::
    DESTINATION lib/cmake/myLibrary
)
```

---

# Library Version Information

The following section creates a CMake file containing the version number of the library.

This is used with:

```cmake
find_package(<name> <version>)
```

when another project wants to use this library.

If a version is specified, `find_package()` can make sure that the library found is compatible with the requested version.

---

## 16. Include CMake Package Configuration Helpers

`CMakePackageConfigHelpers` provides additional helper functions for creating CMake package configuration files.

```cmake
include(CMakePackageConfigHelpers)
```

---

## 17. Create the Package Version File

The `write_basic_package_version_file()` function comes from `CMakePackageConfigHelpers`.

This writes a CMake file containing the version number of our library.

The file should have the form:

```text
<name>ConfigVersion.cmake
```

The version can be specified directly, or `${myLibrary_VERSION}` can be used.

`${myLibrary_VERSION}` comes from:

```cmake
project(
    <name>
    VERSION <version number>
)
```

### `COMPATIBILITY`

`AnyNewerVersion` specifies that this version or any newer compatible version can be used.

```cmake
write_basic_package_version_file(
    "myLibraryConfigVersion.cmake"
    VERSION ${myLibrary_VERSION}
    COMPATIBILITY AnyNewerVersion
)
```

---

## 18. Install the Package Configuration Files

The following installs the configuration and version files into the correct location.

The version CMake file is created in the build directory.

The build directory can be accessed using:

```cmake
${CMAKE_CURRENT_BINARY_DIR}
```

The `myLibraryConfig.cmake` file must be created by us and placed in the source directory.

It should contain:

```cmake
include(CMakeFindDependencyMacro)
#find_dependency(xxx 2.0)
include(${CMAKE_CURRENT_LIST_DIR}/myLibraryTargets.cmake)
```

> **Important:** Make sure to remove the `#` from lines that are intended to be active CMake commands.

The following line is commented out because this example has no dependencies:

```cmake
#find_dependency(xxx 2.0)
```

If the library depended on another library, that dependency would be specified here.

---

### Installation Files

The `FILES` argument specifies the files to install.

The two files come from different locations:

| File                           | Location         |
| ------------------------------ | ---------------- |
| `myLibraryConfig.cmake`        | Source directory |
| `myLibraryConfigVersion.cmake` | Build directory  |

`DESTINATION` specifies the installation location.

It is best practice to install these files in:

```text
lib/cmake/<target name>
```

```cmake
install(
    FILES
        "myLibraryConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/myLibraryConfigVersion.cmake"
    DESTINATION lib/cmake/myLibrary
)
```

---

# Complete CMakeLists.txt

The following is the complete CMake file assembled from the sections above.

```cmake
# Note: The indenting and format of the CMake calls in the file
# are to make it easier to read and understand. They are not
# required by CMake.
# The comments are valid as of CMake version 3.23.

cmake_minimum_required(VERSION 3.23)

project(myLibrary LANGUAGES CXX VERSION 1.0.0)

configure_file(myLibraryConfig.h.in myLibraryConfig.h)

if (MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

find_package(GTest REQUIRED)

add_library(myLibrary SHARED)

target_sources(myLibrary
    PRIVATE
        myLibrary.cpp
    PUBLIC
        FILE_SET HEADERS
            BASE_DIRS ${PROJECT_SOURCE_DIR}
            FILES myLibrary.hpp
)

# If the library depended on another library:
#
# target_link_libraries(myLibrary
#     PRIVATE
#         someDependedOnLibrary
# )

target_include_directories(myLibrary
    PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}
)

add_executable(myLibrary_unittests)

target_sources(
    myLibrary_unittests
    PRIVATE
        mylibrary_unittests.cpp
)

target_include_directories(
    myLibrary_unittests
    PRIVATE
        "${GTEST_INCLUDE_DIRS}"
)

target_link_libraries(myLibrary_unittests
    PRIVATE
        GTest::GTest
        GTest::Main
        myLibrary
)

install(TARGETS myLibrary
    EXPORT myLibraryTargets
    FILE_SET HEADERS
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(EXPORT myLibraryTargets
    FILE myLibraryTargets.cmake
    NAMESPACE myLibrary::
    DESTINATION lib/cmake/myLibrary
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "myLibraryConfigVersion.cmake"
    VERSION ${myLibrary_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(
    FILES
        "myLibraryConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/myLibraryConfigVersion.cmake"
    DESTINATION lib/cmake/myLibrary
)
```

# Required Supporting Files

The example also requires the following files:

```text
project/
├── CMakeLists.txt
├── myLibrary.cpp
├── myLibrary.hpp
├── myLibraryConfig.h.in
├── myLibraryConfig.cmake
└── mylibrary_unittests.cpp
```

### `myLibraryConfig.h.in`

```c
#define myLibrary_VERSION_MAJOR @myLibrary_VERSION_MAJOR@
#define myLibrary_VERSION_MINOR @myLibrary_VERSION_MINOR@
#define myLibrary_VERSION_PATCH @myLibrary_VERSION_PATCH@
```

### `myLibraryConfig.cmake`

```cmake
include(CMakeFindDependencyMacro)

# Add dependencies here if necessary.
# find_dependency(xxx 2.0)

include(${CMAKE_CURRENT_LIST_DIR}/myLibraryTargets.cmake)
```

```
```
