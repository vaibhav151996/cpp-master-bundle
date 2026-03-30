# Book 18: System Design and Advanced C++

## The Complete Guide to Architecture, Performance, and Low-Level Systems Programming

---

### **Target Level:** Expert
### **Prerequisites:** Books 1–17
### **Learning Outcomes:**
By the end of this book, you will:
- Design large-scale C++ systems with proper architecture
- Optimize hot paths with cache-awareness and SIMD
- Understand template metaprogramming at an advanced level
- Know memory layout, alignment, and allocator design
- Network programming with sockets and protocols
- Master build systems, testing, and CI/CD for C++ projects

---

## Chapter 1: System Architecture

### 1.1 Component Design

```
Principles:
1. Separation of Concerns — each module has one responsibility
2. Low Coupling — modules have minimal dependencies
3. High Cohesion — related functionality stays together
4. Dependency Injection — pass dependencies, don't construct them
5. Interface Segregation — small, focused interfaces
```

### 1.2 Layered Architecture

```
┌──────────────────────────────────┐
│         Presentation Layer        │  (UI, CLI, API endpoints)
├──────────────────────────────────┤
│          Application Layer        │  (Use cases, orchestration)
├──────────────────────────────────┤
│           Domain Layer            │  (Business logic, entities)
├──────────────────────────────────┤
│        Infrastructure Layer       │  (DB, network, filesystem)
└──────────────────────────────────┘
```

### 1.3 Error Handling Strategy

```cpp
// Use exceptions for exceptional cases:
class DatabaseError : public std::runtime_error {
    using std::runtime_error::runtime_error;
};

// Use std::expected (C++23) or error codes for expected failures:
std::expected<User, Error> findUser(int id) {
    if (id <= 0) return std::unexpected(Error::InvalidId);
    auto user = db.query(id);
    if (!user) return std::unexpected(Error::NotFound);
    return *user;
}

// Use std::optional for "might not have a value":
std::optional<int> parseInt(const std::string& s) {
    try { return std::stoi(s); }
    catch (...) { return std::nullopt; }
}
```

---

## Chapter 2: Performance Optimization

### 2.1 Cache-Friendly Programming

Understanding the CPU cache hierarchy is **the single most impactful optimization** you can make. A cache miss to RAM costs 100x more than an L1 hit.

```
CPU Cache Hierarchy — Latency & Size:

  CPU Core
  ┌──────────────────┐
  │   Registers      │  ~0.3ns    (bytes)
  │  ┌────────────┐  │
  │  │  L1 Cache  │  │  ~1ns      32 KB   ← per core, split I/D
  │  │  ┌──────┐  │  │
  │  │  │  L2  │  │  │  ~3-5ns    256 KB  ← per core
  │  │  └──────┘  │  │
  │  └────────────┘  │
  └──────────────────┘
        │
  ┌─────┴──────────────┐
  │     L3 Cache       │  ~10-20ns  8-32 MB  ← shared across cores
  └────────────────────┘
        │
  ┌─────┴──────────────┐
  │     Main RAM       │  ~60-100ns  GBs     ← 100x slower than L1!
  └────────────────────┘
        │
  ┌─────┴──────────────┐
  │     SSD / Disk     │  ~10,000-100,000ns
  └────────────────────┘

  Cache line = 64 bytes. When you read 1 byte, the CPU loads 64 bytes.
  → Sequential access = free prefetching.
  → Random access = cache miss on every read.
```

**Row-major vs Column-major traversal:**

```
Matrix in memory (row-major, C++ default):

  Row 0:  [a00][a01][a02][a03]  ← 64-byte cache line covers these
  Row 1:  [a10][a11][a12][a13]
  Row 2:  [a20][a21][a22][a23]

  ✔ Row-major (i, then j):      ✘ Column-major (j, then i):
  Access: a00 a01 a02 a03       Access: a00 a10 a20 a01 a11 a21
  Cache:  HIT HIT HIT HIT       Cache:  HIT MISS MISS HIT MISS MISS
  Sequential → prefetcher wins   Strided → cache misses everywhere
```

