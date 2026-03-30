# Book 15: Multithreading and Concurrency

## The Complete Guide to Concurrent and Parallel Programming in C++

---

### **Target Level:** Advanced
### **Prerequisites:** Books 1–14
### **Learning Outcomes:**
By the end of this book, you will:
- Create and manage threads with `std::thread` and `std::jthread`
- Protect shared data with mutexes, locks, and lock-free techniques
- Coordinate threads with condition variables, barriers, and latches
- Use `std::async`, `std::future`, and `std::promise` for task-based parallelism
- Master atomic operations and the C++ memory model
- Understand common pitfalls: data races, deadlocks, priority inversion

---

## Chapter 1: Fundamentals

### 1.1 Threads vs Processes

| Feature | Process | Thread |
|---------|---------|--------|
| Memory | Separate address space | Shared address space |
| Communication | IPC (pipes, sockets, shared memory) | Direct shared memory |
| Creation cost | High | Low |
| Context switch | Expensive | Cheaper |
| Isolation | Full | None (bugs affect all threads) |

**Conceptual Difference:**

A **process** is a fully independent program execution with its own virtual address space, file descriptors, and system resources. The operating system isolates processes from each other — one crashing process cannot corrupt another.

A **thread** is a lightweight unit of execution *within* a process. All threads in a process share the same heap memory, global variables, file descriptors, and code segment. Each thread gets its own **stack** (for local variables and function call frames) and its own **program counter** (so it can execute different code simultaneously). Because threads share memory, they can communicate instantly — but this shared access is also the root cause of every concurrency bug.

```
┌─────────────────────── Process ──────────────────────┐
│                                                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ Thread 1 │  │ Thread 2 │  │ Thread 3 │            │
│  │          │  │          │  │          │            │
│  │ Stack    │  │ Stack    │  │ Stack    │            │
│  │ PC       │  │ PC       │  │ PC       │            │
│  │ Registers│  │ Registers│  │ Registers│            │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘            │
│       │              │              │                  │
│  ┌────┴──────────────┴──────────────┴─────┐           │
│  │         SHARED MEMORY (Heap)           │           │
│  │   Global variables, dynamic objects    │           │
│  └────────────────────────────────────────┘           │
│                                                       │
│  Code segment │ File descriptors │ PID                │
└───────────────────────────────────────────────────────┘
```

### 1.2 Thread Lifecycle

A thread goes through several states during its lifetime:

```
                    ┌──────────────┐
     std::thread()  │              │
   ─────────────────►   CREATED    │
                    │              │
                    └──────┬───────┘
                           │ OS schedules
                           ▼
                    ┌──────────────┐
                    │              │◄──────────── resume/
          ┌────────│   RUNNING    │              notify/
          │        │              │──────────┐   unlock
          │        └──────┬───────┘          │
          │               │                  │
   mutex.lock()     return from         wait()/
   wait()           thread function     lock()/
   sleep_for()            │             sleep()
          │               │                  │
          │        ┌──────▼───────┐          │
          │        │              │          │
          └───────►│   BLOCKED    │◄─────────┘
                   │  (WAITING)   │
                   └──────────────┘
                          
                   ┌──────────────┐
                   │              │
                   │  TERMINATED  │  ← join() collects result
                   │              │
                   └──────────────┘
```

### 1.3 When to Use Threads

- **CPU-bound work:** Parallel computation (math, image processing, physics)
- **I/O-bound work:** Overlapping I/O with computation (network, disk)
- **Responsiveness:** Keep UI responsive while doing background work
- **Don't use when:** Work is trivially small, or shared state makes correctness hard

**The Fundamental Tradeoff:**

Threads exchange *simplicity* for *performance*. Single-threaded code is deterministic — the same inputs always produce the same outputs in the same order. Multi-threaded code introduces **non-determinism**: the OS scheduler decides when each thread runs, and different runs may interleave operations differently. This makes concurrency bugs (data races, deadlocks) extremely hard to reproduce and debug.

**Rule of Thumb:** Use the simplest concurrency model that solves your problem:
1. No concurrency (sequential) — if fast enough
2. `std::async` (task-based) — fire-and-forget parallelism
3. Thread pool — reusable worker threads
4. Raw `std::thread` — full control when needed
5. Lock-free atomics — maximum performance, maximum complexity

---

## Chapter 2: `std::thread`

