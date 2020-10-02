---
title: Golang Memory Leaks
category: Programming
tags: 
  - Golang
  - Memory Leak
  - Green Threads
  - Investigation
thumbnail: images/Golang.png
---
Recently, I had a potential memory leak in production at work. After checking Grafana, I saw that when clients use a specific service, its memory rises until the process hits an out of memory exception. After an investigation, I found out the source of the memory leak as well as the reason to why it happened. To diagnose the problem, I used Golang's profiling tool called `pprof`.
In this post, I will explain how I diagnosed a potential memory leak, show how I used `pprof` to investigate it and explain the pitfalls to avoid when writing highly concurrent Golang code.


### Preface
Our clients use our system regularly through a proxy service, to which, we provide access to. The said memory leak was in the proxy service, which led to restarts, disconnects and hiccups, which we all can agree, is annoying and a terrible user experience.

### The Memory Leak
After getting complaints from clients about hiccups and disconnects, I started digging for the problem.
first thing I did, was go into the company's Grafana and check the memory and cpu of the proxy service.
I looked at the service and compared metrics from 3 different scenarios:
- Service is after restart and idle
- Service is under load
- Service was under load and now idle

All the overlapping lines below are idle instances of the service, only the green line gets traffic.
After traffic stops hitting the single instance, we can see its memory level stays well above the others.
![](./leak.png)


Before jumping on a debugging adventure (some may call it a nightmare), I made a small checklist of things I want to try:
- I was using Golang version 1.12.5, so I wanted to make sure that the potential leak is not coming from the runtime (even runtimes have issues).
- Try agressive garbage collection using [```debug.SetGCPercent(10)```](https://golang.org/pkg/runtime/debug/#SetGCPercent)
- Try Manual [```debug.FreeOSMemory()```](https://golang.org/pkg/runtime/debug/#FreeOSMemory). Runtimes are all about performance, after memory has been garbage collected, it is not freed back to the OS immediately for performance reasons.
- Try to reproduce the leak in a staging environment with a small load test to confirm my suspicions.


After completing my checklist and seeing the potential leak was still there, I started debugging the service with `pprof`.

### Golang's Profiling Tool (Pprof)


### The Fix


### Conclusion
