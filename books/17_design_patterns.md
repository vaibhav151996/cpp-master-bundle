# Book 17: Design Patterns in C++

## The Complete Guide to Software Design Patterns with Modern C++ Implementation

---

### **Target Level:** Advanced → Expert
### **Prerequisites:** Books 1–16
### **Learning Outcomes:**
By the end of this book, you will:
- Understand and implement all 23 Gang of Four (GoF) design patterns
- Know when and why to apply each pattern
- Implement patterns using modern C++ (smart pointers, templates, lambdas)
- Recognize anti-patterns and pattern misuse
- Apply SOLID principles to design decisions

---

## Chapter 1: SOLID Principles

The five SOLID principles are the foundation of good object-oriented design. They guide you toward code that is easy to maintain, extend, and test.

| Principle | Meaning |
|-----------|--------|
| **S** — Single Responsibility | A class should have one reason to change |
| **O** — Open/Closed | Open for extension, closed for modification |
| **L** — Liskov Substitution | Derived classes must be substitutable for base classes |
| **I** — Interface Segregation | Prefer many specific interfaces over one general one |
| **D** — Dependency Inversion | Depend on abstractions, not concretions |

**Single Responsibility — detailed:**

A class that does too many things is fragile. If you change one responsibility, you risk breaking the others.

```
❌ BAD: class UserManager                ✔ GOOD: separate classes
  ┌──────────────────────┐              ┌─────────────┐  ┌──────────────┐
  │ createUser()         │              │ UserService │  │ UserValidator│
  │ validateEmail()      │              │ createUser()│  │ validate()   │
  │ sendWelcomeEmail()   │              └─────────────┘  └──────────────┘
  │ saveToDatabase()     │              ┌─────────────┐  ┌──────────────┐
  │ generateReport()     │              │ EmailSender │  │ UserRepo     │
  └──────────────────────┘              │ sendEmail() │  │ save()       │
  5 reasons to change!                  └─────────────┘  └──────────────┘
                                        Each class has ONE job.
```

**Dependency Inversion — detailed:**

High-level modules should not depend on low-level modules. Both should depend on **abstractions** (interfaces).

```
❌ BAD:                                  ✔ GOOD:

  OrderService                           OrderService
       │                                      │
       ▼ depends on concrete               ▼ depends on interface
  MySQLDatabase                          IDatabase (abstract)
                                          /        \
                                   MySQLDB       PostgresDB

Now you can swap MySQL for Postgres without changing OrderService.
```

### Pattern Categories Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    DESIGN PATTERNS                       │
├───────────────────┬───────────────────┬───────────────────┤
│   CREATIONAL        │   STRUCTURAL        │    BEHAVIORAL        │
│   (Object Creation) │   (Composition)      │    (Communication)   │
│                     │                     │                     │
│  • Singleton        │  • Adapter           │  • Observer          │
│  • Factory Method   │  • Decorator         │  • Strategy          │
│  • Abstract Factory │  • Proxy             │  • Command           │
│  • Builder          │  • Facade            │  • State             │
│  • Prototype        │  • Composite         │  • Template Method   │
│                     │  • Bridge            │  • Iterator          │
│                     │  • Flyweight         │  • Chain of Resp.    │
│                     │                     │  • Mediator          │
│                     │                     │  • Memento           │
│                     │                     │  • Visitor           │
└───────────────────┴───────────────────┴───────────────────┘

  How to choose:
  • Need to control object creation?     → Creational
  • Need to compose objects/interfaces?  → Structural
  • Need to manage object interaction?   → Behavioral
```

---

## Chapter 2: Creational Patterns

### 2.1 Singleton — One Global Instance

```
Singleton Structure:

┌──────────────────────────┐
│         Logger             │
├──────────────────────────┤
│ - Logger() [private]       │    ← Constructor is PRIVATE
│ - instance_: Logger        │    ← Single static instance
├──────────────────────────┤
│ + instance(): Logger&      │    ← Public access point
│ + log(msg): void           │
└──────────────────────────┘

         ^          ^          ^
     classA     classB     classC
     calls      calls      calls
   instance() instance() instance()
         \        |        /
          \       |       /
     All get the SAME object
```

```cpp
class Logger {
    Logger() = default;
public:
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;

    static Logger& instance() {
        static Logger logger;  // Thread-safe since C++11 (Meyers' Singleton)
        return logger;
    }

