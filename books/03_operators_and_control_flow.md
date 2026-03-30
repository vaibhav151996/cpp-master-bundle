# Book 3: Operators & Control Flow

## The Complete Guide to C++ Operators, Expressions, and Decision Making

---

### **Target Level:** Beginner
### **Prerequisites:** Book 1 (C++ Foundations), Book 2 (Data Types & Variables)
### **Learning Outcomes:**
By the end of this book, you will:
- Master every operator in C++ with deep understanding of behavior and edge cases
- Understand operator precedence and associativity completely
- Know how operators work at the assembly/machine level
- Master all conditional statements (if, switch, ternary)
- Understand short-circuit evaluation, sequence points, and evaluation order
- Handle operator overloading concepts (introduced, detailed in OOP book)
- Write clean, correct conditional logic for production code

---

## Chapter 1: Conceptual Foundations

### 1.1 What Are Operators?

Operators are **symbols that tell the compiler to perform specific mathematical, logical, or other operations**. In C++, operators are essentially shorthand for function calls. In fact, almost every operator can be overloaded — defined as a function for user-defined types.

```cpp
int result = 3 + 4;
// The compiler sees this as: operator+(3, 4)
```

### 1.2 Expressions, Statements, and Side Effects

**Expression:** A combination of values, variables, operators, and function calls that produces a value.
```cpp
a + b           // Expression — evaluates to a value
x = 5           // Expression — evaluates to 5 (and has the side effect of assigning to x)
func()          // Expression — evaluates to the return value
```

**Statement:** An expression followed by a semicolon.
```cpp
a + b;          // Expression statement (useless — result discarded)
x = 5;          // Expression statement (useful — side effect of assignment)
```

**Side Effect:** Any observable change to program state beyond returning a value.
```cpp
x = 5;          // Side effect: x changes value
std::cout << x; // Side effect: output to console
i++;            // Side effect: i changes value
```

---

## Chapter 2: Arithmetic Operators

### 2.1 Basic Arithmetic

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 3` | `1` (integer) |
| `%` | Modulus (remainder) | `5 % 3` | `2` |
| `+` | Unary plus | `+5` | `5` |
| `-` | Unary minus | `-5` | `-5` |

### 2.2 Integer Division — The Most Common Beginner Mistake

```cpp
int a = 5, b = 3;
int result = a / b;     // result = 1 (NOT 1.666...)
// Integer division truncates toward zero

double d = a / b;       // d = 1.0 (NOT 1.666...)
// The division happens FIRST (as int/int = int), THEN converts to double

double correct = static_cast<double>(a) / b;  // d = 1.666...
// OR
double correct2 = a / static_cast<double>(b); // d = 1.666...
// OR
double correct3 = a * 1.0 / b;               // d = 1.666...
// Converting either operand to double forces floating-point division
```

**Division rules:**
- `int / int` → `int` (truncates toward zero)
- `double / int` → `double`
- `int / double` → `double`
- `double / double` → `double`

### 2.3 Modulus Operator `%`

Returns the remainder of integer division. Only works with integer types.

```cpp
10 % 3  = 1    // 10 = 3 * 3 + 1
7  % 2  = 1    // 7  = 2 * 3 + 1
12 % 4  = 0    // 12 = 4 * 3 + 0 (evenly divisible)
15 % 7  = 1    // 15 = 7 * 2 + 1
```

**Negative modulus (C++11 guarantees truncation toward zero):**
```cpp
-7 % 3  = -1   // (-7) / 3 = -2 (toward zero), remainder = -7 - (-2*3) = -1
7 % -3  = 1    // 7 / (-3) = -2 (toward zero), remainder = 7 - (-2*-3) = 1
```

**Common use of modulus:**
```cpp
// Check if even or odd:
if (n % 2 == 0) { /* even */ }

// Wrap around (clock arithmetic):
int hour = (currentHour + 5) % 24;   // Always stays in 0-23 range

// Get last digit:
int lastDigit = number % 10;

