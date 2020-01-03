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
Concurrency - processing a lot of data in a non order fashion. Sounds simple isn't it? Well, its not! Do not confuse concurrency with parallelism. Operations can be concurrenct on a single thread, where each computation gets a fraction of execution time and parallelism is when multiple computations occur at the same slice of time. Concurrency allows our applications to run faster and can lead to parallelism, thanks to multicore processors, but it also opens the door to many pitfalls. That is why there are different types of concurrency models out in the wild, each trying to squeeze the maximum performance out of a system in its own way. In this post, I will explain the concurrency model that Golang utilizes, which is called CSP (Communicating Sequential Processes) and show its benefits and drawbacks along with some examples. If you are on a concurrency spree, make sure to check out Node.js' concurrency model called the event loop [here](/2019/06/09/Node-JS-Event-Loop-0/).


## Concurrency vs Parallelism
I remember my father bringing home my first pc with a `Pentium 3` processor and a `Windows 95` OS on it. I immediately started playing with `Paint` and a bunch of other programs and I got the sense of illusion that my pc was fast as hell. You automatically think, parallel execution, right? Actually, `Pentium 3` was single core, meaning there could only be one computation happening at some slice of time, so how come the pc acted like it did everything in parallel? The answer is concurrency. 

By default, when your app requests resources from the OS, it receives one random core to run on (single thread), unless you explicitly ask to create more OS threads which will run on other cores, if available.


Now, let's say we have 1 core available. The maximum parallel computations we can achieve is 1. And let's say we have 50 applications open on our 1 core pc, it means we have `1 threads * 50 applications = 50 threads` running. If the OS were to give each thread its full time to finish, our pc would have one app working and everything else stuck in place, which is neither efficient nor good user experience. Instead, the OS has a complex piece of software component called a `scheduler`.


The scheduler's job is to give a slice of time for each thread to do some computations and when the scheduler decides to switch to another thread, even if the previous one didn't finish, it puts it aside and gives some time for another thread to run. That way, all of our applications receive some time to run and our system as a whole feels more alive. That is called `concurrency`

Now, when does concurrency become parallelism? When you have multiple cores. Take our example above,and change 1 core to 4. Everythings works the same, but faster, because we have 4 workers doing some work instead of 1. But don't be fooled, with parallel work we have a new set of problems. 



## What's wrong with multithreaded applications?

## Enter CSP

## CSP Pros

## CSP Cons

## Examples


## Conculsion

 because concurrenct operations lead to parallel operations which usually get better performance than single threaded operations, I say usually because there are exceptions, mainly due to context switching between OS threads take a lot of valuable time. one that has many variations, be it multithreading, event loop, actor model or csp. Since the first multi processor, programming languages tried to find the sweet spot between developer ease to code and raw performance. Each concurrency model has its own benefits and drawbacks and in this post I am going to cover Golang's concurrency model which is called CSP style concurrency. 


Lately I have been using Golang quite a lot at work because I needed a language that is static typed, compiled and highly concurrent. But, before diving into the language, I decided to deep dive to Golang's concurrency model. I believe learning how a language operated behind the curtains, even a top level overview, will help tremendously even if you are not familiar with it.
To get fast performance we need parallel execution and to get parallel execution we need concurrency.