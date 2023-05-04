---
title: Legacy C in modern C++
date: 2020-06-13
description: This post deals with the strategy of integration of old C code in a modern C++ architecture, beyond the technical point of view.
tags:
- c
- c++
- c++20
- github

---
**\[tl;dr\]** The integration of `C` in `C++` was designed a long time ago and the mechanisms are efficient but spartan, forcing the client code to take care to manage memory and data access. When integrating into modern `C++`, where there is often a culture of good memory management and pointer manipulation, these mechanisms are not comfortable. There are solutions to interface `C` code more cleanly, through OOP or or `C++` smart pointers.

***

Code source of the full example available on [hnrck/c-integration-modern-cpp](https://github.com/hnrck/c-integration-modern-cpp).

***

# Table of Contents

* [Motivations](#motivations)
* [Context](#context)
* [Introduction](#introduction)
* [Discussion](#discussions)
  * [Integrating C in C++](#integrating-c-in-c++)
  * [Wrapped integration](#wrapped-integration)
  * [Smart pointers integration](#smart-pointers-integration)
* [Experimentation](#experimentation)
  * [GitHub repository](#github-repository)
  * [Compilation](#compilation)
  * [Execution](#execution)
* [Conclusion](#conclusion)
  * [Summary](#summary)
  * [Final word](#final-word)

***

# Motivations

It has been a long time since the last time I wrote an article. 
Not for lack of an idea, I thought of this one shortly after the first article.
But writing the first article took me a lot of time and energy, and basically it was too long to be useful.
I think my PhD student background played a key factor at the time.
I had an idea, a willingness to share something, and I reapplied the same rigid structure that I used in my scientific articles, which is clearly not science here.
This blog should rather be used to say what goes through my head, even if it's not as rigorous as my scientific work.

As of now, I have been a PhD for almost a year. 
I left a multinational company to join another one, and it works for me for now.
I am working more on execution than simulation at the moment, but I have inherited a certain background in simulation that I would like to share.

***

# Context

In the simulation business, reusability and compatibility are very important.
A simulation model can have a very long lifetime, and today with MBSE approaches, a simulation model can exceed the lifetime of the component it simulates.

In this context, it is reasonable to be willing to create models that are as accessible as possible outside of the environment in which they were created, and as far as I am aware, the worst thing that can happen to a simulation model that is intended to be reused is to be developed in an exotic, or worse, specific language.

***

# Introduction

In my PhD thesis, I had abstracted the problem by treating the model interface as a signature, implementable, and I think this compromise is a good way forward.
Here is the compromise I propose: Implementing simulation models in a stable, portable language that can be trusted to be reused as is for decades to come, without modifying a single line of code, and in parallel, improving the simulator to take advantage of technical improvements.
From my point of view, and if there is a guarantee that the simulation scheduling is mastered, which is what I propose in my thesis, then the best of both worlds will be made use of.

On the one hand there is a set of models on which it is possible to capitalize and keep an archive for analysis, and on the other hand the performance of the simulation is constantly improving.

From my point of view, there is only one language that offers confidence in reusability and compatibility, and that is `C`, ANSI standard.

No matter what architecture is used, no matter what kind of embedded system, no matter how old it is, there is always a way to compile and run C89.
So the question is, why not develop everything in `C89` since it's so good?
This is a good question, unfortunately the `C89` suffers from shortcomings when it comes to quickly implementing a complex architecture...
Specifically `C89` does not offer the possibility to easily implement object-oriented or functional paradigms, and these are the paradigms that designers use, and they are right to do so.
I'm the first one to criticize the object-oriented, I think this paradigm is too far from reality, but I must admit that when I design a solution, I rely on it a lot, and it simplifies my job.

`C++` is just one of those languages that implement the object-oriented paradigm, that's the main difference with `C`, and that's where there is a huge divergence with `C`.
Beyond the technical question of integration, how to integrate elements designed in one language with its paradigms in another language, with its own paradigms.

***

# Discussions

When I took up the `ROSACE` case study for my thesis, a lot of implementation work had already been done on it, including in `C`.
I wanted to quickly reuse these `C` elements, but in a much more modern solution.
I hesitated between `C++` and `python`, and in the end, it was `C++` which answered best to the other needs of my thesis.

## Integrating C in C++

Integrating `C` code into `C++` is technically incredibly simple, one just have to use:
```C
extern "C"
```

Thus, for a function declaration in a `C` header as following:
```C
int old_func(int x);
```

This function can be used in `C++` code as in `C` code:
```C++
const auto X = old_func(42);
```

As long as the header is included such as in the following:
```C++
extern "C" {
    #include <header.h>
}
```

Or if the header includes the following code:
```C
#ifdef __cplusplus
extern "C" {
#endif

/* ...declarations ... */

#ifdef __cplusplus
}
#endif
```

Which is not always the case with legacy code.

But the question of reuse strategy comes up very quickly.

Two solutions emerge: either to take over the paradigms of the old language or to implement those of the new one.
Here in the integration of `C89` in `C++` (11 at the time of my thesis, but I will use C++20 in this article), it is either a question of choosing the procedural approach and reusing functions in the same way as one would do in `C`, or using classes to encapsulate.
Thus, a declared `C` structure:

```C
struct old_struct {
    int x;
};
typedef struct old_struct old_struct_t;
```

Can be manipulated in `C++` as a native `C++ struct`, i.e. with default constructor and all public members.

```C++
auto os = old_struct();
os.x = 42;
const auto X = os.x;
```

However, the use of C-structures is not just for packaging data, and there are sometimes associated functions that process these structures.
Here let's simply consider a getter and a setter on the x property:

```C
void old_struct_set_x(old_struct_t *p_old_struct, int x);
```

```C
int old_struct_get_x(const old_struct_t *p_old_struct);
```

The passage by reference does not exist in `C`, it will be necessary to switch back to a way of calling the functions on these not so modern structures, directly inherited from C:

```C++
auto os = old_struct();
old_struct_set_x(&os, 42);
const auto X = old_struct_get_x(&os);
```

More complicated, if the legacy code contains private implementations, such as:

```C
struct old_pimpl;
typedef struct old_pimpl old_pimpl_t;
```

In `C++`, it will be necessary to manipulate all the allocation, initialization, destruction and manipulation functions as in `C`.
Here let's consider these functions:

Allocation and initialization:
```C
old_pimpl_t *old_pimpl_new();
```

Finalization and destruction:
```C
void old_pimpl_del(old_pimpl_t *p_old_pimpl);
```

Data manipulation:
```C
void old_pimpl_set_x(old_pimpl_t *p_old_pimpl, int x);
```

```C
int old_pimpl_get_x(const old_pimpl_t *p_old_pimpl);
```

The usage of the old private implementation is heavy:
```C++
auto p_op = old_pimpl_new();
old_pimpl_set_x(p_op, 42);
const auto X = old_pimpl_get_x(p_op);
old_pimpl_del(p_op);
```

## Wrapped integration

The obvious solution to avoid inheriting the cumbersome use of `C` is to adapt the old structures and interface with a single class.
For our `old_struct`, the `OldStructWrapper` class can be declared.

```C++
class OldStructWrapper final {
// ...
};
```

Aggregation with a public structure consists of a simple private declaration:
```C++
class OldStructWrapper final {
// ...
private:
    old_struct os;
// ...
};
```

Data manipulation can be hidden behind methods to get the most out of `C++`.
The client of the `OldStructWrapper` class will not have to explicitly manipulate pointers and references:
```C++
class OldStructWrapper final {
// ...
public:
// ...
    void set_x(int x) {
        old_struct_set_x(&os, x);
    }
// ...
};
```

The compiler may receive more complete information, such as the prohibition of not retrieving the result, or the purity of the method:
```C++
class OldStructWrapper final {
// ...
public:
// ...
    [[nodiscard]] auto get_x() const -> int {
        return old_struct_get_x(&os);
    }
// ...
};
```

The `OldStructWrapper` class can therefore be completely explicitly implemented:
```C++
/// Old struct wrapper
class OldStructWrapper final {
private:
    old_struct os; ///< Old struct composition
public:
    /// Old struct constructor
    OldStructWrapper() = default;

    /// Old struct destructor
    ~OldStructWrapper() = default;

    /// Old struct copy constructor
    OldStructWrapper(const OldStructWrapper &) = default;

    /// Old struct copy assignment
    /// \return Copy of other old struct wrapper
    OldStructWrapper &operator=(const OldStructWrapper &) = default;

    /// Old struct move constructor
    OldStructWrapper(OldStructWrapper &&) = default;

    /// Old struct move assignment
    /// \return Old struct wrapper
    OldStructWrapper &operator=(OldStructWrapper &&) = default;

    /// Property x setter
    /// \param x value to set
    void set_x(int x) {
        old_struct_set_x(&os, x);
    }

    /// Property x getter
    /// \return x value
    [[nodiscard]] auto get_x() const -> int {
        return old_struct_get_x(&os);
    }
};
```

There must be at least 3 lines of code to declare the class, 1 to associate the old structure and 2 lines for each function to be reused.
The more lines there are in my code and the less I feel comfortable as a written line is a line to maintain, but it must be admitted here that the addition remains acceptable.

For private implementations, the work is more complex.
For aggregation, the compiler can't know the size in memory at compile time, so you have to use a pointer:
```C++
class OldPimplWrapper final {
private:
    old_pimpl_t *p_op{nullptr};
// ...
};
```

The constructor and destructor must then be overridden to allocate the old private implementation and deallocate it:
```C++
class OldPimplWrapper final {
// ...
public:
    OldPimplWrapper() : p_op{old_pimpl_new()} {};

    ~OldPimplWrapper() {
        old_pimpl_del(p_op);
    }
// ...
};
```

In addition, the copying of data from old private implementations has to be considered when copying the wrappers:
```C++
class OldPimplWrapper final {
// ...
public:
// ...
    OldPimplWrapper(const OldPimplWrapper &other) : p_op{old_pimpl_new()} {
        set_x(other.get_x());
    }

    OldPimplWrapper &operator=(const OldPimplWrapper &other) {
        set_x(other.get_x());
        return *this;
    }
// ...
```

As well as moving the memory in the move constructor / assignment of the wrappers:
```C++
class OldPimplWrapper final {
// ...
public:
// ...
    OldPimplWrapper(OldPimplWrapper &&other) noexcept: p_op{nullptr} {
        p_op = other.p_op;
        other.p_op = nullptr;
    }

    OldPimplWrapper &operator=(OldPimplWrapper &&other) noexcept {
        p_op = other.p_op;
        other.p_op = nullptr;
        return *this;
    }
// ...
};
```

Before being able to encapsulate the old functions as in the case of the public structure:
```C++
class OldPimplWrapper final {
// ...
public:
// ...
    void set_x(int x) {
        old_pimpl_set_x(p_op, x);
    }
// ...
};
```

```C++
class OldPimplWrapper final {
// ...
public:
// ...
    [[nodiscard]] auto get_x() const -> int {
        return old_pimpl_get_x(p_op);
    }
// ...
};
```

The `OldPimplWrapper` class can therefore be completely explicitly implemented:
```C++
/// Old private implementation wrapper
class OldPimplWrapper final {
private:
    p_old_pimpl_t p_op{nullptr}; ///< Pointer to old private implementation, null by default

public:
    /// Old private implementation constructor
    OldPimplWrapper() : p_op{old_pimpl_new()} {};

    /// Old private implementation destructor
    ~OldPimplWrapper() {
        old_pimpl_del(p_op);
    }

    /// Old private implementation copy constructor
    /// \param other Old private implementation wrapper to copy
    OldPimplWrapper(const OldPimplWrapper &other) : p_op{old_pimpl_new()} {
        set_x(other.get_x());
    }

    /// Old private implementation copy assignment
    /// \param other Old private implementation wrapper to copy
    /// \return Copy of other
    OldPimplWrapper &operator=(const OldPimplWrapper &other) {
        set_x(other.get_x());
        return *this;
    }

    /// Old private implementation move constructor
    /// \param other Old private implementation wrapper to move
    OldPimplWrapper(OldPimplWrapper &&other) noexcept: p_op{nullptr} {
        p_op = other.p_op;
        other.p_op = nullptr;
    }

    /// Old private implementation move assignment
    /// \param other old private implementation wrapper to move
    /// \return Other
    OldPimplWrapper &operator=(OldPimplWrapper &&other) noexcept {
        p_op = other.p_op;
        other.p_op = nullptr;
        return *this;
    }

    /// Property x setter
    /// \param x value to set
    void set_x(int x) {
        old_pimpl_set_x(p_op, x);
    }

    /// Property x getter
    /// \return x value
    [[nodiscard]] auto get_x() const -> int {
        return old_pimpl_get_x(p_op);
    }
};
```

Here are at least 3 lines of code to declare the class, 1 to associate the old structure and 2 lines for each function to be reused, 1 line for the constructor, 2 for the destructor, at least 5 for copying and 6 for moving, unless they are explicitly deleted with the delete statement, which makes the class maintenance easier, but makes the client integration more complex.
Thus, the main drawback of this solution is the more or less important quantity of code to add between the old library and the client, and the cost that this implies in code maintenance.

The end result is valuable, old private structures and implementations can be manipulated as simple classes:
```C++
auto os = OldStructWrapper();
os.set_x(42);
const auto X = os.get_x();
```

```C++
auto op = OldPimplWrapper();
op.set_x(42);
const auto X = op.get_x();
```

And  copy / move constructor / assignment help the client with high level memory manipulation, especially for the fan of '`}`' as the best destructor.
```C++
OldStructWrapper os_cpy_1;
OldStructWrapper os_cpy_2;

{
    auto os = OldStructWrapper();
    os.set_x(42);
    os_cpy_1 = os; // Copy assignment
    os_cpy_2 = OldStructWrapper(os); // Copy constructor and move assignment
}

os_cpy_2.set_x(24);

const auto X_1 = os_cpy_1.get_x();
const auto X_2 = os_cpy_2.get_x();
```

```C++
OldPimplWrapper op_cpy_1;
OldPimplWrapper op_cpy_2;

{
    auto op = OldPimplWrapper();
    op.set_x(42);
    op_cpy_1 = op; // Copy assignment
    op_cpy_2 = OldPimplWrapper(op); // Copy constructor and move assignment
}

op_cpy_2.set_x(24);

const auto X_1 = op_cpy_1.get_x();
const auto X_2 = op_cpy_2.get_x();
```

## Smart pointers integration

What has just been presented is an integration solution based on the use of classes to hide the complexity of memory management, while providing cleaner access to data manipulation.
Access to data manipulation is just a plus and the key is to improve memory management, and in modern `C++` a native solution exists.
It is called smart pointers.
Smart pointers are one of the mechanics of RAII, whose objective is to create a notion of data ownership.
When the owner of a single pointer is destroyed (end of scope or end of life of an object), the single pointer is also destroyed, which is perfect for the fans of '`}`' as the best destuctor.
Use of smart pointers is very simple, the declaration of aliases can be done this way:
```C++
using UpOldStruct = std::unique_ptr<old_struct>;
```

Allocation and initialization can be even be done in a single instruction using standard library tools:
```C++
std::make_unique<T>(T *);
```

Such as in the following example:
```C++
auto up_os = std::make_unique<old_struct>();
old_struct_set_x(up_os.get(), 42);
const auto X = old_struct_get_x(up_os.get());
```

For the private implementation, the tools already exist to easily implement the use, the declaration for aliases is done as with a classical structure, specifying that we will provide a handle to the destructor:
```C++
using UpOldPimpl = std::unique_ptr<old_pimpl_t, void (*)(p_old_pimpl_t)>;
```

The creation is done by allocating the memory and providing the connection to the destuctor, and then manipulating the smart pointer like a classical structure pointer:
```C++
auto up_op = UpOldPimpl(old_pimpl_new(), old_pimpl_del);
old_pimpl_set_x(up_op.get(), 42);
const auto X = old_pimpl_get_x(up_op.get());
```
Note: as for now, `make_unique` can not be used with destructor overloading.

The same can be done with shared pointers:
```C++
std::make_shared<T>(T *);
```

```C++
auto sp_os = std::make_shared<old_struct>();
old_struct_set_x(sp_os.get(), 42);
const auto X = old_struct_get_x(sp_os.get());
```

```C++
auto sp_op = std::shared_ptr<old_pimpl>(old_pimpl_new(), old_pimpl_del);
old_pimpl_set_x(sp_op.get(), 42);
const auto X = old_pimpl_get_x(sp_op.get());
```

***

# Experimentation

## GitHub repository

The code source of the full example available is on [hnrck/c-integration-modern-cpp](https://github.com/hnrck/c-integration-modern-cpp).

The GitHub repository is structured as following:

```bash 
    .
    ├── CMakeLists.txt
    ├── LICENSE
    ├── README.md
    ├── extern
    │   └── Catch2
    ├── inc
    ├── old_func
    ├── old_pimpl
    ├── old_struct
    └── tests
```

* inc - containing the integration in headers `*.h`, using the legacy `C` in modern `C++`.
* [`CMakeLists.txt`](https://github.com/hnrck/c-integration-modern-cpp/blob/master/CMakeLists.txt) - allowing to build the project quickly.
* `old_*/inc` - containing the legacy `C` headers.
* `old_*/src` - containing the legacy `C` implementations.
* [`LICENSE`](https://github.com/hnrck/c-integration-modern-cpp/blob/master/LICENSE) - the license of the project.
* `tests` - containing the examples implementations runnable, as exposed in this article.

To test the project, the easiest way is to clone the github repository:

``` bash
git clone https://github.com/hnrck/c-integration-modern-cpp.git --branch 1.0.0
```

## Compilation

These commands build the legacy `C` libraries in `c-integration-modern-cpp/build/` as well as the integration executables.

``` bash
cmake -Bc-integration-modern-cpp-build -Hc-integration-modern-cpp
cmake --build c-integration-modern-cpp-build -t all
```

## Execution

The main applications can be executed with:

```bash 
cmake --build c-integration-modern-cpp-build -t test
```

[![asciicast](https://asciinema.org/a/orjHjbV7VHNKdRH8j1WF526UX.svg)](https://asciinema.org/a/orjHjbV7VHNKdRH8j1WF526UX)

# Conclusion

The integration of `C` in `C++` was designed a long time ago and the mechanisms are efficient but spartan, forcing the client code to take care to manage memory and data access.
When integrating into modern `C++`, where there is often a culture of good memory management and pointer manipulation, these mechanisms are not comfortable.
There are solutions to interface `C` code more cleanly, through classes or smart pointers.
None of these solutions is perfect and all of them have limitations, but they have their advantages that we will summarize in the following paragraph.


## Summary

Solution | Benefits | Drawbacks
---|---|---
Wrappers | Consistent use, copy and move of memory can be managed, No explicit pointer manipulation for the customer | Additionnal code to implement and maintain
Smart pointers | Very little code to implement and maintain, consistent use of smart pointers, clean memory management | Difficult copy, requires the customer to manipulate pointers, `make_unique` instruction can not be used with private implementation structure

## Final word

For `RROSACE`, the simulation model library was written in `C89`, and the simulation framework in `C++11`.

My first strategy was to use the least amount of code to lighten maintenance, so I preferred smart pointers.
Before presenting my work and publishing the code, I revised my strategy and switched to wrapping, which has another implicit advantage, that of not having to explain the mechanics of smart pointers and RAII to the audience.
