---
title: Functional Programming in 10 minutes
date: 2019-06-22 18:13:57
category: Paradigms
tags: 
  - Programming Paradigms
  - Functional
---
Like most people, when I first saw the ideas of functional programming, I found them very strange due to the fact that, I, like most us, got used to structural and object oriented programming paradigms. The structural takes away the `goto` definitions from our code and replaces them with `if/else/do/while`, which forces the code to execute in an order. The object oriented, encapsulates local variables and methods long after a function returns (what eventually became a constructor) and through the use of function pointers introduced polymorphism. In this post, I will introduce you to functional programming. I believe that FP is a powerful tool and by learning it we gain a different perspective on how we can write code. Learning FP is difficult, and like any difficult subject, learning it should be done in stages.

## Functional Programming in a Nutshell
It is the direct result of Alonso Church, who invented Lambda Calculus in 1936. FP treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.

Before I jump to the characteristics, I want to point out that the terms `highlighted` will be explained later on. All code examples will be written in Javascript.

## The Characteristics of Functional Programming
  1. Immutability - Once a variable is created, it cannot be mutated.
  2. Declerative - Describes **what** the program must accomplish in terms of the problem domain, rather than describe **how** to accomplish it as a sequence of the programming language primitives.
  3. Higher-order functions and functions as first class citizens - Functions can take other functions as arguments or return them as results. With the help of `closures`, functions allow `currying` and `partial application`.
  4. Pure functions - functions that have no `side effects`.
  5. Recursion - For loops inherently mutate state, remember incrementing i ? As FP does not allow mutating state everything is done with recursion. Do not worry about stackoverflow errors, as FP has optimized recursions called `tail call recursions`.

We saw some characteristics, now, lets see what are closures, currying, partial application, side effects and tail call recursions.

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
  /* prints Will Smith */
  console.log(lastNameSmith) 
  /* prints Will.I.AM */
  console.log(lastNameIAM) 
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
```javascript
  function entireName(firstName, middleName, lastName){
    return firstName + middleName + lastName
  }
  /*
    the above in its curried form lookes like this:
  */
  function entireName(firstName){
    return function middleName(middleName){
      return function lastName(lastName){
        return firstName + middleName + lastName
      }
    }
  }

  const withNameWill = entireName("Will")
  const withMiddleNameI = withNameWill("I")
  const withLastNameAM = withMiddleNameI("AM")
  /*
  Note that purely functional languages curry by default, like `Haskell`, others have different libraries for automatic currying of functions
  for example with ramda.js we could wrap our entireName function in a curry function
  */
  const { curry } = require("ramda")
  function entireName(firstName, middleName, lastName){
    return firstName + middleName + lastName
  }
  /* Below function is automatically curried like the example above and it is a lot more readable */
  const curriedEntireName = curry(entireName)
```

## Partial Application
Partial application works similar to currying. The difference between them is that currying splits a function to functions that receive one argument, also called unary, while partial application allows passing multiple arguments. Lets see an example
```javascript
  function entireName(firstName, middleName, lastName){
    return firstName + middleName + lastName
  }
  /*
    Look how we passed "Will" and "I" together. With currying you have to pass a single argument at a time
    until all arguments are passed. With partial Application you can pass any number of arguments and it will either execute if all arguments were given, or return a function that expects the original number of arguments minus the ones passed.
  */
  const { partial } = require("ramda")
  const partialWithTwoArgs = partial(entireName, ["Will", "I"])
  const fullName = partial(partialWithTwoArgs, ["AM"])
  console.log(fullName)
  /*
    prints "Will I AM"
  */
```

## No Side Effects
In programming, a side effect is when a procedure changes a variable from outside its scope.
For example:
```javascript
function updateVariable(){
  dbConnection.set("side_effect", true)
}
```
We changed some state outside of our function - this is called a side effect.
Functional programming is against side effects, but without side effects we would not be able to write software. Instead we try to limit the amount of side effects, allowing only a portion of our code to carefully do them.

## Recursion
Recursion in computer science is a method of solving a problem where the solution depends on solutions to smaller instances of the same problem (as opposed to iteration). Basically, a function calls itself multiple times until some condition is met.
For example, lets see a fibonacci recursion, do not mind the big O notation:
```javascript
function fibonacci(num) {
  if (num <= 1) return 1;
  return fibonacci(num - 1) + fibonacci(num - 2);
}
```
This function will call itself repeatedly until the condition above is met and until it does that, the function will create multiple frames of itself, possibly leading to stackoverflow.
To address the recursion problem, tail-call optimization is used. Tail-call optimization is where you are able to avoid allocating a new stack frame for a function because the calling function will simply return the value that it gets from the called function. Usually, the compiler takes care of tail call optimization in function languages, I will not elaborate on when tail call optimization does not occur, but if you get into FP in more detail, make sure to read about it.