### 2.1 Creating Threads

```cpp
#include <thread>
#include <iostream>

void task(int id) {
    std::cout << "Thread " << id << " running\n";
}

int main() {
    std::thread t1(task, 1);   // Launch thread with argument
    std::thread t2(task, 2);

    t1.join();   // Wait for t1 to finish
    t2.join();   // Wait for t2 to finish
}
```

### 2.2 join() vs detach()

```cpp
std::thread t(task);

// Option 1: join — block until thread completes
t.join();  // Current thread waits

// Option 2: detach — run independently (daemon thread)
t.detach();  // Thread continues even if main exits
// WARNING: dangling references if thread accesses local variables!

// If neither join() nor detach() is called before thread destructor,
// std::terminate() is called!
```

### 2.3 Passing Arguments

```cpp
// By value:
void byValue(int x);
std::thread t(byValue, 42);

// By reference (use std::ref):
void byRef(int& x);
int val = 0;
std::thread t(byRef, std::ref(val));

// Moving:
void byMove(std::string&& s);
std::thread t(byMove, std::move(str));

// Lambda:
std::thread t([&val]() {
    val = 42;
});
```

### 2.4 `std::jthread` (C++20) — Better Thread

```cpp
#include <thread>
#include <stop_token>

// jthread = joining thread — automatically joins in destructor!
void task(std::stop_token stoken) {
    while (!stoken.stop_requested()) {
        // Do work...
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    std::cout << "Stop requested, cleaning up\n";
}

{
    std::jthread t(task);
    // ... do other work ...
    t.request_stop();  // Signal the thread to stop
}  // Destructor automatically joins — no forgetting!
```

### 2.5 Hardware Concurrency

```cpp
unsigned int cores = std::thread::hardware_concurrency();
// Returns number of concurrent threads supported (typically = CPU cores)
```

---

## Chapter 3: Mutual Exclusion — Mutexes

### 3.1 The Data Race Problem

A **data race** occurs when two or more threads access the same memory location concurrently, at least one of them writes, and there is no synchronization. Data races are **undefined behavior** in C++ — the compiler is allowed to assume they never happen, which means it can optimize in ways that make the bug even worse.

```cpp
int counter = 0;  // Shared resource

void increment() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // DATA RACE! Not atomic!
        // Read counter, increment, write back — can interleave
    }
}

std::thread t1(increment);
std::thread t2(increment);
t1.join(); t2.join();
// counter might be 134527 instead of 200000!
```

**Why does this happen?** The operation `++counter` is NOT a single instruction. It decomposes into three steps:

```
Thread 1                    Thread 2
────────                    ────────
1. READ counter (= 42)      
                            1. READ counter (= 42)
2. INCREMENT (= 43)         
                            2. INCREMENT (= 43)
3. WRITE counter (= 43)     
                            3. WRITE counter (= 43)  ← LOST UPDATE!

Both threads read 42, both write 43.
One increment is completely lost!
```

This is called a **lost update** or **read-modify-write race**. Over 200,000 iterations, thousands of updates can be lost, producing unpredictable results.

### 3.2 `std::mutex`

A **mutex** (mutual exclusion) ensures only one thread enters the **critical section** at a time. Any thread that tries to lock an already-locked mutex will **block** (sleep) until the mutex becomes available.

```
Thread 1                        Thread 2
────────                        ────────
mtx.lock()  → ACQUIRED          
  ++counter (safe)              mtx.lock() → BLOCKED (waiting...)
  ++counter (safe)                        (sleeping...)
mtx.unlock() → RELEASED                  (sleeping...)
                                mtx.lock() → ACQUIRED (wakes up!)
                                  ++counter (safe)
                                mtx.unlock() → RELEASED
```

```cpp
#include <mutex>

std::mutex mtx;
int counter = 0;

void safeIncrement() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();
        ++counter;     // Now this is safe — only one thread at a time
        mtx.unlock();
    }
}
// Problem: if exception thrown between lock/unlock, mutex stays locked forever!
```

**Important:** Mutexes are **not free**. Each lock/unlock involves a kernel syscall (or at least an atomic compare-and-swap). Locking a mutex ~25ns on modern hardware. Don't lock inside tight loops if you can batch operations instead.

### 3.3 `std::lock_guard` — RAII Lock

