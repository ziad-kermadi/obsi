
# **CMake Cheat Sheet for C++ Interviews**

CMake is a powerful build system widely used in **C++ projects**. Below is a **concise but comprehensive cheat sheet** covering the **most important concepts for an interview**.

---

## **1. Basic CMake Structure**
A **CMakeLists.txt** file is used to **configure and build** a C++ project.

```cmake
cmake_minimum_required(VERSION 3.10)  # Minimum required CMake version
project(MyProject)                    # Define project name

add_executable(my_program main.cpp)    # Create an executable from source files
```

---

## **2. Running CMake**
### **Step-by-Step Build Process**
```bash
mkdir build && cd build   # Create and enter the build directory
cmake ..                  # Generate Makefiles
make                      # Compile the project
./my_program              # Run the output executable
```

- `cmake ..` → Runs CMake to generate **Makefiles/Ninja/Visual Studio projects**.
- `make` → Builds the program using the generated files.

---

## **3. Setting C++ Standard**
Specify the required **C++ version**:

```cmake
set(CMAKE_CXX_STANDARD 17)       # Use C++17
set(CMAKE_CXX_STANDARD_REQUIRED ON)  # Ensure the standard is enforced
set(CMAKE_CXX_EXTENSIONS OFF)    # Disable compiler-specific extensions
```

✅ **Ensures portability** by enforcing strict C++ standard usage.

---

## **4. Organizing Multi-File Projects**
For **multiple source files**, list them or use `aux_source_directory()`:

```cmake
add_executable(my_program main.cpp utils.cpp math.cpp)
```
or
```cmake
file(GLOB SRC_FILES "src/*.cpp")   # Automatically get all .cpp files
add_executable(my_program ${SRC_FILES})
```

✅ **Automates file management**, preventing manual updates.

---

## **5. Adding Include Directories**
If your headers are in a separate **"include"** directory:

```cmake
include_directories(include)  
target_include_directories(my_program PRIVATE include)
```

✅ **Ensures compiler finds header files** during compilation.

---

## **6. Linking External Libraries**
### **(i) Static & Shared Libraries**
```cmake
target_link_libraries(my_program PRIVATE mylib)
```
or if using a full path:
```cmake
target_link_libraries(my_program PRIVATE /path/to/mylib.a) # Static
target_link_libraries(my_program PRIVATE /path/to/mylib.so) # Shared
```

### **(ii) Finding System Libraries**
```cmake
find_package(OpenGL REQUIRED)
target_link_libraries(my_program PRIVATE OpenGL::GL)
```

✅ **Ensures portability** by dynamically linking to system-installed libraries.

---

## **7. Using `add_subdirectory()` for Multi-Module Projects**
For a structured project:
```
project_root/
│── CMakeLists.txt   # Main project config
│── src/
│   ├── CMakeLists.txt   # Submodule CMakeLists
│   ├── module1.cpp
│   ├── module2.cpp
```
Main `CMakeLists.txt`:
```cmake
add_subdirectory(src)
```
`src/CMakeLists.txt`:
```cmake
add_library(my_library module1.cpp module2.cpp)
target_include_directories(my_library PUBLIC ../include)
```

✅ **Encapsulates modular projects** in maintainable **subdirectories**.

---

## **8. Build Types & Optimization**
Control **debug vs release** builds:

```cmake
set(CMAKE_BUILD_TYPE Release)   # Default to Release mode
```
To enable **manual selection**:
```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake -DCMAKE_BUILD_TYPE=Release ..
```

| **Build Type**  | **Optimization Level** |
|----------------|----------------------|
| **Debug**      | No optimization (`-O0`), includes debug symbols |
| **Release**    | Optimized (`-O3`), no debug symbols |
| **RelWithDebInfo** | Optimized with debug info (`-O2 -g`) |
| **MinSizeRel** | Optimized for small binary size |

✅ **Improves performance** by optimizing production builds.

