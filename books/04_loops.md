# Book 4: Loops

## The Complete Guide to Iteration in C++

---

### **Target Level:** Beginner
### **Prerequisites:** Books 1–3
### **Learning Outcomes:**
By the end of this book, you will:
- Master every loop construct in C++ (for, while, do-while, range-based for)
- Understand loop mechanics at the assembly/CPU level
- Write efficient, correct loops for any scenario
- Handle nested loops, loop control statements, and edge cases
- Understand loop optimizations (unrolling, vectorization, invariant hoisting)
- Avoid common loop pitfalls and undefined behavior

---

## Chapter 1: Conceptual Foundations

### 1.1 What is Iteration?

Iteration (looping) is the basic mechanism for repeating a block of code. Every non-trivial program uses loops. Loops are how we:
- Process collections of data
- Perform repetitive calculations
- Wait for conditions to be met
- Implement algorithms

### 1.2 The Three Components of Every Loop

Every loop has three essential components:
1. **Initialization** — Setting up the starting state
2. **Condition** — Testing whether to continue
3. **Update** — Changing state to eventually terminate

If any component is missing or incorrect, you get either:
- An **infinite loop** (condition never becomes false)
- A **loop that never executes** (condition is false from the start)
- An **off-by-one error** (loop runs one too many or one too few times)

---

## Chapter 2: The `for` Loop

### 2.1 Syntax and Mechanics

```cpp
for (initialization; condition; update) {
    body;
}
```

**Execution flow:**
```
1. Execute initialization (once)
2. Evaluate condition
   → If false: EXIT loop
   → If true: 
       3. Execute body
       4. Execute update
       5. Go to step 2
```

**Example:**
```cpp
for (int i = 0; i < 5; i++) {
    std::cout << i << " ";
}
// Output: 0 1 2 3 4
```

**Line-by-line trace:**
```
i = 0: condition (0 < 5) = true → print 0 → i becomes 1
i = 1: condition (1 < 5) = true → print 1 → i becomes 2
i = 2: condition (2 < 5) = true → print 2 → i becomes 3
i = 3: condition (3 < 5) = true → print 3 → i becomes 4
i = 4: condition (4 < 5) = true → print 4 → i becomes 5
i = 5: condition (5 < 5) = false → EXIT
```

### 2.2 Scope of Loop Variables

```cpp
for (int i = 0; i < 5; i++) {
    // i is accessible here
}
// i is NOT accessible here — it was declared in the for statement

// If you need i after the loop:
int i;
for (i = 0; i < 5; i++) {
    // ...
}
std::cout << "Loop ran " << i << " times\n";  // i = 5
```

### 2.3 For Loop Variations

**Multiple variables:**
```cpp
for (int i = 0, j = 10; i < j; i++, j--) {
    std::cout << i << " " << j << "\n";
}
// Output: 0 10, 1 9, 2 8, 3 7, 4 6
```

**Empty components:**
```cpp
int i = 0;
for (; i < 5; ) {           // Initialization and update omitted
    std::cout << i++;
}

for (;;) {                    // Infinite loop — all three parts omitted
    break;                    // Must break or return to exit
}
```

**Counting backward:**
```cpp
for (int i = 10; i > 0; i--) {
    std::cout << i << " ";
}
// Output: 10 9 8 7 6 5 4 3 2 1

// DANGER with unsigned types:
for (unsigned int i = 10; i >= 0; i--) {
    // INFINITE LOOP! unsigned can never be < 0
    // When i = 0, i-- wraps to 4294967295
}

// Safe unsigned countdown:
for (unsigned int i = 10; i > 0; i--) {
    std::cout << i;
}
// Or:
for (unsigned int i = 11; i-- > 0; ) {
    std::cout << i;  // 10 9 8 7 6 5 4 3 2 1 0
}
```

**Stepping by values other than 1:**
```cpp
for (int i = 0; i < 100; i += 5) {
    std::cout << i << " ";  // 0 5 10 15 20 ... 95
}

for (double x = 0.0; x < 1.0; x += 0.1) {
    std::cout << x << " ";
    // WARNING: Floating-point accumulation error!
    // After 10 iterations, x might be 0.999999... or 1.0000001...
    // May execute 10 or 11 times depending on rounding!
}

// SAFE floating-point iteration:
int n = 10;
for (int i = 0; i < n; i++) {
    double x = static_cast<double>(i) / n;
    // x is computed fresh each time — no accumulation
}
```

