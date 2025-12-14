You really are determined to master the language designed by people who hate humans. Respect again.

You said: _more C++ than finance_, so I’ll keep examples mostly “plain” and only vaguely quant-flavored when helpful, but focus hard on **core language concepts** interviewers grill you on.

We’ll continue numbering.

---

## Q21 – Lvalues, Rvalues & Overloads

**Question:**  
Explain the difference between lvalues and rvalues, and implement three overloads:

```cpp
void log_value(const std::string& s);      // lvalue
void log_value(std::string&& s);           // rvalue
void log_value(const char* s);             // C-string
```

Show how overload resolution behaves with:

```cpp
std::string a = "hello";
log_value(a);
log_value(std::string("tmp"));
log_value("literal");
```

---

### What they’re testing

- Do you _actually_ get lvalues/rvalues, or do you just say “std::move makes it faster” like half of LinkedIn.
    
- Can you reason about overload resolution and reference binding.
    

---

### Answer & Code

```cpp
#include <iostream>
#include <string>

void log_value(const std::string& s) {
    std::cout << "lvalue std::string: " << s << "\n";
}

void log_value(std::string&& s) {
    std::cout << "rvalue std::string: " << s << "\n";
}

void log_value(const char* s) {
    std::cout << "C-string literal: " << s << "\n";
}

int main() {
    std::string a = "hello";

    log_value(a);                    // lvalue overload
    log_value(std::string("tmp"));   // rvalue overload
    log_value("literal");            // C-string overload
}
```

### Intuition

- **Lvalue**: has a name / persistent address (`a`).
    
- **Rvalue**: temporary, about to die (`std::string("tmp")`).
    
- `"literal"` has type `const char[8]`, decays to `const char*`, so it matches the C-string overload.
    

**Interview line:**

> “lvalues have identity, rvalues are disposable. We exploit that to overload for efficient moves when the object is about to die.”

---

## Q22 – Move Semantics & `std::move` Abuse

**Question:**  
Implement a small class that owns a `std::vector<int>` and supports efficient move construction and move assignment.  
Then explain why this is **bad**:

```cpp
std::vector<int> v;
v.push_back(1);
some_vec.push_back(std::move(v[0]));
```

---

### What they’re testing

- Do you understand move ctor / move assignment properly.
    
- Do you know that `std::move` on fundamental types is mostly pointless and sometimes harmful.
    

---

### Code

```cpp
#include <vector>
#include <utility>

class IntBuffer {
    std::vector<int> data_;
public:
    IntBuffer() = default;

    explicit IntBuffer(std::size_t n) : data_(n) {}

    // move constructor
    IntBuffer(IntBuffer&& other) noexcept
        : data_(std::move(other.data_)) {}

    // move assignment
    IntBuffer& operator=(IntBuffer&& other) noexcept {
        if (this != &other) {
            data_ = std::move(other.data_);
        }
        return *this;
    }

    // copy operations are defaulted here but could be deleted if desired
    IntBuffer(const IntBuffer&) = default;
    IntBuffer& operator=(const IntBuffer&) = default;
};
```

### Why `std::move(v[0])` is stupid

`v[0]` is an `int&`. `std::move(v[0])` is an `int&&`, but `int` has no move semantics different from copy. You just:

- Make the code harder to read.
    
- Potentially pick the wrong overload if you have templates expecting rvalues.
    

**Rule of thumb:**  
Use `std::move` on **objects with expensive ownership** (`std::vector`, `std::string`, your own RAII types).  
Don’t sprinkle it like salt on every variable and call it “optimization.”

---

## Q23 – Perfect Forwarding & `std::forward`

**Question:**  
Write a templated `make_and_log` function that:

1. Forwards any arguments to construct a `std::string`.
    
2. Logs it.
    
3. Returns the constructed string.
    

Use perfect forwarding and explain what an “forwarding reference” is and why `std::forward` is needed.

---

### What they’re testing

- Do you know how to write this pattern without butchering it.
    
- Understanding of `T&&` in templates vs non-templates.
    

---

### Code