// Check divisibility:
if (year % 4 == 0 && (year % 100 != 0 || year % 400 == 0)) {
    // Leap year
}
```

### 2.4 Increment and Decrement Operators

```cpp
int x = 5;

// Pre-increment: increment FIRST, then use the value
int a = ++x;   // x becomes 6, a = 6

// Post-increment: use the value FIRST, then increment
int b = x++;   // b = 6 (old value), x becomes 7
```

**Under the hood:**
```cpp
// Pre-increment (++x) is equivalent to:
x = x + 1;
// returns the new value of x

// Post-increment (x++) is equivalent to:
int temp = x;   // Save old value
x = x + 1;      // Increment
// returns temp (old value)
```

**Performance note:** For primitive types, there is no performance difference. For iterators and complex objects, **prefer pre-increment** (`++it`) because post-increment creates a temporary copy.

```cpp
// For iterators:
for (auto it = vec.begin(); it != vec.end(); ++it) {  // Preferred
    // ...
}
```

**Undefined behavior warning:**
```cpp
int i = 5;
int j = i++ + ++i;  // UNDEFINED BEHAVIOR in C++14 and earlier!
// Don't modify a variable more than once in the same expression
```

### 2.5 Arithmetic at the Assembly Level

For `int c = a + b;`, the compiler generates:
```asm
mov  eax, [a]        ; Load a into register eax
add  eax, [b]        ; Add b to eax
mov  [c], eax        ; Store result in c
```

For `double d = a + b;`:
```asm
movsd  xmm0, [a]     ; Load a into SSE register xmm0
addsd  xmm0, [b]     ; Add b using SSE double-precision add
movsd  [d], xmm0     ; Store result
```

Integer and floating-point arithmetic use **completely different CPU hardware**.

---

## Chapter 3: Relational Operators

### 3.1 Comparison Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `==` | Equal to | `5 == 5` | `true` |
| `!=` | Not equal to | `5 != 3` | `true` |
| `<` | Less than | `3 < 5` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<=` | Less than or equal | `5 <= 5` | `true` |
| `>=` | Greater than or equal | `5 >= 3` | `true` |

All relational operators return a `bool` value (`true` or `false`).

### 3.2 The `==` vs `=` Mistake

```cpp
int x = 5;

if (x = 3) {    // WRONG! This ASSIGNS 3 to x, then checks if 3 is non-zero (true)
    // Always executes!
}

if (x == 3) {   // CORRECT! This checks if x equals 3
    // Executes only if x is 3
}
```

**Defensive technique (Yoda conditions):**
```cpp
if (3 == x) {   // If you accidentally write = instead of ==, you get a compiler error
    // because you can't assign to a literal
}
```

Modern compilers warn about `=` in conditionals (with `-Wall`), making Yoda conditions less necessary.

### 3.3 Floating-Point Comparison

**Never use `==` to compare floating-point numbers:**
```cpp
double a = 0.1 + 0.2;
double b = 0.3;

// WRONG:
if (a == b) { /* may not execute */ }

// CORRECT — absolute tolerance:
const double EPSILON = 1e-9;
if (std::abs(a - b) < EPSILON) { /* correct */ }

// BETTER — relative tolerance (for values of varying magnitude):
bool approximatelyEqual(double a, double b, double relTol = 1e-9) {
    return std::abs(a - b) <= relTol * std::max(std::abs(a), std::abs(b));
}
```

### 3.4 Three-Way Comparison `<=>` (C++20)

The **spaceship operator** returns one of three results:

```cpp
#include <compare>

auto result = 5 <=> 3;    // result is std::strong_ordering::greater
auto result2 = 3 <=> 5;   // result is std::strong_ordering::less
auto result3 = 5 <=> 5;   // result is std::strong_ordering::equal

// Use in classes to auto-generate all six comparison operators:
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;
    // Automatically generates ==, !=, <, >, <=, >= based on member comparison
};
```

---

## Chapter 4: Logical Operators

### 4.1 Logical Operators Table

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `&&` | Logical AND | `true && false` | `false` |
| `\|\|` | Logical OR | `true \|\| false` | `true` |
| `!` | Logical NOT | `!true` | `false` |

