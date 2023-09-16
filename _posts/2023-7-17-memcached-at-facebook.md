---
layout: post
title: "How facebook scaled memcached"
author: Job Hernandez
tags: internals internet backend
---

*version 1.1*: rephrased a few parts and added my interpretation, 7/20/23

*version 1*: initial version, 7/17/23


### Introduction

The following is my reflection of the paper [Scaling Memcached at Facebook](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf). I am not an expert so please read the paper if you find my reflection interesting. In some parts of the following I included my interpretation of what I think is going on. So, again, please read the paper. It is a beautiful paper that will teach you a lot of things. I mainly wrote this to engage with the paper and enhance my understanding. After reading this experience paper I have learned that if I would like to work on similar stuff then I must know computer science fundamentals; for example, in the paper the authors talk about how they used UDP instead of TCP for get requests because UDP is connectionless and as a result each web server thread can communicate directly with memcached servers without establishing a connection thereby reducing latency. In my humble opinion this indicates that it pays to know backend engineering fundamentals such as communicating protocols – e.g., TCP, UDP. Without knowing the fundamentals, someone who is interested in working on distributed key-value stores will be significantly limited; moreover, reading about distributed systems is pretty interesting and expands your mind and arguably you can make better decisions as a backend programmer. 

I find this type of infrastructure work really fascinating. I had already listened to an InfoQ Instagram [talk](https://www.youtube.com/watch?v=hnpzNAPiC0E) about how Instagram engineers scaled the infrastructure of the respective website I think a lot of interesting problems arise from big websites at the scale of Facebook and Google.

As the paper says: big websites impose computational, network and I/O demands. This makes me think that  Steve Yegge is right – he said that if one wants to be an effective engineer at *Big Tech* companies then computer science fundamentals are crucial. I can see how understanding how the operating system manages the CPU and memory can give light as to how big websites operate. If one understand operating systems one will be have more ability to work on websites that are massive in scale. But I also understand that most mainstream work does not require CS fundamentals which is unfortunate; nevertheless, personally I want to understand the whole system and I have been in journey to accomplish this.

### Why did FB use memcached?

To begin with I would like to define a few terms based on the paper. A *frontend cluster* consists of a web server and multiple memcached servers. A *backend storage* cluster is the main database, I believe. While a *region* consists of multiple frontend clusters and a web server.

At Facebook, Memcached was used as a building block to build a distributed key-value store that handles billions of requests and stores trillions of items.

Anyways, in the paper the authors claimed that the design decisions that were factored in were the following. Given that users consume more than they create there are a lot of fetching of data which means that a cache may be beneficial.

The FB engineers used memcached to reduce the read load of the databases. That is, since there are so many user requests there was a lot of data being fetched from the database. As a result, if you then use a cache then you reduce the load because instead of the web server fetching from the database the web server instead checks in the cache thereby reducing the read load.

### How did the cache work?

For a given request, the web server will check the cache; if it is not in the cache the web server will look in the database and then the web server will populate the key-value in the cache. In contrast, if user requests consist of writes then the web server writes to the database as usual but to avoid inconsistency between the data in the database and the data in the cache, the web server deletes the corresponding data item on the cache; for instance, suppose a given user request $$ k_{1} $$ writes data $$ d_{1} $$ on the database. In turn $$ d_{1} $$ will be stored in the cache. Suppose further that a subsequent user request updates $$ d_{1} $$ with $$ d_{2} $$. If this happen then the data $$ d_{1} $$ on the cache is inconsistent with the new data $$ d_{2} $$. Remember, this involves key-value pairs and as a result the value of the given key needs to be consistent accross the cache and database. The Facebook engineers also used memcached as a generic cache -- e.g., the engineers stored precomputed ML results that could be used by other applications.

### In a cluster how did they reduce latency?

The quicker the memcached server responds to a given user request, the quicker the response time of a given user request. Interestingly, a single user request may result in hundreds of individual memcached requests. They placed hundreds of memcached servers in a cluster to reduce load on databases and other services. Cache items were distributed across the memcached servers. As a result, web servers need to talk with a lot of memcached servers to handle a user request. How did they reduce latency? First, they focused on the memcached client which runs on each server; the client is responsible for serialization, compression, request routing, error handling and request batching. They also used a graph, a directed acyclic graph to maximize the number of items that can be fetched concurrently. With respect to client-server communication, the engineers embedded the complexity of a system into a stateless client rather than in the memcached servers. 

The engineers also reduced latency by using UDP for handling get requests. UDP reduces latency in this case because UDP is connectionless and as a result each web server thread is able to directly communicate with memcached servers directly without establishing or maintaining a connection thereby reducing the overhead. 

Web servers also rely on a lot of parallelism and over subscription to achieve high throughput.

An interesting connection I made between TCP and my studies of computer systems is the following. TCP maintains a connection so this means that it consumes memory and as a result it is expensive to maintain a TCP connection between every web server thread and memcached server without some form of connection coalescing. How does coalescing work? By reducing the network, cpu, and memory resources needed by high throughput TCP connections.

### How did they handle replication in a given region?

A region consists of multiple frontend clusters and a storage cluster. Each frontend cluster consists of web server and memcached servers. Multiple frontend clusters share a backend storage cluster. If multiple frontend clusters, which consist of a server and hundreds of memcache servers, share the same backend storage then I wonder how multiple clusters coordinate with the same storage. The backend storage cluster is responsible for invalidating cached data to keep frontend clusters consistent. How does this system accomplish this? Invalidating daemons – each daemon inspects the SQL statements and extracts deletes and broadcasts these deletes to the memcached deployment (frontend clusters) in a given region. This is done so the deletes  from the database get invalidated  in memcached.

### Deployment accross multiple regions

To further scale the Facebook site, the engineers deployed in multiple regions -- i.e., different geographic areas. Some benefits of this include:

- Reduce latency, latency is reduced because servers that are closer to the end user can significantly reduce latency; and

- Mitigate effects of natural disasters; and

- Cheaper power and economic incentives.

FB deployed accross  multiple regions to gain the above benefits. A region consists of a storage backend cluster and several frontend clusters (web servers, memcached servers). One region is designated to hold the master databases and the other regions are designated the replicas. What is the primary challenge of this architecture? Maintaining consistency. Maintaining consistency in memcached and persistent storage is the primary challenge when scaling across multiple regions. How did they solve this challenge where the challenge is the this. Replicas may lag behind the master database and as result replicas could have stale data.

Interestingly, The way they invalidate deletes by using daemons avoids race conditions in which an invalidation arrives before the data has been replicated from the master region.


