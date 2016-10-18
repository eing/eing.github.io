---
layout: post
title:  "What can you test in Software Design Phase? Part I"
date:   2016-08-03 18:43:59
author: Eing Ong
categories: Testing
tags: shift-left
---
It is an established best practice to "Test early and often" and shift left testing such as validating requirements at the beginning of software development. Since requirements testing is pertaining to the knowledge of the domain, my focus on this blog will be in the **design phase**. Testability is becoming widely adopted and build testability into the product is a best practice. One good example is [Airlock](https://code.facebook.com/posts/520580318041111/airlock-facebook-s-mobile-a-b-testing-framework/) which was created by Facebook to enable them to measure data and execute A/B testing.

I'd like to take this further to widen the scope to include **usability, payload, latency, availability, versioning, security, tooling and data**.

We will exercise the architecture design by assessing the risks in these areas, in additon to validating testability. The goals and outcome of this assessment are

* Vet architecture design on testability, performance, usability and security implications 
* Identify any potential design gaps that would surface in development phase
* Address all net promoter detractors that would surface in production
* Produce initial set of test artifacts (test plan & test cases) as input to development phase

My goal of this blog is to identify the key categories and help guide you with the questions that you can ask and what you should look out for when you are in software design phase. These questions may be uncomfortable for you when asking at first but you'll develop the confidence to approach these topics with practice over time.

<h2>Usability</h2>

Usability is the driving force for making complicated things seem easy. By using the data below, it will help us test for complexity and walk in our customers shoes.

Questions you might want to ask are

* What is the most common change that users will do? How do we verify our assumptions?
* What is the least used/unused feature?
* How can our users get help if confused/giving up? How do we detect their confusion?
* Is accessibility a requirement?

| Workflow | # pages | # fields | # clicks | Any additional steps of friction | 
| -------- | ------- | -------- | -------- | -------------------------------- |
| E.g. First Time Use (FTU) | | | | |


<h2>Payload</h2>

Payload transmitted with each user request has potential adverse impacts to entire ecosystem performance. Hence, minimizing payload is an important website performance best practice. Optimizing content efficiency is a top best practice in [websites performance in Google](https://developers.google.com/web/fundamentals/performance/). This is further emphasized by [GlobalDots](http://www.globaldots.com/googles-web-performance-best-practices-4-minimize-payload-size/). If there is additional payload or new payload in the design phase, ask these questions

* What is the size of the smallest possible payload and the largest possible payload?
* What is the average payload that most of our customers will incur?

| Payload size | Size in bytes | Total under normal load | Peak load | Projected future load | 
| ------------ | ------------- | ----------------------- | --------- | ----------------------|
| Small  | | | | |
| Medium | | | | |
| Large  | | | | |

<h2>Latency</h2>

Page loading time is a major contributing factor to [page abandonment](https://blog.kissmetrics.com/loading-time/). We need to ensure we meet the users’ perceived performance expectations and when to kick in fallback implementations to meet the SLA. Behind the scene, cache hits/misses, fallbacks, marshaling, remarshaling of payload as well as extracting and processing of the payload are all part of the response time. User perceived latency is when the data that user wanted is displayed, while other data can still be loading in the background . Tools such as [PageSpeed](https://developers.google.com/speed/pagespeed/) can help you identify areas to improve upon. The table below outlines key considerations and the data metrics that test our design if it will meet the response time SLA. So what are the latency concerns we need to ask and consider during the design phase?

* How many server hops and regions hops does it take for the new workflow?
* In peak load, what would the response time look like? What levers do we have for optimizations?
* What is the growth projection on the number of users to be supported?

| Workflow | # regions hops (e.g. USA-EU-USA) | Latency for regional hops | # server hops in 1 region | Latency for server hops | Total latency estimate |
| -------- | -------------------------------- | ------------------------- | ------------------------- | ------------------------| ---------------------- |
| Most common workflow  | | | | |
| Most complex workflow | | | | |

<h2>Availability</h2>

Understanding our dependencies before a catastrophic failure will help us understand how resilient our new system will be in such an event and how we can build in resiliency during design phase. These are the questions I'd ask -

* What happens if a dependency becomes unavailable? Can the customer still proceed? 
* Can we test with each dependency one at a time and then start combining multiple failures?
* What fallback implementations make sense?
* What are the synchronous versus asynchronous calls made in these scenarios? 

| Dependencies | Is redundancy available? <br> Where and what zones? <br> What is the repair time SLA? | Is fallback available | Is call synchronous? |
| ------------ | ------------------------------------------------------------------------------------- | --------------------- | -------------------- |
|  | | | | |

Availability requires both software and hardware solutions. 
For software solutions, one example is using Netflix Hystrix design pattern and testing with Simian army. On infrastructure end, my recommendation is to read [AWS : Architecting in the Cloud](https://d0.awsstatic.com/whitepapers/AWS_Cloud_Best_Practices.pdf) and test your infrastructure against the recommeded best practices.

<h2>Versioning</h2>

What are the versioning concerns we need to ask and consider during the design phase?

* Will there be alpha/beta versions? Can we A/B test with small set of users?
* How do we support API versioning?
* What is our API policy? Is backwards compatibility supported? What is our deprecation policy (e.g. announcements, time frames)? 
* How do we roll back and roll forward?

The table below helps to organize and use data to quantify answers to the questions listed above.

| Production versions | Is backwards compatible? <br> Are there changes in the database schema? | Are toggle switches supported for features? | Is rollback and roll forward supported? Are there changes in database schema? |
| ------------ | ------------------------------------------------------------------------------------- | --------------------- | -------------------- |
| Alpha/Beta | | | | |
| N + 1 | | | | |
| N (Rollback) | | | | |

This concludes the first part of this Software Design Testing blog. I'll elaborate on security, and tooling which are important areas as well. Here's [Part 2](/testing/2016/08/06/WhatCanYouTestAtDesignPhase-Part2/).