### 4.2 Truth Tables

**AND (`&&`):**
| A | B | A && B |
|---|---|--------|
| false | false | false |
| false | true  | false |
| true  | false | false |
| true  | true  | **true** |

Both must be true.

**OR (`||`):**
| A | B | A \|\| B |
|---|---|---------|
| false | false | false |
| false | true  | **true** |
| true  | false | **true** |
| true  | true  | **true** |

At least one must be true.

**NOT (`!`):**
| A | !A |
|---|-----|
| false | **true** |
| true  | false |

### 4.3 Short-Circuit Evaluation

C++ guarantees that `&&` and `||` evaluate left-to-right and stop as soon as the result is known.

```cpp
// Short-circuit AND:
if (ptr != nullptr && ptr->value > 10) {
    // If ptr IS nullptr, the second condition is NEVER evaluated
    // This prevents a null pointer dereference crash!
}

// Short-circuit OR:
if (isEmpty() || container.front() == target) {
    // If isEmpty() returns true, front() is never called
    // Prevents accessing front() on an empty container!
}
```

**Performance implication:** Put the cheapest/most-likely-to-short-circuit condition first:
```cpp
// Prefer:
if (fastCheck() && slowDatabaseQuery()) { ... }

// Over:
if (slowDatabaseQuery() && fastCheck()) { ... }
```

### 4.4 Common Logic Patterns

```cpp
// Range check:
if (x >= 0 && x < size) { /* x is a valid index */ }

// Multiple conditions:
if (age >= 18 && hasLicense && !isSuspended) { /* can drive */ }

// De Morgan's Laws — useful for simplifying conditions:
// !(A && B) is equivalent to (!A || !B)
// !(A || B) is equivalent to (!A && !B)

// Example:
if (!(x > 0 && y > 0)) { /* at least one is non-positive */ }
// is the same as:
if (x <= 0 || y <= 0) { /* at least one is non-positive */ }
```

---

## Chapter 5: Bitwise Operators

### 5.1 Bitwise Operators Table

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `&` | Bitwise AND | `0b1100 & 0b1010` | `0b1000` |
| `\|` | Bitwise OR | `0b1100 \| 0b1010` | `0b1110` |
| `^` | Bitwise XOR | `0b1100 ^ 0b1010` | `0b0110` |
| `~` | Bitwise NOT | `~0b1100` | `0b...0011` |
| `<<` | Left shift | `0b0001 << 3` | `0b1000` |
| `>>` | Right shift | `0b1000 >> 2` | `0b0010` |

### 5.2 How Bitwise Operations Work

```
  1 1 0 0   (12 in decimal)
& 1 0 1 0   (10 in decimal)
---------
  1 0 0 0   (8 in decimal)
Each bit is ANDed independently

  1 1 0 0   (12)
| 1 0 1 0   (10)
---------
  1 1 1 0   (14)
Each bit is ORed independently

  1 1 0 0   (12)
^ 1 0 1 0   (10)
---------
  0 1 1 0   (6)
Each bit is XORed independently (1 if different, 0 if same)
```

### 5.3 Common Bit Manipulation Techniques

```cpp
// 1. Set a bit (turn it ON)
x |= (1 << n);        // Set bit n

// 2. Clear a bit (turn it OFF)
x &= ~(1 << n);       // Clear bit n

// 3. Toggle a bit (flip it)
x ^= (1 << n);        // Toggle bit n

// 4. Check if a bit is set
if (x & (1 << n)) {   // True if bit n is 1
    // bit n is set
}

// 5. Check if a number is even or odd
if (x & 1) { /* odd */ } else { /* even */ }

// 6. Multiply by power of 2
x << 3;  // x * 8  (x * 2³)

// 7. Divide by power of 2
x >> 2;  // x / 4  (x / 2²)

// 8. Swap two numbers without temp variable
a ^= b;
b ^= a;
a ^= b;   // a and b are now swapped!

// 9. Check if power of 2
bool isPowerOf2 = (x > 0) && ((x & (x - 1)) == 0);

// 10. Get lowest set bit
int lowestBit = x & (-x);

// 11. Count set bits (population count)
int count = __builtin_popcount(x);     // GCC/Clang intrinsic
// C++20: #include <bit>  std::popcount(x);

// 12. Clear the lowest set bit
x &= (x - 1);
```

