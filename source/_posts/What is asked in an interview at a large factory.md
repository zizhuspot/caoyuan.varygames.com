---
title: What is asked in an interview at a large factory
date: 2023-11-06 00:04:00
categories: 
  - Technology
tags: 
  - Backend
  - Interview
  - Backend
  - Go
  - development
  - companies
  - network
  - importance
description: Interviewing is a tense and interesting process; tense because of the importance, interesting because of the unpredictable results. Sometimes, with a glance or a word, the interviewer will decide whether you will stay or go. So, is it like that in the interviews of big Internet companies?
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/976db374271649658327403113ca8400%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---



Table of Contents

1. Background
2. Self Introduction & Past Experience
3. Go Language Knowledge Points
4. Operating System & Computer Composition Principles
5. Middleware Principles
6. Networking & Distributed
7. Algorithms & Data Structures
8. Summary

## 1. Background

### 1.1 Personal Information

Bachelor's degree, 4 years of back-end development, 2.5 years of Go language practice. I like singing, dancing, Rap, basketball, etc 🐶.

Main programming languages are Go and Java, working with technology stacks such as MySQL, Oracle, Redis, RabbitMQ, Kafka, SpringBoot, Design Patterns, Networking, Microservices, Distributed and so on.

Interview purpose: Looking for a few experienced interviewers to see their level and also to understand the market situation.

### 1.2 Interview position

This interview is for a backend development engineer (Golang), and the position information is as follows:

Position temptation:

1. Internet industry, giving higher than peer treatment, according to performance and other circumstances against the standard 16 salary. 2;
2. core positions, development space is huge;

Position responsibilities:

1. participate in the company's Internet product design and development, architecture design;
2. Cooperate with the company's strategy, complete the company and department OKR tasks;
3. Follow up the industry cutting-edge technology, continuous technical construction;

Requirements: 1.

1. Bachelor degree or above, with 2 years or above experience in Golang language programming and development;
2. familiar with the common commands of Linux, and have the ability to write basic shell script. 3. proficient in Go language;
3. proficient in Go language, familiar with network programming interfaces, familiar with TCP/IP, UDP protocols, and have a deep understanding of network communication programming models. 4. proficient in common design patterns;
4. proficiency in common design patterns, proficiency in object-oriented programming methodology, and certain design capabilities;
5. proficient in using MySQL, good database design and rich experience in optimization;
6. familiarity with NoSQL technology (Redis, Memcached, etc.);
7. good technical architecture skills and project experience, good at finding technical problems and proposing solutions;
8. strong learning ability, good communication skills and teamwork spirit, able to work under pressure.

It is not difficult to see from the above, the position and benefits and other information written in full, so the candidate company and HR are quite plus points 🐶.

## 2. Self-introduction & past experience

The interviews were held in the evening because of the on-the-job status, and the interviews were held at 8:00pm.

The interviewer might be at home, so he didn't turn on the camera and explained it to me (this point is a plus, I hope that the interviewer of the candidate company is so polite, otherwise it will reduce the probability of being interviewed and passed 🐶).

