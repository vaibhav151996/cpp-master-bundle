# Book 14: Smart Pointers and Memory Management

## The Complete Guide to Ownership, Lifetimes, and Resource Management in C++

---

### **Target Level:** Intermediate → Advanced
### **Prerequisites:** Books 1–13
### **Learning Outcomes:**
By the end of this book, you will:
- Master `unique_ptr`, `shared_ptr`, and `weak_ptr`
- Understand ownership semantics and RAII
- Solve circular references, dangling pointers, and memory leaks
- Know when to use raw pointers vs smart pointers
- Implement custom deleters and allocators
- Understand the internal implementation of each smart pointer

---

## Chapter 1: The Problem — Why Smart Pointers?

### 1.1 Raw Pointer Pitfalls

```cpp
// Problem 1: Memory leaks
void leaky() {
    int* p = new int(42);
    if (someCondition()) return;  // LEAK! p never deleted
    delete p;
}

// Problem 2: Double delete
int* p = new int(42);
int* q = p;
delete p;
delete q;  // UNDEFINED BEHAVIOR: double delete

// Problem 3: Dangling pointer
int* p = new int(42);
int* q = p;
delete p;
std::cout << *q;  // UNDEFINED BEHAVIOR: dangling pointer

// Problem 4: Exception safety
void unsafe() {
    int* a = new int(1);
    int* b = new int(2);  // If this throws, a leaks!
    delete a;
    delete b;
}
```

### 1.2 RAII — Resource Acquisition Is Initialization

The foundational C++ idiom: tie resource lifetime to object lifetime.

```cpp
class FileHandle {
    FILE* f_;
public:
    FileHandle(const char* name) : f_(fopen(name, "r")) {
        if (!f_) throw std::runtime_error("Can't open file");
    }
    ~FileHandle() { if (f_) fclose(f_); }  // Always cleaned up

    // Non-copyable:
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    // Movable:
    FileHandle(FileHandle&& other) noexcept : f_(other.f_) {
        other.f_ = nullptr;
    }
};

void safe() {
    FileHandle fh("data.txt");
    // Even if exception thrown here, destructor runs → file closed
}
```

Smart pointers are RAII wrappers for heap-allocated objects.

---

## Chapter 2: `std::unique_ptr` — Exclusive Ownership

### 2.1 Basics

```cpp
#include <memory>

// Creation — ALWAYS use make_unique:
auto p = std::make_unique<int>(42);

// Access:
std::cout << *p;      // Dereference
std::cout << p.get(); // Raw pointer (don't delete it!)

// Cannot copy (exclusive ownership):
// auto p2 = p;        // ERROR: deleted copy constructor

// Can move (transfer ownership):
auto p2 = std::move(p);
// p is now nullptr, p2 owns the resource

// Automatic cleanup:
{
    auto p = std::make_unique<int>(42);
}  // p goes out of scope → memory freed automatically
```

### 2.2 `unique_ptr` with Arrays

```cpp
auto arr = std::make_unique<int[]>(10);  // Array of 10 ints
arr[0] = 42;
// Automatically calls delete[] instead of delete
```

### 2.3 `unique_ptr` with Custom Deleters

```cpp
// Custom deleter for C resources:
auto fileDeleter = [](FILE* f) { if (f) fclose(f); };
std::unique_ptr<FILE, decltype(fileDeleter)> file(
    fopen("data.txt", "r"), fileDeleter
);

// Custom deleter for COM objects:
struct ComDeleter {
    void operator()(IUnknown* p) const { if (p) p->Release(); }
};
std::unique_ptr<IUnknown, ComDeleter> comPtr(createComObject());
```

### 2.4 `unique_ptr` in Containers

```cpp
std::vector<std::unique_ptr<Animal>> zoo;
zoo.push_back(std::make_unique<Dog>("Rex"));
zoo.push_back(std::make_unique<Cat>("Whiskers"));

for (const auto& animal : zoo) {
    animal->speak();  // Polymorphism + ownership
}
```

### 2.5 Factory Pattern with `unique_ptr`

```cpp
std::unique_ptr<Shape> createShape(const std::string& type) {
    if (type == "circle") return std::make_unique<Circle>();
    if (type == "square") return std::make_unique<Square>();
    return nullptr;
}
```

### 2.6 Internals

`unique_ptr` has **zero overhead** compared to a raw pointer:
- Same size as raw pointer (no reference count)
- Destructor calls `delete` (or custom deleter)
- No heap allocation beyond the managed object

---

## Chapter 3: `std::shared_ptr` — Shared Ownership

### 3.1 Basics

```cpp
// Creation — ALWAYS use make_shared:
auto p = std::make_shared<int>(42);

// Can be copied (shared ownership):
auto p2 = p;  // Both own the resource
// Reference count: 2

std::cout << p.use_count();  // 2

p.reset();   // p releases ownership (count → 1)
p2.reset();  // p2 releases ownership (count → 0) → memory freed
```

### 3.2 Internals: Control Block

`make_shared` allocates ONE block containing both:
1. The managed object
2. A control block (reference counts + deleter + allocator)

