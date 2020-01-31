---
title: Implementing Ports and Adapters
category: Architecture
tags: 
  - Ports and Adapters
  - Golang
  - Hexagonal
  - Software Architecture
---

There are different architectures that allow you to keep focus on your business domain and allow for fast paced development and changes. Examples would be: Clean Architecture, Onion Architecture and Ports and Adapters (also called hexagonal).
[In my previous post](/2019/06/11/Implementing-Clean-Architecture/), I talked about Clean Architecture and how it helps get your code more modular and developer friendly after a somewhat short learning curve. After joining a new team, I noticed Clean Architecture did not really settle and the team found it to be somewhat over abstracted, so I decided to play around with a variation of Ports and Adapters.
In this post I will show what is Ports and Adapters and how I implemented it in Golang. Github repo is available [here](https://github.com/yuraxdrumz/ports-and-adapters).

### Disclaimer
Some of the things that I am going to write and show are my personal experiences and opinions.

### Core Idea
Ports and Adapters architecture divides a system into several loosely-coupled interchangeable components. The components communicate with each other through abstracted API's with the use of interfaces and their implementations.
This approach is an alternative to the traditional layered architecture, where components are divided into layers. There are no layers in Ports and Adapters, only business logic <- ports <- adapters, meaning there are no restrictions on how to structure the applications, only that an adapter relies on a port and the business logic relies on different ports

### Business Logic
All of your business specific use cases.
Example: upon adding a user to a board, send an email, save the user in the database, grant permissions from an auth service to that user to view the board.


### Ports
The interfaces to all of the components in your system. There are two kinds of ports: driven and driver.

#### Driven Ports
Interfaces that your application business logic uses for its needs. 

#### Driver Ports
Interfaces for the outside world, a.k.a API

### Adapters
Implementations of our ports. They can be either driven or driver, depending on the port we use

#### Driven Adapter
Example: Service-To-Service adapter, for when we need to request some data from another service in our business logic use case

#### Driver Adapter
Example: GUI adapter, for when we need to convert events triggered by a GUI app to events defined the port

### Ports and Adapters In Practice
lets build something overused, like a shopping cart.