**AoS vs SoA Memory Layout:**

```
Array of Structures (AoS):
┌───┬───┬───┬────┬────┬────┬──────┬───┬───┬───┬────┬────┬────┬──────┐
│ x │ y │ z │ vx │ vy │ vz │ mass │ x │ y │ z │ vx │ vy │ vz │ mass │ ...
└───┴───┴───┴────┴────┴────┴──────┴───┴───┴───┴────┴────┴────┴──────┘
  Particle 0 (28 bytes)            Particle 1 (28 bytes)

  If you only need 'x': you load 28 bytes but use only 4.
  Cache utilization: 4/28 = 14%  ← WASTEFUL

Structure of Arrays (SoA):
┌───┬───┬───┬───┬───┬───┐  all x values packed together
│ x │ x │ x │ x │ x │ x │  ...
└───┴───┴───┴───┴───┴───┘
┌───┬───┬───┬───┬───┬───┐  all y values packed together
│ y │ y │ y │ y │ y │ y │  ...
└───┴───┴───┴───┴───┴───┘

  If you only need 'x': every byte in cache line is useful.
  Cache utilization: 100%  ← OPTIMAL for single-field access
  Also enables SIMD: process 4/8/16 x-values in one instruction.
```

```cpp
// CPU cache hierarchy:
// L1: ~32 KB, ~1ns       (per core)
// L2: ~256 KB, ~3-5ns    (per core)
// L3: ~8-32 MB, ~10-20ns (shared)
// RAM: GBs, ~60-100ns

// Cache line = 64 bytes on most architectures

// BAD: Column-major traversal (cache misses):
for (int j = 0; j < cols; ++j)
    for (int i = 0; i < rows; ++i)
        matrix[i][j] += 1;  // Jumps rows (64 * cols bytes apart)

// GOOD: Row-major traversal (cache-friendly):
for (int i = 0; i < rows; ++i)
    for (int j = 0; j < cols; ++j)
        matrix[i][j] += 1;  // Sequential memory access

// Structure of Arrays (SoA) vs Array of Structures (AoS):
// AoS — bad if you only access one field:
struct Particle { float x, y, z, vx, vy, vz, mass; };
std::vector<Particle> particles;  // 28 bytes per particle

// SoA — great for SIMD and when accessing one field at a time:
struct Particles {
    std::vector<float> x, y, z, vx, vy, vz, mass;
};
```

### 2.2 Avoiding Unnecessary Copies

```cpp
// 1. Pass by const reference for read-only:
void process(const std::string& s);

// 2. Move when transferring ownership:
void takeOwnership(std::string s) { data_ = std::move(s); }

// 3. Return by value (NRVO/copy elision):
std::string createString() {
    std::string result = "hello";
    return result;  // NRVO: no copy, no move
}

// 4. Reserve containers:
std::vector<int> v;
v.reserve(1000);  // One allocation instead of ~10 reallocations

// 5. Use emplace instead of push:
v.emplace_back(args...);   // Constructs in-place, no temporary
```

### 2.3 Branch Prediction

Modern CPUs predict which way a branch will go **before evaluating the condition**. When the prediction is wrong, the pipeline is flushed (~15-20 cycles wasted). Sorted data makes branches predictable.

