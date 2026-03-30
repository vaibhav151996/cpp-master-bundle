# Book 21: Essential Topics — Exception Handling, File I/O, Namespaces, Preprocessor, Type Casting, Enums & More

## Comprehensive Coverage of Critical C++ Topics Not Fully Covered Elsewhere

---

### **Target Level:** Beginner → Advanced
### **Prerequisites:** Books 1–6 minimum
### **Learning Outcomes:**
By the end of this book, you will:
- Master C++ exception handling (try/catch/throw, custom exceptions, exception safety)
- Perform all types of file I/O (text, binary, streams)
- Understand namespaces and code organization
- Know all preprocessor directives and macros
- Master all C++ casting operators
- Use enums, unions, and bitfields effectively

---

## Chapter 1: Exception Handling

### 1.1 The Basics

```cpp
#include <stdexcept>
#include <iostream>

double divide(double a, double b) {
    if (b == 0.0)
        throw std::runtime_error("Division by zero");
    return a / b;
}

int main() {
    try {
        double result = divide(10.0, 0.0);
        std::cout << result;
    }
    catch (const std::runtime_error& e) {
        std::cerr << "Error: " << e.what() << "\n";
    }
    catch (const std::exception& e) {
        std::cerr << "General error: " << e.what() << "\n";
    }
    catch (...) {
        std::cerr << "Unknown error\n";
    }
}
```

### 1.2 Standard Exception Hierarchy

```
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::domain_error
│   ├── std::length_error
│   └── std::out_of_range
├── std::runtime_error
│   ├── std::range_error
│   ├── std::overflow_error
│   └── std::underflow_error
├── std::bad_alloc            (new fails)
├── std::bad_cast             (dynamic_cast fails)
├── std::bad_typeid           (typeid on null polymorphic pointer)
└── std::bad_exception
```

### 1.3 Custom Exceptions

```cpp
class DatabaseError : public std::runtime_error {
    int errorCode_;
public:
    DatabaseError(const std::string& msg, int code)
        : std::runtime_error(msg), errorCode_(code) {}

    int code() const { return errorCode_; }
};

class ConnectionError : public DatabaseError {
public:
    ConnectionError(const std::string& host)
        : DatabaseError("Cannot connect to " + host, 1001) {}
};

// Usage:
try {
    throw ConnectionError("db.example.com");
} catch (const DatabaseError& e) {
    std::cerr << e.what() << " (code: " << e.code() << ")\n";
}
```

### 1.4 Exception Safety Guarantees

| Guarantee | Description | Example |
|-----------|-------------|---------|
| **No-throw** | Never throws. Use `noexcept`. | Destructors, move operations |
| **Strong** | If exception thrown, state unchanged (rollback). | `std::vector::push_back` |
| **Basic** | If exception thrown, invariants maintained, no leaks. | Most operations |
| **No guarantee** | Anything can happen. | Avoid this! |

```cpp
// noexcept specifier:
void safeCleanup() noexcept {
    // MUST NOT throw — if it does, std::terminate() is called
}

// Move operations should be noexcept for STL compatibility:
class Widget {
public:
    Widget(Widget&& other) noexcept;              // Move constructor
    Widget& operator=(Widget&& other) noexcept;   // Move assignment
};
// Why? std::vector::push_back uses move only if noexcept,
// otherwise falls back to copy (for strong exception safety)

// noexcept operator (query):
static_assert(noexcept(safeCleanup()));
```

### 1.5 RAII and Exception Safety

```cpp
// BAD — exception causes leak:
void bad() {
    int* p = new int(42);
    riskyOperation();   // If this throws, p is leaked!
    delete p;
}

// GOOD — RAII handles cleanup:
void good() {
    auto p = std::make_unique<int>(42);
    riskyOperation();   // If this throws, unique_ptr destructor frees p
}

// GOOD — lock_guard handles unlock:
void safeLock(std::mutex& m) {
    std::lock_guard lock(m);
    riskyOperation();   // If this throws, mutex is automatically unlocked
}
```

