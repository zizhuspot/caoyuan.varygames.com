---
title: Write a good technical proposal with these 17 diagrams
date: 2023-11-05 00:11:00
categories: 
  - Technology
tags: 
  - technical
  - proposal
  - mountain
  - different
  - understand
  - generates
  - diagramming
  - design
description: The view from the side of a mountain is different from the view from near and far. In order to better understand the software system, we need to use a variety of diagramming tools to understand the system design from different perspectives.
cover: https://s2.loli.net/2023/11/05/WdFD1YG6LSJ8ATr.webp
---
The view from the side of a mountain is different from the view from near and far. In order to better understand the software system, we need to use a variety of charting tools, from different perspectives, a comprehensive understanding of the system design. So that in the design phase enough to fully predict the system bottlenecks, implementation difficulties, development time, etc., in the business function to achieve good scalability, performance, high reliability, high availability. Achieve absolute data security and excellent performance. Support rapid iteration of business requirements.

The software system can be layered as follows

1. Iass: Infrastructure software, including operating systems (networking, storage, compute), virtual machines, Docker, and other basic software.
2. Paas Platform as a Service: This includes our everyday messaging, caching, database, and other middleware. It should also include frameworks and libraries. For example, ioc, orm, rpc frameworks; images, special file processing third-party libraries and so on.
3. Saas Software-as-a-Service: Some of our everyday applications, be it 2B, 2C fall under this category.

In the following, we will focus on which diagramming tools can be used to analyse the application software system (Saas layer). These diagrams should be read by developers, product managers, business architects, system architects, technical administrators, and so on.

# use case diagram

Use case diagrams are the clearest and easiest to understand diagrams to use from the user's point of view.

## element (e.g. in array)

1. How to use our system. If the system has already been completed, a clean version of the user manual can be used instead.
2. What kinds of use processes are there and what are the application scenarios.
3. What needs to be done for each process。

The use case diagram first needs to be analysed in terms of who uses the system, which can be analysed with the help of the following questions

* Who will use the main functions of the system.
* Who will need the support of the system to do their job.
* Who will need to maintain, manage the system, and keep it in working order.