```
Branch Prediction Example (if val > threshold):

  Unsorted data: [3, 17, 1, 25, 8, 30, 2, 19, 5, 28]
  Predictions:    N  Y  N   Y  N   Y  N   Y  N   Y   (random pattern)
  Accuracy:       ~50% → many mispredictions → SLOW

  Sorted data:   [1, 2, 3, 5, 8, 17, 19, 25, 28, 30]
  With threshold = 10:
  Below:          N  N  N  N  N   (all below → predict N)
  Above:                          Y   Y   Y   Y   Y  (all above → predict Y)
  Accuracy:       ~99% → pipeline runs at full speed → FAST

  Branchless alternative (eliminates the problem):
  Instead of:  if (a < b) result = a; else result = b;
  Use:         result = b + ((a-b) & ((a-b) >> 31));  // no branch!

  Rule of thumb:
  • Sort data before branching when possible
  • Use [[likely]]/[[unlikely]] hints for known distributions
  • For critical paths, consider branchless arithmetic
```

```cpp
// Sort data before branching:
// FAST: sorted data → branch predictor succeeds ~100%
std::sort(data.begin(), data.end());
for (auto val : data) {
    if (val > threshold) sum += val;  // Predictable branch
}

// Use branchless code for critical paths:
// Branchless min:
int branchlessMin(int a, int b) {
    return b + ((a - b) & ((a - b) >> 31));
}

// [[likely]] / [[unlikely]] hints (C++20):
if (ptr != nullptr) [[likely]] {
    process(ptr);
}
```

### 2.4 Compile-Time Computation

```cpp
// Move computation to compile time:
constexpr auto lookupTable = []() {
    std::array<int, 256> table{};
    for (int i = 0; i < 256; ++i)
        table[i] = /* compute */;
    return table;
}();

// constexpr algorithms:
constexpr int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; ++i) {
        int c = a + b;
        a = b;
        b = c;
    }
    return b;
}

static_assert(fibonacci(10) == 55);  // Verified at compile time
```

---

## Chapter 3: Memory Management & Allocators

### 3.1 Custom Allocators

**Arena Allocator** — the simplest and fastest allocator. Allocations are O(1) (just bump a pointer). Deallocation is all-or-nothing (`reset()`). Perfect for per-frame game data, request-scoped data, or parsing.

```
Arena Allocator — Bump Pointer:

  buffer_  (pre-allocated, e.g. 4KB)
  ┌──────────────────────────────────────────────────┐
  │ used │ used │ used │   ... free space ...         │
  └──────────────────────────────────────────────────┘
  ^                    ^                              ^
  buffer_           offset_                       capacity_

  allocate(N):   ptr = buffer_ + aligned_offset
                 offset_ += N
                 return ptr      ← O(1), no bookkeeping!

  reset():       offset_ = 0     ← "frees" everything instantly

  Step-by-step:
  1. alloc(16)   [################|                        ]  offset=16
  2. alloc(8)    [################|########|                ]  offset=24
  3. alloc(32)   [################|########|################|  ]  offset=56
  4. reset()     [                                          ]  offset=0

  ✔ No fragmentation  ✔ Cache-friendly (contiguous)  ✔ No per-object free
  ✘ Cannot free individual objects   ✘ Fixed size (grow = new chunk)
```

**Pool Allocator** — fixed-size block allocation with O(1) alloc/dealloc via a free list. Ideal for many same-sized objects (particles, nodes, entities).

```
Pool Allocator — Free List:

  Chunk of memory (64 blocks, each sizeof(T) bytes):
  ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┐
  │ blk0 │ blk1 │ blk2 │ blk3 │ blk4 │ blk5 │ ...  │
  └──┬───┴──┬───┴──┬───┴──┬───┴──┬───┴──┬───┴──────┘
     │      │      │      │      │      │
     └──►───┘──►───┘──►───┘──►───┘──►───┘──► null
          Embedded free list (each block has next pointer)

  allocate():    pop head of free list     ← O(1)
  deallocate():  push back onto free list  ← O(1)

  After alloc(blk0), alloc(blk1), dealloc(blk0):
  ┌──────┬──────┬──────┬──────┬──────┐
  │ FREE │ USED │ blk2 │ blk3 │ ...  │
  └──┬───┴──────┴──┬───┴──┬───┴──────┘
     │             │      │
     └──────►──────┘──►───┘──► null
     freeList_ points to blk0, then blk2, then blk3...

  ✔ O(1) alloc/dealloc  ✔ No fragmentation (fixed size)
  ✘ Only works for same-sized objects
```