### 1.6 When NOT to Use Exceptions

```
DON'T use exceptions for:
- Normal control flow (use return values, optional, expected)
- Performance-critical hot loops
- Embedded systems with -fno-exceptions
- Conditions that are expected to occur frequently

DO use exceptions for:
- Truly exceptional conditions (file not found, out of memory)
- Constructor failures (only way to signal error)
- Deep call stacks where error must propagate many levels
```

### 1.7 `std::expected` (C++23) — Modern Error Handling

```cpp
#include <expected>

std::expected<int, std::string> parseInt(const std::string& s) {
    try {
        return std::stoi(s);
    } catch (...) {
        return std::unexpected("Invalid integer: " + s);
    }
}

auto result = parseInt("abc");
if (result) {
    std::cout << *result;
} else {
    std::cerr << result.error();
}
```

---

## Chapter 2: File I/O

### 2.1 Text File I/O

```cpp
#include <fstream>
#include <string>

// Writing:
std::ofstream outFile("output.txt");
if (outFile.is_open()) {
    outFile << "Hello, World!\n";
    outFile << "Line 2\n";
    outFile.close();
}

// Reading — line by line:
std::ifstream inFile("output.txt");
std::string line;
while (std::getline(inFile, line)) {
    std::cout << line << "\n";
}

// Reading — word by word:
std::ifstream wordFile("data.txt");
std::string word;
while (wordFile >> word) {
    std::cout << word << " ";
}

// Reading entire file into string:
std::ifstream file("data.txt");
std::string content(
    (std::istreambuf_iterator<char>(file)),
    std::istreambuf_iterator<char>()
);

// Append mode:
std::ofstream appendFile("log.txt", std::ios::app);
appendFile << "New log entry\n";
```

### 2.2 Binary File I/O

```cpp
// Writing binary:
struct Record {
    int id;
    double value;
    char name[32];
};

Record rec{1, 3.14, "Alice"};
std::ofstream binOut("data.bin", std::ios::binary);
binOut.write(reinterpret_cast<const char*>(&rec), sizeof(rec));
binOut.close();

// Reading binary:
Record loaded;
std::ifstream binIn("data.bin", std::ios::binary);
binIn.read(reinterpret_cast<char*>(&loaded), sizeof(loaded));
std::cout << loaded.id << " " << loaded.value << " " << loaded.name;
```

### 2.3 File Positioning

```cpp
std::fstream file("data.bin", std::ios::in | std::ios::out | std::ios::binary);

// Get position:
auto pos = file.tellg();  // Get (read) position
auto wpos = file.tellp(); // Put (write) position

// Set position:
file.seekg(0, std::ios::beg);   // Seek to beginning
file.seekg(0, std::ios::end);   // Seek to end
file.seekg(-10, std::ios::cur); // Seek back 10 bytes from current
file.seekg(100);                // Seek to absolute position 100

// Get file size:
file.seekg(0, std::ios::end);
auto fileSize = file.tellg();
file.seekg(0, std::ios::beg);
```

### 2.4 String Streams

```cpp
#include <sstream>

// String → values:
std::istringstream iss("42 3.14 hello");
int i; double d; std::string s;
iss >> i >> d >> s;  // i=42, d=3.14, s="hello"

// Values → string (formatting):
std::ostringstream oss;
oss << "Value: " << 42 << ", Pi: " << std::fixed << std::setprecision(2) << 3.14159;
std::string result = oss.str();  // "Value: 42, Pi: 3.14"

// Parsing CSV:
std::string csvLine = "Alice,30,Developer";
std::istringstream lineStream(csvLine);
std::string field;
while (std::getline(lineStream, field, ',')) {
    std::cout << field << "\n";
}
```

### 2.5 `std::filesystem` (C++17)

