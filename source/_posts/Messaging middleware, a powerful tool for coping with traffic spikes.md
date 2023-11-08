---
title: Messaging middleware, a powerful tool for coping with traffic spikes
date: 2023-11-07 01:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - data
  - immediately
  - recognize
  - development
  - consumptio
  - middleware
  - major
description: When the amount of data (passengers) is too much, the system (the speedboat carrying passengers) can not immediately consume, the data will first be put into a consumption queue (shore step) to wait, play a role in traffic peak shaving. Inside the distributed system, a major way to realize the consumption queue is to use message middleware.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/86a42f45d8c34a98bff5843acea4c24b%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp
---
### 1. Introduction

On the weekend, I drove to the beach with my friends. Those who have been to Yangmeikeng should know that you need to take a speedboat to go there from Yangmeikeng to Lukzui Mountain Resort.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/86a42f45d8c34a98bff5843acea4c24b%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Worthy of Shenzhen tour attractions on the top 5 places, 4 or 5 pm when the queue for the boat is still very many people, buy a good ticket we were called to a shore steps waiting to get on the boat, the scene is slightly chaotic.

**The crowds were a bit heavy, but there weren't a lot of boats arriving to take passengers. **

Just as I was generally sweating for the staff maintaining order, I saw them go back and forth ordering several groups of people to get those people on the boat in an orderly fashion.

Not long after that, a thin, dark, middle-aged man came to call us, saying that a boat could hold 10 people, so he ordered the 10 people in front of us to stay put, and the 10 people who had been ordered could get on the boat.

Sure enough, software design comes from life, and this is the classic data consumption problem in system design!

### 2. Message middleware

When the amount of data (passengers) is too much, the system (the speedboat carrying passengers) can not consume the data immediately, it will put the data into a **consumption queue** (the shore ladder) and wait for it, playing the role of a traffic peak shaving.

** In distributed systems, a major way to implement consumption queues is to use message middleware **.

#### What is Message Middleware

Message Broker is an infrastructure component for delivering messages, notifications and events in distributed systems.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e2439e84a7344d8d845b41f848ec1263%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

It allows data and information to be exchanged asynchronously between different components, applications, or systems to enable clipped, decoupled, and scalable communication.

The fundamentals of message middleware include the following key concepts:

1. **Message Producer (Producer):** This is the sender of the message, usually an application or component, which sends the message to the message middleware.
2. **Message Consumer:** This is the receiver of the message, usually an application or component, which receives and processes the message from the message middleware.
3. **Message Queue: ** This is the core component of the message middleware, it is a queue to store messages, message producers will be put into the queue, message consumers from the queue to get the message. The message queue usually uses the first-in-first-out (FIFO) principle.
4. **Message Topic (Topic):** In addition to message queues, message middleware also supports message topics, which allow publish-subscribe model of message communication. Message publishers publish messages to topics, while subscribers can subscribe to specific topics to receive related messages.

Advantages of message middleware include:

* **Decoupling:** Message middleware allows producers and consumers to operate independently; they do not need to be directly aware of each other's existence. This decoupling makes the system more flexible and maintainable.
* **Scalability:** By increasing the capacity of the message middleware, it can easily handle more message traffic and consumers.
* ** Asynchronous Communication:** The message middleware allows asynchronous communication, where producers can continue to work without having to wait for a message to be processed, thus improving the performance and responsiveness of the system.
* **Message Persistence:** Messages are usually persisted so that they are not lost even if the message middleware or consumer fails.

There are many different implementations and protocols for message middleware, and some of the popular ones include ActiveMQ, RocketMQ, RabbitMQ, Kafka, and others.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/d96af938b105410a8ed242aedda89319%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

They have different characteristics and advantages in different usage scenarios and requirements.

Message middleware is widely used in a variety of applications, including ** microservice architecture, big data processing, real-time data analysis, log collection, event-driven architecture ** and so on.

Next, we introduce common messaging middleware and their advantages, disadvantages and applicable scenarios to help you make an informed choice in application development.

### 3. ActiveMQ

**Features:**

* ActiveMQ is a Java-based open source messaging middleware that implements the JMS (Java Message Service) specification.
* Support for multiple messaging models , including peer-to-peer and publish-subscribe .
* Provides high availability and load balancing , supports master-slave replication , can be used to build high-availability systems .
* Suitable for Java applications, but there is some support for clients in other programming languages.

**Benefits:**

* Easy to use for rapid development and prototyping.
* Integrates with the Spring framework for easy integration with Spring applications.
* Suitable for small to medium sized systems and intra-enterprise communications.

**Disadvantages:** * Relatively low performance.

* Relatively low performance, not suitable for high throughput and latency demanding scenarios.
* Does not support large-scale message flow, not suitable for big data and real-time analysis applications.

** Applicable Scenarios:** ActiveMQ is suitable for internal communications that require simple messaging and small to medium-sized systems. It performs well in intra-enterprise communication and lightweight applications, but is not suitable for high performance, high throughput and large-scale data processing.