```cpp
// Arena/Linear allocator — O(1) allocation, no individual deallocation:
class ArenaAllocator {
    char* buffer_;
    size_t capacity_;
    size_t offset_ = 0;

public:
    ArenaAllocator(size_t size)
        : buffer_(new char[size]), capacity_(size) {}
    ~ArenaAllocator() { delete[] buffer_; }

    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) {
        size_t aligned = (offset_ + alignment - 1) & ~(alignment - 1);
        if (aligned + size > capacity_) throw std::bad_alloc();
        void* ptr = buffer_ + aligned;
        offset_ = aligned + size;
        return ptr;
    }

    void reset() { offset_ = 0; }  // "Free" everything at once
};

// Pool allocator — fixed-size blocks, O(1) alloc/dealloc:
template<typename T>
class PoolAllocator {
    struct Block { Block* next; };
    Block* freeList_ = nullptr;
    std::vector<char*> chunks_;

    void addChunk() {
        char* chunk = new char[sizeof(T) * 64];
        chunks_.push_back(chunk);
        for (int i = 0; i < 64; ++i) {
            auto block = reinterpret_cast<Block*>(chunk + i * sizeof(T));
            block->next = freeList_;
            freeList_ = block;
        }
    }

public:
    T* allocate() {
        if (!freeList_) addChunk();
        Block* block = freeList_;
        freeList_ = block->next;
        return reinterpret_cast<T*>(block);
    }

    void deallocate(T* ptr) {
        auto block = reinterpret_cast<Block*>(ptr);
        block->next = freeList_;
        freeList_ = block;
    }

    ~PoolAllocator() {
        for (auto chunk : chunks_) delete[] chunk;
    }
};
```

### 3.2 Placement New

```cpp
// Construct object at specific memory location:
alignas(Widget) char buffer[sizeof(Widget)];
Widget* w = new (buffer) Widget(args...);
w->~Widget();  // Must manually call destructor

// Used in custom allocators, memory-mapped I/O, etc.
```

### 3.3 Memory-Mapped Files

```cpp
// Platform-specific (POSIX):
#include <sys/mman.h>
int fd = open("data.bin", O_RDONLY);
void* mapped = mmap(nullptr, fileSize, PROT_READ, MAP_PRIVATE, fd, 0);
// Access mapped memory directly — OS handles paging
munmap(mapped, fileSize);
close(fd);

// Windows:
// CreateFileMapping + MapViewOfFile
```

---

## Chapter 4: Template Metaprogramming (Advanced)

### 4.1 Type Traits

Type traits let you **inspect and transform types at compile time**. They are the building blocks of generic programming — enabling different code paths based on type properties.

```
Type Traits Decision Tree:

  template<typename T>
  void process(T value) {
      if constexpr (std::is_integral_v<T>) {
          // Integer-specific code (bitwise ops, etc.)
      } else if constexpr (std::is_floating_point_v<T>) {
          // Float-specific code (epsilon comparison, etc.)
      } else if constexpr (std::is_pointer_v<T>) {
          // Pointer-specific code (null check, etc.)
      } else {
          // Generic fallback
      }
  }

  Common type traits:
  ┌─────────────────────────┬───────────────┬──────────────────────────┐
  │ Check                   │ True for       │ Modern C++ alternative     │
  ├─────────────────────────┼───────────────┼──────────────────────────┤
  │ is_integral_v<T>        │ int, char, bool│ std::integral concept      │
  │ is_floating_point_v<T>  │ float, double  │ std::floating_point        │
  │ is_trivially_copyable_v │ POD-like types │ (can memcpy safely)        │
  │ is_base_of_v<Base,Der>  │ inheritance    │ std::derived_from concept  │
  │ is_same_v<T, U>         │ exact type match │ std::same_as concept     │
  └─────────────────────────┴───────────────┴──────────────────────────┘

  Modern C++20 recommendation: Use concepts instead of SFINAE/enable_if.
```

