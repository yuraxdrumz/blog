---
title: Microservices Epiphany
date: 2019-07-16 00:46:10
category: Architecture
tags: 
  - Microservices
  - DDD
  - Maybe a rant
---
![](./what.jpg)
The majority of the posts I see about microservices talk about the differences vs monoliths and how everyone is rushing to build microservices in this fast paced world we live in. Count me in as well. Recently, I read *Implementing Domain Driven Design* by Vaughn Vernon, which seemed unrelated to microservices at first but soon changed my perspective on things. What I experienced, like the title suggests, was an epiphany that I was building microservices wrong all along. In fact, I was building smaller monoliths, separated by a url subdomain. *Head Explodes!* In this short post, I will show a couple of symptoms that I found are a sign your microservices architecture might suffer in the long run.

## Symptoms of a monolith
One of the things I noticed after practicing DDD for a while, was that all the services I recently wrote were small monoliths. I mention DDD due to the fact that I understood some concepts, like bounded contexts, context maps and etc that helped me see question my past design choices. The **first symptom** is that your services do not communicate. If you have a microservices architecture, you should have a mesh of interconnected components, either RESTful or evented. In fact, if you had the "joy" between deciding where to join responses, either at the gateway level or at the service level, risking over coupling, it means you have services with some boundaries and you reuse past implementations. Otherwise, you simply have a bunch of services under the same domain.

## Continuing the streak
The **second symptom** I noticed and the first thing that should have startled me a long time ago was a *one rules them all database*. My team and I had a legacy MongoDB Replica which was the only database we had. Not only is it a single point of failure, but even worse, is the fact that, having everything stored in one place lures you in favor of adding *just* another feature in this and that "microservices" and soon you have a bunch of monoliths bombed with 30 features each with non related behavior whatsoever and you excuse yourself with *Why should I duplicate the data, I have everything right here*.

## How Domain Driven Design helped
Understanding parts like where to set clear boundaries between services, defining a shared langauge with the domain experts, and seeking reuse together with careful design, led me to the understanding that things that might seem unrelated at first like DDD and microservices, have in fact so much in common. After all, there are hundreds, if not thousands of tutorials on how to build microservices out in the wild. So how come my team and I repeated the few things everyone warned us not to do. Maybe it was the laziness to refactor or the so-called "developers ego" or perhaps the notion of "it works so why bother changing?" Either way, learning something that seemed unrelated and applying it to something existing opened my eyes.