---
title: Redis persistence doesn't make sense, so that's it for today.
date: 2023-11-07 02:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - Interview
  - Redis
  - memory
  - development
  - framework
  - message
  - corresponding
description: Whether domestic or foreign, from the Fortune 500 companies to small startups are using Redis, many cloud service providers also built on Redis as the basis of the corresponding caching services, message queuing services, and memory storage services, when you use these services, in fact, it is in the use of Redi
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/86d2084082dd4cf2a25a2a2acc2cd0ca%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---
Table of Contents

1. Introduction
2. RDB
3. AOF
4. Comparison of Two Persistence Mechanisms
5. Summary

## 1. Introduction

Q: What is Redis?

A: Redis, or Remote Dictionary Server, is an in-memory cache database written in C and widely used in Internet products.

Whether domestic or foreign, from the top 500 companies to small startups are using Redis, many cloud service providers also Redis-based caching services, message queuing services, and memory storage services, when you use these services, in fact, is to use Redis.

When you use these services, you're actually using Redis.** As a developer, you're bound to be asked about it in interviews, even if you don't use it on the job! **

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/86d2084082dd4cf2a25a2a2acc2cd0ca%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

_Q: Why is Redis so important and what are some common application scenarios for it? _

A: The reason lies in the fact that Redis is a purely in-memory operation with high execution efficiency and supports rich data structures itself. It has a wide range of application scenarios, including but not limited to ** caching, event publish or subscribe, distributed locking ** and so on.

_Q: Are all Redis operations in-memory? _

A: No. The single-threadedness of redis means that it is a single-threaded operation when **receiving client IO requests for reads and writes**. But there are multi-threaded scenarios for redis itself, such as asynchronous deletion, persistence, and cluster synchronization.

Redis as a common knowledge point in the interview, the common eight-legged text must have been familiar with. However, in today's increasingly voluminous Internet market, relevant knowledge points, can we answer the depth and breadth of the interviewer wants?

For example, today we want to review the knowledge point, Redis persistence mechanism.

_Q: What are the common Redis persistence mechanisms? _

A: RDB and AOF.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/658c3d8e5c174f7dac1ac0a7fbb1e852%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

## 2. RDB

#### 2.1 Introduction

RDB (Redis Database Backup file), or snapshot mode, is the default data persistence method in Redis.

RDB is actually a **timer event** within Redis that checks the **times** and **times** of changes to the current data every once in a while to see if they meet the trigger conditions specified in the configuration file.

When the conditions are met, Redis creates a child process through the operating system call fork(), which ** shares the same address space, file system, and semaphores** as the parent process. After that, Redis traverses the entire memory space through the child process, copies the data set to a temporary file, and when the copy is complete, notifies the parent process to replace the original file with the new RDB file to complete the data persistence operation.

At the same time, during the persistence process, the parent process can still provide services to the outside world, and the parent and child processes realize the separation of data segments through the multi-process **COW (copy and write) mechanism** of the operating system, so as to ensure that the parent and child processes do not affect each other.

#### 2.2 Summary of advantages and disadvantages

During RDB persistence, the Redis fock child process saves all Redis data to a newly created dump.rdb file, which is a resource-consuming and time-wasting operation. This is a resource-consuming and time-consuming operation. Therefore, Redis servers should not create rdb files too often, or the performance of the server will be seriously affected.

In addition to this, the biggest shortcoming of RDB persistence is: **There can be a lot of data loss during the last persistence process**. We imagine a scenario, in the process of RDB persistence, Redis server suddenly down, then the child process may have generated rdb file, but the parent process has not yet had time to use it to cover the old rdb file, the buffer in the file has not been saved, will lead to a large number of data loss.

The advantage of RDB data persistence is that it restores very quickly, so it is more suitable for large-scale data recovery. If you are not particularly sensitive to the integrity of the data (allowing for the loss of data during the persistence process), then RDB persistence is very suitable.

## 3. AOF

#### 3.1 Introduction

AOF, append only log file, is also known as append mode, or log mode.

AOF logs all write commands executed by the server, **and only those commands that modify memory**, and Redis writes these commands to the appendonly.aof file at regular intervals.

We can re-execute the AOF file to restore the dataset when the server starts up, a process known as **command replay**.

#### 3.2 Three Persistence Mechanisms

When Redis receives a modification command from a client, it will first perform the corresponding checksum, and if the command is error-free, it will immediately store the command in a buffer, and then append the buffer data to the .aof file at a certain rate.

In this way, even if there is an unexpected downtime, you only need to store the command in the aof file and perform a "command reenactment" to restore to the state before the downtime.

In the above execution process, there is a very important part of the **command write, which is a disk IO operation**: in order to improve the write efficiency, Redis does not write the content directly to disk, but first puts it into a memory buffer, and then only when the buffer is full or meets the persistence policy of the AOF does it actually write the content in the buffer to the disk (fsync operation). disk (fsync operation).

