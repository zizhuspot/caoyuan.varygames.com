---
title: Unlocking the Black Art of MySQL: Transactions and Isolation
date: 2023-11-07 07:04:00
categories: 
  - Technology
tags: 
  - Backend
  - MySQL
  - Interviews
  - serialized
  - development
  - result
  - network
  - Frequent
description: Frequent locking may result in reading data with no way to modify it, and modifying data with no way to read it, greatly degrading database read and write performance, just as serialized isolation levels do.
cover: https://s2.loli.net/2023/11/07/U5QlW9EceiO4YVT.webp
---
## 1. Introduction



![](https://s2.loli.net/2023/11/07/vaxf5TzS4n3sNq7.webp)

In MySQL, the most frequently asked questions are transaction, isolation level, and MVCC, whether it is a large Internet company, a small factory, or even a state-owned enterprise, their coverage rate is as high as 80%.

In fact, the interviewer also knows that everyone will memorize the eight-legged text, but can say that understand, and even say thorough candidates are very rare.

So today I'm going to take you to unlock the black technology hidden in the bottom of MySQL: transactions and isolation.

## 2. Transactions

## 2.1 Straight Reward

First, let's talk about transactions.

Transactions are like a magic show that **ensure that a series of database operations either all execute successfully or none at all. **

Let's say you're watching a live stream and you want to reward the hostess with 500 bucks. You need to deduct your account balance and increase the hostess's account amount at the same time.

![](https://s2.loli.net/2023/11/07/95FQwdimPVATEDf.webp)

If one of the two operations of transferring money fails, then you can lose money or have it disappear and the beauty queen won't receive it.

This is where transactions come in handy.

It can ensure that both operations either succeed or fail at the same time, there will never be an embarrassing situation of half-success and half-failure.

So, let's **summarize:**

* Q: Why do databases have transactions?

* A: In order to ensure that the business runs properly and the data is ultimately consistent.

> For those who don't understand what ultimate consistency is, take a look at this previous post of mine: [In-depth: Distributed, CAP, and BASE Theory](https://link.juejin.cn?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%) 3DMzI5Nzk2MDgwNg%3D%3D%26mid%3D2247484896%26idx%3D1%26sn%3D60dd09486fc9ecc652af917d8a311419%26chksm% 3Decac51e9dbdbd8ffc10b79699ea7e4a8fb00aabc743b15cc5c3311970a9e3046592cbb879364%26scene%3D21%23wechat_redirect "http://mp.weixin.qq. com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484896&idx=1&sn=60dd09486fc9ecc652af917d8a311419&chksm= ecac51e9dbdbd8ffc10b79699ea7e4a8fb00aabc743b15cc5c3311970a9e3046592cbb879364&scene=21#wechat_redirect")

### 2.2 Transaction Characterization

Now that you understand what a transaction is and why you need one, let's talk about the 4 characteristics of transactions.

Let's talk about the 4 characteristics of transactions: **Atomicity, Consistency, Isolation, and Durability**, or ACID for short.

#### Atomicity

Atomicity means that **a transaction contains operations that are either all successful or all unsuccessful**.

For example, the initial balances of accounts A and B are $800 and $100. At this point, A transfers $500 to B. The breakdown is A account minus $500 and B account plus $500.

The end result is that account A has a balance of $300 and account B has a balance of $600. The operation of updating the balance of these two accounts is either performed in full or not performed at all.

Taking the example of rewarding a beautiful anchorwoman, atomicity ensures that either the money is still there, or the money is transferred to the account of the anchorwoman and the anchorwoman's **thank you brother** is rewarded!

#### Consistency (Consistency)

**The state of consistency is maintained before the transaction is executed, and after it is executed**.

Two things happen to accounts A and B after a transfer:

1. the money is transferred to account B. At this point, accounts A and B are $300 and $600 respectively;
2. the money is transferred out of the process of database network disconnection, the transaction is rolled back, A, B account or 800, 100 dollars.

In any case, before and after the transaction, the total amount of A and B bank accounts should be $900, which is inconsistent.

#### Isolation

Isolation is when more than one user accesses the database concurrently, no matter whether it is operating the same library or the same table, the transaction opened by the database for each user can not be interfered by the operation of other transactions, and multiple concurrent transactions should be isolated from each other.

For example, when A transfers money to B, no matter how others transfer money, it will not affect their transactions.

![](https://s2.loli.net/2023/11/07/J8RdDvo3PZykHh2.webp)

Taking the example of giving a reward to a beautiful anchorwoman, isolation is: no matter how many people are giving the anchor a reward, it will not affect your affairs of transferring money, and it will not affect the anchor to call you a **good brother**!

#### Persistence (Durability)

**Once a transaction has been submitted, then the change to the data in the database is persistent [i.e., saved to disk]** , even in the case of database system failure will not be lost to submit the operation of the transaction.

Take the example of rewarding a beautiful anchorwoman, persistence is: you just transfer money to the anchor, the money into her account, no matter how many voices **thank you good brother** harvested anchor, the money can not come back.

Next, we **summarize: **

* Q: Why do transactions have these characteristics?

* A: We want to ensure that the data consistency of the transaction, we need some means to achieve, these means are several characteristics of the transaction.

They are atomicity, consistency, isolation, and persistence, where ** consistency is the goal, and atomicity, consistency, and isolation are all means to achieve data consistency**.

## 3. Transaction Concurrency and Isolation

### Transaction concurrency

Concurrency is the ability of a computer system or program to handle multiple tasks or operations at the same time, that is, to allow multiple user processes to work on the same critical area.

> For those who want to understand concurrency from a process or processor perspective, see this previous article of mine: [GPM Scheduling Model](https://link.juejin.cn?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI5Nzk2MDgwNg% 3D%3D%26mid%3D2247484182%26idx%3D1%26sn%3D6d3f54eea5622a2d7f6323cbb553fdd8%26chksm% 3Decac571fdbdbde09cc8beb982e5df0caafdf5c87587cd3fbd69ca86c33724e9368ab957beac3%26scene%3D21%23wechat_redirect "http://mp.weixin.qq. com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484182&idx=1&sn=6d3f54eea5622a2d7f6323cbb553fdd8&chksm= ecac571fdbdbde09cc8beb982e5df0caafdf5c87587cd3fbd69ca86c33724e9368ab957beac3&scene=21#wechat_redirect")

Take the reward anchor for example, concurrency is multiple viewers want to reward the anchor, if you transfer money together, then the anchor account balance how to modify it?

![](https://s2.loli.net/2023/11/07/KJnSksRliU54F9Z.webp)

The task here is to transfer money, the user process is the server process responsible for the transaction, and the critical zone is the storage space for the anchor account.

If transaction concurrency occurs, it can cause some unexpected problems, such as the common **Dirty Write, Dirty Read, Duplicate Read, and Phantom Read**.

### Dirty writes

Dirty writing means that during transaction concurrency, **one transaction can modify the data of another ongoing transaction, which may result in one write transaction overwriting the data of another write transaction**.

When you and Xiao Shuai together to the beauty of the hostess reward, you rewarded 500 dollars, Xiao Shuai rewarded 1,000 dollars, in the write database, you write the data is Xiao Shuai's data to be overwritten.

The final result is that your money is gone, and the hostess is saying **thanks for the reward** in the live broadcast!

### Transaction isolation

500 bucks is gone, and the hostess is still ignoring you, you're sad, but you don't know what to do?

Don't be sad! Transaction isolation can help you.

![](https://s2.loli.net/2023/11/07/FM8A7eVjqhanWwR.webp)

MySQL provides transaction isolation levels, including: **Read uncommitted, Read committed, Repeatable reads, and Serialization**, to solve various concurrency problems in transactions, and to cure all kinds of unhappiness.

### RU - Read uncommitted

RU (Read Uncommitted) means that if a transaction starts writing data, another transaction is not allowed to write at the same time, but other transactions are allowed to read this row of data.

RU can exclude writes, but does not exclude read thread implementations.

This isolation level solves the dirty write problem above, but there may be **dirty reads, i.e., transaction B reads data that transaction A has not committed**.

You want to give the beautiful anchorwoman reward 500 bucks, found that the bank card balance is only 300 bucks, then you thought of a few days ago you borrowed 500 bucks Xiaoshuai, so let Xiaoshuai pay back the money.

Xiao Shuai is very clear about the database isolation mechanism, know that you are in the RU transaction isolation level. So he says he'll pay you back immediately, and the following scenario occurs:

![](https://s2.loli.net/2023/11/07/U5QlW9EceiO4YVT.webp)

* Shuai: open transaction A, transfer money to you 500, the transaction is not submitted;

* You: open transaction B, check the balance, and find that the balance has been added 500, so the Shuai Shuai debit note torn off, and ready to give the anchor reward;
* Shuai: see the debit note is gone, so revoke the transaction A. His money is not a penny less, and you only read the balance in his transaction A, but the real balance did not increase, that is, a dirty read has occurred;
* You: The balance of the reward payment is insufficient, and you lose the debit note worth 500 bucks.

You are so disappointed that you plan to cut ties with Handsome and continue to learn the rest of the isolation mechanism to see how to prevent dirty reads from occurring.

### RC - Read committed

This level of isolation prevents other transactions from accessing a row of data (both read and write) while it is being written to by one transaction. This ensures that the data read by a transaction is committed, **solving the problem of dirty reads**.

However, RC will appear ** unrepeatable read ** problem, for example: transaction A need to read the data twice, after reading the first data, there is another transaction B to update the data and submit the transaction.

At this time, transaction A read the data again, the data has changed, that is, ** transaction in the two read data inconsistent **.

In order to salvage his friendship, Handsome transfers you 520 bucks, but he thinks it's okay to pay you back only 500 bucks, so he asks you to pay him back 20 bucks.

You'll be too busy watching the show, so you won't have time to transfer the money. He suggests that you tell him the account password of your bank card, and he'll only transfer 20 bucks.

![](https://s2.loli.net/2023/11/07/YIzGXKCmLRN5Jl3.webp)

To be on the safe side, you open a transaction to check the card balance and tell Shuai the password, the following scenario happens next:

* You: open transaction A, inquired about the bank card balance of 820;
* You: open transaction B, withdraw 800, and submit transaction B. * You: open transaction B, withdraw 800, and submit transaction B;
* You: in transaction A again to check the balance, found that the bank card only 20 dollars, the occurrence of unrepeatable read.

Not only did you not get the money you borrowed, but you lost 280 bucks. The more you think about it, the more angry you get, and you scold the marshal. Then you continue to study the isolation mechanism to see how to prevent the unrepeatable read problem.

### RR - Repeatable read

When the same data is read multiple times within the same transaction, no other transaction can access the data (including reads and writes) while the transaction is still open.

This isolation level ** solves the problem of dirty reads and unrepeatable reads ** but there may be ** phantom reads **.

If transaction A reads the data several times, another transaction B inserts or deletes data in the middle of the data rows, then transaction A reads again, you may find that the number of rows of data has changed.

In short, **RR - Repeatable Read ensures that the current transaction will not read other transactions have committed `update` operations, but can not sense other transactions `insert &#x548C; delete` operations **.

Shuai knows you won't lend money again, and was scolded by you, and is resentful. So he thought of using your bank account to mess things up, and the next scenario happened:

* You: open transaction A, want to query the transaction just a few times, the transaction to see the result is 2 times;
* Shuai: open transaction B, found that you can not modify your balance data, so simply to your bank card inside the write 100 times the transaction record, the transaction amount up to tens of millions of dollars, submit transaction B. * You: continue inside transaction A, the transaction amount up to tens of millions of dollars;
* You: inside the transaction A continue to query the number of transactions, found that it became 102 times;

At this point, the police uncle came to the door, said someone reported you malicious money laundering, need to assist in the investigation.

![](https://s2.loli.net/2023/11/07/jawgtfP8QVhFcqH.webp)

Luckily, after some explanation and investigation through the logs of the bank's database, it was discovered that someone had maliciously tampered with the transaction records, and you returned home safe and sound.

At this point, you learned the hard way and were shocked to realize that you had made a bad friend! So you sink your teeth into the isolation mechanism.

### Serializable

At this isolation level, transactions can only be executed sequentially, **solving the problems of dirty reads, unrepeatable reads and phantom reads**. However, it is more expensive and has very low performance, and is generally seldom used.

In this case, every time a viewer, like you, wants to give a reward to the anchor, you need to wait in line until the previous transaction transaction is completely finished.

At this point, you learn about the wonders of transactions and the importance of isolation, and plan to learn about databases and stop watching beautiful anchors dance.

![](https://s2.loli.net/2023/11/07/CpQ3D67UbVwc5Fl.webp)

Handsome, on the other hand, gets lost further and further down the road of bureau-oriented programming.

## 4. Summary

Let's summarize that databases solve the various problems that arise from transaction concurrency through isolation levels:

* RU, read uncommitted solves the problem of dirty writes, but dirty reads may occur;
* RC, read committed solves the dirty read problem, but unrepeatable reads may occur;
* RR, Repeatable solves the problem of unrepeatable reads, but phantom reads may occur;
* Serializable, serialization solves the problem of phantom reads, but performance is low.

> How does MySQL implement transaction isolation?

The answer is locking. The higher the transaction level, the more concurrent transactions are solved, which also means more locks are added.

Comparison of number of locks: RU-Read Uncommitted < RC-Read Committed < RR-Repeatable < Serializable-Serialized.

![](https://s2.loli.net/2023/11/07/neNL5jRqzHZv1Od.webp)

However, frequent locking may result in no way to modify the data when reading it, and no way to read it when modifying it, which greatly reduces the database read and write performance, just like the serialized isolation level.

So, in order to trade off data security and performance, MySQL databases use RR, the Repeatable Read isolation level, by default.

