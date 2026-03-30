# Book 19: Real-World Systems Programming in C++

## Building Production-Grade Software: Game Engines, Databases, Trading Systems, and More

---

### **Target Level:** Expert
### **Prerequisites:** Books 1–18
### **Learning Outcomes:**
By the end of this book, you will:
- Understand how C++ powers real-world high-performance systems
- Know architectural patterns used in game engines, databases, and OS kernels
- Design and implement components of production systems
- Understand embedded systems, trading systems, and compilers
- Bridge the gap from textbook knowledge to industry-ready engineering

---

## Chapter 1: Game Engine Architecture

### 1.1 Core Architecture

```
┌────────────────────────────────────────────┐
│              Game Application               │
├──────────┬──────────┬──────────┬───────────┤
│  Physics │ Renderer │  Audio   │ Scripting │
├──────────┴──────────┴──────────┴───────────┤
│           Entity Component System           │
├──────────┬──────────┬──────────┬───────────┤
│ Resource │  Scene   │  Input   │  Network  │
│ Manager  │  Graph   │ Manager  │  Manager  │
├──────────┴──────────┴──────────┴───────────┤
│        Platform Abstraction Layer           │
│     (Window, FileSystem, Threading)         │
├─────────────────────────────────────────────┤
│        Memory Management (Allocators)       │
└─────────────────────────────────────────────┘
```

### 1.2 Entity Component System (ECS)

The dominant architecture for modern game engines. Separates data from behavior:

```cpp
// Component = pure data:
struct Position { float x, y, z; };
struct Velocity { float dx, dy, dz; };
struct Sprite { int textureId; float width, height; };

// System = logic that operates on components:
class MovementSystem {
public:
    void update(float dt, std::vector<Position>& positions,
                const std::vector<Velocity>& velocities) {
        for (size_t i = 0; i < positions.size(); ++i) {
            positions[i].x += velocities[i].dx * dt;
            positions[i].y += velocities[i].dy * dt;
            positions[i].z += velocities[i].dz * dt;
        }
    }
};

// Entity = just an ID:
using Entity = uint32_t;

// Why ECS?
// - Cache-friendly (Structure of Arrays layout)
// - Easy to parallelize (systems operate on independent data)
// - Composable (add/remove components dynamically)
// - Used by: Unity DOTS, Unreal Mass, EnTT library
```

### 1.3 Game Loop

```cpp
class GameEngine {
    bool running_ = true;
    static constexpr double FIXED_TIMESTEP = 1.0 / 60.0;  // 60 Hz physics

public:
    void run() {
        double accumulator = 0.0;
        auto previousTime = std::chrono::high_resolution_clock::now();

        while (running_) {
            auto currentTime = std::chrono::high_resolution_clock::now();
            double frameTime = std::chrono::duration<double>(
                currentTime - previousTime).count();
            previousTime = currentTime;

            // Cap frame time to avoid spiral of death:
            frameTime = std::min(frameTime, 0.25);

            processInput();

            // Fixed timestep for physics (deterministic):
            accumulator += frameTime;
            while (accumulator >= FIXED_TIMESTEP) {
                updatePhysics(FIXED_TIMESTEP);
                accumulator -= FIXED_TIMESTEP;
            }

            // Variable timestep for interpolated rendering:
            double alpha = accumulator / FIXED_TIMESTEP;
            render(alpha);
        }
    }
};
```

### 1.4 Frame Allocator (Per-Frame Memory)

```cpp
class FrameAllocator {
    char* buffer_;
    size_t capacity_;
    size_t offset_ = 0;

public:
    FrameAllocator(size_t size) : buffer_(new char[size]), capacity_(size) {}
    ~FrameAllocator() { delete[] buffer_; }

    void* allocate(size_t size) {
        size_t aligned = (offset_ + 15) & ~15;  // 16-byte alignment
        if (aligned + size > capacity_) return nullptr;
        void* ptr = buffer_ + aligned;
        offset_ = aligned + size;
        return ptr;
    }

    void reset() { offset_ = 0; }  // Called at start of each frame
};

// Usage: all per-frame allocations are O(1), no fragmentation
// Reset at frame boundary — no individual free needed
```

### 1.5 Real Engines in C++

