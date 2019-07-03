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
When we set our sight on a new language we often tend to look at the pros, but, sometimes, the cons may be a deal breaker. That is why, I believe, it is just as important to take the cons into consideration while choosing a job/language/framework and basically almost everything in life.
People often argue about language X being better/scalable/faster than language Y, but, after writing software for the past 6 years, I realized that the language does not matter as much as the  implementations that we write.
Lately, I have decided to find a new programming language as my daily driver to replace `Node.js`. It occured to me, that maybe a context switch, to a more strict and functional language, will teach me a thing or two and maybe do good for me and my team. The least we can do is try.
In this post, I will highlight some of Javascript's pros and cons. Later on, I will set a list of must-have features I expect from a programming language, afterwards, I will explain why I chose `Elixir`. After listing Javascript's and Elixir's pros and cons, I will try to answer one question - **Do the pros of language X outweigh its cons**.

## Javscript - The Pros
  1. Low development times
  2. The biggest community in the world.
  3. A lot of battle-tested frameworks and projects online
  4. Hiring is easier than other languages due to the popularity.
  5. Runs on both the browser and the server.

## Javascript - The Cons 
  1. Single threaded - If you have a long running sync operation, you entire application will halt.
  2. Memory leaks will bite without large amounts of debugging and profiling.
  3. As easy as it is to start programming in Node.js, as hard it is to master it and use it the right way. The event loop, which I had covered extensively [in my previous post](/2019/06/09/Node-JS-Event-Loop-0/), shows how hard it is to fully understand it.
  4. The coupling to npm. I had several times where `npm install` failed in production because of some strange dependency not getting installed even though it was available in the registry. I even had published packages disappear into thin air.
  5. The hybrid approach - The JS community advocates that it is great that you can write the same thing in a lot of ways, both in the `Object Oriented` and `Functional` programming paradigms, but in reality, not having a unified approach to writing code leads to worse codebases, especially in large groups with tens of developers from different backgrounds.

## Does Javascript's pros outweigh its cons ?
After having to deal with nasty memory leaks, tough debugging sessions and data being mutated wherever and whenever, I decided to tilt towards **no**. Encouraging developers to write in a particular way is good, but having the language strictly enforce something seems like a more viable option.


## What are some of the things I expect from a programming language
  1. Functional programming paradigm - Writing in a functional style for the past 3 years with [Ramda.js](https://ramdajs.com/) has taught me a lot and I wanted a purely functional language. I believe currying, functions as first class citizens and data immutability helps with the struggles of debugging and is easier to reason about.
  2. Language syntax - I wanted a simple to understand syntax that has no drastic changes throughout its lifespan.
  3. Compiled language - If everything compiled, I can sleep better at night. I am not expecting runtime errors to be unheard of, but, if I can minimize them, why not?
  4. Good standard library - To minimize the need for 3rd party modules.
  5. Strong community - For when I have a non trivial problem at hand.
  6. Concurrency - I want a language that plays nice with concurrency, without worrying about a single thread struggling to keep up or handling a threadpool and the accompanied mutexes/locks/deadlocks.

After playing with a few languages, I stumbled across Elixir.

## What is Elixir ?
from the elixir [website](https://elixir-lang.org/)
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Elixir leverages the Erlang VM, known for running low-latency, distributed and fault-tolerant systems, while also being successfully used in web development and the embedded software domain.

After reading the initial documentation, seeing that there is less than 10 open issues in the language's [github](https://github.com/elixir-lang/elixir) and writing a couple of projects for the past few months, I came back to my list and looked if `Elixir` has ticked all the boxes.
  - [x] *Is it Functional?*
  - [x] *Is the syntax simple?* taken from Ruby, which is very easy to understand.
  - [x] *Is is compiled?* Yes, it is. It is still dynamic typed though, so run time type errors will occur, but Elixir has guards and pattern matching for that.
  - [x] *Does it have a good standard library?* It has a detailed and well documented standard library.
  - [X] *Is the community active?* Elixir has its own forums, which are really active.
  - [X] *Is it concurrent?* All Elixir code runs inside lightweight threads of execution (called processes) that are isolated and exchange information via messages - [Actor Model](https://en.wikipedia.org/wiki/Actor_model).

## Elixir - The Pros
  1. Highly concurrent - thanks to the Actor Model
  2. Functional - so we gain data immutability
  3. Scalable - distrubution of work across multiple nodes and communication is baked in the Erlang VM. 
  4. Highly available - Due to their lightweight nature, it is not uncommon to have hundreds of thousands of processes running concurrently in the same machine. Isolation allows processes to be garbage collected independently, reducing system-wide pauses, and using all machine resources as efficiently as possible (vertical scaling).
  5. Fault Tolerance - Elixir provides supervisors, which describe how to restart parts of your system when things go awry, going back to a known initial state that is guaranteed to work

## Elixir - The cons
  1. Young - the language is still pretty young and it will take time until it matures and shapes to a full featured language.
  2. Functional programming paradigm is difficult to learn.
  3. Hiring will be a pain as there are not a lot of experts in this domain.
  4. Up until v1.9 it was very confusing on how to compile a release in an Elixir app.
  5. Raw processing - If all you do is number crunching, maybe you are better of with a different language.

## Does Elixir's Pros outweigh its cons ?
Companies, like Pinterest and Discord use Elixir for their highly sensitive systems and are very happy with the results, which is great, because the language is getting battle tested.
Great debugging tools, like remote connect and real time process inspections are a big selling point, as well as not having a single thread, for when something goes wrong. So, my answer for now, will be - **yes**.