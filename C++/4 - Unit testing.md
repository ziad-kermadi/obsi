
# **C++ Unit Testing Cheat Sheet**

Unit testing in C++ ensures that individual **functions, methods, or classes** work correctly. The most popular testing frameworks for C++ include **Google Test (GTest)**, **Catch2**, and **Boost.Test**. Below is a **quick-reference cheat sheet** covering **setup, assertions, test structure, and best practices**.

---

## **1. Choosing a Testing Framework**
| **Framework**   | **Pros** | **Cons** |
|---------------|---------|---------|
| **Google Test (GTest)** | Most widely used, rich assertions, mock support | Requires compilation, not header-only |
| **Catch2** | Single-header, simple syntax, BDD support | Fewer built-in features than GTest |
| **Boost.Test** | Part of Boost, highly customizable | Heavier dependency, more complex setup |

---

## **2. Setting Up Google Test (GTest)**
### **Installation**
1. Install Google Test:
   ```bash
   sudo apt install libgtest-dev
   ```
   Or build from source:
   ```bash
   git clone https://github.com/google/googletest.git
   cd googletest
   mkdir build && cd build
   cmake ..
   make
   sudo make install
   ```

2. Link against `gtest` in CMake:
   ```cmake
   cmake_minimum_required(VERSION 3.10)
   project(MyTests)

   add_executable(test_runner test.cpp)
   find_package(GTest REQUIRED)
   target_link_libraries(test_runner GTest::GTest GTest::Main pthread)
   ```

---

## **3. Writing a Basic Test with Google Test**
```cpp
#include <gtest/gtest.h>

// Function to test
int add(int a, int b) { return a + b; }

// Test case
TEST(AdditionTest, HandlesPositiveNumbers) {
    EXPECT_EQ(add(2, 3), 5);
}

TEST(AdditionTest, HandlesNegativeNumbers) {
    EXPECT_EQ(add(-2, -3), -5);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

**Run the test:**
```bash
./test_runner
```

---

## **4. Common Google Test Assertions**
| **Assertion** | **Description** | **Example** |
|--------------|----------------|-------------|
| `EXPECT_EQ(a, b)` | Passes if `a == b` | `EXPECT_EQ(5, add(2,3));` |
| `EXPECT_NE(a, b)` | Passes if `a != b` | `EXPECT_NE(4, add(2,3));` |
| `EXPECT_LT(a, b)` | Passes if `a < b` | `EXPECT_LT(2, 3);` |
| `EXPECT_GT(a, b)` | Passes if `a > b` | `EXPECT_GT(5, 3);` |
| `EXPECT_TRUE(x)` | Passes if `x` is true | `EXPECT_TRUE(5 > 3);` |
| `EXPECT_FALSE(x)` | Passes if `x` is false | `EXPECT_FALSE(2 > 5);` |
| `EXPECT_THROW(statement, ExceptionType)` | Passes if `statement` throws `ExceptionType` | `EXPECT_THROW(throw std::runtime_error("error"), std::runtime_error);` |
| `EXPECT_NO_THROW(statement)` | Passes if `statement` does not throw | `EXPECT_NO_THROW(func());` |

- `EXPECT_*` → **Continues** running even if the test fails.
- `ASSERT_*` → **Stops** execution on failure.

---

## **5. Test Fixtures (Setup & Teardown)**
Use **test fixtures** when multiple tests need **common setup**.

```cpp
class MathTest : public ::testing::Test {
protected:
    int a, b;
    
    void SetUp() override {
        a = 10;
        b = 5;
    }
};

TEST_F(MathTest, AdditionTest) {
    EXPECT_EQ(a + b, 15);
}

TEST_F(MathTest, SubtractionTest) {
    EXPECT_EQ(a - b, 5);
}
```

---

## **6. Parameterized Tests**
Run the **same test** with **multiple inputs**.

```cpp
class MathParamTest : public ::testing::TestWithParam<std::tuple<int, int, int>> {};