```cpp
#include <filesystem>
namespace fs = std::filesystem;

// Check existence:
if (fs::exists("data.txt")) { /* ... */ }

// File properties:
auto size = fs::file_size("data.txt");
auto lastWrite = fs::last_write_time("data.txt");

// Create/remove:
fs::create_directory("output");
fs::create_directories("a/b/c");
fs::remove("temp.txt");
fs::remove_all("output");  // Recursive

// Copy/rename:
fs::copy("src.txt", "dst.txt");
fs::rename("old.txt", "new.txt");

// Path manipulation:
fs::path p = "/home/user/file.txt";
p.filename();     // "file.txt"
p.stem();         // "file"
p.extension();    // ".txt"
p.parent_path();  // "/home/user"
p / "subdir";     // "/home/user/file.txt/subdir"

// Iterate directory:
for (const auto& entry : fs::directory_iterator(".")) {
    std::cout << entry.path().filename() << "\n";
}
```

---

## Chapter 3: Namespaces

### 3.1 Basics

```cpp
namespace Math {
    constexpr double PI = 3.14159265358979;

    double circleArea(double r) { return PI * r * r; }

    namespace Trig {
        double degToRad(double deg) { return deg * PI / 180.0; }
    }
}

// Usage:
double area = Math::circleArea(5.0);
double rad = Math::Trig::degToRad(90.0);
```

### 3.2 Nested Namespaces (C++17)

```cpp
namespace Company::Product::Module {
    void function() {}
}
// Equivalent to: namespace Company { namespace Product { namespace Module { ... } } }
```

### 3.3 `using` Declarations and Directives

```cpp
// using declaration — import specific name:
using Math::PI;
double area = PI * r * r;

// using directive — import everything (AVOID in headers):
using namespace Math;
double area = circleArea(5.0);

// using directive in .cpp file (acceptable):
using namespace std;  // Only in implementation files, never in headers

// Type alias with using:
using StringVec = std::vector<std::string>;
using Callback = std::function<void(int)>;
```

### 3.4 Anonymous Namespace (Internal Linkage)

```cpp
namespace {
    int helper() { return 42; }   // Only visible in this translation unit
    int localVar = 0;             // Equivalent to static at file scope
}
// Preferred over static for functions/variables with internal linkage
```

### 3.5 Inline Namespaces (Versioning)

```cpp
namespace MyLib {
    inline namespace v2 {
        void process() { /* v2 implementation */ }
    }
    namespace v1 {
        void process() { /* v1 implementation */ }
    }
}

MyLib::process();      // Calls v2 (inline namespace)
MyLib::v1::process();  // Explicitly call v1
```

---

## Chapter 4: Preprocessor Directives

### 4.1 Include Guards

```cpp
// Traditional include guard:
#ifndef MYHEADER_H
#define MYHEADER_H

// Header content

#endif // MYHEADER_H

// Modern alternative (non-standard but universally supported):
#pragma once
```

### 4.2 Macros

```cpp
// Object-like macro:
#define MAX_SIZE 1024
#define VERSION "2.0"

// Function-like macro (AVOID — prefer constexpr/templates):
#define MAX(a, b) ((a) > (b) ? (a) : (b))
// Pitfall: MAX(i++, j++) evaluates arguments multiple times!

// Variadic macro:
#define LOG(fmt, ...) printf(fmt "\n", ##__VA_ARGS__)

// Stringizing operator (#):
#define STRINGIFY(x) #x
std::cout << STRINGIFY(hello);  // Prints: hello

// Token pasting (##):
#define MAKE_VAR(name) var_##name
int MAKE_VAR(count) = 0;  // Expands to: int var_count = 0;

// Modern alternatives to macros:
// Instead of #define MAX_SIZE 1024:
constexpr int MAX_SIZE = 1024;

// Instead of #define MAX(a,b):
template<typename T>
constexpr T myMax(T a, T b) { return a > b ? a : b; }
```

### 4.3 Conditional Compilation

