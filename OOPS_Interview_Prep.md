# Object-Oriented Programming (OOPs) — Interview Preparation

## 1. What is OOP?
A programming paradigm based on the concept of "objects," which bundle data (attributes) and behavior (methods) together. It contrasts with procedural programming, where logic and data are separate. OOP aims to make code modular, reusable, and easier to maintain by modeling real-world entities.

## 2. Class
A class is a blueprint or template for creating objects. It defines the attributes (variables) and methods (functions) that objects created from it will have. A class itself doesn't occupy memory for data until an object is instantiated.

## 3. Object
An object is a runtime instance of a class. It has its own copy of the instance variables (unless static) and can invoke the methods defined in the class. Objects are created using constructors.

## 4. The Four Pillars of OOP

### 4.1 Encapsulation
Wrapping data (variables) and methods that operate on that data into a single unit (class), and restricting direct access to some of an object's components using access modifiers (private, protected, public). This is achieved through getters/setters. Benefit: protects internal state from unintended external interference, improves modularity.

### 4.2 Abstraction
Hiding complex implementation details and showing only the essential features of an object. Achieved through abstract classes and interfaces. Difference from encapsulation: abstraction hides complexity at the *design* level (what to show), while encapsulation hides data at the *implementation* level (how to protect it).

### 4.3 Inheritance
A mechanism where a new class (child/derived/subclass) acquires the properties and behaviors of an existing class (parent/base/superclass). Promotes code reuse. Types:
- **Single inheritance** – one child, one parent
- **Multilevel inheritance** – chain of inheritance (A→B→C)
- **Hierarchical inheritance** – one parent, multiple children
- **Multiple inheritance** – one child, multiple parents (not directly supported in Java/C# due to the Diamond Problem; supported in C++ via virtual base classes; achieved in Java via interfaces)
- **Hybrid inheritance** – combination of the above

### 4.4 Polymorphism
"Many forms" — the ability of an object/method to behave differently based on context.
- **Compile-time (Static) Polymorphism**: Method Overloading — same method name, different parameter list, resolved at compile time.
- **Runtime (Dynamic) Polymorphism**: Method Overriding — subclass provides a specific implementation of a method already defined in its parent class, resolved at runtime via dynamic method dispatch (virtual functions in C++).

## 5. Constructors and Destructors
- **Constructor**: A special method automatically called when an object is created; used to initialize object state. Types: default, parameterized, copy constructor.
- **Destructor**: Called automatically when an object is destroyed/goes out of scope, used to release resources (relevant mainly in C++; Java/C# use garbage collection instead).

## 6. Access Modifiers
- **Private**: accessible only within the class.
- **Protected**: accessible within the class and its subclasses (and same package in Java, depending on language).
- **Public**: accessible from anywhere.
- **Default/Package-private** (Java): accessible within the same package.

## 7. Abstract Class vs Interface
| Aspect | Abstract Class | Interface |
|---|---|---|
| Methods | Can have both abstract and concrete methods | Traditionally only abstract methods (Java 8+ allows default/static methods) |
| Variables | Can have instance variables | Only public static final constants |
| Inheritance | A class can extend only one abstract class | A class can implement multiple interfaces |
| Constructor | Can have a constructor | Cannot have a constructor |
| Use case | "is-a" relationship with shared code | Defines a contract/capability ("can-do") |

## 8. Method Overloading vs Overriding
| Overloading | Overriding |
|---|---|
| Same class, same method name, different signature | Parent-child classes, same method name and signature |
| Resolved at compile time | Resolved at runtime |
| Achieves compile-time polymorphism | Achieves runtime polymorphism |
| Return type can differ | Return type must be same or covariant |

## 9. Association, Aggregation, and Composition
- **Association**: A general relationship between two independent classes (e.g., Teacher and Student).
- **Aggregation**: A "has-a" relationship where the child can exist independently of the parent (weak ownership) — e.g., a Department has Professors, but Professors can exist without the Department.
- **Composition**: A stronger "has-a" relationship where the child cannot exist without the parent (strong ownership) — e.g., a House has Rooms; if the House is destroyed, the Rooms cease to exist.

## 10. Static vs Instance Members
- **Static (class) members**: belong to the class itself, shared across all objects, accessed without creating an object.
- **Instance members**: belong to individual objects; each object has its own copy.

## 11. This / Super Keyword
- **this**: refers to the current object instance; used to resolve naming conflicts between instance variables and parameters.
- **super**: refers to the immediate parent class; used to call parent constructors/methods.

## 12. Virtual Functions and Dynamic Binding (C++ specific)
A virtual function is a member function declared in a base class that can be overridden by a derived class, enabling runtime polymorphism via a vtable (virtual table) mechanism. Without `virtual`, function calls are resolved at compile time (static binding).

## 13. Diamond Problem
Occurs in multiple inheritance when a class inherits from two classes that both inherit from a common base class, causing ambiguity about which path's method/attribute to use. C++ solves this with virtual inheritance; Java avoids it by disallowing multiple class inheritance (only interfaces).

## 14. SOLID Principles
- **S** — Single Responsibility Principle: a class should have only one reason to change.
- **O** — Open/Closed Principle: classes should be open for extension but closed for modification.
- **L** — Liskov Substitution Principle: subtypes must be substitutable for their base types without breaking the program.
- **I** — Interface Segregation Principle: clients should not be forced to depend on interfaces they don't use.
- **D** — Dependency Inversion Principle: depend on abstractions, not concrete implementations.

## 15. Object Slicing (C++)
Happens when a derived class object is assigned to a base class object (by value), causing the derived-specific data to be "sliced off," leaving only the base part.

## 16. Friend Function/Class (C++)
A function or class that is not a member of a class but is granted access to its private and protected members.

## 17. Exception Handling in OOP
Using try/catch/finally (or try/except) blocks to handle runtime errors gracefully without crashing the program, keeping objects in a consistent state.

## 18. Garbage Collection
An automatic memory management feature (in Java, C#, Python) that reclaims memory occupied by objects no longer referenced, preventing memory leaks. Contrast with C++, where the programmer manually manages memory (new/delete).

## 19. Interface Achieving Multiple Inheritance
In Java, since multiple class inheritance is not allowed, interfaces let a class implement multiple interfaces to gain multiple "capabilities" without ambiguity, since interfaces (pre-Java 8) had no method bodies.

## 20. Coupling and Cohesion
- **Coupling**: degree of interdependency between modules/classes — low coupling is desirable.
- **Cohesion**: degree to which elements within a module belong together — high cohesion is desirable.

---

## 21. Constructor Types (know the four)
```java
class Point {
    int x, y;
    Point()                { this(0, 0); }        // default / no-arg
    Point(int x, int y)    { this.x = x; this.y = y; }   // parameterised
    Point(Point other)     { this(other.x, other.y); }   // copy constructor
}
```
- If you write **no** constructor, the compiler supplies a default no-arg one. If you write **any** constructor, that free one disappears.
- **Constructor chaining**: `this(...)` calls another constructor in the same class; `super(...)` calls the parent's. `super()` runs implicitly first if you do not write it.
- **Order of construction**: parent constructor → child field initialisers → child constructor body.
- Constructors have no return type, cannot be `static`, `final`, or `abstract`, and are not inherited (though they are invoked through `super`).
- In C++ there is also a **destructor** `~Point()` called automatically when the object goes out of scope, and it should be `virtual` in any base class you delete polymorphically.

## 22. Static Keyword — Every Meaning
| Context | Meaning |
|---|---|
| Static variable (class member) | One copy shared by all objects; belongs to the class |
| Static method | Callable without an instance; cannot use `this` or access instance members directly |
| Static block (Java) | Runs once when the class is first loaded, used for one-time initialisation |
| Static nested class (Java) | Nested class that does not hold a reference to the outer instance |
| Static local variable (C/C++) | Retains its value between calls; scope is local, lifetime is the whole program |
| Static function (C) | Internal linkage — visible only in that translation unit |

**Why can't a static method be overridden?** Overriding is resolved at runtime using the object; static methods are bound at compile time using the reference type. Redeclaring a static method in a subclass is **hiding**, not overriding.

## 23. Final / Const
- `final` variable — assign once (a constant). `final` method — cannot be overridden. `final` class — cannot be extended (`String` is final).
- A `final` reference means the *reference* cannot be reassigned, not that the object is immutable.
- C++ `const` on a member function (`int get() const;`) promises the function does not modify the object, which lets it be called on `const` objects.
- **Immutability** is what you actually want in many designs: make fields `private final`, provide no setters, do not leak mutable internals, and make the class `final`.

## 24. Shallow Copy vs Deep Copy
Shallow copy duplicates the object but copies references, so both objects share the nested objects — mutating one is visible through the other. Deep copy recursively duplicates everything, so the copies are independent. Java's `Object.clone()` is shallow by default; deep copy requires overriding `clone()`, using a copy constructor, or serialising and deserialising. In C++, the compiler-generated copy constructor is shallow, which is why any class holding a raw pointer needs the **Rule of Three** (destructor, copy constructor, copy assignment) — or the Rule of Five in modern C++ (adding the move constructor and move assignment).

## 25. equals() and hashCode() Contract (Java) / operator== (C++)
- If two objects are `equals()`, they **must** have the same `hashCode()`. The reverse is not required.
- Break this and hash-based collections misbehave: two equal keys land in different buckets, so a `HashMap` lookup silently fails.
- Always override both together, using the same fields for both.
- `==` compares references for objects (identity); `equals()` compares content, if overridden.

## 26. Composition Over Inheritance
Inheritance is a compile-time, permanent "is-a" relationship that exposes the parent's implementation to the child — a change in the base class can break every subclass (the fragile base class problem). Composition is a runtime "has-a" relationship: the class holds an object and delegates to it, so behaviour can be swapped and the coupling is only to the interface.

Prefer inheritance only when the subtype genuinely satisfies the Liskov Substitution Principle. The classic failure: `Square extends Rectangle` breaks because setting the width of a square must also change its height, violating what callers expect from a Rectangle.

## 27. Design Patterns Worth Naming (creational, structural, behavioural)
- **Singleton** (creational) — exactly one instance with a global access point. Used for a configuration object, a logger, a connection pool. Must be made thread-safe (double-checked locking with a `volatile` field, or an enum/eager initialisation in Java).
- **Factory Method** (creational) — a method decides which concrete class to instantiate, so callers depend on the interface, not the implementation.
- **Builder** (creational) — constructs a complex object step by step; avoids telescoping constructors.
- **Adapter** (structural) — wraps an incompatible interface so it can be used where another is expected.
- **Decorator** (structural) — adds behaviour to an object at runtime by wrapping it, without changing its class.
- **Facade** (structural) — one simplified interface over a complicated subsystem.
- **Observer** (behavioural) — subjects notify a list of subscribers when their state changes. This is the model behind event listeners and React state updates.
- **Strategy** (behavioural) — encapsulate interchangeable algorithms behind a common interface and pick one at runtime. My **ModeOS** backends are a textbook Strategy: an abstract `AudioBackend` with WirePlumber, PulseAudio, ALSA, and Mock implementations chosen at runtime by availability. My **ModelAuth** detectors are the same shape — four interchangeable detection algorithms behind one calling convention.
- **MVC** — separates data (model), presentation (view), and input handling (controller).

## 28. Generics / Templates
- Purpose: write code once that works for many types while keeping compile-time type safety, and avoid casting.
- Java uses **type erasure** — generic type information is removed at compile time, so `List<String>` and `List<Integer>` are the same class at runtime, and you cannot do `new T[]`.
- C++ uses **templates**, which are instantiated per type at compile time — faster and more flexible (the type must merely support the operations used), but produces larger binaries and famously verbose errors.
- **Bounded types**: `<T extends Comparable<T>>` restricts what T can be.

## 29. Exception Handling Details
- **try / catch / finally / throw / throws**. `finally` always runs — used for releasing resources — except on `System.exit()` or a JVM crash.
- **Checked exceptions** (Java) must be declared or handled; they represent recoverable conditions like a missing file. **Unchecked** (RuntimeException) represent programming errors like null dereference or bad index.
- **Error** is not meant to be caught (`OutOfMemoryError`, `StackOverflowError`).
- **Custom exceptions**: extend `Exception` (checked) or `RuntimeException` (unchecked) to give a domain-meaningful name.
- Best practice: catch the most specific exception first, never swallow an exception silently, do not use exceptions for ordinary control flow, and prefer try-with-resources (Java) or RAII (C++) so cleanup is automatic.
- **RAII** in C++: resource acquisition is initialisation. An object acquires a resource in its constructor and releases it in its destructor, so leaving scope — even by exception — always cleans up. This is why C++ has no `finally`.

## 30. Memory Management Across Languages
- **C/C++**: manual. `malloc`/`free`, `new`/`delete`. Mistakes cause memory leaks, double frees, dangling pointers, and use-after-free bugs. Modern C++ replaces raw pointers with **smart pointers**: `unique_ptr` (single owner, moved not copied), `shared_ptr` (reference counted), `weak_ptr` (non-owning, breaks reference cycles).
- **Java**: automatic garbage collection. The heap is split into young generation (Eden + two survivor spaces) and old generation; most objects die young, so minor GCs are cheap. Collectors: Serial, Parallel, CMS, G1, ZGC. `finalize()` is deprecated and unreliable.
- **Python**: reference counting plus a cyclic garbage collector for reference cycles.
- A **memory leak in a garbage-collected language** is still possible: an unused object that is still reachable — a static collection that keeps growing, an unremoved listener, or a cache with no eviction.

---

# Practice Questions with Answers (OOPs)

### Conceptual / Definition-based

**1. What is OOP and why is it used?**
A paradigm that models a program as a collection of objects, each bundling data (attributes) with the behaviour that operates on it (methods). It is used because it maps closely to how we think about real problems, keeps related code together, restricts access to internal state, and makes systems easier to extend and reuse. The alternative — procedural programming — separates data and functions, which scales poorly as a codebase grows.

**2. Explain the four pillars with real examples.**
**Encapsulation** — a `BankAccount` keeps `balance` private and exposes `deposit()`/`withdraw()` so no caller can set a negative balance directly. **Abstraction** — you call `car.start()` without knowing about the fuel injection; an interface names what is possible without saying how. **Inheritance** — `SavingsAccount extends Account` reuses common behaviour and adds interest. **Polymorphism** — a single `List<Shape>` where calling `draw()` on each element runs the circle's or the square's version.

**3. Difference between a class and an object.**
A class is a blueprint or type — a compile-time construct describing what fields and methods instances will have; it occupies no memory for instance data. An object is a runtime instance created from that blueprint, with its own copy of the instance fields and its own identity. One class, many objects.

**4. What is encapsulation and how is it achieved?**
Bundling data with the methods that operate on it, and restricting direct access to the internal state. Achieved with private fields plus public getters and setters, so the class controls every read and write and can validate, log, or change the internal representation without breaking callers. It is what makes a class's invariants enforceable.

**5. What is abstraction and how does it differ from encapsulation?**
Abstraction is about **design** — deciding what to expose and hiding complexity behind a simple interface. Encapsulation is about **implementation** — physically restricting access to data using access modifiers. Abstraction hides *complexity*, encapsulation hides *data*. Abstraction is achieved with abstract classes and interfaces; encapsulation with access modifiers.

**6. What is polymorphism and what are its types?**
"Many forms" — one interface serving multiple underlying types. **Compile-time (static) polymorphism** is method overloading and operator overloading, resolved by the compiler from the argument types. **Runtime (dynamic) polymorphism** is method overriding, resolved from the actual object type at execution time using a virtual table. Runtime polymorphism is what makes code work with types written after it was compiled.

**7. What is inheritance and what are its types?**
A mechanism where a class acquires the fields and methods of another. Types: single, multilevel (A→B→C), hierarchical (many children of one parent), multiple (many parents — supported in C++, not for classes in Java), and hybrid. It enables reuse and establishes an "is-a" relationship, but it also couples the child to the parent's implementation, which is why composition is often the safer default.

### Comparison-based

**8. Abstract class vs interface.**
An abstract class can hold state (instance fields), constructors, and both abstract and concrete methods, with any access modifier — and a class can extend only one. An interface (classically) declares only a contract with no state, all members implicitly public, and a class may implement many, which is how Java gets multiple inheritance of *type*. Modern Java interfaces can carry `default` and `static` methods, but still no instance state. **Use an abstract class** when subclasses share code and state and are clearly the same kind of thing; **use an interface** when unrelated classes need to promise the same capability.

**9. Method overloading vs overriding.**
Overloading: same method name, different parameter list (type, number, or order), within the same class or inherited; resolved at compile time; the return type alone cannot distinguish overloads. Overriding: a subclass provides its own implementation of an inherited method with the *same* signature; resolved at runtime; the overriding method cannot reduce visibility and cannot throw broader checked exceptions. Overloading is polymorphism of convenience; overriding is polymorphism of substitution.

**10. Association vs aggregation vs composition.**
Association is any relationship between two classes (a Teacher and a Student). Aggregation is a weak "has-a" where the part can outlive the whole (a Department has Professors; delete the department and the professors still exist). Composition is a strong "has-a" where the part cannot exist without the whole (a House has Rooms; destroy the house and the rooms go with it). In code, composition usually means the owner creates and owns the lifetime of the part.

**11. Compile-time vs runtime polymorphism.**
Compile-time (overloading) is resolved by the compiler from static types, is faster, and offers no runtime flexibility. Runtime (overriding) is resolved via a vtable lookup using the object's actual type, costs one indirection, and is what makes frameworks and plugin architectures possible.

**12. Shallow copy vs deep copy.** See section 24.

**13. `this` vs `super`.**
`this` refers to the current object — used to disambiguate a field from a parameter of the same name, to pass the current object, or as `this(...)` to chain constructors. `super` refers to the parent — used to call the parent's overridden method (`super.draw()`), access a hidden parent field, or invoke the parent constructor with `super(...)`, which must be the first statement.

**14. Static vs instance members.**
Static members belong to the class, exist once regardless of how many objects are created, are initialised when the class is loaded, and are accessed via the class name. Instance members belong to each object and are created with it. A static method cannot access instance members without an object reference, because there is no `this`.

**15. Aggregation vs inheritance — when to use each?**
Inheritance when the relationship is genuinely "is-a" and substitutable (LSP holds); composition/aggregation when it is "has-a" or when you want to swap behaviour at runtime. See section 26 for the full argument.

### Scenario/Application-based

**16. Design a library management system using OOP.**
Classes: `Book` (isbn, title, author, copies), `Member` (id, name, borrowed list), `Loan` (book, member, issue date, due date), `Library` (catalogue, members, issue/return operations). Use an abstract `LibraryItem` with `Book`, `DVD`, `Magazine` as subclasses so new item types can be added without touching the lending logic (open/closed principle). `Member` and `Loan` are an aggregation — a loan references a member but does not own it. A `FineCalculator` interface with different strategies (student versus faculty) is a Strategy pattern. Encapsulate `availableCopies` so it can never go negative.

**17. How would you model a payment system with multiple payment methods?**
Define an interface `PaymentMethod { PaymentResult pay(Amount a); }` and implement `CreditCardPayment`, `UpiPayment`, `NetBankingPayment`, `WalletPayment`. A `PaymentProcessor` holds a `PaymentMethod` reference and calls `pay()` polymorphically — it never needs a chain of if-else on the type. Adding a new method means adding one class and no edits elsewhere: that is the Open/Closed Principle plus the Strategy pattern. A factory maps a user's chosen type string to the right implementation.

**18. Where did you use OOP in your own projects?**
**ModeOS** is the clearest: `modeos/backends/base.py` declares abstract base classes `AudioBackend`, `DisplayBackend`, and `NightLightBackend` using Python's `ABC` and `@abstractmethod`. Each declares `name`, `is_available()`, `get_*()`, and `set_*()`. Concrete subclasses implement each real system (WirePlumber, PulseAudio, ALSA, sysfs backlight, brightnessctl, xrandr, GNOME gsettings, KDE D-Bus, gammastep, redshift) plus a Mock implementation for testing. At startup the app asks each backend `is_available()` and picks the first that works. That is abstraction (the caller never knows which tool is used), polymorphism (one call site, many implementations), and the Strategy pattern — and it is why the same code runs on GNOME, KDE, and inside a Docker container. `ModeConfig` is a dataclass with a `from_dict` factory method that validates and clamps every field, which is encapsulation of the validation rules.

**19. How does OOP help in large codebases?**
It gives you boundaries. Each class owns its data and invariants, so a bug is localised. Interfaces let teams work in parallel against a contract before an implementation exists. Polymorphism removes sprawling conditionals. Inheritance and composition remove duplication. And a well-named class hierarchy is documentation — a new developer can read the type names and understand the domain.

### Code-based / Tricky

**20. Can a constructor be private? Why would you do that?**
Yes. It prevents outside instantiation, which is used for the **Singleton** pattern (a private constructor plus a static accessor), for **static utility classes** that should never be instantiated, for **factory methods** that control which instance you get, and for the **Builder** pattern where only the builder may construct the object.

**21. Can you override a static method? A private method? A final method?**
No to all three, but for different reasons. Static: bound at compile time by reference type, so redeclaring it in a subclass **hides** it. Private: not visible to the subclass at all, so a same-named method is simply a new, unrelated method. Final: explicitly forbidden by the compiler — that is what `final` means.

**22. What is the diamond problem and how is it solved?**
If B and C both inherit from A, and D inherits from both B and C, then D gets two copies of A's members and it is ambiguous which to use. C++ solves it with **virtual inheritance** (`class B : virtual public A`), which makes D share a single A subobject. Java avoids it by forbidding multiple class inheritance; interfaces are safe because they carry no state, and if two interfaces provide conflicting `default` methods the compiler forces the implementing class to override and disambiguate explicitly with `Interface.super.method()`.

**23. What is object slicing?**
A C++ problem. Assigning a derived object to a base-class **value** copies only the base part — the derived fields are "sliced" off and virtual dispatch is lost. `Base b = derived;` slices. The fix is to use references or pointers: `Base& b = derived;` or `Base* b = &derived;`. This does not occur in Java or Python because variables are always references.

**24. What is a virtual function and how does it work internally?**
A member function declared `virtual` in the base class, so calls through a base pointer or reference dispatch to the derived override. The compiler gives each polymorphic class a **vtable** — an array of function pointers — and each object a hidden **vptr** pointing to its class's vtable. A virtual call is a vptr dereference plus an indexed jump, costing one extra indirection. A **pure virtual** function (`virtual void draw() = 0;`) has no implementation and makes the class abstract.

**25. Why should a base class destructor be virtual?**
If you `delete` a derived object through a base-class pointer and the destructor is not virtual, only the base destructor runs — the derived part is never cleaned up, leaking whatever it owned. Making it virtual makes destruction dispatch correctly. Rule: any class intended to be inherited from and deleted polymorphically needs a virtual destructor.

**26. What is a friend function?**
A C++ function or class granted access to another class's private and protected members. It is not a member of the class and has no `this`. Used mainly for operator overloading where the left operand is not your class (`operator<<` for streams) and for tightly coupled helper classes. It deliberately breaks encapsulation, so it should be rare and justified.

**27. What happens if a constructor throws an exception?**
The object is never fully constructed, so its destructor is not called — but the destructors of any fully-constructed member subobjects and base classes *are*. This is why raw resource acquisition in constructors is dangerous and why RAII wrappers (smart pointers) are the correct pattern: each member cleans itself up.

### Design principle-based

**28. Explain the SOLID principles with an example each.**
**S — Single Responsibility**: a class should have one reason to change. A `Report` class that formats *and* emails a report has two; split it into `ReportFormatter` and `ReportMailer`. **O — Open/Closed**: open for extension, closed for modification. Adding a new payment method should mean a new class, not editing a switch statement. **L — Liskov Substitution**: a subclass must be usable anywhere its parent is, without surprising the caller. `Square extends Rectangle` breaks it. **I — Interface Segregation**: many small interfaces beat one fat one. Do not force a `Printer` class to implement `scan()` and `fax()`. **D — Dependency Inversion**: depend on abstractions, not concretions. A `NotificationService` should take a `MessageSender` interface, so email, SMS, or a mock can be injected — which is also what makes it testable.

**29. What are coupling and cohesion, and what do you want?**
Coupling measures how much one module depends on another's internals; cohesion measures how focused a single module is. You want **low coupling and high cohesion**. Low coupling means a change in one place does not ripple; high cohesion means everything in a class belongs together. Dependency injection and programming to interfaces both lower coupling.

**30. What is DRY, KISS, and YAGNI?**
DRY (Don't Repeat Yourself) — every piece of knowledge should have one authoritative representation. KISS (Keep It Simple) — prefer the simplest thing that works. YAGNI (You Aren't Gonna Need It) — do not build for imagined future requirements. They are in tension with SOLID sometimes, and knowing when to stop abstracting is a real skill.

### Miscellaneous

**31. Is Java purely object-oriented? Is C++? Is Python?**
Java is not *purely* object-oriented because it has primitive types (`int`, `char`, `boolean`) that are not objects — though autoboxing hides this. C++ is a multi-paradigm language: you can write pure C in it. Python is arguably the closest to purely object-oriented of the three, because literally everything including integers, functions, and classes themselves is an object — but it does not enforce OOP. Smalltalk is the usual "purely object-oriented" answer.

**32. How does Python do OOP differently?**
No true access modifiers — a single underscore `_x` is a convention meaning "internal", and a double underscore `__x` triggers name mangling, but nothing is truly private. Multiple inheritance is allowed and resolved by the **MRO (Method Resolution Order)** using the C3 linearisation algorithm, accessible via `ClassName.__mro__`. `self` is explicit. Abstract classes come from the `abc` module. `@property` turns a method into an attribute-style getter, which is how Python does encapsulation without ceremony. **Duck typing** means an object's suitability is decided by which methods it has, not by which class it inherits from.

**33. What is garbage collection and how does it work?**
The runtime automatically reclaims memory for objects that are no longer reachable. **Reachability** is determined by tracing from GC roots (stack variables, static fields, active threads). Mark-and-sweep marks everything reachable and frees the rest; a copying/generational collector additionally compacts. Java's heap is generational because most objects die young. You cannot force GC — `System.gc()` is only a suggestion.

**34. What is an interface with default methods, and why was it added?**
Java 8 allowed interfaces to have method bodies marked `default`. It was added so new methods could be added to an existing interface (like `Collection.stream()`) without breaking every class that already implements it — backward compatibility. It is not a way to smuggle state in, since interfaces still cannot hold instance fields.

**35. Explain the difference between an abstract method and a concrete method.**
An abstract method declares a signature with no body and forces subclasses to implement it; a class with even one abstract method must itself be abstract and cannot be instantiated. A concrete method has an implementation that subclasses inherit and may optionally override.

---

# Rapid-Fire One-Liners

- **Message passing** — objects interacting by calling each other's methods.
- **Instance variable / field** — per-object state. **Local variable** — per-method-call state on the stack.
- **Access modifier order (Java, widest to narrowest)**: `public` → `protected` → default/package-private → `private`.
- **Method signature** — name plus parameter types. The return type is *not* part of it in Java.
- **Constructor overloading** — multiple constructors distinguished by their parameter lists.
- **`instanceof` / `isinstance`** — runtime type check; heavy use of it usually means you should be using polymorphism instead.
- **Upcasting** — a derived reference assigned to a base reference; always safe and implicit. **Downcasting** — the reverse; needs an explicit cast and can throw at runtime.
- **Marker interface** — an interface with no methods used purely to tag a class (`Serializable`, `Cloneable`).
- **Inner class vs static nested class** — an inner class holds an implicit reference to the enclosing instance; a static nested one does not.
- **Anonymous class / lambda** — an unnamed one-off implementation of an interface, defined at the point of use.
- **Covariant return type** — an override may return a subtype of what the parent returned.
- **Cohesive constructor rule** — a constructor should leave the object in a fully valid state; never require callers to call an `init()` afterwards.

---

# Linking OOP to My Projects

- **ModeOS** — abstract base classes with `ABC`/`@abstractmethod`, four families of pluggable backends, a `ModeConfig` dataclass with a validating factory method, and runtime backend selection. Pure Strategy pattern and dependency inversion.
- **ClassRoom Code** — the server is layered into routes, services, and a database layer, so a route never touches SQL directly (separation of concerns). The database engines (`sqlite.js`, `postgres.js`, `oracle.js`, `mongodb.js`) all satisfy one common interface behind `dbEngines/index.js`, so the judging code is written once and works for any engine. Same with the executors: Judge0 and the local runner are interchangeable behind `execution.js`. Zod schemas encapsulate validation.
- **NetSpecter** — the v2 architecture has a `DetectorEngine` dispatcher and a family of protocol detector classes (`FTPDetector`, `MailDetector`, `RedisDetector`, `TokenDetector`) sharing a `DetectionResult` model. Adding a new protocol means adding one class and registering it — Open/Closed in practice.
- **ModelAuth** — four detector functions with an identical calling convention and an identical return shape (`index`, `flagged`, plus statistic), so `evaluate.py` benchmarks all of them through the same `compute_metrics(detector_fn, ...)` function. That uniform contract is what made a fair comparison possible.
