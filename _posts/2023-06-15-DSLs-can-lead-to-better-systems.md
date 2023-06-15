---
layout: post
title: "DSLs can lead to better systems"
author: Job Hernandez
---
*draft*

### Intro
Conjecture: DSLs can lead to better systems in the industry

This essay is a summary of the paper [A Domain Specific Language for Microservices](https://drive.google.com/file/d/1aYupExDuAbUheDX4aycrxZDrc6stTcig/view) and it also has some of my thoughts about this paper.

### A DSL for microservices 
According to the above paper Twitter’s consumer product consists of independent services communicating via Thrift RPC interfaces; an interesting thing I learned is that a typical user level request gets handled by multiple services; at the time of this paper Twitter had a microservices architecture but despite the autonomy a microservice architecture gave engineering teams it was too costly to maintain each service; according to the paper some of the benefits a service architecture gave included:

Each service has its own codebase so a team can develop each service independently;
Each service is independently deployed so there is no need of central coordination; and
Each team has an owning team so domain knowledge is dependent on the service.

There is a quote that reminded me of the book `Software Design for Flexibility` by Gerald Sussman. Here is the quote:

> Often application logic of a service is modest and does not justify this overhead; it is therefore pragmatic to build new functionality into an existing service rather than bring up a new one.

This quote is talking about how the costs that arise from a microservice architecture cannot be justified when a given service is modest.

All of this is the first hint as to why building systems that are additive is crucial. At least, this is the argument by Dr. Sussman. As I said this quote reminded me of chapter 1 in `Software Design for Flexibility` because the authors talk about how one needs to build systems that grow and adapt as requirements change; instead of heavily modifying code one should `add` to the system to increase functionality. The quote from the Twitter engineers was hinting that a better design was to be able to add services without the costs of a microservices architecture.

One thing I would like to point out that I found insightful is that when one adds more and more functionality to a microservice it becomes a monolith, obviating the benefits of a microservice architecture.

The solution that the Twitter engineers figured out is to build the Strato service system. Strato is a system that allows applications to be added – a form of additive software design. Engineering teams work on application logic instead of developing a service. 

The application logic is built using a DSL called StratoQL; a StratoQL expression provides implicit concurrency whereby the developer can concurrently call different services. Remember that in the beginning of this essay I pointed out that a typical request is handled by multiple services; well, this is still possible with the DSL.

### Lessons
Some lessons of this paper is that Dr. Felleisen, the Racket creator, and Dr. Gerald Sussman may be right about the importance of writing DSLs to solve problems. One argument in `Software Design for Flexibility` is that DSLs are a technique for organizing a system in a way where functionality can be added by adding to the system instead of modifying the system heavily. Or consider the lessons from `Structure and Interpretation of Computer Programs`. The main lesson in this book is that computer science is about controlling complexity; abstraction is key for controlling complexity. DSLs are the ultimate abstraction and what this paper showed is that DSLs can also reduce costs.

One thing I am wondering now is how DSLs can be applied to web development. It may be possible to write a DSL for web development where concurrency is implicit and a language where different api endpoints can be called concurrently.