TEST_P(MathParamTest, Addition) {
    auto [x, y, expected] = GetParam();
    EXPECT_EQ(add(x, y), expected);
}

INSTANTIATE_TEST_SUITE_P(
    MathTests, MathParamTest,
    ::testing::Values(
        std::make_tuple(2, 3, 5),
        std::make_tuple(-2, -3, -5),
        std::make_tuple(0, 0, 0)
    )
);
```

---

## **7. Mocking with Google Mock**
Mock objects are useful for testing **dependencies** without real implementations.

```cpp
#include <gmock/gmock.h>

class Database {
public:
    virtual int getData() = 0;
    virtual ~Database() = default;
};

class MockDatabase : public Database {
public:
    MOCK_METHOD(int, getData, (), (override));
};

TEST(MockTest, DatabaseTest) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, getData()).WillOnce(::testing::Return(42));

    EXPECT_EQ(mockDb.getData(), 42);
}
```

---

## **8. Testing Exceptions & Edge Cases**
**Checking Exception Handling**
```cpp
void riskyFunction(bool shouldThrow) {
    if (shouldThrow) throw std::runtime_error("Error occurred");
}

TEST(ExceptionTest, ThrowsWhenExpected) {
    EXPECT_THROW(riskyFunction(true), std::runtime_error);
}

TEST(ExceptionTest, DoesNotThrow) {
    EXPECT_NO_THROW(riskyFunction(false));
}
```

---

## **9. Catch2 Framework (Header-Only Alternative)**
If you prefer a **lightweight alternative**, **Catch2** is a great choice.

### **Setup:**
1. Download `catch.hpp`:
   ```bash
   wget https://raw.githubusercontent.com/catchorg/Catch2/main/single_include/catch2/catch.hpp
   ```
2. Include in your test:
   ```cpp
   #define CATCH_CONFIG_MAIN
   #include "catch.hpp"

   int add(int a, int b) { return a + b; }

   TEST_CASE("Addition works") {
       REQUIRE(add(2, 3) == 5);
   }
   ```

3. Compile & Run:
   ```bash
   g++ test.cpp -o test_runner && ./test_runner
   ```

---

## **10. Best Practices for C++ Unit Testing**
✅ **Write Tests Before Code** → Follow **TDD (Test-Driven Development)**.  
✅ **Test Edge Cases** → Include **empty inputs, large values, and boundary conditions**.  
✅ **Keep Tests Independent** → No dependencies between tests.  
✅ **Use Mocks & Fakes** → Replace **external dependencies** for reliable tests.  
✅ **Automate Tests** → Run tests **on every code change** using **CI/CD**.  
✅ **Use Fixtures** → Avoid **duplicate setup code** with `TEST_F`.  
✅ **Make Tests Fast** → **Minimize setup** time to make testing efficient.  

---

## **Final Summary Table**

| **Feature**      | **Google Test (GTest)** | **Catch2** | **Boost.Test** |
|-----------------|------------------|---------|------------|
| **Ease of Use** | ✅✅✅ | ✅✅ | ✅✅ |
| **Header-Only** | ❌ | ✅ | ❌ |
| **Assertions** | ✅✅✅ | ✅✅ | ✅✅✅ |
| **Mocking Support** | ✅ (via Google Mock) | ❌ | ❌ |
| **Parameterized Tests** | ✅ | ✅ | ✅ |
| **Performance** | ✅✅ | ✅✅✅ | ✅ |
| **CI/CD Integration** | ✅✅✅ | ✅✅ | ✅✅✅ |

---

## **Final Takeaways**
- **Google Test (GTest) is the most powerful** → Use it for professional projects.
- **Catch2 is great for quick and lightweight testing**.
- **Mocks (Google Mock) allow dependency isolation** for unit tests.
- **Automate testing** to improve **code reliability and maintainability**.

🚀 **Now you have everything you need to start unit testing in C++! Let me know if you need more details!** 🚀