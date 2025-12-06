


----------------
--------
----------

# **C++ Terminal Commands & Tools Cheat Sheet for Interviews**  

This cheat sheet covers the **most important terminal commands** and tools you need for **compiling, building, debugging, and running C++ code** in an interview.

---

## **1. Compiling & Running C++ Code**
### **GCC / Clang (Basic Compilation)**
| **Command** | **Description** |
|------------|----------------|
| `g++ main.cpp -o output` | Compile `main.cpp` into an executable named `output` |
| `./output` | Run the compiled executable |
| `g++ main.cpp -std=c++17 -o output` | Compile using **C++17 standard** |
| `g++ main.cpp -Wall -Wextra -o output` | Enable warnings (`-Wall` for common, `-Wextra` for additional) |
| `g++ main.cpp -g -o output` | Enable **debug symbols** for `gdb` debugging |
| `g++ main.cpp -O2 -o output` | Compile with **optimization level 2** (faster execution) |

### **Clang Compiler Alternative**
```bash
clang++ main.cpp -o output
./output
```
✅ **GCC and Clang are interchangeable**, but Clang provides **better diagnostics**.

---

## **2. Running C++ Code Without Manual Compilation**
### **Using `g++ -x c++ -` (Inline Execution)**
```bash
echo '#include <iostream> int main(){std::cout<<"Hello";}' | g++ -x c++ - && ./a.out
```
✅ **Runs a C++ one-liner** without creating source files.

### **Using `clang++` with `-std=c++20`**
```bash
clang++ -std=c++20 -o output main.cpp
```
✅ **Useful for C++20 features**.

---

## **3. Debugging Tools**
### **Using `gdb` (GNU Debugger)**
| **Command** | **Description** |
|------------|----------------|
| `g++ -g main.cpp -o output` | Compile with debug symbols |
| `gdb ./output` | Start debugging session |
| `break main` | Set a breakpoint at `main()` |
| `run` | Run the program in debug mode |
| `next` / `step` | Step through code execution |
| `print var` | Print the value of a variable |
| `backtrace` | Show the call stack |
| `quit` | Exit GDB |

---

### **Using `valgrind` (Memory Debugging)**
```bash
valgrind --leak-check=full ./output
```
✅ Detects **memory leaks and invalid memory accesses**.

---

## **4. Build Systems (Make, CMake, Ninja)**
### **Using `Make`**
| **Command** | **Description** |
|------------|----------------|
| `make` | Build using `Makefile` |
| `make clean` | Remove compiled binaries |
| `make -j4` | Build using 4 CPU threads (faster) |

#### **Example `Makefile`**
```make
CC = g++
CFLAGS = -Wall -Wextra -std=c++17
TARGET = my_program
SRCS = main.cpp utils.cpp

$(TARGET): $(SRCS)
	$(CC) $(CFLAGS) $(SRCS) -o $(TARGET)

clean:
	rm -f $(TARGET)
```
✅ **Run `make` to build automatically**.

---

### **Using CMake**
```bash
mkdir build && cd build
cmake .. && make
```
✅ **CMake is the standard for C++ projects**.

#### **Example `CMakeLists.txt`**
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)
set(CMAKE_CXX_STANDARD 17)

add_executable(my_program main.cpp)
```

---

## **5. Code Analysis & Linting**
### **Checking Code with `clang-tidy`**
```bash
clang-tidy main.cpp --fix-errors
```
✅ **Automatically suggests improvements** in your C++ code.

### **Checking Formatting with `clang-format`**
```bash
clang-format -i main.cpp
```
✅ **Auto-formats your C++ code**.

---

## **6. Profiling Performance**
### **Using `time` Command**
```bash
time ./output
```
✅ **Shows execution time** of the program.

### **Using `perf` (Linux Performance Profiler)**
```bash
perf stat ./output
```
✅ **Gives CPU usage, cache misses, and execution time**.

---

## **7. File Operations**
### **Viewing & Editing Files**
| **Command** | **Description** |
|------------|----------------|
| `cat file.cpp` | View the contents of a file |
| `nano file.cpp` | Open file in Nano editor |
| `vim file.cpp` | Open file in Vim editor |

### **Searching in Code Files**
```bash
grep "main" main.cpp
grep -rn "function_name" src/
```
✅ **Search for function definitions in multiple files**.

---

## **8. Checking & Killing Processes**
### **Finding Running C++ Programs**
```bash
ps aux | grep output
pgrep output
```

### **Killing a Process**
```bash
kill -9 <PID>
```
✅ **Stops a hanging C++ process**.

---

## **9. Running Tests**
### **Google Test Framework**
```bash
./test_runner --gtest_filter=TestSuiteName.*
```
✅ **Runs unit tests for C++ projects**.

### **Using `ctest` (For CMake Projects)**
```bash
ctest --output-on-failure
```
✅ **Runs tests automatically**.

---

## **10. Final Summary Table**
| **Task** | **Command** |
|---------|------------|
| **Compile** | `g++ main.cpp -o output` |
| **Compile with Debugging** | `g++ -g main.cpp -o output` |
| **Run Executable** | `./output` |
| **Check Warnings** | `g++ -Wall -Wextra main.cpp -o output` |
| **Use C++17/20** | `g++ -std=c++20 main.cpp -o output` |
| **Debug with GDB** | `gdb ./output` |
| **Check Memory Leaks** | `valgrind ./output` |
| **Use Makefile** | `make` |
| **Run CMake Build** | `cmake .. && make` |
| **Format Code** | `clang-format -i main.cpp` |
| **Lint Code** | `clang-tidy main.cpp --fix-errors` |
| **Run Tests (GTest)** | `./test_runner --gtest_filter=TestSuiteName.*` |
| **Check Execution Time** | `time ./output` |

---

## **11. Final Takeaways**
✅ **Know basic compilation (`g++ main.cpp -o output`)**.  
✅ **Understand `make` and `CMake` for project building**.  
✅ **Use `gdb` and `valgrind` for debugging**.  
✅ **Use `clang-format` and `clang-tidy` for clean code**.  
✅ **Run tests using `Google Test` or `ctest`**.  

🚀 **Master these commands to be well-prepared for any C++ coding interview!** 🚀
