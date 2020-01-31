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
In this post I will show what is Ports and Adapters and how I implemented it in Golang. Github repo is available [here](https://github.com/yuraxdrumz/ports-and-adapters-golang).

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
Example: GUI adapter, for when we need to convert events triggered by a GUI app to events defined by the port

### Prerequisites
Because Ports and Adapters does not define a specific folder structure, I will not focus on the folder structure itself, 
but you can take a reference from the structure in the github repository.
One thing to note is I replaced the `driven` ports with the name `out` and the driver ports with the name `in`, as it confused developers that are new to Ports and Adapters

### Ports and Adapters In Practice
lets build a shopping cart. Our cart will have two use cases, `addToCart` and `removeFromCart`.
We can create one port for all the use cases regarding the cart, or we can do a port per use case in the cart. It is up to you and the level of abstraction you seek.
I chose to put all the cart use cases under one port

#### Ports In Practice

Let's see an example of the port for the cart
`cart/ports.go`
```go
package cart

type Item struct {
	Name string
	Id string
	Description string
}

type Port interface {
	Add(item Item) error
	Remove(itemID string) error
}
```

Before we continue we need to see what kind of ports/adapters we need for our add and remove use cases. 
Let's say that we need a 3rd party warehouse service we talk and a database to save the changes.

Our warehouse package ports will look like this:
`warehouse/ports.go`

```go
package warehouse

type Port interface {
	checkIfAvailable(itemID string) (bool, error)
	removeItemFromWarehouse(itemID string) (bool, error)
}

```

Our Repository package ports will look like this:
`repository/ports.go`

```go
package repository

import "github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart"

type Port interface {
	addItemToDB(item cart.Item) (bool, error)
	removeItemFromDB(item cart.Item) (bool, error)
}
```
I like to use the repository design pattern to abstract database interaction. 
I already wrote about [here](/2019/06/11/Implementing-Clean-Architecture/#Repository-Pattern)

Did you notice we didn't choose how to interact with our app, or what database to use ? 
These architectural pattern allows us to delay these decisions to allow maximum focus of the developer on the business logic.

Now that we finished the ports part, lets look at the adapters:

#### Adapters In Practice