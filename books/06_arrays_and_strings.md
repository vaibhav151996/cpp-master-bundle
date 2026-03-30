# Book 6: Arrays & Strings

## The Complete Guide to Sequential Data Storage in C++

---

### **Target Level:** Intermediate
### **Prerequisites:** Books 1–5
### **Learning Outcomes:**
By the end of this book, you will:
- Master C-style arrays and their memory layout
- Understand multidimensional arrays and pointer decay
- Know C-strings inside out (null-terminated character arrays)
- Master `std::string` and all its operations
- Understand `std::string_view` (C++17)
- Know `std::array` vs C-arrays vs `std::vector`
- Handle Unicode and encoding considerations

---

## Chapter 1: C-Style Arrays

### 1.1 Array Fundamentals

An array is a **contiguous block of memory** storing elements of the same type.

```cpp
int arr[5] = {10, 20, 30, 40, 50};
```

**Memory layout:**
```
Address:  0x1000  0x1004  0x1008  0x100C  0x1010
Value:    [  10  ][  20  ][  30  ][  40  ][  50  ]
Index:       0       1       2       3       4
```

Each `int` is 4 bytes. Element at index `i` is at address: `base_address + i * sizeof(int)`.

### 1.2 Declaration and Initialization

```cpp
// Declaration with size:
int arr[5];                          // Uninitialized (contains garbage values)

// With initializer list:
int arr[5] = {1, 2, 3, 4, 5};      // All elements specified
int arr[5] = {1, 2};                // Partial: {1, 2, 0, 0, 0}
int arr[5] = {};                     // All zeros: {0, 0, 0, 0, 0}
int arr[5] = {0};                    // All zeros (first is 0, rest default to 0)

// Size deduced from initializer:
int arr[] = {1, 2, 3};              // Size is 3

// C++17 Class Template Argument Deduction (CTAD) with std::array:
std::array arr = {1, 2, 3, 4, 5};  // std::array<int, 5>

// Designated initializers (C++20):
int arr[5] = {[2] = 30, [4] = 50};  // {0, 0, 30, 0, 50}
```

### 1.3 Accessing Elements

```cpp
int arr[5] = {10, 20, 30, 40, 50};

arr[0];    // 10 (first element)
arr[4];    // 50 (last element)
arr[5];    // UNDEFINED BEHAVIOR — out of bounds!
arr[-1];   // UNDEFINED BEHAVIOR — negative index!

// Arrays don't check bounds! This is a major source of bugs and security vulnerabilities.
```

### 1.4 Array Decay — The Most Important Concept

When you pass an array to a function, it **decays** to a pointer to its first element:

```cpp
void printArray(int arr[], int size) {
    // arr is actually int* — NOT an array!
    // sizeof(arr) = sizeof(int*) = 8 (on 64-bit) — NOT the array size!
    for (int i = 0; i < size; i++) {
        std::cout << arr[i] << " ";
    }
}

int numbers[5] = {1, 2, 3, 4, 5};
printArray(numbers, 5);  // numbers decays to int*

// These are ALL equivalent function signatures:
void f(int arr[]);       // Looks like array, but is actually int*
void f(int arr[100]);    // The 100 is IGNORED — still just int*
void f(int* arr);        // The truth: it's a pointer
```

**To preserve size information, use `std::array`, `std::span`, or templates:**

```cpp
// Template approach — preserves size:
template<size_t N>
void printArray(int (&arr)[N]) {
    for (size_t i = 0; i < N; i++) {
        std::cout << arr[i] << " ";
    }
}

// std::span (C++20) — best modern approach:
void printArray(std::span<int> arr) {
    for (int x : arr) {
        std::cout << x << " ";
    }
}
```

### 1.5 Common Array Operations

```cpp
int arr[5] = {5, 2, 8, 1, 9};

// Find maximum:
int maxVal = arr[0];
for (int i = 1; i < 5; i++) {
    if (arr[i] > maxVal) maxVal = arr[i];
}

// Reverse:
for (int i = 0; i < 5 / 2; i++) {
    std::swap(arr[i], arr[4 - i]);
}

// Copy:
int copy[5];
std::copy(std::begin(arr), std::end(arr), std::begin(copy));

// Sort:
std::sort(std::begin(arr), std::end(arr));

// Search:
auto it = std::find(std::begin(arr), std::end(arr), 8);
if (it != std::end(arr)) {
    std::cout << "Found at index " << (it - std::begin(arr));
}
```

---

## Chapter 2: Multidimensional Arrays

### 2.1 2D Arrays

```cpp
int matrix[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};

// Access element at row i, column j:
matrix[1][2] = 7;  // Row 1, Column 2

// Memory layout (row-major — all of row 0, then row 1, then row 2):
// [1][2][3][4] [5][6][7][8] [9][10][11][12]
// ← Row 0  → ← Row 1   → ← Row 2      →
```

