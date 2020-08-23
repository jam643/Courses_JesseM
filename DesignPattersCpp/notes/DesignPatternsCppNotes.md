---
tags: [DesignPatterns]
title: DesignPatternsCppNotes
created: '2020-08-23T05:31:28.092Z'
modified: '2020-08-23T05:35:27.757Z'
---

# Design Patterns In Modern C++
[Udemy Course Site](https://www.udemy.com/course/patterns-cplusplus/)

## Section 2: SOLID Design Principles
*   Single responsibility (separation of concerns)
    *   E.g. add journal entry and save functionality are different classes that just do one thing. Allows other types of object to have their own ‘save functionality in a centralized class.
    *   Only have one reason to change internals
*   Open-Closed principle:
    *   Open to extension close to needing new modifications to support it. 
    *   Demo specification design. Where `filter(items&lt;T>, specification&lt;trait>)` returns items with trait and we don’t need to write new spec for each trait but can template instead.
    *   BR: maybe something like `solve(state, solver&lt;forces>,etc)`. But maybe limit openness if we never use it.
*   Liskov Substitution Principle
    *   Derived class should be substituted without issue (e.g. derived class methods should make sense for all). E.g. deriving ‘square’ from ‘rectangle’ can be misleading since setWidth may change both values and thus not interchangeable.
        *   Soln is to have single class with extra params or use factory pattern to create entirely diff classes via similar builder call
*   Interface Segregation Principle (YAGNI: you aint going to need it):
    *   Don’t make interface too large which make too many assumptions about functionality of derived classes.
        *   Solution is to break up into several interfaces. Can then have derived class inherit from multiple interfaces.
    *   BR break up MPCController interface to e.g. bookkeep, solve, etc separately
*   Dependency Inversion Principle
    *   High level modules/classes should not depend on low level
        *   Soln: add general functions to interface class, but keep implementation in derived classes. Then high level class can use pure virtual fns without depending on low level classes. 
        *   Dependency injection related concept: provide derived class as default when constructing abstract class

## Section 3: Builder
*   Gamma Categorization (design pattern categories):
    *   Creational Patters: explicit creation and implicit (e.g. dependency injection, reflection, etc)
        *   Single vs piecewise (step by step) initialization
    *   Structural Patterns
        *   E.g. concerned with structure of class members
        *   E.g. pattern are wrappers that mimic underlying class
        *   Good API design
    *   Behavioral Patterns
        *   No central theme
*   Builder:
    *   Construct object piece by piece rather than object with 10 args
    *   XBuilder is separate class that has constructor and several functions that collect data. And then some function for constructing X.
    *   Fluent Interface (chain build functions together in one line):
        *   Can add chain of build commands in single line: e.g. `builder.add_child(“foo”).add_child(“bar”)` by having add_child return reference (or pointer) to builder. E.g. via `return *this` in add_child.
        *   X class can also have a static method to create builder. E.g.: \
`X::create(“foo”).add_child(“bar”).build()`
        *   Can prevent client from creating class X directly by making X constructor private and making XBuilder a friend.
        *   Can add implicit conversion operator `operator X() const{return x}` or can explicitly return via `XBuilder.build()`
    *   Groovy Style Builder:
        *   Use uniform initialization (e.g. via {}) to provide succinct way of building objects
    *   Builder Facets (use separate builders to construct separate aspects of on object):

## Discussion Group
*   Cool clion generation of constructors/ stream operators
*   Interface segregation could contain debug structs
*   Structured binding in C++17
    *   `for(auto&& [first, second] : list)` like python. Cool!
*   Polymorphism via reference and pointer. Ref is rare but can be done
*   Dependency Injection?
*   Builder not super useful if we build in one line, since that can just be constructor. Also seems bad practice that MPCController internally requires Builder rather than allowing TKSystem.
*   Some MPCSolver/Controller constructors have many params and could benefit from builders
*   Forbes builder demo:
    *   TKSystem `x{1,2,3,4}` can be hard to catch bug to catch switched values. There is process to change call to TK `x{Speed{1}, Accel{2},...}` which makes more explicit. 
    *   Can then create builders that allow construction with arguments in any order
    *   This essentially recreates Named Arguments that python allows
*   Can also enforce units which results in compiler error if we try Accel a = speed/time;
    *   Enforced in Speed optimizer

