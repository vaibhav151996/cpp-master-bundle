# Book 5: Functions

## The Complete Guide to Functions, Overloading, Recursion, and Callable Objects in C++

---

### **Target Level:** Beginner → Intermediate
### **Prerequisites:** Books 1–4
### **Learning Outcomes:**
By the end of this book, you will:
- Master function declaration, definition, and calling conventions
- Understand parameter passing (by value, by reference, by pointer, by const reference)
- Master function overloading and name mangling
- Understand recursion deeply (stack frames, base cases, optimization)
- Know inline functions, constexpr functions, and their compiler implications
- Understand function pointers, lambdas, and `std::function`
- Master default arguments, variadic functions, and trailing return types

---

## Chapter 1: Conceptual Foundations

### 1.1 Why Functions Exist

Without functions, programs would be monolithic blocks of code — unreadable, unmaintainable, and full of duplication. Functions provide:

1. **Abstraction** — Hide implementation details behind a meaningful name
2. **Reusability** — Write once, call many times
3. **Modularity** — Break complex problems into smaller, testable pieces
4. **Readability** — `calculateTax(income)` is self-documenting
5. **Testing** — Individual functions can be unit tested

### 1.2 How Functions Work at the Machine Level

When a function is called, the CPU:

1. **Pushes** the return address onto the stack (where to resume after the function)
2. **Pushes** the function arguments onto the stack (or passes them in registers)
3. **Jumps** to the function's code
4. **Allocates** a stack frame for local variables
5. **Executes** the function body
6. **Places** the return value in a register (usually `eax`/`rax` for integers, `xmm0` for doubles)
7. **Pops** the stack frame
8. **Jumps back** to the return address

```
Stack during execution of func(10, 20):

┌──────────────────────┐  ← High memory
│  ... main's locals   │
├──────────────────────┤
│  Return address      │  ← Where to resume in main()
├──────────────────────┤
│  Argument: 20        │  ← Second argument
│  Argument: 10        │  ← First argument
├──────────────────────┤
│  Saved base pointer  │  ← Old RBP
├──────────────────────┤
│  func's local vars   │  ← Stack frame for func()
│  ...                 │
└──────────────────────┘  ← RSP (stack pointer) points here
```

**Note:** Modern calling conventions (x86-64 System V, Windows x64) pass the first 4-6 arguments in registers, not on the stack. This is faster.

---

## Chapter 2: Function Declaration and Definition

### 2.1 Function Declaration (Prototype)

A **declaration** tells the compiler: "This function exists, here's its interface."

```cpp
// Declaration (prototype) — usually in header files
int add(int a, int b);          // Parameter names are optional in declarations
int add(int, int);              // Same declaration, names omitted
double calculateArea(double radius);
void printMessage(const std::string& msg);
```

### 2.2 Function Definition

A **definition** provides the actual implementation.

```cpp
// Definition — usually in source (.cpp) files
int add(int a, int b) {
    return a + b;
}

double calculateArea(double radius) {
    return 3.14159265358979 * radius * radius;
}

void printMessage(const std::string& msg) {
    std::cout << msg << std::endl;
}
```

### 2.3 Declaration vs Definition — Why Separate?

```cpp
// calculator.h — DECLARATIONS
#ifndef CALCULATOR_H
#define CALCULATOR_H

int add(int a, int b);
int multiply(int a, int b);

#endif

// calculator.cpp — DEFINITIONS
#include "calculator.h"

int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// main.cpp — USAGE
#include "calculator.h"

int main() {
    int result = add(3, 4);    // Compiler knows add() exists from the header
    return 0;
}
```

This separation enables:
- **Separate compilation** — Change `calculator.cpp` without recompiling `main.cpp`
- **Information hiding** — Users see the interface (header), not the implementation
- **Faster compilation** — Only recompile files that changed

---

## Chapter 3: Parameters and Return Values

### 3.1 Pass by Value

The function gets a **copy** of the argument. Changes to the parameter don't affect the original.

```cpp
void doubleValue(int x) {
    x *= 2;     // Modifies the local copy only
}

int main() {
    int num = 5;
    doubleValue(num);
    std::cout << num;  // Still 5! The original is unchanged
    return 0;
}
```

