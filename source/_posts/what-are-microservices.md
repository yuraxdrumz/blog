---
title: What Are Microservices
tags:
  - Microservices
  - Design
  - Node.js
  - Backend For Frontend
  - API Gateway
  - Sidecar Pattern
  - Software Architecture
category: Architecture
date: 2019-11-05 20:24:24
---

![](./legos-2.jpg)
After having written and implemented several microservice architectures, I wanted to have a go at explaining microservices from my point of view and share my insights. In this post I will explain what are microservices, what are their pros and cons, how they communicate and the different approaches towards building microservices.

## What are microservices
There are a lot of explanations out in the wild regarding what are microservices. Everyone keeps mentioning loose coupling and independant deployments, but in my experience, that is not always true. Sometimes, you will have to deploy several microservices together because they are coupled, a.k.a, orchestration. In my point of view, microservices need to control their own copy of the application data and they should be self contained, meaning, the business logic they handle should only be tied to a specific microservice and to it alone.

## Microservices Pros
  - The complexity of the app is too large for a single team/repo/server to handle
  - Smaller codebase for each team to work on and **if possible** deploy independently
  - Faster deployments
  - Safer deployments, we avoid breaking the entire app, each of the services should be self contained
  - Freedom in choosing tech stack, language, database as every service is independant and knows how to speak to the other services

## Microservices Cons
  - Having a lot of small services means a lot of deployments from various teams, you will need strong CI/CD tools and best practices to keep up.
  - Integration testing and integration in general will be tougher, as in the real world some services will have to deploy together, so orchestration is needed.
  - Each service having its own copy of the data means your data will be eventually consistent at best, distributed transactions are usually a performance bottleneck
  - Identifying which services belong together and which are not is hard.
  - Communication is a pain in the butt. Do we need gRPC or Websockets or HTTP or Event bus? A lot of factors to consider

## Microservices communication
Microservices can communicate with each other either sync or async:

### Sync - HTTP 1.1
Pros:
  * Most tutorials, examples and implementations are in HTTP, so it is easier to pick it up and get started

Cons:
  * For applications that need to do long running work and/or react to certain events, this type of design can quickly become a bottleneck

### Async - gRPC, websockets, events
Pros:
  * Makes more sense when the application does a lot of background work and with the use of gRPC/websockets notifies the user when it is done.

Cons:
  * Tougher to manage, as each event can trigger a lot of reactions which cascade to more events being created.

## Approaches to building microservices

### Side Car
Expose the services directly. Each service will have a `side-car`, which is a service attached to each of your services, hence side-car, which does all the logging, forwarding, service discovery, ssl termination, authentication and authorization.
![](./side-car.png)

Pros:
  * Direct client/server communication, means better performance than dealing with extra layers

Cons:
  * Must conform to client communication mechanism. If the client uses HTTP 1.1 your services must use it too.

### API Gateway
Expose an `API Gateway`, which will act as another layer between your services and the clients. 

![](./gateway.png)

Pros:
  * Services can be hidden from the public internet, which means better security
  * Internal communication can be whatever you like, because you have an extra translation layer

Cons:
  * Another layer to take care of
  * Reduced performance because of said layer

### API Composer
This acts as an `API Gateway`, but it does the requests to the services on behalf of the client and composes their responses as well.

![](./api-composer.png)

Pros:
  * Removes the burden from the client of making multiple requests to the server, as the composer composes all the responses for the client
  * Decouples the services even further as their communication is reduced

Cons:
  * Easy to leak functionality outside of services
  * Extra layer to take care of

### Backend For Frontend
This acts like an `API Composer`, but you create a composer per client (mobile vs tv). This allows you to make different data requests to your inner services based on the client. For example you have a mobile client and a tv client, which both request data streams from your services. The data and bandwidth needs of your mobile and tv greatly differ, so instead of dealing with managing compatability with a lot of clients in a single composer, you create a composer per client.

![](./bff.png)

Pros:
  * Allows each team to focus on their specific client needs without breaking the other composers. All the composers rely on the services layer.
  * Allows the team that writes the front end to use the composer to their needs as long as the services layer can support their needs, hence the backend for frontend pattern.

Cons:
  * Easy to leak functionality outside of services
  * Duplication of code between multiple composers

## Wrapping Up

We saw what are microservices, how they communicate, what do they offer and the different techniques to build them.
Before rushing to build microservices, remember that there is a lot more to them. 
People say monolith like it is a bad word, when in reality, they talk about the code and that it is coupled, undocumented and not modular. 
It is perfectly fine and desired to run your code on a single server with a single database on a single platform, as long as you keep your code modular.
Microservices are an **evolutionary step** you need to take as your business needs progress and become more demanding/complex  to the point microservices are helpful and not a burden.