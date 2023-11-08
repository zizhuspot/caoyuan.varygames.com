---
title: A Deep Dive into RabbitMQ Sequential Consumption, Dead Letter Queues
date: 2023-11-06 02:04:00
categories: 
  - Backend
tags: 
  - RabbitMQ
  - Backend
  - Message Queue
  - important
  - development
  - framework
  - messages
  - consumption
description: RabbitMQ is an open source messaging middleware that implements the Advanced Message Queuing Protocol (AMQP) while providing a variety of important components to support the production, transport and consumption of messages.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/RabbitMQ.webp
---
## 1. Introduction

Previous article ([A great tool for dealing with traffic spikes - messaging middleware](http://mp.weixin.qq.com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247485140&idx=1&sn=) 62aa2a762363cc8c704fc427ef50a2b6&chksm=ecac52dddbdbdbcb3f2aa1e0178e17acfa0357b3f945e15caf92f62a3cfbce4c28b3a143250c&scene=21# wechat_redirect)), we have introduced the use of message middleware, mainly used as: decoupling, peak shaving, asynchronous communication, application decoupling, and introduces the industry's commonly used several kinds of message middleware, advantages and disadvantages of the comparison and the use of scenarios.

In today's article, let's talk about `RabbitMQ`, which is the earliest messaging middleware used in the work of small ❤, mainly used for asynchronous consumption of large amounts of data.

## 2. RabbitMQ

## 2.1 Core Components

RabbitMQ is an open source messaging middleware that implements the Advanced Message Queuing Protocol (AMQP) and provides a variety of important components to support the production, transport, and consumption of messages.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/ba450ee6b26c44ae9c10e5030bfd0ee7%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

1. **Producer:** The producer is the sender of the message and is responsible for publishing the message to the RabbitMQ server. The message can contain any content, such as tasks, logs, notifications, etc. 2.
2. **Channel:** The channel used for pushing and receiving messages. 3.
3. **Exchange:** A switch is a relay station for messages, it receives messages from producers and routes them to one or more queues. Different types of switches, such as `fanout&#xFF0C;direct&#xFF0C;topic&#xFF0C;headers`, support different routing rules.
4. **Queue (queue):** Queue is a buffer for messages, messages are stored in the queue before being sent to the consumer, the consumer gets the message from the queue and processes it.
5. **Consumer:** A consumer is the receiver of a message, it gets the message from the queue and processes it. Consumers can be multiple and they can run on different applications or servers.

### 2.2 Workflow

The way RabbitMQ works is based on collaboration between producers, switches and queues. It is a simple messaging process:

1. A queue is bound (`Binding`) to the switch, which defines the routing rules for the message;
2. the producer posts the message to the switch, which routes the message to one or more queues based on the binding rules;
3. consumers fetch messages from the queues and process them.

This model is highly flexible and can easily handle a large number of messages while ensuring reliable delivery.

### 2.3 Features

When it comes to messaging middleware, the first thing that comes to mind is `Kafka`, but `RabbitMQ` is also the preferred choice of many financial or Internet companies to build reliable, scalable and high-performance systems.

Why is this?

It starts with the characteristics of RabbitMQ, which are twofold: one is power and the other is reliability!

RabbitMQ focuses on message reliability and flexibility, suitable for task queuing and messaging. RabbitMQ focuses on message reliability and flexibility, and is suitable for task queuing and messaging. Kafka is a distributed streaming platform that focuses on log storage and data distribution.

** Sequential consumption is also a type of reliability, RabbitMQ can use a single queue or multiple single queues to ensure sequential consumption. **

In addition to this, RabbitMQ provides persistent queues and messages to ensure that messages are not lost if the RabbitMQ server goes down. Additionally, producers can use the publish acknowledgement mechanism to confirm that a message has been received.

RabbitMQ relative to kafka reliability is better, the data is less likely to be lost, which for some data-sensitive business, obviously more suitable for the former.

And, RabbitMQ native support ** dead letter queue **, you can better deal with unfinished business messages, as well as the implementation of ** delayed queue ** and other features, the next we introduce one by one.

## 3. Guaranteed sequential consumption

RabbitMQ provides several queue models to guarantee sequential consumption of messages. This is important for certain applications such as order processing, payments, and inventory management.

#### Scenarios for misordered message consumption

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/8ddf9fc781934910944aa265c8b80a1c%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

As shown in the above figure, there are three business messages with `delete add update` operations, but the `Consumer` does not consume them in order, and they are eventually stored in the order of `Add Update Delete;`, and data misordering occurs.

RabbitMQ's solution to the problem of message ordering is to ensure it in three stages.

#### 1. Sending messages: into the queue

When messages are sent, business is needed to ensure orderliness, that is, to ensure that the order in which producers enter the queue is ordered.

In distributed scenarios if it is difficult to ensure that the order of each server into the queue, you can add distributed locks to solve the problem. Or in the business producer's message with `Message Increment ID`, as well as the timestamp of the message generated.

#### 2. Messages in the queue

In RabbitMQ, messages are stored in a queue, and messages in the same queue are `First In First Out (FIFO)`, which **RabbitMQ helps us to ensure the order**.

RabbitMQ can't guarantee the order of messages in different queues, just as we can't guarantee that messages in different queues will be served before those in other queues, just as we can't guarantee that messages in different queues will be served before those in other queues.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/3a474f6a70834d74b98354f3acf19507%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

#### 3. Consumption messages: out of queue

In general, the sequential consumption after queue out is left to the consumer to guarantee. By guaranteeing the order of consumption, we also usually mean the order in which consumers consume messages.

** With multiple consumers, it is usually not possible to guarantee message order. **

This is equivalent to the case where we are in a queue for food, and there are multiple aunties who serve food, but each auntie does not serve food at the same speed, which corresponds to the different consumption capabilities of our consumers.

So, in order to ensure message ordering, we can use only one consumer to receive business messages.

It's as if there is only one aunty who is cooking, so if she comes early, she will be able to cook earlier. But obviously, this is not very efficient, so we need to weigh the pros and cons when using it: **see if the business needs sequentiality more, or if it needs consumption efficiency more**.

### Priority queues

Another roundabout way to ensure sequential consumption is to use a Priority Queue.

After RabbitMQ 3.5, ** Priority Queue comes into effect when the number of consumers is low and if the server detects that a consumer is not able to consume messages in a timely manner. **

There are two specific prioritization strategies:

1. set the priority of the queue
2. set the priority of the message

When declaring a queue, we can set the maximum priority of the queue via the `x-max-priority` attribute, or set the priority of the message from 1 to 10 via the `Priority` attribute.

The Golang implementation code is as follows:

```go

props := make(map[string]interface{})

props["x-max-priority"] = 10

ch.Publish(
   "tizi365",     
   "", 
   false,
   false,
   amqp.Publishing{
       Priority:5, 
       DeliveryMode:2,  
       ContentType: "text/plain",
       Body:       []byte(body),
  })
```

When priority queue consumption is in effect, ** will first consume the high priority messages in the high priority queue, so as to realize the sequential consumption **.

However, it should be noted that the conditions for priority queue triggering are relatively harsh, and it is best not to use it in cases where the order of business messages needs to be strictly guaranteed!

## 4. Dead Message Queues

In RabbitMQ, when a message becomes dead in the queue (Messages that consumers can't process properly), it will be re cast to a switch (i.e., a dead letter switch), ** the consumption queue bound to the dead letter switch is the dead letter queue**.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/0973eb9589e7416a8295207c0f041d09%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

### Generation of dead letters

The following conditions need to be met for a dead message to be generated:

1. the message has been manually rejected by the consumer and the `requeue` policy is False;
2. the message has expired (TTL). 3. the queue has reached its maximum length and the queue is not full;
3. the queue has reached its maximum length and the message cannot fit.

### Steps for handling dead messages

When a dead message is generated, if we define a `Dead letter switch;` (which is actually an ordinary switch, just used to handle dead messages, so it is called dead message switch), and then bind a queue on the dead message switch (called `dead letter queue`).

Finally, if there is a consumer listening to the dead letter queue, dead letter messages are handled as normal business messages, from the switch to the queue, and then consumed normally by `Dead message queue + message expiration` (the consumer listening to the dead letter queue).

## 5. Delayed Queues

RabbitMQ itself does not support delayed queues, but we can use the RabbitMQ plugin `rabbitmq-delayed-message-exchange`, or use `Dead message queue + message expiration; to get a delayed queue. ` are realized.

### 5.1 Application Scenarios

When we shop in e-commerce, or buy tickets in 12306, we will probably encounter such a scenario: each time we place an order, there is a period of product lock time in the middle of the order to pay for the order, and the order will be closed if the order is not paid for after the time has passed **.

The state transition diagram is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/5312f7a0d5f54aa583ad9928e9bf56a6%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

### 5.2 Plugin Implementation

#### 1. Install the plugin

`Github` address:

```ruby
https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases
```

Download the `rabbitmq_delayed_message_exchange-3.8.9-0199d11c.ez` file from the github release page under assets, and place the file in the rabbitmq plugin directory (plugins directory).

> Tip: The version number may be different from this tutorial, if your rabbitmq is the latest version, just choose the latest version of the plugin.

#### 2. Activate the plugin

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

#### 3. Define the exchange

Set the custom switch attributes to support sending delayed messages via `x-delayed-type`:

```go
   props := make(map[string]interface{})

   props["x-delayed-type"] = "direct"

   err = ch.ExchangeDeclare(
       "delay.queue",   
       "fanout", 
       true,     
       false,    
       false,
       false,
       props,      
  )
```

#### 4. Send delayed messages

With the message header (x-delay), set the message delay time.

```go
       msgHeaders := make(map[string]interface{})

       msgHeaders["x-delay"] = 6000

       err = ch.Publish(
           "delay.queue",     
           "", 
           false,
           false,
           amqp.Publishing{
               Headers:msgHeaders, 
               ContentType: "text/plain",
               Body:       []byte(body),
          })
```

### 5.3 Dead Message Queue + Message Expiration Scheme

The core idea of this scheme is to create dead message switches, queues and consumers to listen for dead messages.

Then create timed expired messages, for example, if the order payment time is 30min, set the `TTL` of the message to 30min, ** put the message into a queue with no consumers to consume it, and when the message expires, it will become a dead message. **

The dead message is resent to the dead message switch, then we consume the message in the dead message queue and determine if the item has been paid for based on the item ID.

If it has not been paid, the order is canceled and the order status is changed to `&#x5F85;&#x4E0B;&#x5355;`. If it has been paid, modify the item status to `&#x5DF2;&#x5B8C;&#x6210;` and drop this dead letter message.

## 5. Summary

RabbitMQ is a powerful messaging middleware that plays a key role in many Internet applications, such as the Huawei Camera SDK's surveillance image data reporting, asynchronous consumption in most e-commerce systems, and so on.

I hope that today's article can help you better understand RabbitMQ and use it to build a reliable messaging system in your work, the next article ❤ will bring the core workflow of Kafka, the underlying principles and common interview questions, stay tuned!


