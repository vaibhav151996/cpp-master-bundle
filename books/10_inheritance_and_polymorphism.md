# Book 10: Inheritance & Polymorphism

## The Complete Guide to Code Reuse, Type Hierarchies, and Runtime Dispatch in C++

---

### **Target Level:** Intermediate → Advanced
### **Prerequisites:** Books 1–9
### **Learning Outcomes:**
By the end of this book, you will:
- Master single, multiple, and multilevel inheritance
- Understand virtual functions, vtables, and dynamic dispatch
- Know abstract classes, pure virtual functions, and interfaces
- Understand object slicing, covariant returns, and virtual destructors
- Master runtime polymorphism with deep internal understanding
- Know CRTP and static polymorphism

---

## Chapter 1: Inheritance Fundamentals

### 1.1 What is Inheritance?

Inheritance allows a class (derived) to acquire the properties and behaviors of another class (base):

```cpp
class Animal {
protected:
    std::string name_;
    int age_;
public:
    Animal(const std::string& name, int age) : name_(name), age_(age) {}
    
    void eat() const {
        std::cout << name_ << " is eating.\n";
    }
    
    std::string getName() const { return name_; }
};

class Dog : public Animal {    // Dog IS-A Animal
    std::string breed_;
public:
    Dog(const std::string& name, int age, const std::string& breed)
        : Animal(name, age), breed_(breed) {}   // Call base constructor
    
    void bark() const {
        std::cout << name_ << " says: Woof!\n";
    }
};

Dog d("Rex", 3, "German Shepherd");
d.eat();     // Inherited from Animal
d.bark();    // Dog's own method
```

### 1.2 Access Control with Inheritance

| Base Member | `public` inheritance | `protected` inheritance | `private` inheritance |
|-------------|---------------------|------------------------|---------------------|
| `public` | `public` | `protected` | `private` |
| `protected` | `protected` | `protected` | `private` |
| `private` | Not accessible | Not accessible | Not accessible |

```cpp
class Base {
public:    int pub;
protected: int prot;
private:   int priv;
};

class PublicDerived : public Base {
    // pub is public, prot is protected, priv is inaccessible
};

class ProtectedDerived : protected Base {
    // pub is protected, prot is protected, priv is inaccessible
};

class PrivateDerived : private Base {
    // pub is private, prot is private, priv is inaccessible
};
```

**Use `public` inheritance 99% of the time.** The others are rare.

### 1.3 Types of Inheritance

```cpp
// Single inheritance:
class B : public A { };

// Multilevel inheritance:
class C : public B { };  // C → B → A

// Multiple inheritance:
class D : public A, public B { };

// Hierarchical inheritance:
class Dog : public Animal { };
class Cat : public Animal { };

// Diamond inheritance (problematic):
class A { };
class B : public A { };
class C : public A { };
class D : public B, public C { };
// D has TWO copies of A! Ambiguity!
```

### 1.4 The Diamond Problem and Virtual Inheritance

```cpp
class Animal {
public:
    std::string name;
};

class Flyer : virtual public Animal { };    // virtual inheritance
class Swimmer : virtual public Animal { };  // virtual inheritance

class Duck : public Flyer, public Swimmer { };
// With 'virtual', Duck has only ONE copy of Animal
```

---

## Chapter 2: Virtual Functions and Polymorphism

### 2.1 The Problem: Static Binding

```cpp
class Shape {
public:
    void draw() const {
        std::cout << "Drawing shape\n";
    }
};

class Circle : public Shape {
public:
    void draw() const {   // Hides Shape::draw
        std::cout << "Drawing circle\n";
    }
};

Shape* s = new Circle();
s->draw();   // "Drawing shape" — calls Shape::draw, NOT Circle::draw!
// The compiler uses the POINTER TYPE (Shape*), not the OBJECT TYPE (Circle)
```

### 2.2 The Solution: Virtual Functions

```cpp
class Shape {
public:
    virtual void draw() const {    // virtual keyword enables dynamic dispatch
        std::cout << "Drawing shape\n";
    }
    
    virtual ~Shape() = default;    // ALWAYS make base class destructor virtual
};

class Circle : public Shape {
public:
    void draw() const override {   // override = compiler checks it actually overrides
        std::cout << "Drawing circle\n";
    }
};

class Rectangle : public Shape {
public:
    void draw() const override {
        std::cout << "Drawing rectangle\n";
    }
};

// Polymorphism in action:
Shape* s = new Circle();
s->draw();   // "Drawing circle" — correct! 

// Polymorphic container:
std::vector<std::unique_ptr<Shape>> shapes;
shapes.push_back(std::make_unique<Circle>());
shapes.push_back(std::make_unique<Rectangle>());

for (const auto& shape : shapes) {
    shape->draw();  // Calls the correct draw() for each shape
}
```

