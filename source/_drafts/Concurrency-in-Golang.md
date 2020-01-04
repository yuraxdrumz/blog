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
Concurrency is when several actions are executed during overlapping time periods, but, do not confuse concurrency with parallelism. Concurrency allows our applications to run faster and can lead to parallelism, thanks to multicore processors, but it also opens the door to many pitfalls. That is why there are different types of concurrency models out in the wild, each trying to squeeze the maximum performance out of a system in its own way. In this post, I will explain what is concurrency, how it leads to parallelism and the concurrency model that Golang utilizes, which is called CSP (Communicating Sequential Processes) and show its benefits and drawbacks along with some examples. If you are on a concurrency spree, make sure to check out Node.js' concurrency model called the event loop [here](/2019/06/09/Node-JS-Event-Loop-0/).


## Concurrency vs Parallelism
I remember my father bringing home my first pc with a `Pentium 3` processor and a `Windows 95` OS on it. I immediately started playing with `Paint` and a bunch of other programs and I got the sense of illusion that my pc was fast as hell. You automatically think, parallel execution, right? Actually, `Pentium 3` was single core, meaning there could only be one computation happening at any slice of time, so how come I thought the pc did everything in parallel? The answer is concurrency. 

When your app requests resources from the OS, it receives one random core to run on (single thread). The initial thread can create more OS threads which will run on other cores, if they exist.

Now, let's say we have 1 core available and we have 50 single threaded applications running, it means we have `1 thread * 50 applications = 50 threads` running. If the OS were to give each thread its full time to finish, our pc would have one app working and everything else stuck, which is neither efficient nor good user experience. Instead, the OS has a complex piece of software component called a `scheduler`.

### The Scheduler
The scheduler's job is to give a slice of time for each thread to do some computations and when it decides to switch to another thread, it puts the currently executing thread aside and gives another thread some time to run. With this interrupting technique, all of our threads receive some time to run and our system as a whole feels more responsive.

The handling of multiple operations, each at its own slice of time is what is called `concurrency`. Now, add multiple cores to the equation. When you have several computations happening on several cores and the timing between the computations overlaps, the concurrent operations become parallel and we have what is called `parallelism`.

Take our example above and change 1 core to 4. All of our applications would work the same if they were single threaded, but the system as a whole would be faster. If our applications were multithreaded, they would have the potential to be even faster, but don't be fooled, with parallel work we have a new set of problems.

### Context Switching
The act of swapping threads on a core is called a context switch.
When the scheduler decides to switch context to another thread, it needs to clear all the registers of the core that was chosen, take the new threads latest instruction to run, populate all the registers with the new data and then start executing. All this work is spent switching context instead of doing computations. 

The context switch performance hit will depend on a lot of factors, but it is safe to say it will be several thousands cpu cycles on each context switch.
That is why IO intensive operations, like handling network calls, will benefit from multicore cpus, but CPU intensive operations, like number crunching, will suffer.

## What's wrong with multithreaded applications?
Multithreaded applications allowed us to take advantage of multicore processors and lured us thinking that parallelism is easy.
Before multithreading, developers were writing imperative code (one, that executes line by line) on a single thread and now they need to think about mutexes, semaphores, atomic data structures, cpu caches, deadlocks and race conditions.


## Enter CSP

## CSP Pros

## CSP Cons

## Examples


## Conculsion