    void log(const std::string& msg) {
        std::cout << "[LOG] " << msg << "\n";
    }
};

// Usage:
Logger::instance().log("App started");
```

**When to use:** Logging, configuration, thread pool, database connection pool.
**Caution:** Hidden dependencies, difficult to test. Consider dependency injection instead.

### 2.2 Factory Method — Deferred Object Creation

The Factory pattern **decouples object creation from usage**. The caller doesn't need to know which concrete class is instantiated — it just works with the interface.

```
Factory Method Structure:

                 ┌──────────────┐
                 │   Button     │  ← Abstract Product (interface)
                 │  + render()  │
                 └──────┬───────┘
                       │
              ┌────────┴────────┐
              │                  │
   ┌──────────┴────┐  ┌─────┴─────────┐
   │ WindowsButton  │  │ LinuxButton     │  ← Concrete Products
   │ + render()     │  │ + render()      │
   └───────────────┘  └───────────────┘

   createButton("windows")  ──►  returns unique_ptr<WindowsButton>
   createButton("linux")    ──►  returns unique_ptr<LinuxButton>

   Client code just uses Button* — doesn't know/care which type!
```

```cpp
// Product interface:
class Button {
public:
    virtual ~Button() = default;
    virtual void render() = 0;
};

class WindowsButton : public Button {
public:
    void render() override { std::cout << "Windows button\n"; }
};

class LinuxButton : public Button {
public:
    void render() override { std::cout << "Linux button\n"; }
};

// Factory:
std::unique_ptr<Button> createButton(const std::string& os) {
    if (os == "windows") return std::make_unique<WindowsButton>();
    if (os == "linux")   return std::make_unique<LinuxButton>();
    throw std::invalid_argument("Unknown OS");
}

// Usage:
auto btn = createButton("windows");
btn->render();
```

### 2.3 Abstract Factory — Family of Related Objects

```cpp
// Families of products:
class ScrollBar { public: virtual ~ScrollBar() = default; };
class WindowsScrollBar : public ScrollBar {};
class LinuxScrollBar : public ScrollBar {};

// Abstract Factory:
class GUIFactory {
public:
    virtual ~GUIFactory() = default;
    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<ScrollBar> createScrollBar() = 0;
};

class WindowsFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<WindowsButton>();
    }
    std::unique_ptr<ScrollBar> createScrollBar() override {
        return std::make_unique<WindowsScrollBar>();
    }
};
```

### 2.4 Builder — Step-by-Step Complex Object Construction

The Builder pattern constructs complex objects **step by step** through a fluent API. Unlike a constructor with many parameters, the Builder makes the construction process readable and flexible.

```
Builder Pattern Flow:

  Query::Builder()           Returns Builder&
       │                     at each step (chaining)
       ▼
  .from("users")   ─────►  Sets table_
       │
  .where("age>18") ─────►  Adds to conditions_[]
       │
  .where("active") ─────►  Adds to conditions_[]
       │
  .orderBy("name") ─────►  Sets orderBy_
       │
  .limit(10)       ─────►  Sets limit_
       │
  .build()         ─────►  Returns completed Query object
                           "SELECT * FROM users
                            WHERE age>18 AND active
                            ORDER BY name LIMIT 10"
```

**When to use:** Objects with many optional parameters, complex construction sequences, or when you want to build different representations using the same process.

```cpp
class Query {
    std::string table_;
    std::vector<std::string> conditions_;
    std::string orderBy_;
    int limit_ = -1;

public:
    class Builder {
        Query q;
    public:
        Builder& from(std::string table) {
            q.table_ = std::move(table);
            return *this;
        }
        Builder& where(std::string cond) {
            q.conditions_.push_back(std::move(cond));
            return *this;
        }
        Builder& orderBy(std::string col) {
            q.orderBy_ = std::move(col);
            return *this;
        }
        Builder& limit(int n) {
            q.limit_ = n;
            return *this;
        }
        Query build() { return std::move(q); }
    };

    std::string toSQL() const {
        std::string sql = "SELECT * FROM " + table_;
        if (!conditions_.empty()) {
            sql += " WHERE ";
            for (size_t i = 0; i < conditions_.size(); ++i) {
                if (i > 0) sql += " AND ";
                sql += conditions_[i];
            }
        }
        if (!orderBy_.empty()) sql += " ORDER BY " + orderBy_;
        if (limit_ >= 0) sql += " LIMIT " + std::to_string(limit_);
        return sql;
    }
};