![](https://s2.loli.net/2023/11/07/5Cx2DOfP9Asab6X.webp)

**Interviewer from Candidate: Hi, I'm an interviewer from xx company, this interview is mainly about tech stack examination and some programming questions, can you introduce yourself first? **

Self-introduction: Uh, okay. Then roughly say your graduation school, major, time, projects, good skills, 3~5 minutes.

> PS: The interviewer of the candidate company tries to turn on the camera as much as possible, if you really can't turn on the camera, you'd better explain it. You can introduce yourself at the beginning of the interview, and it will be a plus, even if it is a simple explanation.
> In order to minimize the number of words in the Q&A, the interviewer of the candidate company and the candidate will answer in the form of one question and one answer, i.e. Q&A.

**Q: Details of Project 1, what optimizations were done (I said I did system optimization when I introduced myself)**

A：MySQL high-end CRUD operation, Redis raging cache, Kafka delay bugs, etc., the eight stock text ready: [MySQL summary](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%) 3DMzI5Nzk2MDgwNg%3D%3D%26mid%3D2247484042%26idx%3D1%26sn%3D1620b3df43419745708f6f4c60a9ad9a%26chksm% 3Decac5683dbdbdf95197214ec82fe119ce0bc64c95b6806a8124a381f2c01131d41d6678b87e6%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26tokens 23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484042&idx=1&sn=1620b3df43419745708f6f4c60a9ad9a&chksm =ecac5683dbdbdf95197214ec82fe119ce0bc64c95b6806a8124a381f2c01131d41d6678b87e6&token=443585135&lang=zh_CN&scene=21#wechat_redirect" )

**Q: What are the specific responsibilities of Project 2 and why did you change jobs? **

A: The context of Project 2 is... A: The background of Project 2 is..., what value do I provide to users, what modules am I responsible for, what are my specific responsibilities, what are my effects or contributions, and what is the final result. You can **use the SMART principle to state that the responsibilities are clear, the goals are clear, and the results are desirable**.

The reason for changing jobs is that the current business has stabilized and I want to have more growth opportunities, and the technology and culture of your company is more attractive to me... I'm not sure if you're a good person, but I'm a good person, and I'm not sure if you're a good person! **

Compliment each other NB, by the way, prepare for why jump ship words and techniques, because of personal development, the work of geographical issues? Or is there a technical pursuit, want to learn more business and technical scenarios? In any case, you can not say some sensitive reasons, such as "more work, less money, the leader of the stupid hang" and so on.

> Although you do not intend to change jobs, you need to consider your reasons for jumping ship before the interview, so that the interviewer will know that you are "prepared", otherwise you will have to "rat tail juice".

## 3. Go Language Knowledge Points

**Q: Tell me the difference between make and new inside Go language? **

A: In Go, make and new are two very common keywords, and their differences are as follows:

* make can only be used to allocate and initialize data of types slice, map, and chan; new can allocate data of any type;
* new allocations return pointers, i.e., type *Type; make returns references, i.e., Type;
* new allocates space that is zeroed out, while make allocates space that is initialized.

**Q: Introduce the scheduling model of Goroutine **

A: This article gives a good response: [GPM scheduling](https://mp.weixin.qq.com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484182&idx=1&sn=6d3f54eea5622a2d7f6323cbb553fdd8&chksm =ecac571fdbdbde09cc8beb982e5df0caafdf5c87587cd3fbd69ca86c33724e9368ab957beac3&token=443585135&lang=zh_CN&scene=21#wechat_redirect )

**Q: What happens when you read or write to a closed chan**?

A: it will generate a panic, this question examines the basic use of chan, you need to understand the three situations in which chan generates a panic:

* Close an empty chan
* Close the already closed Chan again
* Send data to the already closed Chan

**Q: The underlying structure of a slice** **A: A slice is a variable-length object.

A: A slice is an array of variable length, and its data structure is defined as follows:

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

where Pointer is a pointer to an array, len represents the length of the current slice, cap is the capacity of the current slice, and cap>=len.

## 4. Operating Systems & Computer Composition Principles

**Q: Why is the overhead of process switching greater than that of thread switching**?

A: Process is the basic unit of resource allocation. Process switching involves refreshing the heap memory, replacing the global catalog of page tables, and switching the virtual address space.

Threads, on the other hand, are essentially just a group of processes sharing resources, so all threads of a process will share the virtual address space, which can **save the virtual address space switching**. And, when storing memory contexts, threads only need to save the data in **registers and program counters**.

> If you still don't understand the difference between processes and threads, continue to this article:[GPM Scheduling](https://mp.weixin.qq.com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484182&idx=1&sn=6d3f54eea5622a2d7f6323cbb553fdd8&chksm =ecac571fdbdbde09cc8beb982e5df0caafdf5c87587cd3fbd69ca86c33724e9368ab957beac3&token=443585135&lang=zh_CN&scene=21#wechat_redirect )

**Q: What's the difference between userspace and kernel space**

A: Kernel space is the area that the **operating system kernel** accesses. When accessing external devices such as disks, network cards, etc., you must first load the data into kernel space, so kernel space is a protected memory space.

The **user space is a memory area accessible to common applications** and cannot be directly loaded with data from external devices.

In the early days of operating systems, there was no distinction between kernel space and user space, so applications (such as writing a script) could access arbitrary memory space, which could pose some security risks: for example, if you wrote a for loop to send data, you could exhaust the entire server's disk network and other resources!

**Q: The difference between the send and sendfile data transfer functions under Linux**.

A: This is a network IO problem under Linux. Among them, send adopts the traditional standard IO method based on data copy, and the client needs to call the following functions when transferring data:

```arduino
buffer = File.read;
Socket.send(buffer)
```

When the send function is called, the computer takes four steps to get the data and transmit it to the network:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/976db374271649658327403113ca8400%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

1. copying disk data to the kernel-space buffer page cache via a DMA Copy operation;
2. copy data from kernel space to the application cache in user space via a CPU Copy operation;
3. copy user-space memory data to the kernel-space Socket Network Send Cache via CPU Copy operation;
4. copy the data from the Socket Cache to the NIC through DMA Copy operation, and then the NIC carries out network transmission.

From the above, we can see that send() needs to perform 4 context switches, 2 DMA and CPU data copy operations, which is a waste of resources and inefficient.

sendfile() is an implementation of the **zero-copy** technique, which avoids copying data to the user space (steps 2 and 3 in the above figure) when the application (user space) does not need to access the data during transmission and instead performs the CPU copy directly in the kernel space.

> Zero-copy: This refers to the fact that when a computer transfers data, it eliminates some unnecessary data copying operations in order to save the number of data copying and sharing bus operations, thus improving the efficiency of data transfer.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/f8af04c7072c41f695e84a0fb7744de5%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Linux introduced sendfile() in version 2.1 in order to improve the efficiency of data transfer. As you can see from the above figure, the zero-copy method during data transfer shortens the overall calling process from 4 to 3 steps and reduces the context switching between kernel space and user space.

However, the data transfer still needs to be copied in kernel space, can this step be omitted as well?

The answer is yes! In version 2.4 of Linux, the DMA Gather technique allows sendfile() to omit the CPU Copy operation as well (step 2 in the above figure), thus realizing zero-copy in the true sense of the word.

## 5. Database Knowledge Examination

**Q: What is phantom read, under what circumstances will appear, how to solve **

A: Phantom reads are a major problem at the Repeatable Read (RR) isolation level, but they occur at all isolation levels except **Serializable**. Phantom reads can be solved by changing MySQL's isolation level to Serializable, but transactions at this isolation level can only be executed sequentially, which is expensive, low performance, and rarely used.

Another solution to phantom reads is to use a gap lock or a critical key lock (gap lock + record lock) under **current read**, or multi-version concurrency control (MVCC) under **snapshot read**.

Under MySQL's InnoDB, all are current reads except for non-blocking query statements like `select xx from table`, which are snapshot reads, such as:

`select + for update`

`select + lock in share mode`

`udpate/insert/delete... `

> For those unfamiliar with isolation levels and MVCC, check out this article: [MySQL summary](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI5Nzk2MDgwNg%3D%3D% 26mid%3D2247484042%26idx%3D1%26sn%3D1620b3df43419745708f6f4c60a9ad9a%26chksm%26 3Decac5683dbdbdf95197214ec82fe119ce0bc64c95b6806a8124a381f2c01131d41d6678b87e6%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26token%3D443585135%26lang%3Dzh_CN%26scene%3D21%26tokens 23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484042&idx=1&sn=1620b3df43419745708f6f4c60a9ad9a&chksm =ecac5683dbdbdf95197214ec82fe119ce0bc64c95b6806a8124a381f2c01131d41d6678b87e6&token=443585135&lang=zh_CN&scene=21#wechat_redirect" ), knowledge points about MySQL locks will be added in subsequent articles in the public.

**Q: Distributed locking know, how to do distributed locking with Redis**

* A: In a multi-threaded environment, in order to ensure that a piece of code can only be accessed by one thread at the same time, if it is a standalone service, we can solve the problem by using a local lock. But in a distributed scenario, such as deploying 3 servers, how to ensure that a timed task can only be completed by a thread on one server?

  The answer is distributed locks. Distributed locks add locks when a thread accesses the code area and release the locks when the access is complete, which generally has the following characteristics:

  * Mutual exclusivity: at the same moment, only one thread can hold the lock;
  * Re-entry: a thread at the same node that has already acquired the lock can repeatedly acquire the lock while it is unreleased;
  * Avoid deadlocks: even if the server that added the lock is down, the lock is guaranteed to be released;
  * High-performance, high-availability: efficient locking and unlocking, and to ensure high availability, to prevent distributed lock failure.

  Redis implements distributed locking by setting an expiration time with the SET command, which is equivalent to the SETNX command:

  ```css
  set key value [EX seconds][PX milliseconds][NX][XX]
  ```

  * EX seconds: set the expiration time in seconds.
  * PX milliseconds: Set expiration time in milliseconds.
  * NX: Set value only if key does not exist.
  * XX: Set value only if key exists.

  Redis is implemented with several features to maintain distributed locking:

  * Mutual exclusivity: value must be unique;
  * Re-entry: depending on the characteristics of different threads, such as multiple servers executing the same piece of code, you can use the server's information (e.g., IP) as the unique value of the lock, and let the server determine whether it is reentrant based on the value information;
  * Avoid deadlocks: add an expiration time to the lock, so that even if the server executing the code crashes, that piece of data in Redis will be deleted periodically;
  * High-performance, highly available: Redis uses a cluster model, multi-node deployment to ensure high availability; Redis's distributed locking method is better than the MySQL or zookeeper locking method.

## 6. Networking & Distributed

**Q: Introduce the sliding window** of TCP

A: When the client and the server are doing TCP transmission, in order to increase the efficiency of the transmission, so the concept of **window** is introduced, that is: **Acknowledgement response is not transmitted in each maximum message segment, but in the form of a window **Acknowledgement**.

We know that when the two ends of the communication in the data transmission, TCP in order to ensure that the data is not discarded, so each message segment needs to be confirmed by the receiver before the sender can continue to send the next message segment, which is equivalent to the **blocking without buffer waiting **. However, the performance of the communication is low due to the different sending and consuming frequencies at the two ends of the communication.

The sliding window mechanism solves this problem, where the window size is the maximum value of data that the sender can continue to send. The implementation of sliding window, the use of buffering mechanism, you can confirm multiple segments at the same time, equivalent to adding a buffer, you can let the sender to send multiple message segments at a time, the consumer side is not so busy before processing.

Moreover, with the sliding window mechanism, when sending multiple data segments, if you receive an answer from a data segment, it means that the previous data segment has been successfully processed. For example: when the client sends 100,101,102 three data segments, ** if the server receives an ACK answer from 101, it means that the data before 102 has been received**, which can avoid the problem of repeated sending of data segments when one end does not receive an ACK answer.

**Q: Introduce the HTTPs**

A: You can read this article, which describes it very clearly: 

**Q: There are a lot of unidentified certificates on the browser, what are the risks if we click on trust? **

A: There will be a risk of **man-in-the-middle attack**. For example, a program with ulterior motives may add root certificates in the dark, and after we click trust it will intercept the TLS handshake request, and then send a temporary certificate signed by itself to the client, and then disguise itself as a browser client to create a connection with the server and do the subsequent negotiation of the secret key.

**Q: Symmetric encryption and asymmetric encryption

A: Symmetric encryption, in which both parties to a message use the same key to encrypt and decrypt the message, uses symmetric cryptographic coding techniques. Since the algorithm is public, the key cannot be disclosed to the public. It is less computationally intensive and encrypts quickly, with the disadvantage of insecurity and difficulty in key management, such as AES, IDEA.

Asymmetric encryption, can only be encrypted and decrypted by pairs of public and private keys, usually public key encryption, private key decryption. Process: Party A generates a pair of keys and discloses one of them as the public key. Party B gets the public key, encrypts the data and sends it to Party A, who decrypts it with a special private key. Asymmetric encryption is relatively safe, but the encryption speed is slow, such as RSA algorithm (RSA supports private key encryption, public key decryption).

**Q: Raft protocol understand it, introduce it**

A: Raft protocol divides distributed nodes into three categories, which are:

* Leader node Leader
* Slave node Follower
* Candidate node Candidate

There are three important scenarios when raft protocol works.

**1) Leader becomes a Candidate**

Each Follower receives periodic heartbeats from the Leader, typically 150~300ms, if no heartbeat packet is received after a period of time, the Follower becomes Candidate.

(**2) Candidate runs for Leader**.

After Follower becomes Candidate, it starts to send voting message to all other surviving nodes, other nodes will reply to its request, if more than half of the nodes reply to the campaign request, then the Candidate will become Leader node. If there is a tie, then each node sets a random time after which the campaign starts and all nodes vote again.

**3) New Leader Starts Working

The new Leader periodically sends heartbeat packets to the Follower, and the Follower receives the heartbeat packets and re-times itself. At this time, if the Leader receives the client request, it will write the data change into the log and copy the data to all Follower, and when most of the Follower make the change, it will submit the data change operation. The Leader will then notify all Follower to commit the changes, at which point all nodes agree on the data.

* ## 7. Algorithms and Data Structures

  **Q: What is the time complexity of fast sorting **

  A: O(nlogn) ~ O(n^2)

  **Q: What's the worst case of time complexity for fast scheduling**

  A: The worst case is when the middle number chosen each time is the smallest or largest element of the current sequence, which makes the recursion as many times as the length of the array.

  **Q: Given a two-dimensional matrix of mxn, write an efficient algorithm to determine whether a target value exists in the matrix** The elements in the matrix have the following characteristics:

  * The elements are incremented row by row, e.g. [1,3,8].
  * The elements are increasing column by column, e.g. [2,5,9].

![](https://s2.loli.net/2023/11/07/EGAVsTLWmIboze8.webp)

A: This question is not difficult, it is essentially a "matrix version" of binary lookup, which is also the original LeetCode question #74, implemented in Go:

```go
func searchMatrix(matrix [][]int, target int) bool {
   if len(matrix) == 0 || len(matrix[0]) == 0 {
       return false
  }
   m, n := len(matrix), len(matrix[0])

   i := sort.Search(m*n, func(i int) bool {
       return matrix[i/n][i%n] >= target
  })
   return i 
```

Q: Given an array, disorganize the elements of the array a bit so that each element appears in a completely random location (each location has exactly equal probability of appearing). **Requires space complexity of O(1) and time complexity of O(n)** .

For example, [1, 2, 3, 4, 5] ==> [2, 4, 1, 5, 3], where the probability of an element appearing in each position is the same.

A: Go code implementation:

```css
func randSort(arr []int) {
   n := len(arr)
   for i:=0; ii++ {
       rd := i+rand.Intn(n-i)
       arr[i], arr[rd] = arr[rd], arr[i]
  }
}
```

**Q: Given a unidirectional chained table, determine whether there are rings in the chained table **

A: LeetCode Original Question - Ring Chained Table, Go Language Code Implementation:

```go
type Node struct {
    Value int
    Next  *Node
}

func hasCycle(head *Node) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if fast == slow {
            return true
        }
    }
    return false
}
```

## 8. Summary

As you can see, the questions on one side are relatively basic, like those involving networks, group principles, and distributions are not deep, especially the algorithmic questions, which are basically LeetCode simple or moderately difficult questions. But this is also a feature of the technical interviews of ** big factories, may not be deep, but more comprehensive. **

However, for candidates, sometimes stuck on a certain knowledge point, the subsequent questions and answers may be more difficult, because some big factory interviewers will seize your weak points, and keep asking until you will not. The most important thing is that we all understand the Internet market today, and if you answer the question well, you may be dropped by the KPI.

Therefore, we have to seize the opportunity to prepare well before the interview, in the technical interview ** try to answer the highlights, as far as possible to answer the comprehensive, and can be cited to reflect three **. For example: when asked about processes and threads, you can mention Go's co-processing and talk about the difference between the three underlying mechanisms. When asked about memory management, you can talk about Go's own implementation mechanism (TCMalloc, etc.), and then talk about memory overflow, optimization, and other scenarios in conjunction with problems in the workplace.



