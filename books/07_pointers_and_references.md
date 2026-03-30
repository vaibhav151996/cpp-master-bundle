# Book 7: Pointers & References

## The Complete Guide to Memory Addresses, Indirection, and Aliasing in C++

---

### **Target Level:** Intermediate
### **Prerequisites:** Books 1–6
### **Learning Outcomes:**
By the end of this book, you will:
- Understand pointers at the deepest level — what they are, how they work in memory
- Master pointer arithmetic and its relationship to arrays
- Handle null pointers, void pointers, and dangling pointers safely
- Master multi-level pointers (pointer to pointer)
- Understand references as compile-time aliases
- Know the difference between pointers and references and when to use each
- Understand how pointers and references work at the assembly level
- Handle common pointer pitfalls and interview questions

---

## Chapter 1: Pointer Fundamentals

### 1.1 What is a Pointer?

A **pointer** is a variable that stores a **memory address**. That's it. It holds a number that represents a location in memory.

```cpp
int x = 42;
int* ptr = &x;    // ptr stores the ADDRESS of x

// If x is at memory address 0x7FFE1234:
// x   contains: 42
// ptr contains: 0x7FFE1234
```

**Visual:**
```
Variable:   x           ptr
Value:      42          0x7FFE1234
Address:    0x7FFE1234  0x7FFE1240
            ↑___________________________│
            ptr "points to" x
```

### 1.2 Declaring Pointers

```cpp
int* p1;        // Pointer to int (preferred style)
int *p2;        // Same thing (C style)
int * p3;       // Same thing

int* a, b;      // GOTCHA: a is int*, but b is just int!
int *a, *b;     // Both are int* (place * with each variable)
```

### 1.3 The Address-Of Operator `&`

```cpp
int x = 42;
int* ptr = &x;     // & gives the address of x

std::cout << "Value of x: " << x << "\n";         // 42
std::cout << "Address of x: " << &x << "\n";      // 0x7FFE1234 (varies)
std::cout << "Value of ptr: " << ptr << "\n";      // 0x7FFE1234 (same as &x)
std::cout << "Address of ptr: " << &ptr << "\n";   // 0x7FFE1240 (ptr's own address)
```

### 1.4 The Dereference Operator `*`

The `*` operator "follows" the pointer to get/set the value at the address:

```cpp
int x = 42;
int* ptr = &x;

std::cout << *ptr << "\n";  // 42 — dereference: get value at address
*ptr = 100;                  // Change x through the pointer
std::cout << x << "\n";      // 100 — x was modified through ptr!
```

### 1.5 Pointer Size

A pointer's size depends on the **platform**, not the pointed-to type:

```cpp
std::cout << sizeof(int*) << "\n";      // 8 (on 64-bit system)
std::cout << sizeof(double*) << "\n";   // 8 (same!)
std::cout << sizeof(char*) << "\n";     // 8 (same!)
std::cout << sizeof(void*) << "\n";     // 8 (same!)

// On 32-bit system: all would be 4
// On 64-bit system: all are 8
// Because a pointer stores an ADDRESS, and addresses are 64 bits on 64-bit systems
```

---

## Chapter 2: Pointer Arithmetic

### 2.1 Adding/Subtracting from Pointers

Pointer arithmetic operates in units of the **pointed-to type size**:

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* ptr = arr;    // Points to arr[0]

ptr + 1;   // Points to arr[1] (advances by sizeof(int) = 4 bytes)
ptr + 2;   // Points to arr[2] (advances by 8 bytes)
ptr + 3;   // Points to arr[3] (advances by 12 bytes)

*(ptr + 0)  // 10 — same as arr[0]
*(ptr + 1)  // 20 — same as arr[1]
*(ptr + 2)  // 30 — same as arr[2]

// In fact, arr[i] is EXACTLY equivalent to *(arr + i)
// And i[arr] also works! *(i + arr) — same thing due to commutativity
```

**The Equivalence:** `arr[i]` ≡ `*(arr + i)` ≡ `*(i + arr)` ≡ `i[arr]`

### 2.2 Pointer Difference

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* p1 = &arr[1];
int* p2 = &arr[4];

ptrdiff_t diff = p2 - p1;  // 3 (number of elements between them)
// NOT 12 (bytes) — it's 3 elements
```

