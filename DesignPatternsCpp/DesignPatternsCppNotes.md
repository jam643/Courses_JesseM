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
    
## Singleton
* Overview
    * Least popular of original design patterns (if you need this, there's a design smell)
    * Only want constructor call once, prevent creating additional copies, avoid thread safety
    * Singleton is a component instantiated only once
* Singleton Implementation
    * delete copy constructor and copy assignment operator `Singleton(Singleton const&) = delete;` and `void operator=(Singleton const&) = delete`
    * Need `static Singleton& Singleton::get(){static Singleton sg; return sg;}` member function to be able to access class
    * Access via `Singleton::get().function()`. Note, can't even do `auto sg = Singleton::get()` since that calls assignment operator.
    * Any subsequent calls to `get()` will return the reference to the existing static instance rather than create a new one.
    * Con: may not be threadsafe in pre C++11 where you can get multiple instances in separate threads
* Testing Issues
    * Unit tests need to use reference to same `Singleton` instance if running together and is therefore constrained
* Singleton in Dependency Injection
    * Solution to above testing issue (test)
    * Make interface for Singleton `SingletonInterface` with the common pure virtual function, and create `DummySingleton` to override
    * In consuming class to Singleton, pass `SingletonInterface` in constructor (dependency injection) e.g. `Consumer(SingletonInterce &sg)`
    * Then production code can pass real Singleton and unittest can inject Dummy and test Consumer with Dummy
* Singleton Lifetime in DI Container
    * Using boost::di to specify lifetime of class to be singleton, e.g. `auto injector = di::make_injector(di::bind<IFoo>().to<Foo>().in(di::singleton))`, says to bind IFoo (interface) to create Foo derived instance instead and set lifetime to singleton (so creating multiple instances via `injector.create...` will make sure it all refers to same instance of `Foo`)
    * alternative to defining own singleton via copy constructor/assignment operator and specify it's lifetime directly
    * Dependency Injection sidenote:
        * E.g. rather than `Controller` creating `ForcesSolver` instance itself, you pass solver interface into controller constructor (i.e. injecting dependency, `Controller(SolverInterface solver)`) so that you can easily swap for different solvers.
        * DI containers, struct that maps class `Controller` to logic to create its dependencies if they haven't been created already. E.g. `DICont->get('Controller')` will inject default ForcesSolver. Can update DICont with `AcadoSolver` so it uses that in future.
* Monostate
    * member variables are all static, so `Monostate m1;` and `Monostate m2` will share member vars, if you change in one, it will change in the other instance
    * This is actually bad way to acheive singleton because it is unclear from the consumer what is happening and doesn't work with inheritance
* Multiton
    * Useful to limit number of instantiations but can do more than one (vs Singleton is just one)
    * e.g. `enum class Importance{primary, secondary, tertiary}` to specify allow 3 creations
    * `typedef Multiton<Object, Importance> mt`, then `auto ob2 = mt::get(Importance::secondary)` where it will return reference to existing object if already created for that enum
    
## Adapter
* Overview
    * We are moving from Creational to Structural patterns now
    * Adapts an existing interface X to conform to required interface Y
* Vector/Raster Demo
    * E.g. `VectorRectangle` class containing 4 lines with `LineToPointAdapter` struct to convert line to points for visualization
* Adapter Caching
    * avoid recomputing adapter output if adapter has already encountered object with same vals using caching
    * Interesting: hashing can be used for performant comparisons to check the equivalence of objects (using boost hash)
        * initialize `size_t 0x...` seed and then combine seed with object members (`boost::hash_combine(seed,obj.x)`)to create unique hash to obj contents
    * Adapter has `static map<size_t, Points> cache`, if `LineToPointCachingAdapter(Line& line)` already has a line with same hash in cache, then it returns that, else it adds to map
    
## Discussion
* cool table showing if user declares e.g. copy constructor, what compiler does with everything else
    * non declared vs deleted changes whether compiler or linker throws (compiler complains if deleted)
* Suggest using GMock instead of DummySingleton class for testing Singleton by mocking SingletonInterface

## Bridge
* Overview
    * Mechanism that decouples an abstraction/interface (hierarchy) from an implementation (hierarchy)
    * E.g. if have 2 shape types and 2 Renderer types, can avoid 4 separate class implementations
* Pimple Idiom
    * Implementation separated into Imp class and main class has pointer to Impl
    * Impl inherits from main class
    * E.g. in `Person.cpp`, have `Person::PersonImpl:greet(){cout<<"bla";)` but fn `Person::greet(){impl->greet();}`
    * Why have this?: 
        * Hide implementation from header (e.g. no private members listed in header, can go into Impl class defined in .cpp file)
        * Compilation speed. If some impl in header, then have to recompile when changes made. Although modern compilers are smart and fast already.
* Shrink-Wrapped Pimpl
    * Prepackaged Pimpl approach with templated `Pimpl<implClass>` class that contains impl as unique ptr.
    * Then in main class use `pimpl<Imp> impl;` in main class instead of `Impl* impl = new Impl()`
    * Why do we need wrapped Pimple to be able to use smart ptr? Can't we use that in traditional Pimpl approach? Can.
* Bridge Implementation
    * E.g. if have 2 shape types (circle, square) and 2 Renderer types (raster, vector), can avoid 4 separate class implementations
    * Shape template class is provided Renderer template class as dependency injection, `Shape::Shape(Renderer& renderer)`. Then `Circle.render(){renderer.render_circle(x,y,rad);}` doesn't need to know whether it is rendering with raster or vector renderer type.
    * Note, each renderer class still needs member function to handle each shape rendering (circle, square, etc). But this way, only need one Circle class since DI renderer handles the implementation
    * BR application: MPCController is passed MPCSolver in constructor like above example
## Composite
* Overview:
    * Allow treat both single (scalar) and composite object uniformly (e.g. Foo and Composite<Foo> share API)
    * Think: powerpoint can resize individual objects or group them together (composite) and perform same resizing of group (common API)
    * In general object can use other objects via inheritance/composition
* Geometric Shapes
    * GraphicObject interface (`draw()` pure virt fn) with derived class Circle and Group (where group contains `vector<GraphicObjects>`.
    * Because group inherits from same interface, it shares same API (`draw()`). But it can contain a group of other derived GraphicObjects (e.g. `vector<GraphicObjects> objects=[Circle, Circle, Group[Circle, Square]]`) 
* Neural Network
    * Idea is that both Neuron and NeuronLayer (`NeuronLayer : public vector<Neuron>`) should be able to connect_to each other interchangeably (Neuron has `vector<Neuron*> in, out;`).
    * Use CTRP (curiously recurring template pattern) base class `SomeNeurons<Self>` that defines `connect_to(T& other)` (where T is templated type of Neuron or NeuronLayer that is passed)
        * Then `Neuron : public SomeNeurons<Neuron>` and `NeuronLayer : public SomeNeurons<NeuronLayer>` and they don't need to override connect_to
        * Hack to get this to work though, Neuron needs to define begin() and end() to share interface with vector<Neuron> so that it can be called in parent SomeNeurons class interchangeably
        * Sidenote: CTRP is when `class Derived : public Base<Derived>` so base is templated with derived class. Useful in this case because base class definition`SomeNeurons::connect_to()` can implement function where type is cast as whatever derived class is in for loop: `for (Neuron& from : *static_cast<Self*>(this))`
* Array-Backed Properties
    * E.g. if you want to have avg/sum etc of member variables, make member variables an array with enum spcifying the index for each member.
    * e.g. `enum Abilities { str, agl, intl, count}; array<int, count> abilities;`, now easy to compute avg value of all ability members since it's in an array but also still have a way to access each with name.
* Overview
    * 'Duck Typing' allows us to add begin()/end() fns (returns `this` and `this+1;` respectively) to a scalar class so it can behave/masquerade like a collection and can use same code regardless.
    
## Discussion
* Pimpl why not use: more pointer dereferencing and abstraction that is hard for IDE's to resolve. Otherwise it's good for reducing 
* BR CRTP example: TransitionC1 and TransitionC2 share similar implementations but don't inherit from each other. CRTP could be used to move implementation to base class.
   
## Decorator
* Overview:
    * Want to augment object with additional functionality but don't want to alter existing code (OCP) and keep new functionality separate (SRP)
    * Options:
        * aggregate the decorator object
        * Inherit from decorated object
* Dynamic Decorator:
    * E.g.
    ```c++
      // decorator 1
      struct ColoredShape : Shape
      {ColoredShape(Shape &shape, const string &color)};
      // decorator 1
      struct TransparentShape : Shape{}; ...
      
      //usage
      Square square{5};
      ColoredShape red_square{square, "red"};
      TransparentShape tran_square{red_square, 0.5};
    ```
    * `ColoredShape` inherits from `Shape` base class but also is passed a shape derived instance at runtime to construct.
    * Allows e.g. `Circle` and `Square` to not duplicate color functionality and separates to different class.
    * B/c decorator inherits from baseclass, decorator instances can be fed into other decorators e.g. `TransparentShape`
    * Downside of dynamic/runtime nature is that ColoredShape obj will not have auto-complete, visible API of e.g. square methods that are not in Shape baseclass.
* Static Decorator
    * avoid prev mentioned issue with Dynamic Decorator not having clear API
    * Using Mixin inheritance, inherit from template `template <T> struct ColoredShape2 : T{static_assert(is_base_of<Shape, T>::value);}`
    * Can do:
    ```c++
      ColoredShape2<Circle> green_circle{"green", 5};
      green_circle.resize(2); //decorator access member fn of Circle not defined in Shape, not possible with Dynamic Decorator
      TransparentShape2<ColoredShape2<Square>> square{0.5, "blue", 10};
    ```
    * requires variading template to make passing variable args to constructor possible
    * In this way, c++ during compile time will create structs ColoredShape2 that inherit from Circle, Square, etc based on what's implemented rather than inherit from Shape baseclass as before.
* Functional Decorator
    * decorate function rather than class.
    * E.g. 
    ```c++
    template <class Func>
    Logger::Logger(const Func &func, const string&name)
    void Logger::operator()() const{cout<<name; func();}
    ```
    * wraps provided function so when `logger()` is called it will act like function call.
    * Modify this implementation to achieve function decorators for generic functions that accept args using variadic template, e.g. `logged_add(2,3)` wraps the function passed to the class during construction 
    * Note: to avoid having to pass function signature in constructor, need C++ to infer type based on provided value (which it doesn't do). Therefore need to 

## Facade
* Overview 
    * Expose several components through single interface
    * Provides simple easy to understand user interface over a large and sophisticated body of code
    
## Flyweight
* Overview:
    * Space optimization technique that uses less memory by storing externally the data associate with similar objects
* Handmade Flyweight
    * Problem: want `User user1{"John","Doe"}; User user2{"John","Smith"};` To share point to same "John" str rather than duplicate
    * `struct User` contains `static boost::bimap<uint32_t, string> names;` to allow bidirection map association
    * `static key add(string){}` adds string to map if doesn't exist and returns key (idx), called during constructor
    * `get_first_name()` and `get_last_name()` then returns string based on key member vars`names.left.find(last_name)->second`
    * thought: why use `map.find()`, since that assumes ordered map with O(log(n)). Why not have an unordered bimap with O(1) access?
    * Thought: could probably use `unordered_map<boost::hash, Object>` instead of bimap (like discussed in Adapter section). Maintains `O(log(n))` `add()` (`or O(1) if unordered`) and `O(1)` `get()` times, but removes need to search map by value. 
* Boost.Flyweight
    * `boost::flyweight<string> first_name, last_name;` to let boost handle flyweight. Can treat vars like normal string but under the hood, they avoid duplication.
        * Can access pointer via `&first_name.get()`
* Text Formatting
    * A simple idea of using a vect<TextRange> to represent begin and end indices of capitalized letters in a string (rather than use a bool for each char in string which uses unecessary extra memory) 
    * thought: weird name `get_range()` that actually appends a range and returns a ref so capitalize can be set

## Proxy
* Overview
    * A class that acts as an interface to a resource. The resource may be remote, expensive to construct, etc.
    * Proxy class has similar interface to the resource (so code doesn't need to be modified) but adds some addition functionality
    * Thought: Composite and Decorators are kinda like Proxy since it shares interface but adds functionality/properties
* Smart Pointer
    * Smart ptr is proxy for raw ptr. Shares similar API (e.g. `->` operator, return false if null, etc) but adds functionality like handling destruction
* Property Proxy (doesn't seem useful)
    * Rather than getter/setter, may want to access member directly e.g. `obj.strength = 10` but in this case `strength = Property<int>(value)` where template class Property overrides assignement= and conversion() operators. So it acts like int but can do additional actions during these operators.
    * Thought: Seems not great to expose fields (even if they are Properties) since it couples interface with implementation and removes ability to perform input validation
    * Sidenote: mentioned metaclass (which is proposed to be added to C++)
        * Would allow specifying different types of classes (e.g. and Interface) where compiler uses reflection and compile time programming to set methods to pure virtual instead of programmer. e.g.
        ```c++
        Interface MyInterface{
            method1(); // compile-time programmed to be pure virtual during compilation
            method2();
        }
        ```
* Virtual Proxy
    * E.g. with `struct Bitmap : Image` that loads bitmap in ctor and then has draw() fn, can create `LazyBitmap : Image` that defers loading to draw()
    * then `Bitmap* bitmap` is pointer in Lazy proxy to allow delayed construction.
* Communication Proxy
    * `LocalPong : Pingable` and `RemotePong : Pingable` share `ping()` API but remote version actual pings website and waits for response (even though it appears in process).
* Proxy vs Decorator
    * Proxy tries to provide identical interface rather than enhance interface with fields/features. Decorator also has reference to obj its decorating