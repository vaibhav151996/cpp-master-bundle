# Book 12: Standard Template Library (STL)

## The Complete Guide to C++ Containers, Iterators, and Algorithms

---

### **Target Level:** Upper Intermediate → Advanced
### **Prerequisites:** Books 1–11
### **Learning Outcomes:**
By the end of this book, you will:
- Master all major STL containers and their performance characteristics
- Understand iterators and their categories
- Know all essential STL algorithms
- Choose the right container for any problem
- Understand internal implementations (hash tables, red-black trees, dynamic arrays)

---

## Chapter 1: STL Architecture

The STL has three pillars:
1. **Containers** — Store data (vector, map, set, etc.)
2. **Iterators** — Access data (pointers into containers)
3. **Algorithms** — Process data (sort, find, transform, etc.)

They work together: algorithms operate on ranges defined by iterators, which point into containers.

---

## Chapter 2: Sequence Containers

### 2.1 `std::vector` — Dynamic Array

The most-used container. Contiguous memory, O(1) random access:

```cpp
#include <vector>

std::vector<int> v;
v.push_back(10);         // Add to end: O(1) amortized
v.push_back(20);
v.push_back(30);

v[0];                     // Access: O(1)
v.at(0);                  // Bounds-checked access
v.front();                // First element
v.back();                 // Last element
v.size();                 // Number of elements
v.capacity();             // Allocated space
v.empty();                // Is it empty?
v.pop_back();             // Remove last: O(1)
v.insert(v.begin() + 1, 15);  // Insert: O(n)
v.erase(v.begin() + 1);       // Erase: O(n)
v.clear();                     // Remove all
v.reserve(100);                // Pre-allocate space
v.shrink_to_fit();             // Release excess memory
v.data();                      // Raw pointer to underlying array

// Initialization:
std::vector<int> v1 = {1, 2, 3, 4, 5};
std::vector<int> v2(10, 0);       // 10 zeros
std::vector<int> v3(v1.begin(), v1.end());  // Copy from range

// Growth strategy: when capacity is exceeded, vector allocates
// a new buffer (typically 2x size), copies elements, frees old buffer.
// This is why push_back is O(1) AMORTIZED.
```

### 2.2 `std::deque` — Double-Ended Queue

O(1) insertion/removal at both ends. Memory is NOT contiguous:

```cpp
#include <deque>

std::deque<int> dq;
dq.push_back(10);    // Add to back: O(1)
dq.push_front(5);    // Add to front: O(1) — vector can't do this efficiently!
dq.pop_back();       // Remove from back: O(1)
dq.pop_front();      // Remove from front: O(1)
dq[0];               // Random access: O(1)
```

### 2.3 `std::list` — Doubly Linked List

O(1) insertion/removal anywhere (given an iterator). No random access:

```cpp
#include <list>

std::list<int> lst = {1, 2, 3, 4, 5};
lst.push_back(6);
lst.push_front(0);

auto it = std::find(lst.begin(), lst.end(), 3);
lst.insert(it, 25);      // O(1) insertion before '3'
lst.erase(it);            // O(1) removal

lst.sort();               // O(n log n) — list has its own sort
lst.unique();             // Remove consecutive duplicates
lst.reverse();            // Reverse in O(n)
lst.merge(other_list);    // Merge two sorted lists
lst.splice(pos, other);   // Move elements from another list
```

### 2.4 `std::forward_list` — Singly Linked List (C++11)

More memory-efficient than `list`. Only forward traversal.

### 2.5 Container Comparison

| Container | Access | Insert/Delete Front | Insert/Delete Back | Insert/Delete Middle | Memory |
|-----------|--------|--------------------|--------------------|---------------------|--------|
| `vector` | O(1) | O(n) | O(1)* | O(n) | Contiguous |
| `deque` | O(1) | O(1) | O(1) | O(n) | Segmented |
| `list` | O(n) | O(1) | O(1) | O(1)† | Scattered |
| `forward_list` | O(n) | O(1) | O(n) | O(1)† | Scattered |

*amortized †given iterator

---

## Chapter 3: Associative Containers

### 3.1 `std::set` — Sorted Unique Elements (Red-Black Tree)

```cpp
#include <set>

std::set<int> s = {5, 3, 1, 4, 2, 3};  // {1, 2, 3, 4, 5} — sorted, no dupes

s.insert(6);           // O(log n)
s.erase(3);            // O(log n)
s.count(4);            // 1 (element exists) or 0
s.find(4);             // Returns iterator (O(log n))
s.contains(4);         // C++20: true/false
s.lower_bound(3);      // Iterator to first element >= 3
s.upper_bound(3);      // Iterator to first element > 3
```

