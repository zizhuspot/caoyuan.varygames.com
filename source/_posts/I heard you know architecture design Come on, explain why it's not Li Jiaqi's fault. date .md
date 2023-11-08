---
title: I heard you know architecture design? Come on, explain why it's not Li Jiaqi's fault.
date: 2023-11-06 19:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - code specification
  - absolutely
  - accompanied
  - development
  - framework
  - abstraction
  - method
description: A method that is too long is one that does too much work inside a method, often accompanied by statements in the method that are not at the same abstraction level, such as a mix of dto and service level code, i.e., the logic is scattered.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/264a5dd1dd204ecb9cde8d105f2eddd1%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp
---
# 1. Introduction

Today, I'm going to talk to you about refactoring, the mysterious skill of programmers! Don't worry, I'm going to help you understand and master this skill with easy-to-understand language and some fun conversations that my 8-year-old niece says she understands.

## 1.1 Background

Code Development:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2a84de458d4d4fcb80fc883fa7ed21a7%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

One month later:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/9aea7c9997a048768166d4805cdf47e7%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Later **there's time** change it (don't worry, there won't be time for that, and I won't change it when there is time).

Six months later:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/61eb652ce31c47c3ae56d8c0fa1c7129%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

As above, it is a scenario that any developer will experience: the early code simply cannot be reviewed, or else it will surely fall into deep suspicion, is such a bad code really from their own hands?

What's more, most of the current systems are collaborative development, and each programmer's naming conventions and coding habits are different, leading to a situation of one system code, multiple flavors.

#### What is refactoring

Yeon: Hey uncle, I heard you're going to share refactoring, what's new about that?

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/264a5dd1dd204ecb9cde8d105f2eddd1%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

❤：Hi Yeon! Refactoring is the process of improving the design of existing code to make it more understandable and easier to maintain, without changing its functionality. Think of it as giving your code a beauty makeover, but on the inside it's still the same code, not an "unrecognizable person".

#### Why Refactor

Lulu: Wow, that sounds awesome, why do we need to refactor?

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e06b300927364a3eb074e56e16bddda3%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

❤：Haha, good question, Lulu! Because code is alive and getting bigger day by day, when it becomes difficult to understand and modify, it is like a heavy elephant that slows down our pace. Refactoring is like putting weight on the elephant, making it lighter and more flexible, and development speed can be increased quite a bit!

In the same way that you guys have a little cleanliness fetish and love to clean up your room, programmers with code cleanliness fetishes refactor their code all the time!

#### When to refactor

Yeon: Sounds reasonable, but when should you use refactoring?

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/36907733ca3a42d692a515132d16313f%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

❤：Good question, Yeon! There are several scenarios:

* When you see several places in your code that look exactly the same, this is a good time to consider merging them into one to reduce redundancy.
* When you have a function or method that looks thicker than a dictionary, break it up into smaller parts that are better understood.
* When you want to fix a bug, but realize that the original code structure is so complex that fixing it becomes as hard as solving a puzzle, refactoring before fixing it is a good idea.
* When you want to add new functionality, but the code doesn't let you extend it easily, it's also a good idea to refactor first and then extend.

#### Steps to refactoring

Lulu: I see uncle, so what are the exact steps of refactoring?

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/3041751045e04a7d94b45053272acd98%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

❤：Good question, Lulu, it looks like you're seriously thinking about it! Let me walk you through the basic steps of refactoring next!

# 2. How to refactor

Before refactoring, we need to identify the bad-flavored code inside the code.

By bad flavor, I mean the superficial messiness of the code, and the deep-seated corruption. Simply put, it's code that just doesn't feel right.

## 2.1 Bad-flavored code

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/11a13edcea64423e9e1f9d409a0d8a6f%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

In the book Refactoring-Improving the Design of Existing Code, these two dozen bad taste situations are described, and we'll pick the most common ones below.

### 1) Methods that are too long

Methods that are too long are those that do too much work in a single method, often accompanied by statements in the method that are not at the same abstraction level, e.g., a mix of dto and service level code, i.e., fragmented logic.

In addition to this, methods that are too long tend to cause a number of additional problems.

#### Problem 1: Excessive annotations

Methods that are too long are difficult to understand and require a lot of comments. If 10 lines of code require 20 lines of comments, the code is hard to read. If 10 lines of code require 20 lines of comments, the code is hard to read. Especially when reading code, you often need to remember a lot of context.

#### Problem 2: Procedure-Oriented

The problem with process-oriented code is that when the logic is complex, the code can be difficult to maintain.

On the contrary, we often use object-oriented design thinking in code development, i.e., abstracting things into objects with common characteristics.

#### Solution Ideas

To solve the problem of long methods, we follow the principle that whenever we feel the need to write a comment to explain the code, we write this part of the code into a separate method and name it according to the intent of this code.

> Method naming principle: it can summarize what to do, not how to do it.

### 2) Overly large classes

A class that does too many things, for example a class whose implementation contains both product logic and order logic. There will be too many instance variables and methods that are difficult to manage at creation time.

In addition to this, too large a class tends to cause two problems.

#### Problem 1: Redundancy and duplication

When a class contains the logic of two modules inside, the two modules are prone to create dependencies. This can easily lead to the problem of "you take me, I look at you" during the code writing process.

That is, in both modules, the program structure or methods with the same intent are seen related to the other module.

#### Problem 2: Poor coupling structure

When the naming of a class is insufficient to describe what it is doing, the probability is that the coupling is poorly structured, which is contrary to the goal of writing code with "high cohesion and low coupling".