For example, a simple user profile change ![](https://s2.loli.net/2023/11/05/WdFD1YG6LSJ8ATr.webp)

Use case diagrams are easily overlooked by system designers because of their simplicity and straightforwardness. In fact, they are the friendliest and most straightforward way for an uninitiated person to get a quick overview of what functionality our system provides to whom. What are the connections between the functions?

## use case statute

The use case statute is a detailed description of the use case, which generally includes a brief description, main event flow, alternative event flow, pre-conditions, post-conditions and priorities.

The use case statute focuses on both success scenarios described in the main event flow and abnormal scenarios described in the alternative event flow, which is conducive to promoting systematic thinking, discovering abnormal scenarios, improving system functionality, and enhancing ease of use.

Postconditions should cover all possible end-of-use-case states. That is, postconditions should not only be the state after the use case ends successfully, but also include the state after the use case ends due to an error.

## typical example

![](https://s2.loli.net/2023/11/05/8tF7BwZ94GlahqS.webp) The general case can be described only for important requirements, the use case statute for critical use cases. General requirements can be ignored. (They should never be ignored in the PRD)

Use case diagrams and system pages A more detailed user manual would allow for a quicker and comprehensive understanding of the system's functionality. For example, show our system page. It is more visual and clear.

Only with a better understanding of what functionality the system provides and what roles it has can you understand why the system is designed the way it is? Some people find use case diagrams superfluous because they know enough about the system itself. But they don't realise that others are still completely new to the system. Use case diagrams are the most straightforward way to understand a system.

# Data model diagram

Programmes = data structures + algorithms, a software programme is the processing of input data to output specific data according to a certain algorithm. The data is the core of the programme, and it is also the part that is prone to change, such as the most common change: the need to add or subtract fields.

At this point the data needs to be modelled and the relationships between the models sorted out in order to put the most closely related data into a model that can be extended independently. A data model diagram describes the relationships between models and what fields are in each model. Three Elements ● Models ● Attributes ● Relationships between Models

Data model diagrams include E-R diagrams, database entity diagrams. etc.

* E-R diagrams are simpler using Chinese and do not involve table structures.
* Database Model Layer: Describes the relationships and table design at the database level, which is more complex than an ER diagram, but more comprehensive and understandable to all people.。
* Most of the time our design documents are for product and R&D, technical managers. It is also readable using database entity diagrams. (Let him learn if he can't read it)

Personally, I think it is possible to ignore the E-R diagram in the design document and go straight to the database entity diagram. However, this requires the database entity diagram to have sufficient textual descriptions, such as attribute notes, relationship descriptions, etc.

## E-R graphical symbol

![](https://s2.loli.net/2023/11/05/L6iMf4uOUhvIPcy.webp)

# Database entity diagram (database ER diagram)

![](https://s2.loli.net/2023/11/05/YOg3s2bK8ewxBMD.webp)

## The process of sorting out ER diagrams for databases

The design of database ER diagrams, entity diagrams or domain model diagrams is a test of design experience. It requires domain experts to communicate requirements based on use case diagrams, use case flowcharts, iterative requirements Constantly push the following questions

1. Where the business is expanding, where it is changing, and where it plans to evolve in the future.
2. Where is the system expanding? How to achieve scalability
3. Which domain entities should be included.
4. Where should the boundaries of the domain model be. Are the associations 1V1, 1 to N, etc.?

The process of analysing database ER diagrams can be done using design methods such as DDD.

# flow chart

System Design Phase Only the core and critical business processes need to be covered. Some simple processes in the minutiae should be ignored. (There is energy and time to cover non-core processes)

## Management Processes and User Processes

Take the marketing system as an example, it is divided into management process and user process.

* Operations creates a marketing campaign. This process is a management process. This process does not have high performance requirements. The traffic portal is different.
* User orders and other behaviours, triggering certain marketing activities, this process for the performance requirements, security requirements are very high.

Different processes have different focuses. For example, the dubbo rpc system can be divided into the initialisation process and the method invocation process.

* The rpc provider interface provider needs to register the interface at initialisation time. rpc consumer needs to listen to the interface at initialisation time.
* rpc call flow From the consumer side to the provider is the call flow.

## Flowcharting

![](https://s2.loli.net/2023/11/05/3Pytuj7nq5T6fMA.webp) Flowcharts are more flexible in the way they are drawn. The following are personal experiences and habits

* Component Thinking: can describe control flow calls between components
* Data Thinking: Can describe changes in data flow between data.

Generally, boxes are used to represent components, and lines are used to represent calls to methods, actions, or data.

## Component thinking

Thinking in terms of components Design a flowchart. Requires that the components of the system be abstracted first, and which steps are handled by each component. It is a simplified version of the invocation timing diagram, with inputs and outputs diluted. (Timing diagram describes the method invocation hierarchy, which is more detailed and clearer.)

The following is an example of a control flow diagram of a component invocation that exposes a service on the dubbo Provider side.

![](https://s2.loli.net/2023/11/05/57xthRkIDPHUezX.webp)

## data mentality

A data flow diagram describes the transfer of data between components, and the data flow describes what the inputs and outputs of a component or system are.![](https://s2.loli.net/2023/11/05/YWrXO4DCod3qSf8.webp) 

In fact, in most business systems, the use of data flow diagrams is not very well delineated, because two or three models are mainly processed within a business process. The boundaries between the inputs and outputs of the components are not clear, and for this reason it is possible to combine the data flow diagram and the control chart into one. The boxes still represent the components, but the lines can include both actions and data.![](https://s2.loli.net/2023/11/05/S4kLOybHuoJWqgr.webp)

## Non-functional design in flowcharting

High reliability, high availability, performance bottlenecks, flow charts can be introduced to the core read and write process of high availability, high reliability design; that is, how to ensure the reliability of the data, how to ensure the availability of the system. Which node in the process is the performance bottleneck. How to optimise and so on

以下仅供参考

1. 数据存在哪里，同步/异步写入，异步写入的一致性保证
2. 并发操作如何保证一致性（例如库存？）
3. 高并发场景如何提高系统可用性，读流程如何优化，写流程如何优化
4. 是否 Stand-by 设计双活。
5. 幂等性，重试策略、负载均衡策略

# 时序图

A timing diagram is a more detailed system flowchart. A typical system flowchart covers key system components, key data processing nodes, but is not specific to any class or method. Timing diagrams require attention to the control flow of program execution, and the presentation of timing is more like a method call stack during system execution.

## Timing diagram elements

1. method call stack (core)
2. branching and looping description.
3. method and entry descriptions
4. annotations for the main key nodes

## Timing diagram example

![](https://s2.loli.net/2023/11/05/w1tkhcvZnMPUxQf.webp)

You can see that the timing diagram is accurate to the extent that a class calls a method of a class, showing the depth and level of nesting of method calls. Together with the important key node comments, the reader can see the overall method invocation system even without reading the code. Through the timing diagram we can get

1. where a class is located in the timing diagram and what it is responsible for.
2. what are the upstream and downstream dependencies of a class in the current flow

# Class Diagram

A class diagram describes the dependencies between classes (combinations, inheritance, interfaces).

## Class diagram elements

1. inheritance, implementation, and combinatorial dependencies
2. key core methods of the class. (Remember to add comments to describe what capabilities the class extends)

Here is a class diagram of a Spring ioc container.

Translated with www.DeepL.com/Translator (free version)![](https://s2.loli.net/2023/11/05/MYiQfKpJUqdLWc1.webp)

The arrow relationship of a class diagram Describes whether there is a dependency, integration/combination/interface implementation between two classes.

## When to use class diagrams?

### Simple business processes don't need class diagrams.

For example, if there is only one implementation of an interface, and there is no complex inheritance, you don't need to write a class diagram.

### Complex inheritance systems require class diagrams.

Complex inheritance systems require class diagrams. In order to achieve maximum reusability and extensibility, a large number of inheritance and interface implementation classes are used to improve scalability and reusability. At this time, without a class diagram, it is impossible to fully understand the inheritance system of an interface or a class. It's not clear where a class fits into the inheritance system.

## Why can't class diagrams be classified as flowcharts?

In general, only flowcharts and timing diagrams can specify a class. When the reader sees several classes in a flowchart or timing diagram, he or she wonders how the classes are related.

At this point, you can choose whether you need to organize a class diagram to show the dependencies.

## Class diagrams and design patterns

Class diagrams are used in the introduction of design patterns. For example, the class diagram for the Factory pattern![](https://s2.loli.net/2023/11/05/ZPh7eBrF8LYxq1s.webp)

# System Architecture Diagram

System architecture diagrams are used to describe the components, modules, etc. within an application. In general, there are two types of diagrams: full system architecture diagram and single application architecture diagram.

## System Architecture Diagram

Business Architecture Diagrams show the hierarchy and relationship between various systems of an organization from the perspective of business logic.A business architecture diagram is a neat presentation of the hierarchy and relationships between the various systems of an organization from the perspective of business logic.![](https://s2.loli.net/2023/11/05/uXvIAkdPpbQch1m.webp)

## Single Application Architecture Diagram

Single-application business architecture diagrams can be categorized into the classic three-tier structure according to the hierarchy: presentation layer, business logic layer, and data layer.![](https://s2.loli.net/2023/11/05/agOcbteNo67mxB2.webp)

# Application Architecture Diagram

The application architecture diagram focuses on the location of the application within the system. (Similar to the system-wide Business Architecture Diagram above). The application architecture needs to describe the location of the application within the system.

The following is an application architecture diagram for a coupon system. It basically describes the position of an application service within the entire coupon-related microservices.

For example, the CouponJob service is responsible for issuing coupons, and there are various forms of coupon activities associated with this service in the upper layer. The CouponJob service is responsible for issuing coupons, and there will be various forms of coupon activities associated with this service. Redemption codes, coupon pages, and so on, all rely on the CouponJob coupon service.

![](https://s2.loli.net/2023/11/05/gzm5xUhOsBcNAMk.webp)

Inter-application dependencies Ideally, the dependencies should be unidirectional. If there are obvious cyclic dependencies between two upstream and downstream services, then you need to consider whether the two systems are heavily coupled, and whether the two systems implement similar functionality. Do they need to be merged into a single service?

Is it more appropriate to put business modules with strong correlation into one service?

Application architecture describes the dependencies between applications and where they are located in the system. Is it a top-tier application or a bottom-tier application? When designing an application architecture diagram, it is not recommended to include the module divisions within the application in the application architecture diagram, as this will result in an overly large architecture diagram, which is not conducive to understanding.

This will result in a large diagram that is not easy to understand **An architecture diagram only needs to describe a single view**. (with a single responsibility as far as possible)

The following is the application architecture diagram for HBase![](https://s2.loli.net/2023/11/05/WeKA7odP8CkMpOz.webp)

This can be seen in the table

* Zookeeper is responsible for cluster management tasks such as survivability monitoring.
* master is not responsible for data read and write, only responsible for statements such as DDL build table, responsible for RegionServer failover process
* RegionServer is responsible for receiving client read and write data
* HDFS as a distributed storage, receive RS read and write (As for the RegionServer internal what modules, how to read and write naturally with the help of other architectural diagrams. Each architecture diagram describes the system from only one viewpoint)

# Deployment Architecture Diagram

Deployment architecture diagrams focus on describing how an application is deployed online.

## Deployment architecture diagrams focus on the core concerns

* Whether traffic is coming from the admin side or the user side.
  - Is it authenticated?
* Which nginxes is the traffic coming from, public or intranet, and which domain name (public intranet, different domain name).
* Whether the application is load balanced, what is the policy?
* Whether the application is deployed in multiple server rooms, and in which server rooms the application is deployed.
  - How the traffic is routed between the server rooms; whether there are cross server room calls.
  - Whether the application is setup by user dimension
  - Whether or not the server room routing is done by region dimension
  - What are the routing rules for rpc calls and where are they managed?
* Deployment room Is it a virtual machine, physical machine, or Docker. private cloud, public cloud, or hybrid cloud architecture
* Are the dependent databases and applications deployed in the same machine room?

* Other middleware such as MQ, Redis, Es, etc. Where is the machine room deployed. Is it across server rooms, etc.

The following deployment architecture describes The application is deployed on containers and user requests are SLB load balanced. Static resource access, database services are deployed on RDS.

![](https://s2.loli.net/2023/11/05/SCqvBjE5oauVKyt.webp)

# All-link calls Structural diagram

The following scenarios require grooming of the full-link upstream and downstream dependency graphs

1. Interface parameters need to be changed and interface implementation needs to be changed. Confirm that it does not affect the upstream (generally the implementation details are blocked to the upstream.)
2. Downgrade the interface, migrate to a new interface.
3. notify the upstream of flow limiting, fusing, and degrading policies.

When the above occurs, we need to realize which upstreams depend on us, in what business scenarios they depend on us, and how to unify the upstream and downstream dependencies on system rpc interfaces, http interfaces, mq consumers, shared databases, and so on, in terms of importance and priority. This can be done in the form of a table, for example![](https://s2.loli.net/2023/11/04/3zbSgrtCMoUO5lp.webp) For method calls within the project, module dependencies, we can quickly sort out the upstream dependencies. For example, using IDE shortcuts. However, in the case of microservices, we need to obtain the call chain diagram, and we can only rely on the service governance framework and code scanning tools to sort out the upstream dependencies of each interface.

This may reveal the need to authenticate the interface to prevent arbitrary callers from being able to invoke our service. At least we can be aware that the interface is being invoked to prevent high traffic and unreasonable business scenarios from being invoked. It also makes it easier for us to upgrade in the future.

## Each architecture diagram is a unique perspective ##

## Architecture diagram perspectives

* Use Case Diagram: Users, product managers, and testers can visualize and clearly understand the usage scenarios of our system and what it can do.

* Database Entity Diagram: business architects, business experts, product managers to understand the modeling process, whether there are unclear domain delineation and domain coupling problems.
* Flowchart: for development engineers, business architects, business experts to more clearly understand the core process, how the data flow between components, how each component calls the
* System Architecture Diagram: Development engineers can use this to quickly understand the total number of modules in the system, how to carry out layering. The responsibilities of each layer.
* Application architecture diagram: Business architects can see whether there is coupling between microservices, whether the dependency is unreasonable, and whether the boundaries are clear. Development engineers can see where the system they are responsible for fits into the overall architecture.
* Application Deployment Architecture: Business architects, development engineers, and operation engineers can have a clearer understanding of the service deployment environment, traffic ingress, load balancing strategy, routing strategy, middleware deployment, and so on.

Above we have analysed what specific diagrams should be included in the design document, in addition to the

# Design Documentation Basics

1. The design document should be written according to the following basic principlesDocumentation is for people. To be thorough enough for the target audience, each reader's perspective must be taken into account.
2. Can be landed. Require documents with sufficient design details, can accurately predict the technical difficulties, for technical difficulties to put forward a clear solution (to cover the key requirements and processes)
3. An architecture diagram should only describe one view. The pursuit of a large and comprehensive architecture diagram will only make it difficult to read and impossible to modify it. (Try to have a single responsibility)
