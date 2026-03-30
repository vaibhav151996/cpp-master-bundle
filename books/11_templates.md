# Book 11: Templates

## The Complete Guide to Generic Programming in C++

---

### **Target Level:** Upper Intermediate → Advanced
### **Prerequisites:** Books 1–10
### **Learning Outcomes:**
By the end of this book, you will:
- Master function and class templates
- Understand template specialization (full and partial)
- Know template argument deduction and SFINAE
- Understand variadic templates and fold expressions
- Know concepts (C++20) for constraining templates
- Understand template metaprogramming foundations

---

## Chapter 1: Function Templates

### 1.1 The Problem: Code Duplication

```cpp
int maxInt(int a, int b) { return (a > b) ? a : b; }
double maxDouble(double a, double b) { return (a > b) ? a : b; }
// Same logic, different types!
```

### 1.2 The Solution: Function Templates

```cpp
template<typename T>
T maxValue(T a, T b) {
    return (a > b) ? a : b;
}

maxValue(3, 5);         // T = int, returns 5
maxValue(3.14, 2.71);   // T = double, returns 3.14
maxValue('a', 'z');     // T = char, returns 'z'

// Explicit template argument:
maxValue<double>(3, 5.5);  // Force T = double
```

### 1.3 How Templates Work

Templates are **not functions** — they are **blueprints**. The compiler generates actual code (instantiates) when you use the template with specific types:

```
Template:           template<typename T> T max(T a, T b)
Instantiation 1:    int max<int>(int a, int b)       ← Generated code
Instantiation 2:    double max<double>(double a, double b)  ← Generated code
```

Each instantiation is a **separate function** in the compiled binary. This is "zero-cost" — the generated code is identical to what you'd write by hand.

### 1.4 Multiple Template Parameters

```cpp
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

add(1, 2.5);      // T=int, U=double, returns double(3.5)

// C++14 simplification (return type deduction):
template<typename T, typename U>
auto add(T a, U b) {
    return a + b;
}
```

### 1.5 Non-Type Template Parameters

```cpp
template<typename T, int N>
class FixedArray {
    T data_[N];
public:
    T& operator[](int i) { return data_[i]; }
    constexpr int size() const { return N; }
};

FixedArray<int, 10> arr;   // Array of 10 ints (N is a compile-time constant)
FixedArray<double, 5> darr; // Array of 5 doubles
```

---

## Chapter 2: Class Templates

```cpp
template<typename T>
class Stack {
    std::vector<T> data_;
    
public:
    void push(const T& value) { data_.push_back(value); }
    
    T pop() {
        T top = data_.back();
        data_.pop_back();
        return top;
    }
    
    const T& top() const { return data_.back(); }
    bool empty() const { return data_.empty(); }
    size_t size() const { return data_.size(); }
};

Stack<int> intStack;
intStack.push(1);
intStack.push(2);
std::cout << intStack.pop();  // 2

Stack<std::string> strStack;
strStack.push("Hello");
strStack.push("World");
```

### 2.1 Class Template with Multiple Parameters

```cpp
template<typename Key, typename Value>
class Pair {
    Key key_;
    Value value_;
public:
    Pair(const Key& k, const Value& v) : key_(k), value_(v) {}
    Key& key() { return key_; }
    Value& value() { return value_; }
};

Pair<std::string, int> p("age", 25);
```

---

## Chapter 3: Template Specialization

### 3.1 Full Specialization

```cpp
// Primary template:
template<typename T>
class Serializer {
public:
    static std::string serialize(const T& val) {
        return std::to_string(val);
    }
};

// Full specialization for std::string:
template<>
class Serializer<std::string> {
public:
    static std::string serialize(const std::string& val) {
        return "\"" + val + "\"";  // Wrap in quotes
    }
};

// Full specialization for bool:
template<>
class Serializer<bool> {
public:
    static std::string serialize(const bool& val) {
        return val ? "true" : "false";
    }
};

Serializer<int>::serialize(42);            // "42"
Serializer<std::string>::serialize("Hi");  // "\"Hi\""
Serializer<bool>::serialize(true);         // "true"
```

### 3.2 Partial Specialization

```cpp
// Primary template:
template<typename T, typename U>
class Pair { /* general implementation */ };

// Partial specialization when both types are the same:
template<typename T>
class Pair<T, T> { /* specialized for same-type pairs */ };

// Partial specialization for pointer types:
template<typename T>
class Pair<T*, T*> { /* specialized for pointer pairs */ };
```

---

## Chapter 4: Variadic Templates (C++11)

### 4.1 Parameter Packs

```cpp
template<typename... Args>
void print(Args... args) {
    // args is a parameter pack containing 0 or more arguments
    (std::cout << ... << args) << "\n";  // C++17 fold expression
}

print(1, " ", 2.5, " ", "hello");  // "1 2.5 hello"

// Recursive unpacking (C++11):
template<typename T>
void print(T arg) {
    std::cout << arg << "\n";
}

template<typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << " ";
    print(rest...);
}
```

### 4.2 Fold Expressions (C++17)

