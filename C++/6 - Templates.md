
# **C++ Templates Cheat Sheet**

Templates in C++ allow you to write **generic and reusable code** for **functions, classes, and aliases** without specifying a fixed data type.

---

## **1. Function Templates**
Used to create **generic functions** that work with any data type.

### **Basic Function Template**
```cpp
template <typename T>
T maxValue(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    cout << maxValue(5, 10);    // Works with int
    cout << maxValue(3.5, 2.1); // Works with double
}
```
✔ **Reduces function overloading** for multiple types.

### **Function Template with Multiple Types**
```cpp
template <typename T1, typename T2>
void printPair(T1 a, T2 b) {
    cout << a << " - " << b << endl;
}

printPair(5, "Hello");  // int and string
printPair(3.5, 2);      // double and int
```

---

## **2. Class Templates**
Used to create **generic classes** where the data type is a parameter.

### **Basic Class Template**
```cpp
template <typename T>
class Box {
    T value;
public:
    Box(T v) : value(v) {}
    void show() { cout << "Value: " << value << endl; }
};

int main() {
    Box<int> intBox(10);
    Box<double> doubleBox(5.5);
    
    intBox.show();     // Output: Value: 10
    doubleBox.show();  // Output: Value: 5.5
}
```

### **Class Template with Multiple Types**
```cpp
template <typename T1, typename T2>
class Pair {
    T1 first;
    T2 second;
public:
    Pair(T1 f, T2 s) : first(f), second(s) {}
    void show() { cout << first << " - " << second << endl; }
};

Pair<int, string> p(1, "One");
p.show();  // Output: 1 - One
```

---

## **3. Template Specialization**
You can provide a **specialized version** of a template for a specific type.

```cpp
template <typename T>
class Example {
public:
    void show() { cout << "Generic Template" << endl; }
};

// Specialization for int
template <>
class Example<int> {
public:
    void show() { cout << "Specialized for int" << endl; }
};

Example<double> e1;
e1.show();  // Output: Generic Template

Example<int> e2;
e2.show();  // Output: Specialized for int
```

---

## **4. Template Defaults**
You can **set default types or values** in templates.

```cpp
template <typename T = int, typename U = double>
class Sample {
    T a;
    U b;
public:
    Sample(T x, U y) : a(x), b(y) {}
};

Sample<> obj(5, 3.14);   // Defaults: T = int, U = double
```

---

## **5. Variadic Templates (Multiple Arguments)**
Useful for handling **an arbitrary number of template parameters**.

```cpp
template <typename T, typename... Args>
void print(T first, Args... rest) {
    cout << first << " ";
    if constexpr (sizeof...(rest) > 0) print(rest...);
}

print(1, 2.5, "hello", 'A'); // Output: 1 2.5 hello A
```

---

## **6. Alias Templates (Type Aliases)**
Simplify **long template type names**.

```cpp
template <typename T>
using Vec = std::vector<T>;

Vec<int> numbers;  // std::vector<int>
```

---

## **7. Concept-Based Templates (C++20)**
Enforce **constraints** on template parameters.

```cpp
#include <concepts>

template <std::integral T> // Accepts only integral types
T square(T x) { return x * x; }

cout << square(4);  // ✅ Works
cout << square(4.5); // ❌ Compile error (not an integer)
```

---

## **8. Final Summary Table**

| **Feature**               | **Syntax** | **Use Case** |
|--------------------------|-----------|-------------|
| **Function Template** | `template <typename T> T func(T x);` | Generic function for multiple types |
| **Class Template** | `template <typename T> class C {}` | Generic classes |
| **Multiple Template Types** | `template <typename T1, typename T2> class C {}` | When a class or function needs more than one type |
| **Template Specialization** | `template <> class C<int> {}` | Customize behavior for a specific type |
| **Default Template Types** | `template <typename T=int>` | Provide default template type |
| **Variadic Templates** | `template <typename... Args>` | Handle multiple template parameters |
| **Alias Templates** | `template <typename T> using Alias = std::vector<T>;` | Shorten complex template names |
| **Concept-Based Templates** (C++20) | `template <std::integral T>` | Restrict template parameters |

---

🚀 **This cheat sheet covers everything you need to start using C++ templates efficiently! Let me know if you need more examples.** 🚀