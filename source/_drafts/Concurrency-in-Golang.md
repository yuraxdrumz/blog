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
Concurrency is about `handling` a lot of things at once, but, do not confuse concurrency with parallelism, which is about `executing` a lot of things at once. Concurrency allows our applications to run faster and can lead to parallelism, thanks to multicore processors, but it also opens the door to many pitfalls. That is why there are different types of concurrency models out in the wild, each trying to squeeze the maximum performance out of a system in its own way. In this post, I will explain what is concurrency, how it leads to parallelism, what kinds of concurrency models we have and at last, an in depth look at how the concurrency model that Golang utilizes, which is called CSP (Communicating Sequential Processes), works and show its benefits and drawbacks along with some examples. If you are on a concurrency spree, make sure to check out Node.js' concurrency model called the event loop [here](/2019/06/09/Node-JS-Event-Loop-0/).


## Concurrency vs Parallelism

### Prerequisite
When your app requests resources from the OS, it receives one random core to run on (single thread). The initial thread can create more OS threads which will run on other cores, if they exist.

Now, let's say we have 1 core available and we have 50 single threaded applications running, it means we have `1 thread * 50 applications = 50 threads` running. If the OS were to give each thread time to run until it terminates, our pc would have one app working and everything else stuck, which is neither efficient nor good user experience. Instead, the OS has a complex piece of software component called a `scheduler`.

### The Scheduler
The scheduler's job is to give a slice of time for each thread and when it decides to switch to another thread, it puts the currently executing thread aside. With this interrupting technique, which is called `context switching`, all of our threads receive some time to run and our system as a whole feels more responsive.

### Context Switching
When the scheduler decides to switch context to another thread, it needs to clear all the registers of the core that was chosen, take the new threads latest instruction to run, populate all the registers with the new data and then start executing. All this work is spent switching context instead of doing computations. 

The context switch performance hit will depend on a lot of factors, but it is safe to say it will be several thousands cpu cycles on each context switch.
That is why IO intensive operations, like handling network calls, will benefit from multicore cpus, but CPU intensive operations, like number crunching, will suffer.


### Concurrency Leads to Parallelism
The handling of multiple operations, each at its own slice of time is what is called `concurrency`. Now, add multiple cores to the equation. When you have several computations happening on several cores and the timing between the computations overlaps, the concurrent operations become parallel and we have what is called `parallelism`.

Take our example above and change 1 core to 4. All of our applications would work the same if they were single threaded, but the system as a whole would be faster. If our applications were multithreaded, they would have the potential to be even faster, but don't be fooled, with parallel work we have a new set of problems.

## What's Wrong With Multithreaded Applications?

### The Problem
Before multithreading, developers were writing imperative code (one, that executes line by line) on a single thread and now they need to think about mutexes, semaphores, atomic data structures, cpu caches, deadlocks and race conditions. Another thing is, if we have cpu intensive applications, we need to make sure that multithreading benefits us or otherwise we will have even worse performance than a single threaded application.

### A Solution
To help developers focus more on writing their business logic instead of thinking about mutexes and race conditions, different concurrency models came to life:

  - Event loop - everything is run on a single thread with the event loop delegating all async IO operations to the OS and waiting for callbacks and all other synchronous operations like file system operations are delegated to a background thread pool. This eliminates the developer side of handling multithreaded scenarios as only the main thread executes the callbacks from the event loop one by one. An example would be `Node.js`.
  - Actor model - actors communicate with each other by sending asynchronous messages which are stored in other actors' mailboxes until they are processed. Each actor is self contained and there is no way for actors to introduce accidental race conditions, because they dont share memory. An example would be `Elixir`, `Erlang`,  `Akka`.
  - CSP (communicating sequential processes) - each actor can send messages to other actors `synchronously`, if an actor is unavailable to receive a message, both the sending and the receiving actors are scheduled aside meanwhile other actors are executed

### No Magic Here
It is important to understand that all the concurrency models are implemented in user space, as there are no concepts like actors and event loops in the OS. Usually, these implementations will be injected in the run time of the language or compiled together with your application. It always comes down to the OS scheduler and the OS threads.


## Enter CSP

## CSP Pros

## CSP Cons

## Examples


## Conculsion
