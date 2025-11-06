---
weight: 100
---
# GOF Design Patterns : Introduction

![](/images/Screenshot%202025-01-29%20092059.png)

## Introduction

In this series, we are going to discuss design patterns introduced in the book ***Design Patterns: Elements of Reusable Object\-Oriented Software***, which has been used in the industry for a long time.

The book ***Design Patterns: Elements of Reusable Object\-Oriented Software*** was published in 1994. It was authored by *Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides*. They discussed *23 design patterns* that can be used for solving commonly recurring problems in the industry. Since the authors are a group of four people, the book later got nicknamed ***Gang of Four Design Patterns***. In short, we call it GoF Design Patterns.

Here is the link to the book: [Link](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8)

## What Are Design Patterns?

Now let's start discussing our main topic: Design Patterns. Programming has been practiced for decades. During this time, developers have faced many different problems. However, most of these problems are common and recurring throughout the industry. This means developers encounter the same set of problems repeatedly.

Each developer has tried to solve these problems in various ways, leading to the discovery of more efficient solutions to these recurring issues. Over time, these efficient solutions became standard, fundamental knowledge across the industry.
These reusable templates for solving design problems in programming are called Design Patterns. A design pattern is essentially a reusable solution to a commonly occurring problem in many contexts of software design.

## Categories of Design Patterns

The GoF book divides these patterns into three categories:

1. ***Creational Design Patterns***: Patterns that deal with strategies to create objects.
2. ***Structural Design Patterns***: Patterns that deal with strategies to maintain the structure of classes in an application.
3. ***Behavioral Design Patterns***: Patterns that deal with strategies to achieve efficient interaction between objects.

## A Brief Overview

Now, let’s briefly explore the design patterns that fall into these categories and their purposes. Later, we will discuss each of these design patterns in a separate article.

### Creational

* [***Singleton***](/docs/lld/gof/creational/singleton/): Restricts multiple initializations of a class and ensures that only one instance of the class exists during the entire runtime.
* [***Factory***](/docs/lld/gof/creational/factory-method/): Provides an interface for creating objects in a superclass while allowing subclasses to alter the type of objects created.
* [***Abstract Factory***](/docs/lld/gof/creational/abstract-factory/): Similar to the Factory Pattern, it adds another layer of abstraction. It works around a super\-factory that creates other factories.
* [***Builder***](/docs/lld/gof/creational/builder/): Constructs complex objects step by step.
* [***Prototype***](/docs/lld/gof/creational/prototype/): Enables the creation of new objects by copying an existing object and modifying it as needed.

### Structural

* ***Adapter***: Provides an interface between two unrelated entities so that they can work together.
* ***Composite***: Organizes objects into tree structures to represent whole\-part hierarchies.
* ***Proxy***: Provides a surrogate or placeholder for another object to control access to it.
* ***Flyweight***: Optimizes memory usage by sharing a common state among multiple objects.
* ***Facade***: Hides the complexities of the system and provides an interface for clients to access the system without worrying about its complexity.
* ***Bridge***: Decouples functional abstraction from implementation so that both can vary independently.
* ***Decorator***: Modifies the functionality of an object at runtime.

### Behavioral

* ***Template Method***: Creates a template method stub and defers some implementation steps to subclasses.
* ***Mediator***: Provides a centralized communication medium between different objects in a system.
* ***Chain of Responsibility***: Achieves loose coupling by passing client requests through a chain of objects for processing.
* ***Observer***: Useful when you need to monitor the state of an object and get notified whenever it changes.
* ***Strategy***: Provides multiple algorithms for a specific task, allowing the client to choose the implementation at runtime.
* ***Command***: Encapsulates a request as an object, containing all the information needed to execute the request.
* ***State***: Allows an object to change its behavior based on its internal state.
* ***Visitor***: Defines a mechanism for traversing elements in an object structure and performing operations on them without altering their structure.
* ***Interpreter***: Defines a grammatical representation for a language and provides an interpreter to process the grammar.
* ***Iterator***: Traverses elements of a collection without exposing its underlying representation.
* ***Memento***: Saves and restores the previous state of an object without revealing its implementation details.

Now we have a basic understanding of the design patterns discussed in the GoF book. Let’s deep dive into each one, one by one.

Happy Coding 🙌