---
title: Switching from Node.js to Elixir
date: 2019-06-17 21:11:52
category: Programming
tags: 
  - Elixir
  - Actor Model
  - Functional
  - Node.js
---
After writing software for the past 6 years, I realized that the implementations that we write are far more important than the languages we choose, but, having a developer friendly, safe, strict language, can help you achieve the above mentioned implementations. That is why I set to search for a new programming language, that will replace my daily driver - `Node.js`. 
In a search for a new programming language, I asked myself two questions - *what are the must-have features I expect from a programming language?* and *do its pros outweigh its cons?*
In this post, I will weigh some of Javascript's pros and cons, afterwards, I will list the features, I believe, a programming language should have, afterwards, I will explain why I chose to learn `Elixir`. Last, I will weigh Elixir's pros and cons.

## Javscript - The Pros
  1. Low development times - it is very easy to get started with Javascript
  2. The biggest community in the world - thousands of questions on stackoverflow and npm - the biggest package registry in the world.
  3. A lot of battle-tested frameworks and projects online - due to the popularity, there are a lot of open source projects out there.
  4. Hiring is easier than other languages due to the popularity.
  5. Runs on both the browser and the server - great for front end developers that want or need to make the switch.

## Javascript - The Cons 
  1. Single threaded - If you have a long running sync operation, your entire application will halt.
  2. Memory leaks are hard to diagnose and will bite without large amounts of debugging and profiling.
  3. As easy as it is to start programming in Node.js, as hard it is to master it. The event loop, which I had covered extensively [in my previous post](/2019/06/09/Node-JS-Event-Loop-0/), is not a simple beast to handle.
  4. The coupling to npm. I had several times where `npm install` failed in production because of some strange dependency not getting installed even though it was available in the registry. I even had published packages disappear into thin air.
  5. The hybrid approach - The JS community advocates that it is great that you can write the same thing in a lot of ways, both in the `Object Oriented` and `Functional` programming paradigms, but in reality, not having a unified approach to writing code leads to worse codebases, especially in large groups with tens of developers from different backgrounds.

## Do Javascript's pros outweigh its cons ?
After having to deal with nasty memory leaks, tough debugging sessions and data being mutated wherever and whenever, I decided to tilt towards **no**. The single threaded nature is both good and bad, the syntax changes so fast, I have seen callbacks wrapped in promises and awaited on using the new async await syntax and debugging in production **can** be a nightmare in large projects.


## What are some of the things I expect from a programming language
  1. Functional programming paradigm - Writing in a functional style for the past 3 years with [Ramda.js](https://ramdajs.com/) has taught me a lot and I wanted a purely functional language. Switching to legacy projects written in an OO way always felt harder to reason about. I believe currying, functions as first class citizens and data immutability helps with the struggles of debugging.
  2. Language syntax - I wanted a simple to understand syntax that has no drastic changes throughout its lifespan.
  3. Compiled language - If everything compiled, I can sleep better at night. I am not expecting runtime errors to be unheard of, but, if I can minimize them, why not?
  4. Good standard library - To minimize the need for 3rd party modules.
  5. Strong community - For when I have a non trivial problem at hand.
  6. Concurrency - I want a language that can handle concurrency, without worrying about a single thread struggling to keep up or handling a threadpool and the accompanied mutexes/locks/deadlocks.

After playing with a few languages, I stumbled across Elixir.

## What is Elixir ?
from the elixir [website](https://elixir-lang.org/)
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Elixir leverages the Erlang VM, known for running low-latency, distributed and fault-tolerant systems, while also being successfully used in web development and the embedded software domain.

After reading the initial documentation, seeing that there is less than 10 open issues in the language's [github](https://github.com/elixir-lang/elixir) and writing a couple of projects for the past few months, I came back to my list and looked if `Elixir` has ticked all the boxes.
  - [x] *Is it Functional?*
  - [x] *Is the syntax simple?* inspired by `Ruby`, which is very easy to understand.
  - [x] *Is is compiled?* It is still dynamic typed though, so run time type errors will occur, but Elixir has guards and pattern matching for that.
  - [x] *Does it have a good standard library?* It has a detailed and well documented standard library.
  - [X] *Is the community active?* Elixir has its own forums, which are really active.
  - [X] *Is it concurrent?* All Elixir code runs inside lightweight threads of execution (called processes) that are isolated and exchange information via messages - [Actor Model](https://en.wikipedia.org/wiki/Actor_model).

## Elixir - The Pros
  1. Highly concurrent - thanks to the Actor Model
  2. Functional - so we gain data immutability
  3. Scalable - distrubution of work across multiple nodes and communication is baked in the Erlang VM. 
  4. Highly available - Due to their lightweight nature, it is not uncommon to have hundreds of thousands of processes running concurrently in the same machine. Isolation allows processes to be garbage collected independently, reducing system-wide pauses, and using all machine resources as efficiently as possible (vertical scaling).
  5. Pattern matching - Elixir has pattern matching and guard clauses.

## Elixir - The Cons
  1. Young - It will take time until Elixir matures and shapes to a full featured language.
  2. Functional programming paradigm is difficult to learn.
  3. Hiring will be a pain as there are not a lot of experts in this domain.
  4. Up until v1.9 it was very confusing on how to compile a release.
  5. Raw processing - If all you do is number crunching, maybe you are better of with a different language. Elixir puts an emphasis on being highly concurrent and fault tolerant, but there are better languages out there for raw processing power.

## Do Elixir's Pros outweigh its cons ?
Companies, like Pinterest and Discord use Elixir for their highly sensitive systems and are very happy with the results. The language has a great standard library, good community, great debugging tools, like remote host connect and real time process inspections. The Erlang VM utilizes all of the machines resources and provides a full featured and stable runtime. My answer for now, will be - **yes**.