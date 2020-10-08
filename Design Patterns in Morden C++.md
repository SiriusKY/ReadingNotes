# Design Patterns in Modern C++
Reusable Approaches for Object-Oriented Software Design


## CHAPTER 1: Introduction

### 1. Important Concepts
* Curiously Recurring Template Pattern

Idea: an inheritor passes *itself* as a template argument to its base class.
```cpp
struct Foo : Somebase<Foo>{
    ...
}
```

* Mixin Inheritance
```cpp
template <typename T> struct Mixin : T {
    ...
}
```

* Properties
```cpp
class Person{
    int age_;
public:
    int get_age() const {
        return age_;
    }
    void set_age(int value) const {
        age_ = value;
    }
    __declspec(property(get=get_age, put=set_age)) int age;
}
```

### 2. The SOLID Design Principles
* Single Responsibility Principle (SRP)
    1. Each class has only one responsibility, and therefore has only one reason to change.

* Open-Closed Principle (OCP)
    1. A type is open for extension but closed for modification.

* Liskov Substitution Principle (LSP)
    1. If an interface takes an object of type Parent, it should equally take an object of type Child without anything breaking.
* Interface Segregation Principle (ISP)
    1. Segregate parts of a complicated interface into separate interfaces so as to avoid forcing implementors to implement functionality that they do not really need.
* Dependency Inversion Principle (DIP)
    1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
    2. Abstractions should not depend on details. Details should depend on abstractions.

## CHAPTER 2: Builder
### Key characteristics of a Builder:
* Builders can have a fluent interface that is usable for complicated construction using a single invocation chain. To support this, builder functions should return this or *this.
* To force the user of the API to use a Builder, we can make the target object’s constructors inaccessible and then define a static create() function that returns the builder.
* A builder can be coerced to the object itself by defining the appropriate operator.
* Groovy-style builders are possible in C++ thanks
to uniform initializer syntax. This approach is very general, and allows for the creation of diverse DSLs.
* A single builder interface can expose multiple subbuilders. Through clever use of inheritance and fluent interfaces, one can jump from one builder to another with ease.

Note that: simple objects that are unambiguously constructed from a limited number of sensibly named constructor parameters should probably use a constructor (or dependency injection) without necessitating a Builder as such.