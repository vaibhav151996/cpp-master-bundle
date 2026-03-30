# Book 13: Modern C++ (C++11/14/17/20/23)

## The Complete Guide to Modern C++ Evolution

---

### **Target Level:** Intermediate → Advanced
### **Prerequisites:** Books 1–12
### **Learning Outcomes:**
By the end of this book, you will:
- Understand every major feature introduced from C++11 through C++23
- Write idiomatic, modern C++ code
- Master move semantics, value categories, and perfect forwarding
- Use `constexpr`, `consteval`, `constinit` for compile-time programming
- Understand structured bindings, `std::optional`, `std::variant`, `std::any`

---

## Chapter 1: C++11 — The Revolution

### 1.1 `auto` — Type Deduction

```cpp
auto x = 42;                    // int
auto y = 3.14;                  // double
auto s = std::string("hello");  // std::string
auto v = std::vector{1, 2, 3};  // std::vector<int> (CTAD, C++17)

// With references and const:
const auto& ref = x;            // const int&
auto&& uref = 42;               // int&& (universal reference with auto)

// Deduction rules:
auto a = {1, 2, 3};    // std::initializer_list<int>
// auto b{1, 2, 3};    // Error in C++17 (single-element only)
auto c{42};             // int (C++17)
```

### 1.2 Range-Based For Loops

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

for (int x : v) { }            // Copy
for (int& x : v) { x *= 2; }  // Modify
for (const auto& x : v) { }   // Read-only (preferred)
for (auto&& x : v) { }        // Universal reference
```

### 1.3 Lambda Expressions

```cpp
// Basic lambda:
auto add = [](int a, int b) { return a + b; };
add(3, 4);  // 7

// Capture modes:
int x = 10;
auto f1 = [x]() { return x; };          // Capture by value
auto f2 = [&x]() { x++; };              // Capture by reference
auto f3 = [=]() { return x; };          // All by value
auto f4 = [&]() { x++; };              // All by reference
auto f5 = [x]() mutable { x++; };      // Modify captured value copy
auto f6 = [&x, y = x * 2]() { };       // Mix + init capture (C++14)

// Generic lambda (C++14):
auto generic = [](auto a, auto b) { return a + b; };

// Lambda with explicit return type:
auto divide = [](double a, double b) -> double { return a / b; };

// Immediately-invoked lambda:
int result = [](int x) { return x * x; }(5);  // 25

// C++20: Template lambda:
auto tmpl = []<typename T>(std::vector<T>& v) { return v.size(); };

// C++23: recursive lambda via deducing this:
auto fib = [](this auto&& self, int n) -> int {
    return n <= 1 ? n : self(n - 1) + self(n - 2);
};
```

### 1.4 Move Semantics and Rvalue References

The single most important C++11 feature for performance:

```cpp
// Value categories:
// lvalue  — has identity, can't be moved from (named variable)
// prvalue — has no identity, can be moved from (temporary)
// xvalue  — has identity, can be moved from (std::move result)

// Rvalue reference: binds to temporaries
void process(std::string&& s) {
    // s is an rvalue reference — we can steal its resources
    std::string local = std::move(s);  // Move, not copy
}

process(std::string("hello"));   // OK: temporary binds to &&
process("hello");                // OK: implicit conversion creates temporary

std::string a = "hello";
// process(a);                   // ERROR: can't bind lvalue to &&
process(std::move(a));           // OK: cast to rvalue
// a is now in a valid-but-unspecified state
```

### 1.5 Move Constructor and Move Assignment

```cpp
class Buffer {
    int* data_;
    size_t size_;
public:
    // Move constructor — steal resources:
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // Move assignment — steal and clean up:
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }
};

// std::move doesn't move — it CASTS to rvalue reference
// The actual moving happens in the move constructor/assignment
```

### 1.6 Perfect Forwarding

```cpp
template<typename T>
void wrapper(T&& arg) {   // Universal/forwarding reference
    // std::forward preserves the value category:
    target(std::forward<T>(arg));
}

// If called with lvalue: T = int&, arg = int&     → forwards as lvalue
// If called with rvalue: T = int,  arg = int&&    → forwards as rvalue
```

### 1.7 `nullptr`

```cpp
// Old C++: NULL is 0 (an integer!)
// New C++: nullptr is a null pointer literal of type std::nullptr_t
void foo(int);
void foo(int*);

