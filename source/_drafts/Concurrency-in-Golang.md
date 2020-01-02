---
title: Concurrency in Golang
tags:
  - Golang
  - Go
  - Concurrency
  - Parallelism
  - Threads
  - Green Threads
category: Programming
---

![](./concurrency.png)
Concurrency - processing a lot of data at the same time. Sounds simple isn't it ? Well, its not! Do not confuse concurrency with parallelism. Operations can be concurrenct on a single thread, where each computation gets a fraction of execution time and parallelism is when multiple computations occur at the same slice of time. Concurrency allows our applications to run faster and eventually can lead to parallelism, thanks to multicore processors, but it also opens the door to many pitfalls. That is why there are different types of concurrency models out in the wild, each trying to address a specific set of problems. In this post, I will explain Golang's concurrency model which is called CSP (Communicating Sequential Processes) and show its benefits and drawbacks along with some examples. If you are on a concurrency spree, make sure to check out Node.js' concurrency model called the event loop [here](/2019/06/09/Node-JS-Event-Loop-0/).


## Concurrency vs Parallelism
Before I continue on to explain Golang's CSP model, I want to make sure you understand the difference between concurrency and parallelism.

## What's wrong with multithreaded applications?

## Enter CSP

## CSP Pros

## CSP Cons

## Examples


## Conculsion

 because concurrenct operations lead to parallel operations which usually get better performance than single threaded operations, I say usually because there are exceptions, mainly due to context switching between OS threads take a lot of valuable time. one that has many variations, be it multithreading, event loop, actor model or csp. Since the first multi processor, programming languages tried to find the sweet spot between developer ease to code and raw performance. Each concurrency model has its own benefits and drawbacks and in this post I am going to cover Golang's concurrency model which is called CSP style concurrency. 


Lately I have been using Golang quite a lot at work because I needed a language that is static typed, compiled and highly concurrent. But, before diving into the language, I decided to deep dive to Golang's concurrency model. I believe learning how a language operated behind the curtains, even a top level overview, will help tremendously even if you are not familiar with it.
To get fast performance we need parallel execution and to get parallel execution we need concurrency.