```cpp
// Platform detection:
#ifdef _WIN32
    #include <windows.h>
#elif defined(__linux__)
    #include <unistd.h>
#elif defined(__APPLE__)
    #include <TargetConditionals.h>
#endif

// Debug/Release:
#ifdef NDEBUG
    // Release mode
#else
    // Debug mode
    #define ASSERT(cond) if (!(cond)) { __debugbreak(); }
#endif

// Feature detection:
#if __cplusplus >= 202002L
    // C++20 features available
#elif __cplusplus >= 201703L
    // C++17 features available
#endif

// Compiler detection:
#if defined(__clang__)
    // Clang
#elif defined(__GNUC__)
    // GCC
#elif defined(_MSC_VER)
    // MSVC
#endif
```

### 4.4 Predefined Macros

```cpp
std::cout << __FILE__;         // Current file name
std::cout << __LINE__;         // Current line number
std::cout << __func__;         // Current function name (C++11)
std::cout << __DATE__;         // Compilation date "Jan 01 2024"
std::cout << __TIME__;         // Compilation time "12:34:56"
std::cout << __cplusplus;      // C++ standard version (202002 for C++20)
```

### 4.5 `#pragma`

```cpp
#pragma once                   // Include guard
#pragma warning(push)          // MSVC: save warning state
#pragma warning(disable: 4996) // MSVC: disable specific warning
#pragma warning(pop)           // MSVC: restore warning state
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wunused-variable"  // GCC/Clang
#pragma GCC diagnostic pop
#pragma pack(push, 1)          // Set struct packing alignment to 1
struct Packed { char a; int b; };  // sizeof = 5, not 8
#pragma pack(pop)
```

---

## Chapter 5: Type Casting

### 5.1 The Four C++ Casts

```cpp
// 1. static_cast — compile-time checked, most common:
double d = 3.14;
int i = static_cast<int>(d);          // double → int (truncation)
Base* b = static_cast<Base*>(derived); // Derived* → Base* (upcast)
Derived* d = static_cast<Derived*>(b); // Base* → Derived* (downcast, UNSAFE!)
// Use when: converting numeric types, upcasting, enum ↔ int

// 2. dynamic_cast — runtime-checked, requires polymorphic types (virtual):
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b);
if (d) {
    d->derivedMethod();  // Safe: verified at runtime
} else {
    // Cast failed — b doesn't point to Derived
}

// With references (throws std::bad_cast on failure):
try {
    Derived& d = dynamic_cast<Derived&>(*b);
} catch (const std::bad_cast& e) {
    std::cerr << "Cast failed\n";
}
// Use when: safe downcasting in polymorphic hierarchies

// 3. const_cast — add/remove const:
const std::string& s = "hello";
std::string& mutable_s = const_cast<std::string&>(s);
// ONLY safe if the original object was non-const!
// Use when: interfacing with C APIs that take non-const but don't modify

// 4. reinterpret_cast — bit-level reinterpretation:
int x = 42;
void* ptr = reinterpret_cast<void*>(&x);
int* back = reinterpret_cast<int*>(ptr);
// Use when: type-punning, hardware registers, serialization
// DANGEROUS: almost always undefined behavior unless specifically allowed
```

### 5.2 Casting Summary

| Cast | Checked | Use Case | Safety |
|------|---------|----------|--------|
| `static_cast` | Compile-time | Numeric conversions, upcasts | Medium |
| `dynamic_cast` | Runtime | Safe downcasts (polymorphic) | High |
| `const_cast` | Compile-time | Add/remove const | Low |
| `reinterpret_cast` | None | Bit reinterpretation | Very Low |

### 5.3 C-Style Cast (AVOID)

```cpp
int i = (int)3.14;            // C-style cast
int j = int(3.14);            // Functional cast

// These try static_cast, then const_cast, then reinterpret_cast.
// You don't know which one is applied — use explicit C++ casts instead.
```

---

## Chapter 6: Enums

