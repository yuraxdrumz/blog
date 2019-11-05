---
title: Microservices - backend for frontend pattern
date: 2019-11-05 20:24:24
category: Architecture
tags: 
  - Microservices
  - Design
  - Node.js
  - Software Architecture
---
![](./legos-2.jpg)
In the last couple of months, I got the chance to refactor and design a huge project. This project was the backbone to many future projects and was of high priority. The project was a big monolith with an even bigger database without any documentation or people to explain the business requirements behind the code, so we had to refactor incrementally and figure stuff out along the way. In this post, I will explain what is the "Backend For Frontend" microservices pattern and why did my team and I choose it for our use case.

## The Monolith Refactor
After looking trough the code, our team quickly identified that the project was a monolith, not in the sense of - *it runs as a tiered software application on a single server*, but in the sense that - *the code was not modular and coupled*. We knew we had to break it down to microservices eventually. Breaking a large codebase right away may lead to accidental complexity and some may see it as a form of premature optimization, especially when we were not familiar with the business rules. We decided to tackle the business logic through several steps:
  1. If some business logic handles its own data (usually not the case in this project), create it as a separate project. It is not guaranteed to be a separate microservice.
  2. If some business logic is coupled with another only at the application level, we can refactor it and further decide where it belongs.
  3. If some business logic is coupled with another at the data layer, we can refactor the coupled logic to a separate service with the subset of data it needs from the current db (because we refactor incrementally, this approach requires to update the old db as well, until you refactor everything and replace it entirely. I prefer doing it with *change streams / triggers* from the database to avoid adding the insert at the application level). After seeing what it does and how it behaves, we can decide whether the logic belongs in the same service or not.
  4. rinse and repeat until everything is refactored

## If Not a Monolith Then What ?
Microservices are loosly coupled servers which are independently deployble and are in control of their own copy of the data (a subset of the entire application data) and together they form the one entity your clients see from the outside.
here are a few reasons we chose to go the microservices route:
  - The complexity of the app was too large for a single team/repo/server to handle
  - Smaller codebase for each team to work on and deploy independently
  - Faster deployments
  - Safer deployments, we avoid breaking the entire app, because it is loosly coupled
  - Freedom in choosing tech stack, language, database as every service is independant and knows how to speak to the other services

## Before You Go Microservices
Not everything is green on the other side, before you go down this rabbit hole, here are a few things to consider
  - Having a lot of small services means a lot of deployments from various teams, you will need strong CI/CD tools and best practices to keep up.
  - Integration testing and integration in general will be tougher, as in the real world some services will have to deploy together, so orchestration is needed.
  - Each service having its own copy of the data means your data will be eventually consistent at best, distributed transactions are usually a performance bottleneck
  - Identifying which services belong together and which are not is hard.
  - Communication is a pain in the butt. Do we need gRPC or Websockets or HTTP or Event bus? A lot of factors to consider

## Microservice communication

## Backend For Frontend

## Why

## How

## Pros

## Cons

## Conclusion