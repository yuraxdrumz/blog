---
title: Functional Programming in 10 minutes
date: 2019-06-22 18:13:57
category: Paradigms
tags: 
  - Programming Paradigms
  - Functional
---
Like most people, when I first saw the ideas of Functional Programming, I found them very strange due to the fact that, I, like most us, got used to structural and object oriented programming paradigms. The structural takes away the `goto` definitions from our code and replaces it with `if/else/do/while`, which forces the code to execute in an order. The object oriented, encapsulates local variables and methods long after a function returns (what eventually became a constructor) and through the use of function pointers introduced polymorphism. In this post, I will introduce you to functional programming. I believe that FP is a powerful tool and by learning it we gain a different perspective on how we can write code. Learning FP is difficult, and like any difficult subject, learning it should be done in stages.

## What is Functional Programming
It is the direct result of Alonso Church, who invented Lambda Calculus in 1936. FP treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.

Before I jump to the characteristics, I want to point out that the terms highlighted will be explained later on. It is important to grasp the general idea first, later on we will explain the highlighted terminology and show some examples.

## Characteristics of Functional Programming
  1. Higher-order functions and functions as first class citizens - Functions can take other functions as arguments or return them as results. With the help of `closures`, functions allow `currying` and `partial application`.
  2. Pure functions - functions that have `no side effects`.
  3. Recursion - For loops inherently mutate state, remember incrementing i ? As FP does not allow mutating state everything is done with recursion. Do not worry about stackoverflow errors, as FP has optimized recursions called `tail call recursions`.

