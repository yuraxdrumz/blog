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
People often argue about language X being better/scalable/faster than language Y, but, after writing software for the past 6 years, I realized that the language does not matter as much as the  implementations that we write.
Lately, I have decided to find a new programming language as my daily driver to replace Node.js. One of the reasons, being, that most Javascript code I encountered was either poorly written or barely documented or tangled or a mix and match of the above. I tried to remediate the situation by using Typescript, but it occured to me, that maybe a context switch, to a more strict and functional language, will do good for me and my team. The least we can do is try, right?
In this post, I will highlight some of Javascript's pros and cons. Later on, I will set myself a list of must-have features I expect from a programming language, afterwards, I will explain what is `Elixir`, what are its pros and cons and why I have chosen it as my daily driver.

When we see a new and shiny language or framework, we often tend to look at the pros, but, sometimes, the cons may be a deal breaker. That is why, I believe it is just as important to take the cons into consideration while choosing a job/language/framework and basically almost everything in life.

## Javscript - The Pros
Writing in Node.js has its pros and cons. Among the pros, you can list low development times, the biggest community in the world and easier hiring due to the popularity of the language.

## Javascript - The Cons 
  1. Javascript has 1 execution thread, which means that **when** something goes wrong and you failed to handle it, the entire process will halt.
  2. Memory leaks will bite without large amounts of debugging and profiling.
  3. The syntax changes so fast, I have seen legacy codebases with callbacks, later wrapped in promises and then awaited using the new async await syntax.
  4. As easy as it is to start programming in Node.js, as hard it is to master it and use it the right way. The event loop, which I had covered extensively [in my previous post](/2019/06/09/Node-JS-Event-Loop-0/), shows how hard it is to fully understand it.
  5. The coupling to npm. I had several times where `npm install` failed in production because of some strange dependency not getting installed even though it was available in the registry. I even had published packages disappear into thin air.
  6. The hybrid approach - The JS community advocates that it is great that you can write the same thing in a lot of ways, both in the `Object Oriented` and `Functional` programming paradigms, but in reality, not having a unified approach to writing code leads to worse codebases, especially in large groups with tens of developers from different backgrounds.

## Does Javascript's pros outweigh its cons ?
In my opinion, no. After having to deal with memory leaks and data being mutated wherever and whenever, I decided to tilt towards no. Encouraging developers to write in a particular way is good, but having the language strictly enforce something seems like a more viable option.


## What are some of the things I expect from a programming language
  1. Functional programming paradigm - Writing in a functional style for the past 3 years with [Ramda.js](https://ramdajs.com/) has taught me a lot and I wanted a purely functional language. I believe currying, functions as first class citizens and data immutability helps with the struggles of debugging and is easier to reason about.
  2. Language syntax - I wanted a simple to understand syntax that has no drastic changes throughout its lifespan.
  3. Compiled language - If everything compiled, I can sleep better at night. I am not expecting runtime errors to be unheard of, but, if I can minimize them, why not?
  4. Good standard library - To minimize the need for 3rd party modules.
  5. Strong community - For **when** something goes wrong.
  6. Concurrency - I want a language that plays nice with concurrency, without worrying about a single thread struggling to keep up or handling a threadpool and the accompanied mutexes/locks/deadlocks.

After playing with a few languages, I stumbled across Elixir.

## What is Elixir ?
from the elixir [website](https://elixir-lang.org/)
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Elixir leverages the Erlang VM, known for running low-latency, distributed and fault-tolerant systems, while also being successfully used in web development and the embedded software domain.

After reading the initial documentation, playing with a sample project, seeing that there is less than 10! open issues in the language's [github](https://github.com/elixir-lang/elixir) and writing a couple of projects for the past few months, I came back to my list and looked if `Elixir` has ticked all the boxes.
  - [x] *Is it Functional?*
  - [x] *Is the syntax simple?* taken from Ruby, which is very easy to understand.
  - [x] *Is is compiled?* Yes, it is. It is still dynamic typed though, so Runtime type errors may occur.
  - [x] *Does it have good standard library?* It has a detailed and well documented standard library.
  - [X] *Is the community active?* Elixir has its own forums which are really active.
  - [X] *Is it concurrent?* Elixir is built on top of the Erlang vm, which was used in the 80's for telecom companies. The Erlang vm has a scheduler for every core and it creates small units of execution, called *processes*. Processes in Erlang are very light weight and talk to each other via messaging - [Actor Model](https://en.wikipedia.org/wiki/Actor_model).

## Does Elixir's Pros outweigh its cons ?
To better understand if Elixir is production grade and a good fit, lets briefly see its pros and then its cond
