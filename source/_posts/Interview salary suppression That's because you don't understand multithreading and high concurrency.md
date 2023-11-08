---
title: Interview salary suppression? That's because you don't understand multithreading and high concurrency.
date: 2023-11-06 21:04:00
categories: 
  - Technology
keywords: 
tags: 
  - Interview
  - developers
  - multithreading
  - interviews
  - development
  - stranger
  - network
  - daily
description: As developers, whether in job interviews or in their daily work, I'm sure you're no stranger to high concurrency and multithreading.
cover: https://s2.loli.net/2023/11/07/KYPIFmZ3aANpi9t.webp
---
## 1. Introduction

As developers, whether it is a job interview, or in the daily work, I believe that we are no stranger to high concurrency and multi-threading.

During job interviews, the backend job requirements that roll out of the sky often require us to **familiarize ourselves with high concurrency and multi-process/multi-threading**:

![](https://s2.loli.net/2023/11/07/HVMlbD9rdPBv3KY.webp)

In our daily work, with the rise and development of mobile Internet applications, the system tasks and problems we face are becoming more and more complex.

Whether we are building large-scale web applications, processing huge data sets, or developing high-performance games, we all need to deal with a common challenge: high concurrency.

### 1.1 What is high concurrency?

** High concurrency refers to a large number of users or programs accessing and using a service or resource at the same time period. **

This means that we need to handle a large number of requests, data, and tasks at the same time. How to handle this situation efficiently becomes a critical technical task.

High concurrency is an area of challenge, but also an area of opportunity.

### 1.2 What does multithreading have to do with high concurrency?

Solving the problem of high concurrency not only improves the performance of the system, but also improves the user experience and brings more business opportunities for the enterprise.

Multi-threading technology is one of the most important tools to address the challenge of high concurrency. **

Therefore, in this post, Xiao ❤ will take you together to explore high concurrency and multithreading in depth, and familiarize you with the working principle of multithreading, application scenarios, as well as practical solutions to solve the problem of high concurrency.

I believe that no matter you are a junior programmer or a developer with some experience, you will be able to find useful information in this article.

## 2. High Concurrency

## 2.1 Concurrency and Parallelism

#### Concurrency

Concurrency is the execution of multiple tasks in the same time period. On a single-core processor, multiple threads switch execution between themselves by means of time-slice rotation, resulting in a concurrent scenario.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/654db73469bd4dc2a47a289d9fa56709%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Equivalent to our video recordings, when the video frame rate is high enough (i.e., switching between multiple images in a second), our naked eye treats it as a continuous and smooth video.

#### Parallel

On a multicore processor, true concurrency can be achieved by multiple threads performing different tasks at the same time - that is, in parallel.

![](https://s2.loli.net/2023/11/07/KYPIFmZ3aANpi9t.webp)

Parallelism is the execution of multiple tasks at the same moment, usually requiring a multi-core processor. **Parallelism is a subset of concurrency, and true parallelism can only be achieved if the hardware supports multiple parallel execution units**.

#### Thinking Questions

**Scenario 1:** Why is it difficult for us to focus on answering the phone while playing a game with an intense group battle?

**Scenario 2:** We can listen to music while fiddling with the steering wheel without interfering with each other while driving. Why don't you guess whether our brains run concurrently or in parallel?

### 2.2 How much concurrency is too much concurrency?

Having understood the concept of concurrency, let's now talk about high concurrency.

High concurrency is a relative concept that depends on the performance and processing power of the system. ** Typically, a system is said to be highly concurrent when the number of requests or transactions it needs to handle exceeds its normal load. **

### 2.3 Challenges of High Concurrency

While high concurrency brings many opportunities, it also comes with many challenges.

For example, a highly concurrent system needs to handle a large number of requests in a short period of time without degrading the performance or responsiveness of the system.

This may involve **multiple users** accessing a website at the same time, **multiple clients** requesting server data at the same time, or **multiple threads** accessing shared resources at the same time.

In a distributed system, whether it's multiple users accessing, or multiple clients accessing a server, it all boils down to the business threads of each server accessing shared resources, so **high concurrency challenges are almost always related to multithreading. **

When faced with high concurrency, the following problems specifically arise.

![](https://s2.loli.net/2023/11/07/JkSq59HPau3Ge4p.webp)

#### 1. Competing conditions

Multiple threads accessing shared resources at the same time may lead to data inconsistency problems. For example, multiple threads depositing money into the same bank account at the same time may result in an incorrect balance.

#### 2. Deadlock

Multiple threads wait for each other to release resources, causing the system to stall. For example, thread A waits for thread B to release a lock, and thread B waits for thread A to release a lock, creating a deadlock.

#### 3. Resource Contention

Multi-threaded access to shared resources can lead to resource contention problems that can degrade performance. For example, multiple threads competing for a database connection at the same time results in a slower database response.

#### 4. Thread Safety

There is a need to ensure that no errors are raised when multiple threads access shared data. For example, in a multi-threaded environment, there is a need to ensure that reading and writing to data is safe.

#### 5. Debugging Difficulty

Since the order of execution of multithreading is uncertain, problems may appear at different times. So debugging of multithreaded programs is relatively complex and problems are difficult to reproduce.

### 2.4 Solving High Concurrency Problems

In order to solve the high concurrency problem, appropriate techniques and methods need to be used, which are as follows.

![](https://s2.loli.net/2023/11/07/YIw83fNqVbmaA7U.webp)

#### 1. Lock Mechanism

Locks are used to **protect shared resources** by ensuring that only one thread can access them at a time.

Locks can be categorized as `&#x4E92;&#x65A5;&#x9501;` and `&#x8BFB;&#x5199;&#x9501;`. Mutual-exclusion locks are used to exclusively occupy a resource, while read/write locks allow more than one thread to read a resource at the same time, but only one thread is allowed to write to it.

Specific implementation details can be found using the locking mechanisms provided by the programming language, such as the `synchronized` keyword in Java or `threading.Lock` in Python.

#### 2. Concurrent Data Structures

Use concurrent data structures such as concurrent queues and hash tables to **reduce resource contention**. These data structures are optimized to work efficiently in a multithreaded environment.

For example, Java provides `ConcurrentHashMap`, which is a thread-safe hash table that can be used in highly concurrent environments without explicit locking.

#### 3. Thread Pooling

Manage and reuse threads to improve performance. A thread pool controls the number of threads, **to avoid wasting resources by having too many threads**.

In Java, you can use `ExecutorService` to create and manage thread pools. This avoids frequent creation and destruction of threads and improves efficiency.

#### 4. Message Passing

Inter-thread communication through message passing model avoids shared memory. Message passing ensures that data is passed safely between threads, **reducing competing conditions**.

For example, in Go, you can use channels (`channel`) for messaging to ensure the safe passing of data.

#### 5. Atomic operations

Atomic operations are indivisible operations that ensure that **multiple threads operating on shared variables are safe**. Atomic operations are often supported by third-party libraries or features that can be used to implement various synchronization mechanisms.

In C/C++, you can use atomic operations to manipulate shared variables, for example with the `atomic` library. In MySQL, the transaction threads of the `InnoDB` engine can carry their own atomicity features.

## 3. Multithreading

## 3.1 Processes and Threads

When a task in concurrent work is completed, it will be switched from one segment of the program to another to be executed, and a series of states of the previous segment of the program operation will be lost if not saved, so the operating system introduces processes to carry out resource isolation.

#### 1. Processes

**Processes are used to delineate the basic unit of resources required when a program runs**, it has an independent address space, an independent stack, when the process is switched, it can ensure that the respective data storage is not affected.

Because the process involves the consumption of a large number of resources, it is strictly controlled by the computer operating system (can be understood as: the approval of land resources in each province and city, are very cautious, especially the first-tier cities, so by the core department of unified control).

Therefore, ** process switching occurs in the kernel state, by the computer core program to unify the scheduling. **

> **Trivia:**
> The operating system is divided into kernel state and user state, and the `CPU` (Central Processing Unit) in the kernel state can access arbitrary data.
> The CPU in the kernel state can access any data, > including peripheral devices such as network cards and hard disks, and there is no preemption of the occupied CPU.
> Whereas a CPU in the user state can only access memory in a restricted manner and is not allowed to access peripheral devices, and the CPU in the user state may be seized by other programs.

#### 2. Threads

When a process is switched, the kernel state has to be switched, so it consumes a lot of resources, for which the concept of threads is introduced.

A **thread is the smallest unit of operating system scheduling and is an execution process within a program**. A process can contain multiple threads, which share process resources such as memory space and file handles, but each has a separate stack memory.

The thread itself takes up almost no resources, it ** shares address space and heap ** with other threads in the process, so the scheduling time consumption is relatively small, but it has an independent CPU context (including CPU registers, program counters, etc.).

Threads are like sharing the same land resources with the threads inside the same process, but threads have their own office buildings, and switching between threads is unified scheduling by the operating system.

> Trivia: Threads are divided into kernel threads and user threads, and user threads must be bound to kernel threads before they can run.

### 3.2 Multithreading Concepts

Multithreading is a type of concurrent execution that allows a program to be divided into multiple independent threads, each of which can perform tasks independently. It's like being able to work on a piece of land resource at the same time without affecting each other.

#### 1. Creating and managing threads

Creating and managing threads involves the scheduling mechanism of the operating system, which is implemented in different ways in different programming languages. Let's take Python as an example:

```arduino
import threading

def my_function():
  

thread = threading.Thread(target=my_function)
thread.start()  
```

#### 2. Thread synchronization and mutual exclusion

When multiple threads access a shared resource at the same time **, it may lead to a race condition **, where multiple threads compete with each other for the resource, which may result in inconsistent data.

To solve this problem, we use a locking mechanism to ensure that only one thread can access the shared resource at the same time.

```csharp
import threading

lock = threading.Lock()

def my_function():
   lock.acquire()  

   lock.release() 
```

### 3.3 Multi-threaded applications

Multi-threading can not only improve the performance of the program, but also improve the user experience. In real life, we often encounter multithreaded application scenarios.

#### 1. Web server

Imagine a popular social media site with millions of users visiting the site at different times at the same time.

These users request different pages, upload photos, make posts, and also have some background tasks such as data backup, new post push, and so on.

At this point, the **Web server needs to handle requests from multiple users at the same time. Each user request can be seen as a thread, and multithreading allows the server to respond to multiple requests at the same time. **

For example, one user may request to view their profile, while another user may request to post a new status update. These two requests can be handled by different threads at the same time, improving the server's response time.

#### 2. Database system

Suppose an online banking system where thousands of customers access their account information simultaneously, checking balances, transferring funds, and so on. In addition, the banking system needs to handle customers' deposit and withdrawal operations.

At this point, the **database system needs to handle multiple customer requests** at the same time. Each customer request can be regarded as a thread, and multiple threads can query the database at the same time to ensure that each customer's account information is up-to-date.

#### 3. Game Interaction

A multiplayer online game where dozens of players participate in the game at the same time. This game needs to handle player actions, physics simulation, AI computation, and multiplayer game interaction simultaneously.

At this point, the **game engine can use multiple threads to handle different aspects of the task**. One thread can be responsible for rendering the game graphics, another thread can handle the player's actions, and another thread can be responsible for simulating the physics effects in the game.

With multi-threading, the feedback of the game system will be smoother and the player can enjoy a highly interactive game experience.

## 4. Summary

When talking about multithreading and concurrency, it's like the busy streets of our daily lives, where everyone is dealing with their own things, but at the same time need to coordinate their interactions with others.

![](https://s2.loli.net/2023/11/07/RnMyHUVpPFILv7t.webp)