foo(NULL);     // Ambiguous! Could call foo(int) or foo(int*)
foo(nullptr);  // Always calls foo(int*)
```

### 1.8 `enum class` (Scoped Enumerations)

```cpp
enum class Color { Red, Green, Blue };
enum class Size : uint8_t { Small = 0, Medium, Large };

Color c = Color::Red;        // Must qualify
// int x = Color::Red;       // ERROR: no implicit conversion
int x = static_cast<int>(Color::Red);  // Explicit conversion
```

### 1.9 `constexpr` — Compile-Time Computation

```cpp
constexpr int square(int x) { return x * x; }
constexpr int val = square(5);  // Computed at compile time

// C++14: relaxed constexpr (loops, local variables allowed)
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i)
        result *= i;
    return result;
}

// C++17: constexpr if
template<typename T>
auto toString(T value) {
    if constexpr (std::is_same_v<T, std::string>) {
        return value;
    } else {
        return std::to_string(value);
    }
}
```

### 1.10 Uniform Initialization

```cpp
int x{42};                           // Direct initialization
std::vector<int> v{1, 2, 3};        // Initializer list
std::map<std::string, int> m{{"a", 1}, {"b", 2}};

struct Point { int x, y; };
Point p{10, 20};                     // Aggregate initialization

// Prevents narrowing:
// int x{3.14};  // ERROR: narrowing conversion
```

### 1.11 Variadic Templates

```cpp
// Base case:
void print() {}

// Recursive case:
template<typename T, typename... Args>
void print(T first, Args... rest) {
    std::cout << first << " ";
    print(rest...);
}

print(1, "hello", 3.14, 'a');  // 1 hello 3.14 a
```

### 1.12 `static_assert`

```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
static_assert(std::is_integral_v<int>);  // C++17: no message required
```

### 1.13 `decltype`

```cpp
int x = 5;
decltype(x) y = 10;        // int
decltype(x + 1.0) z = 3.0; // double

// Trailing return type:
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}
```

### 1.14 Smart Pointers (Preview — Full Coverage in Book 14)

```cpp
#include <memory>
auto ptr = std::make_unique<int>(42);      // unique_ptr
auto shared = std::make_shared<int>(42);   // shared_ptr
```

### 1.15 Multithreading (Preview — Full Coverage in Book 15)

```cpp
#include <thread>
std::thread t([]() { std::cout << "Hello from thread\n"; });
t.join();
```

---

## Chapter 2: C++14 — The Refinement

### 2.1 Generic Lambdas
```cpp
auto add = [](auto a, auto b) { return a + b; };
```

### 2.2 Return Type Deduction
```cpp
auto foo() { return 42; }  // Compiler deduces int
```

### 2.3 Variable Templates
```cpp
template<typename T>
constexpr T pi = T(3.14159265358979323846);

double area = pi<double> * r * r;
float areaF = pi<float> * r * r;
```

### 2.4 Binary Literals and Digit Separators
```cpp
int bin = 0b1010'1100;      // Binary literal with separator
int big = 1'000'000;        // One million, readable
```

### 2.5 `std::make_unique`
```cpp
auto p = std::make_unique<int>(42);  // Was missing from C++11
```

---

## Chapter 3: C++17 — The Big Features

### 3.1 Structured Bindings

```cpp
// From pairs:
auto [first, second] = std::make_pair(1, "hello");

// From maps:
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
for (const auto& [key, value] : m) {
    std::cout << key << ": " << value << "\n";
}

// From structs:
struct Point { int x, y; };
auto [x, y] = Point{10, 20};

// From arrays:
int arr[] = {1, 2, 3};
auto [a, b, c] = arr;
```

### 3.2 `if` and `switch` with Initializer

```cpp
if (auto it = m.find("key"); it != m.end()) {
    // use it — scope limited to if/else
}

switch (auto val = compute(); val) {
    case 0: break;
    case 1: break;
}
```

### 3.3 `std::optional`

```cpp
#include <optional>

std::optional<int> findValue(int key) {
    if (key > 0) return key * 10;
    return std::nullopt;
}

auto result = findValue(5);
if (result) {                  // or result.has_value()
    std::cout << *result;      // or result.value()
}

int val = result.value_or(0);  // Default if empty
```

### 3.4 `std::variant` — Type-Safe Union

```cpp
#include <variant>

std::variant<int, double, std::string> v;
v = 42;
v = 3.14;
v = "hello";

// Access:
std::get<std::string>(v);          // "hello" — throws if wrong type
std::get_if<std::string>(&v);     // Pointer or nullptr