#### Solution

Split large classes into small classes based on business logic, and if there is a dependency between two classes, associate them through foreign keys and so on. When duplicate code occurs, try to merge and present it as much as possible, and the program will become more concise and maintainable.

### 3) Logical decentralization

Logical fragmentation is due to unreasonable dependencies at the code architecture level or object level, which usually leads to two problems:

#### Fragmentation

A class is often modified in different directions for different reasons.

#### Scattered changes

When a certain change occurs, it needs to be modified in multiple classes.

### 4) Other bad flavors

#### Data Mud Clusters

A data mudball is a confusing blend of many data items that are not easily reused and extended.

It is easier to categorize many data items when they always appear together and when they appear together. We can then consider encapsulating data into data objects by business. The reverse example is below:

```go
func AddUser(age int, gender, firstName, lastName string) {}
```

After the refactoring:

```go
type AddUserRequest struct {
   Age int
   Gender string
   FirstName string
   LastName string
}
func AddUser(req AddUserRequest) {}
```

#### Basic Type Paranoia

In most high-level programming languages, there are basic types and structural types. In Go, basic types are int, string, bool, and so on.

Basic type paranoia refers to the fact that when defining variables for an object, we often use the basic type without considering the actual business meaning of the variable.

An example is the following:

```go
type QueryMessage struct {
Role        int         `json:"role"`
Content  string    `json:"content"`
}
```

After the refactoring:

```go

type MessageRole int

const (
HUMAN     MessageRole = 0
ASSISTANT MessageRole = 1
)

type QueryMessage struct {
Role        MessageRole   `json:"role"`
Content  string               `json:"content"`
}
```

* This is the request field when ChatGPT asks a question, we can see that the conversation role is of type int, and 0 means human, 1 means chat assistant.

  When we use int to represent the conversation role, there is no way to know more information directly from the definition.

  But by defining it with `type MessageRole int`, we can clearly see that there are two types of conversation roles based on constant values: HUMAN & ASSISTANT.

  #### Confusing Hierarchical Calls

  Our general system is hierarchical based on business service, transit controller and database access dao. Generally, the controller calls the service and the service calls the dao.

  If we call the dao directly from the controller, or if the dao calls the controller, there will be a hierarchical confusion, which can be optimized.

  ### 5) Problems caused by bad flavors

  YanYan: Uncle, do all these bad flavors need to be addressed, and what kind of impact do you think these bad flavors of code will bring?

  ❤: Yes, code with too many bad flavor codes will bring **four "difficult "** .

  * **Difficult to understand**: new developer students can't understand the code of the people watching, a module after two weeks of reading still don't know what it means. Maybe it's not the level of the developer is not enough, may be the code is written too hard to say.

  * * * difficult to reuse * * *: either read can not read, or barely read but do not dare to use, for fear of what dark pit. Or the system is so coupled that it's hard to separate the reusable parts.

  * **Difficult to change**: affects the whole body, i.e., scattershot modification. Move one piece of code and the whole module is almost gone.

  * **Difficult to test**: change is not good to test, it is difficult to carry out functional verification. The naming is messy and the structure is confusing, and new problems may be measured during testing.

  # 3. Refactoring Tips

  Lulu: Oh, so that's it, can we remove them then?

  ❤: Of course you can! Just like you guys love to clean up your room, every responsible (code cleanliness) programmer will consider code refactoring.

  And the industry already has a better way of thinking about the refactoring problem: removing the "bad taste" from the code through continuous refactoring.

  ### 1) Naming conventions

  A good naming convention should conform to:

  * Accurately describe what is done
  * Formatting that conforms to common conventions

  #### common conventions

  Let's take Huawei's internal Go language development specification as an example:

  Scenario Constraints Example Project names are all lowercase, separated by an underscore '-' for multiple words user-order Package names are all lowercase, separated by an underscore '-' for multiple words config-sit Structure names are initialized in uppercase Student interfaces use Restful API naming, with the last portion of the path being a resource noun such as [get] api/v1/ student constant name initial capitalization, camel naming CacheExpiredTime variable name initial lowercase, camel naming userName, password

  ### 2) Refactoring techniques

  YanYan: Wow, so many mature specifications can be used ah! So besides the specification, do we need to pay attention to anything else?

  ❤: Good question Yeon! Next I will also introduce some common refactoring techniques:

  * **Extract function**: break a long function into smaller chunks, easier to understand and reuse.
  * **Rename**: rename variables, functions, classes, etc. to make more sense.
  * **eliminate redundancy**: find similar chunks of code and merge them to reduce duplication.
  * **Move**: move functions or fields to more appropriate places to make the code more organized.
  * **Abstract generic classes**: pull out generic functions into a class to increase code reusability.
  * **Introduce parameter objects**: when there are too many variables, pass in objects to eliminate data clumps.
  * **Use the guard statement**: reduce the use of else and make the code structure clearer.

# 4. Summary

Lulu: Uncle, you're so funny, I feel like I can refactor too!

❤: Lulu is awesome, I believe in you! The idea of refactoring is everywhere, just like all life should be white space, your life will be very wonderful too. In programming, refactoring can make the code more beautiful, easier to read and understand, and improve the development efficiency, which is a skill that all programmers should master.

YanYan: I can do it, too! In the future, I will also write code and do code refactoring, and I will also like my uncle's article.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/eb2bf78dae3443ffabec61ef6881e50d%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

❤：Hahaha, well da, you guys are great! Just like you guys like to clean and paint and read poetry, if you want to write code in the future, they'll be very clean and poetic too!