### 2.4 Range-Based `for` Loop (C++11)

The modern way to iterate over containers and arrays:

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// By value (copies each element):
for (int n : numbers) {
    std::cout << n << " ";
}

// By reference (can modify elements):
for (int& n : numbers) {
    n *= 2;  // Doubles each element
}

// By const reference (read-only, no copies):
for (const int& n : numbers) {
    std::cout << n << " ";
}

// With auto (most idiomatic):
for (const auto& n : numbers) {
    std::cout << n << " ";
}
```

**Range-based for with different containers:**
```cpp
// Arrays:
int arr[] = {10, 20, 30};
for (auto x : arr) { std::cout << x; }

// Strings:
std::string s = "Hello";
for (char c : s) { std::cout << c << ' '; }

// Maps:
std::map<std::string, int> ages = {{"Alice", 30}, {"Bob", 25}};
for (const auto& [name, age] : ages) {    // C++17 structured bindings
    std::cout << name << ": " << age << "\n";
}

// Initializer lists:
for (auto x : {1, 2, 3, 4, 5}) {
    std::cout << x;
}
```

**How range-based for works internally:**
```cpp
// This:
for (auto x : container) { /* ... */ }

// Is equivalent to:
{
    auto&& __range = container;
    auto __begin = std::begin(__range);
    auto __end = std::end(__range);
    for (; __begin != __end; ++__begin) {
        auto x = *__begin;
        /* ... */
    }
}
```

### 2.5 At the Assembly Level

A simple for loop:
```cpp
int sum = 0;
for (int i = 0; i < 10; i++) {
    sum += i;
}
```

Generates approximately:
```asm
    xor  eax, eax          ; sum = 0
    xor  ecx, ecx          ; i = 0
.loop:
    add  eax, ecx          ; sum += i
    inc  ecx               ; i++
    cmp  ecx, 10           ; compare i with 10
    jl   .loop             ; if i < 10, jump to .loop
    ; eax now contains sum = 45
```

With optimization (`-O2`), the compiler may compute the result at compile time:
```asm
    mov  eax, 45           ; sum = 45 (compiler computed the loop!)
```

---

## Chapter 3: The `while` Loop

### 3.1 Syntax and Mechanics

```cpp
while (condition) {
    body;
}
```

**Execution flow:**
```
1. Evaluate condition
   → If false: EXIT loop
   → If true:
       2. Execute body
       3. Go to step 1
```

The condition is checked BEFORE each iteration. If false initially, the body never executes.

### 3.2 Common `while` Loop Patterns

**Reading input until sentinel:**
```cpp
int value;
std::cout << "Enter numbers (-1 to stop): ";
while (std::cin >> value && value != -1) {
    std::cout << "You entered: " << value << "\n";
}
```

**Processing until done:**
```cpp
int n = 12345;
while (n > 0) {
    int digit = n % 10;    // Get last digit
    std::cout << digit;
    n /= 10;               // Remove last digit
}
// Output: 54321 (digits in reverse)
```

**Iterating with pointers:**
```cpp
const char* str = "Hello";
while (*str != '\0') {
    std::cout << *str;
    str++;
}
// Output: Hello
```

**Event loop (game/GUI pattern):**
```cpp
bool running = true;
while (running) {
    processInput();
    updateGameState();
    render();
    
    if (userQuit()) {
        running = false;
    }
}
```

### 3.3 `while` vs `for` — When to Use Which

| Use `for` when... | Use `while` when... |
|-------------------|---------------------|
| You know the number of iterations | Number of iterations is unknown |
| Iterating over a range | Waiting for a condition |
| Counter-based iteration | Event-driven loops |
| You need init, condition, and update in one place | Initialization is complex or separate |

---

## Chapter 4: The `do-while` Loop

### 4.1 Syntax and Mechanics

```cpp
do {
    body;
} while (condition);   // Note the semicolon!
```

**Execution flow:**
```
1. Execute body
2. Evaluate condition
   → If true: Go to step 1
   → If false: EXIT loop
