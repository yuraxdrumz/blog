---
title: Why Elixir looks promising
date: 2019-06-17 21:11:52
category: Programming
tags: 
  - Elixir
  - Actor Model
  - Functional
---
In this post, I will show why Javascript's cons got to me and why I decided to switch to Elixir after using Node.js for the past 3 years.
Writing in Node.js has its pros and cons. Among the pros, you can list low development times, the biggest community in the world and easier hiring due to the popularity of the language, but I believe it is just as important to take the cons into consideration while choosing a job/language/framework and basically almost everything in life.

# Javascript - The Cons 
  1. Javascript is poorly designed
  2. Memory leaks from legacy code will bite you in your behind without large amounts of debugging and profiling.
  3. The language changes so fast, I have seen legacy codebases with callbacks inside promises wrapped in async awaits
  4. As easy as it is to start programming in Node.js, as hard it is to learn it and use it the right way.
  5. The coupling to npm. I had at least 4 times where `npm install` failed in production to some strange dependency or published packages just disappeard for no apparant reason.
  6. The hybrid approach - The JS community advocates that it is great that you can write the same thing in a lot of ways, in reality, I saw the same thing being written across multiple modules in a bunch of different ways. Of course, you can the say the developers that wrote it did not know what they are doing, but restricting them in the first place would have forced them to open of the documentation and use it correctly.