---

## **9. Defining Compiler Flags**
Manually set **compiler flags** for debugging or optimization:

```cmake
add_compile_options(-Wall -Wextra -pedantic)  # Enable warnings
```
or:
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -O2")
```

✅ **Ensures cleaner, more optimized code** with stricter warnings.

---

## **10. Running Unit Tests in CMake**
Use **Google Test** with CMake:

```cmake
enable_testing()
add_subdirectory(tests)
add_test(NAME MyTest COMMAND my_test_executable)
```
Run tests:
```bash
ctest --output-on-failure
```

✅ **Automates testing** for continuous integration workflows.

---

## **11. Using `FetchContent` for Dependencies (Modern CMake)**
Instead of **manually downloading libraries**, use `FetchContent`:

```cmake
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG release-1.12.1
)
FetchContent_MakeAvailable(googletest)
target_link_libraries(my_program PRIVATE gtest_main)
```

✅ **Manages dependencies** without requiring external package managers.

---

## **12. Installing & Exporting a C++ Library**
To **install and export** a library:

```cmake
install(TARGETS my_library DESTINATION lib)
install(FILES my_header.h DESTINATION include)
export(TARGETS my_library FILE my_library-config.cmake)
```
After installation:
```bash
cmake --install .
```

✅ **Creates a reusable library** for system-wide use.

---

## **13. Using `find_package()` with External Libraries**
For built-in packages:

```cmake
find_package(Threads REQUIRED)
target_link_libraries(my_program PRIVATE Threads::Threads)
```

For **custom third-party libraries**:

```cmake
find_package(MyLibrary REQUIRED)
target_link_libraries(my_program PRIVATE MyLibrary::MyLibrary)
```

✅ **Dynamically links** libraries without manual path setting.

---

## **14. Generating Compilation Database for IDEs**
For **better auto-completion** in VSCode & CLion:

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

Run:
```bash
ln -s build/compile_commands.json .
```
✅ **Enables smart IDE features** like IntelliSense.

---

## **15. CMake Summary Table**

| **Feature**         | **CMake Command** | **Description** |
|---------------------|------------------|----------------|
| **Project Setup** | `project(MyProject)` | Defines project name |
| **C++ Standard** | `set(CMAKE_CXX_STANDARD 17)` | Ensures C++17 compilation |
| **Executable** | `add_executable(my_program main.cpp)` | Creates an executable |
| **Include Paths** | `target_include_directories()` | Adds header file directories |
| **Linking Libraries** | `target_link_libraries(my_program PRIVATE lib)` | Links static/shared libraries |
| **Find System Packages** | `find_package(OpenGL REQUIRED)` | Finds and links system libraries |
| **Subdirectories** | `add_subdirectory(src)` | Adds a subproject/module |
| **Compiler Flags** | `set(CMAKE_CXX_FLAGS "-Wall -Wextra")` | Adds warning flags |
| **Build Type** | `set(CMAKE_BUILD_TYPE Release)` | Debug, Release, etc. |
| **Testing** | `enable_testing()` | Enables unit testing |
| **Fetching Dependencies** | `FetchContent_Declare()` | Downloads dependencies from GitHub |
| **Installation** | `install(TARGETS my_library DESTINATION lib)` | Installs libraries for reuse |

---

## **16. Final Takeaways**
✅ **Learn `add_executable()`, `target_link_libraries()`, and `find_package()` first**.  
✅ **Understand how to manage dependencies** using `FetchContent` or `find_package()`.  
✅ **Use `CMAKE_BUILD_TYPE` for debugging vs optimized builds**.  
✅ **Practice writing `CMakeLists.txt` from scratch** for small projects.  
✅ **Know how to link libraries & manage headers** using `target_include_directories()`.  

🚀 **This CMake cheat sheet covers the essential knowledge needed for C++ interviews! Let me know if you need more in-depth details!** 🚀