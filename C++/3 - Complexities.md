
# **Time Complexity Cheat Sheet with Intuition**

This cheat sheet covers **common time complexities**, their **intuitions**, and examples of algorithms that fall into each category.

---

## **1. Complexity Classes Overview**

| **Big-O Notation** | **Name**             | **Intuition** | **Example Algorithms** |
|--------------------|----------------------|--------------|-----------------------|
| **O(1)**          | Constant Time         | Takes the same amount of time, no matter the input size | Array indexing, Hash table lookup (`std::unordered_map`) |
| **O(log n)**      | Logarithmic Time      | Each step reduces the problem size by a fraction | Binary Search, Tree operations (Balanced BST) |
| **O(n)**          | Linear Time           | Time grows directly with input size | Iterating through an array, Finding min/max |
| **O(n log n)**    | Linearithmic Time     | Combination of linear and logarithmic growth | Merge Sort, QuickSort (average case), Heap Sort |
| **O(n²)**         | Quadratic Time        | Every element interacts with every other element | Bubble Sort, Selection Sort, Insertion Sort |
| **O(n³)**         | Cubic Time            | Triple nested loops | Floyd-Warshall Algorithm (All-Pairs Shortest Path) |
| **O(2ⁿ)**         | Exponential Time      | Doubles with each step | Recursive Fibonacci, Backtracking (Subset Generation) |
| **O(n!)**         | Factorial Time        | Explodes rapidly as input size increases | Traveling Salesman Problem (TSP), Permutations |

---

## **2. Intuition & Examples of Time Complexities**

### **O(1) - Constant Time Complexity**
- **Definition**: The execution time **does not change** regardless of the input size.
- **Example**: Accessing an array element by index.
- **Intuition**: Imagine having **10 million books**, but finding the first book in `O(1)` because you have a **direct index** to it.

```cpp
int arr[1000];
int x = arr[5];  // Always takes the same time regardless of size
```

---

### **O(log n) - Logarithmic Time Complexity**
- **Definition**: The number of operations **grows logarithmically** with input size.
- **Example**: Binary Search, where each step **halves the problem size**.
- **Intuition**: Imagine searching for a word in a **dictionary** by opening it in the middle instead of scanning each page.

```cpp
int binarySearch(int arr[], int left, int right, int x) {
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == x) return mid;
        if (arr[mid] < x) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

---

### **O(n) - Linear Time Complexity**
- **Definition**: Execution time grows **linearly** with input size.
- **Example**: Finding the maximum element in an array.
- **Intuition**: Walking through a **crowded room**, checking each person **one by one**.

```cpp
int findMax(int arr[], int n) {
    int maxVal = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > maxVal) maxVal = arr[i];
    }
    return maxVal;
}
```

---

### **O(n log n) - Linearithmic Time Complexity**
- **Definition**: Execution time grows **slightly more than linear**, but far less than quadratic.
- **Example**: Merge Sort, QuickSort (on average).
- **Intuition**: **Dividing and conquering**—breaking a problem down, sorting smaller parts, and merging them.

```cpp
void mergeSort(int arr[], int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        mergeSort(arr, l, m);
        mergeSort(arr, m + 1, r);
        merge(arr, l, m, r);  // Merging sorted halves
    }
}
```

---

### **O(n²) - Quadratic Time Complexity**
- **Definition**: Execution time grows **proportionally to the square of the input size**.
- **Example**: Bubble Sort, Nested loops iterating over the same dataset.
- **Intuition**: If each **person in a room shakes hands with everyone else**, total handshakes = **n × n**.

```cpp
void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```

---

### **O(n³) - Cubic Time Complexity**
- **Definition**: Execution time **grows cubically** with input size.
- **Example**: Floyd-Warshall Algorithm (All-Pairs Shortest Path).
- **Intuition**: **Three nested loops**, like iterating over every possible triplet in a dataset.

```cpp
void floydWarshall(int graph[][V]) {
    for (int k = 0; k < V; k++) {
        for (int i = 0; i < V; i++) {
            for (int j = 0; j < V; j++) {
                if (graph[i][k] + graph[k][j] < graph[i][j])
                    graph[i][j] = graph[i][k] + graph[k][j];
            }
        }
    }
}
```

---

### **O(2ⁿ) - Exponential Time Complexity**
- **Definition**: Execution time **doubles with each increase in input size**.
- **Example**: Recursive Fibonacci, Backtracking (Subset Generation).
- **Intuition**: Each step **branches into two or more possibilities**.

```cpp
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

---

### **O(n!) - Factorial Time Complexity**
- **Definition**: Execution time **grows factorially with input size**.
- **Example**: Brute-force solution to **Traveling Salesman Problem (TSP)**, generating all **permutations**.
- **Intuition**: **Rearranging a deck of cards**—the number of possible orders grows **extremely fast**.

```cpp
void permute(std::string str, int l, int r) {
    if (l == r) std::cout << str << std::endl;
    else {
        for (int i = l; i <= r; i++) {
            std::swap(str[l], str[i]);
            permute(str, l + 1, r);
            std::swap(str[l], str[i]); // backtrack
        }
    }
}
```

---

## **Final Summary Table**

| **Complexity**  | **Growth** | **Intuition** | **Example Algorithm** |
|----------------|-----------|--------------|-----------------------|
| **O(1)** | Constant | Always takes the same time | Hash table lookup, array access |
| **O(log n)** | Logarithmic | Halves the problem at each step | Binary Search |
| **O(n)** | Linear | Processes each element once | Iterating through an array |
| **O(n log n)** | Linearithmic | Divide-and-conquer | Merge Sort, QuickSort |
| **O(n²)** | Quadratic | Nested loops over elements | Bubble Sort, Selection Sort |
| **O(n³)** | Cubic | Triple nested loops | Floyd-Warshall Algorithm |
| **O(2ⁿ)** | Exponential | Branching at each step | Recursive Fibonacci, Backtracking |
| **O(n!)** | Factorial | Permutations of elements | Traveling Salesman Problem (TSP) |

---

### **Key Takeaways**
1. **Lower complexity is always better.**
2. **Exponential and factorial complexities should be avoided** for large inputs.
3. **Logarithmic and linearithmic complexities are optimal for sorting and searching.**
4. **Recognizing nested loops and recursion helps identify time complexity.**

🚀 **This cheat sheet should help you quickly analyze algorithm efficiency! Let me know if you need more insights!** 🚀