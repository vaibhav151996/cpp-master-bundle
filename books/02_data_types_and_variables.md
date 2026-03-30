# Book 2: Data Types & Variables

## The Complete Guide to C++ Type System, Variables, and Memory Representation

---

### **Target Level:** Beginner
### **Prerequisites:** Book 1 — C++ Foundations
### **Learning Outcomes:**
By the end of this book, you will:
- Understand every primitive data type in C++ and its memory representation
- Master type modifiers and their effects on range, size, and behavior
- Understand variables, scope, lifetime, and linkage at a deep level
- Know the difference between `const`, `constexpr`, `#define`, and `consteval`
- Understand literals (integer, floating-point, character, string, boolean)
- Master storage classes and their impact on variable lifetime and visibility
- Understand how the compiler and CPU handle different types

---

## Chapter 1: Conceptual Foundations

### 1.1 Why Do Data Types Exist?

At the hardware level, a computer's memory is just a vast array of bytes. Each byte is 8 bits, and each bit is either 0 or 1. The number `01000001` could mean:
- The integer **65**
- The character **'A'** (ASCII code 65)
- Part of the floating-point number **1.0** (if it's part of a 4-byte float)
- Part of a memory address
- A machine instruction

**Data types tell the compiler how to INTERPRET the bytes in memory.** Without types, the compiler wouldn't know whether the bytes `01000001` represent a number, a character, or something else entirely.

### 1.2 Static vs Dynamic Typing

**C++ uses static typing** — every variable's type is known at compile time:
```cpp
int x = 42;       // Compiler knows x is int at compile time
double y = 3.14;  // Compiler knows y is double at compile time
```

**Python uses dynamic typing** — types are determined at runtime:
```python
x = 42       # x is int... for now
x = "hello"  # now x is a string!
```

Static typing advantages:
- **Faster execution** — the compiler generates optimal machine code for each type
- **Earlier error detection** — type mismatches caught at compile time, not runtime
- **Better optimization** — compiler knows exact memory layout
- **Self-documenting** — types serve as documentation

### 1.3 The C++ Type System Hierarchy

```
C++ Types
├── Fundamental Types
│   ├── Void type (void)
│   ├── Nullptr type (std::nullptr_t)  [C++11]
│   ├── Arithmetic Types
│   │   ├── Integral Types
│   │   │   ├── bool
│   │   │   ├── Character Types (char, wchar_t, char8_t, char16_t, char32_t)
│   │   │   └── Integer Types (short, int, long, long long) [signed/unsigned]
│   │   └── Floating-Point Types (float, double, long double)
│   └── cv-qualified versions of the above
├── Compound Types
│   ├── Reference Types (T&, T&&)
│   ├── Pointer Types (T*)
│   ├── Pointer-to-Member Types
│   ├── Array Types (T[N])
│   ├── Function Types
│   ├── Enumeration Types (enum, enum class)
│   └── Class Types (class, struct, union)
└── User-Defined Types (through typedef, using, class, struct, enum)
```

---

## Chapter 2: Primitive Types (Core Theory)

### 2.1 Integer Types

#### 2.1.1 The `int` Type

The `int` type is the "natural" integer type for the target platform. On virtually all modern systems (32-bit and 64-bit), `int` is 32 bits (4 bytes).

```cpp
int x = 42;
int y = -100;
int z = 0;
```

**Memory representation:** Integers are stored in binary using **two's complement** notation for signed integers.

**How two's complement works (for a 32-bit int):**

The number 42:
```
Binary:  00000000 00000000 00000000 00101010
         = 32 + 8 + 2 = 42
```

The number -42:
```
Step 1: Write 42 in binary:   00000000 00000000 00000000 00101010
Step 2: Flip all bits:        11111111 11111111 11111111 11010101
Step 3: Add 1:                11111111 11111111 11111111 11010110
Result: -42 in two's complement
```

This system has an elegant property: the hardware can use the SAME addition circuit for both positive and negative numbers!

**Range of int (32-bit, signed):**
- Minimum: $-2^{31} = -2{,}147{,}483{,}648$
- Maximum: $2^{31} - 1 = 2{,}147{,}483{,}647$
- Approximately $\pm 2.1$ billion

#### 2.1.2 The `short` Type

A shorter integer (typically 16 bits / 2 bytes).

```cpp
short s = 32000;
```

**Range (16-bit, signed):** $-32{,}768$ to $32{,}767$

**When to use:** Rarely in modern code. Only when memory is extremely constrained (embedded systems) or when matching a specific data format (network protocols, file formats).

#### 2.1.3 The `long` Type

A longer integer. On most 64-bit systems:
- Windows: 32 bits (same as int!)
- Linux/macOS: 64 bits

```cpp
long l = 1000000L;  // The 'L' suffix indicates a long literal
```

**Gotcha:** `long` is NOT guaranteed to be larger than `int`. On Windows 64-bit, both are 32 bits.

#### 2.1.4 The `long long` Type (C++11)

Guaranteed to be at least 64 bits.

```cpp
long long ll = 9'000'000'000'000'000'000LL;  // 9 quintillion
```

**Range (64-bit, signed):** $-2^{63}$ to $2^{63} - 1$
- Approximately $\pm 9.2 \times 10^{18}$

#### 2.1.5 Size Guarantees

The C++ standard only guarantees **minimum** sizes:
```
sizeof(char)      == 1 byte  (exactly, always)
sizeof(short)     >= 2 bytes
sizeof(int)       >= 2 bytes
sizeof(long)      >= 4 bytes
sizeof(long long) >= 8 bytes
```

And the ordering: `sizeof(char) <= sizeof(short) <= sizeof(int) <= sizeof(long) <= sizeof(long long)`

For **exact-width types**, use `<cstdint>`:
```cpp
#include <cstdint>

int8_t   a;  // Exactly 8 bits
int16_t  b;  // Exactly 16 bits
int32_t  c;  // Exactly 32 bits
int64_t  d;  // Exactly 64 bits

uint8_t  e;  // Unsigned 8 bits (0 to 255)
uint16_t f;  // Unsigned 16 bits (0 to 65535)
uint32_t g;  // Unsigned 32 bits (0 to 4,294,967,295)
uint64_t h;  // Unsigned 64 bits (0 to 18,446,744,073,709,551,615)
```

#### 2.1.6 Checking Sizes on Your System
```cpp
#include <iostream>
#include <climits>

int main() {
    std::cout << "sizeof(char):      " << sizeof(char)      << " bytes\n";
    std::cout << "sizeof(short):     " << sizeof(short)     << " bytes\n";
    std::cout << "sizeof(int):       " << sizeof(int)       << " bytes\n";
    std::cout << "sizeof(long):      " << sizeof(long)      << " bytes\n";
    std::cout << "sizeof(long long): " << sizeof(long long) << " bytes\n";
    
    std::cout << "\nint range: " << INT_MIN << " to " << INT_MAX << "\n";
    std::cout << "long range: " << LONG_MIN << " to " << LONG_MAX << "\n";
    
    return 0;
}
```

### 2.2 Floating-Point Types

Computers cannot represent most real numbers exactly. Instead, they use **IEEE 754 floating-point representation**, which is a scientific notation in binary.

#### 2.2.1 The `float` Type (32-bit)

```cpp
float f = 3.14f;   // The 'f' suffix is important!
float g = 3.14;    // Warning: implicit conversion from double to float
```

**IEEE 754 Single Precision layout (32 bits):**
```
[S][EEEEEEEE][MMMMMMMMMMMMMMMMMMMMMMM]
 1    8 bits         23 bits
 │    │              └── Mantissa (fraction)
 │    └── Exponent
 └── Sign (0 = positive, 1 = negative)
```

**Value** = $(-1)^S \times 1.M \times 2^{E - 127}$

- **Precision:** ~7 decimal digits
- **Range:** $\pm 3.4 \times 10^{38}$
- **Smallest positive:** $\approx 1.2 \times 10^{-38}$

#### 2.2.2 The `double` Type (64-bit)

```cpp
double d = 3.14159265358979;  // Default floating-point type
```

**IEEE 754 Double Precision layout (64 bits):**
```
[S][EEEEEEEEEEE][MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM]
 1    11 bits                    52 bits
```

- **Precision:** ~15-16 decimal digits
- **Range:** $\pm 1.8 \times 10^{308}$

**Always prefer `double` over `float`** unless you have a specific reason (GPU programming, memory constraints, SIMD).

#### 2.2.3 The `long double` Type

Extended precision. Size varies by platform:
- x86 Linux/macOS: 80 bits (10 bytes, padded to 12 or 16)
- MSVC Windows: 64 bits (same as double!)
- Some platforms: 128 bits

```cpp
long double ld = 3.14159265358979323846L;
```

#### 2.2.4 Floating-Point Gotchas

**Problem 1: Precision loss**
```cpp
float f = 0.1f;
// f is NOT exactly 0.1 !
// f = 0.100000001490116119384765625 (closest representable value)
```

0.1 in binary is a repeating fraction: $0.0\overline{0011}$ — just like 1/3 = 0.333... in decimal.

**Problem 2: Comparison failures**
```cpp
double a = 0.1 + 0.2;
double b = 0.3;

if (a == b) {
    // This may NOT execute!
    // a = 0.30000000000000004
    // b = 0.29999999999999999
}

// Correct way:
const double EPSILON = 1e-9;
if (std::abs(a - b) < EPSILON) {
    // This works correctly
}
```

**Problem 3: Large and small numbers**
```cpp
float big = 16'777'216.0f;   // 2^24
float result = big + 1.0f;   // result == 16777216.0f (no change!)
// The float doesn't have enough precision to represent 16777217
```

**Problem 4: Special values**
```cpp
double inf = 1.0 / 0.0;       // +infinity
double neg_inf = -1.0 / 0.0;  // -infinity
double nan = 0.0 / 0.0;       // NaN (Not a Number)

// NaN has a strange property:
if (nan == nan) {
    // This is FALSE! NaN is not equal to itself!
}

// Check for NaN:
#include <cmath>
if (std::isnan(nan)) {
    // This is true
}
```

### 2.3 The `char` Type

The `char` type stores a single character. It is exactly 1 byte (8 bits).

```cpp
char c = 'A';          // Character literal (single quotes)
char d = 65;           // Same as 'A' (ASCII code)
char newline = '\n';   // Escape sequence
```

**ASCII Table (important values):**
```
'0' = 48     '9' = 57      (digits: 48-57)
'A' = 65     'Z' = 90      (uppercase: 65-90)
'a' = 97     'z' = 122     (lowercase: 97-122)
' ' = 32                    (space)
'\n' = 10                   (newline)
'\0' = 0                    (null terminator)
```

**Useful relationships:**
```cpp
// Convert digit character to integer:
int digit = '7' - '0';  // = 55 - 48 = 7

// Convert lowercase to uppercase:
char upper = 'a' - 32;  // = 97 - 32 = 65 = 'A'
// Or use standard function:
char upper2 = std::toupper('a');

// Check if character is a digit:
bool isDigit = (c >= '0' && c <= '9');
```

**Signed vs unsigned char:**
```cpp
signed char   sc = -128;  // Range: -128 to 127
unsigned char uc = 255;   // Range: 0 to 255
char          c = 'A';    // Signedness is implementation-defined!
```

**Gotcha:** Whether plain `char` is signed or unsigned varies by compiler/platform. If you need a specific range, use `signed char` or `unsigned char`.

#### Unicode Character Types (C++11/20):
```cpp
char8_t  c8  = u8'A';   // C++20: UTF-8 character
char16_t c16 = u'好';    // UTF-16 character
char32_t c32 = U'🎉';   // UTF-32 character (can hold any Unicode code point)
wchar_t  wc  = L'好';    // Wide character (platform-dependent size: 16 or 32 bits)
```

### 2.4 The `bool` Type

```cpp
bool isReady = true;
bool isDone = false;
```

- `true` is stored as `1` (internally)
- `false` is stored as `0` (internally)
- `sizeof(bool)` is typically 1 byte (not 1 bit — CPUs can't address individual bits)

**Implicit conversions to bool:**
```cpp
int x = 42;
if (x) {
    // Executes! Any non-zero integer converts to true
}

int* ptr = nullptr;
if (ptr) {
    // Doesn't execute. nullptr converts to false
}

double d = 0.0;
if (d) {
    // Doesn't execute. 0.0 converts to false
}
```

### 2.5 The `void` Type

`void` means "no type" or "nothing." It's used in three main contexts:

```cpp
// 1. Function that returns nothing
void printHello() {
    std::cout << "Hello!" << std::endl;
    // No return value needed
}

// 2. Function that takes no parameters (optional in C++, required in C)
int getRandomNumber(void) {  // 'void' parameter is optional in C++
    return 42;
}

// 3. void pointer — can point to any type
void* ptr = &someVariable;
// You cannot dereference a void* directly; you must cast it first
```

---

## Chapter 3: Type Modifiers

### 3.1 `signed` and `unsigned`

By default, `int`, `short`, `long`, and `long long` are signed (can hold negative values).

```cpp
unsigned int  ui = 4'294'967'295U;  // 0 to 4,294,967,295
signed int    si = -42;             // -2,147,483,648 to 2,147,483,647
```

**Unsigned arithmetic behavior:**
```cpp
unsigned int a = 0;
a = a - 1;  // a = 4294967295 (wraps around — this is DEFINED behavior for unsigned)

// This is called "modular arithmetic" — unsigned values wrap around like a clock
// (0 - 1) mod 2^32 = 4294967295
```

**Signed overflow is UNDEFINED BEHAVIOR:**
```cpp
int b = INT_MAX;   // 2147483647
b = b + 1;         // UNDEFINED BEHAVIOR! (signed overflow)
// The compiler may assume this never happens and optimize accordingly
```

### 3.2 `short` and `long` Modifiers

```cpp
short int si = 100;         // short (≥16 bits)
long int li = 100000L;      // long (≥32 bits)
long long int lli = 100LL;  // long long (≥64 bits)

// The 'int' keyword is optional with modifiers:
short s = 100;
long l = 100000L;
long long ll = 100LL;
unsigned long ul = 42UL;
```

### 3.3 Summary Table

| Type | Typical Size | Minimum Size | Signed Range | Unsigned Range |
|------|-------------|-------------|--------------|----------------|
| `char` | 1 byte | 1 byte | -128 to 127 | 0 to 255 |
| `short` | 2 bytes | 2 bytes | -32,768 to 32,767 | 0 to 65,535 |
| `int` | 4 bytes | 2 bytes | ~-2.1B to ~2.1B | 0 to ~4.3B |
| `long` | 4 or 8 bytes | 4 bytes | varies | varies |
| `long long` | 8 bytes | 8 bytes | ~-9.2×10^18 to ~9.2×10^18 | 0 to ~1.8×10^19 |
| `float` | 4 bytes | 4 bytes | ±3.4×10^38 | N/A |
| `double` | 8 bytes | 8 bytes | ±1.8×10^308 | N/A |
| `bool` | 1 byte | 1 byte | true/false | N/A |

---

## Chapter 4: Variables and Scope

### 4.1 Variable Declaration and Initialization

**Declaration:** Tells the compiler a variable exists and its type.
**Definition:** Allocates storage for the variable.
**Initialization:** Gives the variable its first value.

```cpp
int x;               // Declaration + Definition (uninitialized — DANGEROUS!)
int y = 42;          // Declaration + Definition + Copy Initialization
int z(42);           // Declaration + Definition + Direct Initialization
int w{42};           // Declaration + Definition + Brace Initialization (C++11, preferred)
int v = {42};        // Declaration + Definition + Copy-list Initialization (C++11)
auto a = 42;         // Type deduced as int (C++11)
```

#### Why Brace Initialization `{}` is Preferred (C++11)

```cpp
int a = 3.14;   // Compiles! Silently narrows 3.14 to 3 (data loss!)
int b(3.14);    // Compiles! Same silent narrowing
int c{3.14};    // ERROR! Narrowing conversion detected by compiler!
int d = {3.14}; // ERROR! Same protection
```

Brace initialization:
- Prevents **narrowing conversions** (catches bugs at compile time)
- Works uniformly for all types (primitives, arrays, structs, classes)
- Is the "uniform initialization" syntax of modern C++

### 4.2 Scope

**Scope** determines where a variable can be accessed. C++ has several types of scope:

#### Block Scope (Local Variables)
```cpp
void function() {
    int x = 10;      // x is visible from here...
    
    {
        int y = 20;  // y is visible from here...
        x = x + y;   // x is still visible
    }                 // ...to here. y is DESTROYED
    
    // y is NOT accessible here — compiler error
    std::cout << x;   // x is still accessible
}                     // x is DESTROYED here
```

#### Function Scope
Only applies to labels (used with `goto` — rarely used in modern C++):
```cpp
void func() {
    goto end;      // Can jump to 'end' anywhere in the function
    int x = 5;
end:               // Label has function scope
    return;
}
```

#### File Scope (Global Variables)
```cpp
int globalVar = 100;  // Visible from this point to end of file
                      // Also accessible from other files via 'extern'

int main() {
    std::cout << globalVar;  // OK
    return 0;
}
```

#### Namespace Scope
```cpp
namespace MyApp {
    int version = 1;     // Accessible as MyApp::version
    
    void init() {
        version = 2;     // Direct access within nameaspace
    }
}

int main() {
    std::cout << MyApp::version;  // Access via namespace prefix
    return 0;
}
```

#### Class Scope
```cpp
class MyClass {
public:
    int publicVar;       // Accessible from anywhere with an object
    void method() {
        int localVar;    // Block scope within method
        publicVar = 10;  // Access class member
    }
private:
    int privateVar;      // Only accessible within this class
};
```

### 4.3 Variable Lifetime

**Lifetime** = how long a variable exists in memory (related to but different from scope).

| Category | Created | Destroyed | Example |
|----------|---------|-----------|---------|
| Automatic (local) | When scope is entered | When scope is exited | `int x = 5;` inside a function |
| Static local | First time control passes through declaration | Program exit | `static int count = 0;` |
| Global | Before `main()` | After `main()` returns | `int g = 42;` at file scope |
| Dynamic | When `new` is called | When `delete` is called | `int* p = new int(42);` |
| Thread-local | Thread creation | Thread exit | `thread_local int t = 0;` |

```cpp
#include <iostream>

void counter() {
    static int count = 0;  // Initialized ONCE, persists across calls
    count++;
    std::cout << "Called " << count << " times\n";
}

int main() {
    counter();  // "Called 1 times"
    counter();  // "Called 2 times"
    counter();  // "Called 3 times"
    return 0;
}
```

### 4.4 Name Shadowing

When an inner scope declares a variable with the same name as an outer scope, the inner variable "shadows" the outer one:

```cpp
int x = 100;  // Global x

int main() {
    int x = 50;  // Local x — shadows global x
    
    {
        int x = 10;  // Inner x — shadows local x
        std::cout << x << "\n";    // Prints 10 (inner)
    }
    
    std::cout << x << "\n";        // Prints 50 (local)
    std::cout << ::x << "\n";      // Prints 100 (global — :: accesses global scope)
    
    return 0;
}
```

**Best Practice:** Avoid shadowing. Use `-Wshadow` compiler flag to get warnings.

---

## Chapter 5: Constants

### 5.1 `const` — Runtime Constant

`const` means "this value cannot be changed after initialization."

```cpp
const int MAX_SIZE = 100;
MAX_SIZE = 200;  // ERROR! Cannot modify a const variable

const double PI = 3.14159265358979;
```

**`const` with pointers (the most confusing C++ topic for beginners):**
```cpp
int x = 10;
int y = 20;

// Read RIGHT to LEFT:
const int* ptr1 = &x;     // Pointer to const int
// "ptr1 is a pointer to an int that is const"
// You can change ptr1 (point to something else)
// You CANNOT change *ptr1 (the value it points to)
ptr1 = &y;    // OK — changing the pointer
*ptr1 = 30;   // ERROR — cannot modify through pointer-to-const

int* const ptr2 = &x;     // Const pointer to int
// "ptr2 is a const pointer to an int"
// You CANNOT change ptr2 (must always point to x)
// You CAN change *ptr2 (the value at the address)
ptr2 = &y;    // ERROR — cannot change const pointer
*ptr2 = 30;   // OK — modifying the value

const int* const ptr3 = &x;  // Const pointer to const int
// Can't change ptr3, can't change *ptr3
```

**The "clockwise/spiral rule"** or simply: **read the declaration from right to left**, replacing `*` with "pointer to" and `const` with "constant."

### 5.2 `constexpr` — Compile-Time Constant (C++11)

`constexpr` guarantees that the value is computed at **compile time**, not runtime.

```cpp
constexpr int ARRAY_SIZE = 100;  // Must be known at compile time
int arr[ARRAY_SIZE];              // OK — compiler knows the size

constexpr double PI = 3.14159265358979;

constexpr int square(int x) {
    return x * x;
}
constexpr int result = square(5);  // Computed at compile time — result is 25
```

**Difference between `const` and `constexpr`:**
```cpp
int getSize() { return 100; }

const int a = getSize();      // OK — a is const but computed at RUNTIME
constexpr int b = getSize();  // ERROR — getSize() is not constexpr

constexpr int c = 42;         // OK — 42 is a compile-time constant
const int d = 42;             // OK — but d MIGHT be a compile-time constant
                               // (compiler may or may not optimize)
```

Use `constexpr` when you NEED compile-time evaluation. Use `const` when you just want immutability.

### 5.3 `consteval` — Immediate Functions (C++20)

```cpp
consteval int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

int x = factorial(5);  // OK — compiler computes 120 at compile time

int n;
std::cin >> n;
int y = factorial(n);  // ERROR! consteval functions MUST be evaluated at compile time
```

### 5.4 `constinit` — Constant Initialization (C++20)

```cpp
constinit int global_val = 42;  // Must be initialized with a constant expression
                                 // Unlike constexpr, the value CAN be changed later

void modify() {
    global_val = 100;  // OK — constinit only controls initialization, not mutability
}
```

### 5.5 `#define` Macros (Legacy — Avoid)

```cpp
#define MAX_SIZE 100
#define PI 3.14159
```

**Why macros are inferior to `const`/`constexpr`:**
1. **No type checking** — `#define PI 3.14` has no type; `constexpr double PI = 3.14;` does
2. **No scope** — macros are global, polluting the entire translation unit
3. **No debugging support** — debugger doesn't know the name "MAX_SIZE"
4. **Text substitution bugs:**
```cpp
#define SQUARE(x) x * x
int result = SQUARE(3 + 1);  // Expands to 3 + 1 * 3 + 1 = 7 (not 16!)

// Fix requires parentheses everywhere:
#define SQUARE(x) ((x) * (x))
```

**Modern guideline:** Use `constexpr` for compile-time constants. Never use `#define` for constants in new C++ code.

### 5.6 `enum` and `enum class`

**Traditional enum (C++03):**
```cpp
enum Color { RED, GREEN, BLUE };        // RED=0, GREEN=1, BLUE=2
enum Direction { NORTH, SOUTH, EAST, WEST };

Color c = RED;
int x = RED;      // Implicit conversion to int — allowed but dangerous
// if (c == NORTH) // This compiles! Comparing Color to Direction — bug!
```

**Scoped enum (C++11 — preferred):**
```cpp
enum class Color { Red, Green, Blue };
enum class Direction { North, South, East, West };

Color c = Color::Red;
// int x = Color::Red;         // ERROR — no implicit conversion
int x = static_cast<int>(Color::Red);  // OK — explicit conversion
// if (c == Direction::North)  // ERROR — can't compare different enum types
```

**Enum with underlying type:**
```cpp
enum class Status : uint8_t {
    Active = 1,
    Inactive = 0,
    Pending = 2
};
// Each value is stored as a uint8_t (1 byte)
```

---

## Chapter 6: Literals

### 6.1 Integer Literals

```cpp
int dec = 42;          // Decimal (base 10)
int oct = 052;         // Octal (base 8) — starts with 0
int hex = 0x2A;        // Hexadecimal (base 16) — starts with 0x
int bin = 0b00101010;  // Binary (base 2) — starts with 0b (C++14)

// With digit separators (C++14):
int big = 1'000'000;
int mask = 0xFF'FF'00'00;
int bits = 0b1010'1010'1010'1010;
```

**Literal suffixes:**
```cpp
auto a = 42;      // int
auto b = 42U;     // unsigned int
auto c = 42L;     // long
auto d = 42UL;    // unsigned long
auto e = 42LL;    // long long
auto f = 42ULL;   // unsigned long long
```

### 6.2 Floating-Point Literals

```cpp
double d1 = 3.14;        // double (default)
double d2 = 3.14e2;      // 3.14 × 10² = 314.0
double d3 = 1.5e-3;      // 1.5 × 10⁻³ = 0.0015

float f1 = 3.14f;        // float (f suffix)
float f2 = 3.14F;        // float (F suffix)

long double ld = 3.14L;  // long double (L suffix)

// Hexadecimal floating-point (C++17):
double hf = 0x1.fp10;    // 1.9375 × 2^10 = 1984.0
```

### 6.3 Character Literals

```cpp
char c1 = 'A';           // Regular character
char c2 = '\n';          // Escape sequence — newline
char c3 = '\t';          // Tab
char c4 = '\\';          // Backslash
char c5 = '\'';          // Single quote
char c6 = '\"';          // Double quote
char c7 = '\0';          // Null character (string terminator)
char c8 = '\x41';        // Hexadecimal: 0x41 = 65 = 'A'
char c9 = '\101';        // Octal: 101₈ = 65₁₀ = 'A'
```

**Complete escape sequence table:**
| Sequence | Meaning |
|----------|---------|
| `\n` | Newline |
| `\t` | Horizontal tab |
| `\r` | Carriage return |
| `\b` | Backspace |
| `\a` | Bell/alert |
| `\f` | Form feed |
| `\v` | Vertical tab |
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |
| `\?` | Question mark |
| `\0` | Null character |
| `\xhh` | Hex value |
| `\ooo` | Octal value |

### 6.4 String Literals

```cpp
const char* s1 = "Hello, World!";     // C-style string (null-terminated char array)
std::string s2 = "Hello, World!";     // C++ string object

// Raw string literals (C++11):
const char* path = R"(C:\Users\student\Documents)";
// Output: C:\Users\student\Documents (no escape processing)

// Raw string with delimiter (for strings containing parentheses):
const char* s3 = R"delim(She said "Hello (World)")delim";
// Output: She said "Hello (World)"

// UTF-8 string (C++11):
const char* u8s = u8"Hello, 世界!";

// Wide string:
const wchar_t* ws = L"Wide string";

// UTF-16 and UTF-32 (C++11):
const char16_t* u16s = u"UTF-16 string";
const char32_t* u32s = U"UTF-32 string";
```

### 6.5 Boolean Literals

```cpp
bool a = true;   // 1
bool b = false;  // 0
```

### 6.6 Pointer Literal

```cpp
int* ptr = nullptr;  // C++11: type-safe null pointer
// int* ptr = NULL;   // Legacy C/C++ — avoid (NULL is just 0)
// int* ptr = 0;      // Also works but misleading — avoid
```

**Why `nullptr` is better than `NULL`:**
```cpp
void func(int x)    { std::cout << "int\n"; }
void func(char* p)  { std::cout << "pointer\n"; }

func(NULL);      // Calls func(int)! NULL is 0, which is an int.
func(nullptr);   // Calls func(char*)! nullptr is a pointer type.
```

### 6.7 User-Defined Literals (C++11)

```cpp
#include <chrono>
using namespace std::chrono_literals;

auto duration = 5s;      // 5 seconds
auto ms_val = 100ms;     // 100 milliseconds
auto minutes = 2min;     // 2 minutes

// Custom user-defined literal:
constexpr long double operator""_km(long double val) {
    return val * 1000.0L;  // Convert km to meters
}

long double distance = 5.5_km;  // 5500.0 meters
```

---

## Chapter 7: Storage Classes

Storage classes control a variable's **lifetime**, **visibility (linkage)**, and **storage location**.

### 7.1 `auto` (C++11 meaning)

In C++11, `auto` became a **type deduction** keyword (it originally meant "automatic storage" in C/C++03, which was the default and never needed to be explicitly stated).

```cpp
auto x = 42;           // int
auto y = 3.14;         // double
auto z = "hello";      // const char*
auto w = std::string("hello");  // std::string
auto v = {1, 2, 3};   // std::initializer_list<int>

// Very useful with complex types:
std::map<std::string, std::vector<int>> myMap;
auto it = myMap.begin();  // Much cleaner than writing the full iterator type
```

### 7.2 `static`

`static` has different meanings depending on context:

**1. Static local variables — persist across function calls:**
```cpp
void counter() {
    static int count = 0;  // Initialized once, lives until program ends
    count++;
    std::cout << count << "\n";
}
// counter() → 1
// counter() → 2
// counter() → 3
```

**2. Static global variables/functions — internal linkage (file-private):**
```cpp
// file1.cpp
static int secret = 42;           // Only visible in file1.cpp
static void helper() { /* ... */ } // Only visible in file1.cpp
```

**3. Static class members — shared across all objects:**
```cpp
class Player {
public:
    static int playerCount;  // Shared by ALL Player objects
    Player() { playerCount++; }
};
int Player::playerCount = 0;  // Must define outside class

// Player::playerCount is shared — there's only ONE copy
```

### 7.3 `extern`

Declares that a variable or function is defined in another translation unit:

```cpp
// file1.cpp
int globalVar = 42;  // Definition

// file2.cpp
extern int globalVar;  // Declaration — "this exists somewhere else"
void func() {
    std::cout << globalVar;  // Uses the variable from file1.cpp
}
```

**How `extern` works at the linker level:**
1. `file2.cpp` compiles successfully — the compiler trusts the `extern` declaration
2. At link time, the linker looks for the definition of `globalVar`
3. It finds it in `file1.o` (compiled from file1.cpp)
4. It connects the reference in `file2.o` to the definition in `file1.o`

### 7.4 `register` (Deprecated in C++17)

```cpp
register int counter = 0;  // Hint to compiler: keep this in a CPU register

// Modern compilers ignore this — they do register allocation better than humans
// Deprecated in C++17, but still valid syntax
```

### 7.5 `mutable`

Allows a class member to be modified even in a `const` object or through a `const` member function.

```cpp
class Cache {
public:
    int getValue() const {
        if (!cached_) {
            cachedValue_ = expensiveComputation();  // OK because mutable
            cached_ = true;
        }
        return cachedValue_;
    }
private:
    mutable int cachedValue_;
    mutable bool cached_ = false;
    
    int expensiveComputation() const { return 42; }
};

const Cache c;
c.getValue();  // OK — even though c is const, mutable members can change
```

### 7.6 `thread_local` (C++11)

Each thread gets its own independent copy of the variable.

```cpp
thread_local int errorCode = 0;  // Each thread has its own errorCode

void setError(int code) {
    errorCode = code;  // Sets this thread's copy only
}
```

### 7.7 `volatile`

Tells the compiler that a variable may change unexpectedly (by hardware, another thread, or a signal handler). Prevents the compiler from optimizing away reads/writes.

```cpp
volatile int* hardwareRegister = (volatile int*)0x40001000;
// The compiler will always read from the actual memory address,
// never from a cached copy in a CPU register

while (*hardwareRegister == 0) {
    // Without volatile, compiler might optimize this to:
    // int temp = *hardwareRegister;
    // while (temp == 0) { } — infinite loop!
}
```

**Note:** `volatile` does NOT provide atomicity or thread safety. Use `std::atomic` for that.

---

## Chapter 8: Type Conversions

### 8.1 Implicit Conversions (Coercions)

The compiler automatically converts types in certain situations:

**Integer promotion:**
```cpp
char c = 'A';       // 65
int result = c + 1;  // c is promoted to int, result = 66
```

**Arithmetic conversion (usual arithmetic conversions):**
```cpp
int a = 5;
double b = 2.5;
auto result = a + b;  // int is converted to double, result is 7.5 (double)
```

**Conversion hierarchy:**
```
long double ← double ← float
    ↑
long long ← long ← int ← short ← char/bool
```
The "smaller" type is converted to the "larger" type.

**Narrowing conversions (data loss potential):**
```cpp
int x = 3.14;      // 3 (decimal part lost) — WARNING, maybe not error
float f = 1e100;   // Overflow — undefined behavior!
int i = 100000;
short s = i;       // Data loss if i > 32767
```

### 8.2 Explicit Conversions (Casts)

#### C-style cast (avoid in C++):
```cpp
double pi = 3.14;
int truncated = (int)pi;  // C-style cast — works but unsafe
```

#### C++ casts (use these):

**`static_cast`** — Most common, compile-time checked:
```cpp
double pi = 3.14;
int truncated = static_cast<int>(pi);  // 3 — explicit, safe, documented

// Converting between related types:
enum class Color { Red, Green, Blue };
int colorValue = static_cast<int>(Color::Green);  // 1
```

**`dynamic_cast`** — For polymorphic types (has runtime cost):
```cpp
Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
if (derivedPtr != nullptr) {
    // Cast succeeded — basePtr actually pointed to a Derived
}
```

**`const_cast`** — Adds or removes const:
```cpp
const int x = 42;
int* ptr = const_cast<int*>(&x);  // Removes const — DANGEROUS
*ptr = 100;  // Undefined behavior! x was originally declared const
```

**`reinterpret_cast`** — Reinterprets the bit pattern (most dangerous):
```cpp
int x = 42;
float* fp = reinterpret_cast<float*>(&x);
// Treats the bytes of int x as if they were a float
// Almost always wrong — only for very low-level code
```

### 8.3 Type Aliases

```cpp
// C-style (typedef):
typedef unsigned long ulong;
typedef std::vector<std::pair<std::string, int>> NameScoreList;

// C++11 style (using — preferred):
using ulong = unsigned long;
using NameScoreList = std::vector<std::pair<std::string, int>>;

// Template aliases (only possible with 'using'):
template<typename T>
using Vec = std::vector<T>;

Vec<int> numbers;          // std::vector<int>
Vec<std::string> names;    // std::vector<std::string>
```

---

## Chapter 9: Advanced and Expert-Level Coverage

### 9.1 How Types Map to CPU Instructions

When you write `int a = b + c;`, the compiler generates instructions specific to the data type:

```
int:    ADD instruction (integer ALU)
float:  FADD instruction (FPU) or ADDSS (SSE)
double: FADD or ADDSD (SSE2)
```

The CPU has different circuitry for integer and floating-point arithmetic. The choice of type directly affects which hardware unit processes your data.

### 9.2 Type Sizes and Alignment

CPUs access memory most efficiently when data is **aligned** — stored at addresses that are multiples of the data size.

```cpp
struct Example {
    char a;    // 1 byte  — offset 0
    // 3 bytes padding (to align int to 4-byte boundary)
    int b;     // 4 bytes — offset 4
    char c;    // 1 byte  — offset 8
    // 7 bytes padding (to align double to 8-byte boundary)
    double d;  // 8 bytes — offset 16
};
// sizeof(Example) = 24 bytes (not 14 as you might expect!)
```

**Reordering for less padding:**
```cpp
struct BetterExample {
    double d;  // 8 bytes — offset 0
    int b;     // 4 bytes — offset 8
    char a;    // 1 byte  — offset 12
    char c;    // 1 byte  — offset 13
    // 2 bytes padding
};
// sizeof(BetterExample) = 16 bytes (saved 8 bytes!)
```

**Rule of thumb:** Order struct members from largest to smallest.

### 9.3 Strict Aliasing Rule

The compiler assumes that pointers of different types don't alias (point to the same memory):

```cpp
int x = 42;
float* fp = (float*)(&x);  // WRONG! Violates strict aliasing
float f = *fp;              // UNDEFINED BEHAVIOR

// Exception: char* can alias any type
char* bytes = (char*)(&x);
for (int i = 0; i < sizeof(int); i++) {
    printf("%02x ", (unsigned char)bytes[i]);
}
// This is well-defined — char* has special aliasing permissions
```

### 9.4 `std::numeric_limits` (C++11)

```cpp
#include <limits>

std::cout << std::numeric_limits<int>::min() << "\n";       // -2147483648
std::cout << std::numeric_limits<int>::max() << "\n";       // 2147483647
std::cout << std::numeric_limits<double>::epsilon() << "\n"; // ~2.22e-16
std::cout << std::numeric_limits<double>::infinity() << "\n";
std::cout << std::numeric_limits<float>::digits10 << "\n";  // 6 (reliable decimal digits)
std::cout << std::numeric_limits<double>::digits10 << "\n"; // 15
```

### 9.5 `decltype` — Querying Types (C++11)

```cpp
int x = 42;
decltype(x) y = 100;    // y is int (same type as x)

const int& ref = x;
decltype(ref) z = x;    // z is const int& (same type as ref)

// Useful in templates:
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}
```

### 9.6 `sizeof` Operator

```cpp
int x;
std::cout << sizeof(int) << "\n";   // Size of type in bytes
std::cout << sizeof(x) << "\n";     // Size of variable
std::cout << sizeof x << "\n";      // Parentheses optional for variables

int arr[10];
std::cout << sizeof(arr) << "\n";          // 40 (10 × 4 bytes)
std::cout << sizeof(arr) / sizeof(arr[0]); // 10 (number of elements)
// Modern: use std::size(arr) or arr.size() for containers
```

---

## Chapter 10: Interview & Competitive Programming Section

### 10.1 Common Interview Questions

**Q1: What is the difference between `float` and `double`?**
A: `float` is 32 bits with ~7 decimal digits precision. `double` is 64 bits with ~15-16 decimal digits precision. Always prefer `double` unless you have specific constraints.

**Q2: Can `sizeof(int)` be 2?**
A: Yes! The C++ standard only guarantees `sizeof(int) >= 2`. On some embedded platforms, int can be 16 bits. This is why portable code uses `<cstdint>` types like `int32_t`.

**Q3: What happens when you assign a negative number to an `unsigned int`?**
A: The value wraps around using modular arithmetic. `unsigned int x = -1;` makes x the maximum unsigned value (4294967295 on 32-bit systems). This is well-defined behavior.

**Q4: Why is `0.1 + 0.2 != 0.3`?**
A: Because 0.1 and 0.2 cannot be represented exactly in binary floating-point (IEEE 754). The tiny representation errors accumulate, making the sum slightly different from the representation of 0.3.

**Q5: What is the difference between `const` and `constexpr`?**
A: `const` means the value can't be modified after initialization but may be set at runtime. `constexpr` guarantees the value is computed at compile time, enabling its use in constant expressions (array sizes, template arguments).

**Q6: What is `nullptr` and why was it introduced?**
A: `nullptr` is a type-safe null pointer literal of type `std::nullptr_t` introduced in C++11. It replaces `NULL` (which is just `0`) to eliminate ambiguity in function overloading between integer and pointer parameters.

**Q7: What is the size of an empty struct?**
A: 1 byte. C++ requires every object to have a unique address, so empty structs/classes must have non-zero size. (Exception: Empty Base Optimization (EBO) can make base class zero-sized.)

### 10.2 Tricky Output Questions

```cpp
// Q: What's the output?
#include <iostream>
int main() {
    unsigned int x = 10;
    int y = -20;
    if (x + y > 0) {
        std::cout << "Greater";
    } else {
        std::cout << "Lesser or Equal";
    }
}
```
**Answer:** "Greater" — because `y` is implicitly converted to unsigned before addition. `-20` becomes a huge unsigned number (4294967276), and `10 + 4294967276 = 4294967286`, which is > 0.

```cpp
// Q: What's the output?
#include <iostream>
int main() {
    char c = 255;
    std::cout << (int)c << "\n";
}
```
**Answer:** Implementation-defined! If `char` is signed (most common), 255 overflows to -1. If `char` is unsigned, it prints 255.

---

## Chapter 11: Practice Section

### 11.1 Beginner Exercises

1. Write a program that prints the size (in bytes) and range of every fundamental type on your system.

2. Demonstrate the difference between `int`, `float`, and `double` precision by computing `1.0/3.0` with each type and printing 20 decimal places.

3. Write a program that converts between decimal, hexadecimal, octal, and binary representations of a number.

4. Create a program that demonstrates overflow behavior for both `signed int` and `unsigned int`.

5. Write a program using `const`, `constexpr`, and `#define` and explain the differences in comments.

### 11.2 Intermediate Exercises

6. Write a program that examines the memory layout of a `float` — extract and print the sign, exponent, and mantissa bits.

7. Create a struct and demonstrate padding/alignment. Reorder members to minimize size.

8. Write a program that demonstrates all five types of variable lifetime (automatic, static, global, dynamic, thread-local).

9. Implement a temperature class that uses user-defined literals: `auto t = 100.0_celsius;`

### 11.3 Advanced Exercises

10. Write a `type_traits`-like utility that prints the type category of any variable (integral, floating-point, pointer, array, class, etc.) using template metaprogramming.

11. Implement a `SafeInt` class that detects integer overflow at runtime and throws an exception.

12. Write a program that demonstrates strict aliasing violations and shows how the optimizer can generate incorrect code when the rule is violated (compare with and without `-fstrict-aliasing`).

---

## Chapter 12: Mini Projects

### Project 1: Type Explorer Tool (Beginner)
Build a command-line tool that:
- Displays all fundamental types with their sizes, ranges, and alignment on the current system
- Shows the binary representation of any entered number
- Demonstrates type conversion behavior

### Project 2: IEEE 754 Visualizer (Intermediate)
Build a program that:
- Takes a decimal number as input
- Shows its exact IEEE 754 representation (sign, exponent, mantissa)
- Computes and displays the actual stored value
- Shows the representation error

### Project 3: Memory Layout Analyzer (Advanced)
Build a tool that:
- Takes a struct definition as input
- Displays the memory layout with padding bytes highlighted
- Suggests optimal member ordering to minimize size
- Shows alignment requirements for each member

---

## Chapter 13: Summary & Mastery Checklist

### What You Must Master Before Moving Forward

- [ ] You can name every primitive type and know its size and range
- [ ] You understand IEEE 754 floating-point and its pitfalls
- [ ] You know the difference between `signed` and `unsigned` and overflow behavior
- [ ] You understand variables: scope, lifetime, linkage
- [ ] You can use `const`, `constexpr`, `consteval`, and `constinit` correctly
- [ ] You know when to use each C++ cast (`static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`)
- [ ] You understand type promotion and implicit conversion rules
- [ ] You know all storage classes and when to use each
- [ ] You can explain struct padding and alignment
- [ ] You understand the strict aliasing rule
- [ ] You know the difference between `nullptr`, `NULL`, and `0`
- [ ] You can write and use user-defined literals

### Self-Evaluation

Rate yourself 1-5 on each:
1. Can you predict the output of code involving mixed signed/unsigned arithmetic? ___
2. Can you explain why floating-point comparison with `==` is dangerous? ___
3. Can you calculate struct sizes accounting for padding? ___
4. Can you explain the memory representation of negative integers? ___
5. Can you choose the right type and constant mechanism for any situation? ___

**If all ratings are 4+, you are ready for Book 3: Operators & Control Flow.**

---

*End of Book 2: Data Types & Variables*
*Next: Book 3 — Operators & Control Flow*
