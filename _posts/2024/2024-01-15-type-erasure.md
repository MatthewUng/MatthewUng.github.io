---
layout: post
title: C++ Type Erasure
date: 2024-01-15
---


# A Quick Look at std::function
Before diving into the implementation details of type erasure, 
let's take a look at what behaviors `std::function` need to support.
```cpp
struct S {
  int operator()() { return 1; }
};

int g() { return 2; }

int main(){
    // f represents a function that takes no arguments and returns an int.
    std::function<int()> f;

    // Assign f is invocable struct
    f = S();
    std::cout << f() << std::endl; // prints 1

    // Reassign f to function pointer
    f = g;
    std::cout << f() << std::endl; // prints 2

    // Reassign f to lambda
    f = [](){ return 3; };
    std::cout << f() << std::endl; // prints 3

    // We can also copy `std::function` object
    std::function<int()> f2(f);
    std::cout << f2() << std::endl; // prints 3

    return 0;
}
```

In our example, `std::function` needs to support function-esque objects, 
such as classes with `operator()()` defined, function pointers, and even lambdas.
All of these have different types, yet we handle them identically using an object of type `std::function<int()>`.
We can even assign these objects around as if they were interchangeable.

Under the hood, this behavior is implemented using the **Type Erasure** design pattern.  (Albeit with numerous optimizations that are not described here.)


# Iterating toward Type Erasure
The type erasure pattern allows us to interact polymorphically with **different objects** with the **same interface** that **do not share an 
inheritance hierarchy** with **value semantics**.

<!-- Type erasure allows us to treat objects with similar behaviors polymorphically but with value semantics. -->

Implementation-wise, type erasure combines the following design patterns: the [external polymorphism pattern]({% post_url 2024-01-07-external-polymorphism %}), 
the [bridge pattern](https://en.wikipedia.org/wiki/Bridge_pattern), and the [prototype pattern](https://en.wikipedia.org/wiki/Prototype_pattern).

Suppose we have the following interface that many objects implicitly adhere to via duck-typing.
```cpp
struct Animal {
  virtual string make_noise() = 0;
};

struct Dog {
    string make_noise() { return "bork"; }
};

struct Cat {
    string make_noise() { return "meow"; }
};
```


To abstract away the implicit common interface between them and to treat the objects polymorphically, we use the external polymorphism pattern to obtain: 
```cpp
struct AnimalInterface {
  virtual string make_noise() = 0;
  virtual ~AnimalInterface() = default;
};

template<typename T>
struct AnimalAdapter : public AnimalInterface {
    AnimalAdapter(T animal) : animal_(animal) {}

    string make_noise() override {
        return animal_.make_noise();
    }

    T animal_;
};
```

However, usage of external polymorphism is unwieldy as it forces users to instantiate their own `AnimalAdapter`.
To better hide the implementation details, we can wrap the use of external polymorphism into its own class.


```cpp
// Animal class wraps usage of external polymorphism within.
class Animal {
    // AnimalInterface and AnimalAdapter are defined as before
    struct AnimalInterface {
        string make_noise() = 0;
        virtual ~AnimalInterface() = default;
    };
    template<typename T>
    struct AnimalAdapter : public AnimalInterface {
        AnimalAdapter(T animal) : animal_(animal) {}

        string make_noise() override {
            return animal_.make_noise();
        }
        T animal_;
    };
    
    // Internally, `Animal` only contains a pointer to `AnimalAdapter`
    unique_ptr<AnimalInterface> animal_;

    public:
    // This templated constructor allows us to construct different implementations of `AnimalAdapter`
    template<typename T>
    Animal(T ani) : animal_(make_unique<AnimalAdapter<T>>(ani)) {}

    string make_noise() {
        return animal_->make_noise();
    }
};
```
By wrapping the usage external polymorphism into its own classe, we are effectively leveraging the bridge pattern.
The internal pointer within `Animal` to `AnimalInterface` allows for different possible implementations.


Finally, to support the value semantics that we desire, we add the following copy constructors and copy assignment operators by levaraging the prototype pattern.
```cpp
class Animal {
    struct AnimalInterface {
        virtual string make_noise() = 0;

        // Added `clone` interface method
        virtual AnimalInterface* clone() = 0;
        virtual ~AnimalInterface() = default;
    };

    template<typename T>
    struct AnimalAdapter : public AnimalInterface {
        AnimalAdapter(T animal) : animal_(animal) {}
        string make_noise() override {
            return animal_.make_noise();
        }

        // Added `clone` method
        AnimalInterface* clone() override {
            return new AnimalAdapter<T>(animal_);
        }

        T animal_;
    };
    
    unique_ptr<AnimalInterface> animal_;

    public:
    template<typename T>
    Animal(T ani) : animal_(make_unique<AnimalAdapter<T>>(ani)) {}
    // Added copy ctor using prototype 
    Animal(const Animal& other) : animal_(other.animal_->clone()){ }

    // Added assignment operators
    Animal& operator=(const Animal& other) {
        animal_ = unique_ptr<AnimalInterface>(other.animal_->clone());
        return *this;
    }
    template<typename T>
    Animal& operator=(T ani) {
        animal_ = make_unique<AnimalAdapter<T>>(ani);
        return *this;
    }

    string make_noise() {
        return animal_->make_noise();
    }
};
```

Finally, we obtain an `Animal` class whose behavior resembles that of `std::function`!
```cpp
int main(){
    // Dog class satisfies interface for `Animal`
    Animal animal1(Dog{});

    cout << animal1.make_noise() << endl; // prints "bork"

    // Reassign to class satisfying animal interface
    animal1 = Cat{};
    cout << animal1.make_noise() << endl; // prints "meow"

    // Reassign to another `Animal` instance
    Animal animal2 = Dog();
    animal1 = animal2;
    cout << animal1.make_noise() << endl; // prints "bork"
}
```


#### References
1. [Breaking Dependencies: Type Erasure - A Design Analysis - Klaus Iglberger - CppCon 2021](https://youtu.be/4eeESJQk-mw)
1. [https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Type_Erasure](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Type_Erasure)
