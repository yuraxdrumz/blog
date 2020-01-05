---
title: The Linux Process Scheduler
tags:
  - Concurrency
  - Parallelism
  - Threads
  - Scheduler
  - Linux
category: Operating Systems
---

![](./concurrency.jpg)
The linux scheduler is a complex piece of software which allows our linux distros to maximize hardware utilization without starving our processes. In this post, I will introduce you to the types of schedulers, which type Linux uses and how it achieves its goal.

### Types
### The Cooperative Scheduler
### The Preemptive Scheduler
### Data Structure
### Timeslices
### Priorities
### Context switching
### Conclusion

The scheduler is the component of the kernel that selects which process to run next.
There are two types of schedulers:

  - `preemptive` - the scheduler decides when a process is to cease running and a new process is to resume running (context switching)
  - `cooperative` - a process does not stop running until it voluntary decides to do so

### Cooperative
Now, let's say we had a `cooperative` scheduler, 1 core available and 50 single threaded applications running. That would mean we have `1 thread * 50 applications = 50 threads` running. Our pc would have one app working and everything else stuck, which is neither efficient nor good user experience. That is why modern OS's use a `preemptive` scheduler.

### Preemptive
When the scheduler decides to switch context to another thread,
In Linux, each process receives a timeslice and a priority:
   - `timeslice` - the maximum time a process can run before being replaced by another process
   - `priority` - a number that represents the prioirty of the process running. Linux has a `nice` value (from -20 to 19) for standard priority and it has a `real-time` value (from 0 to 99) for higher priority, like I/O from the user

A context switch can occur on several conditions:
  - When a processes timeslice reaches 0
  - Hardware interrupt
  - Transition between the user mode and kernel mode
