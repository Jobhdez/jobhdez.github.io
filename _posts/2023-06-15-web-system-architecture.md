---
layout: post
title: Web System Architecture
author: Job Hernandez
---

*draft*

### Intro
One of the most interesting parts about web development is web system architecture. I do not know why but when I hear people talking about a given architecture I get excited. My first exposure to architecture was a [talk](https://www.youtube.com/watch?v=hnpzNAPiC0E&t) by an Instagram engineer in which she spoke about how they scaled the infrastructure at Instagram. I found it interesting because she spoke about how to scale by using multicore programming and I liked how the components of the Instagram architecture interacted.

So, I naturally enjoyed my time reading `Scalable Web Architecture and Distributed Systems`, the first chapter in the book `The Architecture of Open Source Applications.

In this essay I will try to summarize this chapter and share some of my thoughts about how I would design a social media application.

According to the author of this chapter some key design decisions for building scalable web applications consist of:
Availability;
Performance;
Reliability; and
Scalability.

A system is said to be available if it is always up and running. If a system is unavailable even for one minute the given company may lose thousands or even millions of dollars. 

The availability of a system is achieved through replication. The purpose of replication is to enable the given system to continue operating despite failures; distributed systems hide partial failure by replicating servers for instance.

Replication is also implemented for reliability and performance; replicas are conducive to the reliability of a system because if one replica crashes other replicas can keep the system going; moreover, replicas are conducive to performance in two cases. Firstly, suppose you are scaling in size; that is, an increasing number of processes need to access data; by having two replicas you can serve more processes. Secondly, suppose you are scaling in a geographical area; by using replicas near the geographic location of your processes you decrease latency times. 

The performance of a web system is important because performance affects the usage and user satisfaction as well as search engine rankings which correlate with revenue and retention. Performance can be defined as fast responses and low latency.

Another key design principle is reliability; as noted above one way to achieve reliability is through replicas. Reliability means that given a request will always get the same data in response and if the data changes the request will get new data.

Finally, another key design principle is scalability; scalability has to do with increasing the capability of your web system to handle more load.

### System architecture

#### Services
Often it is important to decouple service into functions; suppose, there exists a service that retrieves and uploads images through the network. If only one server handles these two functions there may be problems because writes are not asynchronous while reads are asynchronous; so during an upload the connection will be maintained and the client will not be able to start other tasks – it will have to wait until the image gets written. So, in such cases it is beneficial to to decouple these two functions into their own services.

#### Sharding
Suppose you have a tremendously large dataset that cannot fit in one server. You can either scale vertically whereby you use additional CPU cores, replace CPU with a faster CPU, add a bigger storage or you can scale horizontally whereby you add additional nodes – e.g., add a new server.

With respect to horizontal scaling one common technique is partitioning or sharding; for example, a given single file server can get replaced by multiple file servers each containing a set of images. One problem with this type of distribution of data  is data locality – i.e.,the closer the data to the operation the better the performance. Another problem with the distribution of data is inconsistency. When services share a resource there may be race conditions.

#### How do you make web systems fast and scalable?
To make web systems fast and scalable you need to utilize caches, global caches, proxies, load balancers, and queues.