| Engine | Language | Notable Features |
|--------|----------|-----------------|
| Unreal Engine | C++ | Blueprint visual scripting, AAA games |
| Godot (core) | C++ | Open source, GDScript bindings |
| id Tech | C/C++ | Doom, Quake, Vulkan renderer |
| CryEngine | C++ | Photorealistic rendering |
| Custom engines | C++ | EA DICE (Frostbite), Naughty Dog |

---

## Chapter 2: Database Engine Internals

### 2.1 Architecture

```
┌─────────────────────────────┐
│       SQL Parser/Planner     │
├─────────────────────────────┤
│      Query Optimizer         │
├─────────────────────────────┤
│      Execution Engine        │
├──────────┬──────────────────┤
│  Buffer  │   Transaction    │
│  Pool    │   Manager (ACID) │
├──────────┴──────────────────┤
│      Storage Engine          │
│  (B+ Tree, LSM Tree, etc.)  │
├─────────────────────────────┤
│      Disk Manager            │
└─────────────────────────────┘
```

### 2.2 B+ Tree (Core Index Structure)

```cpp
// B+ Tree properties:
// - All data in leaf nodes
// - Internal nodes store keys for navigation
// - Leaf nodes linked together (range scans)
// - Optimized for disk I/O (high branching factor → low height)

template<typename Key, typename Value, int ORDER = 128>
class BPlusTree {
    struct Node {
        bool isLeaf;
        int numKeys;
        Key keys[ORDER - 1];
    };

    struct InternalNode : Node {
        Node* children[ORDER];
    };

    struct LeafNode : Node {
        Value values[ORDER - 1];
        LeafNode* next;  // Linked list for range scans
    };

    // Typical database page = 4KB or 8KB
    // B+ tree with ORDER=128 and 4-level tree can index billions of rows
};
```

### 2.3 Buffer Pool Manager

```cpp
class BufferPool {
    struct Frame {
        char data[PAGE_SIZE];  // Typically 4096 bytes
        int pageId = -1;
        bool dirty = false;
        int pinCount = 0;
    };

    std::vector<Frame> frames_;
    std::unordered_map<int, int> pageTable_;  // pageId → frameIndex

public:
    Frame* fetchPage(int pageId) {
        if (pageTable_.count(pageId)) {
            auto& frame = frames_[pageTable_[pageId]];
            frame.pinCount++;
            return &frame;
        }
        // Evict a page (LRU/Clock algorithm), read from disk
        int victim = findVictim();
        if (frames_[victim].dirty) writeToDisk(frames_[victim]);
        readFromDisk(pageId, frames_[victim]);
        pageTable_[pageId] = victim;
        frames_[victim].pinCount = 1;
        return &frames_[victim];
    }
};
```

### 2.4 Write-Ahead Logging (WAL)

```
Every modification is first written to a log file (WAL),
then applied to in-memory pages, then eventually flushed to disk.

Recovery: replay log from last checkpoint.
Ensures durability (the D in ACID) even if the system crashes.

Used by: PostgreSQL, SQLite, MySQL InnoDB, RocksDB
```

### 2.5 Databases Written in C++

| Database | Type | Notes |
|----------|------|-------|
| MySQL | Relational | InnoDB storage engine |
| MongoDB | Document | WiredTiger engine |
| RocksDB | Key-Value | LSM tree, by Facebook |
| ClickHouse | Columnar | Analytics, by Yandex |
| ScyllaDB | Wide-column | C++ rewrite of Cassandra |
| LevelDB | Key-Value | by Google |

---

## Chapter 3: High-Frequency Trading Systems

### 3.1 Requirements

```
Latency: < 1 microsecond for critical path
Throughput: millions of messages/second
Determinism: predictable, consistent performance (no GC pauses!)
Reliability: 99.999% uptime

Why C++? Zero-overhead abstractions, no garbage collector,
hardware-level control, template metaprogramming for zero-cost abstractions
```

### 3.2 Low-Latency Techniques

