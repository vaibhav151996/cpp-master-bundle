# Book 9: Object-Oriented Programming

## The Complete Guide to Classes, Objects, Encapsulation, and OOP Design in C++

---

### **Target Level:** Intermediate → Advanced
### **Prerequisites:** Books 1–8
### **Learning Outcomes:**
By the end of this book, you will:
- Master class design with proper encapsulation
- Understand constructors, destructors, and the object lifecycle
- Master the Rule of Three/Five/Zero
- Understand `this` pointer, `static` members, and `friend` functions
- Know operator overloading deeply
- Write production-quality classes following RAII

---

## Chapter 1: Classes and Objects

### 1.1 Defining a Class

```cpp
class BankAccount {
private:    // Data hiding — only class methods can access
    std::string owner_;
    double balance_;
    int accountNumber_;

public:     // Interface — accessible from outside
    // Constructor
    BankAccount(const std::string& owner, double initialBalance)
        : owner_(owner), balance_(initialBalance), 
          accountNumber_(generateAccountNumber()) {}
    
    // Methods
    void deposit(double amount) {
        if (amount > 0) {
            balance_ += amount;
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance_) {
            balance_ -= amount;
            return true;
        }
        return false;
    }
    
    // Getter (accessor)
    double getBalance() const { return balance_; }
    std::string getOwner() const { return owner_; }
    
private:
    static int nextAccountNumber_;
    static int generateAccountNumber() { return nextAccountNumber_++; }
};

int BankAccount::nextAccountNumber_ = 1000;

// Usage:
BankAccount account("Alice", 1000.0);
account.deposit(500.0);
account.withdraw(200.0);
std::cout << account.getBalance();  // 1300.0
```

### 1.2 Access Specifiers

| Specifier | Within Class | Derived Class | Outside |
|-----------|-------------|---------------|---------|
| `public` | Yes | Yes | Yes |
| `protected` | Yes | Yes | No |
| `private` | Yes | No | No |

### 1.3 Encapsulation

**Encapsulation** = bundling data with the methods that operate on it, and restricting direct access to internal state.

```cpp
// BAD — no encapsulation:
struct Account {
    double balance;  // Anyone can set to -1000!
};

// GOOD — encapsulated:
class Account {
    double balance_;
public:
    void deposit(double amount) {
        if (amount > 0) balance_ += amount;  // Validation!
    }
};
```

---

## Chapter 2: Constructors & Destructors

### 2.1 Constructor Types

```cpp
class Widget {
    int id_;
    std::string name_;
    int* data_;
    
public:
    // Default constructor
    Widget() : id_(0), name_("unnamed"), data_(nullptr) {}
    
    // Parameterized constructor
    Widget(int id, const std::string& name)
        : id_(id), name_(name), data_(new int[100]()) {}
    
    // Copy constructor
    Widget(const Widget& other)
        : id_(other.id_), name_(other.name_), data_(nullptr) {
        if (other.data_) {
            data_ = new int[100];
            std::copy(other.data_, other.data_ + 100, data_);
        }
    }
    
    // Move constructor (C++11)
    Widget(Widget&& other) noexcept
        : id_(other.id_), name_(std::move(other.name_)), data_(other.data_) {
        other.data_ = nullptr;  // Steal the resource
    }
    
    // Copy assignment operator
    Widget& operator=(const Widget& other) {
        if (this != &other) {
            delete[] data_;
            id_ = other.id_;
            name_ = other.name_;
            if (other.data_) {
                data_ = new int[100];
                std::copy(other.data_, other.data_ + 100, data_);
            } else {
                data_ = nullptr;
            }
        }
        return *this;
    }
    
    // Move assignment operator (C++11)
    Widget& operator=(Widget&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            id_ = other.id_;
            name_ = std::move(other.name_);
            data_ = other.data_;
            other.data_ = nullptr;
        }
        return *this;
    }
    
    // Destructor
    ~Widget() {
        delete[] data_;    // Clean up resources
    }
};
```

### 2.2 Member Initializer List

Always use initializer lists instead of assignment in constructors:

```cpp
// PREFERRED — direct initialization:
class Point {
    double x_, y_;
public:
    Point(double x, double y) : x_(x), y_(y) {}
};

// LESS EFFICIENT — assignment in body:
class Point {
    double x_, y_;
public:
    Point(double x, double y) {
        x_ = x;  // Default-constructed first, then assigned
        y_ = y;
    }
};
```

Initializer lists are **required** for: `const` members, reference members, and members without default constructors.

### 2.3 Rule of Three / Five / Zero

**Rule of Three (C++03):** If you define any one of: destructor, copy constructor, copy assignment — define ALL three.

**Rule of Five (C++11):** Add move constructor and move assignment to the Rule of Three.

**Rule of Zero (Modern C++):** Prefer not defining any special member functions. Use smart pointers and standard containers that manage resources for you.

```cpp
// Rule of Zero (best — compiler generates everything):
class ModernWidget {
    int id_;
    std::string name_;
    std::vector<int> data_;  // Manages its own memory
    // No destructor, copy/move constructors needed!
};
```

### 2.4 `explicit` Keyword

Prevents implicit type conversions:

```cpp
class Temperature {
    double celsius_;
public:
    explicit Temperature(double c) : celsius_(c) {}
    double getCelsius() const { return celsius_; }
};

Temperature t1(36.6);          // OK — direct initialization
Temperature t2 = 36.6;        // ERROR! explicit prevents implicit conversion
Temperature t3 = Temperature(36.6);  // OK — explicit construction
```

### 2.5 Delegating Constructors (C++11)