**Address calculation:** Element at `[i][j]` is at: `base + (i * num_cols + j) * sizeof(int)`

### 2.2 Traversal

```cpp
// Row-major traversal (cache-friendly):
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 4; j++) {
        std::cout << matrix[i][j] << " ";
    }
    std::cout << "\n";
}
```

### 2.3 Passing 2D Arrays to Functions

```cpp
// Must specify all dimensions except the first:
void print(int arr[][4], int rows) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < 4; j++) {
            std::cout << arr[i][j] << " ";
        }
        std::cout << "\n";
    }
}

// Using templates (flexible):
template<size_t R, size_t C>
void print(int (&arr)[R][C]) {
    for (size_t i = 0; i < R; i++) {
        for (size_t j = 0; j < C; j++) {
            std::cout << arr[i][j] << " ";
        }
        std::cout << "\n";
    }
}
```

---

## Chapter 3: `std::array` (C++11)

A fixed-size array with the same performance as C-arrays but with STL container interface:

```cpp
#include <array>

std::array<int, 5> arr = {1, 2, 3, 4, 5};

arr[0] = 10;           // No bounds checking
arr.at(0) = 10;        // WITH bounds checking (throws std::out_of_range)

arr.size();            // 5 (knows its own size!)
arr.empty();           // false
arr.front();           // First element
arr.back();            // Last element
arr.data();            // Pointer to underlying array
arr.fill(0);           // Set all elements to 0

// Works with STL algorithms:
std::sort(arr.begin(), arr.end());

// Can be returned from functions (unlike C-arrays):
std::array<int, 3> getValues() {
    return {1, 2, 3};
}

// Can be compared:
std::array<int, 3> a = {1, 2, 3};
std::array<int, 3> b = {1, 2, 3};
if (a == b) { /* true! */ }
```

**`std::array` vs C-array:**
| Feature | C-array | `std::array` |
|---------|---------|--------------|
| Knows its size | No | Yes (`.size()`) |
| Bounds checking | No | Yes (`.at()`) |
| Assignable/copyable | No | Yes |
| Returnable from functions | No | Yes |
| Works with STL | Partially | Fully |
| Performance | Same | Same |
| Stack/heap | Both | Both |

**Always prefer `std::array` over C-arrays in modern C++.**

---

## Chapter 4: C-Strings

### 4.1 What Are C-Strings?

A C-string is a **null-terminated array of characters**:

```cpp
char str[] = "Hello";
// Actually stored as: {'H', 'e', 'l', 'l', 'o', '\0'}
// Size: 6 bytes (5 characters + null terminator)
```

**The null terminator `\0` is critical** — it tells functions where the string ends.

```
Memory: [H][e][l][l][o][\0]
Index:   0   1  2  3  4   5
```

### 4.2 C-String Operations

```cpp
#include <cstring>

char s1[20] = "Hello";
char s2[20] = "World";

strlen(s1);           // 5 (does NOT count \0)
strcmp(s1, s2);        // Negative (H < W lexicographically)
strcpy(s1, s2);       // Copies s2 into s1: s1 = "World"
strcat(s1, "!");      // Appends: s1 = "World!"
strchr(s1, 'o');      // Returns pointer to first 'o'
strstr(s1, "orl");    // Returns pointer to "orl" substring
```

### 4.3 C-String Dangers

```cpp
// Buffer overflow:
char buf[5];
strcpy(buf, "Hello World!");  // BUFFER OVERFLOW — UB!
// "Hello World!" is 13 bytes but buf is only 5

// Use safe versions:
strncpy(buf, "Hello World!", sizeof(buf) - 1);
buf[sizeof(buf) - 1] = '\0';  // Ensure null termination

// Missing null terminator:
char bad[5] = {'H', 'e', 'l', 'l', 'o'};  // No \0!
strlen(bad);   // UNDEFINED! Reads past the array looking for \0

// Comparing with ==:
char a[] = "hello";
char b[] = "hello";
if (a == b) { }         // WRONG! Compares POINTERS, not content
if (strcmp(a, b) == 0) { }  // CORRECT — compares character by character
```

**Modern guideline:** Avoid C-strings entirely. Use `std::string`.

---

## Chapter 5: `std::string` Class

### 5.1 The Modern Way

`std::string` manages its own memory, knows its length, and is safe:

```cpp
#include <string>

std::string s = "Hello, World!";
std::string empty;                     // Empty string ""
std::string repeated(10, '*');         // "**********"
std::string copy = s;                  // Deep copy
std::string moved = std::move(s);      // Move (s is now empty)
```

### 5.2 String Operations