### 3.2 `std::map` — Sorted Key-Value Pairs (Red-Black Tree)

```cpp
#include <map>

std::map<std::string, int> ages;
ages["Alice"] = 30;              // Insert/update: O(log n)
ages["Bob"] = 25;
ages.insert({"Charlie", 35});

ages["Alice"];                    // Access: O(log n) — creates entry if missing!
ages.at("Alice");                 // Access: O(log n) — throws if missing

// Iteration (sorted by key):
for (const auto& [name, age] : ages) {
    std::cout << name << ": " << age << "\n";
}

// Check existence:
if (ages.count("Alice")) { /* exists */ }
if (ages.contains("Alice")) { /* C++20 */ }

// Find:
auto it = ages.find("Alice");
if (it != ages.end()) {
    std::cout << it->second;  // 30
}

// Erase:
ages.erase("Alice");

// Insert or assign (C++17):
ages.insert_or_assign("Alice", 31);

// Try emplace (C++17):
ages.try_emplace("Dave", 28);  // Only inserts if key doesn't exist
```

### 3.3 `std::multiset` and `std::multimap`

Allow duplicate keys:
```cpp
std::multiset<int> ms = {1, 2, 2, 3, 3, 3};
ms.count(3);  // 3

std::multimap<std::string, int> mm;
mm.insert({"Alice", 90});
mm.insert({"Alice", 85});  // Duplicate key allowed
```

---

## Chapter 4: Unordered Containers (Hash Tables)

### 4.1 `std::unordered_map`

O(1) average access (hash table). NOT sorted:

```cpp
#include <unordered_map>

std::unordered_map<std::string, int> um;
um["Alice"] = 30;     // O(1) average
um["Bob"] = 25;

um.count("Alice");     // O(1) average
um.bucket_count();     // Number of hash buckets
um.load_factor();      // Elements / buckets
um.max_load_factor();  // Threshold before rehash
```

### 4.2 `std::unordered_set`

```cpp
std::unordered_set<int> us = {5, 3, 1, 4, 2};
us.insert(6);      // O(1) average
us.count(4);       // O(1) average
```

### 4.3 `map` vs `unordered_map`

| Feature | `std::map` | `std::unordered_map` |
|---------|-----------|---------------------|
| Implementation | Red-Black Tree | Hash Table |
| Access | O(log n) | O(1) average, O(n) worst |
| Ordered | Yes (sorted by key) | No |
| Memory | Less overhead | More overhead (hash buckets) |
| Use when | Need sorted order or range queries | Need fastest lookup |

---

## Chapter 5: Container Adaptors

### 5.1 `std::stack` — LIFO

```cpp
#include <stack>
std::stack<int> stk;
stk.push(10);
stk.push(20);
stk.top();    // 20
stk.pop();    // Removes 20
stk.size();   // 1
stk.empty();  // false
```

### 5.2 `std::queue` — FIFO

```cpp
#include <queue>
std::queue<int> q;
q.push(10);
q.push(20);
q.front();    // 10
q.back();     // 20
q.pop();      // Removes 10
```

### 5.3 `std::priority_queue` — Max Heap

```cpp
std::priority_queue<int> pq;     // Max element on top
pq.push(30);
pq.push(10);
pq.push(20);
pq.top();    // 30 (max)
pq.pop();    // Removes 30
pq.top();    // 20

// Min heap:
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;
```

---

## Chapter 6: Iterators

### 6.1 Iterator Categories

| Category | Operations | Containers |
|----------|-----------|------------|
| Input | Read, ++ | `istream_iterator` |
| Output | Write, ++ | `ostream_iterator` |
| Forward | Read/Write, ++ | `forward_list`, `unordered_*` |
| Bidirectional | Read/Write, ++, -- | `list`, `set`, `map` |
| Random Access | Read/Write, ++, --, +n, -n, [] | `vector`, `deque`, `array` |
| Contiguous (C++20) | Random Access + contiguous memory | `vector`, `array`, `string` |

### 6.2 Iterator Usage

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};

// Forward iteration:
for (auto it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
}

// Reverse iteration:
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    std::cout << *it << " ";
}

// Const iterators:
for (auto it = v.cbegin(); it != v.cend(); ++it) {
    // *it = 0;  // ERROR: can't modify through const iterator
}
```

---

## Chapter 7: STL Algorithms

```cpp
#include <algorithm>
#include <numeric>

std::vector<int> v = {5, 3, 1, 4, 2};