```cpp
void safeIncrement() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++counter;
    }  // Automatically unlocked when lock goes out of scope
}

// C++17: CTAD simplifies:
std::lock_guard lock(mtx);
```

### 3.4 `std::unique_lock` — Flexible Lock

```cpp
std::unique_lock<std::mutex> lock(mtx);
// Can be:
lock.unlock();              // Manually unlock
lock.lock();                // Re-lock
// Can be constructed without locking:
std::unique_lock<std::mutex> lock2(mtx, std::defer_lock);
// Required for condition_variable::wait()
// Can be moved but not copied
```

### 3.5 `std::scoped_lock` (C++17) — Lock Multiple Mutexes

```cpp
std::mutex m1, m2;

// Locks both m1 and m2 atomically — prevents deadlock:
std::scoped_lock lock(m1, m2);
// Equivalent to std::lock(m1, m2) but RAII
```

### 3.6 Mutex Types

| Mutex | Description |
|-------|-------------|
| `std::mutex` | Basic, non-recursive |
| `std::recursive_mutex` | Same thread can lock multiple times |
| `std::timed_mutex` | try_lock_for(), try_lock_until() |
| `std::shared_mutex` | Multiple readers OR one writer |

### 3.7 `std::shared_mutex` — Reader-Writer Lock (C++17)

```cpp
#include <shared_mutex>

std::shared_mutex rwMutex;
std::map<std::string, int> cache;

int read(const std::string& key) {
    std::shared_lock lock(rwMutex);  // Multiple readers OK
    return cache[key];
}

void write(const std::string& key, int value) {
    std::unique_lock lock(rwMutex);  // Exclusive access
    cache[key] = value;
}
```

---

## Chapter 4: Condition Variables

### 4.1 Producer-Consumer Pattern

The **producer-consumer** pattern is one of the most important concurrency patterns. Producers generate data and place it in a shared buffer; consumers take data from the buffer and process it. A condition variable coordinates them so consumers sleep when the buffer is empty and wake up when data arrives.

```
┌──────────┐    push()     ┌────────────────────┐    pop()      ┌──────────┐
│ PRODUCER │ ─────────► │   SHARED BUFFER    │ ─────────► │ CONSUMER │
│          │  notify_one()  │   (std::queue)     │   (blocks    │          │
│ Thread A │    ┌──────►  │  ┌──┬──┬──┬──┬──┐  │   if empty) │ Thread B │
└──────────┘    │        │  │3 │5 │1 │  │  │  │            └──────────┘
                 │        │  └──┴──┴──┴──┴──┘  │
                 │        └────────────────────┘
    condition_variable      Protected by mutex
```

**How `cv.wait()` works internally (step-by-step):**
1. Caller must hold the mutex (via `unique_lock`)
2. `wait()` **atomically** unlocks the mutex and puts the thread to sleep
3. When another thread calls `notify_one()` or `notify_all()`, the sleeping thread wakes up
4. `wait()` **re-locks the mutex** before returning
5. The predicate is checked — if false (spurious wakeup), go back to sleep

```cpp
#include <queue>
#include <mutex>
#include <condition_variable>

std::queue<int> buffer;
std::mutex mtx;
std::condition_variable cv;
bool done = false;

void producer() {
    for (int i = 0; i < 10; ++i) {
        {
            std::lock_guard lock(mtx);
            buffer.push(i);
        }
        cv.notify_one();  // Wake one waiting consumer
    }
    {
        std::lock_guard lock(mtx);
        done = true;
    }
    cv.notify_all();  // Wake all consumers to check done flag
}

void consumer() {
    while (true) {
        std::unique_lock lock(mtx);
        cv.wait(lock, [&] {
            return !buffer.empty() || done;
        });
        // wait() atomically: unlocks mutex → sleeps →
        // wakes on notify → re-locks mutex → checks predicate

        if (buffer.empty() && done) break;

        int val = buffer.front();
        buffer.pop();
        lock.unlock();

        std::cout << "Consumed: " << val << "\n";
    }
}
```

### 4.2 Spurious Wakeups

Always use the predicate form of `wait()`:
```cpp
cv.wait(lock, [&]{ return ready; });
// NOT:
cv.wait(lock);  // May wake up without notification!
```

---

## Chapter 5: Futures and Promises

### 5.1 `std::async` — Simplest Task-Based Parallelism

