---
title: Come on, get a taxi system.
date: 2023-11-06 07:04:00
categories: 
  - Backend
tags: 
  - Architecture
  - Backend
  - Interviews
  - resume
  - development
  - framework
  - network
  - dropshipping
description: You've used dropshipping, right? See you wrote in your resume that you know architecture design right, if you were asked to design an online car system, what aspects would you consider
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/47d20c8d266040b391ef2d32f0d6f7d9%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp
---
**Catalog**

1. Introduction
2. Netiquette System
   3.

  3. Requirements Design
  4. Outline Design
  5. Detailed Design
  6. Experience Optimization
7. Summary

# 1. Introduction

## 1.1 Typhoon

Last week, Shenzhen was affected by Typhoon Sura, and from 12:00 on September 1, the city started the first-level emergency response for typhoon and flood control.

The specific impact on the workers in Shenzhen, the day from 4 p.m. in the city to implement the "five stops": stopping work, stopping business, stopping the city, the day has been closed, after 7 p.m. stopping transportation.

Since the market was closed at 4 pm, most companies left work early. Some of them left work in a hurry, like this:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/9b42790c20bd47ea8eda2dd91b298b73%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

There are early dismissals, like this:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/713603fe04df40a791d496bee1fbc2d5%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

And then there are those like us who have to telecommute from home:

## 1.2 Crashing a taxi

It's around 4pm and the buses and subways are packed.

So I thought I'd take a cab home when I was about to leave work (and work from home), but then I opened the drop:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/fc6f19e60ac64366823455031e6ad011%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

There were 142 people in line, and the size and length of the line made my heart skip a beat.

Based on historical experience, the response time for calling the car on a rainy day has to be pushed back by about half an hour. What's more, it was typhoon weather!

DDT ah DDT, you can not prepare in advance, this waiting time, will make you lose a lot of order share.