// Usage (fluent API):
auto query = Query::Builder()
    .from("users")
    .where("age > 18")
    .where("active = 1")
    .orderBy("name")
    .limit(10)
    .build();
```

### 2.5 Prototype — Clone Existing Objects

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual void draw() const = 0;
};

class Circle : public Shape {
    double radius_;
public:
    Circle(double r) : radius_(r) {}
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
    void draw() const override {
        std::cout << "Circle(r=" << radius_ << ")\n";
    }
};
```

---

## Chapter 3: Structural Patterns

### 3.1 Adapter — Make Incompatible Interfaces Work Together

```cpp
// Existing class (can't modify):
class LegacyPrinter {
public:
    void printOldWay(const std::string& text) {
        std::cout << "*** " << text << " ***\n";
    }
};

// Target interface:
class Printer {
public:
    virtual ~Printer() = default;
    virtual void print(const std::string& text) = 0;
};

// Adapter:
class PrinterAdapter : public Printer {
    LegacyPrinter legacy_;
public:
    void print(const std::string& text) override {
        legacy_.printOldWay(text);
    }
};
```

### 3.2 Decorator — Add Behavior Dynamically

The Decorator pattern wraps objects to add behavior **without modifying the original class**. Decorators are composable — you can stack them like layers.

```
Decorator Wrapping (Russian Doll Structure):

  Before: stream->write("Hello")
  ┌────────────┐
  │ FileStream │ ───► writes "Hello" to file
  └────────────┘

  After: decorating with encryption + compression
  ┌────────────────────────────────────────────────┐
  │ CompressedStream                                │
  │  ┌────────────────────────────────────────┐  │
  │  │ EncryptedStream                          │  │
  │  │  ┌────────────────────────────────┐  │  │
  │  │  │ FileStream (actual I/O)       │  │  │
  │  │  └────────────────────────────────┘  │  │
  │  └────────────────────────────────────────┘  │
  └────────────────────────────────────────────────┘

  Call flow:  write("Hello")
    → CompressedStream::write()  → compresses data
      → EncryptedStream::write() → encrypts data
        → FileStream::write()    → writes to disk

  Each decorator adds ONE behavior, delegates to the wrapped object.
  You can mix & match: Encrypted-only, Compressed-only, both, neither.
```

```cpp
class Stream {
public:
    virtual ~Stream() = default;
    virtual void write(const std::string& data) = 0;
};

class FileStream : public Stream {
public:
    void write(const std::string& data) override {
        std::cout << "Writing to file: " << data << "\n";
    }
};

// Decorator base:
class StreamDecorator : public Stream {
protected:
    std::unique_ptr<Stream> wrapped_;
public:
    StreamDecorator(std::unique_ptr<Stream> s) : wrapped_(std::move(s)) {}
};

class EncryptedStream : public StreamDecorator {
public:
    using StreamDecorator::StreamDecorator;
    void write(const std::string& data) override {
        std::string encrypted = encrypt(data);
        wrapped_->write(encrypted);
    }
private:
    std::string encrypt(const std::string& s) {
        std::string result = s;
        for (char& c : result) c ^= 0x5A;
        return result;
    }
};

class CompressedStream : public StreamDecorator {
public:
    using StreamDecorator::StreamDecorator;
    void write(const std::string& data) override {
        std::string compressed = compress(data);
        wrapped_->write(compressed);
    }
private:
    std::string compress(const std::string& s) { return s; /* placeholder */ }
};

// Usage — compose decorators:
auto stream = std::make_unique<CompressedStream>(
    std::make_unique<EncryptedStream>(
        std::make_unique<FileStream>()
    )
);
stream->write("Hello");  // Compressed → Encrypted → Written to file
```

### 3.3 Observer — Event Notification System

The Observer pattern defines a **one-to-many dependency**: when one object (the subject) changes state, all its dependents (observers) are notified automatically. This is the foundation of event-driven programming.

```
Observer Pattern Flow:

  ┌───────────────┐
  │  EventEmitter  │ (Subject)
  │  "data" event  │
  └─────┬───┬─────┘
        │     │
   emit(│"data│", "Hello")
        │     │
   ┌────▼─┐  ┌▼──────┐  ┌─────────┐
   │Logger │  │Display │  │Analytics│  (Observers)
   │handler│  │handler │  │handler  │
   └───────┘  └────────┘  └─────────┘

   All registered handlers are called.
   Adding/removing observers doesn't change the emitter.
   → Real-world: GUI events, pub/sub, reactive programming
```