`std::async` is the highest-level concurrency tool in C++. It launches a function asynchronously (optionally in a new thread) and returns a `std::future` that will hold the result.

```
 Main Thread                    Worker Thread
 ───────────                    ─────────────

 auto f = async(compute, 42)
    │                           ┌────────────┐
    │   (launches thread)──────►│  compute() │
    │                           │  running...│
    │ (does other work)         │            │
    │                           │  result=   │
    ├───────────────            │  1764      │
    │ f.get()                   └───┬────────┘
    │  (blocks until ready)         │
    │◄────── result = 1764 ───────┘
    │
    ▼ uses result (1764)
```

```cpp
#include <future>

int compute(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return x * x;
}

// Launch asynchronously:
auto future = std::async(std::launch::async, compute, 42);

// Do other work while compute runs...

int result = future.get();  // Block until result ready (1764)
// get() can only be called ONCE!
```

### 5.2 Launch Policies

```cpp
// std::launch::async    — Always launches a new thread
// std::launch::deferred — Lazy: runs when get() is called (on calling thread)
// Default: async | deferred — implementation decides

auto f1 = std::async(std::launch::async, task);     // New thread
auto f2 = std::async(std::launch::deferred, task);  // Lazy (no thread)
```

### 5.3 `std::promise` and `std::future` — Manual Channel

While `std::async` creates the thread for you, `promise`/`future` let you establish a **one-shot communication channel** between any two threads:

```
 Thread A (Producer)              Thread B (Consumer)
 ────────────────────              ────────────────────

 promise<int> prom               future<int> fut = prom.get_future()
                                
 ┌────────────────┐                      │
 │ Computing...   │               fut.get() ─ BLOCKS
 │                │                      │
 │ result = 42    │                      │ (sleeping...)
 └────────┬───────┘                      │
          │                              │
 prom.set_value(42)                       │
          │                              │
          └─────── shared state ───────►│
                                          │
                                   UNBLOCKS, returns 42
```

```cpp
std::promise<int> prom;
std::future<int> fut = prom.get_future();

std::thread t([&prom]() {
    int result = heavyComputation();
    prom.set_value(result);  // Send result to future
});

int result = fut.get();  // Receive result (blocks until set)
t.join();

// Exception through promise:
try {
    prom.set_exception(std::make_exception_ptr(
        std::runtime_error("failed")
    ));
} catch (...) {}

// At fut.get(): exception is rethrown
```

### 5.4 `std::shared_future` — Multiple Consumers

```cpp
std::promise<int> prom;
std::shared_future<int> sfut = prom.get_future().share();

// Multiple threads can call sfut.get() — each gets the same value
std::thread t1([sfut]() { std::cout << sfut.get(); });
std::thread t2([sfut]() { std::cout << sfut.get(); });
```

### 5.5 `std::packaged_task`

```cpp
std::packaged_task<int(int, int)> task([](int a, int b) {
    return a + b;
});

auto future = task.get_future();
std::thread t(std::move(task), 3, 4);  // Run task in thread
int result = future.get();  // 7
t.join();
```

---

## Chapter 6: Atomic Operations

### 6.1 `std::atomic` — Lock-Free Shared Variables

```cpp
#include <atomic>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // Atomic! No mutex needed
        // Internally uses hardware atomic instructions (lock xadd on x86)
    }
}

// Other operations:
counter.store(42);                    // Atomic write
int val = counter.load();             // Atomic read
int old = counter.exchange(100);      // Swap, return old
bool ok = counter.compare_exchange_strong(expected, desired);
counter.fetch_add(1);                 // Atomic add
counter.fetch_sub(1);                 // Atomic subtract
```

### 6.2 `std::atomic_flag` — The Simplest Atomic (Spinlock)

```cpp
std::atomic_flag spinlock = ATOMIC_FLAG_INIT;

void lock() {
    while (spinlock.test_and_set(std::memory_order_acquire)) {
        // Spin (busy-wait)
    }
}

void unlock() {
    spinlock.clear(std::memory_order_release);
}
```

### 6.3 Memory Ordering

Memory ordering controls how atomic operations interact with non-atomic memory accesses in other threads. This is one of the most advanced — and most misunderstood — topics in C++.

**Why does this matter?** Modern CPUs and compilers **reorder** instructions for performance. In single-threaded code, this is invisible. In multi-threaded code, it can cause one thread to "see" another thread's operations in a different order than they were written.

