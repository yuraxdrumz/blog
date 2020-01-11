---
title: The Linux Scheduler For Developers
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
  - Linux
category: Operating Systems
---

![](./concurrency.jpg)
The linux scheduler is a complex piece of software and it is responsible for making sure cores are not idle if there are threads that need work to be done. The Linux scheduler went through several rewrites throughout the years, but I want to focus on the CFS in this blog post. The CFS, which stands for "Completely Fair Scheduler", is the new "desktop" process scheduler implemented by Ingo Molnar and merged in Linux 2.6.23 and is still in use today.


Lately, I took a deep dive into how the OS handles the scheduling of threads (tasks). I found out the scheduler is not trivial at all and has a lot of interesting features we, as developers, can advantage of or at least keep in the back of our minds.

## What is a scheduler
The scheduler is the component of the kernel that selects which process to run next.
There are two types of schedulers:

  - `preemptive` - the scheduler decides when a process is to cease running and a new process is to resume running (context switching)
  - `cooperative` - a process does not stop running until it voluntary decides to do so


## Why CFS ?
The CFS came to life following a well studied classic scheduling algorithm called `weighted fair queuing`, originally invented for packet networks.
The CFS uses a red-black tree data strcuture, which we will cover in the next section. 
The CFS has a scheduling complexity of O(log N), where N is the number of tasks in the runqueue. Choosing a task can be done in constant time, but reinserting a task after it has run requires O(log N) operations.

## Red-Black Tree
## Timeslices
## Priorities
## Context switching
A context switch can occur on several conditions:
  - When a processes timeslice reaches 0
  - Hardware interrupt
  - Transition between the user mode and kernel mode

## Conclusion