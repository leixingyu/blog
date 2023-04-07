---
title: "Object-Oriented Design: A Comprehensive Guide to Building Robust and Scalable Software Systems"
tags:
  - pipeline
categories:
  - learning log
date: 2023-04-02
description: Learn about important principles and techniques in Object-Oriented Design in this comprehensive guide. Discover how to create a conceptual model, use UML class diagrams, and apply software development principles and design patterns. Improve your software development skills and create efficient, reusable code with these essential tips and best practices.
---
  
> This blog is a second processed product/summary of the tutorial series: 
> [Linkedin Learning: Object-Oriented Programming](https://www.linkedin.com/learning/topics/object-oriented-programming).
> Bullet points extracted by me and reformatted with help from ChatGPT.

Object-oriented design is a fundamental concept in software development that 
focuses on modeling the behavior and characteristics of real-world objects in a program.
By understanding the basics of object identity, properties and attributes, behavior, and class, developers can create well-designed, efficient, and maintainable software.

## Introduction

**Object Identity:**
Objects are unique entities within a program that are separate and distinguishable from one another, even if they belong to the same class or type. Each object has a unique identifier that differentiates it from other objects, and this identity is essential for managing and manipulating objects within a program.

**Properties and Attributes:**
Objects have properties or attributes that define their characteristics, such as size, weight, color, and shape. These attributes are essential for representing real-world objects in a program and can be used to distinguish between different objects of the same type.

**Behavior:**
Objects have behavior or actions that they can perform, such as filling, swimming, or ringing. Behavior is typically defined as methods or functions that manipulate an object's attributes or interact with other objects in the program.

One of the challenges of object-oriented design is **identifying what can be an object 
in a program**. In general, objects should be represented by nouns and not verbs, 
and they should have a unique identity, attributes, and behavior. The focus should be on identifying the main objects in the program and their interactions with one another.

A class is a template for creating objects in a program. It defines the name, type, attributes, and behavior of objects that belong to that class. A class can be used to create multiple objects of the same type and is an essential component of object-oriented design.

### Steps for Object-Oriented Design

The process of object-oriented design involves analyzing and designing the program before writing the code. 
This includes gathering requirements, describing the application, identifying the main objects, describing the interactions between objects, and creating a class diagram using Unified Modeling Language (UML).

## Software/System Requirements

In object-oriented design, understanding the software/system requirements is essential for creating a 
well-designed, efficient, and maintainable program.

### Sets of Requirements
Software/system requirements can be divided into two sets of questions: 
- what does it need to do? 
- how should it do it? 

The first set of questions refers to the functionality of the program, while the second set refers to performance, legal, support, and security requirements.

- **Function Requirements:**
refer to what the system/application must do. These requirements capture the bare necessities of the program and define the core functionality that the program needs to provide.

- **Non-Functional Requirements:**
define how the system/application should do something. These requirements are concerned with aspects such as usability, reliability, performance, supportability, design, implementation, interface, and physical constraints.

### FURPS Requirement Model

The FURPS requirement model is a framework for thinking about software/system requirements. 
FURPS stands for functionality, usability, reliability, performance, and supportability. 

- **Functionality** 
refers to the capabilities of the system, while the other four elements are non-functional requirements.

- **Usability** 
focuses on the user and include factors such as human factors UI and documentation. 
The usability of the program is essential for ensuring that users can efficiently and effectively use the system.

- **Reliability** 
is concerned with predictability, availability, and failure rate. 
These requirements ensure that the program operates consistently and reliably.

- **Performance** 
relates to speed, efficiency, and limitations. These requirements ensure that the program operates efficiently and effectively, even when processing large amounts of data.

- **Supportability**
is concerned with extensibility, configurability, and testability. These requirements ensure that the program can be maintained and updated over time.

Design, Implementation, Interface, and Physical Constraints
In addition to the FURPS requirements, other non-functional requirements include design (how the program is built), implementation (the programming language used and standards to follow), interface (the ability of the program to interface with other systems), and physical constraints (the hardware limitations of the system).

## Use Case

The use case is a scenario that outlines a particular goal of an actor (user) with the system. It consists of the following elements:

```commandline
[Title] 
    Short and simple goal
[Primary Actor] 
    The user who desires to achieve the goal
[Success Scenario] 
    Steps or paragraphs describing how to achieve the goal
```

To create a use case,
identify the actors who interact with the system and separate them by their roles in the use case, not job titles.
Focus on the intent of the goal and not go into too specific steps or too broad a goal.
The scenario should be concise and avoid detailed technical implementation or UI elements.
Focus on the "sunny-day" scenario and avoid including rarely encountered scenarios.
Avoid needless words and keep the scenario short and concise.

## Conceptual Model

In object-oriented design, creating a conceptual model is a 
crucial step in developing a software or system. 
A conceptual model helps to identify the objects and their relationships that will form the backbone of the software. 
Here are the steps involved in creating a conceptual model.

- **Identify Objects:**
The first step is to identify the objects that will be used as class objects in the system. 
This can be done by looking at the user story or use case and picking out the relevant **nouns.** 

  - For example, in a user story about ordering a meal, the relevant objects might include "customer," "menu," and "order."


- **Identify Class Relationships:**
Once the objects have been identified, the next step is to connect them with lines to show their relationships.
It is important to write down the **verbs** on each line to avoid using generic words like "use." 
Instead, use verbs that describe the relationship between the objects. 

  - For example, in the meal ordering system, the "customer" might "place" an "order" from the "menu."


- **Figure Out Class Responsibilities:**
The final step is to figure out the responsibilities of each class. 
This involves identifying the behaviors or actions that each class will perform. 
This can be done by looking at the user story or use case and picking out the relevant verbs. 

  - For example, in the meal ordering system, the "menu" class might have the responsibility of "displaying" the available items while the "order" class might have the responsibility of "calculating" the total cost of the meal.

It is important to note that each behavior should belong to the class that is most responsible for it. 
A behavior usually involves multiple objects, but each object should be responsible for itself. 
Avoid putting too much responsibility on a single object, which can lead to a "god object" that controls everything in the system.

In addition to these steps, another useful tool for creating a conceptual model is CRC (Class Responsibility Collaboration) cards. 
These cards can be used to organize the classes and their responsibilities, making it easier to visualize the system and identify any potential issues.

## Class Diagram and Class Relationship

In object-oriented design, UML class diagrams are used to represent the structure of a system's classes and the relationships between them. 
Here are some important considerations when creating UML class diagrams.

Each class in a UML class diagram should have a name, attributes, and behaviors. 
When creating classes, it's important to focus on the behaviors of the objects, 
rather than just their attributes. This is because objects are meant to do things, 
not just hold data. 
In general, it's recommended to keep as many class attributes and methods as private as possible, 
only making them public if you are certain that other objects will need to use them.

**Constructor:**

Constructors are considered as behavior and should be treated as such when creating UML class diagrams.
In UML, a constructor is represented as a method with the same name as the class.

**Interface:**

In UML, abstract classes and interfaces are represented as types. 
Abstract classes are used to represent a general category of objects, 
while interfaces are used to represent shared capabilities or behaviors. 
For example, a draw() interface can be shared between different types of objects that can be drawn on a screen. 
While objects of different types cannot be categorized under the same abstract class, 
they can all share the draw() interface to be able to draw on the screen. 
A system can iterate through all objects and call the draw() interface to update the screen.

## Principles and Patterns

There are several principles and patterns that can help make your software development more efficient and effective.

1. Single Responsibility Principle: A class should have only one responsibility. Avoid creating god objects that try to do everything. Instead, split responsibilities into multiple classes.

2. Don't Repeat Yourself (DRY): Avoid duplicating code. Reuse code and extract common functionality into methods or classes.

3. You Ain't Gonna Need It (YAGNI): Don't overdo it. Only add features that are necessary and useful. Avoid adding unnecessary complexity.

4. Error Handling and Prompt to Guide Users: Design your software to handle errors gracefully and provide clear prompts to guide users. Good error handling can improve the user experience and prevent bugs.

5. Software Testing: Testing is an important part of software development. Write automated tests to ensure that your code is working as intended and to catch regressions.

6. Design Patterns: Design patterns are reusable solutions to common software design problems. 

### Design Pattern

They provide a template to help structure your code around. 
One of the most well-known books on design patterns is _"Design Patterns: Elements of Reusable Object-Oriented Software"_ 
by the "Gang of Four", which describes 23 patterns.

- **Creational Patterns:** 
These patterns deal with the instantiation of objects. Examples include Abstract Factory, Builder, Factory Method, Prototype, and Singleton.

- **Structural Patterns:** 
These patterns deal with how classes are designed and composed. Examples include Adapter, Bridge, Composite, Decorator, Facade, Flyweight, and Proxy.

- **Behavioral Patterns:** 
These patterns deal with communication between objects. Examples include Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, and Visitor.

By following these principles and patterns, you can create software that is more modular, maintainable, and scalable.

