# Book 8: Structures & Dynamic Memory

## The Complete Guide to Custom Data Types and Heap Management in C++

---

### **Target Level:** Intermediate
### **Prerequisites:** Books 1–7
### **Learning Outcomes:**
By the end of this book, you will:
- Master struct definition, initialization, and usage
- Understand nested structures and arrays of structures
- Master dynamic memory allocation with `new`/`delete`
- Understand heap vs stack memory deeply
- Know about memory leaks, fragmentation, and RAII fundamentals
- Understand `union`, `enum class`, and aggregate types

---

## Chapter 1: Structures (`struct`)

### 1.1 What is a struct?

A **struct** groups related variables of different types under one name:

```cpp
struct Student {
    std::string name;
    int age;
    double gpa;
    bool isActive;
};

// Creating and using:
Student s1;
s1.name = "Alice";
s1.age = 20;
s1.gpa = 3.85;
s1.isActive = true;

// Aggregate initialization:
Student s2 = {"Bob", 21, 3.90, true};

// C++20 designated initializers:
Student s3 = {.name = "Charlie", .age = 19, .gpa = 3.75, .isActive = true};
```

### 1.2 Memory Layout

```cpp
struct Example {
    char a;      // 1 byte  → offset 0
    // 3 bytes padding
    int b;       // 4 bytes → offset 4
    char c;      // 1 byte  → offset 8
    // 7 bytes padding
    double d;    // 8 bytes → offset 16
};
// sizeof(Example) = 24 (not 14!)

// Optimized ordering:
struct OptimizedExample {
    double d;    // 8 bytes → offset 0
    int b;       // 4 bytes → offset 8
    char a;      // 1 byte  → offset 12
    char c;      // 1 byte  → offset 13
    // 2 bytes padding
};
// sizeof(OptimizedExample) = 16 (saved 8 bytes!)
```

### 1.3 Struct Operations

```cpp
struct Point {
    double x, y;
};

// Copy (member-wise):
Point p1 = {3.0, 4.0};
Point p2 = p1;           // Deep copy of all members

// Comparison (must define yourself or use C++20 default):
bool operator==(const Point& a, const Point& b) {
    return a.x == b.x && a.y == b.y;
}

// C++20 — auto-generate comparisons:
struct Point {
    double x, y;
    auto operator<=>(const Point&) const = default;
};

// Passing to functions (prefer const reference):
double distance(const Point& a, const Point& b) {
    double dx = a.x - b.x;
    double dy = a.y - b.y;
    return std::sqrt(dx * dx + dy * dy);
}
```

### 1.4 Struct with Methods

In C++, structs can have methods (unlike C):

```cpp
struct Rectangle {
    double width, height;
    
    double area() const {        // const = doesn't modify the struct
        return width * height;
    }
    
    double perimeter() const {
        return 2 * (width + height);
    }
    
    void scale(double factor) {
        width *= factor;
        height *= factor;
    }
};

Rectangle r = {5.0, 3.0};
std::cout << r.area();        // 15.0
r.scale(2.0);
std::cout << r.area();        // 60.0
```

**Note:** In C++, `struct` and `class` are nearly identical. The only difference is default access: `struct` members are `public` by default, `class` members are `private` by default.

### 1.5 Nested Structures

```cpp
struct Address {
    std::string street;
    std::string city;
    std::string state;
    int zip;
};

struct Employee {
    std::string name;
    int id;
    double salary;
    Address homeAddress;       // Nested struct
    Address workAddress;       // Another instance
};

Employee emp;
emp.name = "Alice";
emp.homeAddress.city = "Seattle";
emp.workAddress.zip = 98101;
```

### 1.6 Self-Referential Structures (Linked Lists)

```cpp
struct Node {
    int data;
    Node* next;    // Pointer to another Node (can't be Node directly — infinite size!)
};

// Creating a simple linked list:
Node* head = new Node{10, nullptr};
head->next = new Node{20, nullptr};
head->next->next = new Node{30, nullptr};

// Traversal:
Node* current = head;
while (current != nullptr) {
    std::cout << current->data << " → ";
    current = current->next;
}
std::cout << "null\n";
// Output: 10 → 20 → 30 → null

// Cleanup:
while (head != nullptr) {
    Node* temp = head;
    head = head->next;
    delete temp;
}
```