In general, ActiveMQ domestic Internet companies landing less , mostly traditional enterprises in use.

### 4. RocketMQ

**Features:** ** RocketMQ is an open source MQ framework from Alibaba.

* RocketMQ is Alibaba's earlier open source MQ framework , based on the Java language written , and later donated to Apache , is a fast , reliable , scalable distributed messaging middleware .
* Supports publish-subscribe and peer-to-peer messaging models.
* With high performance and low latency , suitable for large-scale messaging .
* Supports a rich set of client-side languages, including Java, C++, Python, Go, and so on.

** Advantages:** * High performance and low latency, suitable for large-scale messaging.

* High performance and low latency, suitable for high-throughput large-scale applications.
* Supports multiple messaging models, suitable for different business scenarios.
* With powerful monitoring and management tools.

**Disadvantages:**

* Deployment and configuration are relatively complex and require some specialized knowledge.
* The community is relatively small , compared with some other messaging middleware , documentation and ecosystem is relatively immature .

** Applicable Scenarios:** RocketMQ is suitable for large-scale applications that require high performance, low latency, and scalability, such as e-commerce platforms, financial systems, and Internet of Things applications.

### 5. RabbitMQ

**Features:**

* RabbitMQ is an open source messaging middleware that implements the AMQP (Advanced Message Queuing Protocol) specification.
* Supports a wide range of messaging models, including peer-to-peer, publish-subscribe, and RPC.
* Provides reliability messaging , support for transactions and message acknowledgment .
* Has multiple client libraries and supports multiple programming languages.

**Benefits:**

* Mature technology with high stability, widely used in enterprise applications.
* Provides high availability and load balancing mechanisms.
* Supports multiple programming languages, suitable for cross-language applications.

**Disadvantages:**

* Relatively low performance, not suitable for large-scale applications with high throughput.
* Deployment and configuration complex, need some learning costs.
* itself is erlang language development, source code is more difficult to analyze, need solid erlang language skills.

** Applicable Scenarios: ** RabbitMQ is suitable for enterprise-level applications that require reliability and transaction support, but do not have particularly high performance requirements.

### 6. Kafka

**Features:**

* Kafka is a high-throughput, low-latency distributed messaging middleware for large-scale data processing and real-time stream processing.
* Mainly used in the publish-subscribe model to store messages as logs.
* Highly scalable and available , suitable for building large-scale real-time data streaming applications .
* Support for a variety of clients, including Java, Python, Go and so on. ** Pros:** ** Cons:** ** Applicable Scenarios:** Kafka is suitable for applications that require high throughput, low latency, and large-scale data processing, such as log collection, real-time data analytics, and event-driven architecture.
  *
  - Complex to deploy and configure, requires specialized knowledge.
  - Not suitable for small-scale applications, high relative complexity.
  - High throughput and low latency for large-scale data processing and real-time stream processing.
  - High scalability and support for building large-scale data pipelines.
  - Data persistence and data replication to ensure data reliability.



### 7. Technology Selection

#### RabbitMQ and Kafka

RabbitMQ and Kafka are two of the most commonly used messaging middleware, the main differences between them are:

* **Performance:** The performance of message middleware is mainly measured by throughput. Kafka's stand-alone QPS can reach the million level, RabbitMQ's stand-alone QPS is in the 10,000 level, and kafka is even higher;
* **Data Reliability:** kafka and rabbitMQ are equipped with multi-copy mechanism, data reliability is higher;
* Consumption mode: Kafka is actively pulled by the client, and RabbitMQ supports two modes: active pull and server push. So RabbitMQ message real-time is higher, and for the consumer is more simple; and kafka can be pulled by the consumer according to their own situation, throughput is higher;
* ** idempotency:** kafka supports idempotency for a single producer, single partition, single session, while RabbitMQ does not support it;
* **Other features:** RabbitMQ supports priority queues, delayed queues, dead message queues (**store queues of messages that can't be consumed**), and more.

#### How to choose the right messaging middleware

In application development, choosing the right messaging middleware depends on specific requirements:

* If your application is a **small to medium-sized system** with low performance requirements, and is more concerned with ease of use and rapid development, then ActiveMQ may be a good choice.
* If you need to handle **large-scale messaging** and are looking for high performance and low latency, then RocketMQ or Kafka may be a better fit, depending on your application type and requirements.
* If your application is an **enterprise application** that requires reliability and transactional support, but does not require high performance, then RabbitMQ may be a good choice.
* The final choice also depends on your technology stack, the experience of your team, and specific business requirements. It is recommended that you carefully evaluate your application requirements before choosing a messaging middleware and make your choice on a case-by-case basis.

Of course, no matter which messaging middleware you choose, you need to have an in-depth understanding of its features and how to use it to ensure that it can meet the application requirements to build an efficient and reliable distributed system.

### 8. Conclusion

Regardless of which message middleware is used, we can often see the wonderful use of consumption queues in our daily lives.

**With these buffering methods, our daily travel and consumption order can be well protected. **

