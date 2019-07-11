---
title: Context Maps in Domain Driven Design
date: 2019-07-10 15:52:33
category: Architecture
tags: 
  - DDD
  - Software Architecture
  - Context Maps
---
Ideally, it would be great to have a single place that incorporates all of our models, but in reality, our systems fragment to multiple models and we need to understand how to approach building them in way that allows future changes quickly.
Strategic Domain Driven Design is a high level approach to distributed software architecture and is an essential part of DDD. One of its features is context maps, which allows grasping the different relationships between bounded contexts (a boundary within which the ubiquitous language is consistent) and gives the teams a better understanding on how they affect each other. In this short post I will introduce you to the basics of context maps.

## How to approach Strategic DDD
When working on a project, we can have teams that are large by count or located abroad, we can also have external systems and legacy systems to communicate with. The idea is to have some sort of wiki that teams can refer to, that is neither overly complex nor missing information. Understanding the relationships early on can help diagnose problems, as wrong relationships between components can quickly turn into a *big ball of mud*.

## Types of context maps
Starting from the top, each context map type has greater level of communication between teams at the expense of control over the domain:
  - Shared Kernel - This is a shared domain model between two teams. Each team has an agreed upon subset of the domain along with its model.
  - Partnership - The teams have a mutual dependency on each other for delivery.
  - Customer-Supplier - The teams define interfaces to adhere and one team acts as a downstream (customer) to another team, which is the upstream (supplier). The upstream team can make changes without fear of breaking something downstream. The domains can evolve independently as long as the upstream context fulfills its interfaces.
  - Open Host Service - A bounded context that offers a defined set of functionalities exposed to other systems. Any downstream system can implement their own integration.
  - Published Language - Similar to Open Host Service, however it models a domain as a common language between bounded contexts
  - Conformist - The downstream team conforms to the model of the upstream and there is no translation of models between them. If the upstream is a mess and the dev team behind it does not want / cant change it, the mess is propagated downstream.
  - Anticorruption Layer (ACL) - A layer that isolates/abstracts the downstream's models from another system's models by translation.
  - Separate Ways - There is no connection between the bounded contexts. The teams can find their own solutions in ther domains.

## Example
![](./sddd.jpg)
Once we identify what are our domains and boundaries we can start drawing context maps between them. In this example we can see the U for upstream and D for downstream. The online banking services acts as a supporting or a generic subdomain to our core PFM Banking domain. We use ACL when we receive a response and model it appropriately internally. We have a partnership relationship with the Expense Tracking Domain, which means we need to deliver together as we may have a mutual dependency. Last, we have a conformist relationship with the Web User Profiling. Maybe, the team cant change or wont change their implementation of a model, so we need to do it on our side, on the core domain.


## Summary
We saw why do we need Strategic Domain Driven Design and how we can leverage a part of it called context maps to help us build better systems. We saw what kinds of context maps exist, what each one does and at last, we saw a small example with a few context maps.