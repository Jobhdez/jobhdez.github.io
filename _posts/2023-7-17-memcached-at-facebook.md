---
layout: post
title: "How facebook scaled memcached"
author: Job Hernandez
---

*version 1*: initial version
----

## Distributed key-value store at facebook

### Introduction

The following is my summary of the paper [Scaling Memcached at Facebook](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf). Memcached was used as a building block to build a distributed key-value store that handles billions of requests and stores trillions of items.

After reading this experience paper I have learned that if I would like to work on similar stuff then I must know computer science fundamentals; for example, in the paper the authors talk about how they used UDP instead of TCP for get requests because UDP is connectionless and as a result each web server thread can communicate directly with memcached servers without establishing a connection thereby reducing latency. In my humble opinion this indicates that it pays to know backend engineering fundamentals such as communicating protocols – e.g., TCP, UDP. Without knowing the fundamentals, someone who is interested in working on distributed key-value stores will be significantly limited; moreover, reading about distributed systems is pretty interesting and expands your mind and arguably you can make better decisions as a programmer. 

I find this type of infrastructure work really fascinating. I had already listened to an InfoQ talk about how instagram engineers scaled the infrastructure at instagram. I think a lot of interesting problems arise from big websites at the scale of facebook and Google.

As the paper says: big websites impose computational, network and I/O demands. This makes me think that  Steve Yegge is right – he said that to be an effective engineer at *Big Tech* companies computer science fundamentals are crucial. I can see how understanding how the operating system manages the CPU and memory can give light as to how big websites operate. But I also understand that most mainstream work does not require CS fundamentals.

### Summary

Anyways, in the paper the authors claimed that the design decisions that were factored in were the following. Given that users consume more than they create there are a lot of fetching of data where caching may provide benefits.

Why did FB use memcached? The FB engineers used memcached to lighten the read load of the databases. For a given request, the web server will check the cache; if it is not in the cache the web server will look in the database and then the web server will populate the key-value in the cache. On the other hand, for write requests, the web server writes to the database and then deletes any stale data on memcached instead of updating it. They also used memcached as a generic cache. The engineers stored precomputed ML results that could be used by other applications.

In a cluster how did they reduce latency? The quicker the memcached response the quicker the response time of a given user request. Interestingly, a single user request may result in hundreds of individual memcached requests. They placed hundreds of memcached servers in a cluster to reduce load on databases and other services. Cache items were distributed across the memcached servers. As a result, web servers need to talk with a lot of memcached servers to handle a user request. How did they reduce latency? First, they focused on the memcached client which runs on each server; the client is responsible for serialization, compression, request routing, error handling and request batching. They also used a graph, a directed acyclic graph to maximize the number of items that can be fetched concurrently. With respect to client-server communication, the engineers embedded the complexity of a system into a stateless client rather than in the memcached servers. 

The engineers also reduced latency by using UDP for handling get requests. UDP reduces latency in this case because UDP is connectionless and as a result each web server thread is able to directly communicate with memcached servers directly without establishing or maintaining a connection thereby reducing the overhead. 

Web servers rely on a lot of parallelism and over subscription to achieve high throughput.

An interesting connection I made between TCP and my studies of computer systems is the following. TCP maintains a connection so this means that it consumes memory and as a result it is expensive to maintain a TCP connection between every web server thread and memcached server without some form of connection coalescing. How does coalescing work? By reducing the network, cpu, and memory resources needed by high throughput TCP connections.

How did they handle replication in a given region? A region consists of multiple frontend clusters and a storage cluster. Each frontend cluster consists of web server and memcached servers. Multiple frontend clusters share a backend storage cluster. The backend storage cluster is responsible for invalidating cached data to keep frontend clusters consistent. How does this system accomplish this? Invalidating daemons – each daemon inspects the SQL statements and extracts deletes and broadcasts these deletes to the memcached deployment (frontend clusters) in a given region. This is done so the deletes  from the database get invalidated  in memcached.

How did they handle consistency across regions? Benefits of a broader geographic placement of data centers:

Reduce latency, latency is reduced because servers that are closer to the end user can significantly reduce latency

Mitigate effects of natural disasters

Cheaper power and economic incentives 

FB deployed multiple regions to gain benefits. A region consists of a storage backend cluster and several frontend clusters (web servers, memcached servers). One region is designated to hold the master databases and the other regions are designated the replicas. What is the primary challenge of this architecture? Maintaining consistency. Maintaining consistency in memcached and persistent storage is the primary challenge when scaling across multiple regions. How did they solve this challenge? The challenge is: replicas may lag behind the master database. The way they invalidate deletes using daemons avoids race conditions in which an invalidation arrives before the data has been replicated from the master region.


 