### 5.4 Bit Flags Pattern

```cpp
enum Permissions : unsigned int {
    READ    = 1 << 0,  // 0b0001 = 1
    WRITE   = 1 << 1,  // 0b0010 = 2
    EXECUTE = 1 << 2,  // 0b0100 = 4
    DELETE  = 1 << 3   // 0b1000 = 8
};

unsigned int userPerms = READ | WRITE;        // 0b0011 = 3

// Check permission:
if (userPerms & READ) { /* can read */ }

// Add permission:
userPerms |= EXECUTE;                         // 0b0111

// Remove permission:
userPerms &= ~WRITE;                          // 0b0101

// Toggle permission:
userPerms ^= DELETE;                          // 0b1101
```

### 5.5 Shift Operator Details

**Left shift (`<<`):**
```cpp
int x = 1;      // 0b00000001
x << 1;         // 0b00000010 = 2     (x * 2)
x << 3;         // 0b00001000 = 8     (x * 8)
```
Shifts bits left, fills with 0s from the right. Equivalent to multiplication by $2^n$.

**Right shift (`>>`):**
```cpp
int x = 16;     // 0b00010000
x >> 1;         // 0b00001000 = 8     (x / 2)
x >> 3;         // 0b00000010 = 2     (x / 8)
```

For unsigned types: fills with 0s from the left (logical shift).
For signed types: implementation-defined! (usually arithmetic shift — fills with sign bit).

**Shift undefined behaviors:**
```cpp
int x = 1;
x << 32;        // UB! Shift count >= type width
x << -1;        // UB! Negative shift count
(-1) << 1;      // UB before C++20! Shifting negative values
                // (Well-defined in C++20 as two's complement)
```

---

## Chapter 6: Assignment Operators

### 6.1 Simple and Compound Assignment

| Operator | Equivalent | Example |
|----------|-----------|---------|
| `=`  | | `x = 5` |
| `+=` | `x = x + y` | `x += 3` → `x = x + 3` |
| `-=` | `x = x - y` | `x -= 3` |
| `*=` | `x = x * y` | `x *= 3` |
| `/=` | `x = x / y` | `x /= 3` |
| `%=` | `x = x % y` | `x %= 3` |
| `&=` | `x = x & y` | `x &= 0xFF` |
| `\|=` | `x = x \| y` | `x \|= FLAG` |
| `^=` | `x = x ^ y` | `x ^= MASK` |
| `<<=` | `x = x << y` | `x <<= 2` |
| `>>=` | `x = x >> y` | `x >>= 2` |

### 6.2 Assignment Returns a Value

The `=` operator returns a reference to the left operand:
```cpp
int a, b, c;
a = b = c = 42;    // Right-to-left: c=42, b=c, a=b
// All three variables are now 42

// This is why if (x = 5) works (but is usually a bug)
```

---

## Chapter 7: Operator Precedence and Associativity

### 7.1 Complete Precedence Table (Highest to Lowest)

| Precedence | Operator(s) | Associativity | Description |
|-----------|-------------|---------------|-------------|
| 1 | `::` | Left | Scope resolution |
| 2 | `a++` `a--` `()` `[]` `.` `->` | Left | Postfix |
| 3 | `++a` `--a` `+a` `-a` `!` `~` `*` `&` `sizeof` `new` `delete` `(type)` | Right | Unary/Prefix |
| 4 | `.*` `->*` | Left | Pointer-to-member |
| 5 | `*` `/` `%` | Left | Multiplicative |
| 6 | `+` `-` | Left | Additive |
| 7 | `<<` `>>` | Left | Shift |
| 8 | `<=>` | Left | Three-way comparison |
| 9 | `<` `<=` `>` `>=` | Left | Relational |
| 10 | `==` `!=` | Left | Equality |
| 11 | `&` | Left | Bitwise AND |
| 12 | `^` | Left | Bitwise XOR |
| 13 | `\|` | Left | Bitwise OR |
| 14 | `&&` | Left | Logical AND |
| 15 | `\|\|` | Left | Logical OR |
| 16 | `?:` | Right | Ternary/Conditional |
| 17 | `=` `+=` `-=` `*=` `/=` etc. | Right | Assignment |
| 18 | `,` | Left | Comma |

