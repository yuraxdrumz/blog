---
title: Event Storming in Domain Driven Design
date: 2019-07-14 21:22:40
category: Architecture
tags: 
  - DDD
  - Software Architecture
  - Event Storming
---
![](./misunderstanding.png)
<!-- One of the parts of Strategic Domain Driven Design is the ubiquitous language. It is the communication tool shared between the domain experts and the developers. Unlike regular languages where a word can have several meanings, the ubiquitous language should have exactly one for each word in a single bounded context. After getting acquinted with *Implementing Domain Driven Design* by Vaughn Vernon and the DDD "lingo", I started to pay close attention to the development process at work and at the same time, I was leading a project where a single misunderstood requirement changed the entire implementation of a design I did. How ,you ask? Well, the word **collect** was lost in translation. That moment, I realized, that if I asked everyone involved in the project to do a few sessions and establish some common language until we have all the initial requirements met, we would not have lost precious time. In this post, I will show an example requirement similar to what I had at work and how the ubiquitous language could have saved me and the company time and money. -->

event storming also results in more valuable insights as participants more readily engage in the process and offer their suggestions and expertise
Event storming is not data modeling. Instead, it results in a fully behavioral model that can be quickly implemented and validated
Perhaps the greatest value of event storming is in the conversations it generates.