```cpp
class Rectangle {
    double width_, height_;
public:
    Rectangle() : Rectangle(1.0, 1.0) {}               // Delegates to parameterized
    Rectangle(double side) : Rectangle(side, side) {}   // Square
    Rectangle(double w, double h) : width_(w), height_(h) {}  // Full
};
```

---

## Chapter 3: The `this` Pointer

Every member function has an implicit parameter `this` — a pointer to the object the function is called on:

```cpp
class Counter {
    int count_;
public:
    Counter(int c) : count_(c) {}
    
    // this->count_ is implicit in member access:
    int get() const { return count_; }       // same as this->count_
    
    // Returning *this enables method chaining:
    Counter& increment() {
        count_++;
        return *this;
    }
    
    Counter& add(int n) {
        count_ += n;
        return *this;
    }
};

Counter c(0);
c.increment().increment().add(10).increment();
std::cout << c.get();  // 13
```

---

## Chapter 4: Static Members

### 4.1 Static Data Members

Shared across ALL objects of the class. Only one copy exists:

```cpp
class Player {
    std::string name_;
    static int totalPlayers_;   // Shared by all players
    
public:
    Player(const std::string& name) : name_(name) { totalPlayers_++; }
    ~Player() { totalPlayers_--; }
    
    static int getTotalPlayers() { return totalPlayers_; }
};

int Player::totalPlayers_ = 0;  // Must define outside class

Player p1("Alice"), p2("Bob");
std::cout << Player::getTotalPlayers();  // 2
```

### 4.2 Static Member Functions

Can only access static data members (no `this` pointer):

```cpp
class MathUtils {
public:
    static double pi() { return 3.14159265358979; }
    static int factorial(int n) {
        int result = 1;
        for (int i = 2; i <= n; i++) result *= i;
        return result;
    }
};

double area = MathUtils::pi() * r * r;  // Called without an object
```

---

## Chapter 5: Friend Functions and Classes

```cpp
class Secret {
    int hidden_ = 42;
    
    friend void reveal(const Secret& s);  // This function can access private members
    friend class Inspector;                 // This entire class can access private members
};

void reveal(const Secret& s) {
    std::cout << s.hidden_;  // OK — friend has access
}

class Inspector {
public:
    void inspect(const Secret& s) {
        std::cout << s.hidden_;  // OK — friend class
    }
};
```

**When to use friends:**
- Operator overloading (e.g., `operator<<` for streams)
- Closely related classes that need to share internals
- Testing utilities

---

## Chapter 6: Operator Overloading

### 6.1 Overloadable Operators

```cpp
class Vector2D {
    double x_, y_;
public:
    Vector2D(double x = 0, double y = 0) : x_(x), y_(y) {}
    
    // Arithmetic operators (member functions):
    Vector2D operator+(const Vector2D& other) const {
        return {x_ + other.x_, y_ + other.y_};
    }
    
    Vector2D operator-(const Vector2D& other) const {
        return {x_ - other.x_, y_ - other.y_};
    }
    
    Vector2D operator*(double scalar) const {
        return {x_ * scalar, y_ * scalar};
    }
    
    // Compound assignment:
    Vector2D& operator+=(const Vector2D& other) {
        x_ += other.x_;
        y_ += other.y_;
        return *this;
    }
    
    // Comparison (C++20):
    auto operator<=>(const Vector2D&) const = default;
    
    // Subscript:
    double& operator[](int index) {
        return (index == 0) ? x_ : y_;
    }
    
    // Function call:
    double operator()(double t) const {
        return x_ * t + y_;  // Linear function
    }
    
    // Stream output (friend, non-member):
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        return os << "(" << v.x_ << ", " << v.y_ << ")";
    }
};

Vector2D a(1, 2), b(3, 4);
Vector2D c = a + b;              // (4, 6)
std::cout << c << "\n";          // Uses operator<<
c += a;                           // (5, 8)
std::cout << c[0] << "\n";       // 5.0 (x component)
```

---

## Chapter 7: Interview Questions

**Q1: What is encapsulation?** — Bundling data with methods and restricting direct access to internal state through access specifiers.

**Q2: What is the Rule of Five?** — If you define any of destructor, copy constructor, copy assignment, move constructor, or move assignment, define all five.

**Q3: What is the difference between struct and class?** — Default access: struct is public, class is private. Otherwise identical.

**Q4: What is a copy constructor vs assignment operator?** — Copy constructor creates a new object as a copy. Assignment copies into an existing object.

**Q5: Can constructors be virtual?** — No. Constructors cannot be virtual. Destructors should be virtual in base classes with virtual functions.

---

## Chapter 8: Practice & Projects

### Project 1: Bank Account System
Full OOP system with Account, SavingsAccount, CheckingAccount classes, transaction history, and balance tracking.

### Project 2: Matrix Class
Complete matrix Class with +, -, *, transpose, determinant, inverse, and operator overloading.

### Project 3: Simple Game Entity System
Base Entity class with Position, Health, and methods. Derived Player, Enemy, NPC classes.

---

## Chapter 9: Summary & Mastery Checklist

- [ ] You can design classes with proper encapsulation
- [ ] You understand all constructor types and the member initializer list
- [ ] You know the Rule of Three/Five/Zero and when to apply each
- [ ] You understand `this` pointer and method chaining
- [ ] You can use `static` members and `friend` functions
- [ ] You can overload common operators
- [ ] You understand `explicit` and delegating constructors

**Proceed to Book 10: Inheritance & Polymorphism.**

---

*End of Book 9: Object-Oriented Programming*