### 7.2 Precedence Pitfalls

```cpp
// Pitfall 1: Bitwise vs comparison
if (x & MASK == 0) { }     // WRONG! Parsed as x & (MASK == 0)
if ((x & MASK) == 0) { }   // CORRECT

// Pitfall 2: Shift vs addition
int val = 1 << 4 + 1;      // WRONG! Parsed as 1 << (4+1) = 32, not (1<<4)+1 = 17
int val = (1 << 4) + 1;    // CORRECT = 17

// Pitfall 3: Assignment vs equality
if (x = 5) { }             // Assigns 5, always true (not a comparison!)
if (x == 5) { }            // Correct comparison

// Pitfall 4: Logical vs bitwise
if (a & b || c) { }        // Parsed as (a & b) || c
if (a & (b || c)) { }      // What you might have meant
```

**Best Practice:** When in doubt, use parentheses! They make your intent explicit and prevent bugs.

### 7.3 Associativity

**Left-to-right:** Most operators
```cpp
int x = 10 - 5 - 2;  // (10 - 5) - 2 = 3 (not 10 - (5-2) = 7)
```

**Right-to-left:** Assignment, unary, ternary
```cpp
a = b = c = 5;  // a = (b = (c = 5)) → all become 5
int x = -++y;   // -(++y)
```

---

## Chapter 8: Conditional Statements (Control Flow)

### 8.1 The `if` Statement

```cpp
if (condition) {
    // Executes if condition is true (non-zero)
}
```

**What counts as "true"?**
```cpp
if (42) { }          // true — any non-zero integer
if (-1) { }          // true — negative numbers are non-zero
if (0) { }           // false — zero is false
if (0.001) { }       // true — non-zero float
if (0.0) { }         // false — zero float
if (ptr) { }         // true — non-null pointer
if (nullptr) { }     // false — null pointer
if ("hello") { }     // true — string literal decays to non-null pointer
```

### 8.2 `if-else`

```cpp
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) {
    grade = 'B';
} else if (score >= 70) {
    grade = 'C';
} else if (score >= 60) {
    grade = 'D';
} else {
    grade = 'F';
}
```

### 8.3 Nested `if` — The Dangling Else Problem

```cpp
// This is ambiguous:
if (a > 0)
    if (b > 0)
        std::cout << "both positive";
else
    std::cout << "???";   // Does this belong to the inner or outer if?

// Answer: In C++, else binds to the NEAREST unmatched if (inner)
// Equivalent to:
if (a > 0) {
    if (b > 0) {
        std::cout << "both positive";
    } else {
        std::cout << "???";   // Executes when a>0 but b<=0
    }
}

// Always use braces to make intent clear!
```

### 8.4 `if` with Initializer (C++17)

```cpp
// Before C++17:
auto it = myMap.find("key");
if (it != myMap.end()) {
    // use it
}
// 'it' is still in scope here — potential misuse

// C++17:
if (auto it = myMap.find("key"); it != myMap.end()) {
    // use it
}
// 'it' is out of scope here — cleaner!

// Also works with status codes:
if (auto [iter, success] = myMap.insert({"key", 42}); success) {
    std::cout << "Inserted at position: " << iter->first << "\n";
}
```

### 8.5 The `switch` Statement

```cpp
switch (expression) {
    case constant1:
        // code
        break;
    case constant2:
        // code
        break;
    case constant3:
    case constant4:     // Fall-through (handles both 3 and 4)
        // code
        break;
    default:
        // code (when no case matches)
        break;
}
```

