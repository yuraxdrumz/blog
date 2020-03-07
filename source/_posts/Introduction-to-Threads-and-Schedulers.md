---
title: Introduction to Threads and Schedulers
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
  - Green Threads
  - Kernel Threads
  - Threading Models
category: Operating Systems
date: 2020-03-07 16:26:21
---


![](./concurrency.jpg)
A scheduler is a complex piece of software that is responsible for making sure cores are not idle if there are Threads that need work to be done.
The fast switching of Threads, also called a context switch, is what gives us the illusion that all of our processes run in parallel.
In this blog post I will explain what a thread is, what a scheduler is, what threading models exist and finally how programming languages are affected by scheduling.

## Threads

### OS Threads
An OS Thread is a set of instructions that waits to be called and it is administered by the kernel. Every application you run creates a Process and each Process is given an initial OS Thread, which in turn can create more OS Threads.
OS Threads can run concurrently by taking a turn on a single core, or in parallel each running at the same time on different cores.
When an OS Thread receives running time on a core it executes its instructions until it is pulled of the core.

### OS Thread States
A thread can have three states

  - `Waiting` - the thread is stopped because it is waiting for a sync call (mutex), a system call or a disk.
  - `Runnable` - the thread is ready to be run on a core so it can execute its instructions
  - `Executing` - the thread is executing instructions on the core.


### User Space Threads
These Threads only exist in our programs in the user space and are invisible to the kernel. 
These Threads are also called `green Threads` or `lightweight Threads`. The handling and scheduling of these Threads occurs in user space as well, depending on the `Threading model`.

## What is a scheduler
The scheduler is the component of the kernel that selects which thread to run next.
There are two types of schedulers:

  - `preemptive` - the scheduler decides when a Thread is to cease running and a new Thread is to resume running (context switching).
  An upside to this type of scheduling is there is a degree of fairness to all Threads.
  A downside to this type of scheduling is because Threads get interrupted and switched, there is an overhead to store/restore the Threads state on the core each context switch, which is very expensive computation wise.
  - `cooperative` - a process does not stop running until it voluntarily decides to do so. 
  An upside to this type of scheduling is the scheduler is much simpler to implement and use.
  There is no context switching so we get better performance.
  A downside to this type of scheduling is if a program forgot to yield control back to the scheduler, other programs will not get a chance to run on that core.
  This type of concurrency works well for processes designed to work together, a.k.a not a multi purpose OS with 3rd party programs.

All major OS's today use a preemptive type of scheduler.
Windows 3.1 and MacOS 9 had a cooperative scheduler.

## Threading Models

### 1-1
When we call to create a user space Thread in our program, it invokes a system call to spawn an OS Thread.
This type of Threading model does not need its own scheduler in user space.
Examples would be: `Java (JVM)`, `C`, `Rust`

#### Benefits
No need for a user space scheduler.

#### Drawbacks
Creating a large number of threads consumes a lot of system resources.
The same drawbacks to context switching apply here to due using the same OS scheduler.


### N-1
We have multiple user space Threads and only one OS Thread.
This type of Threading model does need its own scheduler in user space.
The prime example would be `Node.js`.

#### Benefits
Due to running on a single OS Thread, there is no overhead thinking about race conditions and mutexes.

#### Drawbacks
Cannot leverage multi core processors


### M-N
With this Threading model we create multiple green Threads that run on multiple OS Threads.
This type of Threading model does need its own scheduler in user space.
Examples would be: `RxJava`, `Akka`, `Go`

#### Benefits
Leveraging the best of both worlds

#### Drawbacks
You need a really good implementation of a user space scheduler to make this threading model utilize the most out of your system
Data races and sync issues can occur, so it is up to the developers to handle them.

## User Space Scheduling
We said earlier that all major OS's today use a `preemptive` type of scheduler, but it does not mean that all user space schedulers are `preemptive` as well.

### Preemptive + Cooperative
Take, for example `Node.js`, its event loop is actually a `cooperative` scheduler. 
All phases in the event loop are run only after the previous phase finished running, meaning it gave control back to the event loop.
Another example for `cooperative` scheduling is `Go`.
We dont have to explicitly yield control back to its scheduler, it does that by itself on each function invocation.

Because we have the OS `preemptive` scheduler that allows for concurrency and can lead to parallelism, we can leverage the `cooperative` scheduling in user space, thus achieving both performance and concurrency.

### Preemptive + Preemptive
Like we said earlier, `preemptive` scheduling is hard because we need to take into account context switches and synchronization primitives.
But there are a few languages like `Erlang`, `Elixir` and `Haskell` that use a `preemptive` scheduler in user space as well.
Due to their functional nature, the languages above do not share memory between `processes`, so they dont need synchronization.
The `Erlang` vm, for example, creates a `preemptive` scheduler per CPU core, allowing for maximum concurrency.
Even if you block a Thread in your application with synchronous code, the vm will still give other Threads time to run.

## Conclusion
We saw that Threads are sets of instructions and that the scheduler uses one of two types of scheduling techniques to run these Threads on our CPU cores, so that our systems can achieve maximum efficiency. Later on, we saw how the different Threading Models work and the differences between them. Finally, we saw that there are programming languages that leverage `cooperative` scheduling in user space, while there are others that leverage `preemptive` scheduling in user space.
