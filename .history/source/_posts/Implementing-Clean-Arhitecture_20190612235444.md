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
Last year, after reading Robert Martin's *Clean Architecture*, I decided to implement it in a project at work. One of the reasons, except my usual "I have to implement this cool thing right away!" was working on legacy projects accompanied with the good ol' *big ball of mud* code. Who of us does not want to take a look at legacy code from, lets say, 3 years ago and say: *wow! this project is pretty clean and straightforward, its written in X and Y*. The purpose of this post is to show you how can one implement *Clean Architecture* in practice and still understand it years from now, whether you work alone or in a team. Everything shown will be written in *Typescript* on *Node.js* using *Object Oriented* programming paradigm.

# Disclaimer
Some of the things that I am going to write and show are my personal experiences and opinions, you may have read Robert Martin's *Clean Architecture* and thought, interpreted or implemented otherwise. All the architectures have the same goals in the end.

## Core Idea
The idea behind *Clean Architecture* is that we have layers. Each layer is encapsulated by another layer and the only way to communicate is with *The Dependency Rule*. *The Dependency Rule* states that source code dependencies can only point inwards, meaning each layer can be dependant on the layer beneath it, but never the other way around. 
## Entities and Use-Cases
The core of this architecture are your entities, which represent your classes/types/basic methods. 
A layer above the entities layer is your use-cases. Use-cases are your application specific business rules, for example, if we are talking about a shopping cart, then `addToCart` will be a use case, because it needs to recieve a type `product` and, for examples sake, check rate extractor, afterwards check currency converter and then add to DB and return response. Also, you do not want to couple your use-cases to some input or output, for example having a dependency on a certain library/framework/database like *Express*, *GraphQL* or *MongoDB*, instead you want to pass a contract (interface) of some type in the constructor and pass the implementation itself at higher layers.
## Repository Pattern
For database interactions it is recommended to use the *Repository Pattern* which encapsulates all your database interactions through an abstraction layer. The repository pattern does give you a bit freedom to replace databases with ease, but this rule only applies when your interactions are basic CRUD operations! If you have a many to many relationships which require a graph database, switching to mongodb at the repository layer will not help you much as it is not built for that purpose, so take interactions into consideration at design level! 
## Interface Adapters
After the use-cases layer we have the Interface Adapters layer. Here, you convert your data from the form most convenient for entities and use cases, into the form most convenient for whatever persistence framework is being used, like the database, web or whatever you like. I like to call it, the implementations of the use-cases.

## Frameworks and Drivers
The last layer is the Frameworks and Drivers. Here you call all of your dependencies that conform to contracts you defined in your use-cases. That way you can replace dependencies without the use-cases knowing anything about it, according to the L in S.O.L.I.D, which is called the Liskov substitution principle.

## Liskov Substitution Principle
Liskov's substitution principle states that if a system is using a type **T** which is an implementation of type **S** and we switch the implementation to type **Z** which is also of type **S** , the behaviour of the program should not change.

A Small diagram taken from to illustrate our layers, notice the arrows only pointing inward!
[https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

![](./Implementing-Clean-Arhitecture/CleanArchitecture.jpg)
 

## Clean Architecture In Practice
lets build something overused, like a shopping cart. We will first decide what are our use cases and from that we would be able to conclude an initial data model - our entities. Later on, we will create Interface Adapters(implementations) and at the final later we will simply glue all of our dependencies and implementations and see how clean architecture could benefit us in future projects.

## Use-Cases
Because we chose a shopping cart, our use-cases will be pretty straight forward - `addToCart` and `removeFromCart`. Lets say `addToCart` needs to 