**Example:**
```cpp
int day = 3;
switch (day) {
    case 1: std::cout << "Monday";    break;
    case 2: std::cout << "Tuesday";   break;
    case 3: std::cout << "Wednesday"; break;
    case 4: std::cout << "Thursday";  break;
    case 5: std::cout << "Friday";    break;
    case 6: std::cout << "Saturday";  break;
    case 7: std::cout << "Sunday";    break;
    default: std::cout << "Invalid day";
}
```

**The `[[fallthrough]]` attribute (C++17):**
```cpp
switch (x) {
    case 1:
        doSomething();
        [[fallthrough]];    // Explicitly says: I INTEND to fall through
    case 2:
        doSomethingElse();  // Executes for both case 1 and 2
        break;
    case 3:
        doThirdThing();
        break;
}
```

Without `[[fallthrough]]`, the compiler warns about accidental fall-through.

**`switch` with initializer (C++17):**
```cpp
switch (auto val = computeValue(); val) {
    case 1: /* ... */ break;
    case 2: /* ... */ break;
    default: /* ... */ break;
}
```

**`switch` limitations:**
- Cases must be **compile-time integer constants** (not variables, not floats, not strings)
- The expression must evaluate to an **integral or enumeration type**
- You cannot use ranges (like `case 1..5:`) — this is a GCC extension, not standard

### 8.6 The Ternary Operator `?:`

The only operator that takes three operands:

```cpp
// Syntax: condition ? value_if_true : value_if_false

int max = (a > b) ? a : b;

// Equivalent to:
int max;
if (a > b) {
    max = a;
} else {
    max = b;
}
```

**Nested ternary (use sparingly):**
```cpp
std::string grade = (score >= 90) ? "A"
                  : (score >= 80) ? "B"
                  : (score >= 70) ? "C"
                  : (score >= 60) ? "D"
                  : "F";
```

**Ternary as lvalue (advanced):**
```cpp
int a = 5, b = 10;
(a > b ? a : b) = 20;  // Assigns 20 to b (since b > a)
// This works because ternary can return an lvalue!
```

**When to use ternary vs if-else:**
- Use ternary for **simple, short expressions** that fit on one line
- Use if-else for **complex logic** with multiple statements
- Never nest ternary more than one level deep

---

## Chapter 9: Other Important Operators

### 9.1 The Comma Operator `,`

Evaluates both operands and returns the value of the right one:
```cpp
int x = (1, 2, 3);     // x = 3 (only the last value is kept)

// Common use in for loops:
for (int i = 0, j = 10; i < j; ++i, --j) {
    std::cout << i << " " << j << "\n";
}
```

### 9.2 The `sizeof` Operator

```cpp
sizeof(int)           // 4 (typically)
sizeof(double)        // 8
sizeof(char)          // 1 (always, by definition)

int arr[10];
sizeof(arr)           // 40 (10 × 4)
sizeof(arr[0])        // 4
sizeof(arr) / sizeof(arr[0])  // 10 (number of elements)
```

`sizeof` is a **compile-time** operator — it does not evaluate its operand:
```cpp
int x = 5;
sizeof(x++);  // x is NOT incremented! sizeof doesn't evaluate the expression.
```

### 9.3 The `typeid` Operator

```cpp
#include <typeinfo>

int x = 42;
std::cout << typeid(x).name() << "\n";     // "i" or "int" (implementation-defined)
std::cout << typeid(3.14).name() << "\n";  // "d" or "double"
```

### 9.4 Cast Operators

As covered in Book 2:
- `static_cast<T>(expr)` — Compile-time type conversion
- `dynamic_cast<T>(expr)` — Runtime polymorphic cast
- `const_cast<T>(expr)` — Add/remove const
- `reinterpret_cast<T>(expr)` — Bit reinterpretation

---

## Chapter 10: Advanced and Expert-Level Coverage

### 10.1 Sequence Points and Evaluation Order

**Before C++17:** The order in which function arguments are evaluated is **unspecified**:
```cpp
int i = 0;
func(i++, i++);     // UB! Two unsequenced modifications of i
```

