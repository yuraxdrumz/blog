---
title: Concurrency vs Parallelism
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
category: Programming
---

![](./concurrency.jpg)
Concurrency allows our systems to handle multiple tasks by quickly switching between them in overlapping timeframes. When you add multicore processors to the mix, concurrency can lead to parallelism, which improves the performance of our systems and applications. In this post, I will explain what is concurrency and how it leads to parallelism. If you are on a concurrency spree, make sure to check out Node.js' concurrency model called the event loop [here](/2019/06/09/Node-JS-Event-Loop-0/).


## Prerequisite
When your app requests resources from the OS, it receives one random core to run on (single thread). The initial thread can create more OS threads which will run on other cores, if they exist.

Now, let's say we have 1 core available and we have 50 single threaded applications running, it means we have `1 thread * 50 applications = 50 threads` running. If the OS were to give each thread time to run until it terminates, our pc would have one app working and everything else stuck, which is neither efficient nor good user experience. Instead, the OS has a complex piece of software component called a `scheduler`.

## The Scheduler
The scheduler's job is to give a slice of time for each thread and when it decides to switch to another thread, it interrupts the currently executing thread and puts it in a queue. With this interrupting technique, which is called `context switching`, all of our threads receive some time to run and our system as a whole feels more responsive.

## Context Switching
When the scheduler decides to switch context to another thread, it needs to clear all the registers of the core that was chosen, take the new threads latest instruction to run, populate all the registers with the new data and then start executing. All this work is spent switching context instead of doing computations. 

The context switch performance hit will depend on a lot of factors, but it is safe to say it will be several thousands cpu cycles on each context switch.
That is why IO intensive operations, like handling network calls, will benefit from multicore cpus, but CPU intensive operations, like number crunching, will suffer.


## Concurrency Leads to Parallelism
The handling of multiple operations, each at its own slice of time is what is called `concurrency`. Now, add multiple cores to the equation. When you have several computations happening on several cores and the timing between the computations overlaps, the concurrent operations become parallel and we have what is called `parallelism`.

Take our example above and change 1 core to 4. All of our applications would work the same if they were single threaded, but the system as a whole would be faster. If our applications were multithreaded, they would have the potential to be even faster.