```cpp
#include <functional>

class EventEmitter {
    std::unordered_map<std::string, std::vector<std::function<void(const std::string&)>>> listeners_;

public:
    void on(const std::string& event, std::function<void(const std::string&)> handler) {
        listeners_[event].push_back(std::move(handler));
    }

    void emit(const std::string& event, const std::string& data = "") {
        if (listeners_.count(event)) {
            for (auto& handler : listeners_[event]) {
                handler(data);
            }
        }
    }
};

// Usage:
EventEmitter emitter;
emitter.on("data", [](const std::string& d) {
    std::cout << "Received: " << d << "\n";
});
emitter.emit("data", "Hello!");
```

### 3.4 Strategy — Interchangeable Algorithms

The Strategy pattern lets you **swap algorithms at runtime** without changing the code that uses them. In modern C++, `std::function` + lambdas often replace the traditional class hierarchy approach.

```
Strategy Pattern:

  ┌──────────────┐
  │    Sorter     │ (Context)
  │              │         setStrategy()
  │  strategy_ ──┼──────────────────────────┐
  │  sort()      │                          │
  └──────────────┘                          ▼
                             ┌──────────────────────────┐
                             │   Sorting Strategy       │
                             │   (std::function)        │
                             └──────────────────────────┘
                             Can be swapped to ANY of:
                             ┌─────────┐ ┌─────────────┐ ┌───────────┐
                             │QuickSort│ │MergeSort    │ │RadixSort  │
                             │ lambda  │ │ lambda      │ │ lambda    │
                             └─────────┘ └─────────────┘ └───────────┘

  Unlike inheritance, you swap behavior at RUNTIME with a single line.
  Modern C++ advantage: no need for an abstract Strategy base class.
```

```cpp
// Modern C++ approach — use std::function instead of class hierarchy:
class Sorter {
    std::function<void(std::vector<int>&)> strategy_;

public:
    void setStrategy(std::function<void(std::vector<int>&)> s) {
        strategy_ = std::move(s);
    }

    void sort(std::vector<int>& data) {
        if (strategy_) strategy_(data);
    }
};

// Usage:
Sorter sorter;
sorter.setStrategy([](std::vector<int>& v) {
    std::sort(v.begin(), v.end());     // Quick sort
});
sorter.sort(data);

sorter.setStrategy([](std::vector<int>& v) {
    std::stable_sort(v.begin(), v.end());  // Merge sort
});
sorter.sort(data);
```

### 3.5 Proxy — Controlled Access

A Proxy wraps another object to control access to it. Common types: **lazy proxy** (defer expensive creation), **protection proxy** (access control), **remote proxy** (network access), **caching proxy** (memoize results).

```
Proxy Pattern — Lazy Loading:

  Without proxy:                     With LazyImage proxy:
  ┌──────────┐  creates immediately  ┌──────────────┐ creates only on
  │RealImage │── loads 50MB from ──  │ LazyImage    │ first display()
  │          │   disk at startup     │ ┌──────────┐ │
  └──────────┘                       │ │RealImage │ │ ← null until needed
                                     │ └──────────┘ │
  Even if never displayed!           └──────────────┘
                                     Saves memory + startup time
```

```cpp
class Image {
public:
    virtual ~Image() = default;
    virtual void display() = 0;
};

class RealImage : public Image {
    std::string filename_;
public:
    RealImage(std::string fn) : filename_(std::move(fn)) {
        std::cout << "Loading " << filename_ << " from disk...\n";
    }
    void display() override {
        std::cout << "Displaying " << filename_ << "\n";
    }
};

class LazyImage : public Image {     // Proxy
    std::string filename_;
    std::unique_ptr<RealImage> real_;
public:
    LazyImage(std::string fn) : filename_(std::move(fn)) {}
    void display() override {
        if (!real_) real_ = std::make_unique<RealImage>(filename_);
        real_->display();
    }
};
// Image not loaded from disk until display() is actually called
```

### 3.6 Facade — Simplified Interface to Complex Subsystem

A Facade provides a **simple interface** to a complex subsystem. Instead of the client managing multiple classes and their interactions, the Facade coordinates everything behind one clean method.

