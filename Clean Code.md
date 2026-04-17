# Clean Code

Test #2

Working code does not mean *good* code. Bad code works too. It would be easier if it didn't, but it does. In this chapter we'll mention some good practices when writing code, so the code will be easier to read, understand and maintain.

Before we get to real work, let's define what *good* code means.

* good code is elegant and pleasing to read. It is easy to understand what happens with variables and functions, hence it is easy to understand the *flow*
* good code is focused. Each function, variable, class or module has one purpose and one purpose only. Its responsibility is not polluted by surrounding details.
* good code is neat and simple
* good code does not contain duplication
* good code is thoroughly tested
* good code contains only what's necessary. That means it has only classes, methods and variables that are fundamental for required functionality
* good code is upgradeable. It should be simple to extend its functionality

Okay, now we are off to a good start. You know how good code looks, now let's see how we can achieve it.

## Clear names

The first rule of elegant code is: write clear names.
Here are some general guidelines:

* Camel Case is a must
* **Intention-Revealing names** - choose names that represent the true purpose of a variable/method/argument/class/package
  * use names which can be used in everyday language
    * when you discuss or explain your code, it will be much easier to do so
  * use searchable names and avoid names having poor variance
    * you will find what you are looking for much faster
    * the code will be more readable
  * avoid unnecessary prefixes like m_, o_
  * use names related to the domain you are working on and use naming with meaningful context
    * in case of no code documentation, such names will sometimes document the code itself
* **Method naming**
  * the name of a method should specify the exact purpose of a method
  * in most cases, the name should start with a verb (e.g checkAvailableRooms)
    * if a method returns a boolean, write its name as a question (e.g isEmpty, isRoomAvailable)
  * use consistent naming patterns across your codebase

## Methods

Java is an object-oriented language. This means that objects communicate with each other, usually by methods. It is important to write good structured methods:

* a method should do one task and one task only
* keep methods small enough to execute one task *well*
* comment parts of a method that might be unclear
* prefer pure functions when possible
* use method references for simple lambda expressions

## Objects and data structures

While coding, you are going to deal with objects and data structures. There is a difference between the two: objects hide their data behind abstractions and expose methods that operate on that data. Data structure expose their data and have no meaningful methods.

When working with objects, make sure to implement these rules:

* **Data Abstraction** - expose abstract interfaces that allow its users to manipulate the *essence* of the data, without having to know its implementation
* **The Law of Demeter** - method *f* of a class *C* should only call the methods of:
  * class *C*
  * an object created by *f*
  * an object passed as an argument to *f*
  * an object held in an instance variable of class*C*
* Don't let methods know too much
* Mixing different levels of details or abstraction in same function is troubling
* Consider using immutable objects when possible
* Use builder pattern for complex object creation
* Prefer composition over inheritance

## Error Handling

Error handling is an important part of your code. It can make or break your application, so make sure you do it right:

* Use exceptions rather than return codes
* Use unchecked exceptions and provide enough context to be able to log them
* Define exception classes when needed
* Wrap exceptions thrown by *third-party* API calls
* Never return nor pass *null*. Read this again.
* Use [special case pattern][1] to deal with special cases
* Use try-with-resources for resource management
* Consider using Optional for methods that might not return a value
* Use pattern matching for exception handling in Java 17+

## Boundaries

* Use non-existing code by defining a fake class and later use the [*Adapter*][2] pattern to bridge the gap to the real API
* The *Adapter* represents the only place to change when real API evolves
* Use interfaces to define clear boundaries between components

## Classes

A class is a template for an object. The goal is to code good classes because that leads to good code:

* Begin with a list of variables:
  * public static constants
  * private static variables
  * private instance variables
* Classes should be small and follow the single responsibility principle so the class has only one *reason to change*
* Maintain cohesion, which results in many small classes
* Organise for change and isolate from change by minimising coupling
* Consider using records for simple data classes
* Use sealed classes to define restricted hierarchies
* Prefer immutable classes when possible
* Use builder pattern for complex object creation
* Use dependency injection for better testability


[1]: https://martinfowler.com/eaaCatalog/specialCase.html
[2]: https://sourcemaking.com/design_patterns/adapter
