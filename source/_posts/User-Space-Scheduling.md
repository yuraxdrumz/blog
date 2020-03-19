---
title: User Space Scheduling
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
  - Green Threads
  - Kernel Threads
  - Threading Models
category: Programming
date: 2020-03-07 16:26:21
thumbnail: images/multi.jpg
---


A scheduler is a complex piece of software that is responsible for making sure cores are not idle if there are Threads that need work to be done.
The fast switching of Threads, also called a context switch, is done by the scheduler and it gives us the illusion that all of our processes run in parallel.
We have schedulers both in the Kernel of our favorite OS and in the user space, where our programs live and run.
Some types of programming languages leverage one type of scheduling, while others leverage another.
In this blog post, I will explain what a scheduler is and compare how scheduling affects different programming languages.

Before we jump to the scheduling section, let's refresh our memory with what Threads are.

## Threads
A Thread is a set of instructions that waits to be called and there are two types of Threads:
  1. Kernel Threads (OS Threads)
  2. Green Threads (lightweight Threads)

### Kernel Threads
Every application you run creates a process and each process is given an initial Kernel Thread, which in turn can create more Kernel Threads.
Kernel Threads can run concurrently by taking a turn on a single core, or in parallel each running at the same time on different cores.
When a Thread receives running time on a core it executes its instructions until it is pulled of the core.

### Kernel Thread States
A Kernel Thread can have three states

  - `Waiting` - the Thread is stopped because it is waiting for a sync call (mutex), a system call or a disk.
  - `Runnable` - the Thread is ready to be run on a core so it can execute its instructions
  - `Executing` - the Thread is executing instructions on the core.


### Green Threads
These Threads only exist in our programs in the user space and are invisible to the Kernel. 
The handling and scheduling of these Threads occurs in user space as well, depending on the `Threading model`.
The states of user space threads depend on how the scheduler that administers them is implemented.

## Scheduler
The scheduler is a component that selects which Thread to run next and there are two types of schedulers:

### Preemptive
The scheduler decides when a Thread is to cease running and a new Thread is to resume running (context switching).
Preemptive scheduling has to solve getting all kinds of software from all kinds of places to efficiently share a CPU.
#### Benefits
There is a degree of fairness to all Threads.
#### Drawbacks
Because Threads get interrupted and switched, there is an overhead to store/restore the Thread's state each context switch, which is very expensive computation wise.

### Cooperative
A process does not stop running until it voluntarily decides to do so. 

#### Benefits
Works well for processes designed to work together.
The scheduler is much simpler to implement.
There is no context switching so we get better performance.
#### Drawbacks
If a program forgot to yield control back to the scheduler, other programs will not get a chance to run on that core.

All major OS's today use a preemptive type of scheduler.
Windows 3.1 and MacOS 9 had a cooperative scheduler.

## Threading Models
There are different Threading models that explain how our Green Threads are linked to Kernel threads.

### 1-1
When we call to create a Green Thread in our program, it invokes a system call to spawn an Kernel Thread.
This type of Threading model does not need its own scheduler in user space.
Examples would be: `Java (JVM)`, `C`, `Rust`

#### Benefits
No need for a user space scheduler as we reuse the preemptive OS scheduler.

#### Drawbacks
Creating a large number of threads consumes a lot of system resources.
The same drawbacks to context switching apply here to due using the same OS scheduler.

### N-1
We have multiple Green Threads and only one Kernel Thread.
This type of Threading model does need its own scheduler in user space.
The prime example would be `Node.js`.

#### Benefits
Due to running on a single Kernel Thread, there is no overhead thinking about race conditions and mutexes.

#### Drawbacks
Cannot leverage multi core processors

### M-N
With this Threading model we create multiple Green Threads that run on multiple Kernel Threads.
This type of Threading model does need its own scheduler in user space.
Examples would be: `RxJava`, `Akka`, `Go`

#### Benefits
Leveraging the best of both worlds

#### Drawbacks
You need a really good implementation of a user space scheduler to make this threading model utilize the most out of your system
Data races and sync issues can occur, so it is up to the developers to handle them.

## Kernel and User Space Scheduling
We said earlier that all major OS's today use a preemptive type of scheduler, but it does not mean that all user space schedulers are preemptive as well.

### Preemptive Kernel and Cooperative User Space
Take, for example `Node.js`, its event loop is actually a cooperative scheduler. 
All phases in the event loop are run only after the previous phase finished running, meaning it gave control back to the event loop.
Another example for cooperative scheduling is `Go`, however we dont have to explicitly yield control back to its scheduler, as it does that by itself on each function invocation.

Because we have the OS preemptive scheduler, we can leverage the cooperative scheduling in user space, thus achieving both performance and concurrency.

### Preemptive Kernel and Preemptive User Space
Preemptive scheduling is hard because we need to take into account context switches and synchronization primitives, however, there are a few languages like `Erlang`, `Elixir` and `Haskell` that use a preemptive scheduler in user space as well.
Due to their functional nature, the languages above do not share memory between processes, so they dont need synchronization.
The `Erlang` vm, for example, creates a preemptive scheduler per CPU core, allowing for maximum concurrency.
Even if you block a Thread in your application with synchronous code, the vm will still give other Green Threads time to run.

## Conclusion
We saw what Threads are and how the scheduler uses them to not let the system idle if there is work to be done. Last, we saw how each of the Threading models affects how programming languages / runtimes work and how they are used.