There are three persistence strategies (i.e., the frequency of fsync operations) for AOF:

* always: every time the server writes a command, it calls the fsync function once to write the command inside the buffer to the hard disk. In this mode, a server failure will not result in the loss of any command data that has been successfully executed, but its execution speed is very slow;
* everysec (default): the server calls the fsync function once every second to write the commands in the buffer to the hard disk. In this mode, if the server fails, the command data executed within one second at most will be lost, and it is usually used as the AOF configuration policy;
* no: the server does not call the fsync function, the operating system decides when to write the commands in the buffer to the hard disk. In this mode, when the server encounters unexpected downtime, the number of lost commands is uncertain, so this strategy, uncertainty is greater, and is not commonly used.

Redis is still at risk of losing data if the data in the cache is not written to disk before experiencing downtime. The number of commands lost depends on when the commands are written to disk: ** The earlier the commands are written to disk, the less data will be lost in the event of an accident. **

Since is fsync is a disk IO operation, it is slow! If Redis has to fsync once (ALWAYS) to execute a command, it will severely impact Redis performance.

In production servers, Redis usually fsyncs every 1s or so by default (everysec) to maintain high performance and minimize data loss.

The last strategy (no), which lets the operating system decide when to synchronize data to disk, has many uncertainties and is not recommended.

> Note: The sync and fsync functions are two functions provided by the operating system to prevent data inconsistencies in caches and files caused by "delayed writes".
> The sync function puts the modified data into the cache write queue and returns without waiting for the end of the IO operation.
> fsync, on the other hand, waits for the end of the IO operation before returning, and ensures that modified blocks are written to disk immediately to ensure that the file data is consistent with the cache.
> i.e., the Linux fsync() function flushes the contents of a given file from the kernel cache to the hard disk synchronously, and sync() operates asynchronously.

#### 3.3 Rewrite Mechanism

Under the AOF persistence policy, Redis runs for a long time, and the aof file gets longer and longer. If the machine is down and restarted, the command to reenact the entire aof file will be very time-consuming, which will cause Redis to be unable to provide services to the public for a long time.

Therefore, in order to keep the size of the aof file within a reasonable range, Redis provides an AOF rewrite mechanism, that is, the aof file for "thinning": the Redis server can create a new AOF file to replace the existing AOF file, ** the old and the new files save the same database state, the difference is that the new file does not have the task redundancy commands ** so the size of the file will be smaller than the old file, the old file will not have the task redundancy commands ** so the new file will be smaller than the old file. The difference is that the new file does not have task redundancy commands**, so the file size is much smaller than the old one.

Redis provides two ways to rewrite an AOF file: manually execute the BGREWRITEAOF command, or configure a policy to rewrite it automatically. AOF file rewriting is similar to the RDB persistence process, which involves forking a child process to manipulate the original AOF file.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e7f77e6cf4c840b88631aaf6106b5687%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As shown in the figure: the parent process continues to process new requests during the AOF rewrite process. If new commands are added, they are appended to the AOF rewrite buffer and then directly appended to the new AOF file.

## Comparison of AOF and RDB

RDB persistence mechanism AOF persistence mechanism full backup, save the whole database incremental backup at a time, save only one command to modify the database at a time each time to execute the persistence operation of the interval is longer, the interval of saving is one second by default (everysec) data is saved in a binary format, its restore speed is faster using text format to restore data, so the data restore speed is generally execute the SAVE command will block. SAVE command will block the server, but manually or automatically triggered BGSAVE will not block the server AOF persistence will not block the server at any time.

Before Redis 4.0, we could only choose RDB or AOF as the persistence mechanism; after Redis 4.0, we can configure Redis persistence to be a hybrid mechanism, i.e., RDB+AOF are both used as persistence methods, for details, please refer to the redis.conf file:

```bash

rdbcompression yes

rdbchecksum yes

dbfilename dump.rdb

appendfilename "appendonly.aof"

appendfsync everysec

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-rewrite-incremental-fsync yes

rdb-save-incremental-fsync yes
```

> If for data recovery, there is both an RDB file and an AOF file, we should recover the data through the AOF file first, as this maximizes the security of the data.

## 5. Summary

2023 has come and gone, and the days of the Internet expanding like it did in previous years and grabbing a share of the application windfall are gradually passing! If the tide of the computer industry is receding for a short or long period of time, what will determine whether programmers will "dry swim" on the beach? Is it marginal business, or is it an aging crisis?

I do not think so, in fact, the rising tide and the ebbing tide is a trend, some people catch the crisis early, early to wear a good bathing suit to the shore; some people are not enough to the status quo, to the sea deeper swim. They are the wise man in the Internet wave, the wise man will not worry about the wave receding, because they have been prepared in advance. And the opportunity is always in favor of these early prepared people!

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a2def93e9814100923de8f63b012c01~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

Young people, let's join hands and dogpile under the wave of the Internet together~

