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

Note: simple objects that are unambiguously constructed from a limited number of sensibly named constructor parameters should probably use a constructor (or dependency injection) without necessitating a Builder as such.

## CHAPTER 3: Factories

### 1. Factory Method
```cpp
struct Point {
private:
    float x, y;

protected:
    Point(const float x, const float y)
            : x{x}, y{y} {}

public:
    static Point NewCartesian(float x, float y) {
        return {x, y};
    }


    static Point NewPolar(float r, float theta) {
        return {r * cos(theta), r * sin(theta)};
    }
    // other members here
};
```
Each of the above static functions is called a *Factory Method*.  
To create a point, simply write:
```cpp
auto p = Point::NewPolar(5, M_PI_4);
```

### 2. Factory
```cpp
struct Point {
    float x, y;

    Point(float x, float y) : x(x), y(y) {}
};

struct PointFactory {
    static Point NewCartesian(float x, float y) {
        return Point{x, y};
    }

    static Point NewPolar(float r, float theta) {
        return Point{r * cos(theta), r * sin(theta)};
    }
};
```
Write as follows:
```cpp
auto my_point = PointFactory::NewCartesian(3, 4);
```

### 3. Inner Factory
```cpp
struct Point {
private:
    Point(float x, float y) : x(x), y(y) {}

    struct PointFactory {
    private:
        PointFactory() {}

    public:
        static Point NewCartesian(float x, float y) {
            return {x, y};
        }

        static Point NewPolar(float r, float theta) {
            return {r * cos(theta), r * sin(theta)};
        }
    };

public:
    float x, y;
    static PointFactory Factory;
};
```
Use the factory as:
```cpp
auto pp = Point::Factory.NewCartesian(2, 3);
```

### 4. Abstract Factory
```cpp
struct HotDrink {
    virtual void prepare(int volume) = 0;
};

struct Tea : HotDrink {

    void prepare(int volume) override {
        cout << "Take tea bag, boil water, pour " << volume << "ml." << endl;
    }
};

struct Coffee : HotDrink {

    void prepare(int volume) override {
        cout << "Take coffee bag, boil water, pour " << volume << "ml." << endl;
    }
};

struct HotDrinkFactory {
    virtual unique_ptr<HotDrink> make() const = 0;
};

struct TeaFactory : HotDrinkFactory {
    unique_ptr<HotDrink> make() const override {
        return make_unique<Tea>();
    }
};

struct CoffeeFactory : HotDrinkFactory {
    unique_ptr<HotDrink> make() const override {
        return make_unique<Coffee>();
    }
};

class DrinkFactory {
    map<string, unique_ptr<HotDrinkFactory>> hot_factories;
public:
    DrinkFactory() {
        hot_factories["coffee"] = make_unique<CoffeeFactory>();
        hot_factories["tea"] = make_unique<TeaFactory>();
    }

    unique_ptr<HotDrink> make_drink(const string &name) {
        auto drink = hot_factories[name]->make();
        drink->prepare(200); // oops!
        return drink;
    }
};
```
Use the factory as:
```cpp
auto drink_factory = DrinkFactory();
auto coffee = drink_factory.make_drink("coffee");
```

### 5. Functional Factory
```cpp
class DrinkWithVolumeFactory {
    map<string, function<unique_ptr<HotDrink>()>> factories;
public:
    DrinkWithVolumeFactory() {
        factories["tea"] = [] {
            auto tea = make_unique<Tea>();
            tea->prepare(200);
            return tea;
        }; // similar for Coffee
    }

    unique_ptr<HotDrink> make_drink(const string &name) {
        return factories[name]();
    }

};
```
Use the factory as:
```cpp
auto drink_factory = DrinkWithVolumeFactory();
auto tea = drink_factory.make_drink("tea");
```

### 6. Summary
1. terminology
* A *factory method* is a class member that acts as a way of creating object. It typically replaces a constructor.
* A *factory* is typically a separate class that knows how to construct objects, though if you pass a function (as in std::function or similar) that constructs objects, this argument is also called a factory.
* An *abstract factory* is, as its name suggests, an abstract class that can be inherited by concrete classes that offer a family of types. Abstract factories are rare in the wild.
2. A factory has several critical advantages over a constructor call, namely:
* A factory can say no, meaning that instead of actually returning an object it can return, for example, a nullptr.
* Naming is better and unconstrained, unlike constructor name.
* A single factory can make objects of many different types.
* A factory can exhibit polymorphic behavior, instantiating a class and returning it through its base class’ reference or pointer.
* A factory can implement caching and other storage optimizations; it is also a natural choice for approaches such as pooling or the Singleton pattern.

Note: Factory is different from Builder in that, with a Factory, you typically create an object in one go, whereas with Builder, you construct the object piecewise by providing information in parts.