```
Facade Pattern:

  Without Facade:                    With Facade:
  Client must know all subsystems    Client uses ONE simple method

  Client ──┬──► AudioDecoder         Client ──► MediaFacade.playMovie()
           ├──► VideoDecoder                         │
           ├──► SubtitleParser                       ▼
           └──► Renderer                ┌────────────────────┐
                                        │   MediaFacade      │
  Client code is COUPLED to 4 classes   │                    │
  and must know the correct order.      │  AudioDecoder ───┐ │
                                        │  VideoDecoder ───┤ │
                                        │  SubtitleParser ─┤ │
                                        │  Renderer ───────┘ │
                                        └────────────────────┘
                                        Complexity is HIDDEN.
```

```cpp
class MediaFacade {
    AudioDecoder audio_;
    VideoDecoder video_;
    SubtitleParser subs_;
    Renderer renderer_;

public:
    void playMovie(const std::string& file) {
        auto audioStream = audio_.decode(file);
        auto videoStream = video_.decode(file);
        auto subtitles = subs_.parse(file);
        renderer_.render(videoStream, audioStream, subtitles);
    }
};

// User just calls:
MediaFacade player;
player.playMovie("movie.mp4");
```

### 3.7 Composite — Tree Structures

The Composite pattern lets you treat **individual objects and groups of objects uniformly**. A `Directory` and a `File` both implement `size()` — the directory recursively sums its children's sizes.

```
Composite Tree Structure:

  root/                           FileSystemEntry (interface)
  ├── readme.txt (100 bytes)        ├── File      (leaf)
  ├── src/                          └── Directory (composite)
  │   ├── main.cpp (500 bytes)          contains vector<FileSystemEntry>
  │   └── utils.cpp (300 bytes)
  └── docs/
      └── guide.md (200 bytes)

  root.size() = ?                   Recursive calculation:
  root->size()                      100 + src->size() + docs->size()
        = 100 + (500 + 300) + (200) = 100 + 800 + 200 = 1100

  Key insight: Directory.size() and File.size() have the SAME interface.
  Client code doesn't care if it's a file or deeply nested folder tree.
```

```cpp
class FileSystemEntry {
public:
    virtual ~FileSystemEntry() = default;
    virtual int size() const = 0;
    virtual std::string name() const = 0;
};

class File : public FileSystemEntry {
    std::string name_;
    int size_;
public:
    File(std::string n, int s) : name_(std::move(n)), size_(s) {}
    int size() const override { return size_; }
    std::string name() const override { return name_; }
};

class Directory : public FileSystemEntry {
    std::string name_;
    std::vector<std::unique_ptr<FileSystemEntry>> children_;
public:
    Directory(std::string n) : name_(std::move(n)) {}
    void add(std::unique_ptr<FileSystemEntry> entry) {
        children_.push_back(std::move(entry));
    }
    int size() const override {
        int total = 0;
        for (const auto& child : children_) total += child->size();
        return total;
    }
    std::string name() const override { return name_; }
};
```

---

## Chapter 4: Behavioral Patterns

### 4.1 Command — Encapsulate Actions as Objects

The Command pattern turns requests into **standalone objects** containing all information about the request. This enables undo/redo, queuing, logging, and macro recording.

```
Command Pattern with Undo/Redo Stack:

  User actions:   Insert "Hello"  →  Insert " World"  →  Undo

  Undo Stack:     ┌────────────┐  ┌────────────┐  ┌────────────┐
                  │ Insert     │  │ Insert     │  │ Insert     │
                  │ "Hello"    │  │ " World"   │  │ "Hello"    │
                  └────────────┘  └────────────┘  └────────────┘
                                  ┌────────────┐
                                  │ Insert     │  ← popped, undo() called
                                  │ " World"   │     erases " World"
                                  └────────────┘

  Document:       ""          "Hello"       "Hello World"   "Hello"
                  initial     after insert  after insert    after undo

  Each Command object knows how to execute() AND undo().
```

