---
title: Is it worth switching from Node.js to Elixir ?
date: 2019-06-17 21:11:52
category: Programming
tags: 
  - Elixir
  - Actor Model
  - Functional
  - Node.js
---
Writing in Node.js has its pros and cons. Among the pros, you can list low development times, the biggest community in the world and easier hiring due to the popularity of the language, but I believe it is just as important to take the cons into consideration while choosing a job/language/framework and basically almost everything in life.
After writing numerous projects in Node.js for the last 3 years, both in `Object Oriented` and `Functional` styles, I set out to find a new programming language for the web that would allow me to write highly available, fault tolerant programs.
First, I will explain what are some of Javascript's cons that I believe are worth mentioning, afterwards I will explain about `Elixir` and what are its pros and cons.

## Javascript - The Cons 
  1. Javascript has 1 execution thread, which means that **when** something goes wrong and you failed to catch it, the entire process will halt.
  2. Memory leaks will bite without large amounts of debugging and profiling.
  3. The language changes so fast, I have seen legacy codebases with callbacks, later wrapped in promises and then awaited using the new async await syntax.
  4. As easy as it is to start programming in Node.js, as hard it is to learn it and use it the right way. The event loop, which I had covered extensively [in my previous post](/2019/06/09/Node-JS-Event-Loop-0/), shows how hard it is to fully understand the event loop.
  5. The coupling to npm. I had several times where `npm install` failed in production because of some strange dependency not getting installed even though it was available in the registry. I even had published packages disappear into thin air.
  6. The hybrid approach - The JS community advocates that it is great that you can write the same thing in a lot of ways, both in the `Object Oriented` and `Functional` programming paradigms, but in reality, not having a unified approach to writing code leads to worse codebases and a mix and match, especially in large groups with tens of developers from different backgrounds.

## What is Elixir ?
from the elixir [website](https://elixir-lang.org/)
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Elixir leverages the Erlang VM, known for running low-latency, distributed and fault-tolerant systems, while also being successfully used in web development and the embedded software domain.