**Memory diagram:**
```
Before call:      main's stack: num = 5
During call:      doubleValue's stack: x = 5 (copy)
After x *= 2:     doubleValue's stack: x = 10
After return:     main's stack: num = 5 (unchanged)
```

**When to use:** For small types (int, double, char, bool, pointers) — copying is cheap.

### 3.2 Pass by Reference

The function gets a **reference** (alias) to the original variable. Changes affect the original.

```cpp
void doubleValue(int& x) {    // Note the &
    x *= 2;     // Modifies the original!
}

int main() {
    int num = 5;
    doubleValue(num);
    std::cout << num;  // 10! The original was modified
    return 0;
}
```

**When to use:**
- When you need to **modify** the argument
- When passing **large objects** (avoid copying)

### 3.3 Pass by Const Reference

The function gets a reference but **cannot modify** the original. Best of both worlds: no copy + no modification.

```cpp
void printString(const std::string& s) {
    std::cout << s;
    // s += "!";  // ERROR! Cannot modify through const reference
}
```

**When to use:** For large objects you want to **read but not modify** (strings, vectors, structs, classes).

### 3.4 Pass by Pointer

The function receives a **pointer** to the original. Similar to reference but nullable. 

```cpp
void doubleValue(int* ptr) {
    if (ptr != nullptr) {    // Always check for null!
        *ptr *= 2;
    }
}

int main() {
    int num = 5;
    doubleValue(&num);       // Pass address of num
    std::cout << num;        // 10
    
    doubleValue(nullptr);    // Safe — function handles null
    return 0;
}
```

**When to use:** When the argument might be **optional** (nullable), or for C-style APIs.

### 3.5 Parameter Passing Guidelines (Modern C++)

| Parameter Type | When to Use | Syntax |
|---------------|-------------|--------|
| `T` (by value) | Small, cheap-to-copy types (int, double, pointers) | `void f(int x)` |
| `const T&` (const ref) | Large objects you only read | `void f(const std::string& s)` |
| `T&` (ref) | Need to modify the argument | `void f(int& x)` |
| `T*` (pointer) | Optional arguments (nullable) | `void f(int* ptr)` |
| `T&&` (rvalue ref) | Sink parameters (taking ownership) | `void f(std::string&& s)` |

### 3.6 Return Values

```cpp
// Return by value (most common):
int add(int a, int b) {
    return a + b;   // Returns a copy (but compiler optimizes with RVO/NRVO)
}

// Return by reference (return a reference to existing data):
int& getElement(std::vector<int>& v, int index) {
    return v[index];  // Returns reference to element in v
}

// DANGER: Never return a reference to a local variable!
int& bad() {
    int x = 42;
    return x;      // UNDEFINED BEHAVIOR! x is destroyed when function returns
}

// Return multiple values (C++17 structured bindings):
std::tuple<int, double, std::string> getInfo() {
    return {42, 3.14, "hello"};
}

auto [id, value, name] = getInfo();  // C++17 structured bindings
```

### 3.7 Return Value Optimization (RVO/NRVO)

```cpp
std::string createMessage() {
    std::string msg = "Hello, World!";  // Local variable
    return msg;     // Looks like it copies, but...
}

std::string result = createMessage();
// The compiler eliminates the copy! It constructs 'result' directly
// in the caller's stack space. This is Called Named Return Value Optimization (NRVO).
// Guaranteed in C++17 for RVO (unnamed temporaries).
```

---

## Chapter 4: Function Overloading

### 4.1 What is Overloading?

Multiple functions with the **same name** but **different parameter lists**:

```cpp
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}

std::string add(const std::string& a, const std::string& b) {
    return a + b;
}

// The compiler picks the right version based on arguments:
add(1, 2);           // Calls int add(int, int)
add(1.5, 2.5);       // Calls double add(double, double)
add(1, 2, 3);        // Calls int add(int, int, int)
add("ab"s, "cd"s);   // Calls string add(string, string)
```

### 4.2 Overloading Rules

Functions can be overloaded if they differ in:
- **Number of parameters**
- **Types of parameters**
- **Order of parameter types**
- **const/volatile qualification** (for member functions)