```
┌─────────────────────────────┐
│ Control Block               │
│ ┌─────────────────────────┐ │
│ │ strong_count: 2         │ │
│ │ weak_count: 0           │ │
│ │ deleter                 │ │
│ │ allocator               │ │
│ └─────────────────────────┘ │
│ Object Data (e.g., int 42)  │
└─────────────────────────────┘
```

### 3.3 `shared_ptr` from Raw Pointer (AVOID)

```cpp
int* raw = new int(42);
std::shared_ptr<int> sp1(raw);
// std::shared_ptr<int> sp2(raw);  // DISASTER! Two control blocks!
// Both will try to delete raw → double delete

// RULE: Never create two shared_ptrs from the same raw pointer.
// Use make_shared or copy/move an existing shared_ptr.
```

### 3.4 `enable_shared_from_this`

```cpp
class Widget : public std::enable_shared_from_this<Widget> {
public:
    std::shared_ptr<Widget> getShared() {
        return shared_from_this();  // Safe: uses existing control block
    }
};

auto w = std::make_shared<Widget>();
auto w2 = w->getShared();  // Same control block
```

### 3.5 `shared_ptr` with Custom Deleters

```cpp
// Different syntax than unique_ptr — deleter is NOT part of the type:
auto sp = std::shared_ptr<FILE>(
    fopen("data.txt", "r"),
    [](FILE* f) { if (f) fclose(f); }
);

// This means shared_ptrs with different deleters have the SAME TYPE.
// This is called "type erasure."
```

### 3.6 Thread Safety

- The control block (reference count) is **thread-safe** (atomic operations).
- The managed object is **NOT thread-safe** — you must synchronize access yourself.

```cpp
auto sp = std::make_shared<int>(42);
// SAFE: copying/assigning sp from multiple threads
// UNSAFE: modifying *sp from multiple threads without synchronization
```

### 3.7 Performance Cost

Compared to raw pointer:
- 2× the size (pointer to object + pointer to control block)
- Atomic reference count increment/decrement (10-20ns overhead per copy)
- One extra heap allocation if not using `make_shared`

---

## Chapter 4: `std::weak_ptr` — Non-Owning Observer

### 4.1 Purpose

`weak_ptr` observes a `shared_ptr`-managed object without affecting its lifetime. Solves **circular reference** problems.

```cpp
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;

// weak_ptr does NOT increment the reference count
std::cout << sp.use_count();  // Still 1

// To access the object, lock() creates a temporary shared_ptr:
if (auto locked = wp.lock()) {
    std::cout << *locked;  // 42
} else {
    std::cout << "Object was destroyed";
}

sp.reset();  // Object destroyed (count → 0)
std::cout << wp.expired();  // true
```

### 4.2 Solving Circular References

```cpp
// BROKEN — circular reference → memory leak:
struct Node {
    std::shared_ptr<Node> next;
    std::shared_ptr<Node> prev;  // ← PROBLEM!
    ~Node() { std::cout << "Destroyed\n"; }  // Never called!
};

auto a = std::make_shared<Node>();
auto b = std::make_shared<Node>();
a->next = b;
b->prev = a;
// a has ref count 2 (a + b->prev)
// b has ref count 2 (b + a->next)
// Neither ever reaches 0 → LEAK!

// FIXED — use weak_ptr for back-reference:
struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;    // ← FIXED!
    ~Node() { std::cout << "Destroyed\n"; }
};
```

### 4.3 Caching Pattern

```cpp
class ImageCache {
    std::unordered_map<std::string, std::weak_ptr<Image>> cache_;

public:
    std::shared_ptr<Image> getImage(const std::string& path) {
        auto it = cache_.find(path);
        if (it != cache_.end()) {
            if (auto sp = it->second.lock()) {
                return sp;  // Cache hit
            }
            cache_.erase(it);  // Expired, clean up
        }
        // Cache miss — load image
        auto img = std::make_shared<Image>(path);
        cache_[path] = img;
        return img;
    }
};
```

---

## Chapter 5: Ownership Guidelines

### 5.1 When To Use What

| Scenario | Use |
|----------|-----|
| Single owner, no sharing | `std::unique_ptr` (factory returns, class members) |
| Multiple owners | `std::shared_ptr` |
| Non-owning observer | `std::weak_ptr` or raw pointer |
| No ownership semantics | Raw pointer or reference |
| Stack-allocated objects | Automatic storage (no pointer needed) |

### 5.2 Function Parameter Guidelines (Herb Sutter's Rules)

```cpp
// Don't pass smart pointers unless you need ownership semantics:

// ✅ READ access — use const reference:
void read(const Widget& w);

// ✅ MODIFY existing object — use reference:
void modify(Widget& w);

// ✅ TAKE ownership — use unique_ptr by value:
void takeOwnership(std::unique_ptr<Widget> w);

// ✅ SHARE ownership — use shared_ptr by value:
void shareOwnership(std::shared_ptr<Widget> w);

// ❌ DON'T: pass const unique_ptr& (just pass Widget& instead)
// ❌ DON'T: pass const shared_ptr& (doesn't share ownership)
```

### 5.3 The Rule of Zero