```
Ordering Strength (weakest → strongest):

  relaxed        acquire/release       seq_cst
  ────────       ───────────────      ───────
  Fastest        Balanced              Safest
  No ordering    Paired sync           Total order
  (counters)     (producer-consumer)   (default)
```

**Acquire-Release explained visually:**

```
Thread A (Producer)                Thread B (Consumer)
───────────────────                ───────────────────

  data = 42;               │       (these operations
  moreData = 99;           │        are guaranteed visible
  ────── RELEASE fence ────┤        after the acquire)
  flag.store(true,         │
    memory_order_release)  │
                           │
                           │  flag.load(memory_order_acquire)
                           ├───── ACQUIRE fence ─────
                           │  assert(data == 42);     // GUARANTEED!
                           │  assert(moreData == 99); // GUARANTEED!
```

Everything written **before** a release-store is visible to any thread that does an acquire-load and **sees the stored value**.

```cpp
// From weakest to strongest:
std::memory_order_relaxed    // No ordering guarantees (fastest)
std::memory_order_acquire    // Reads after this see all writes before release
std::memory_order_release    // Writes before this are visible after acquire
std::memory_order_acq_rel   // Both acquire and release
std::memory_order_seq_cst   // Total ordering (default, safest, slowest)

// Example: Producer-Consumer with atomics
std::atomic<bool> ready{false};
int data = 0;

void producer() {
    data = 42;                                          // Non-atomic write
    ready.store(true, std::memory_order_release);       // Release
}

void consumer() {
    while (!ready.load(std::memory_order_acquire)) {}   // Acquire
    assert(data == 42);  // Guaranteed! Release-acquire synchronized
}
```

### 6.4 `std::atomic<std::shared_ptr>` (C++20)

```cpp
// Atomic operations on shared_ptr:
std::atomic<std::shared_ptr<Config>> globalConfig;

void updateConfig() {
    auto newConfig = std::make_shared<Config>(loadFromFile());
    globalConfig.store(newConfig);
}

void useConfig() {
    auto config = globalConfig.load();
    // Use config safely
}
```

---

## Chapter 7: C++20 Synchronization

### 7.1 `std::latch` — One-Time Barrier

```cpp
#include <latch>

std::latch startSignal(1);       // Count = 1
std::latch doneSignal(numThreads);  // Count = numThreads

void worker() {
    startSignal.wait();          // Wait for start signal
    // Do work...
    doneSignal.count_down();     // Signal completion
}

// In main:
startSignal.count_down();        // Release all workers
doneSignal.wait();               // Wait for all to finish
```

### 7.2 `std::barrier` — Reusable Barrier

```cpp
#include <barrier>

auto onPhaseComplete = []() noexcept {
    std::cout << "Phase complete\n";
};

std::barrier sync_point(numThreads, onPhaseComplete);

void worker() {
    for (int phase = 0; phase < 3; ++phase) {
        // Do phase work...
        sync_point.arrive_and_wait();  // Synchronize
    }
}
```

### 7.3 `std::counting_semaphore`

```cpp
#include <semaphore>

std::counting_semaphore<5> semaphore(5);  // Max 5 concurrent accesses

void accessResource() {
    semaphore.acquire();     // Decrement (blocks if 0)
    // Use limited resource...
    semaphore.release();     // Increment
}

// Binary semaphore:
std::binary_semaphore gate(0);
// producer: gate.release();   // Signal
// consumer: gate.acquire();   // Wait for signal
```

---

## Chapter 8: Common Concurrency Pitfalls

### 8.1 Deadlock

A **deadlock** occurs when two or more threads are each waiting for a resource held by the other, creating a circular dependency where none can proceed.

**The Classic Deadlock Scenario:**

```
  Thread 1                          Thread 2
  ────────                          ────────
            ┌────────┐
  LOCKS ───►│ Mutex A │
            └────────┘
                                    ┌────────┐
                          LOCKS ───►│ Mutex B │
                                    └────────┘

  Now wants     ┌────────┐
  Mutex B ───►  │ BLOCKED│ (B is held by Thread 2)
                └────────┘
                      ┌────────┐
  Now wants  ◄───────│ BLOCKED│ (A is held by Thread 1)
  Mutex A              └────────┘

  ╱╱╱╱╱╱╱╱╱ DEADLOCK! Neither thread can proceed ╱╱╱╱╱╱╱╱╱
```