Functions **cannot** be overloaded based on:
- **Return type alone** — `int foo()` vs `double foo()` is ambiguous
- **Parameter names** — `f(int x)` vs `f(int y)` are the same signature

### 4.3 Name Mangling

The compiler renames overloaded functions to make them unique. This is called **name mangling**:

```cpp
int add(int, int)        →  _Z3addii     (GCC mangling)
double add(double, double) →  _Z3adddd
int add(int, int, int)   →  _Z3addiii
```

This is why you need `extern "C"` when linking with C code:
```cpp
extern "C" {
    int c_function(int x);  // Don't mangle this name — C doesn't do mangling
}
```

### 4.4 Overload Resolution

When the compiler sees `add(1, 2.5)`, it must decide which overload to call. The process:

1. Find all candidate functions named `add`
2. Filter viable candidates (correct number of args, convertible types)
3. Rank conversions:
   - Exact match (best)
   - Promotion (int→long, float→double)
   - Standard conversion (int→double)
   - User-defined conversion (worst)
4. Choose the best match (or report ambiguity)

```cpp
void f(int x);       // Version 1
void f(double x);    // Version 2

f(42);       // Exact match: Version 1
f(3.14);     // Exact match: Version 2
f(3.14f);    // Promotion: float→double, calls Version 2
f('A');      // Promotion: char→int, calls Version 1
f(42L);      // Ambiguous! long→int and long→double are both conversions
```

---

## Chapter 5: Default Arguments

```cpp
void printLine(std::string text, char fill = '-', int width = 40) {
    std::cout << text << std::string(width - text.length(), fill) << "\n";
}

printLine("Hello");              // Uses defaults: fill='-', width=40
printLine("Hello", '=');         // Uses default width=40
printLine("Hello", '*', 60);     // No defaults used
```

**Rules:**
1. Default arguments must be at the **end** of the parameter list
2. Specified in the **declaration** (header), not in the definition
3. Once a parameter has a default, all subsequent parameters must also have defaults

```cpp
// VALID:
void f(int a, int b = 10, int c = 20);

// INVALID:
void f(int a = 5, int b, int c = 20);  // b has no default after a does
```

---

## Chapter 6: Recursion

### 6.1 What is Recursion?

A function that **calls itself**. Every recursive function needs:
1. **Base case** — condition to stop recursion
2. **Recursive case** — the function calls itself with a "smaller" problem

```cpp
int factorial(int n) {
    if (n <= 1) return 1;             // Base case
    return n * factorial(n - 1);      // Recursive case
}
```

### 6.2 How Recursion Works — Stack Frames

```cpp
factorial(4) calls:
    factorial(4) → 4 * factorial(3)
        factorial(3) → 3 * factorial(2)
            factorial(2) → 2 * factorial(1)
                factorial(1) → returns 1      ← Base case
            factorial(2) → returns 2 * 1 = 2
        factorial(3) → returns 3 * 2 = 6
    factorial(4) → returns 4 * 6 = 24
```

**Stack during recursion:**
```
┌─────────────────────┐
│ factorial(1): n=1   │  ← Currently executing (top of stack)
├─────────────────────┤
│ factorial(2): n=2   │  ← Waiting for result
├─────────────────────┤
│ factorial(3): n=3   │  ← Waiting for result
├─────────────────────┤
│ factorial(4): n=4   │  ← Waiting for result
├─────────────────────┤
│ main()              │
└─────────────────────┘
```

Each recursive call adds a stack frame (~dozens of bytes). Too much recursion causes **stack overflow**.

### 6.3 Classic Recursive Problems

**Power function:**
```cpp
double power(double base, int exp) {
    if (exp == 0) return 1.0;
    if (exp < 0) return 1.0 / power(base, -exp);
    return base * power(base, exp - 1);
}
```

**Fibonacci (naive — O(2^n)):**
```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

**Fibonacci (optimized with memoization — O(n)):**
```cpp
int fib(int n, std::unordered_map<int, int>& memo) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
    return memo[n];
}
```

**Tower of Hanoi:**
```cpp
void hanoi(int n, char from, char to, char aux) {
    if (n == 0) return;
    hanoi(n - 1, from, aux, to);
    std::cout << "Move disk " << n << " from " << from << " to " << to << "\n";
    hanoi(n - 1, aux, to, from);
}
// hanoi(3, 'A', 'C', 'B');
```

### 6.4 Tail Recursion

A function is **tail recursive** if the recursive call is the **last operation** before returning:

```cpp
// NOT tail recursive (multiplication happens after recursive call):
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);    // Must multiply AFTER recursion returns
}