```

The body executes **at least once**, regardless of the condition.

### 4.2 When to Use `do-while`

**Input validation (most common use case):**
```cpp
int choice;
do {
    std::cout << "Enter 1-5: ";
    std::cin >> choice;
    if (choice < 1 || choice > 5) {
        std::cout << "Invalid! Try again.\n";
    }
} while (choice < 1 || choice > 5);
```

**Menu-driven programs:**
```cpp
int option;
do {
    std::cout << "\n=== Menu ===\n";
    std::cout << "1. Add\n";
    std::cout << "2. Delete\n";
    std::cout << "3. View\n";
    std::cout << "0. Exit\n";
    std::cout << "Choice: ";
    std::cin >> option;
    
    switch (option) {
        case 1: addItem(); break;
        case 2: deleteItem(); break;
        case 3: viewItems(); break;
        case 0: std::cout << "Goodbye!\n"; break;
        default: std::cout << "Invalid!\n";
    }
} while (option != 0);
```

**Number processing:**
```cpp
// Sum of digits
int n = 12345;
int sum = 0;
do {
    sum += n % 10;
    n /= 10;
} while (n > 0);
// sum = 15 (1+2+3+4+5)
// Works correctly even if n = 0 (processes at least once → sum = 0)
```

---

## Chapter 5: Nested Loops

### 5.1 How Nested Loops Work

```cpp
for (int i = 0; i < 3; i++) {           // Outer: runs 3 times
    for (int j = 0; j < 4; j++) {       // Inner: runs 4 times PER outer iteration
        std::cout << "(" << i << "," << j << ") ";
    }
    std::cout << "\n";
}
```

**Output:**
```
(0,0) (0,1) (0,2) (0,3)
(1,0) (1,1) (1,2) (1,3)
(2,0) (2,1) (2,2) (2,3)
```

Total iterations: $3 \times 4 = 12$

**Time complexity:** Nested loops multiply. Two nested loops each running $n$ times = $O(n^2)$.

### 5.2 Pattern Printing Examples

**Right triangle:**
```cpp
int n = 5;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        std::cout << "* ";
    }
    std::cout << "\n";
}
// *
// * *
// * * *
// * * * *
// * * * * *
```

**Pyramid:**
```cpp
int n = 5;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n - i; j++) std::cout << " ";   // Spaces
    for (int j = 1; j <= 2 * i - 1; j++) std::cout << "*"; // Stars
    std::cout << "\n";
}
//     *
//    ***
//   *****
//  *******
// *********
```

**Multiplication table:**
```cpp
for (int i = 1; i <= 10; i++) {
    for (int j = 1; j <= 10; j++) {
        std::cout << std::setw(4) << i * j;
    }
    std::cout << "\n";
}
```

### 5.3 Matrix Operations

```cpp
const int ROWS = 3, COLS = 3;
int matrix[ROWS][COLS] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Matrix traversal (row-major — cache-friendly):
for (int i = 0; i < ROWS; i++) {
    for (int j = 0; j < COLS; j++) {
        std::cout << matrix[i][j] << " ";
    }
    std::cout << "\n";
}

// Matrix transpose:
int transposed[COLS][ROWS];
for (int i = 0; i < ROWS; i++) {
    for (int j = 0; j < COLS; j++) {
        transposed[j][i] = matrix[i][j];
    }
}
```

---

## Chapter 6: Loop Control Statements

### 6.1 `break` — Exit the Loop Immediately

```cpp
for (int i = 0; i < 100; i++) {
    if (i == 5) break;      // Exits when i reaches 5
    std::cout << i << " ";
}
// Output: 0 1 2 3 4
```

**`break` in nested loops — only breaks the innermost loop:**
```cpp
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) break;     // Only breaks inner loop
        std::cout << "(" << i << "," << j << ") ";
    }
}
// Output: (0,0) (1,0) (2,0)
```

**Breaking out of multiple nested loops:**
```cpp
// Method 1: Use a flag
bool found = false;
for (int i = 0; i < 10 && !found; i++) {
    for (int j = 0; j < 10 && !found; j++) {
        if (matrix[i][j] == target) {
            std::cout << "Found at (" << i << "," << j << ")\n";
            found = true;
        }
    }
}

// Method 2: Extract to a function (preferred)
auto findInMatrix = [&](int target) -> std::pair<int, int> {
    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            if (matrix[i][j] == target) {
                return {i, j};
            }
        }
    }
    return {-1, -1};
};