### 2.3 Pointer Comparison

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* p1 = &arr[1];
int* p2 = &arr[3];

if (p1 < p2) {
    std::cout << "p1 points to a lower address\n";  // True
}
```

### 2.4 Iterating with Pointers

```cpp
int arr[5] = {10, 20, 30, 40, 50};

// Using pointer arithmetic (classic C style):
for (int* p = arr; p < arr + 5; p++) {
    std::cout << *p << " ";
}

// This is exactly what iterators do internally!
```

---

## Chapter 3: Null Pointers

### 3.1 The Three Ways to Express "Points to Nothing"

```cpp
int* p1 = nullptr;    // C++11 — ALWAYS use this
int* p2 = NULL;       // C/C++03 — legacy, avoid
int* p3 = 0;          // Works but misleading — avoid
```

### 3.2 Why `nullptr` is Better

```cpp
void func(int x)    { std::cout << "int\n"; }
void func(int* p)   { std::cout << "pointer\n"; }

func(NULL);       // Calls func(int)! NULL is #define NULL 0
func(nullptr);    // Calls func(int*)! nullptr is truly a pointer type
```

`nullptr` has type `std::nullptr_t`, which is implicitly convertible to any pointer type but NOT to integral types.

### 3.3 Always Check for Null

```cpp
void processData(int* data) {
    if (data == nullptr) {
        std::cerr << "Error: null pointer\n";
        return;
    }
    // Safe to dereference
    std::cout << *data << "\n";
}
```

### 3.4 Dereferencing Null — Undefined Behavior

```cpp
int* ptr = nullptr;
*ptr = 42;            // UNDEFINED BEHAVIOR! Typically: segmentation fault / crash
```

---

## Chapter 4: Pointers and Arrays

### 4.1 Array Name as Pointer

The name of an array is (in most contexts) a pointer to its first element:

```cpp
int arr[5] = {10, 20, 30, 40, 50};

int* ptr = arr;         // arr decays to &arr[0]
std::cout << *ptr;       // 10
std::cout << *(ptr + 2); // 30
std::cout << ptr[2];     // 30 (same thing)
```

### 4.2 Where Array Name is NOT a Pointer

```cpp
int arr[5] = {10, 20, 30, 40, 50};

sizeof(arr);   // 20 (5 × 4 bytes) — sizeof sees it as an array
&arr;          // Type is int(*)[5] (pointer to array of 5 ints), NOT int**
```

### 4.3 Dynamic Arrays

```cpp
int n = 10;
int* arr = new int[n];           // Allocate array on heap

for (int i = 0; i < n; i++) {
    arr[i] = i * 10;              // Use like normal array
}

delete[] arr;                     // MUST use delete[] for arrays (not delete)
arr = nullptr;                    // Good practice: nullify after delete

// Modern alternative — use std::vector!
std::vector<int> vec(n);
// No manual memory management needed
```

---

## Chapter 5: Pointer to Pointer

### 5.1 Multi-Level Pointers

```cpp
int x = 42;
int* ptr = &x;       // ptr points to x
int** pptr = &ptr;    // pptr points to ptr

std::cout << x;       // 42
std::cout << *ptr;    // 42
std::cout << **pptr;  // 42

**pptr = 100;         // Changes x to 100
```

**Memory diagram:**
```
Variable:  x          ptr           pptr
Value:     42         0x1000        0x1008
Address:   0x1000     0x1008        0x1010
           ↑__________|             |
                       ↑____________|
```

### 5.2 Use Cases

**Dynamic 2D array:**
```cpp
int rows = 3, cols = 4;
int** matrix = new int*[rows];
for (int i = 0; i < rows; i++) {
    matrix[i] = new int[cols];
}

matrix[1][2] = 42;  // Access like 2D array

// Cleanup (reverse order):
for (int i = 0; i < rows; i++) {
    delete[] matrix[i];
}
delete[] matrix;
```

**Modifying a pointer through a function:**
```cpp
void allocate(int** ptr, int value) {
    *ptr = new int(value);
}

int* p = nullptr;
allocate(&p, 42);
std::cout << *p;    // 42
delete p;
```

---

## Chapter 6: Void Pointers

### 6.1 The Generic Pointer

`void*` can point to any type but cannot be dereferenced directly:

```cpp
int x = 42;
double d = 3.14;
char c = 'A';