### 6.1 Unscoped Enum (C Legacy)

```cpp
enum Color { Red, Green, Blue };
// Problems:
// - Names pollute enclosing scope
// - Implicit conversion to int
// - Can't forward declare

Color c = Red;
int x = c;         // Implicit conversion to int — undesirable
// if (c == 0) {}  // Compiles — Red == 0
```

### 6.2 Scoped Enum (C++11 — Preferred)

```cpp
enum class Color { Red, Green, Blue };
enum class Size : uint8_t { Small = 0, Medium = 1, Large = 2 };

Color c = Color::Red;
// int x = c;                   // ERROR: no implicit conversion
int x = static_cast<int>(c);    // Explicit conversion
// if (c == 0) {}               // ERROR: can't compare to int

// Underlying type:
static_assert(sizeof(Size) == 1);  // uint8_t

// Using enum (C++20):
void handleColor(Color c) {
    using enum Color;   // Bring all enumerators into scope
    switch (c) {
        case Red: break;    // Don't need Color::Red
        case Green: break;
        case Blue: break;
    }
}
```

### 6.3 Enum as Flags (Bitmask)

```cpp
enum class Permission : uint8_t {
    None    = 0,
    Read    = 1 << 0,   // 0001
    Write   = 1 << 1,   // 0010
    Execute = 1 << 2,   // 0100
    All     = Read | Write | Execute  // 0111
};

// Need to overload operators for enum class:
constexpr Permission operator|(Permission a, Permission b) {
    return static_cast<Permission>(
        static_cast<uint8_t>(a) | static_cast<uint8_t>(b)
    );
}

constexpr Permission operator&(Permission a, Permission b) {
    return static_cast<Permission>(
        static_cast<uint8_t>(a) & static_cast<uint8_t>(b)
    );
}

constexpr bool hasPermission(Permission p, Permission flag) {
    return (p & flag) == flag;
}

// Usage:
Permission perms = Permission::Read | Permission::Write;
if (hasPermission(perms, Permission::Read)) {
    std::cout << "Can read\n";
}
```

---

## Chapter 7: Unions and Bitfields

### 7.1 Unions

```cpp
union Data {
    int i;
    float f;
    char c;
};

Data d;
d.i = 42;
// d.f is now undefined — only one member active at a time!
// Reading inactive member is UNDEFINED BEHAVIOR (type punning)
```

### 7.2 `std::variant` (C++17 — Type-Safe Union)

```cpp
#include <variant>

std::variant<int, float, std::string> v;
v = 42;
v = "hello";

// Safe access:
std::string s = std::get<std::string>(v);  // OK
// std::get<int>(v);                        // throws bad_variant_access

// Check active type:
if (std::holds_alternative<std::string>(v)) {
    std::cout << std::get<std::string>(v);
}

// Visit pattern:
std::visit([](auto&& arg) {
    std::cout << arg << "\n";
}, v);
```

### 7.3 Bitfields

```cpp
struct Flags {
    uint8_t read    : 1;   // 1 bit
    uint8_t write   : 1;   // 1 bit
    uint8_t execute : 1;   // 1 bit
    uint8_t         : 5;   // 5 bits padding
};

static_assert(sizeof(Flags) == 1);

Flags f{};
f.read = 1;
f.write = 1;

// Common in: network protocols, hardware registers, file formats
// WARNING: bit layout is implementation-defined (not portable for serialization)
```

---

## Chapter 8: `typedef` and `using` Aliases

```cpp
// C-style (still works):
typedef unsigned long ulong;
typedef void (*FuncPtr)(int, int);

// Modern C++ (preferred):
using ulong = unsigned long;
using FuncPtr = void(*)(int, int);
using StringVec = std::vector<std::string>;

// Template alias (only possible with using, not typedef):
template<typename T>
using Vec = std::vector<T, CustomAllocator<T>>;

Vec<int> v;  // std::vector<int, CustomAllocator<int>>
```