// Tail recursive (recursive call IS the last operation):
int factorial_tail(int n, int accumulator = 1) {
    if (n <= 1) return accumulator;
    return factorial_tail(n - 1, n * accumulator);  // Nothing left to do after call
}
```

**Why it matters:** A smart compiler can optimize tail recursion into a loop (tail call optimization/TCO), eliminating stack frame overhead. However, **C++ does NOT guarantee TCO** — it's a compiler optimization that may or may not happen.

### 6.5 Recursion vs Iteration

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| Readability | More natural for tree/graph problems | More natural for linear problems |
| Performance | Overhead per call (stack frames) | Generally faster |
| Memory | Stack space per call (risk overflow) | Constant memory for loop variables |
| Debugging | Harder (deep call stacks) | Easier |
| Use when | Trees, graphs, divide-and-conquer | Sequences, arrays, simple repetition |

**Rule of thumb:** If a problem naturally decomposes into smaller subproblems of the same type (trees, fractals, divide-and-conquer), use recursion. Otherwise, prefer iteration.

---

## Chapter 7: Inline Functions

### 7.1 What is Inlining?

An **inline** function is a suggestion to the compiler to **replace the function call with the function body** at the call site:

```cpp
inline int square(int x) {
    return x * x;
}

int result = square(5);
// Compiler may replace this with:
// int result = 5 * 5;
```

### 7.2 Why Inline?

- **Eliminates function call overhead** (pushing/popping stack, jump)
- **Enables further optimizations** (constant folding, dead code elimination)
- **Very important for small, frequently-called functions**

### 7.3 `inline` Keyword Semantics

In modern C++, `inline` has two meanings:
1. **Hint to inline** — compiler may ignore it (it knows better)
2. **Allows multiple definitions** — the linker merges duplicates (this is the primary meaning now)

```cpp
// In a header file:
inline int add(int a, int b) {
    return a + b;
}
// Without 'inline', including this header in two .cpp files would cause
// a "multiple definition" linker error. With 'inline', it's legal.
```

### 7.4 When NOT to Inline

- Large functions (increases code size → instruction cache misses)
- Recursive functions (can't be fully inlined)
- Functions with complex control flow

Modern compilers (`-O2`) inline functions automatically based on heuristics, regardless of the `inline` keyword.

---

## Chapter 8: constexpr and consteval Functions

### 8.1 `constexpr` Functions (C++11/14/17/20)

A function that CAN be evaluated at compile time:

```cpp
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

constexpr int f5 = factorial(5);  // Computed at compile time: 120
int n = getUserInput();
int fn = factorial(n);            // Computed at runtime (n not known at compile time)
```

### 8.2 `consteval` Functions (C++20)

Must ALWAYS be evaluated at compile time:

```cpp
consteval int compiletimeSquare(int x) {
    return x * x;
}

constexpr int a = compiletimeSquare(5);   // OK: compile time
int x = 5;
int b = compiletimeSquare(x);             // ERROR: x is not a constant expression
```

---

## Chapter 9: Lambda Expressions (C++11)

### 9.1 What is a Lambda?

An **anonymous function** defined inline, capturing variables from enclosing scope:

```cpp
auto greet = []() {
    std::cout << "Hello, World!\n";
};
greet();  // Output: Hello, World!
```

### 9.2 Lambda Syntax

```cpp
[capture](parameters) -> return_type {
    body
}
```

**Examples:**
```cpp
// No capture, no parameters:
auto hello = []() { std::cout << "Hello!\n"; };

// With parameters:
auto add = [](int a, int b) { return a + b; };
int result = add(3, 4);  // 7

// With explicit return type:
auto divide = [](int a, int b) -> double { return static_cast<double>(a) / b; };

// Capture by value:
int x = 10;
auto f = [x]() { return x * 2; };  // Captures a COPY of x
x = 20;
std::cout << f();  // 20 (captured x was 10)