But on the other hand, this kind of emergency warning, can not completely blame the taxi platform, after all, the vehicle scheduling is also need a certain amount of time. In this kind of everyone scrambling to escape (bushi time, around the vehicle estimate is not quite enough.

### Roll up

It was a long wait, so I went back to the office to continue reading technical articles. At this point, it occurred to me that after this emergency vehicle dispatch, if I were a development engineer at DDT, how would I need to handle this situation?

If the interviewer from DDT was in front of me, how would he consider the candidate's technical depth and product thinking?

# 2. Design a "net car system"

Interviewer: "You've used DDT before, right? In your resume, you wrote that you know architecture design, right? If you are asked to design an online car system, what aspects will you consider?"

## 2.1 Requirements Analysis

The core function of a network car system (such as DDT) is to send the passenger's taxi order to the attached network car driver, the driver receives the order, picks up the passenger at the boarding point, and the passenger gets off the car to complete the order.

Among them, the driver takes a share (ranging from 70%-80%) through a percentage agreed upon by the platform, and the passenger can open a secret-free payment based on the credit value of a third-party platform (e.g., Alipay) to automatically pay for the order after getting off the bus. The use case diagram is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/47d20c8d266040b391ef2d32f0d6f7d9%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Both passengers and drivers have a registration and login function, which belongs to the passenger user module and driver user module. The other core functions of the online taxi system are passenger taxi, order allocation, and driver delivery.

## 2.2 Outline Design

Net taxi system is a model of Internet + shared resources, the purpose is to combine vehicles and passengers, a way to save the existing resources, usually a net taxi to multiple users.

So for passengers and drivers, their interaction with the system is different. For example, a person may only take a taxi once a day, while a driver has to make several trips a day.

Therefore, we need to develop two APP applications, respectively, for passengers and drivers to taxi and take orders, the architecture diagram is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/64aa77b44ae24079a121b3846852df8e%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

### 1) Passenger perspective

As shown above: after registering as a user in the cell phone App, the passenger can choose the origin and destination to take a taxi.

The taxi request passes through the load balancing server, goes through a series of filters such as request forwarding, and then arrives at the HTTP gateway cluster, and then the gateway cluster carries out business verification and calls the corresponding micro-services.

For example, a passenger obtaining personal user information, collected address information, etc. on a cell phone can forward the request to **User System**. When they need to call a taxi, they will send the information of origin, destination, personal location, etc. to **Taxi System**.

### 2) Driver's view

As shown in the above figure: after the driver registers as a user in the cell phone App and starts to take orders, he/she opens the location information of his/her cell phone and sends his/her location information to the platform at regular intervals through the **TCP long connection**, and at the same time, receives the order messages issued by the platform.

> The Driver App uses TCP long connection because it has to send and receive system messages at regular intervals, if HTTP push is used:
> HTTP push: > On one hand, it has an impact on the real-time performance, on the other hand, it is not decent to re-establish the connection every time we communicate (it is resource-consuming). 

Driver App: sends the current location information to the platform every 3~5 seconds, including vehicle latitude and longitude, vehicle direction, etc. The TCP server cluster is equivalent to a gateway, which only provides access service to the App by means of TCP long connection, and the geolocation service is responsible for managing the driver's location information.

### 3) Order Receiving

The gateway cluster acts as the **registration center of the business system and is responsible for security filtering, business flow limitation, and request forwarding**.

The service consists of one independently deployed gateway server. When there are too many requests, the traffic pressure can be dispersed to different gateway servers through the load balancing server.

When a user takes a taxi, the request is called to one of the gateway servers through the load balancing server. The gateway will first call the order system to create a taxi order for the user (the order status is "created") and store it in the library.

Then the gateway server calls the taxi system, which encapsulates user information, user location, origin, destination and other data into a message packet, sends it to a message queue (e.g., RabbitMQ), and waits for the system to assign a driver for the user order.

### 4) Order allocation

The **Order Allocation System**, as a consumer of the message queue, listens for orders in the queue in real time. When a new order message is acquired, the order allocation system modifies the order status to "order allocation in progress" and stores it in the queue.

Then, the order allocation system sends the user information, user location, origin and destination to the **Order Push SDK**.

Then, the order push SDK calls the geolocation system to get the real-time location of the driver, and then combines it with the user's boarding point to select the most suitable driver for dispatching the order, and then sends the order message to the **Message Alert System**. At this point, the order distribution system modifies the order status to "Driver has taken the order" status.

The order message is pushed through the specialized message alert system, and the order is pushed to the cell phone App of the matched driver through TCP long connection.

### 5) Order Rejection and Order Grabbing

When the order push SDK assigns a driver, it will consider whether the driver's current order is completed or not. When the most suitable driver is assigned, the driver can also choose to "reject" the order according to his own situation, but the platform will record it to evaluate the driver's efficiency in taking orders.

In the taxi platform, if a driver refuses too many orders, he or she may reduce the weight score of the **allocated orders** for a subsequent period of time, affecting his or her own performance.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/9a4547025ada4cd09af073b873f8bd93%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

The order assignment logic can also be modified to allow additional drivers to grab orders, the specific implementation is:

When the order is created, the order push SDK will push the order message ** to the driver's App** within a certain geographic range, the driver within the range can grab the order after receiving the order message, and the order status will change to "dispatched" after the order grabbing is completed.

## 2.3 Detailed Design

For the detailed design of the taxi platform, we will focus on some core functions of the taxi system, such as: long connection management, address algorithm, experience optimization and so on.

### 1) Advantages of Long Connection

In addition to the HTTP short connection request commonly used on web pages, such as: Baidu search, enter a keyword to initiate a HTTP request, this is the most commonly used short connection.

However, large-scale APPs, especially those involving message push (such as QQ, WeChat, and Meituan), almost always build a complete set of **TCP long connection** channels.

A chart to see the advantages of long connections:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/995f39028c424c90b5cea254a235dbbb%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Image Credit: "Mobile Network Optimization Practices in MMT

With the above figure, we conclude. Compared with short connections, long connections have three advantages:

1. high connection success rate
2. low network latency
3. stable sending and receiving of messages, not easy to be lost

### 2) Long Connection Management

As mentioned earlier, the advantages of long connection are high real-time performance and stable sending and receiving of messages. In the taxi system, drivers need to send their location information regularly and receive order data in real-time, so the **Driver App adopts TCP long connection to access the **system.

Unlike the HTTP stateless connection, the TCP long connection is a stateful connection. The so-called statelessness means that each user request can be sent to a server at will, and the return from each server is the same, so the user does not care which server handles the request.

> Of course, now HTTP2.0 can also be a stateful long connection, we default to HTTP1.x here.

In order to ensure the transmission efficiency and real-time, the server and the user's cell phone app need to maintain the state of the long connection, i.e., **stateful** connection.

So every time a driver App reports information or pushes a message, it passes through a specific connection channel, and the connection channel for the driver App to receive messages and send messages is fixed.

Therefore, the TCP long connection on the driver's side needs to be managed specifically to handle the connection information between the Driver App and the server, and the architecture diagram is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/4a3533b0592e4fb69939c60096c56616%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

In order to ensure that every time a message is received and pushed, the corresponding channel can be found, we need to maintain a mapping relationship between the Driver App and the TCP server, which can be saved in Redis.

When the Driver App logs in for the first time, or disconnects from the server (e.g. the server is down, the user switches networks, the mobile app is closed in the background, etc.) and needs to be reconnected, the **Driver App will re-apply for a server connection through the user's Long Connection Management system** (the available address is stored in Zookeeper), and then refresh Redis's cache after connecting to the server over TCP.

### 3) Address Algorithm

When a passenger takes a taxi, the order push SDK will combine the driver's geographic location with an address algorithm to calculate the most suitable driver to dispatch the order.

Currently, cell phones generally collect latitude and longitude information. The longitude range is from 180 East to 180 West, and the latitude range is from 90 South to 90 North.

We set the west longitude to be negative and the south latitude to be negative, so the longitude range on the earth is [-180, 180] and the latitude range is [-90, 90]. If we take the prime meridian, the equator, as the boundary, the earth can be divided into 4 parts.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c599b35cb09f411186ae0cdb85642b16%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

According to this principle, we can first encode the two-dimensional spatial latitude and longitude into a string to uniquely identify the location information of the driver and passenger. Then use Redis' GeoHash algorithm to get all the driver information attached to the passenger.

The principle of the GeoHash algorithm is to ** convert the latitude and longitude of the passenger into an address-encoded string, indicating a rectangular area, through which the algorithm can quickly find all drivers in the same area **.

Its implementation uses a jump table data structure, the specific implementation is:

A piece of range of an urban area is used as the key of GeoHash, all the drivers within this urban area are stored into a jump table, and when a passenger's geographic location appears in this urban area, all the driver information within the range is obtained. Then further filter out the nearest driver information for dispatching.

### 4) Experience Optimization

#### 1. Distance Algorithm

As an online order dispatching, the effect of distributing orders through the distance algorithm will be relatively poor, because Redis calculates the spatial distance between two points, but the driver must drive along the road, and in the complex urban road conditions, maybe the spatial distance of a few tens of meters to drive more than ten minutes is not known.

Therefore, the subsequent need for a combination of driving distance (rather than spatial distance), the driver's head direction and the boarding point for path planning, to calculate the distance and time for each driver to reach the passenger in the region.

Further, if there are more than one passenger and driver in the area, the waiting time of all of them should be taken into account, so as to optimize the user experience, save the dispatch time, and improve the profit amount.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/297f3cab05374f94b706f099b0a99bee%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

#### 2. Order prioritization

If a taxi order is frequently canceled, it can be judged based on the driver's or passenger's behavior. After the judgment of responsibility, a reputation score is calculated for the passenger and driver, and the user is informed that the reputation score will affect the experience of the passenger and driver, and will be related to the priority of the dispatch order.

##### Driver Prioritization

Considering the driver's reputation score, the number of complaints, the number of orders received by the driver, etc., to assign different priority to drivers with different reputation scores.

##### Passenger Order Prioritization

According to the passenger's taxi time period, taxi distance, boarding point and other information, to make a user profile, in order to rationalize the arrangement of drivers, or appropriate killing (bushi.)

PS: Some bad taxi platforms are doing this 🐶 _ Even a taxi platform was reported to charge differently according to different cell phone systems_.

# 4. Summary

## 4.1 Development of Internet Ridesharing Platforms

Currently, the global online taxi market has reached hundreds of billions of dollars, and the main competitors include companies such as DDT, Uber, and Grab. In China, as the largest online car rental platform, DDT has already occupied the majority of the market share.

The core business logic of online dating is relatively simple, with the main stakeholders being the platform, drivers, vehicles, and consumers.

The platform respectively docking drivers, vehicles [non-essential, there are many drivers are with the car on duty] and passengers, through the effective matching of supply and demand to earn the money saved by the entire sharing economic chain.

Specifically manifested as: passengers and drivers respectively through the network contracting platform taxi and orders, the platform to provide technical support. Passengers pay for the taxi service, and the platform takes a cut (ranging from 10%-30%) from the transaction amount.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/1a6f91e71a04453fba0edf05eea79d13%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

According to the National Interactive Platform for Network Rental Vehicle Supervision and Information, as of the end of February 2023, a total of 303 online vehicle platform companies nationwide have obtained licenses to operate online vehicle platforms.

These platforms are partly **Netjob aggregation platforms** relying on Gaode Taxi, Baidu Maps, and Meituan Taxi; and partly **travel platforms** relying on DDT, Flower Pig, and T3.

## 4.2 Current Situation of Netiquette Platforms

With the unblocking of traveling, the net taxi platforms have come back to life.

However, because the entry threshold of some net car aggregation platforms is too low, more and more problems have been exposed in the past period of time. For example, the low compliance rate of vehicles and drivers, encountering safety accidents, generating liability disputes, and the difficulty of passengers' rights defense, and so on.

Due to its special model, it has been straying from the edge of the law due to its liability boundary problem with online car rental operators.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c9af693b1aa14971994aa1bb8acda2ad%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

However, as the regulation of online taxi aggregation platforms continue to fall, the country has traveled a certain regulatory regulations.

For example, a ride-hailing platform requires vehicles to keep the **communication records of drivers and passengers on file**, in addition to the online communication records of drivers and passengers must be preserved, but also a voice phone or in-vehicle recording conversion, to be kept for a period of time for inspection.

With these humane regulations and continuous technological innovations, the online taxi platform may continue to thrive for some time to come.

### Afterword

Interviewer: Well, specialized and red, all-round development! This young man is good, attention~
