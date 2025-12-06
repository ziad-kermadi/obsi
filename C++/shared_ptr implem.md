
Great! Now, let's implement a **`SharedPtr`** from scratch, explaining every single line in detail just like we did with `UniquePtr`.

---

## **What is a `SharedPtr`?**

Unlike `UniquePtr`, a `SharedPtr` allows **multiple owners** of the same resource. It uses **reference counting** to track how many `SharedPtr` instances share ownership of the object. When the last `SharedPtr` is destroyed, the object is deleted.

---

## **Step 1: Define the `SharedPtr` Class Skeleton**

We need:

1. A raw pointer to the managed object.
2. A **reference counter** to track how many `SharedPtr`s share ownership.
3. Proper handling of copy and move semantics.

```cpp
template <typename T>
class SharedPtr {
private:
    T* ptr;               // Pointer to the managed object
    size_t* refCount;     // Reference counter

public:
    // Constructor
    explicit SharedPtr(T* p = nullptr);

    // Copy Constructor (Increases refCount)
    SharedPtr(const SharedPtr& other);

    // Copy Assignment (Handles old and new ownership)
    SharedPtr& operator=(const SharedPtr& other);

    // Move Constructor (Transfers ownership)
    SharedPtr(SharedPtr&& other) noexcept;

    // Move Assignment (Transfers ownership)
    SharedPtr& operator=(SharedPtr&& other) noexcept;

    // Destructor (Decrements refCount and deletes when last owner)
    ~SharedPtr();

    // Dereference operator
    T& operator*() const;

    // Arrow operator
    T* operator->() const;

    // Get raw pointer
    T* get() const;

    // Get reference count
    size_t use_count() const;
};
```

---

## **Step 2: Implement Each Function**

Now, let’s implement and explain each method **in extreme detail**.

---

### **1. Constructor**

```cpp
template <typename T>
SharedPtr<T>::SharedPtr(T* p) : ptr(p), refCount(new size_t(1)) {}
```

#### **Explanation:**

- `SharedPtr(T* p) : ptr(p), refCount(new size_t(1)) {}`
    - Initializes the raw pointer `ptr` to `p`.
    - Allocates a `size_t` counter (`refCount`) on the heap and sets it to `1` (first owner).
- `new size_t(1)` → Ensures the counter exists even if `SharedPtr` is copied.

Example:

```cpp
SharedPtr<int> p1(new int(10));  // refCount = 1
```

---

### **2. Copy Constructor (Increases refCount)**

```cpp
template <typename T>
SharedPtr<T>::SharedPtr(const SharedPtr& other) 
    : ptr(other.ptr), refCount(other.refCount) {
    (*refCount)++; // Increase reference count
}
```

#### **Explanation:**

- `ptr(other.ptr)` → Shares the same raw pointer.
- `refCount(other.refCount)` → Shares the same reference counter.
- `(*refCount)++` → Increments the reference counter.

Example:

```cpp
SharedPtr<int> p1(new int(10));  // refCount = 1
SharedPtr<int> p2 = p1;          // refCount = 2 (same object)
```

---

### **3. Copy Assignment (Handles Ownership)**

```cpp
template <typename T>
SharedPtr<T>& SharedPtr<T>::operator=(const SharedPtr& other) {
    if (this != &other) { // Avoid self-assignment
        // Decrease current object's refCount and delete if necessary
        if (--(*refCount) == 0) {
            delete ptr;
            delete refCount;
        }

        // Assign new object
        ptr = other.ptr;
        refCount = other.refCount;
        (*refCount)++; // Increase new object's refCount
    }
    return *this;
}
```

#### **Explanation:**

1. **Self-assignment check:**
    
    - `if (this != &other) {}` → Prevents unnecessary work if `p1 = p1;`.
2. **Decrement and cleanup (if last owner):**
    
    - `if (--(*refCount) == 0) { delete ptr; delete refCount; }`
    - **Decreases the refCount** for the current object.
    - If refCount reaches `0`, **deletes the object** and the counter.