**Four conditions must ALL be true for deadlock (Coffman Conditions):**
1. **Mutual exclusion** — resources are non-shareable
2. **Hold and wait** — thread holds one resource while waiting for another
3. **No preemption** — resources can't be forcibly taken
4. **Circular wait** — circular chain of threads waiting for each other

Breaking ANY one condition prevents deadlock.

```cpp
// DEADLOCK: Thread 1 locks A then B, Thread 2 locks B then A
std::mutex m1, m2;

void thread1() {
    std::lock_guard l1(m1);  // Lock A
    std::lock_guard l2(m2);  // Wait for B → DEADLOCK if thread2 has B
}
void thread2() {
    std::lock_guard l1(m2);  // Lock B
    std::lock_guard l2(m1);  // Wait for A → DEADLOCK if thread1 has A
}

// SOLUTION 1: Always lock in same order
// SOLUTION 2: Use std::scoped_lock (atomically locks both)
std::scoped_lock lock(m1, m2);
```

### 8.2 Data Race

```cpp
// A data race = two threads access the same non-atomic variable,
// at least one writes, and there's no synchronization.
// DATA RACES ARE UNDEFINED BEHAVIOR.
```

### 8.3 Race Condition (Logic Error)

```cpp
// Even with proper synchronization, order of operations may not be what you expect:
if (queue.empty()) {     // Thread A checks
                          // Thread B adds item
    // Queue is no longer empty, but thread A thinks it is!
    return;
}
auto item = queue.front(); // Another thread may have taken it!
```

### 8.4 Priority Inversion

Low-priority thread holds a lock needed by high-priority thread. Medium-priority thread preempts low-priority thread, delaying the high-priority thread indirectly.

---

## Chapter 9: Thread Pool Design

A **thread pool** creates a fixed number of worker threads that pull tasks from a shared queue. This avoids the overhead of creating/destroying threads for each task.

```
                    ┌────────────────────────────────────┐
 submit(task)  ───►  │        TASK QUEUE             │
 submit(task)  ───►  │  ┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐      │
 submit(task)  ───►  │  │T6││T5││T4││T3││T2││T1│ ───► │
                    │  └──┘└──┘└──┘└──┘└──┘└──┘      │
                    │  Protected by mutex + cond_var  │
                    └─────────────┬──────────────────────┘
                              │
               ┌─────────────┼─────────────┐
               │              │              │
        ┌──────┴───┐  ┌──────┴───┐  ┌──────┴───┐
        │  Worker 1  │  │  Worker 2  │  │  Worker 3  │
        │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │
        │ │running │ │  │ │running │ │  │ │ idle   │ │
        │ │ T1     │ │  │ │ T2     │ │  │ │ (wait) │ │
        │ └────────┘ │  │ └────────┘ │  │ └────────┘ │
        └────────────┘  └────────────┘  └────────────┘
        Long-lived threads that sleep when idle
        and wake up when tasks are available.
```

**Why use a thread pool instead of creating threads on-demand?**
- Thread creation takes ~20µs on Linux (~100µs on Windows) — unacceptable for small tasks
- Creating thousands of threads wastes memory (each thread stack: ~1-8 MB)
- Thread pool amortizes creation cost, controls max parallelism, and reuses threads
- `std::thread::hardware_concurrency()` workers is the typical pool size

```cpp
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mtx_;
    std::condition_variable cv_;
    bool stop_ = false;

public:
    ThreadPool(size_t numThreads) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers_.emplace_back([this]() {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock lock(mtx_);
                        cv_.wait(lock, [this] {
                            return stop_ || !tasks_.empty();
                        });
                        if (stop_ && tasks_.empty()) return;
                        task = std::move(tasks_.front());
                        tasks_.pop();
                    }
                    task();
                }
            });
        }
    }

    template<typename F, typename... Args>
    auto submit(F&& f, Args&&... args) {
        using ReturnType = std::invoke_result_t<F, Args...>;
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        auto future = task->get_future();
        {
            std::lock_guard lock(mtx_);
            tasks_.emplace([task]() { (*task)(); });
        }
        cv_.notify_one();
        return future;
    }

    ~ThreadPool() {
        {
            std::lock_guard lock(mtx_);
            stop_ = true;
        }
        cv_.notify_all();
        for (auto& w : workers_) w.join();
    }
};

// Usage:
ThreadPool pool(4);
auto f1 = pool.submit(compute, 42);
auto f2 = pool.submit(compute, 100);
std::cout << f1.get() + f2.get();
```