---

## Chapter 9: `volatile` Keyword

```cpp
// volatile tells the compiler: this value may change outside the program's control.
// DO NOT optimize away reads/writes.

volatile int* hardware_register = reinterpret_cast<volatile int*>(0x40021000);
int value = *hardware_register;  // Compiler MUST read from address every time

// Use when:
// - Memory-mapped hardware registers
// - Signal handlers
// - Shared memory with other processes

// DO NOT use for:
// - Thread synchronization (use std::atomic instead!)
// volatile does NOT provide atomicity or memory ordering
```

---

## Chapter 10: `static` Keyword (All Uses)

```cpp
// 1. Static local variable — persists across function calls:
int counter() {
    static int count = 0;
    return ++count;   // 1, 2, 3, ...
}

// 2. Static member variable — shared across all instances:
class Widget {
    static int count_;  // Declared in class
public:
    Widget() { ++count_; }
    static int getCount() { return count_; }
};
int Widget::count_ = 0;  // Defined outside class (once)

// 3. Static member function — no 'this' pointer:
Widget::getCount();  // Call without object

// 4. Static free function/variable — internal linkage (translation unit scope):
static int fileLocal = 42;  // Only visible in this .cpp file
static void helper() {}     // Only visible in this .cpp file

// 5. Static in anonymous namespace (modern alternative to #4):
namespace {
    int fileLocal = 42;   // Preferred over static
}
```

---

## Chapter 11: Assert and Debugging

```cpp
#include <cassert>

// Runtime assert (disabled in Release with NDEBUG):
assert(ptr != nullptr);
assert(index >= 0 && index < size);

// Static assert (compile-time):
static_assert(sizeof(int) == 4, "int must be 4 bytes");
static_assert(std::is_trivially_copyable_v<MyStruct>);

// Custom assert:
#define MY_ASSERT(cond, msg) \
    if (!(cond)) { \
        std::cerr << "ASSERT FAILED: " << #cond << "\n" \
                  << "  File: " << __FILE__ << ":" << __LINE__ << "\n" \
                  << "  Message: " << msg << "\n"; \
        std::abort(); \
    }
```

---

## Chapter 12: Interview Questions

**Q1: What's the difference between `static_cast` and `dynamic_cast`?** — `static_cast` is checked at compile time and can be unsafe for downcasts. `dynamic_cast` is checked at runtime (requires virtual functions) and returns nullptr/throws on failure.

**Q2: What are the exception safety guarantees?** — No-throw (noexcept, never fails), Strong (rollback on failure), Basic (valid state, no leaks), No guarantee (avoid).

**Q3: Explain all uses of `static`.** — Local variable persistence, class-shared member, member function without `this`, internal linkage for free functions/variables.

**Q4: What's the difference between `enum` and `enum class`?** — `enum class` is scoped (no name pollution), has no implicit conversion to int, and can specify underlying type. Always prefer `enum class`.

---

## Chapter 13: Practice & Mini Projects

### Project 1: CSV Parser — Use file I/O, string streams, and `std::variant` for typed columns.
### Project 2: Logging Framework — Use namespaces, enums (log levels), file I/O, and custom exceptions.
### Project 3: Config File Reader — Parse key-value config files with error handling and `std::optional`/`std::expected`.

---

## Chapter 14: Mastery Checklist

- [ ] You can write try/catch blocks with custom exception hierarchies
- [ ] You understand exception safety guarantees (no-throw, strong, basic)
- [ ] You can perform text and binary file I/O
- [ ] You use namespaces properly (including anonymous, inline)
- [ ] You understand all preprocessor directives and when to avoid macros
- [ ] You know all four C++ casts and when each is appropriate
- [ ] You prefer `enum class` over `enum`
- [ ] You understand `volatile`, `static`, and `using` aliases
- [ ] You know the difference between `std::variant` and `union`

---

*End of Book 21: Essential Topics*
