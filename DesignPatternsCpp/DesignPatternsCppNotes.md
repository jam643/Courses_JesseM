# Design Patterns In Modern C++
[Udemy Course Site](https://www.udemy.com/course/patterns-cplusplus/)

## Section 2: SOLID Design Principles
* Single responsibility (separation of concerns)
    * E.g. add journal entry and save functionality are different classes that just do one thing. Allows other types of object to have their own ‘save functionality in a centralized class.
    * Only have one reason to change internals
* Open-Closed principle:
    * Open to extension close to needing new modifications to support it. 
    * Demo specification design. Where `filter(items&lt;T>, specification&lt;trait>)` returns items with trait and we don’t need to write new spec for each trait but can template instead.
    * BR: maybe something like `solve(state, solver&lt;forces>,etc)`. But maybe limit openness if we never use it.
* Liskov Substitution Principle
    * Derived class should be substituted without issue (e.g. derived class methods should make sense for all). E.g. deriving ‘square’ from ‘rectangle’ can be misleading since setWidth may change both values and thus not interchangeable.
        * Soln is to have single class with extra params or use factory pattern to create entirely diff classes via similar builder call
* Interface Segregation Principle (YAGNI: you aint going to need it):
    * Don’t make interface too large which make too many assumptions about functionality of derived classes.
        * Solution is to break up into several interfaces. Can then have derived class inherit from multiple interfaces.
    * BR break up MPCController interface to e.g. bookkeep, solve, etc separately
* Dependency Inversion Principle
    * High level modules/classes should not depend on low level
        * Soln: add general functions to interface class, but keep implementation in derived classes. Then high level class can use pure virtual fns without depending on low level classes. 
        * Dependency injection related concept: provide derived class as default when constructing abstract class

## Section 3: Builder
* Gamma Categorization (design pattern categories):
    * Creational Patters: explicit creation and implicit (e.g. dependency injection, reflection, etc)
        * Single vs piecewise (step by step) initialization
    * Structural Patterns
        * E.g. concerned with structure of class members
        * E.g. pattern are wrappers that mimic underlying class
        * Good API design
    * Behavioral Patterns
        * No central theme
* Builder:
    * Construct object piece by piece rather than object with 10 args in constructor
    * XBuilder is separate class that has constructor and several functions that collect data. And then some function for constructing X.
    * Fluent Interface (chain build functions together in one line):
        * Can add chain of build commands in single line: e.g. `builder.add_child(“foo”).add_child(“bar”)` by having add_child return reference (or pointer) to builder. E.g. via `return *this` in add_child.
        * X class can also have a static method to create builder. E.g.: \
`X::create(“foo”).add_child(“bar”).add_child(“bar2”).build()`
        * Can prevent client from creating class X directly by making X constructor private and making XBuilder a friend.
        * Can add implicit conversion operator `operator X() const{return x}` or can explicitly return via `XBuilder.build()`
    * Groovy Style Builder:
        * Use uniform initialization (e.g. via {}) to provide succinct way of building objects
    * Builder Facets (use separate builders to construct separate aspects of single object):
        * separate builders that inherit from base builder. Note base builder should only have reference to obj to avoid copies of obj in each builder.
        * E.g. `Person::create().lives().at('bla').zip('bla').works().at('bla').earning('bla')` where lives and works are separate builder classes
        * Cool use of Clion auto generate ostream operator!
             
## Discussion Group
* Cool clion generation of constructors/ stream operators
* Interface segregation could contain debug structs
* Structured binding in C++17
    * `for(auto&& [first, second] : list)` like python. Cool!
* Polymorphism via reference and pointer. Ref is rare but can be done
* Dependency Injection?
* Builder not super useful if we build in one line, since that can just be constructor. Also seems bad practice that MPCController internally requires Builder rather than allowing TKSystem.
* Some MPCSolver/Controller constructors have many params and could benefit from builders
* Forbes builder demo:
    * TKSystem `x{1,2,3,4}` can be hard to catch bug to catch switched values. There is process to change call to TK `x{Speed{1}, Accel{2},...}` which makes more explicit. 
    * Can then create builders that allow construction with arguments in any order
    * This essentially recreates Named Arguments that python allows
* Can also enforce units which results in compiler error if we try Accel a = speed/time;
    * Enforced in Speed optimizer

## Factories
* motivation: obj creation too convoluted, constructor not descriptive
    * outsource non-piecewise (unlike builder) obj creation 
    * constructors are limiting: can't override if same param types, and need to have the same constructor name for diff types of construction
        * e.g. `Point(float x, float y)` and `Point(float r, float theta)` doesn't work with override
* Factory Method
    * static methods in class (e.g. `Point::NewCartesian` and `Point::NewPolar`) than can then call the `Point(float x, float y)` constructor, now private.
* Factory
    * separate static methods for constructing class to factory class. Need to either make factory a friend of Point (violates OCP), or make Point constructor public.
* Inner Factory
    * solves having to make factory a friend of main class to access private constructor by adding factory class within main class, and get control over it's lifetime
    * E.g. `Point::Factory::NewPolar(r, theta)`
    * Factory can also be private and exposed by instantiating it within Point as public static variable (controls lifetime of factory). E.g. `Point::factory.NewPolar(r, theta)`
* Abstract Factory
    * One stop shop for creating variety of derived classes with single interface:
        * If you have a variety of derived classes and corresponding derived factories. Then in base factory class, have `map<str, unique_ptr<BaseFactory>>` that allows single `make_obj(string)` function to call corresponding factory based on map, e.g. `auto drink = DrinkFactory("coffee")`
* Functional Factory
    * Factory class is not polymorphic and instead contains map `map<str, function<HotDrink>>` that maps string to lambda funcion within factory.
* Summary:
    * Factory takes car of object creation in less restraining way than constructor.
    * can be internal or external to class
    * can be used as one stop shop o create related objects
    
## Prototype
* motivation:
    * For creating new objects based on existing prototype (partially or fully initialized) object, perform deep copy of existing object (clone)
    * Problem for objects that contain pointer members, if object copied and change this member. Then member is also changed for original (shallow copy)
        * E.g. `auto jane = john`, `jane.address = 14` will modify john address if address member is a pointer
* Prototype
    * To create deep copy, can define copy constructor. E.g. where within copy constructor `address{new Address{*other.address}}` to deep copy (assuming Address has deep copy constructor)
* Prototype Factory
    * want to start with prototype (partially initialized obj) that can be used directly but can be deep copied to create full objects
    * e.g. static method in factory `unique_ptr<Contact> ContactFactory::new_employee(name, suite, Contact prototype)` returns completed/modified prototype
* Prototype via Serialization (easier deep copy using serialization)
    * Issue so far, need to implement copy constructor/assignment ourselves for each level. Other programming languages have 'reflection' which automates this
    * Serialize helps because it converts entire object into it's values (dereference) and then de-serializing will end up deep copying
    * can use boost serialization and make boost serialization Friend of private members of class to be serialized
    * syntax to serialize `ar & address` can handle the fact that address is pointer and will serialize value (assuming Address has it's own serialization method)
    * `auto jane = clone(*john)` where clone fn serializes and then deserializes
    * Interesting that there's no easier way to deep copy obj with embedded objects. Even serialization requires defining method in each level, but is better than copy constructor because it forces you to make a true deep copy and traverse the entire set of obj rather than accidentally shallow copying some vals.
    