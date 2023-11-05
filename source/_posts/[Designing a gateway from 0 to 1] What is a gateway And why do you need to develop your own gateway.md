---
title: Designing a gateway from 0 to 1 What is a gateway And why do you need to develop your own gateway
date: 2023-11-05 11:12:00
categories: 
  - java
tags: 
  - Self-developed
  - gateway
  - development
  - source
  - big factory
  - thinking
  - design
  - process
description: Self-developed a gateway that helped me successfully land a big factory. This is a complete set of my complete design out of a gateway from 0 to 1, the information contains the thinking process, flow charts, source code and other kinds of information.
cover: https://s2.loli.net/2023/11/05/vCi8BjrRVwQ27uW.webp
---
Along the way a lot of epiphanies, right, the first internship in January this year, in October this year into the byte, from no name, to the whole network 3w + fans, there have been dark moments, but when in the trough, how to go up, I hope to see the readers of this article have the opportunity to enter the company of their dreams.

![](https://s2.loli.net/2023/11/05/VLNKsy8tlAWuSdF.webp) This article, as the first article of my gateway, does not involve any code, but just mentions the role of the gateway and why I want to develop my own gateway, and the subsequent articles will be updated continuously. If you need, please watch the demo effect video to get the corresponding full information.

[Link to demo video](https://link.juejin.cn?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1eC4y1n73c%2F%3Fvd_source%) 3D1d4d63e205b3ad352b4771f87295d16d%23reply747752344 "https://www.bilibili.com/video/BV1eC4y1n73c/?vd_source= 1d4d63e205b3ad352b4771f87295d16d#reply747752344")

# What is a Gateway?

A Gateway is an important device in a computer network that is used to connect different networks, protocols or communication systems so that they can communicate and exchange data with each other. The main function of a gateway is to pass data between different networks and ensure that the data is routed and transformed correctly so that devices in different networks can understand and communicate with each other. Following are some of the important concepts and functions of a gateway:

Connecting different networks: Gateways are usually used to connect different network types, such as Local Area Network (LAN) and Wide Area Network (WAN), Ethernet and wireless networks, IPv4 and IPv6, etc., in order to transfer data and communicate between different networks.

Data Conversion and Protocol Translation: Gateways can convert data from one network protocol to another to ensure that devices in different networks can communicate with each other. This usually involves converting data from the format of one protocol to that of another, such as converting data from the TCP/IP protocol to the HTTP protocol.

Security and Fusion Flow Limiting: A gateway can also be used to enhance network security. It can perform firewall functions, monitor data traffic, filter malicious traffic, and enforce access control policies to protect the network from unauthorized access and network attacks.

Data routing: A gateway can determine the best path for packets to ensure that data is delivered efficiently from source to destination. This usually involves looking at the destination address and selecting the appropriate output interface or next leap point to transmit the data.

Protocol translation: When communicating between different networks, it may be necessary to perform protocol translation to ensure that data is interpreted correctly. Gateways can perform such tasks to ensure that devices on different networks can communicate with each other.

# Gateway types

**RESTful API Gateway**

* RESTful API gateways are typically used to manage and provide access to RESTful APIs.
* RESTful APIs are based on the HTTP protocol and are typically used for request-response model communication for operations such as fetching, creating, updating, and deleting resources.
* RESTful API gateways provide routing, authentication, authorization, access control, logging, and monitoring to ensure the security and availability of the API.
* RESTful API gateways typically do not support real-time two-way communication because HTTP is connectionless and does not apply to persistent connections.

**WebSocket API Gateway

* The WebSocket API Gateway is specifically designed to handle real-time bi-directional communication with the WebSocket protocol.
* The WebSocket protocol allows a long term bi-directional connection to be established between a client and a server in order to transfer data in real time for scenarios such as real-time chat, online gaming, real-time notifications, etc. * The WebSocket API Gateway is designed to handle real-time two-way communication.
* WebSocket API Gateway provides WebSocket connection management, routing, load balancing, protocol upgrading and messaging.
* WebSocket API Gateway supports persistent connections without creating new connections between each request like RESTful API.

* # Pros and Cons of Gateways

  ** Advantages**

  * Simplifies client-side development: Gateways allow client applications to communicate with back-end services without having to concern themselves with complex network protocols and details. The client only needs to communicate with the gateway and does not have to interact directly with the back-end service. This greatly simplifies the work of the client and reduces the amount of technical details developers need to deal with. The client no longer needs to know the specific address or API endpoint of the back-end service. It only needs to know how to communicate with the gateway, which is responsible for routing requests to the appropriate back-end service.
  * :: Reduced coupling: Using a gateway as an intermediate layer separates different parts of the system and reduces the tight coupling between them. This helps to improve the maintainability and scalability of the system. Instead of spreading these functions across each individual service, gateways can handle shared functions such as authentication, authorization, and data conversion. This reduces redundancy and duplication of functionality and makes it easier for developers to maintain and update these functions.
  * :: Reduced duplication of wheel-building: By abstracting shared functionality into a gateway, developers can focus more of their efforts on the development of business logic. They don't have to implement the same functionality, such as authentication or request validation, over and over again for each service. The gateway can also provide features such as performance optimization, load balancing, and caching, which reduces the burden on developers and allows them to focus more on implementing the business logic.

  **Disadvantages**

  * Possible performance bottlenecks Microservices architectures are designed to increase flexibility and scalability by breaking large applications into small, relatively independent services. However, when a gateway is used, it can become a performance bottleneck point, especially in high load situations. Gateways typically have to handle a large number of requests and responses, including tasks such as request routing, authentication, authorization, data transformation, and so on. If the gateway is not properly optimized and scaled, it can limit the performance of the entire system.
  * Request Blocking Problem The microservices architecture encourages the use of asynchronous communication and non-blocking IO so that services can process requests in parallel and improve response performance. However, some gateways may rely on synchronous communication models, which can lead to performance degradation If the gateway performs operations that need to wait for a response from an external service and does not use an asynchronous model, the gateway may become a blocking point in the entire request-response chain, affecting the performance and response time of the system.
  * High coupling If the coupling between the gateway and the microservices is too high, it may limit the system's scalability and ability to be deployed independently. For example, if multiple microservices are highly coupled to a particular gateway, changing that gateway may require modifying multiple services, which violates the principles of microservices architecture. High coupling may also lead to collaboration issues between development teams, as they must coordinate additional changes to accommodate changes to the gateway.

# What are the current gateway solutions?

**Nginx:** C/Lua based Benefits: High performance for large scale applications and high traffic loads. Strong extensibility with support for Lua scripts and custom plugins. Versatile, not limited to API gateways, but can also be used as a reverse proxy and load balancer. Cons: Configuration and management requires technical expertise, not user-friendly enough. Advanced features require extensions such as OpenResty

**Kong:** Based on Lua Pros: Easy to extend, provides a rich plugin ecosystem. Supports microservice architecture for complex deployments. Strong integration, can be integrated with other services such as databases and message queues. Cons: Requires management and maintenance, especially in large-scale environments. Some advanced features may require writing custom plugins.

**Apigee:** Pros: Cloud-hosted, eliminating the need to manage your own infrastructure. Provides comprehensive API management capabilities, including analytics, monitoring, and security. Highly scalable for large-scale applications. Cons: Some learning curve, requires knowledge of Apigee configuration and setup. Can be more expensive, especially for small-scale projects.

**AWS API Gateway:** Pros: Seamlessly integrates with the AWS ecosystem, providing high availability and scalability. Simplifies API deployment and management. Can utilize other AWS services such as Lambda functions. Cons: Locked dependencies to AWS cloud, not applicable to hybrid cloud environments. May require knowledge of AWS-specific configurations and settings.

**Istio:** Pros: Provides rich traffic management and security features for microservices. Supports multi-cloud and hybrid cloud environments, not limited to specific cloud providers. Integrates with monitoring tools such as Prometheus and Jaeger. Disadvantages: Higher complexity, takes time to learn and deploy. May require extensive configuration, especially in large-scale environments.

**Spring Cloud Gateway:** Pros: Integrates with the Spring Cloud ecosystem and supports Spring Boot applications. Lightweight and easy to deploy and manage. Supports dynamic routing and filters. Cons: Relatively low feature set, suitable for small to medium sized applications. May not be as suitable as other gateways for complex API management needs.

**Traefik:** Pros: Designed for containerized applications and microservices, with support for Docker and Kubernetes. Auto-discovery of back-end services, dynamic configuration. Lightweight, easy to deploy and manage. Cons: Relatively few features, suitable for smaller applications. May require additional plugins to support advanced features.

**HAProxy:** Pros: High-performance load balancer for high-load environments. Simple to configure and manage. Supports TCP and HTTP load balancing. Cons: Fewer advanced features, not suitable for complex API management needs. Lack of API gateway specific features such as authentication and authorization.

# Why should I develop my own Gateway?

Currently more popular gateways are SpringCloud Gateway, SpringCloud Zuul, they are better gateways. However, these gateways are developed based on Java language, and if we want to use this kind of gateway we must know Java, and if the company's business is mostly go, such as the byte I work for, at present, go is used more, so using Java as a gateway is not very suitable. Second is to use these mature frameworks, because of its face to face, so it means that when we only need to use the gateway in part of the functionality of the direct use of these frameworks will make the business project is too large. Therefore, it is better to consider developing our own gateway, where we only need to implement the more important features that we need, and developing our own gateway also provides us with greater customization capabilities of the gateway.

# What do I need to look for in a homegrown gateway?

1: Scalability. We need to make sure that our gateway is highly scalable, because we need to consider not only the current business requirements, but also the future business requirements. 2: Reasonable architecture configuration. My self-developed gateway will be developed using the domain-driven model DDD. If you don't know about DDD, you can learn about its advantages and disadvantages. 3: interface compatibility: we know that the gateway will generally use the configuration center or registration center, the current mainstream Apollo and Nacos, so we also need to provide strong compatibility, so that in the different configuration centers for switching time to time. 4: three high design: as the first hurdle of the project, the gateway needs to carry far more requests than the background specific services, so the performance of the gateway is a very in-depth consideration and design aspects. Involving JVM tuning, code performance optimization and so on.