// Method 3: goto (controversial but valid for breaking nested loops)
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (condition) goto done;
    }
}
done:;  // Label
```

### 6.2 `continue` — Skip to Next Iteration

```cpp
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;  // Skip even numbers
    std::cout << i << " ";
}
// Output: 1 3 5 7 9
```

**`continue` behavior differs by loop type:**

In `for` loop: continues to the **update expression**, then condition:
```cpp
for (int i = 0; i < 5; i++) {
    if (i == 2) continue;
    std::cout << i;
}
// Output: 0134 (i still increments when continue is hit)
```

In `while` loop: continues to the **condition** (NO automatic update!):
```cpp
int i = 0;
while (i < 5) {
    if (i == 2) {
        i++;          // MUST increment before continue!
        continue;
    }
    std::cout << i;
    i++;
}
// Without the i++ before continue, this would infinite loop!
```

### 6.3 `return` — Exit the Function (and the Loop)

```cpp
int findFirst(const std::vector<int>& v, int target) {
    for (int i = 0; i < v.size(); i++) {
        if (v[i] == target) return i;   // Immediately exits function
    }
    return -1;  // Not found
}
```

---

## Chapter 7: Infinite Loops

### 7.1 Intentional Infinite Loops

```cpp
// Style 1: for(;;)
for (;;) {
    // Server main loop, game loop, etc.
    if (shouldExit) break;
}

// Style 2: while(true)
while (true) {
    // Same purpose
    if (shouldExit) break;
}

// Style 3: do-while
do {
    // Less common
} while (true);
```

### 7.2 Accidental Infinite Loops

```cpp
// Bug 1: Missing update
int i = 0;
while (i < 10) {
    std::cout << i;
    // Forgot i++ — infinite loop!
}

// Bug 2: Wrong condition
for (int i = 10; i > 0; i++) {  // i++ instead of i-- — infinite loop!
    std::cout << i;
}

// Bug 3: Unsigned underflow
for (unsigned int i = 10; i >= 0; i--) {
    // unsigned can never be < 0 — infinite loop!
}

// Bug 4: Floating-point comparison
for (double x = 0.0; x != 1.0; x += 0.1) {
    // x may never exactly equal 1.0 — potential infinite loop!
}
```

### 7.3 Real-World Infinite Loops

**Game loop:**
```cpp
while (window.isOpen()) {
    auto dt = clock.restart();
    handleInput();
    update(dt);
    render();
}
```

**Server loop:**
```cpp
while (server.isRunning()) {
    auto connection = server.acceptConnection();
    handleRequest(connection);
}
```

**Embedded system main loop:**
```cpp
int main() {
    initHardware();
    
    while (true) {     // Embedded systems run forever
        readSensors();
        processData();
        updateOutputs();
        watchdogReset();  // Prevent hardware watchdog reset
    }
    // Never reaches here
}
```

---

## Chapter 8: Advanced Loop Techniques

### 8.1 Loop Optimization — What the Compiler Does

**Loop Invariant Code Motion (LICM):**
```cpp
// Before optimization:
for (int i = 0; i < n; i++) {
    int len = strlen(str);  // Computed every iteration!
    if (len > 0) { /* ... */ }
}

// After LICM (compiler moves invariant out of loop):
int len = strlen(str);      // Computed once
for (int i = 0; i < n; i++) {
    if (len > 0) { /* ... */ }
}
```

**Loop Unrolling:**
```cpp
// Before:
for (int i = 0; i < 100; i++) {
    sum += arr[i];
}

// After unrolling (compiler generates):
for (int i = 0; i < 100; i += 4) {
    sum += arr[i];
    sum += arr[i+1];
    sum += arr[i+2];
    sum += arr[i+3];
}
// Fewer branch instructions, better CPU pipeline utilization
```

**Loop Vectorization (SIMD):**
```cpp
// The compiler may use SIMD instructions (SSE, AVX):
for (int i = 0; i < n; i++) {
    a[i] = b[i] + c[i];
}
// With AVX2, processes 8 floats or 4 doubles simultaneously
```

### 8.2 Cache-Friendly Iteration

```cpp
const int N = 1000;
int matrix[N][N];

// FAST — row-major traversal (cache-friendly):
for (int i = 0; i < N; i++) {
    for (int j = 0; j < N; j++) {
        matrix[i][j] = 0;  // Sequential memory access
    }
}