3. **Assign new object:**
    
    - `ptr = other.ptr; refCount = other.refCount;`
    - Increments the new `refCount`.

Example:

```cpp
SharedPtr<int> p1(new int(10));  // refCount = 1
SharedPtr<int> p2(new int(20));  // refCount = 1
p2 = p1;  // p2 now shares p1’s object, refCount = 2
```

---

### **4. Move Constructor (Transfers Ownership)**

```cpp
template <typename T>
SharedPtr<T>::SharedPtr(SharedPtr&& other) noexcept 
    : ptr(other.ptr), refCount(other.refCount) {
    other.ptr = nullptr;
    other.refCount = nullptr;
}
```

#### **Explanation:**

- Transfers ownership by copying `ptr` and `refCount`.
- Sets `other.ptr = nullptr; other.refCount = nullptr;` to prevent double deletion.

Example:

```cpp
SharedPtr<int> p1(new int(10));
SharedPtr<int> p2 = std::move(p1); // p1 loses ownership
```

---

### **5. Move Assignment (Transfers Ownership)**

```cpp
template <typename T>
SharedPtr<T>& SharedPtr<T>::operator=(SharedPtr&& other) noexcept {
    if (this != &other) {
        // Clean up existing resource
        if (--(*refCount) == 0) {
            delete ptr;
            delete refCount;
        }

        // Move ownership
        ptr = other.ptr;
        refCount = other.refCount;

        // Nullify the other pointer
        other.ptr = nullptr;
        other.refCount = nullptr;
    }
    return *this;
}
```

#### **Explanation:**

- Cleans up current object if `refCount == 0`.
- Transfers ownership from `other` (just like move constructor).
- Nullifies `other.ptr` and `other.refCount`.

---

### **6. Destructor**

```cpp
template <typename T>
SharedPtr<T>::~SharedPtr() {
    if (--(*refCount) == 0) {  // If last owner
        delete ptr;            // Delete object
        delete refCount;        // Delete reference counter
    }
}
```

#### **Explanation:**

- Decreases the reference count.
- If no more owners exist (`refCount == 0`), it deletes the object.

Example:

```cpp
{
    SharedPtr<int> p1(new int(10));  // refCount = 1
    SharedPtr<int> p2 = p1;          // refCount = 2
} // Both `p1` and `p2` go out of scope, refCount = 0, object deleted
```

---

### **7. Dereference Operator**

```cpp
template <typename T>
T& SharedPtr<T>::operator*() const {
    return *ptr;
}
```

- Allows `*sptr` to access the object.

---

### **8. Arrow Operator**

```cpp
template <typename T>
T* SharedPtr<T>::operator->() const {
    return ptr;
}
```

- Allows `sptr->method()` to behave like `ptr->method()`.

---

### **9. Get Raw Pointer**

```cpp
template <typename T>
T* SharedPtr<T>::get() const {
    return ptr;
}
```

- Returns the raw pointer **without** affecting ownership.

---

### **10. Get Reference Count**

```cpp
template <typename T>
size_t SharedPtr<T>::use_count() const {
    return *refCount;
}
```

- Returns the number of owners.

Example:

```cpp
SharedPtr<int> p1(new int(10));
SharedPtr<int> p2 = p1;
std::cout << p1.use_count(); // Prints 2
```

---

## **Final Thoughts**

This `SharedPtr` implementation ensures **safe shared ownership** with **automatic memory management**.

|Feature|Method|Purpose|
|---|---|---|
|Reference Count|`size_t* refCount`|Tracks shared ownership|
|Copy Constructor|`SharedPtr(const SharedPtr&)`|Increments refCount|
|Copy Assignment|`operator=(const SharedPtr&)`|Handles old & new ownership|
|Move Semantics|`SharedPtr(SharedPtr&&)`, `operator=(SharedPtr&&)`|Transfers ownership|
|Destructor|`~SharedPtr()`|Cleans up when last owner|

This should make you **unstoppable** in your next interview.