```cpp
#include <type_traits>

// Compile-time type inspection:
static_assert(std::is_integral_v<int>);
static_assert(std::is_floating_point_v<double>);
static_assert(std::is_same_v<int, int>);
static_assert(std::is_base_of_v<Base, Derived>);
static_assert(std::is_trivially_copyable_v<int>);

// Conditional types:
using Type = std::conditional_t<sizeof(int) == 4, int32_t, int64_t>;

// Remove qualifiers:
using T = std::remove_const_t<const int>;           // int
using U = std::remove_reference_t<int&>;             // int
using V = std::decay_t<const int&>;                  // int
```

### 4.2 Compile-Time Computation with Templates

```cpp
// Compile-time factorial:
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

static_assert(Factorial<5>::value == 120);

// Modern approach with constexpr is preferred (see Book 13)
```

### 4.3 SFINAE & enable_if

```cpp
// Enable function only for integral types:
template<typename T>
std::enable_if_t<std::is_integral_v<T>, T>
safeAdd(T a, T b) {
    if (__builtin_add_overflow(a, b, &a))
        throw std::overflow_error("overflow");
    return a;
}

// C++20 concepts are the modern replacement (see Book 13)
```

---

## Chapter 5: Networking

### 5.1 Socket Programming Basics

Socket programming follows a strict lifecycle. Understanding the sequence of system calls is essential for building reliable network services.

```
TCP Socket Lifecycle:

  SERVER                                    CLIENT
  ──────                                    ──────
  socket()     Create socket                socket()
     │                                         │
  bind()       Bind to port 8080               │
     │                                         │
  listen()     Mark as passive socket          │
     │                                         │
  accept() ─── blocks until ──── connect() ────┘
     │         client connects        │
     │                                │
     │    ┌─── TCP 3-Way Handshake ───┐
     │    │  SYN    ───────────────►  │
     │    │  SYN+ACK ◄────────────── │
     │    │  ACK    ───────────────►  │
     │    └──────────────────────────┘
     │         Connection ESTABLISHED
     │                                │
  read() ◄──── write("Hello") ────── write()
     │                                │
  write("OK") ───── read() ──────► read()
     │                                │
  close()      FIN ──────────────► close()
               ACK ◄──────────────
```

**Key points:**
- `socket()` creates a file descriptor, not a connection
- `bind()` associates the socket with an IP:port
- `listen()` marks it as a passive (accepting) socket
- `accept()` blocks until a client connects, returns a NEW fd for that connection
- The original socket continues to accept more clients

```cpp
// TCP Server (POSIX-style, simplified):
#include <sys/socket.h>
#include <netinet/in.h>

// 1. Create socket
int serverFd = socket(AF_INET, SOCK_STREAM, 0);

// 2. Bind to port
sockaddr_in addr{};
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = INADDR_ANY;
addr.sin_port = htons(8080);
bind(serverFd, (sockaddr*)&addr, sizeof(addr));

// 3. Listen
listen(serverFd, 10);

// 4. Accept connections
int clientFd = accept(serverFd, nullptr, nullptr);

// 5. Read/Write
char buffer[1024];
read(clientFd, buffer, sizeof(buffer));
write(clientFd, "HTTP/1.1 200 OK\r\n\r\nHello", 25);

// 6. Close
close(clientFd);
close(serverFd);
```

### 5.2 Asynchronous I/O Patterns

Choosing the right I/O model dramatically affects server performance. Here's a comparison:

```
I/O Models — from simple to scalable:

  1. Blocking + Thread-per-connection:
     Thread 1: ──read()────────────  (blocked, waiting for data)
     Thread 2: ──read()────────────  (blocked too)
     ✔ Simple  ✘ 10K threads = out of memory

  2. Non-blocking + select/poll:
     Single thread polls ALL sockets:
     ┌──────────────────────────────────────┐
     │ select({fd1, fd2, fd3, ...})         │  ← which sockets have data?
     │ → fd1 ready → read(fd1)             │
     │ → fd3 ready → read(fd3)             │
     │ → loop                              │
     └──────────────────────────────────────┘
     ✔ Few threads  ✘ O(n) scan of all fds each time

  3. epoll (Linux) / kqueue (macOS) / IOCP (Windows):
     Kernel tells you ONLY which fds are ready:
     ┌──────────────────────────────────────┐
     │ epoll_wait() → {fd1, fd3}           │  ← only ready ones returned
     │ → process fd1, fd3                  │
     │ → no scanning!                      │
     └──────────────────────────────────────┘
     ✔ O(1) per ready fd  ✔ Handles 100K+ connections
     This is what Nginx, Redis, Node.js use.

  4. io_uring (Linux 5.1+):
     ┌────────────────────────┐  ┌────────────────────────┐
     │  Submission Queue (SQ) │  │  Completion Queue (CQ) │
     │  [read fd1]            │  │  [fd1: 128 bytes]      │
     │  [write fd2]           │  │  [fd2: OK]             │
     └────────────────────────┘  └────────────────────────┘
     Batch submit syscalls, kernel processes async.
     ✔ Zero-copy  ✔ Batched  ✔ Lowest overhead possible
```

```
1. Blocking I/O + Thread Pool — simple, scales poorly
2. Non-blocking I/O + select/poll — classic Unix approach
3. epoll(Linux)/kqueue(macOS)/IOCP(Windows) — scalable event loops
4. io_uring (Linux 5.1+) — zero-copy, batched syscalls
5. Boost.Asio / Asio — cross-platform async networking

Real-world: Nginx uses epoll, Node.js uses libuv (epoll/kqueue/IOCP)
```

### 5.3 Serialization

```cpp
// Binary serialization:
struct Packet {
    uint32_t type;
    uint32_t length;
    char data[];  // Flexible array member
};

// Write:
void serialize(std::ostream& os, int32_t value) {
    // Convert to network byte order:
    int32_t netValue = htonl(value);
    os.write(reinterpret_cast<const char*>(&netValue), sizeof(netValue));
}

// JSON/Protocol Buffers/FlatBuffers for structured data
```

---

## Chapter 6: Build Systems & Tooling

### 6.1 CMake — The Standard Build System

CMake generates platform-specific build files (Makefiles, Ninja, Visual Studio projects). Understanding the CMake build pipeline helps you debug build issues.

```
CMake Build Pipeline:

  CMakeLists.txt (your configuration)
        │
        ▼  cmake -B build
  ┌─────────────────┐
  │  Configure Step │  Reads CMakeLists.txt, finds compilers,
  │                 │  resolves find_package(), sets variables
  └────────┬────────┘
           │  generates
           ▼
  ┌─────────────────┐
  │  Build Files    │  Makefile, build.ninja, .vcxproj
  └────────┬────────┘
           │  cmake --build build
           ▼
  ┌─────────────────┐
  │  Compile Step   │  .cpp → .o (object files)
  └────────┬────────┘
           │
           ▼
  ┌─────────────────┐
  │   Link Step     │  .o files → executable or library
  └────────┬────────┘
           │
           ▼
     myapp(.exe)  or  libmylib.a / .so

  Key targets:
  add_executable()  → produces an executable
  add_library()     → produces a static/shared library
  target_link_libraries() → connects targets together
```

```cmake
cmake_minimum_required(VERSION 3.20)
project(MyApp VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Library:
add_library(mylib STATIC src/lib.cpp)
target_include_directories(mylib PUBLIC include/)

# Executable:
add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)

# Testing:
enable_testing()
find_package(GTest REQUIRED)
add_executable(tests tests/test_lib.cpp)
target_link_libraries(tests PRIVATE mylib GTest::gtest_main)
add_test(NAME unit_tests COMMAND tests)

# External dependencies:
find_package(fmt REQUIRED)
target_link_libraries(myapp PRIVATE fmt::fmt)
```

