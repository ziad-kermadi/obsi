
# **Access Specifiers in C++: `private` vs. `public` vs. `protected`**

| **Feature**            | **`private`** | **`protected`** | **`public`** |
|------------------------|--------------|----------------|--------------|
| **Accessibility in Same Class** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Accessible in Derived Classes** | ❌ No | ✅ Yes | ✅ Yes |
| **Accessible Outside the Class** | ❌ No | ❌ No | ✅ Yes |
| **Use Case** | Hide internal implementation (Encapsulation) | Allow controlled inheritance access | Expose publicly accessible API |
| **Inheritance Effect (Public Inheritance)** | **Not inherited** | Becomes `protected` in child class | Remains `public` in child class |
| **Inheritance Effect (Protected Inheritance)** | **Not inherited** | Becomes `protected` in child class | Becomes `protected` in child class |
| **Inheritance Effect (Private Inheritance)** | **Not inherited** | Becomes `private` in child class | Becomes `private` in child class |
| **Best Used For** | Private data members, helper functions | Inheritance where derived classes should have access | Public API functions, Interface methods |

---

## **Code Example Demonstrating Differences**
```cpp
#include <iostream>
using namespace std;

class Base {
private:
    int privateVar = 1;  // ❌ Not accessible outside the class
protected:
    int protectedVar = 2;  // ✅ Accessible in derived class
public:
    int publicVar = 3;  // ✅ Accessible everywhere

    void show() {
        cout << "Private: " << privateVar << ", Protected: " << protectedVar << ", Public: " << publicVar << endl;
    }
};

class Derived : public Base {
public:
    void accessMembers() {
        // privateVar = 10;  // ❌ Not accessible
        protectedVar = 20;  // ✅ Accessible in derived class
        publicVar = 30;  // ✅ Accessible in derived class
    }
};

int main() {
    Base b;
    // b.privateVar = 10;  // ❌ Error: Private members are inaccessible
    // b.protectedVar = 20;  // ❌ Error: Protected members are inaccessible
    b.publicVar = 30;  // ✅ Accessible

    Derived d;
    d.accessMembers();
    // d.protectedVar = 40;  // ❌ Error: Protected is inaccessible outside derived class
    d.publicVar = 50;  // ✅ Accessible

    return 0;
}
```

---

### **Key Takeaways**
- **`private`**: Only accessible **inside the same class**.
- **`protected`**: Accessible in **same class + derived classes**, but **not outside**.
- **`public`**: Accessible **everywhere**.

🚀 **Use `private` for encapsulation, `protected` for controlled inheritance, and `public` for APIs!** 🚀