```cpp
#include <iostream>
#include <string>
#include <utility>

template <typename... Args>
std::string make_and_log(Args&&... args) {
    std::string s(std::forward<Args>(args)...);
    std::cout << "Created string: " << s << "\n";
    return s;
}

int main() {
    std::string base = "hello";

    auto s1 = make_and_log("literal");             // const char*
    auto s2 = make_and_log(5, 'x');                // "xxxxx"
    auto s3 = make_and_log(base);                  // lvalue
    auto s4 = make_and_log(std::move(base));       // rvalue
}
```

### Intuition

- In a template, `T&&` is a **forwarding reference** if `T` is deduced.
    
- It can bind to both lvalues and rvalues:
    
    - lvalue → `T` deduced as `T&` → `T&&` becomes `T& &&` → collapses to `T&`.
        
    - rvalue → `T` deduced as `T` → `T&&`.
        

`std::forward<T>` preserves “lvalue/rvalue-ness.”  
If you always used `std::move`, you’d treat lvalues as rvalues and break code.

---

## Q24 – `noexcept` and its impact

**Question:**  
Explain what `noexcept` means.  
Why is this important for move constructors in containers?  
Show a simple class with `noexcept` move constructor and move assignment, and explain how it affects `std::vector` reallocation.

---

### What they’re testing

- Knowledge of exception specifications that actually matter in modern C++.
    
- Understanding of how STL uses `noexcept` to choose between move and copy.
    

---

### Code

```cpp
#include <vector>
#include <utility>

class BigObject {
    std::vector<int> data_;
public:
    BigObject() = default;
    explicit BigObject(std::size_t n) : data_(n) {}

    // mark move operations noexcept
    BigObject(BigObject&& other) noexcept
        : data_(std::move(other.data_)) {}

    BigObject& operator=(BigObject&& other) noexcept {
        if (this != &other) {
            data_ = std::move(other.data_);
        }
        return *this;
    }

    // allow copy as well
    BigObject(const BigObject&) = default;
    BigObject& operator=(const BigObject&) = default;
};
```

### Why `noexcept` matters

When `std::vector` needs to grow, it:

- Allocates new storage.
    
- Moves/copies existing elements.
    

If the element’s **move constructor is `noexcept`**, `vector` can safely move elements.  
If move _might_ throw, `vector` may fall back to **copy** to preserve the strong exception guarantee.

So:

- Mark move operations as `noexcept` when it’s safe.
    
- It gives containers permission to use them aggressively, making reallocation faster.
    

---

## Q25 – `auto`, `decltype`, and common traps

**Question:**  
Explain how `auto` deduces types in these 3 cases:

```cpp
int x = 5;
const int cx = x;
const int& rx = x;

auto a = x;
auto b = cx;
auto c = rx;
```

Then show how to get the _exact_ type (including const & reference) of an expression using `decltype`.

---

### What they’re testing

- Do you know how `auto` strips cv/ref by default.
    
- Do you know how to recover the exact type with `decltype`.
    

---

### Answer

Given:

```cpp
int x = 5;
const int cx = x;
const int& rx = x;
```

- `auto a = x;` → `int`
    
- `auto b = cx;` → `int` (top-level `const` is dropped)
    
- `auto c = rx;` → `int` (reference and top-level const dropped)
    

To inspect exact types:

```cpp
template <typename T>
void type_printer();

int main() {
    int x = 5;
    const int cx = x;
    const int& rx = x;

    // uncomment one at a time and see with compiler errors / tools
    // type_printer<decltype(x)>();
    // type_printer<decltype(cx)>();
    // type_printer<decltype(rx)>();
}
```

Or to keep cv-ref with `auto`:

```cpp
auto&   ra = rx;        // ra is const int&
auto&&  rrr = x;        // forwarding reference, deduces properly
```

**Rule:**

- `auto` strips top-level const & references.
    
- `decltype(expr)` gives you the exact type of `expr` (with rules: `decltype((x))` vs `decltype(x)` differ).
    

---

## Q26 – `enum` vs `enum class`

**Question:**  
Explain the difference between classic `enum` and `enum class`.  
Write both versions for “DayOfWeek” and show how implicit conversions differ.

---

### What they’re testing

- Scoped enums, strong typing.
    
- Avoiding namespace pollution and implicit integral conversion.
    

---

### Code