### 6.2 Package Managers

```
- vcpkg (Microsoft) — cross-platform, integrates with CMake
- Conan — flexible, decentralized
- CPM.cmake — header-only CMake dependency manager
```

### 6.3 Testing Frameworks

```cpp
// Google Test:
#include <gtest/gtest.h>

TEST(MathTest, Addition) {
    EXPECT_EQ(add(2, 3), 5);
    EXPECT_NE(add(2, 3), 6);
    EXPECT_THROW(divide(1, 0), std::runtime_error);
}

// Catch2:
#include <catch2/catch_test_macros.hpp>

TEST_CASE("Vector operations", "[vector]") {
    std::vector<int> v;
    REQUIRE(v.empty());

    SECTION("push_back increases size") {
        v.push_back(1);
        REQUIRE(v.size() == 1);
    }
}

// Google Benchmark:
#include <benchmark/benchmark.h>

static void BM_VectorPushBack(benchmark::State& state) {
    for (auto _ : state) {
        std::vector<int> v;
        for (int i = 0; i < state.range(0); ++i)
            v.push_back(i);
    }
}
BENCHMARK(BM_VectorPushBack)->Range(8, 1 << 20);
```

### 6.4 Sanitizers

```bash
# Address Sanitizer (memory errors):
g++ -fsanitize=address -g main.cpp

# Thread Sanitizer (data races):
g++ -fsanitize=thread -g main.cpp

# Undefined Behavior Sanitizer:
g++ -fsanitize=undefined -g main.cpp

# Memory Sanitizer (uninitialized reads, Clang only):
clang++ -fsanitize=memory -g main.cpp
```

### 6.5 Profiling

```
Tools:
- perf (Linux) — CPU profiling, cache analysis
- Valgrind/Callgrind — detailed profiling (slow)
- Intel VTune — advanced CPU profiling
- Tracy — real-time frame profiler (games)
- gprof — basic GNU profiler

Commands:
perf record ./myapp
perf report

valgrind --tool=callgrind ./myapp
```

---

## Chapter 7: Advanced C++ Idioms

### 7.1 Pimpl (Pointer to Implementation)

The Pimpl idiom hides implementation details behind a pointer. This creates a **compilation firewall** — changes to the implementation don't require recompiling users of the header.

```
Pimpl Compilation Firewall:

  Without Pimpl:                     With Pimpl:
  ┌─────────────┐                    ┌──────────────┐
  │  widget.h   │ includes           │  widget.h    │ includes NOTHING
  │  #include   │ <database.h>       │  struct Impl;│ (forward decl only)
  │  #include   │ <network.h>        │  unique_ptr  │
  │  #include   │ <renderer.h>       │    <Impl>    │
  └──────┬──────┘                    └──────┬───────┘
         │                                  │
  Change database.h?                 Change impl details?
  → RECOMPILE everything             → Recompile ONLY widget.cpp
    that includes widget.h              (header is unchanged)
         │                                  │
    10 min rebuild                      5 sec rebuild

  widget.cpp defines struct Widget::Impl { ... actual members ... };
  All #includes go in .cpp, not .h.
```

Hides implementation details, reduces compilation dependencies:
```cpp
// widget.h — stable interface, no implementation details exposed
class Widget {
    struct Impl;
    std::unique_ptr<Impl> pImpl_;
public:
    Widget();
    ~Widget();
    void doSomething();
};
```

### 7.2 CRTP (Curiously Recurring Template Pattern)

```cpp
template<typename Derived>
class Cloneable {
public:
    std::unique_ptr<Derived> clone() const {
        return std::make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};

class MyClass : public Cloneable<MyClass> { /* ... */ };
```