### 2.3 How Virtual Functions Work — The VTable

When a class has virtual functions, the compiler creates a **virtual function table (vtable)**:

```
Shape vtable:
┌─────────────────────────┐
│ &Shape::draw            │ ──→ Shape::draw() code
│ &Shape::~Shape          │ ──→ Shape destructor code
└─────────────────────────┘

Circle vtable:
┌─────────────────────────┐
│ &Circle::draw           │ ──→ Circle::draw() code  (overridden)
│ &Circle::~Circle        │ ──→ Circle destructor code
└─────────────────────────┘
```

Each object with virtual functions contains a hidden pointer (**vptr**) to its class's vtable:

```
Circle object (in memory):
┌─────────────────────────┐
│ vptr ─────────────────────→ Circle vtable
├─────────────────────────┤
│ ... Circle data members │
└─────────────────────────┘
```

**Virtual function call:** `s->draw()` → follow vptr → look up `draw` in vtable → call the function. This is **one pointer dereference** (~1 nanosecond overhead).

### 2.4 `override` and `final` (C++11)

```cpp
class Base {
public:
    virtual void foo(int x) const;
};

class Derived : public Base {
public:
    void foo(int x) const override;    // OK — matches Base::foo
    // void foo(double x) override;    // ERROR! No matching virtual function
    // override catches the mistake!
};

class Final : public Derived {
public:
    void foo(int x) const final;  // No further overriding allowed
};

class TooFar : public Final {
    // void foo(int x) const override;  // ERROR! foo is final
};

// Can also make the class itself final:
class Sealed final : public Base {
    // Cannot be derived from
};
```

### 2.5 Virtual Destructors — Essential Rule

```cpp
class Base {
public:
    ~Base() { std::cout << "~Base\n"; }
};

class Derived : public Base {
    int* data_;
public:
    Derived() : data_(new int[100]) {}
    ~Derived() { delete[] data_; std::cout << "~Derived\n"; }
};

Base* p = new Derived();
delete p;    // Only calls ~Base! ~Derived is NOT called! MEMORY LEAK!

// FIX: Make destructor virtual:
class Base {
public:
    virtual ~Base() = default;  // Now delete is dispatched correctly
};
```

**Rule:** If a class has ANY virtual functions, its destructor MUST be virtual.

---

## Chapter 3: Abstract Classes and Interfaces

### 3.1 Pure Virtual Functions

```cpp
class Shape {
public:
    virtual double area() const = 0;         // Pure virtual — no implementation
    virtual double perimeter() const = 0;    // Pure virtual
    virtual void draw() const = 0;           // Pure virtual
    
    virtual ~Shape() = default;
    
    // Can still have non-virtual methods:
    void printInfo() const {
        std::cout << "Area: " << area() << ", Perimeter: " << perimeter() << "\n";
    }
};

// Shape is now ABSTRACT — cannot be instantiated:
// Shape s;  // ERROR! Cannot create object of abstract class

class Circle : public Shape {
    double radius_;
public:
    Circle(double r) : radius_(r) {}
    
    double area() const override { return 3.14159 * radius_ * radius_; }
    double perimeter() const override { return 2 * 3.14159 * radius_; }
    void draw() const override { std::cout << "Drawing circle\n"; }
};

// Circle implements ALL pure virtuals — can be instantiated:
Circle c(5.0);
c.printInfo();  // "Area: 78.5398, Perimeter: 31.4159"
```

### 3.2 Interfaces (Pure Abstract Classes)

```cpp
// C++ "interface" — class with ONLY pure virtual functions:
class ISerializable {
public:
    virtual std::string serialize() const = 0;
    virtual void deserialize(const std::string& data) = 0;
    virtual ~ISerializable() = default;
};

class IDrawable {
public:
    virtual void draw() const = 0;
    virtual ~IDrawable() = default;
};

// A class implementing multiple interfaces:
class Widget : public ISerializable, public IDrawable {
    int x_, y_;
public:
    std::string serialize() const override { return std::to_string(x_) + "," + std::to_string(y_); }
    void deserialize(const std::string& data) override { /* parse data */ }
    void draw() const override { std::cout << "Drawing widget at (" << x_ << "," << y_ << ")\n"; }
};
```