```cpp
std::string s = "Hello, World!";

// Information:
s.length();       // 13 (same as s.size())
s.empty();        // false
s.capacity();     // >= 13 (allocated memory)

// Access:
s[0];             // 'H' (no bounds check)
s.at(0);          // 'H' (with bounds check — throws on error)
s.front();        // 'H'
s.back();         // '!'
s.c_str();        // Returns const char* (null-terminated)
s.data();         // Returns char* (C++17: non-const)

// Modification:
s += " Bye!";              // Append
s.append(" End");          // Append
s.insert(5, " Dear");      // Insert at position
s.erase(5, 5);             // Erase 5 chars starting at position 5
s.replace(0, 5, "Hi");     // Replace first 5 chars with "Hi"
s.clear();                 // Empty the string
s.push_back('!');          // Add char at end
s.pop_back();              // Remove last char

// Search:
s = "Hello, World!";
s.find("World");           // Returns 7 (position of "World")
s.find("xyz");             // Returns std::string::npos (not found)
s.rfind("l");              // 10 (last occurrence)
s.find_first_of("aeiou");  // 1 (first vowel)
s.find_last_of("lo");      // 10

// Substring:
s.substr(7, 5);            // "World" (from pos 7, length 5)
s.substr(7);               // "World!" (from pos 7 to end)

// Comparison:
std::string a = "apple", b = "banana";
if (a == b) { }            // Direct comparison!
if (a < b) { }             // Lexicographic comparison
a.compare(b);              // Returns <0, 0, or >0

// Conversion:
int n = std::stoi("42");           // String to int
double d = std::stod("3.14");     // String to double
std::string ns = std::to_string(42);  // Int to string
```

### 5.3 String Memory — Small String Optimization (SSO)

Most `std::string` implementations use SSO:

```cpp
std::string short_str = "Hi";        // Stored INSIDE the string object (no heap allocation)
std::string long_str = "This is a longer string that exceeds SSO";  // Heap allocated
```

Typical SSO buffer: 15-22 characters (implementation-dependent).

```
Short string (SSO — in-place):
┌──────────────────────────────────────┐
│ String Object (stack)                │
│ ┌──────────────────────────────────┐ │
│ │ [H][i][\0][...unused buffer...]  │ │  ← Data stored inside object
│ │ size: 2  │  capacity: 15         │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘

Long string (heap-allocated):
┌──────────────────────────────────────┐         ┌──────────────────────────┐
│ String Object (stack)                │         │ Heap memory              │
│ ┌──────────────────────────────────┐ │    ┌───→│ [T][h][i][s][ ][i][s]...│
│ │ pointer: ─────────────────────────┼─────┘    └──────────────────────────┘
│ │ size: 40  │  capacity: 47        │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

### 5.4 String Iterators and Range-Based For

```cpp
std::string s = "Hello";

// Range-based for:
for (char c : s) {
    std::cout << c << " ";
}

// Modify characters:
for (char& c : s) {
    c = std::toupper(c);
}
// s is now "HELLO"

// Iterators:
for (auto it = s.begin(); it != s.end(); ++it) {
    *it = std::tolower(*it);
}

// Reverse iteration:
for (auto it = s.rbegin(); it != s.rend(); ++it) {
    std::cout << *it;
}
// Output: olleH
```

---

## Chapter 6: `std::string_view` (C++17)

A **non-owning reference** to a string. Zero-copy, lightweight.

```cpp
#include <string_view>

void greet(std::string_view name) {    // Does NOT copy the string
    std::cout << "Hello, " << name << "!\n";
}

greet("World");                        // Works with string literals
greet(std::string("World"));           // Works with std::string
std::string s = "World";
greet(s);                              // Works with std::string variable

// String view operations (read-only — cannot modify):
std::string_view sv = "Hello, World!";
sv.substr(7, 5);         // "World" — returns another string_view (no copy!)
sv.remove_prefix(7);     // sv is now "World!"
sv.remove_suffix(1);     // sv is now "World"
sv.find("or");           // 1
sv.size();               // 5
```

**When to use `std::string_view`:**
- Function parameters that only READ a string
- Parsing and tokenizing (create views into the source string)
- When you need a substring without copying

**When NOT to use:**
- When you need to store the string (string_view doesn't own memory)
- When the source string might be destroyed before the view

```cpp
// DANGER:
std::string_view sv;
{
    std::string temp = "Hello";
    sv = temp;        // sv references temp's memory
}
// temp destroyed! sv is now a DANGLING reference!
std::cout << sv;  // UNDEFINED BEHAVIOR
```

---

## Chapter 7: Advanced String Topics

### 7.1 String Concatenation Performance

```cpp
// BAD — O(n²) due to repeated allocation:
std::string result;
for (int i = 0; i < 10000; i++) {
    result += "word ";  // May reallocate and copy each time!
}

// BETTER — reserve space:
std::string result;
result.reserve(50000);  // Pre-allocate
for (int i = 0; i < 10000; i++) {
    result += "word ";
}