### 7.3 Tag Dispatch

Tag dispatch selects algorithm implementations at compile time based on **iterator category tags**. The compiler chooses the right overload — no runtime cost.

```
Tag Dispatch Selection:

  advance(it, 5)  ──►  What kind of iterator is 'it'?
                      │
       ┌────────────┴────────────┐
       │                         │
  random_access_tag          input_iterator_tag
       │                         │
   it += 5  (O(1))            ++it 5 times (O(n))

  The correct overload is chosen at COMPILE TIME.
  Zero overhead — same as hand-writing the specific version.
```

```cpp
template<typename Iter>
void advance_impl(Iter& it, int n, std::random_access_iterator_tag) {
    it += n;  // O(1)
}

template<typename Iter>
void advance_impl(Iter& it, int n, std::input_iterator_tag) {
    while (n-- > 0) ++it;  // O(n)
}

template<typename Iter>
void advance(Iter& it, int n) {
    advance_impl(it, n, typename std::iterator_traits<Iter>::iterator_category{});
}
```

### 7.4 Policy-Based Design

Policy-based design builds classes from **composable policy templates**. Each policy handles one concern (logging, threading, caching). This gives you combinatorial flexibility without runtime cost.

```
Policy-Based Design Structure:

  Server<LogPolicy, ThreadPolicy>
  ┌─────────────────────────────────┐
  │  Uses LogPolicy::log()          │
  │  Uses ThreadPolicy::runThreaded()│
  └─────────────────────────────────┘

  Mix and match any combination:
  ┌──────────────┐  ┌──────────────┐
  │ ConsoleLog     │  │ SingleThreaded │  → Server<ConsoleLog, SingleThreaded>
  │ FileLog        │  │ MultiThreaded  │  → Server<FileLog, MultiThreaded>
  │ NetworkLog     │  │ ThreadPool     │  → Server<NetworkLog, ThreadPool>
  └──────────────┘  └──────────────┘
  3 log policies     3 thread policies = 9 combinations!

  All resolved at compile time — zero virtual function overhead.
  This is how the C++ standard library is designed (allocators, etc.).
```

```cpp
template<typename LogPolicy, typename ThreadPolicy>
class Server : private LogPolicy, private ThreadPolicy {
public:
    void start() {
        this->log("Starting server");
        this->runThreaded([this]() {
            // Handle connections
        });
    }
};

struct ConsoleLog {
    void log(const std::string& msg) { std::cout << msg << "\n"; }
};

struct SingleThreaded {
    template<typename F>
    void runThreaded(F&& f) { f(); }
};

using SimpleServer = Server<ConsoleLog, SingleThreaded>;
```

---

## Chapter 8: Interview Questions

**Q1: How would you optimize a function that runs in a hot loop?** — Profile first. Check for cache misses (SoA layout). Eliminate branches. Use SIMD. Consider constexpr. Minimize allocations. Inline small functions. Consider lock-free structures for concurrent access.

**Q2: Explain the Pimpl idiom.** — Hide implementation in .cpp file using a forward-declared struct + unique_ptr. Benefits: reduces compile dependencies, stable ABI, faster builds.

**Q3: What is an arena allocator?** — Allocates from a pre-allocated buffer with O(1) allocation (bump pointer). Deallocation is all-or-nothing (reset). Used for per-frame game data, request handling, parsing.

---

## Chapter 9: Mastery Checklist

- [ ] You can design multi-layered C++ applications
- [ ] You understand cache hierarchies and optimize for them
- [ ] You can write custom allocators (arena, pool)
- [ ] You know advanced template metaprogramming (type traits, SFINAE, concepts)
- [ ] You can do TCP/UDP socket programming
- [ ] You use CMake, testing frameworks, and sanitizers effectively
- [ ] You can profile and optimize C++ applications

---

*End of Book 18: System Design and Advanced C++*