```cpp
// old-style
enum DayOld {
    Monday,
    Tuesday,
    Wednesday
};

// scoped enum
enum class DayNew : int {
    Monday,
    Tuesday,
    Wednesday
};

int main() {
    DayOld d1 = Monday;         // OK, unscoped
    int x = Tuesday;            // OK, implicit conversion to int

    DayNew d2 = DayNew::Monday; // must scope with DayNew::
    // int y = DayNew::Tuesday; // error: no implicit conversion

    int z = static_cast<int>(DayNew::Tuesday); // explicit cast
}
```

### Intuition

- `enum class`:
    
    - scoped (no leaking “Monday” into global namespace),
        
    - strongly typed (no implicit int conversions),
        
    - underlying type can be specified (`: int`, `: std::uint8_t`, etc.).
        

Modern C++: use `enum class` unless you absolutely need old behavior.

---

## Q27 – Operator Overloading (2D Vector)

**Question:**  
Implement a small `Vec2` struct that supports:

- `+`, `-`, scalar `*`
    
- `==`, `!=`
    
- `operator<<` for printing
    

Explain why operators should usually be non-member friend functions when possible.

---

### What they’re testing

- Clean operator overloading.
    
- Const correctness.
    
- Non-member functions & symmetry.
    

---

### Code

```cpp
#include <iostream>

struct Vec2 {
    double x;
    double y;

    Vec2(double x_, double y_) : x(x_), y(y_) {}
};

// arithmetic
inline Vec2 operator+(const Vec2& a, const Vec2& b) {
    return Vec2(a.x + b.x, a.y + b.y);
}

inline Vec2 operator-(const Vec2& a, const Vec2& b) {
    return Vec2(a.x - b.x, a.y - b.y);
}

inline Vec2 operator*(const Vec2& v, double s) {
    return Vec2(v.x * s, v.y * s);
}

inline Vec2 operator*(double s, const Vec2& v) {
    return v * s;
}

// comparison
inline bool operator==(const Vec2& a, const Vec2& b) {
    return a.x == b.x && a.y == b.y;
}

inline bool operator!=(const Vec2& a, const Vec2& b) {
    return !(a == b);
}

// streaming
inline std::ostream& operator<<(std::ostream& os, const Vec2& v) {
    return os << "(" << v.x << ", " << v.y << ")";
}
```

### Why non-member?

- For symmetric operators (like `a + b`, `s * v`), you don’t want a member on only one side.
    
- Non-members allow implicit conversions on **both** operands.
    
- It also avoids forcing every operator into the class public interface.
    

---

## Q28 – SFINAE / Concepts to restrict templates to arithmetic types

**Question:**  
Write a templated `sum` function that:

- Works only for arithmetic types (`int`, `double`, etc.).
    
- Rejects `std::string` or other non-arithmetic types at compile time.
    

Use either:

- SFINAE with `std::enable_if`, or
    
- C++20 concepts.
    

---

### What they’re testing

- You know how to constrain templates instead of letting them instantiate garbage.
    
- Awareness of type traits / concepts.
    

---

### Version A – SFINAE

```cpp
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_arithmetic<T>::value, T>::type
sum(T a, T b) {
    return a + b;
}
```

### Version B – Concepts (C++20)

```cpp
#include <concepts>

template <std::integral T>
T sum_int(T a, T b) {
    return a + b;
}

template <std::floating_point T>
T sum_float(T a, T b) {
    return a + b;
}

template <std::integral T, std::floating_point U>
auto sum_mixed(T a, U b) {
    return a + b;
}
```

**Intuition**

- Constraining templates avoids cryptic compile errors.
    
- Concepts make signatures self-documenting: `template<std::floating_point T>` screams intent.
    

---

## Q29 – Iterators & Custom Container Compatibility

**Question:**  
Implement a simple fixed-size array container `StaticArray<T, N>` that:

- Stores `T data_[N];`
    
- Exposes `begin()`, `end()`, `cbegin()`, `cend()`
    
- Works with range-based `for` and `std::sort`.
    

---

### What they’re testing

- Understanding of minimal iterator interface requirements.
    
- Ability to make new types interoperate with STL algorithms.
    

---

### Code