// Capture by reference:
int count = 0;
auto increment = [&count]() { count++; };
increment();
increment();
std::cout << count;  // 2

// Capture all by value:
auto f = [=]() { return x + y; };

// Capture all by reference:
auto f = [&]() { x++; y++; };

// Mixed capture:
auto f = [=, &count]() { count += x + y; };  // x,y by value; count by reference
```

### 9.3 Lambdas with STL Algorithms

```cpp
std::vector<int> nums = {5, 2, 8, 1, 9, 3};

// Sort descending:
std::sort(nums.begin(), nums.end(), [](int a, int b) { return a > b; });

// Count elements > 5:
int count = std::count_if(nums.begin(), nums.end(), [](int n) { return n > 5; });

// Transform:
std::transform(nums.begin(), nums.end(), nums.begin(), [](int n) { return n * 2; });

// Find first even:
auto it = std::find_if(nums.begin(), nums.end(), [](int n) { return n % 2 == 0; });

// Remove elements:
nums.erase(
    std::remove_if(nums.begin(), nums.end(), [](int n) { return n < 3; }),
    nums.end()
);
```

### 9.4 Generic Lambdas (C++14)

```cpp
auto add = [](auto a, auto b) { return a + b; };

add(1, 2);         // int
add(1.5, 2.5);     // double
add("Hello"s, " World"s);  // string
```

### 9.5 Lambda Internals

The compiler transforms a lambda into an anonymous class with `operator()`:

```cpp
// This lambda:
int x = 10;
auto f = [x](int y) { return x + y; };

// Is equivalent to:
class __lambda_1 {
    int x;
public:
    __lambda_1(int x) : x(x) {}
    int operator()(int y) const { return x + y; }
};
auto f = __lambda_1(x);
```

---

## Chapter 10: Function Pointers and `std::function`

### 10.1 Function Pointers

```cpp
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// Declare a function pointer:
int (*operation)(int, int);

operation = add;
std::cout << operation(3, 4);  // 7

operation = multiply;
std::cout << operation(3, 4);  // 12

// Function pointer as parameter (callback pattern):
int apply(int a, int b, int (*func)(int, int)) {
    return func(a, b);
}
apply(3, 4, add);       // 7
apply(3, 4, multiply);  // 12
```

### 10.2 `std::function` (C++11)

A type-erased wrapper for any callable:

```cpp
#include <functional>

std::function<int(int, int)> op;

op = add;                                    // Regular function
op = [](int a, int b) { return a + b; };    // Lambda
op = std::multiplies<int>();                 // Function object

std::cout << op(3, 4);  // Works with anything callable
```

### 10.3 `using` for Function Pointer Types

```cpp
using BinaryOp = int(*)(int, int);
// Or with std::function:
using BinaryOp = std::function<int(int, int)>;

BinaryOp op = add;
```

---

## Chapter 11: Advanced Topics

### 11.1 Variadic Functions (C-style)

```cpp
#include <cstdarg>

double average(int count, ...) {
    va_list args;
    va_start(args, count);
    
    double sum = 0;
    for (int i = 0; i < count; i++) {
        sum += va_arg(args, double);
    }
    
    va_end(args);
    return sum / count;
}

double avg = average(3, 1.0, 2.0, 3.0);  // 2.0
```

**Avoid in modern C++ — use variadic templates or initializer lists instead.**

### 11.2 Variadic Templates (C++11)

```cpp
// Type-safe, modern variadic function:
template<typename... Args>
void print(Args... args) {
    (std::cout << ... << args) << "\n";  // C++17 fold expression
}

print(1, " ", 2.5, " ", "hello");  // Output: 1 2.5 hello
```

### 11.3 Trailing Return Type (C++11)

```cpp
// Traditional:
int add(int a, int b);

// Trailing return type:
auto add(int a, int b) -> int;

// Useful when return type depends on parameters:
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}
```

### 11.4 `[[nodiscard]]` Attribute (C++17)

```cpp
[[nodiscard]] int computeValue() {
    return 42;
}

computeValue();    // WARNING: Return value discarded
int x = computeValue();  // OK
```

### 11.5 Argument-Dependent Lookup (ADL)

```cpp
namespace MyLib {
    struct Widget {};
    void process(Widget w) { /* ... */ }
}