Prefer classes that need no custom destructor, copy/move operations:

```cpp
class Good {
    std::string name_;
    std::vector<int> data_;
    std::unique_ptr<Impl> impl_;
    // Compiler-generated destructor, move, copy (where possible) are perfect
};
```

---

## Chapter 6: Advanced Topics

### 6.1 `std::make_shared` vs `std::shared_ptr` Constructor

```cpp
// make_shared: ONE allocation (object + control block together)
auto p1 = std::make_shared<Widget>();

// Constructor: TWO allocations (object + control block separate)
auto p2 = std::shared_ptr<Widget>(new Widget());

// make_shared advantages:
// - Faster (one allocation instead of two)
// - Exception safe
// make_shared disadvantages:
// - Can't use custom deleter
// - Memory for object can't be freed until all weak_ptrs are gone
```

### 6.2 Aliasing Constructor

```cpp
struct Foo {
    int member;
};

auto foo = std::make_shared<Foo>();
// Create a shared_ptr to member that shares ownership with foo:
std::shared_ptr<int> memberPtr(foo, &foo->member);
// foo and memberPtr share the same control block
```

### 6.3 `std::unique_ptr` with Pimpl Pattern

```cpp
// widget.h
class Widget {
    struct Impl;
    std::unique_ptr<Impl> pImpl_;
public:
    Widget();
    ~Widget();  // Must be declared (defined in .cpp where Impl is complete)
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;
};

// widget.cpp
struct Widget::Impl {
    std::string name;
    std::vector<int> data;
};

Widget::Widget() : pImpl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;  // Impl is complete here
Widget::Widget(Widget&&) noexcept = default;
Widget& Widget::operator=(Widget&&) noexcept = default;
```

### 6.4 Custom Allocators

```cpp
// Pool allocator for shared_ptr:
auto sp = std::allocate_shared<Widget>(PoolAllocator<Widget>{});
```

---

## Chapter 7: Common Mistakes & Anti-Patterns

```cpp
// ❌ MISTAKE 1: Using new instead of make_unique/make_shared
auto p = std::unique_ptr<int>(new int(42));  // BAD
auto p = std::make_unique<int>(42);          // GOOD

// ❌ MISTAKE 2: Unnecessary shared_ptr (unique_ptr suffices)
std::shared_ptr<Widget> w = std::make_shared<Widget>();  // Overkill if only one owner

// ❌ MISTAKE 3: Storing raw pointer from smart pointer and using after destruction
int* raw = nullptr;
{
    auto sp = std::make_shared<int>(42);
    raw = sp.get();
}
// raw is now dangling! sp was destroyed

// ❌ MISTAKE 4: Circular shared_ptr references (use weak_ptr)

// ❌ MISTAKE 5: Creating shared_ptr from 'this'
class Bad {
    std::shared_ptr<Bad> getSelf() {
        return std::shared_ptr<Bad>(this);  // DOUBLE DELETE!
    }
};
// FIX: inherit from enable_shared_from_this

// ❌ MISTAKE 6: Using shared_ptr for polymorphic arrays
// shared_ptr<Base[]> arr(new Derived[10]);  // UNDEFINED BEHAVIOR
```

---

## Chapter 8: Interview Questions

**Q1: What is the difference between `unique_ptr` and `shared_ptr`?** — `unique_ptr` has exclusive ownership (non-copyable, zero overhead). `shared_ptr` has shared ownership (copyable, reference-counted, 2× size + atomic overhead).

**Q2: What happens if you have circular `shared_ptr` references?** — Memory leak. Neither object's reference count reaches zero. Solution: use `weak_ptr` for one direction.

**Q3: Why prefer `make_shared` over `shared_ptr(new T())`?** — `make_shared` does one allocation instead of two, is exception-safe, and is more cache-friendly.

**Q4: Can you convert `unique_ptr` to `shared_ptr`?** — Yes: `std::shared_ptr<T> sp = std::move(up);`. Cannot go the other direction.

**Q5: What is `enable_shared_from_this` for?** — Allows an object managed by `shared_ptr` to safely create additional `shared_ptr` instances pointing to itself without double-deleting.

---

## Chapter 9: Practice & Mini Projects

### Project 1: Graph with Smart Pointers — Implement a graph using `shared_ptr` for nodes and `weak_ptr` for edges to avoid cycles.
### Project 2: Resource Pool — Implement an object pool that uses `shared_ptr` with custom deleters to return objects to the pool.
### Project 3: Plugin System — Use `unique_ptr` with factory functions to load and manage dynamically-created plugin objects polymorphically.

---

## Chapter 10: Mastery Checklist

- [ ] You always use `make_unique`/`make_shared` instead of `new`
- [ ] You understand ownership semantics (unique, shared, non-owning)
- [ ] You can solve circular reference problems with `weak_ptr`
- [ ] You know the internal structure of `shared_ptr` (control block)
- [ ] You apply the Rule of Zero in your class designs
- [ ] You understand custom deleters, aliasing constructor, Pimpl pattern
- [ ] You can explain thread safety guarantees of `shared_ptr`

---

*End of Book 14: Smart Pointers and Memory Management*