```cpp
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);    // Unary right fold
}

sum(1, 2, 3, 4, 5);  // 15

// All four fold variants:
// (args op ...)      — unary right fold
// (... op args)      — unary left fold
// (args op ... op init) — binary right fold
// (init op ... op args) — binary left fold

template<typename... Args>
bool allTrue(Args... args) {
    return (args && ...);   // All must be true
}

template<typename... Args>
bool anyTrue(Args... args) {
    return (args || ...);   // At least one true
}
```

---

## Chapter 5: Concepts (C++20)

### 5.1 The Problem with Unconstrained Templates

```cpp
template<typename T>
T max(T a, T b) { return (a > b) ? a : b; }

// Does not compile for types without operator>:
// max(std::vector<int>{}, std::vector<int>{});
// Error message: pages of incomprehensible template errors
```

### 5.2 Concepts to the Rescue

```cpp
#include <concepts>

// Using a standard concept:
template<std::totally_ordered T>
T max(T a, T b) { return (a > b) ? a : b; }

// Custom concept:
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<Numeric T>
T square(T x) { return x * x; }

square(5);      // OK
square(3.14);   // OK
// square("hello"); // ERROR: "hello" does not satisfy Numeric

// Concept with requires clause:
template<typename T>
concept Printable = requires(T t) {
    { std::cout << t } -> std::same_as<std::ostream&>;
};

template<Printable T>
void display(const T& value) {
    std::cout << value << "\n";
}

// Abbreviated function template (C++20):
void display(const Printable auto& value) {
    std::cout << value << "\n";
}
```

### 5.3 Standard Concepts

```cpp
#include <concepts>

std::integral<T>          // int, char, bool, etc.
std::floating_point<T>    // float, double, long double
std::signed_integral<T>   // signed int, etc.
std::unsigned_integral<T> // unsigned int, etc.
std::same_as<T, U>        // T and U are the same type
std::derived_from<D, B>   // D derives from B
std::convertible_to<F, T> // F is convertible to T
std::totally_ordered<T>   // T supports <, >, <=, >=, ==, !=
std::copyable<T>          // T is copy constructible and assignable
std::movable<T>           // T is move constructible and assignable
std::invocable<F, Args...>  // F can be called with Args
```

---

## Chapter 6: SFINAE and `enable_if`

### 6.1 Substitution Failure Is Not An Error

```cpp
// Pre-C++20 way to constrain templates:
#include <type_traits>

template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
doubleValue(T x) {
    return x * 2;
}

template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
doubleValue(T x) {
    return x * 2.0;
}

// C++17 with if constexpr (simpler):
template<typename T>
T doubleValue(T x) {
    if constexpr (std::is_integral_v<T>) {
        return x * 2;
    } else {
        return x * 2.0;
    }
}
```

---

## Chapter 7: Template Metaprogramming

### 7.1 Compile-Time Computation

```cpp
// Factorial at compile time:
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

constexpr int f10 = Factorial<10>::value;  // 3628800 — computed at compile time!

// Modern alternative with constexpr:
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
```

### 7.2 Type Traits

```cpp
#include <type_traits>

static_assert(std::is_integral_v<int>);           // true
static_assert(std::is_floating_point_v<double>);   // true
static_assert(std::is_pointer_v<int*>);            // true
static_assert(std::is_same_v<int, int>);           // true
static_assert(std::is_base_of_v<Base, Derived>);  // true

// Remove qualifiers:
using T = std::remove_const_t<const int>;          // int
using U = std::remove_reference_t<int&>;           // int
using V = std::decay_t<const int&>;                // int
```

---

## Chapter 8: Interview Questions

**Q1: What are templates?** — Blueprints for generating type-specific code at compile time. They enable generic programming without sacrificing performance.

**Q2: What is template specialization?** — Providing a custom implementation for specific types. Full specialization handles one type; partial specialization handles a category.

**Q3: What are concepts?** — C++20 feature to constrain template parameters, providing clearer error messages and documenting type requirements.

**Q4: What is SFINAE?** — "Substitution Failure Is Not An Error." If template argument substitution fails for an overload, that overload is silently discarded rather than causing a compilation error.

**Q5: What is the difference between `class` and `typename` in template parameters?** — No difference. Both are interchangeable. Convention: use `typename` for types, `class` when the parameter is expected to be a class.

---

## Chapter 9: Practice & Projects

### Project 1: Generic Container Library
Implement Stack, Queue, and PriorityQueue as class templates with complete interfaces.

### Project 2: Compile-Time Math Library
Use template metaprogramming: Fibonacci, power, GCD, prime check — all at compile time.

### Project 3: Type-Safe Units Library
Create a dimensions-aware unit system using templates: `Meter<3> + Meter<3>` works, `Meter<3> + Second<5>` is a compile error.

---

## Chapter 10: Summary & Mastery Checklist

- [ ] You can write function and class templates
- [ ] You understand template instantiation and code generation
- [ ] You can write template specializations (full and partial)
- [ ] You understand variadic templates and fold expressions
- [ ] You can use C++20 concepts to constrain templates
- [ ] You know SFINAE and `std::enable_if`
- [ ] You understand basic template metaprogramming
- [ ] You know common type traits from `<type_traits>`

**Proceed to Book 12: Standard Template Library (STL).**

---

*End of Book 11: Templates*
