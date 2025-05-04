---
layout: post
title: External Polymorphism
date: 2024-01-07
---


# External Polymorhphism
We've all seen how polymorphism works from introductory object orient programming courses 
and it has worked reasonably well!  
However, in the real all-encompassing software world, this basic implementation may not be enough.

Let's take a look at the classical solution first before diving into deficiencies and **External Polymorphism** design pattern.

## Polymorphism with Inheritance
```cpp
struct Animal {
  virtual std::string noise() = 0;
};

struct Cow : Animal {
  std::string noise() override { return "moo"; }
};

struct Dog : Animal {
  std::string noise() override { return "bork"; }
};

struct Cat : Animal {
  std::string noise() override { return "meow"; }
};

void main() {
    // Polymorphism allows us to interact with classes with a shared interface uniformly
    vector<unique_ptr<Animal>> animals;
    animals.emplace_back(new Cow);
    animals.emplace_back(new Dog);
    animals.emplace_back(new Cat);

    // Prints out:
    // moo
    // bork
    // meow
    for(const auto& animal : animals){
        cout << animal->noise() << endl;
    }
}
```


The above solution works, but may hit the following issues:
 * Certain animals may be implemented by a third-party such that we **can not modify the source code**.  
   In this case, we are unable to have these external classes extend our virtual class `Animal`.
 * In some cases, we do not desire or **can not modify storage layout** by adding virtual methods.

## A New Approach

Let's take a look at what the external polymorphism approach would look like.

```cpp
// We have the same set of animals as before.
// This time these animals do not inherit from the `Animal` interface.
struct Cow {
    string noise() { return "moo"; }
};

struct Cat {
    string noise() { return "meow"; }
};

struct Dog {
    string noise() { return "bork"; }
};

// The `Animal` interface will be the abstraction that external users will interact with.
// This interface will be used built on top of the classes we have defined above.
struct Animal {
  virtual string noise() = 0;
};

// Helper method for producing the noise for each animal class that simply defers 
// to the base `noise` method. Refer to usage in `AnimalAdapter` for details.
template<typename T>
string make_noise(const T* animal){
    return animal->noise();
}

// `AnimalAdapter` is the actual class inheriting from the `Animal` interface.
// For each class `T` that we conceptually consider an animal, 
// we have in instantiation of the template: `AnimalAdapter<T>`
template<typename T>
struct AnimalAdapter : public Animal {
    AnimalAdapter(T* animal) : animal_(animal) {}

    string noise() override {
        return make_noise<T>(animal_);
    }

    T* animal_;
}

void main() {
    auto animal1 = Cow{};
    auto animal2 = Dog{};
    auto animal3 = Cat{};

    vector<unique_ptr<Animal>> animals;
    animals.push_back(AnimalAdapter(&animal1))
    animals.push_back(AnimalAdapter(&animal2))
    animals.push_back(AnimalAdapter(&animal3))

    // As with the polymorphism with inheritance example, we interact with the 
    // different animals uniformly.
    // Prints out:
    // moo
    // bork
    // meow
    for(const auto& animal : animals){
        cout << animal.noise() << endl;
    }
}

```

The above code sample details the boiler plate to utilize External Polymorphism.  
As before, we have three animal classes. Instead of having them derive from the `Animal` interface, 
we wrap each of them into adapters that do inherit from the desired public interface: 
`AnimalAdapter<Cow>`, `AnimalAdapter<Dog>`, and `AnimalAdapter<Cat>`.

The `AnimalAdapter::noise()` method defers to the `make_noise` method in the global namespace.
The templatized `makes_noise` method defers to each individual animal's `noise()` method.  

All together, we are able to still leverage polymorphism without having the base classes 
inherit from a common base class.

### The Power of Indirection
The addition of the stand-alone `make_noise` class seems excessive.
If all the target animal classes have a public `noise()` method, we can embed it into `AnimalAdapter` instead: 
```cpp
// Alternate AnimalAdapter defitnition
template<typename T>
struct AnimalAdapter : public Animal {
    AnimalAdapter(T* animal) : animal_(animal) {}

    string noise() override {
        // Just call `noise()` method directly without the indirection with `make_noise`
        return animal_->noise();
    }

    T* animal_;
}

```

However, this indirection in the original definition of `AnimalAdapter` affords us 
more flexibility with alternate implementations and wrapping non-conforming classes.
This is especially useful if the class in question in provided by a third-party.
 
```cpp
// We can specialize the make_noise method for `Dog` to differ than the default implementation.
string make_noise(const Dog* d){
    return d->noise() + ' ' + d->noise();
}

// We have a class that does not conform to having a public `noise` method
struct Human {
    string name();
};

// In this case, we manually define the `make_noise` method.
string make_noise(const Human* h){
    return "Hi, my name is " + h->name();
}

```


#### References
1. [External Polymorphism - An Object Structural Pattern for Transparently Extending C++ Concrete Data Types](https://www.dre.vanderbilt.edu/~schmidt/PDF/C++-EP.pdf)