void* vp;

vp = &x;   // OK
vp = &d;   // OK
vp = &c;   // OK

// *vp;     // ERROR! Can't dereference void*
// Must cast first:
int value = *static_cast<int*>(vp);  // Cast back to int*
```

### 6.2 Use Cases

- **C library functions:** `malloc`, `memcpy`, `qsort` use `void*`
- **Type-erased containers** (rarely needed in modern C++ — use templates or `std::any`)

---

## Chapter 7: References

### 7.1 What is a Reference?

A **reference** is an **alias** — another name for an existing variable. It is NOT a separate object.

```cpp
int x = 42;
int& ref = x;    // ref IS x — they share the same memory

std::cout << x;    // 42
std::cout << ref;  // 42

ref = 100;         // Changes x!
std::cout << x;    // 100

std::cout << &x;   // Same address
std::cout << &ref; // Same address!
```

### 7.2 Reference Rules

1. **Must be initialized when declared:**
```cpp
int& ref;        // ERROR! Must be initialized
int& ref = x;   // OK
```

2. **Cannot be reseated (changed to reference another variable):**
```cpp
int x = 10, y = 20;
int& ref = x;
ref = y;         // Does NOT make ref alias y — it COPIES y's value into x!
std::cout << x;  // 20 (x was changed, ref still aliases x)
```

3. **Cannot be null:**
```cpp
int& ref = nullptr;  // ERROR! References can't be null
```

4. **Cannot reference temporaries (unless const or rvalue reference):**
```cpp
int& ref = 42;         // ERROR! 42 is a temporary
const int& ref = 42;   // OK! const extends the temporary's lifetime
int&& ref = 42;        // OK! rvalue reference (C++11)
```

### 7.3 Pointers vs References

| Feature | Pointer | Reference |
|---------|---------|-----------|
| Can be null | Yes | No |
| Can be reseated | Yes | No |
| Has own address | Yes | No (shares aliased object's address) |
| Must be initialized | No (but should be) | Yes |
| Syntax for access | `*ptr` and `ptr->` | Direct: `ref` and `ref.` |
| Can point to nothing | Yes (nullptr) | No (always valid if not dangling) |
| Indirection visible | Yes (explicit `*`) | No (transparent) |

### 7.4 When to Use Which

| Scenario | Use |
|----------|-----|
| Optional parameter (may be null) | Pointer |
| Always-valid alias | Reference |
| Reseatable binding | Pointer |
| Function output parameter | Reference |
| Large read-only parameter | `const&` |
| Dynamic allocation | Pointer (or smart pointer) |
| Operator overloading | Reference |

### 7.5 Reference Parameters

```cpp
// Pass by reference — modify the original:
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int x = 10, y = 20;
swap(x, y);
// x = 20, y = 10

// Pass by const reference — read-only, no copy:
double calculateLength(const std::string& s) {
    return s.length();
}
```

### 7.6 At the Assembly Level

References are typically implemented as pointers by the compiler:

```cpp
int x = 42;
int& ref = x;
ref = 100;

// Compiler generates the same code as:
int x = 42;
int* ptr = &x;
*ptr = 100;
```

The difference is purely a **compile-time** concept — the compiler enforces reference rules, then generates pointer-like code.

---

## Chapter 8: Advanced Pointer Topics

### 8.1 `const` with Pointers — Complete Guide

```cpp
int x = 10, y = 20;

// 1. Pointer to non-const (can change both pointer and value):
int* p1 = &x;
*p1 = 15;     // OK
p1 = &y;      // OK

// 2. Pointer to const (can change pointer, NOT value):
const int* p2 = &x;
// *p2 = 15;  // ERROR
p2 = &y;      // OK

// 3. Const pointer to non-const (can change value, NOT pointer):
int* const p3 = &x;
*p3 = 15;     // OK
// p3 = &y;   // ERROR

// 4. Const pointer to const (can change NOTHING):
const int* const p4 = &x;
// *p4 = 15;  // ERROR
// p4 = &y;   // ERROR
```

### 8.2 Function Pointers

```cpp
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