```cpp
class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class InsertTextCommand : public Command {
    std::string& document_;
    std::string text_;
    size_t position_;
public:
    InsertTextCommand(std::string& doc, std::string text, size_t pos)
        : document_(doc), text_(std::move(text)), position_(pos) {}
    void execute() override { document_.insert(position_, text_); }
    void undo() override { document_.erase(position_, text_.size()); }
};

// Undo/Redo stack:
class Editor {
    std::string document_;
    std::stack<std::unique_ptr<Command>> undoStack_;

public:
    void execute(std::unique_ptr<Command> cmd) {
        cmd->execute();
        undoStack_.push(std::move(cmd));
    }
    void undo() {
        if (!undoStack_.empty()) {
            undoStack_.top()->undo();
            undoStack_.pop();
        }
    }
};
```

### 4.2 State — Object Changes Behavior Based on State

The State pattern lets an object **alter its behavior** when its internal state changes. The object appears to change its class. Each state is a separate class implementing the same interface.

```
TCP Connection State Machine:

  ┌───────────┐  open()   ┌────────────┐ client    ┌─────────────┐
  │  CLOSED  │───────►│ LISTENING  │────────►│ ESTABLISHED │
  └───────────┘          └────────────┘ connects  └──────┬──────┘
       ▲                                                  │
       │                                            close()
       │                                                  │
       └────────────────  timeout  ─────────────────┘

  Same TCPConnection object, different behavior depending on state:
  - Closed state:      open() transitions to Listening, send() throws
  - Listening state:   open() is no-op, send() throws
  - Established state: send() transmits data, close() resets to Closed

  Each state class handles the SAME methods differently.
```

```cpp
class TCPConnection;

class TCPState {
public:
    virtual ~TCPState() = default;
    virtual void open(TCPConnection& conn) = 0;
    virtual void close(TCPConnection& conn) = 0;
    virtual void send(TCPConnection& conn, const std::string& data) = 0;
};

class TCPConnection {
    std::unique_ptr<TCPState> state_;
public:
    void setState(std::unique_ptr<TCPState> s) { state_ = std::move(s); }
    void open() { state_->open(*this); }
    void close() { state_->close(*this); }
    void send(const std::string& data) { state_->send(*this, data); }
};
// Each state (Closed, Listening, Established) handles transitions differently
```

### 4.3 Template Method — Algorithm Skeleton

The Template Method defines the **skeleton of an algorithm** in a base class, letting subclasses override specific steps without changing the overall structure.

```
Template Method Pattern:

  DataProcessor::process() (non-virtual — FIXED skeleton)
  ┌─────────────────────────────────────────────┐
  │  1. data = loadData()          ← pure virtual (subclass MUST override)  │
  │  2. cleaned = cleanData(data)  ← pure virtual                          │
  │  3. result = analyze(cleaned)  ← pure virtual                          │
  │  4. output(result)             ← virtual with DEFAULT (can override)   │
  └─────────────────────────────────────────────┘

  CSVProcessor                     JSONProcessor
  ┌───────────────────┐            ┌───────────────────┐
  │ loadData() → CSV  │            │ loadData() → JSON │
  │ cleanData() → trim│            │ cleanData() → fix │
  │ analyze() → stats │            │ analyze() → parse │
  │ output() → default│            │ output() → custom │
  └───────────────────┘            └───────────────────┘

  The ALGORITHM STRUCTURE is fixed. Only the STEPS vary.
```

```cpp
class DataProcessor {
public:
    void process() {        // Template method (not virtual)
        auto data = loadData();
        auto cleaned = cleanData(data);
        auto result = analyze(cleaned);
        output(result);
    }
    virtual ~DataProcessor() = default;

protected:
    virtual std::string loadData() = 0;        // Subclass implements
    virtual std::string cleanData(const std::string& d) = 0;
    virtual std::string analyze(const std::string& d) = 0;
    virtual void output(const std::string& r) {
        std::cout << r << "\n";               // Default implementation
    }
};
```

### 4.4 Iterator — Sequential Access Without Exposing Internals

C++ STL iterators ARE this pattern. Custom iterator example:

```cpp
class IntRange {
    int start_, end_;
public:
    IntRange(int s, int e) : start_(s), end_(e) {}

    struct Iterator {
        int current;
        int operator*() const { return current; }
        Iterator& operator++() { ++current; return *this; }
        bool operator!=(const Iterator& other) const {
            return current != other.current;
        }
    };

    Iterator begin() const { return {start_}; }
    Iterator end() const { return {end_}; }
};

// Usage:
for (int x : IntRange(1, 10)) {
    std::cout << x << " ";  // 1 2 3 4 5 6 7 8 9
}
```

### 4.5 Chain of Responsibility