---

## Chapter 4: Object Slicing

```cpp
class Base {
public:
    int x = 10;
    virtual void print() { std::cout << "Base: " << x << "\n"; }
};

class Derived : public Base {
public:
    int y = 20;
    void print() override { std::cout << "Derived: " << x << ", " << y << "\n"; }
};

Derived d;
Base b = d;      // OBJECT SLICING! Derived part (y) is cut off!
b.print();       // "Base: 10" — not "Derived: 10, 20"

// To preserve polymorphism, use pointers or references:
Base& ref = d;
ref.print();     // "Derived: 10, 20" — correct!

Base* ptr = &d;
ptr->print();    // "Derived: 10, 20" — correct!
```

---

## Chapter 5: Advanced Polymorphism

### 5.1 CRTP — Curiously Recurring Template Pattern (Static Polymorphism)

```cpp
template<typename Derived>
class Shape {
public:
    double area() const {
        return static_cast<const Derived*>(this)->areaImpl();
    }
};

class Circle : public Shape<Circle> {
    double radius_;
public:
    Circle(double r) : radius_(r) {}
    double areaImpl() const { return 3.14159 * radius_ * radius_; }
};

class Square : public Shape<Square> {
    double side_;
public:
    Square(double s) : side_(s) {}
    double areaImpl() const { return side_ * side_; }
};

// No vtable overhead — dispatch resolved at compile time
Circle c(5.0);
std::cout << c.area();  // 78.5398 — calls Circle::areaImpl via template
```

### 5.2 `dynamic_cast`

```cpp
class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void specificMethod() { std::cout << "Derived specific\n"; }
};

Base* bp = new Derived();

// Pointer cast — returns nullptr on failure:
Derived* dp = dynamic_cast<Derived*>(bp);
if (dp) {
    dp->specificMethod();  // Safe!
}

// Reference cast — throws std::bad_cast on failure:
try {
    Derived& dr = dynamic_cast<Derived&>(*bp);
    dr.specificMethod();
} catch (const std::bad_cast& e) {
    std::cerr << "Cast failed: " << e.what() << "\n";
}
```

---

## Chapter 6: Interview Questions

**Q1: What is polymorphism?** — The ability to treat objects of different types through a common interface. C++ supports compile-time (templates, overloading) and runtime (virtual functions) polymorphism.

**Q2: What is a vtable?** — A table of function pointers created by the compiler for each class with virtual functions. Objects contain a vptr that points to their class's vtable.

**Q3: What is object slicing?** — When a derived class object is assigned to a base class variable by value, the derived-class data is "sliced off." Use pointers/references to prevent this.

**Q4: Why should base class destructors be virtual?** — Without virtual destructor, `delete basePtr` only calls the base destructor, causing resource leaks in derived classes.

**Q5: What is the diamond problem?** — When a class inherits from two classes that both inherit from a common base, creating ambiguity. Solved with `virtual` inheritance.

**Q6: What is the difference between `override` and `final`?** — `override` tells the compiler to verify the function overrides a virtual function. `final` prevents further overriding.

---

## Chapter 7: Practice & Projects

### Project 1: Shape Hierarchy
Implement Shape (abstract), Circle, Rectangle, Triangle, Polygon with area, perimeter, draw. Use polymorphic container.

### Project 2: Employee Management System
Base Employee → Manager, Engineer, Intern. Virtual method `calculatePay()`. Polymorphic payroll processing.

### Project 3: Plugin System
Abstract Plugin interface. Load different "plugins" (classes implementing the interface). Process them through the base interface.

---

## Chapter 8: Summary & Mastery Checklist

- [ ] You understand single, multiple, and multilevel inheritance
- [ ] You know how virtual functions and vtables work
- [ ] You can design abstract classes and interfaces
- [ ] You understand object slicing and how to avoid it
- [ ] You always use `override`, `final`, and virtual destructors
- [ ] You know `dynamic_cast` and when to use it
- [ ] You understand CRTP for static polymorphism
- [ ] You can solve the diamond problem with virtual inheritance

**Proceed to Book 11: Templates.**

---

*End of Book 10: Inheritance & Polymorphism*