// Sorting:
std::sort(v.begin(), v.end());                   // Ascending
std::sort(v.begin(), v.end(), std::greater<>()); // Descending
std::stable_sort(v.begin(), v.end());            // Preserves equal-element order
std::partial_sort(v.begin(), v.begin() + 3, v.end()); // First 3 sorted

// Searching:
bool found = std::binary_search(v.begin(), v.end(), 3);  // O(log n) on sorted
auto it = std::find(v.begin(), v.end(), 3);               // O(n) linear
auto it2 = std::find_if(v.begin(), v.end(), [](int x) { return x > 3; });
auto it3 = std::lower_bound(v.begin(), v.end(), 3);       // O(log n) on sorted

// Counting:
int count = std::count(v.begin(), v.end(), 3);
int countIf = std::count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });

// Transforming:
std::transform(v.begin(), v.end(), v.begin(), [](int x) { return x * 2; });

// Accumulation:
int sum = std::accumulate(v.begin(), v.end(), 0);
int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<>());

// Min/Max:
auto [minIt, maxIt] = std::minmax_element(v.begin(), v.end());

// Removing:
auto newEnd = std::remove_if(v.begin(), v.end(), [](int x) { return x < 3; });
v.erase(newEnd, v.end());  // Erase-remove idiom

// C++20 simplified:
std::erase_if(v, [](int x) { return x < 3; });

// Reversing:
std::reverse(v.begin(), v.end());

// Unique (removes consecutive duplicates):
std::sort(v.begin(), v.end());
auto last = std::unique(v.begin(), v.end());
v.erase(last, v.end());

// Copying:
std::vector<int> dest(v.size());
std::copy(v.begin(), v.end(), dest.begin());

// For each:
std::for_each(v.begin(), v.end(), [](int x) { std::cout << x << " "; });

// Any/All/None:
bool anyEven = std::any_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
bool allPositive = std::all_of(v.begin(), v.end(), [](int x) { return x > 0; });
bool noneNeg = std::none_of(v.begin(), v.end(), [](int x) { return x < 0; });

// Permutations:
std::next_permutation(v.begin(), v.end());

// Set Operations (on sorted ranges):
std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(result));
std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(result));
std::set_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(result));
```

---

## Chapter 8: Ranges (C++20)

```cpp
#include <ranges>

std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Pipe syntax — composable, lazy, no temporary containers:
auto result = v
    | std::views::filter([](int n) { return n % 2 == 0; })    // Even numbers
    | std::views::transform([](int n) { return n * n; })       // Square them
    | std::views::take(3);                                      // First 3

for (int n : result) {
    std::cout << n << " ";  // 4 16 36
}

// Range algorithms:
std::ranges::sort(v);
auto it = std::ranges::find(v, 5);
bool has = std::ranges::contains(v, 5);  // C++23

// Views:
std::views::iota(1, 11);              // Generate 1..10
std::views::filter(pred);              // Filter elements
std::views::transform(func);          // Transform elements
std::views::take(n);                   // First n elements
std::views::drop(n);                   // Skip first n
std::views::reverse;                   // Reverse order
std::views::keys;                      // Keys of pairs/maps
std::views::values;                    // Values of pairs/maps
std::views::zip(r1, r2);              // C++23: zip ranges
std::views::enumerate(r);             // C++23: index + value
```

---

## Chapter 9: Interview Questions

**Q1: When would you use `vector` vs `list`?** — Use `vector` by default (cache-friendly, O(1) access). Use `list` only when you need frequent insertions/deletions in the middle with stable iterators.

**Q2: What is the difference between `map` and `unordered_map`?** — `map` uses a red-black tree (O(log n), sorted). `unordered_map` uses a hash table (O(1) average, unsorted).

**Q3: What is iterator invalidation?** — Operations like `push_back`, `insert`, `erase` may reallocate memory, making existing iterators/pointers invalid. Each container has specific invalidation rules.

---

## Chapter 10: Practice & Projects

### Project 1: Frequency Analyzer — Count word frequencies in a text file using `unordered_map`.
### Project 2: Task Scheduler — Use `priority_queue` for task scheduling, `map` for time-based events.
### Project 3: In-Memory Database — Implement a simple database using maps, sets, and custom comparators.

---

## Chapter 11: Summary & Mastery Checklist

- [ ] You can choose the right container for any problem
- [ ] You understand time complexity of all container operations
- [ ] You know all essential STL algorithms
- [ ] You can use C++20 ranges and views
- [ ] You understand iterator categories and invalidation rules
- [ ] You know how containers are implemented internally

---

*End of Book 12: Standard Template Library*