int (*operation)(int, int) = add;
std::cout << operation(5, 3);   // 8

operation = sub;
std::cout << operation(5, 3);   // 2

// Array of function pointers:
int (*ops[])(int, int) = {add, sub};
std::cout << ops[0](5, 3);  // 8
std::cout << ops[1](5, 3);  // 2
```

### 8.3 Dangling Pointers

A pointer that references memory that has been freed or gone out of scope:

```cpp
// Case 1: Deleted heap memory
int* ptr = new int(42);
delete ptr;
// ptr is now dangling — still holds the old address
*ptr = 10;       // UNDEFINED BEHAVIOR!
ptr = nullptr;   // Fix: nullify after delete

// Case 2: Out-of-scope local variable
int* getDangling() {
    int local = 42;
    return &local;   // WARNING: returning address of local!
}
int* ptr = getDangling();  // ptr is dangling!

// Case 3: Invalidated iterator/pointer
std::vector<int> v = {1, 2, 3};
int* ptr = &v[0];
v.push_back(4);   // May reallocate! ptr is potentially dangling!
```

### 8.4 Smart Pointer Preview

Smart pointers (covered fully in Book 14) automate memory management:

```cpp
#include <memory>

auto ptr = std::make_unique<int>(42);  // Automatically deletes when out of scope
std::cout << *ptr;  // 42
// No manual delete needed!

auto shared = std::make_shared<int>(42);  // Reference counted
auto copy = shared;  // Both point to same int, ref count = 2
// Deleted when last shared_ptr is destroyed
```

---

## Chapter 9: Interview & Competitive Programming Section

### Common Questions

**Q1: What happens if you dereference a null pointer?**
A: Undefined behavior. Typically causes a segmentation fault (crash), but the compiler is free to do anything.

**Q2: What is a dangling pointer?**
A: A pointer that holds an address of memory that has been freed or is out of scope. Using it is UB.

**Q3: What is the difference between `int* const` and `const int*`?**
A: `const int*` is a pointer to a constant int (value can't change). `int* const` is a constant pointer to an int (pointer can't be reseated).

**Q4: Can a reference be null?**
A: No. References must always alias a valid object. Use pointers when nullability is needed.

**Q5: What is pointer decay?**
A: When an array name is used in most expressions, it "decays" to a pointer to its first element, losing size information.

### Tricky Problems

```cpp
// Q: What is the output?
int arr[] = {10, 20, 30, 40, 50};
int* p = arr;
std::cout << *(p + 3) << " " << p[3] << " " << 3[p];
// Answer: 40 40 40 (all equivalent)

// Q: What is sizeof?
int arr[10];
int* ptr = arr;
std::cout << sizeof(arr) << " " << sizeof(ptr);
// Answer: 40 8 (on 64-bit system)
```

---

## Chapter 10: Practice & Mini Projects

### Exercises
1. Implement `swap` using pointers, references, and a temp variable.
2. Write a function that reverses an array using pointer arithmetic (no indexing).
3. Implement a singly linked list using raw pointers.
4. Write a function to find if two arrays have common elements using pointer-based comparison.

### Project 1: Memory Visualizer (Beginner)
Print the address, value, and pointed-to values of variables, pointers, and double pointers.

### Project 2: Custom String Class (Intermediate)
Implement `MyString` with a `char*` buffer: constructor, destructor, copy, append, compare, find.

### Project 3: Polynomial Calculator (Advanced)
Represent polynomials as linked lists of terms. Support addition, multiplication, evaluation.

---

## Chapter 11: Summary & Mastery Checklist

- [ ] You understand what pointers are and how they work in memory
- [ ] You can perform pointer arithmetic and understand its relationship to arrays
- [ ] You know why `nullptr` is preferred over `NULL`
- [ ] You can use multi-level pointers and understand their use cases
- [ ] You understand references, their rules, and how they differ from pointers
- [ ] You know the four combinations of `const` with pointers
- [ ] You can identify and avoid dangling pointers
- [ ] You understand `void*` and function pointers
- [ ] You know when to use pointers vs references

**If all mastered, proceed to Book 8: Structures & Dynamic Memory.**

---

*End of Book 7: Pointers & References*
*Next: Book 8 — Structures & Dynamic Memory*
