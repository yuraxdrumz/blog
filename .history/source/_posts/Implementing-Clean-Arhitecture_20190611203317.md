---
title: Implementing Clean Arhitecture
category: Architecture
date: 2019-06-11 21:31:56
tags: 
  - Node.js
  - Typescript 
  - Software Architecture
  - Clean Architecture
  - Uncle Bob
---
Last year, after reading Robert Martin's *Clean Architecture*, I decided to implement it in a project at work. I got tired of legacy systems written as a *big ball of mud* and tangled code, so I wanted to try and make a change at work, by introducing this style of architecture and giving a presentation about it to my co-workers. The purpose of this post is to show you how can one implement *Clean Architecture* in practice, in my case I chose **Typescript** as everyone at work were used to **Node.js** and I did not want to make drastic changes right away.

# Disclaimer
Some of the things that I am going to write and show are my opinions, you may have read Robert Martin's *Clean Architecture* and thought or interpreted otherwise and that is fine. All of the architectures have the same goals in the end and one of them is the separation of concerns.

# The Idea Behind Clean Architecture
##### Core Idea
The idea behind *Clean Architecture* is that we have layers. Each layer is encapsulated by another layer and the only way to communicate is with *The Dependency Rule*. *The Dependency Rule* states that source code dependencies can only point inwards, meaning each layer can be dependant on the layer beneath it, but never the other way around. 
##### Entities and Use-Cases
The core of this architecture are your entities, which represent your classes/types/basic methods. 
A layer above the entities layer is your use-cases. Use-cases are your application specific business rules. It is important to note that you do not want to couple your use-cases to some input or output, for example having a dependency on *MongoDB* or *Express*, instead you want to pass a contract(interface) of some type in the constructor and pass the implementation itself at higher layers. 
##### Repository Pattern
For database interactions it is better to use the *Repository Pattern* which encapsulates all your database interaction through an abstraction layer. Note, that the repository pattern does give you a bit freedom to replace databases with ease, but this rule only applies when your interactions are basic CRUD operations! If you have a many to many relationships which require a graph database, switching to mongodb at the repository layer will not help you much as it is not built for that purpose, so take that into consideration! 
##### Interface Adapters
After the use-cases layer we have the Interface Adapters layer. Here, you convert your data from the form most convenient for entities and use cases, into the form most convenient for whatever persistence framework is being used, like the database, web or whatever you like. The last layer is the Frameworks and Drivers. Here you call all of your dependencies that conform to contracts you defined in your use-cases. That way you can replace dependencies withput the use-cases knowing anything about it, according to the L in S.O.L.I.D, which is called the Liskov substitution principle.
##### Liskov Substitution Principle
Liskov's substitution principle states that if a system is using a type **T** which is an implementation of type **S** and we switch the implementation to type **Z** which is also of type **S** , the behaviour of the program should not change.

A Small diagram taken from [https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
insert image here
 
## Implementation