// Visit pattern:
std::visit([](auto&& arg) {
    std::cout << arg << "\n";
}, v);

// Type-safe visitor with overloaded lambdas:
template<typename... Ts>
struct overloaded : Ts... { using Ts::operator()...; };

std::visit(overloaded{
    [](int i) { std::cout << "int: " << i; },
    [](double d) { std::cout << "double: " << d; },
    [](const std::string& s) { std::cout << "string: " << s; },
}, v);
```

### 3.5 `std::any`

```cpp
#include <any>

std::any a = 42;
a = std::string("hello");
a = 3.14;

double val = std::any_cast<double>(a);  // Throws bad_any_cast if wrong
```

### 3.6 `std::string_view`

```cpp
#include <string_view>

void process(std::string_view sv) {
    // Non-owning reference to a string — no allocation
    std::cout << sv.substr(0, 5);
}

process("hello world");                    // No allocation
process(std::string("hello world"));       // No allocation
```

### 3.7 `std::filesystem`

```cpp
#include <filesystem>
namespace fs = std::filesystem;

fs::path p = "/home/user/file.txt";
p.filename();    // "file.txt"
p.extension();   // ".txt"
p.parent_path(); // "/home/user"

fs::exists(p);
fs::file_size(p);
fs::create_directory("dir");
fs::copy("src", "dst");
fs::remove_all("dir");

for (const auto& entry : fs::directory_iterator(".")) {
    std::cout << entry.path() << "\n";
}

// Recursive:
for (const auto& entry : fs::recursive_directory_iterator(".")) {
    if (entry.is_regular_file()) {
        std::cout << entry.path() << " (" << entry.file_size() << " bytes)\n";
    }
}
```

### 3.8 `if constexpr`

```cpp
template<typename T>
std::string stringify(T value) {
    if constexpr (std::is_arithmetic_v<T>) {
        return std::to_string(value);
    } else if constexpr (std::is_same_v<T, std::string>) {
        return value;
    } else {
        return "unknown";
    }
}
```

### 3.9 Fold Expressions

```cpp
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);        // Unary right fold
}

template<typename... Args>
void printAll(Args... args) {
    ((std::cout << args << " "), ...);  // Comma fold
}
```

### 3.10 Class Template Argument Deduction (CTAD)

```cpp
std::pair p(1, 2.0);          // std::pair<int, double>
std::vector v = {1, 2, 3};    // std::vector<int>
std::optional o(42);           // std::optional<int>
```

### 3.11 Nested Namespaces

```cpp
namespace A::B::C {   // Instead of namespace A { namespace B { namespace C {
    void foo() {}
}
```

### 3.12 `[[nodiscard]]`, `[[maybe_unused]]`, `[[fallthrough]]`

```cpp
[[nodiscard]] int compute() { return 42; }
// compute();  // Warning: ignoring return value

[[maybe_unused]] int debug_var = 42;  // No unused warning

switch (x) {
    case 1: do_something();
            [[fallthrough]];
    case 2: do_more(); break;
}
```

---

## Chapter 4: C++20 — The Major Paradigm Shift

### 4.1 Concepts

```cpp
#include <concepts>

template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<Numeric T>
T add(T a, T b) { return a + b; }

// Or with requires:
template<typename T>
    requires Numeric<T>
T multiply(T a, T b) { return a * b; }

// Or shorthand:
auto divide(Numeric auto a, Numeric auto b) { return a / b; }

// Custom concept:
template<typename T>
concept Printable = requires(T t) {
    { std::cout << t } -> std::same_as<std::ostream&>;
};
```

### 4.2 Ranges and Views (Covered in Book 12)

### 4.3 Coroutines

```cpp
#include <coroutine>

// Generator pattern (simplified):
struct Generator {
    struct promise_type {
        int current_value;

        Generator get_return_object() {
            return Generator{
                std::coroutine_handle<promise_type>::from_promise(*this)
            };
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;

    bool next() {
        handle.resume();
        return !handle.done();
    }

    int value() { return handle.promise().current_value; }
};

Generator fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto temp = a;
        a = b;
        b = temp + b;
    }
}
```

### 4.4 Modules

```cpp
// math.ixx (module interface unit)
export module math;

export int add(int a, int b) { return a + b; }
export int multiply(int a, int b) { return a * b; }

// main.cpp
import math;

int main() {
    std::cout << add(3, 4);  // 7
}
```

### 4.5 Three-Way Comparison (Spaceship Operator)

```cpp
#include <compare>

struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;
    // Generates: ==, !=, <, >, <=, >= automatically
};

Point a{1, 2}, b{3, 4};
if (a < b) { }    // Works!
if (a == b) { }   // Works!
```

### 4.6 `consteval` — Guaranteed Compile-Time

```cpp
consteval int immediate_square(int x) { return x * x; }
// consteval functions MUST be evaluated at compile time
// Unlike constexpr which CAN be evaluated at runtime

constexpr int val = immediate_square(5);  // OK
// int x = 5; immediate_square(x);        // ERROR: not compile-time
```

### 4.7 `constinit` — Compile-Time Initialization

```cpp
constinit int global = 42;  // Must be initialized at compile time
                              // But CAN be modified at runtime
// Prevents static initialization order fiasco
```

### 4.8 `std::format` — Python-like String Formatting

```cpp
#include <format>

auto s = std::format("Hello, {}! You are {} years old.", "Alice", 30);
auto f = std::format("{:.2f}", 3.14159);  // "3.14"
auto w = std::format("{:>10}", "right");  // "     right"
auto b = std::format("{:#b}", 42);        // "0b101010"
```

### 4.9 `std::span` — Non-Owning View of Contiguous Data

```cpp
#include <span>

void process(std::span<int> data) {
    for (int x : data) {
        std::cout << x << " ";
    }
}

int arr[] = {1, 2, 3, 4, 5};
std::vector<int> v = {1, 2, 3};

process(arr);   // Works with arrays
process(v);     // Works with vectors
```

---

## Chapter 5: C++23 — The Latest

### 5.1 `std::expected` — Error Handling Without Exceptions
```cpp
#include <expected>

std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) return std::unexpected("division by zero");
    return a / b;
}

auto result = divide(10, 0);
if (result) {
    std::cout << *result;
} else {
    std::cout << result.error();
}
```

### 5.2 `std::print` / `std::println`
```cpp
#include <print>
std::println("Hello, {}!", "World");
std::print("No newline: {}", 42);
```

### 5.3 `std::flat_map` / `std::flat_set`
Sorted containers backed by contiguous storage (faster iteration than `std::map`).

### 5.4 Deducing `this`
```cpp
struct Widget {
    template<typename Self>
    auto&& getName(this Self&& self) {
        return std::forward<Self>(self).name_;
    }
};
```

### 5.5 `std::generator`
Standard coroutine generator replacing custom implementations.

---

## Chapter 6: Interview Questions

**Q1: What is the difference between `std::move` and `std::forward`?** — `std::move` unconditionally casts to rvalue reference (use when you want to transfer ownership). `std::forward` conditionally forwards as lvalue or rvalue based on the original value category (use in template forwarding references).

**Q2: What are value categories in C++?** — lvalue (has identity, not movable), prvalue (no identity, movable), xvalue (has identity, movable: the result of `std::move`). Together: glvalue (has identity) = lvalue + xvalue; rvalue (movable) = prvalue + xvalue.

**Q3: What is the difference between `constexpr`, `consteval`, and `constinit`?** — `constexpr`: function CAN be evaluated at compile time; `consteval`: function MUST be evaluated at compile time; `constinit`: variable MUST be initialized at compile time but can be modified at runtime.

**Q4: When would you use `std::optional` vs `std::variant` vs `std::any`?** — `optional`: may or may not have a value of ONE type. `variant`: always has a value of ONE of several predetermined types (type-safe union). `any`: holds a value of ANY type (type-erased, runtime overhead).

---

## Chapter 7: Practice & Mini Projects

### Project 1: Configuration Parser — Use `std::optional`, `std::variant`, `std::filesystem` to parse a config file.
### Project 2: Expression Evaluator — Use `std::variant` and `std::visit` to evaluate mathematical expression trees.
### Project 3: Generic Serializer — Use C++20 concepts and `if constexpr` to serialize any type to JSON.

---

## Chapter 8: Mastery Checklist

- [ ] You can write idiomatic modern C++ using move semantics
- [ ] You understand value categories (lvalue, prvalue, xvalue)
- [ ] You can use `std::optional`, `std::variant`, `std::any` correctly
- [ ] You understand `constexpr`, `consteval`, `constinit` differences
- [ ] You can write concepts and constrained templates
- [ ] You know structured bindings, `if constexpr`, fold expressions
- [ ] You can use `std::format`, `std::filesystem`, `std::span`

---

*End of Book 13: Modern C++*
