---
layout: post
title: "How the Internet Works"
author: Job Hernandez
tags: internet internals backend
---

*version 1*: initial version, 7/31/23

## Introduction

I am on a journey to learn more about backend software engineering fundamentals. In his YouTube video [How to become a good backend engineer](https://www.youtube.com/watch?v=V3ZPPPKEipA&t) Hussein Nasser argues that a good backend engineer is someone who knows the fundamentals. Some fundamentals include communication protocols such as HTTP, and TCP; cache systems; message queues, web servers. In this article I touch upon the Internet and communication protocols such as TCP, IP, and HTTP and web servers. I have also written about cache systems which you can read [here](https://jobhdez.github.io/2023/07/17/memcached-at-facebook.html) and I have also written a little bit about message queues and web architecture which you can read [here](https://jobhdez.github.io/2023/06/15/web-system-architecture.html). As a disclaimer, be aware that I am not an expert. I write about these topics because I want to consolidate what I study. I love the Internet and the fact that anything works at all is fascinating. 

Here is quote by Alan Kay:

> The Internet was done so well that most people think of it as a natural resource like the Pacific Ocean, rather than something that was man-made. When was the last time a technology with a scale like that was so error-free? 


## What is the Internet?

Network applications are ubiquitous; everytime you browse the web, and send an email you are interacting with network applications.

The Internet can be seen as a network of networks; for example, your home Wifi connects multiple computers to form a Local Area Network (LAN)[^1]. The Internet can then be seen as a network of LANs whose hosts use routers and protocol software to communicate with other hosts.

The Internet can also be seen as a collection of hosts with the following properties[^2]:

- the given hosts correspond to 32-bit IP addresses

- these 32-bit IP addresses correspond to Internet domain names

- processes in different hosts can communicate over a connection

Every network application conforms to the client-server architecture. This architecture consists of one server process and multiple client processes. Internet clients and servers communicate by exchanging streams of bytes over a connection. A connection consists of a client socket and server socket. A socket is the endpoint of a connection.  Each socket is associated with an address and a 16 bit port.

### How does the Internet work?
Clients and servers on the Internet use IP addresses to communicate. Domain names, however, are a human readable representation of IP addresses. When you try to go to `https://google.com` a domain name server is involved to map the domain name `google.com` to an IP address. Subsequently, the source host and destination host will establish a connection over TCP. 

Interestingly, the kernel and networking are related. First, there is a network adapter/driver on each host which is connected on the I/O bus of a computer system. When the packets arrive at this network adapter the kernel creates a virtual memory for the processes involved. Subsequently, the virtual memory addresses get mapped to physical addresses and the CPU starts fetching instructions from memory; moreover, the socket interface is a set of system calls. In other words, the socket interface consists of functions that call the underlying TCP/IP implementation in the kernel. Recall that system calls are the interface of user mode and kernel mode. When a user mode process needs resources such as memory it initiates a system call and the kernel allocates the necessary memory. Well, it turns out that the given client’s socket port is assigned by the kernel when the client sends a connection request. In contrast, the given server’s port is already established, it is well known in advance.

#### The network stack
At the lowest level of the Internet stack there's Local Area Networks (LAN) which essentially is your Wifi network at home. Each host in a LAN is connected to other LANs through routers. What happens when one computer connects with another computer?
To see how computers connect with one another one has to be aware of the protocol stack. The protocol stack consists of the following structure[^3]:

 ```
—---------                      —-------------
| http   |                      | HTTP      |
—--------                        —------------
    |                               ^
    V                               |
—--------                      —-------------
| TCP   |                      |  TCP
—-------                       —---------------
   |                                 ^
   V                                 |
—-------                        —------------
| IP   |                        | IP       |
—------                         —------------
   |                                 ^
   V                                 |
—----------                     —------------
| hardware | —----------------> | hardware
—----------                    —------------
```

Given a connection between computers there are two protocol stacks involved. When one is using the browser the data travels from the Application protocol to the hardware. In other words, the packets would travel from the application layer – e.g., HTTP – to the TCP layer where a port will be attached and then these packets will continue onto the IP layers where the destination IP address and from here the packets move onto the LAN networks which can be created with Wifi or the Ethernet. At the receiving protocol stack a router sends the packets to the correct computer and the packets start getting processed from the IP protocol to the application protocol. 

##### Internet Protocol (IP)
The IP is essential to the Global Internet. The essence is a route table. IP routes the IP packet to the destination hosts[^3].
With respect to IP there are two types of routing: direct routing and indirect routing. Direct routing happens when two hosts on the same IP network communicate whereas in contrast indirect routing happens when two hosts on different IP networks connect. When IP mediates the communication between two hosts the header will consist of the source and destination IP addresses.

When an IP packet travels over the Internet it can go from one IP-router to another while its route table determines where to direct the IP packet.

##### Transmission Control Protocol (TCP)

 TCP is used when guarantee delivery is needed. In order to send a request over HTTP the client and server first needs to establish a connection over TCP. This connection over TCP can be seen as a virtual circuit.

In the [RFC 1180](https://datatracker.ietf.org/doc/html/rfc1180) is says:

> TCP provides a different service than UDP.  TCP offers a connection-oriented byte stream, instead of a connectionless datagram delivery service. TCP guarantees delivery, whereas UDP does not.

The connection or virtual circuits between the two TCP implementations on the respective hosts is full duplex. That is, data can flow in both directions.

##### Hypertext Transfer Protocol (HTTP)

HTTP is an application protocol that mediates the exchange of HTML documents between a web browser and a server[^4]. HTTP is stateless which means that there is no relationship between requests happening one after another; nevertheless, via sessions you can provide some state.




### What is the web?
The web is also a collection of clients and servers that communicate via a text based protocol called
HTTP. HTTP is how information gets exchanged. This information or content is represented by HTML. Technically, web servers and clients see content as a pair consisting of a sequence of bytes and a MIME type. Some MIME types include HTML, images and plain text.

To have something concrete I will talk about the Apache and nginx web servers and briefly dive into the differences in the following section.

#### Web Servers
Here, I will talk about two http servers, namely, Apache and nginx. Apache spawns a process for each new connection whereas nginx is event driven. If a http server is process based then it will not deal with concurrency well.

Here is a key point about why web servers that are thread or process based such as Apache do not handle concurrency well. Everytime the web server spawns a new process the cpu needs to create virtual memory (stack and heap) for each process. As a result, this leads to poor performance due to excessive context switching (switching the CPU among processes). In contrast, the foundation of nginx consists of an event based, asynchronous, single threaded, non-blocking architecture. Such architecture results in better performance, better use of server resources and enables the dynamic growth of web sites. As mentioned above Apache spawns a new process per connection but in contrast nginx consists of workers; each worker in nginx consists of a loop that handles many connections. This run loop is asynchronous so it does not block like the Apache server. Since nginx does not spawn a process or thread for each connection it does not consume a lot of memory given that it does not need to create virtual memory for a given process. As a result, nginx conserves CPU cycles because the CPU does not need to work on allocating stack and heap memory. In other words, one of the main reasons why nginx is more performant than Apache is because it does not spawn a process or thread for each connection - i.e., nginx is event driven. What nginx does instead is keep track of the network and storage, initializes new connections and adds them to the run loop and process asynchronously until completion. Nginx is algo great for parallelization since it spawns multiple workers to handle many connections and as a result you can have a separate worker for each core.

### Conclusion

We have discussed that the Internet can be seen as a network of LAN networks whose hosts, through their protocol implementations, communicate with other hosts. We also briefly discussed how data packets flow from the application HTTP protocol, to the TCP protocol, which in turn sends data to the IP protocol which in turn sends data to the network adapter through a LAN.

### References
[^1]: [Cloudflare - What is a LAN](https://www.cloudflare.com/learning/network-layer/what-is-a-lan/)

[^2]: [Computer Systems - A Programmer's Perspective](https://csapp.cs.cmu.edu/)

[^3]: [A TCP/IP tutorial](https://www.rfc-editor.org/rfc/pdfrfc/rfc1180.txt.pdf)

[^4]: [An Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