```cpp
// 1. Lock-free data structures:
template<typename T>
class SPSCQueue {  // Single-Producer Single-Consumer
    std::vector<T> buffer_;
    std::atomic<size_t> head_{0};
    std::atomic<size_t> tail_{0};
    size_t capacity_;

public:
    SPSCQueue(size_t cap) : buffer_(cap), capacity_(cap) {}

    bool push(const T& val) {
        size_t tail = tail_.load(std::memory_order_relaxed);
        size_t next = (tail + 1) % capacity_;
        if (next == head_.load(std::memory_order_acquire)) return false;
        buffer_[tail] = val;
        tail_.store(next, std::memory_order_release);
        return true;
    }

    bool pop(T& val) {
        size_t head = head_.load(std::memory_order_relaxed);
        if (head == tail_.load(std::memory_order_acquire)) return false;
        val = buffer_[head];
        head_.store((head + 1) % capacity_, std::memory_order_release);
        return true;
    }
};

// 2. Kernel bypass: DPDK for networking, avoid syscall overhead
// 3. CPU pinning: Pin threads to specific CPU cores
// 4. Huge pages: Reduce TLB misses
// 5. Pre-allocate everything: No malloc on critical path
// 6. Avoid virtual functions on hot path (use CRTP)
// 7. Compiler intrinsics for SIMD operations
```

### 3.3 Order Book Implementation

```cpp
struct Order {
    uint64_t orderId;
    double price;
    int quantity;
    bool isBuy;
};

class OrderBook {
    // Price levels ordered by price:
    std::map<double, std::list<Order>, std::greater<>> bids_;  // Descending
    std::map<double, std::list<Order>> asks_;                   // Ascending

public:
    void addOrder(Order order) {
        auto& book = order.isBuy ? bids_ : asks_;
        book[order.price].push_back(order);
    }

    double bestBid() const {
        return bids_.empty() ? 0.0 : bids_.begin()->first;
    }

    double bestAsk() const {
        return asks_.empty() ? 0.0 : asks_.begin()->first;
    }

    double spread() const { return bestAsk() - bestBid(); }
};
```

---

## Chapter 4: Operating System Components

### 4.1 What OS Kernels Use C++ For

```
- Linux kernel: C (no C++), but many userspace tools are C++
- Windows NT kernel: mix of C and C++
- Zircon (Fuchsia OS): C++
- seL4: C/assembly (formally verified microkernel)

C++ in OS-level code:
- Device drivers (Windows WDM)
- File systems
- Network stacks
- Hypervisors
```

### 4.2 Memory-Mapped I/O

```cpp
// Hardware registers at specific physical addresses:
volatile uint32_t* const GPIO_BASE = 
    reinterpret_cast<volatile uint32_t*>(0x3F200000);

// Set pin high:
GPIO_BASE[7] = (1 << pin);  // Set register
GPIO_BASE[10] = (1 << pin); // Clear register

// volatile: compiler must not optimize away reads/writes
```

### 4.3 Interrupt Handling

```cpp
// ISR (Interrupt Service Routine) constraints:
// - Must be fast (microseconds)
// - Cannot allocate memory
// - Cannot use blocking I/O
// - Cannot throw exceptions
// - Typically: acknowledge interrupt, copy data, signal worker thread

void __attribute__((interrupt)) timer_handler() {
    // Minimal work: update counter, acknowledge hardware
    systemTicks++;
    acknowledgeInterrupt(TIMER_IRQ);
}
```

---

## Chapter 5: Embedded Systems

### 5.1 Constraints

```
- Limited memory (KB to MB, not GB)
- No heap allocation (or very limited)
- No exceptions (code size overhead)
- No RTTI (runtime type information)
- Deterministic timing (hard real-time)
- Cross-compilation required

Compiler flags:
-fno-exceptions -fno-rtti -fno-threadsafe-statics -Os
```

### 5.2 Embedded C++ Patterns

```cpp
// Static polymorphism (CRTP) instead of virtual functions:
template<typename Derived>
class Sensor {
public:
    int read() {
        return static_cast<Derived*>(this)->readImpl();
    }
};

class TemperatureSensor : public Sensor<TemperatureSensor> {
public:
    int readImpl() {
        return *reinterpret_cast<volatile int*>(TEMP_REG_ADDR);
    }
};

// Compile-time configuration instead of runtime:
template<uint32_t BaudRate, uint8_t DataBits = 8>
class UART {
    static_assert(BaudRate == 9600 || BaudRate == 115200,
                  "Unsupported baud rate");
public:
    void init() {
        constexpr uint32_t divisor = CLOCK_FREQ / (16 * BaudRate);
        // Configure hardware registers at compile time
    }
};
```

---

## Chapter 6: Compiler/Interpreter Design

