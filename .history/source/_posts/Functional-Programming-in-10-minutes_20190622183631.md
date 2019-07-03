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

Before I jump to the characteristics, I want to point out that the terms highlighted will be explained later on.

## Characteristics of Functional Programming
  1. Immutability - Once a variable is created, it cannot be mutated.
  2. Declerative - Describs **what** the program must accomplish in terms of the problem domain, rather than describe **how** to accomplish it as a sequence of the programming language primitives.
  3. Higher-order functions and functions as first class citizens - Functions can take other functions as arguments or return them as results. With the help of `closures`, functions allow `currying` and `partial application`.
  4. Pure functions - functions that have `no side effects`.
  5. Recursion - For loops inherently mutate state, remember incrementing i ? As FP does not allow mutating state everything is done with recursion. Do not worry about stackoverflow errors, as FP has optimized recursions called `tail call recursions`.

We saw some characteristics, now, lets see what are closures, currying, partial application, no side effects and tail call recursions.

## Closures
Think of a function's variables as a bag. When the function is returned (removed from the stack and its frame is destroyed), its bag remains. We previously said that functions can return other functions and closures come into play when an outer function is destroyed and the inner function wants to access previously passed arguments (the outer functions bag).
```javascript
  function firstName(firstName){
    return function lastName(lastName){
      return firstName + lastName
    }
  }
  const firstNameWill = firstName("Will")
  const lastNameSmith = firstNameWill("Smith")
  const lastNameIAM = firstNameWill(".I.AM")
```
You see how we stored `"Will"` in the "bag" of the firstName function. Even after it returned, we still had access to its variables. It allowed us to reuse the name Will and call it with different last names.

## Currying
In mathematics and computer science, currying is the technique of translating the evaluation of a function that takes multiple arguments into evaluating a sequence of functions, each with a single argument.

Lets see an example without currying
```javascript
  function entireName(firstName, middleName, lastName){
    return firstName + middleName + lastName
  }
  console.log(entireName("Will"))
  // prints "Will undefined undefined"
```
With function currying, if we pass 1 argument, we will recieve a function back and only when all arguments are passed, our function will execute.
We can leverage this the same way we saw with currying, by applying only a few arguments and reusing the closures returned.
```javascript
  function entireName(firstName, middleName, lastName){
    return firstName + middleName + lastName
  }
  const withNameWill = entireName("Will")
  const withMiddleNameI = withNameWill("I")
  const withLastNameAM = withMiddleNameI("AM)
```

## Partial Application