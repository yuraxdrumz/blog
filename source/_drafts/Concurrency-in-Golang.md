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
Concurrency is a complex and interesting topic, one that has many variations, be it multithreading, event loop, actor model or csp. Since the first multi processor, programming languages tried to find the sweet spot between developer ease to code and raw performance. Each concurrency model has its own benefits and drawbacks and in this post I am going to cover Golang's concurrency model which is called CSP style concurrency. If you want to check another concurrency model, I wrote about Node.js' event loop in detail [here](/2019/06/09/Node-JS-Event-Loop-0/).


Lately I have been using Golang quite a lot at work because I needed a language that is static typed, compiled and highly concurrent. But, before diving into the language, I decided to deep dive to Golang's concurrency model. I believe learning how a language operated behind the curtains, even a top level overview, will help tremendously even if you are not familiar with it.
To get fast performance we need parallel execution and to get parallel execution we need concurrency.