---

## Chapter 10: Parallel Algorithms (C++17)

C++17 introduced **execution policies** that let you parallelize standard algorithms with a single extra parameter. The compiler and runtime library handle all thread management.

```
 Sequential             Parallel                Parallel + SIMD
 (default)              (std::execution::par)   (std::execution::par_unseq)

 ┌──────────┐         ┌──────────┐        ┌──────────┐
 │ Thread 1 │         │ Thread 1 │        │ Thread 1 │
 │ [======] │         │ [==]     │        │ [=SIMD=] │
 │          │         └──────────┘        └──────────┘
 │          │         ┌──────────┐        ┌──────────┐
 │          │         │ Thread 2 │        │ Thread 2 │
 │          │         │ [==]     │        │ [=SIMD=] │
 │          │         └──────────┘        └──────────┘
 │          │         ┌──────────┐        ┌──────────┐
 │          │         │ Thread 3 │        │ Thread 3 │
 │          │         │ [==]     │        │ [=SIMD=] │
 └──────────┘         └──────────┘        └──────────┘
 One thread,            Multiple threads,       Multiple threads +
 all work               work divided            vectorized (SSE/AVX)
```

```cpp
#include <algorithm>
#include <execution>

std::vector<int> v(1'000'000);

// Sequential (default):
std::sort(v.begin(), v.end());

// Parallel:
std::sort(std::execution::par, v.begin(), v.end());

// Parallel + vectorized:
std::sort(std::execution::par_unseq, v.begin(), v.end());

// Unsequenced (vectorized, same thread):
std::sort(std::execution::unseq, v.begin(), v.end());  // C++20

// Other parallel algorithms:
std::for_each(std::execution::par, v.begin(), v.end(), process);
std::transform(std::execution::par, v.begin(), v.end(), out.begin(), func);
std::reduce(std::execution::par, v.begin(), v.end());  // Parallel sum
std::transform_reduce(std::execution::par, v.begin(), v.end(), 0, plus, square);
```

---

## Chapter 11: Interview Questions

**Q1: What is a data race? How do you prevent one?** — Two threads access the same non-atomic variable with at least one write and no synchronization. Prevent with mutexes, atomics, or by avoiding shared mutable state.

**Q2: Explain the difference between `join()` and `detach()`.** — `join()` blocks the calling thread until the target thread finishes. `detach()` lets the thread run independently; its resources are reclaimed when it finishes.

**Q3: What is a deadlock? How do you prevent it?** — Two or more threads each hold a lock and wait for the other's lock. Prevent by: consistent lock ordering, `std::scoped_lock`, lock hierarchies, or lock-free programming.

**Q4: What is the difference between `mutex` and `atomic`?** — `mutex` protects a critical section (arbitrary code). `atomic` provides lock-free operations on a single variable. Atomics are faster for simple variables; mutexes are needed for compound operations.

**Q5: When would you use `std::async` vs `std::thread`?** — `std::async` is simpler: launches a task and returns a future. `std::thread` gives full control. Use `async` for fire-and-forget computation; use `thread` when you need thread management (detach, thread pools, etc.).

---

## Chapter 12: Practice & Mini Projects

### Project 1: Thread-Safe Queue — Implement using mutex + condition variable with push/pop/try_pop.
### Project 2: Parallel Matrix Multiply — Divide work across threads for large matrices.
### Project 3: Web Scraper — Use a thread pool with futures to download and parse multiple URLs concurrently.

---

## Chapter 13: Mastery Checklist

- [ ] You can create, join, and detach threads
- [ ] You understand mutex, lock_guard, unique_lock, scoped_lock
- [ ] You can implement producer-consumer with condition variables
- [ ] You can use std::async, future, promise, packaged_task
- [ ] You understand atomic operations and memory ordering
- [ ] You can identify and fix deadlocks and data races
- [ ] You can design a thread pool
- [ ] You know C++20 jthread, latch, barrier, semaphore
- [ ] You can use parallel STL algorithms

---

*End of Book 15: Multithreading and Concurrency*