MyLib::Widget w;
process(w);  // Found via ADL! Compiler searches MyLib because w is a MyLib type
```

---

## Chapter 12: Interview & Competitive Programming Section

### 12.1 Common Interview Questions

**Q1: What is the difference between pass by value and pass by reference?**
A: Pass by value creates a copy — changes don't affect the original. Pass by reference passes an alias — changes affect the original. Use `const&` for read-only access to large objects.

**Q2: What is function overloading? Can you overload by return type?**
A: Overloading allows multiple functions with the same name but different parameters. You CANNOT overload by return type alone — the compiler can't decide which to call from context like `f();`.

**Q3: What is the difference between recursion and iteration?**
A: Both repeat code. Recursion uses function calls (stack frames, risk overflow). Iteration uses loops (efficient, constant stack). Recursion is natural for trees/graphs; iteration for linear processing.

**Q4: What happens if a recursive function has no base case?**
A: Infinite recursion → stack overflow → program crash (segfault).

**Q5: What is name mangling?**
A: The compiler renames overloaded functions with encoded type information to create unique linker symbols. Use `extern "C"` to disable mangling for C interoperability.

### 12.2 Coding Challenges

```cpp
// Implement pow(base, exp) without using library:
double myPow(double base, int exp) {
    if (exp == 0) return 1.0;
    if (exp < 0) return 1.0 / myPow(base, -exp);
    if (exp % 2 == 0) {
        double half = myPow(base, exp / 2);
        return half * half;
    }
    return base * myPow(base, exp - 1);
}

// String reversal (recursive):
std::string reverse(const std::string& s) {
    if (s.length() <= 1) return s;
    return reverse(s.substr(1)) + s[0];
}

// Check palindrome (recursive):
bool isPalindrome(const std::string& s, int left, int right) {
    if (left >= right) return true;
    if (s[left] != s[right]) return false;
    return isPalindrome(s, left + 1, right - 1);
}
```

---

## Chapter 13: Practice Section

### Beginner
1. Write functions for all four arithmetic operations. Overload them for int, double, and string (concatenation for +).
2. Write a recursive function to compute the sum of digits of a number.
3. Write a function that returns multiple values using a struct, pair, and tuple.

### Intermediate
4. Implement a function that takes a callback (function pointer or lambda) and applies it to each element of an array.
5. Write a memoized recursive Fibonacci function using a static map.
6. Implement `std::transform` manually using function pointers.

### Advanced
7. Write a variadic template function `print_all` that prints any number of arguments of any type.
8. Implement a simple function composition: `auto h = compose(f, g);` where `h(x) = f(g(x))`.
9. Write a timer function that takes any callable, measures its execution time, and returns both the result and the duration.

---

## Chapter 14: Mini Projects

### Project 1: Scientific Calculator (Beginner)
- Support +, -, *, /, %, power, sqrt, factorial
- Use function overloading for different types
- Add a history feature storing previous calculations

### Project 2: Sorting Algorithm Library (Intermediate)
- Implement bubble sort, selection sort, merge sort, quicksort
- Each as a function taking a comparison callback (lambda)
- Benchmark each with different data sizes and patterns

### Project 3: Event System (Advanced)
- Implement an event emitter using `std::function` and `std::map`
- Support registering callbacks, emitting events, unregistering
- Thread-safe event queue with async dispatch

---

## Chapter 15: Summary & Mastery Checklist

- [ ] You can declare, define, and call functions correctly
- [ ] You understand pass by value, reference, const reference, and pointer
- [ ] You can write and use overloaded functions
- [ ] You understand name mangling and `extern "C"`
- [ ] You can write recursive functions with proper base cases
- [ ] You understand stack frames and tail recursion
- [ ] You can write and use lambda expressions with captures
- [ ] You know when to use `inline`, `constexpr`, and `consteval`
- [ ] You can use function pointers, `std::function`, and callbacks
- [ ] You understand `[[nodiscard]]` and trailing return types

**If all ratings are 4+, you are ready for Book 6: Arrays & Strings.**

---

*End of Book 5: Functions*
*Next: Book 6 — Arrays & Strings*