// SLOW — column-major traversal (cache-unfriendly):
for (int j = 0; j < N; j++) {
    for (int i = 0; i < N; i++) {
        matrix[i][j] = 0;  // Jumps N*sizeof(int) bytes each access
    }
}
// The cache-friendly version can be 10-100x faster for large matrices!
```

**Why?** C++ stores 2D arrays in row-major order. Accessing `matrix[i][0]`, `matrix[i][1]`, `matrix[i][2]` accesses consecutive memory locations, which are loaded into the CPU cache together. Accessing `matrix[0][j]`, `matrix[1][j]`, `matrix[2][j]` jumps across cache lines.

### 8.3 Sentinel-Controlled Loops

```cpp
// Reading input until end-of-file:
int value;
while (std::cin >> value) {
    // Process value
    // std::cin >> value returns false at EOF or on error
}

// Processing a null-terminated array:
int* arr = getData();
while (*arr != -1) {   // -1 is the sentinel
    process(*arr);
    arr++;
}
```

### 8.4 Loop and a Half Pattern

Sometimes the exit condition is in the middle:
```cpp
while (true) {
    std::string line;
    std::getline(std::cin, line);
    
    if (line.empty()) break;    // Exit condition in the middle
    
    processLine(line);
}
```

---

## Chapter 9: Modern C++ Loop Features

### 9.1 Range-Based For with Structured Bindings (C++17)

```cpp
std::map<std::string, int> scores = {
    {"Alice", 95},
    {"Bob", 87},
    {"Charlie", 92}
};

for (const auto& [name, score] : scores) {
    std::cout << name << ": " << score << "\n";
}
```

### 9.2 `std::ranges` and Views (C++20)

```cpp
#include <ranges>
#include <vector>

std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Filter and transform without creating intermediate containers:
for (int n : numbers 
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * n; })) {
    std::cout << n << " ";
}
// Output: 4 16 36 64 100

// Generate a range:
for (int i : std::views::iota(1, 11)) {    // 1 to 10
    std::cout << i << " ";
}

// Take first N:
for (int i : std::views::iota(1) | std::views::take(5)) {
    std::cout << i << " ";  // 1 2 3 4 5
}
```

### 9.3 `constexpr` Loops (C++14/17)

```cpp
constexpr int computeFactorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

constexpr int fact10 = computeFactorial(10);  // Computed at compile time!
// fact10 = 3628800 — no runtime cost
```

### 9.4 Parallel Loops with `std::for_each` (C++17)

```cpp
#include <algorithm>
#include <execution>
#include <vector>

std::vector<int> data(1'000'000);

// Sequential:
std::for_each(data.begin(), data.end(), [](int& x) { x *= 2; });

// Parallel:
std::for_each(std::execution::par, data.begin(), data.end(), 
              [](int& x) { x *= 2; });

// Parallel + vectorized:
std::for_each(std::execution::par_unseq, data.begin(), data.end(),
              [](int& x) { x *= 2; });
