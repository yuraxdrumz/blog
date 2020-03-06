---
title: Introduction to the OS Scheduler
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
category: Operating Systems
---

![](./concurrency.jpg)
A scheduler is a complex piece of software and it is responsible for making sure cores are not idle if there are threads that need work to be done.
In this blog post I will explain what a thread is, the types of threads, threading models and what a scheduler is.

## Threads

### OS Threads
An OS Thread is a set of instructions that waits to be called and it is administered by the kernel. Every application you run creates a Process and each Process is given an initial OS Thread, which in turn can create more OS Threads.
OS Threads can run concurrently by taking a turn on a single core, or in parallel each running at the same time on different cores.
When an OS Thread receives running time on a core it executes its instructions until it is pulled of the core.

### User Space Threads
These Threads only exist in our programs in the user space and are invisible to the kernel. These Threads are also called `green Threads` or `lightweight Threads`. The handling and scheduling of these Threads occurs in user space as well, depending on the `Threading model`.

## Threading Models

### 1-1
When we call to create a user space Thread in our program, it requests the OS to spawn a Thread.
This type of Threading model does not need its own scheduler in user space.
Examples would be: `Java (JVM)`, `C`, `Rust`

### N-1
We have multiple user space Threads and only one OS thread.
This type of Threading model does need its own scheduler in user space.
The prime example would be `Node.js`.

### M-N
With this Threading model we create multiple green Threads that run on multiple OS Threads.
This type of Threading model does need its own scheduler in user space.
Examples would be: `RxJava`, `Akka`, `Go`


## OS Thread States
A thread can have three states

  - `Waiting` - the thread is stopped because it is waiting for a sync call (mutex), a system call or a disk.
  - `Runnable` - the thread is ready to be run on a core so it can execute its instructions
  - `Executing` - the thread is executing instructions on the core.


## What is a scheduler
The scheduler is the component of the kernel that selects which thread to run next.
There are two types of schedulers:

  - `preemptive` - the scheduler decides when a thread is to cease running and a new thread is to resume running (context switching). The fast context switching is what gives us the illusion all of our processes run in parallel.
  - `cooperative` - a process does not stop running until it voluntarily decides to do so. This type of concurrency works well for processes designed to work together, a.k.a not a multi purpose OS with 3rd party programs

All major OS's today use a preemptive type of scheduler.
Windows 3.1 and MacOS 9 had a cooperative scheduler.

## Context switching
A context switch can occur on several conditions:
  - When a processes timeslice reaches 0
  - Hardware interrupt
  - Transition between the user mode and kernel mode

## Conclusion