// BEST — use stringstream or std::format:
std::ostringstream oss;
for (int i = 0; i < 10000; i++) {
    oss << "word ";
}
std::string result = oss.str();
```

### 7.2 String Formatting

```cpp
// C-style (printf):
printf("Name: %s, Age: %d, GPA: %.2f\n", name, age, gpa);

// C++ streams:
std::cout << "Name: " << name << ", Age: " << age << "\n";

// std::format (C++20):
std::string s = std::format("Name: {}, Age: {}, GPA: {:.2f}", name, age, gpa);

// std::print (C++23):
std::print("Name: {}, Age: {}\n", name, age);
```

### 7.3 Regular Expressions

```cpp
#include <regex>

std::string text = "Email: user@example.com, Phone: 123-456-7890";
std::regex email_pattern(R"(\w+@\w+\.\w+)");
std::regex phone_pattern(R"(\d{3}-\d{3}-\d{4})");

std::smatch match;
if (std::regex_search(text, match, email_pattern)) {
    std::cout << "Email: " << match[0] << "\n";  // user@example.com
}

// Replace:
std::string result = std::regex_replace(text, email_pattern, "[REDACTED]");
```

---

## Chapter 8: Interview & Competitive Programming Section

### Common Questions

**Q1: What's the difference between `char str[]` and `char* str`?**
A: `char str[] = "hello"` creates a modifiable array of 6 characters. `char* str = "hello"` creates a pointer to a read-only string literal — modifying it is UB.

**Q2: What is `std::string::npos`?**
A: It's the maximum value of `size_t` (typically 18446744073709551615). Returned by `find()` when the target is not found.

**Q3: How does `std::string` manage memory?**
A: It uses dynamic allocation on the heap (via an allocator). Short strings may use SSO (Small String Optimization) to store data inline. It handles allocation, reallocation, and deallocation automatically.

### Classic String Problems

```cpp
// Reverse a string:
std::string reverse(std::string s) {
    std::reverse(s.begin(), s.end());
    return s;
}

// Check palindrome:
bool isPalindrome(const std::string& s) {
    int left = 0, right = s.size() - 1;
    while (left < right) {
        if (s[left++] != s[right--]) return false;
    }
    return true;
}

// Count character frequency:
std::unordered_map<char, int> charFreq(const std::string& s) {
    std::unordered_map<char, int> freq;
    for (char c : s) freq[c]++;
    return freq;
}

// First non-repeating character:
char firstUnique(const std::string& s) {
    std::array<int, 256> count = {};
    for (char c : s) count[c]++;
    for (char c : s) if (count[c] == 1) return c;
    return '\0';
}

// String compression: "aabcccccaaa" → "a2b1c5a3"
std::string compress(const std::string& s) {
    std::string result;
    int i = 0;
    while (i < s.size()) {
        char c = s[i];
        int count = 0;
        while (i < s.size() && s[i] == c) { i++; count++; }
        result += c;
        result += std::to_string(count);
    }
    return result.size() < s.size() ? result : s;
}
```

---

## Chapter 9: Practice Section

### Beginner
1. Reverse a string without using library functions.
2. Check if two strings are anagrams.
3. Count vowels, consonants, digits, and spaces in a string.
4. Convert a string to uppercase/lowercase.

### Intermediate
5. Implement your own `strlen`, `strcpy`, `strcmp`, `strcat`.
6. Implement a simple tokenizer (split string by delimiter).
7. Find the longest common prefix among an array of strings.
8. Implement Caesar cipher encryption/decryption.

### Advanced
9. Implement the KMP string matching algorithm.
10. Build a simple text editor line buffer using `std::vector<std::string>`.
11. Implement a trie (prefix tree) for autocomplete suggestions.

---

## Chapter 10: Mini Projects

### Project 1: Word Counter (Beginner)
Read a text file, count word frequencies, display sorted by frequency.

### Project 2: Text Search Tool (Intermediate)
Grep-like tool: search for patterns in files, highlight matches, show line numbers.

### Project 3: JSON Parser (Advanced)
Parse a JSON string into a tree structure. Handle strings, numbers, booleans, arrays, objects.

---

## Chapter 11: Summary & Mastery Checklist

- [ ] You understand C-array memory layout, decay, and limitations
- [ ] You can use `std::array` and know why it's preferred
- [ ] You understand null-terminated C-strings and their dangers
- [ ] You can use `std::string` fluently with all common operations
- [ ] You know `std::string_view` and when to use it
- [ ] You understand SSO (Small String Optimization)
- [ ] You can solve common string manipulation problems efficiently
- [ ] You understand the performance implications of string concatenation

**If all ratings are 4+, you are ready for Book 7: Pointers & References.**

---

*End of Book 6: Arrays & Strings*
*Next: Book 7 — Pointers & References*
