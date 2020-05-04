---
title: "Review: Graduate Introduction to Operating Systems"
path: "/graduate-intro-to-os"
date: "2020-05-04"
excerpt: "As the first course in the Georgia Tech master’s program, I wanted to take something that would cover foundational computer science topics and would push me out of my comfort zone. I am glad I took Graduate Introduction to Operating Systems because I got what I wanted."
tags: ["OMSCS"]
---

As the first course in the Georgia Tech master’s program, I wanted to take something that would cover foundational computer science topics and would push me out of my comfort zone. I am glad I took [Graduate Introduction to Operating Systems](http://www.omscs.gatech.edu/cs-6200-introduction-operating-systems) because I got what I wanted.

The lectures covered typical OS topics such as concurrency, CPU scheduling, memory management, and I/O management, but the last lectures also included distributed systems topics such as consistency protocols and remote procedure calls. As part of the course, I also had to read 11 research papers. Reading the papers was well worth the effort because they go more in depth into lecture concepts. For example, Birrell and Nelson’s seminal paper [*Implementing Remote Procedure Calls*](https://s3.amazonaws.com/content.udacity-data.com/courses/ud923/references/ud923-birrell-nelson-paper.pdf) explains the reasons behind the design of RPC.

Every week, office hours were held by Prof. Ada Gavrilovska, who answered student questions and elaborated on the course material by explaining ideas that weren’t necessarily covered in the lectures. I enjoyed the office hours a lot because Ada is a great teacher and is good at explaining complex topics.

The part of the course which I liked the most was working on programming projects. They did not require writing kernel code. Rather, they required writing user-level programs. Nevertheless, the projects were designed in such a way that it wasn’t feasible to finish them without understanding important operating systems and distributed systems concepts.

The first project required creating a multithreaded client and server in C. The client and the server had to communicate using what could be considered a simplified version of an application layer protocol (e.g., HTTP). So, I had to delve deep into concurrency, thread synchronization, and socket programming.

The second graded project was my favorite: it required implementing multithreaded proxy and cache servers and their communication protocols, again, using C. This project had a requirement that the data transferred from the cache to proxy had to be done via shared memory. Commands sent from the proxy to the cache could be implemented using any IPC mechanism. For the design I came up with, sequenced-packet sockets (`SOCK_SEQPACKET`) looked like the most suitable IPC mechanism. I used semaphores for synchronizing among proxy and cache threads when transferring data via shared memory segments.

For the last project, the goal was to create a rudimentary distributed file system by leveraging RPC. However, this time, the project had to be implemented in C++. The concrete RPC implementation used in the project was [gRPC](https://grpc.io/), and since gRPC is developed by Google, [Protocol Buffers](https://developers.google.com/protocol-buffers/) were used as the [interface definition language](https://en.wikipedia.org/wiki/Interface_description_language). When I finally finished the project, it was fun watching how file changes were being synchronized among multiple clients.

As for the class experience, it was great to have helpful classmates and TAs. One of the memorable moments during the course was when I was confused about the difference between the terms “multiprogramming” and “multitasking”. After struggling to find a succinct explanation online, I realized that I am in a master’s program now and can ask instructors/classmates for help. After posting a question on Piazza, a student provided an answer that clarified everything!

All in all, the course was very enjoyable: the professor and TAs were engaged, and the projects weren’t cookie-cutter, requiring students to make design choices themselves.
