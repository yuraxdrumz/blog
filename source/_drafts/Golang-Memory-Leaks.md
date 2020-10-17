---
title: Golang Memory Leaks
category: Programming
tags: 
  - Golang
  - Memory Leak
  - Green Threads
  - Investigation
  - Profiling
thumbnail: images/Golang.png
---
Recently, I had a memory leak in production at work. After checking Grafana, I saw that when clients use a specific service, its memory rises until it hits an out of memory exception. After a thorough investigation, I found out the source of the memory leak as well as the reason to why it happened. To diagnose the problem, I used Golang's profiling tool called `pprof`.
In this post, I will explain how I diagnosed a potential memory leak, show how I used `pprof` to investigate it and explain the pitfalls to avoid when writing highly concurrent Golang code.


### Preface
Our clients use our system regularly through a proxy service, to which, we provide access to. The said memory leak was in the proxy service, which led to restarts, disconnects and hiccups.

### The Memory Leak
After getting complaints from clients about hiccups and disconnects, I started digging for the problem.
first thing I did, was go into the company's Grafana and check the memory and cpu of the proxy service.
I looked at the service and compared metrics from 3 different scenarios:
- Service is after restart and idle
- Service is under load
- Service was under load and now idle

All the overlapping lines below are idle instances of the service, only the green line gets traffic.
After traffic stops hitting the single instance, we can see its memory level stays well above the others and it does'nt get freed.
![](./leak.png)


Before jumping on a profiling adventure (some may call it a nightmare), I made a small checklist of things I want to try:
- I was using Golang version 1.12.5, so I wanted to make sure that the potential leak is not coming from the runtime (even runtimes have issues).
- Try agressive garbage collection using [debug.SetGCPercent(10)](https://golang.org/pkg/runtime/debug/#SetGCPercent)
- Try Manual [debug.FreeOSMemory()](https://golang.org/pkg/runtime/debug/#FreeOSMemory). Runtimes are all about performance, after memory has been garbage collected, it is not freed back to the OS immediately for performance reasons.
- Try to reproduce the leak in a staging environment with a small load test to confirm my suspicions.

After completing my checklist and seeing the leak was still there, I started profiling the service with `pprof`.

### Golang's Profiling Tool (pprof)
From the [pproff github page](https://github.com/google/pprof)
> pprof is a tool for visualization and analysis of profiling data.
pprof reads a collection of profiling samples in profile.proto format and generates reports to visualize and help analyze the data. It can generate both text and graphical reports (through the use of the dot visualization package).

#### Profiling
A profiling sample is a collection of stack traces with extra information on them, like memory allocations.

There are several profiles you can make
- goroutine    - stack traces of all current goroutines
- heap         - a sampling of all heap allocations
- threadcreate - stack traces that led to the creation of new OS threads
- block        - stack traces that led to blocking on synchronization primitives
- mutex        - stack traces of holders of contended mutexes
- profile      - cpu profile

To start the profiling, I imported pprof and started an HTTP server for it.
Note, that if you already have an HTTP server running, the import statement is sufficient.
I had a TCP server running, so I added another goroutine to listen on a separate port for the pprof.

```golang
import _ "net/http/pprof"

go func() {
	log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

For memory leaks, we want to look at the heap profile. 
To capture a profile we simply run `curl http://localhost:6060/debug/pprof/heap > heap.out` for a certain amount of time.
The profile will give us X amount of time worth of allocations.
There are two types of allocations the profile will collect
- inuse - current allocations
- alloc - total allocations since the program started running, regardless of whether the memory was freed or not.

We can inspect the profile using `go tool` command - `go tool pprof heap.out`.


Now, because we have a memory leak, the memory will go up, meaning more allocations.
We can capture a profile while the server is idle, wait for traffic to hit it and watch the memory go up and afterwards capture another profile.





### The Fix


### Conclusion