### 1.7 `union` — Shared Memory

A `union` stores different types in the **same** memory location. Only one member is active at a time:

```cpp
union Value {
    int i;
    float f;
    char c;
};
// sizeof(Value) = max(sizeof(int), sizeof(float), sizeof(char)) = 4

Value v;
v.i = 42;         // Now i is the active member
std::cout << v.i;  // OK: 42
std::cout << v.f;  // UB: reading inactive member

v.f = 3.14f;      // Now f is the active member
std::cout << v.f;  // OK: 3.14
```

**`std::variant` (C++17) — type-safe alternative:**
```cpp
#include <variant>

std::variant<int, float, std::string> v;
v = 42;
v = 3.14f;
v = "hello";

// Type-safe access:
if (std::holds_alternative<std::string>(v)) {
    std::cout << std::get<std::string>(v);
}
```

---

## Chapter 2: Dynamic Memory Management

### 2.1 Stack vs Heap

| Feature | Stack | Heap |
|---------|-------|------|
| Allocation | Automatic (compiler manages) | Manual (`new`/`delete`) |
| Speed | Very fast (just move stack pointer) | Slower (allocator bookkeeping) |
| Size | Limited (1-8 MB typical) | Virtually unlimited |
| Lifetime | Scope-bound (auto cleanup) | Until explicitly freed |
| Fragmentation | None | Can fragment over time |
| Thread safety | Each thread has own stack | Shared — needs synchronization |

### 2.2 `new` and `delete`

```cpp
// Allocate single object:
int* p = new int;           // Uninitialized
int* p2 = new int(42);     // Initialized to 42
int* p3 = new int{42};     // Brace initialization

// Allocate array:
int* arr = new int[10];           // 10 uninitialized ints
int* arr2 = new int[10]();        // 10 ints, all zero-initialized
int* arr3 = new int[5]{1, 2, 3};  // {1, 2, 3, 0, 0}

// Free single object:
delete p;
delete p2;
delete p3;

// Free array:
delete[] arr;     // MUST use delete[] for arrays!
delete[] arr2;
delete[] arr3;
```

### 2.3 Critical Rules

```cpp
// 1. Every new must have a matching delete:
int* p = new int(42);
delete p;           // ✓

// 2. Every new[] must have delete[]:
int* arr = new int[10];
delete[] arr;       // ✓ — delete[] for arrays!
// delete arr;      // ✗ UNDEFINED BEHAVIOR!

// 3. Never delete stack memory:
int local = 42;
// delete &local;   // ✗ UNDEFINED BEHAVIOR!

// 4. Never delete twice:
int* p = new int(42);
delete p;
// delete p;        // ✗ DOUBLE DELETE = UNDEFINED BEHAVIOR!
p = nullptr;        // Good practice: nullify after delete
delete p;           // Safe: deleting nullptr is a no-op

// 5. Always initialize pointers:
int* p = nullptr;   // Not new int — just null until needed
```

### 2.4 Memory Leaks

A **memory leak** occurs when allocated memory is never freed:

```cpp
void leaky() {
    int* p = new int(42);
    // Function returns without delete — LEAK!
}

void alsoLeaky() {
    int* p = new int(42);
    int* p2 = new int(100);
    p = p2;    // Old address lost! First allocation is leaked!
    delete p;  // Only frees the second allocation
}

// Leak in exception-unsafe code:
void riskyFunction() {
    int* data = new int[1000];
    processData(data);     // If this throws, delete[] never runs!
    delete[] data;
}
```

### 2.5 RAII — Resource Acquisition Is Initialization

The fundamental C++ pattern to prevent leaks:

```cpp
class IntArray {
    int* data_;
    size_t size_;
public:
    IntArray(size_t n) : data_(new int[n]()), size_(n) {}
    ~IntArray() { delete[] data_; }   // Destructor frees memory automatically
    
    int& operator[](size_t i) { return data_[i]; }
    size_t size() const { return size_; }
};

void safeFunction() {
    IntArray arr(1000);    // Allocated in constructor
    arr[0] = 42;
    // Even if an exception occurs, destructor runs and frees memory!
}  // arr destroyed here — destructor calls delete[]
```

This is the foundation of smart pointers, `std::vector`, `std::string`, and all modern C++ resource management.

### 2.6 `new` Failure — `std::bad_alloc`

```cpp
try {
    int* huge = new int[100'000'000'000];  // ~400 GB — will fail
} catch (const std::bad_alloc& e) {
    std::cerr << "Allocation failed: " << e.what() << "\n";
}

// No-throw alternative:
int* p = new(std::nothrow) int[huge_size];
if (p == nullptr) {
    std::cerr << "Allocation failed\n";
}
```

### 2.7 Placement `new`

Construct an object at a specific memory address:

```cpp
#include <new>

char buffer[sizeof(std::string)];               // Raw memory
std::string* s = new (buffer) std::string("Hello");  // Construct in buffer

std::cout << *s;  // "Hello"

s->~string();     // Must manually call destructor (placement new requires this)
```

Used in memory pools, allocators, and `std::vector` internals.

---

## Chapter 3: Dynamic Data Structures

### 3.1 Dynamic Struct Allocation

```cpp
struct Student {
    std::string name;
    int age;
    double gpa;
};

// Single struct:
Student* s = new Student{"Alice", 20, 3.85};
std::cout << s->name;  // Use -> with pointers to structs
delete s;

// Array of structs:
Student* class_ = new Student[3]{
    {"Alice", 20, 3.85},
    {"Bob", 21, 3.90},
    {"Charlie", 19, 3.75}
};
for (int i = 0; i < 3; i++) {
    std::cout << class_[i].name << "\n";
}
delete[] class_;
```

### 3.2 Growing Arrays

```cpp
// Manual resizable array (what std::vector does internally):
size_t size = 0, capacity = 4;
int* arr = new int[capacity];

void push_back(int value) {
    if (size >= capacity) {
        capacity *= 2;
        int* newArr = new int[capacity];
        std::copy(arr, arr + size, newArr);
        delete[] arr;
        arr = newArr;
    }
    arr[size++] = value;
}
```

---

## Chapter 4: Interview Section

**Q1: What is a memory leak?** — Allocated memory that is never freed, accumulating over time.

**Q2: What is the difference between `delete` and `delete[]`?** — `delete` frees a single object. `delete[]` frees an array. Using the wrong one is UB.

**Q3: What is RAII?** — Resource Acquisition Is Initialization. Resources (memory, files, locks) are acquired in constructors and released in destructors, ensuring cleanup even with exceptions.

**Q4: What is the difference between `struct` and `class` in C++?** — Only default access: `struct` defaults to `public`, `class` defaults to `private`. Otherwise identical.

---

## Chapter 5: Practice & Projects

### Exercises
1. Create a `Student` database using an array of structs with CRUD operations.
2. Implement a dynamic array class with push_back, pop_back, resize, and proper destructor.
3. Implement a singly linked list with insert, delete, search, reverse, and print.

### Project 1: Student Records System
Menu-driven program to add, view, search, update, delete student records using dynamic arrays.

### Project 2: Memory Pool Allocator
Fixed-size block allocator that pre-allocates a large chunk and hands out fixed-size blocks.

---

## Chapter 6: Summary & Mastery Checklist

- [ ] You can define and use structs with proper memory layout awareness
- [ ] You understand nested structures and self-referential structures
- [ ] You can use `new`/`delete` and `new[]`/`delete[]` correctly
- [ ] You understand memory leaks and know how to prevent them
- [ ] You understand RAII and why it's fundamental to C++
- [ ] You know the difference between stack and heap allocation
- [ ] You can implement basic dynamic data structures (linked lists, dynamic arrays)
- [ ] You understand union vs variant

**Proceed to Book 9: Object-Oriented Programming.**

---

*End of Book 8: Structures & Dynamic Memory*
