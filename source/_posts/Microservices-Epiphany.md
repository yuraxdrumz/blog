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
The majority of the posts I saw about microservices talked about the differences vs monoliths and how everyone is rushing to build microservices in this fast paced world we live in. Count me in as well. I tought I was building microservices, until, I read *Implementing Domain Driven Design* by Vaughn Vernon and by learning DDD for even just a bit changed my perspective on things. What I experienced, like the title suggests, was an epiphany, that, I was building microservices wrong all along. In fact, I was building smaller monoliths, separated by a subdomain. *Head Explodes!* In this short post, I will show a couple of symptoms that I found are a sign, you are building microservices the wrong way.

## Symptoms of a monolith
One of the things I noticed after practicing DDD for a while, was that all the services I recently wrote were small monoliths. I mention DDD due to the fact that I understood some concepts, like bounded contexts, context maps and etc that helped me see the truth about my past design choices. The **first symptom** is that your services do not communicate. If you have a microservices architecture, you should have a mesh of interconnected components. In fact, if you had the "joy" between deciding where to join responses, either at the gateway level or at the service level, risking over coupling, its a good thing to some extent, it means you have services with clear boundaries and you reuse past implementations.

## Continuing the streak
The **second symptom** I noticed and the first thing that should have startled me a long time ago was a *one rules them all database*. Not only is it a single point of failure, but even worse, is the fact that, having everything stored in one place lures you in favor of adding *just* another feature in this "microservice" and soon it becomes a monolith bombed with 30 features and you excuse yourself with *Why should I duplicate the data, I have everything right here* or the famous *its a legacy database, lets reuse it*. Trust me, it is a no-op.

## How Domain Driven Design helped
Understanding parts like where to set clear boundaries between services, defining a shared langauge with the domain experts, and seeking reuse together with careful design, led me to the understanding that things that might seem unrelated at first like DDD and microservices, have in fact so much in common. After all, there are hundreds, if not thousands of tutorials on how to build microservices out in the wild. So how come my team and I repeated the few things everyone warned not to do. Maybe it is the "developers ego", or perhaps the notion of "it works so why bother changing" ? Either way, learning something that seemed unrelated and applying it to something existing opened my eyes and I will definitely try and implement things differently from now on.