```

---

## Chapter 10: Interview & Competitive Programming Section

### 10.1 Common Interview Questions

**Q1: What is the difference between `while` and `do-while`?**
A: `while` checks the condition before each iteration (body may never execute). `do-while` checks after (body always executes at least once).

**Q2: How do you break out of nested loops?**
A: (1) Use a flag variable, (2) Extract to a function and use `return`, (3) Use `goto` (rare but valid for this specific case).

**Q3: What is the time complexity of two nested loops each running N times?**
A: $O(N^2)$. Three nested loops would be $O(N^3)$.

**Q4: Why should you use pre-increment (`++i`) over post-increment (`i++`) in loop update?**
A: For primitive types, no performance difference. For iterators and complex objects, pre-increment avoids creating a temporary copy.

**Q5: How can a loop run exactly 0 times?**
A: Using `while` or `for` with a condition that is false from the start (e.g., `for (int i = 10; i < 5; i++)`). `do-while` always runs at least once.

### 10.2 Classic Loop Problems

**Fibonacci series:**
```cpp
int n = 10;
int a = 0, b = 1;
for (int i = 0; i < n; i++) {
    std::cout << a << " ";
    int temp = a + b;
    a = b;
    b = temp;
}
// Output: 0 1 1 2 3 5 8 13 21 34
```

**Reverse a number:**
```cpp
int num = 12345, reversed = 0;
while (num > 0) {
    reversed = reversed * 10 + num % 10;
    num /= 10;
}
// reversed = 54321
```

**Check palindrome number:**
```cpp
int num = 12321;
int original = num, reversed = 0;
while (num > 0) {
    reversed = reversed * 10 + num % 10;
    num /= 10;
}
bool isPalindrome = (original == reversed);  // true
```

**GCD (Euclidean algorithm):**
```cpp
int gcd(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}
```

**Prime check:**
```cpp
bool isPrime(int n) {
    if (n < 2) return false;
    if (n < 4) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

**Sieve of Eratosthenes:**
```cpp
std::vector<bool> sieve(int limit) {
    std::vector<bool> is_prime(limit + 1, true);
    is_prime[0] = is_prime[1] = false;
    for (int i = 2; i * i <= limit; i++) {
        if (is_prime[i]) {
            for (int j = i * i; j <= limit; j += i) {
                is_prime[j] = false;
            }
        }
    }
    return is_prime;
}
```

### 10.3 Tricky Output Questions

```cpp
// Q1: What is the output?
for (int i = 0; i < 5; i++);  // Note the semicolon!
    std::cout << i;            // Error: i is out of scope
// This is a common bug — the semicolon makes the loop body empty

// Q2: What is the output?
int i = 0;
while (i < 5)
    std::cout << i;
    i++;                       // NOT inside the loop! Only the first statement is the body
// Infinite loop printing 00000...

// Q3: What is the output?
for (int i = 0; i < 10; i++) {
    if (i == 5) continue;
    if (i == 8) break;
    std::cout << i;
}
// Output: 01234 67
```

---

## Chapter 11: Practice Section

### 11.1 Beginner Exercises

1. Print numbers 1 to 100. Replace multiples of 3 with "Fizz", multiples of 5 with "Buzz", and multiples of both with "FizzBuzz".
2. Calculate the factorial of N using a loop.
3. Print the first N terms of the Fibonacci sequence.
4. Check if a number is a palindrome.
5. Print all prime numbers up to N.

### 11.2 Intermediate Exercises

6. Print Pascal's triangle of height N.
7. Implement binary search using a while loop.
8. Simulate the Collatz conjecture: starting from any positive integer N, if even divide by 2, if odd multiply by 3 and add 1. Count steps until reaching 1.
9. Implement the Sieve of Eratosthenes for finding all primes up to N.
10. Write a program that computes power(base, exponent) using repeated squaring (fast exponentiation).

### 11.3 Advanced Exercises

11. Compare the performance of row-major vs column-major 2D array traversal for a 10000×10000 matrix.
12. Implement a simple string tokenizer using loops (split a string by a delimiter).
13. Write an optimized matrix multiplication using loop tiling for cache efficiency.
14. Implement Newton's method for finding square roots using a while loop with convergence checking.

---

## Chapter 12: Mini Projects

### Project 1: Number Guessing Game (Beginner)
- Computer picks a random number 1-100
- User guesses, program says "higher" or "lower"
- Count attempts, provide rating based on number of guesses
- Use do-while for the main game loop, nested while for input validation

### Project 2: Cellular Automaton (Rule 30/110) (Intermediate)
- Implement a 1D cellular automaton
- Display evolution over N generations using nested loops
- Visualize with ASCII characters
- Allow user to select different rules

### Project 3: Matrix Calculator (Advanced)
- Support addition, subtraction, multiplication of NxM matrices
- Implement determinant calculation
- Implement matrix inverse using Gauss-Jordan elimination
- All operations use nested loops with proper cache-friendly traversal

---

## Chapter 13: Summary & Mastery Checklist

- [ ] You can write for, while, and do-while loops correctly
- [ ] You understand range-based for loops and when to use by-value vs by-reference
- [ ] You can trace loop execution and predict output precisely
- [ ] You understand nested loops and their time complexity
- [ ] You can use break, continue, and return correctly in loops
- [ ] You know the unsigned count-down pitfall
- [ ] You understand floating-point loop iteration issues
- [ ] You know the difference between cache-friendly and cache-unfriendly loops
- [ ] You can use C++17 structured bindings in range-based for
- [ ] You can use C++20 ranges and views for loop transformations

**If all ratings are 4+, you are ready for Book 5: Functions.**

---

*End of Book 4: Loops*
*Next: Book 5 — Functions*