**C++17 changes:**
- Function arguments are still **unsequenced** relative to each other
- But the `<<` operator chains are sequenced left-to-right
- Assignment right-before-left (RHS is evaluated before LHS)

```cpp
// C++17: Well-defined
std::cout << i++ << i++;    // Left << is sequenced before right <<
// But still confusing — don't write code like this

// Still UB in C++17:
int i = 0;
func(i++, i++);  // Arguments are indeterminately sequenced
```

### 10.2 How the Compiler Optimizes Conditions

The compiler performs several optimizations on conditional code:

**Branch prediction hints:**
```cpp
// C++20:
if (x > 0) [[likely]] {
    // Compiler optimizes for this path being taken most often
} else [[unlikely]] {
    // Rare path — compiler may put this code further away in memory
}
```

**Branchless code:** The compiler may replace simple if-else with conditional moves:
```cpp
// Source:
int max = (a > b) ? a : b;

// Compiler may generate (x86):
cmovg eax, ebx    // Conditional move — no branch, no prediction miss
```

### 10.3 Operator Overloading Preview

In C++, most operators can be defined for your own types:
```cpp
struct Vector2D {
    double x, y;
    
    Vector2D operator+(const Vector2D& other) const {
        return {x + other.x, y + other.y};
    }
    
    bool operator==(const Vector2D& other) const {
        return x == other.x && y == other.y;
    }
    
    // C++20 spaceship operator generates all 6 comparison operators
    auto operator<=>(const Vector2D& other) const = default;
};

Vector2D a{1.0, 2.0};
Vector2D b{3.0, 4.0};
Vector2D c = a + b;    // Uses our operator+, c = {4.0, 6.0}
```

Detailed coverage in Book 9 (OOP).

### 10.4 Performance: Branch vs Branchless

Modern CPUs use **branch prediction** to guess which path a conditional will take. When the prediction is wrong, it costs ~10-20 CPU cycles (pipeline flush).

```cpp
// Branch-heavy code (may cause prediction misses):
for (int i = 0; i < n; i++) {
    if (data[i] > threshold) {
        result += data[i];
    }
}

// Branchless alternative (always predictable):
for (int i = 0; i < n; i++) {
    result += (data[i] > threshold) * data[i];
}

// Even better — sort the data first so branches become predicted:
std::sort(data, data + n);
for (int i = 0; i < n; i++) {
    if (data[i] > threshold) {
        result += data[i];
    }
}
```

This is the principle behind the famous "Why is processing a sorted array faster?" interview question.

---

## Chapter 11: Interview & Competitive Programming Section

### 11.1 Common Interview Questions

**Q1: What is the difference between `&` and `&&`?**
A: `&` is bitwise AND (operates on individual bits of integers). `&&` is logical AND (operates on boolean values and supports short-circuit evaluation).

**Q2: What is short-circuit evaluation?**
A: When the compiler skips evaluating the right operand because the left operand already determines the result. `false && anything` is `false` (right side not evaluated). `true || anything` is `true` (right side not evaluated).

**Q3: What is the output?**
```cpp
int x = 5;
std::cout << (x > 3 ? "yes" : "no");
```
A: "yes" — x (5) is greater than 3.

**Q4: Can `switch` work with strings?**
A: No, `switch` in C++ only works with integral types and enumerations. For string matching, use if-else chains or a `std::map`/`std::unordered_map` lookup.

**Q5: How do you swap two variables without a temporary?**
A: Using XOR: `a ^= b; b ^= a; a ^= b;` (but this fails if a and b are the same variable). Or using arithmetic: `a = a + b; b = a - b; a = a - b;` (but this can overflow). In practice, just use `std::swap(a, b);`.

**Q6: What does `x & (x - 1)` do?**
A: It clears the lowest set bit of x. If the result is 0 (and x > 0), then x is a power of 2.

### 11.2 Tricky Output Questions