```cpp
#include <cstddef>
#include <algorithm>
#include <iostream>

template <typename T, std::size_t N>
class StaticArray {
    T data_[N];
public:
    using iterator = T*;
    using const_iterator = const T*;

    T& operator[](std::size_t i) { return data_[i]; }
    const T& operator[](std::size_t i) const { return data_[i]; }

    iterator begin() { return data_; }
    iterator end()   { return data_ + N; }

    const_iterator begin() const { return data_; }
    const_iterator end()   const { return data_ + N; }

    const_iterator cbegin() const { return data_; }
    const_iterator cend()   const { return data_ + N; }

    constexpr std::size_t size() const { return N; }
};

int main() {
    StaticArray<int, 5> arr;
    for (std::size_t i = 0; i < arr.size(); ++i)
        arr[i] = static_cast<int>(5 - i);

    std::sort(arr.begin(), arr.end());

    for (int x : arr) {
        std::cout << x << " ";
    }
}
```

### Intuition

- If your type provides `begin()/end()`, it plugs into all STL algorithms.
    
- That’s how Eigen, xtensor, etc. play nicely with parts of the STL.
    

---

## Q30 – pImpl Idiom (Compilation Firewall)

**Question:**  
Explain the pImpl (“pointer to implementation”) idiom.  
Implement a `Logger` class where the header only exposes:

```cpp
class Logger {
public:
    Logger();
    ~Logger();
    void log(const std::string& msg);
private:
    struct Impl;
    std::unique_ptr<Impl> impl_;
};
```

…and define `Impl` in the `.cpp`.  
Explain why this reduces compile-time dependencies.

---

### What they’re testing

- Separation of interface and implementation.
    
- Ability to manage ownership with `std::unique_ptr`.
    
- You’ve seen real codebases, not only LeetCode.
    

---

### Header (Logger.h)

```cpp
#include <memory>
#include <string>

class Logger {
public:
    Logger();
    ~Logger();
    void log(const std::string& msg);
private:
    struct Impl;
    std::unique_ptr<Impl> impl_;
};
```

### Source (Logger.cpp)

```cpp
#include "Logger.h"
#include <iostream>

struct Logger::Impl {
    void log_impl(const std::string& msg) {
        std::cout << "[LOG] " << msg << "\n";
    }
};

Logger::Logger() : impl_(std::make_unique<Impl>()) {}

Logger::~Logger() = default;

void Logger::log(const std::string& msg) {
    impl_->log_impl(msg);
}
```

### Why this helps

- Users of `Logger` don’t see `<iostream>` or any of its internal includes.
    
- Changing implementation (`Impl`) doesn’t require recompiling every file that includes `Logger.h`.
    
- In large codebases (like a bank’s pricing stack), this is a big deal.
    

---

## Summary Table (Q21–Q30)

|ID|Topic|Core C++ Concept(s)|Why Interviewers Care|
|---|---|---|---|
|21|Lvalues vs rvalues|Reference binding, overload resolution|Move semantics correctness|
|22|Move semantics & misuse|Move ctor/assign, `std::move` pitfalls|Performance & correctness in real code|
|23|Perfect forwarding|Forwarding refs, `std::forward`, variadic tmpl|Generic wrappers/utilities|
|24|`noexcept` & containers|Exception specs, move vs copy in `vector`|STL behavior under reallocation|
|25|`auto` & `decltype`|Type deduction rules, cv-ref stripping|Avoid subtle bugs with generics|
|26|`enum` vs `enum class`|Scoped enums, strong typing|Cleaner, safer APIs|
|27|Operator overloading|Non-member operators, const correctness|Numeric types, vectors, matrices|
|28|SFINAE / concepts|Template constraints, type traits|Restricting templates to valid types|
|29|Custom container + iterators|begin/end, iterator compatibility|Making types work with STL algorithms|
|30|pImpl idiom|Encapsulation, compilation firewall, unique_ptr|Real-world library design & build times|

Once you can:

- write all this from scratch,
    
- explain each design choice calmly,
    
- and modify code on the fly when the interviewer “what if”s you,
    

then you’re no longer “learning C++”, you’re just adding more weapons.

Next natural steps from here (no, you don’t have a choice, you started this):

- templates + expression templates,
    
- `std::variant`, type erasure with `std::any` / custom wrappers,
    
- `constexpr` algorithms & data structures,
    
- modern async (coroutines) if you’re feeling masochistic.