The Chain of Responsibility pattern passes a request along a **chain of handlers**. Each handler decides either to process it or pass it to the next handler. This decouples sender from receiver.

```
Chain of Responsibility Flow:

  HTTP Request arrives:

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  Rate    │───►│  Auth    │───►│ Logging  │───►│ Business │
  │  Limiter │    │  Handler │    │  Handler │    │  Logic   │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       │               │               │               │
  Too many         Bad token?      Log request       Process
  requests?        → REJECT        & pass along      & respond
  → REJECT

  Example: "unauthorized" request
  Rate Limiter: passes (rate OK)
  Auth Handler: REJECTS → returns false, chain stops
  Logging: never reached
  Business: never reached

  Adding a new handler = just insert it in the chain. No other code changes.
```

```cpp
class Handler {
    std::unique_ptr<Handler> next_;
public:
    virtual ~Handler() = default;
    Handler* setNext(std::unique_ptr<Handler> next) {
        next_ = std::move(next);
        return next_.get();
    }
    virtual bool handle(const std::string& request) {
        if (next_) return next_->handle(request);
        return false;
    }
};

class AuthHandler : public Handler {
public:
    bool handle(const std::string& request) override {
        if (request == "unauthorized") {
            std::cout << "Auth rejected\n";
            return false;
        }
        return Handler::handle(request);
    }
};
```

---

## Chapter 5: Modern C++ Pattern Techniques

### 5.1 CRTP — Compile-Time Polymorphism

CRTP (Curiously Recurring Template Pattern) achieves polymorphism **at compile time** — zero overhead, no vtable, no virtual dispatch. The trick: the derived class passes *itself* as a template parameter to the base.

```
CRTP vs Virtual Polymorphism:

  Virtual (runtime):                    CRTP (compile-time):
  ┌──────────┐                          ┌──────────────────┐
  │ Base     │   vtable lookup          │ Base<Concrete>   │  No vtable!
  │ virtual  │──► runtime cost          │                  │  static_cast
  │ f()      │   (cache miss)           │ interface()      │  resolves at
  └────┬─────┘                          └────┬─────────────┘  compile time
       │                                     │
  ┌────┴─────┐                          ┌────┴─────────────┐
  │ Derived  │                          │ Concrete         │
  │ f() ovr  │                          │ implementation() │
  └──────────┘                          └──────────────────┘

  Use Virtual when: you need runtime polymorphism (e.g., plugin systems)
  Use CRTP when:    types are known at compile time (e.g., mixins, counters)

  Cost comparison:
  Virtual call:  ~2-5ns (vtable lookup + possible cache miss)
  CRTP call:     ~0ns   (inlined by compiler, same as direct call)
```

```cpp
template<typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Concrete : public Base<Concrete> {
public:
    void implementation() {
        std::cout << "Concrete implementation\n";
    }
};
// Zero virtual function overhead — resolved at compile time
```

### 5.2 Type Erasure

```cpp
class AnyCallable {
    struct Concept {
        virtual ~Concept() = default;
        virtual void call() = 0;
    };

    template<typename T>
    struct Model : Concept {
        T callable_;
        Model(T c) : callable_(std::move(c)) {}
        void call() override { callable_(); }
    };

    std::unique_ptr<Concept> impl_;

public:
    template<typename T>
    AnyCallable(T callable)
        : impl_(std::make_unique<Model<T>>(std::move(callable))) {}

    void operator()() { impl_->call(); }
};
// std::function uses this pattern internally
```

---

## Chapter 6: Interview Questions

**Q1: What is the difference between Strategy and Template Method?** — Strategy uses composition (inject algorithm object), Template Method uses inheritance (override steps). Strategy is more flexible; Template Method has simpler code.

**Q2: When would you use Factory Method vs Abstract Factory?** — Factory Method creates one product via subclassing. Abstract Factory creates families of related products.

**Q3: What is the problem with Singleton?** — Hidden dependencies, global state, hard to test, violates SRP/OCP. Prefer dependency injection.

---

## Chapter 7: Mastery Checklist

- [ ] You can implement all 23 GoF patterns from memory
- [ ] You know when to use (and NOT use) each pattern
- [ ] You use modern C++ idioms (smart pointers, lambdas, templates)
- [ ] You understand SOLID principles and can evaluate designs
- [ ] You can apply CRTP and Type Erasure patterns

---

*End of Book 17: Design Patterns in C++*