```cpp
// Q1: What is the output?
int a = 5, b = 10;
int c = a > b ? a : b;
std::cout << c;
// Answer: 10

// Q2: What is the output?
int x = 0;
if (x)
    std::cout << "A";
else if (x = 5)
    std::cout << "B";
else
    std::cout << "C";
std::cout << " " << x;
// Answer: B 5  (x=5 assigns 5, which is truthy)

// Q3: What is the output?
int p = 10, q = 20;
int r = (p > q) ? (p + q) : (p - q);
std::cout << r;
// Answer: -10

// Q4: What is the output?
int x = 0xFF;
std::cout << (x & 0x0F) << " " << (x >> 4);
// Answer: 15 15  (0xFF & 0x0F = 0x0F = 15; 0xFF >> 4 = 0x0F = 15)
```

---

## Chapter 12: Practice Section

### 12.1 Beginner Exercises

1. Write a program that determines if a year is a leap year.
2. Build a simple grade calculator using if-else chains.
3. Write a program that takes a character and determines if it's a vowel, consonant, digit, or special character.
4. Implement a simple menu-driven calculator using switch.
5. Use bitwise operators to check if a number is even or odd without using `%`.

### 12.2 Intermediate Exercises

6. Implement all bitwise tricks from Section 5.3 and verify them with test cases.
7. Write a program that converts decimal to binary using bitwise operations.
8. Implement a permissions system using bit flags.
9. Write a function that counts the number of set bits in an integer (without `__builtin_popcount`).
10. Demonstrate precedence pitfalls — write expressions that give unexpected results without parentheses.

### 12.3 Advanced Exercises

11. Implement a branchless `min`/`max` function using bitwise operations.
12. Write a bit manipulation library that provides `setBit`, `clearBit`, `toggleBit`, `checkBit`, `countBits`, `reverseBits` functions.
13. Implement a simple expression evaluator that handles `+`, `-`, `*`, `/` with proper precedence.
14. Compare the performance of branching vs branchless code for a conditional sum over a large array (sorted vs unsorted).

---

## Chapter 13: Mini Projects

### Project 1: Bitwise Permission System (Beginner)
Implement a user permission system:
- Define permissions as bit flags (READ, WRITE, EXECUTE, DELETE, ADMIN)
- Create functions to add, remove, check, and display permissions
- Build a command-line interface to manage user permissions

### Project 2: Expression Parser (Intermediate)
Build a program that:
- Parses mathematical expressions from a string (e.g., "3 + 4 * 2 - 1")
- Respects operator precedence
- Supports parentheses
- Evaluates and prints the result

### Project 3: Decision Tree Engine (Advanced)
Create a configurable decision tree that:
- Reads rules from a configuration file
- Evaluates conditions against input data
- Supports AND, OR, NOT combinations
- Outputs decisions based on the rules

---

## Chapter 14: Summary & Mastery Checklist

### What You Must Master Before Moving Forward

- [ ] You can use all arithmetic, relational, logical, and bitwise operators correctly
- [ ] You understand operator precedence and can predict expression evaluation order
- [ ] You know the difference between bitwise and logical operators
- [ ] You understand short-circuit evaluation and use it defensively
- [ ] You can manipulate individual bits using bitwise operators
- [ ] You can write correct if-else chains, switch statements, and ternary expressions
- [ ] You understand the dangling else problem and always use braces
- [ ] You can use C++17 `if` with initializer and `[[fallthrough]]`
- [ ] You understand undefined behavior related to operators (shift, overflow, sequence)
- [ ] You know when to use branchless code for performance

### Self-Evaluation

Rate yourself 1-5:
1. Can you mentally compute bitwise AND, OR, XOR of two binary numbers? ___
2. Can you detect and fix precedence bugs in expressions? ___
3. Can you implement common bit manipulation tricks (set/clear/toggle/check bit)? ___
4. Can you explain short-circuit evaluation to a teammate? ___
5. Can you write clean, readable conditional logic without nesting > 3 levels? ___

**If all ratings are 4+, you are ready for Book 4: Loops.**

---

*End of Book 3: Operators & Control Flow*
*Next: Book 4 — Loops*