### 6.1 Compilation Pipeline

```
Source Code → Lexer → Parser → AST → Semantic Analysis →
IR Generation → Optimization → Code Generation → Machine Code

GCC, Clang/LLVM, MSVC are all written in C/C++.
```

### 6.2 Simple Expression Parser

```cpp
// Recursive descent parser for: expr = term (('+' | '-') term)*
class Parser {
    std::string input_;
    size_t pos_ = 0;

    char peek() { return pos_ < input_.size() ? input_[pos_] : '\0'; }
    char advance() { return input_[pos_++]; }

    // number = [0-9]+
    double number() {
        double val = 0;
        while (std::isdigit(peek()))
            val = val * 10 + (advance() - '0');
        return val;
    }

    // factor = number | '(' expr ')'
    double factor() {
        if (peek() == '(') {
            advance();  // '('
            double val = expr();
            advance();  // ')'
            return val;
        }
        return number();
    }

    // term = factor (('*' | '/') factor)*
    double term() {
        double val = factor();
        while (peek() == '*' || peek() == '/') {
            char op = advance();
            double right = factor();
            if (op == '*') val *= right;
            else val /= right;
        }
        return val;
    }

public:
    // expr = term (('+' | '-') term)*
    double expr() {
        double val = term();
        while (peek() == '+' || peek() == '-') {
            char op = advance();
            double right = term();
            if (op == '+') val += right;
            else val -= right;
        }
        return val;
    }

    double parse(const std::string& input) {
        input_ = input;
        pos_ = 0;
        return expr();
    }
};

// Usage:
Parser p;
std::cout << p.parse("(2+3)*4");  // 20
```

---

## Chapter 7: Web Server Implementation

### 7.1 Simple HTTP Server

```cpp
#include <string>
#include <sstream>
#include <functional>
#include <unordered_map>

class HttpServer {
    int port_;
    std::unordered_map<std::string,
        std::function<std::string(const std::string&)>> routes_;

public:
    HttpServer(int port) : port_(port) {}

    void get(const std::string& path,
             std::function<std::string(const std::string&)> handler) {
        routes_[path] = std::move(handler);
    }

    std::string handleRequest(const std::string& method,
                               const std::string& path,
                               const std::string& body) {
        if (method == "GET" && routes_.count(path)) {
            std::string responseBody = routes_[path](body);
            return "HTTP/1.1 200 OK\r\n"
                   "Content-Length: " + std::to_string(responseBody.size()) +
                   "\r\n\r\n" + responseBody;
        }
        return "HTTP/1.1 404 Not Found\r\n\r\n";
    }
};

// Real-world C++ web frameworks:
// - Crow — header-only, Flask-like
// - Drogon — high-performance async
// - Oat++ — zero-copy, coroutine-based
// - Pistache — REST framework
```

---

## Chapter 8: Interview Questions

**Q1: Why is C++ used for game engines?** — Zero-overhead abstractions, no garbage collector (predictable frame times), low-level memory control, SIMD support, direct hardware access.

**Q2: What is ECS architecture?** — Entity Component System: entities are IDs, components are data, systems are logic. Cache-friendly (SoA layout), easy to parallelize, composable.

**Q3: How would you design a low-latency trading system?** — Lock-free data structures, kernel bypass networking, CPU pinning, pre-allocated memory pools, no virtual functions on critical path, SPSC queues between components.

**Q4: Explain Write-Ahead Logging.** — All changes logged to disk before modifying data pages. On crash, replay log from checkpoint. Ensures durability and atomicity.

---

## Chapter 9: Practice & Mini Projects

### Project 1: Mini Game Engine — Implement ECS with movement, collision, and rendering systems.
### Project 2: Key-Value Store — Build an LSM-tree based storage engine with WAL.
### Project 3: HTTP Framework — Build a multi-threaded web server with routing and middleware.

---

## Chapter 10: Mastery Checklist

- [ ] You understand ECS architecture and game loop design
- [ ] You can explain B+ tree, buffer pool, and WAL for databases
- [ ] You know low-latency techniques for trading systems
- [ ] You can implement a simple expression parser/interpreter
- [ ] You understand embedded constraints and CRTP-based design
- [ ] You can build a basic HTTP server in C++

---

*End of Book 19: Real-World Systems Programming*
