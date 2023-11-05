---
title: [Designing a Gateway from 0 to 1] Filter Chain Implementation - Route Forwarding Filters
date: 2023-11-05 17:12:00
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
[Link to effects demo](https://www.bilibili.com/video/BV1eC4y1n73c/?vd_source=1d4d63e205b3ad352b4771f87295d16d#reply747752344)

# Analyze

As we know, route forwarding is the last operation to be performed after all the filtering logic is processed by the gateway, it is responsible for forwarding our request to a specified backend service instance, here we refer to the implementation of SpringCloudGateway to simulate a route forwarding filter.

A route forwarding filter in Spring Cloud is a component of the Spring Cloud Gateway (a microservices gateway) that is used to perform filtering and forwarding operations on incoming HTTP requests. These filters allow us to modify, validate, log, etc. requests before they reach the target service. Here are the main roles of route forwarding filters and why they are needed:

* Request modification and redirection: route forwarding filters allow you to modify various parts of the request, including the request header, request body, request parameters, etc., to suit the requirements of the target service. You can add, delete, or modify request information, and even redirect the request to a different target service, enabling dynamic request routing.
* :: Security: With route forwarding filters, you can add security-related features, such as authentication and authorization, to ensure that only authorized users can access certain services. This helps protect individual services in a microservice architecture from unauthorized access.
* Caching: You can use filters to enable caching of requests and responses to lighten the load on the target service, improve performance, and reduce response times. This is useful for services that handle a large number of requests.
* Logging and monitoring: Route forwarding filters can also be used to log information about requests and responses for monitoring and troubleshooting purposes. You can add logging and metrics collection to the filter to understand the performance and status of requests.
* Traffic control: With route forwarding filters, you can implement traffic control and flow limiting to prevent a particular service from being overwhelmed by too many requests. This helps maintain service availability and performance.
* Request forwarding and load balancing: The most common uses are to forward requests to multiple destination services on the backend and to implement load balancing policies to ensure that requests are evenly distributed to different service instances.

# Code Implementation

A lot has been said above about why we should use route forwarding filters, so let's now analyze the implementation of route forwarding filters. One thing we have analyzed from our early architectural design diagrams is that we are using an asynchronous way to send our http requests, and here I am using Netty with AsyncHttpClient to implement the asynchronous IO communication functionality. If you are interested in this piece you can search for yourself: Netty asynchronous IO communication model knowledge and Netty with AsyncHttpClient use.

```c
package blossom.project.core.netty;

import blossom.project.core.Config;
import blossom.project.core.LifeCycle;
import blossom.project.core.helper.AsyncHttpHelper;
import io.netty.buffer.PooledByteBufAllocator;
import io.netty.channel.EventLoopGroup;
import lombok.extern.slf4j.Slf4j;
import org.asynchttpclient.AsyncHttpClient;
import org.asynchttpclient.DefaultAsyncHttpClient;
import org.asynchttpclient.DefaultAsyncHttpClientConfig;
import java.io.IOException;

@Slf4j
public class NettyHttpClient implements LifeCycle {
    private final Config config;

    private final EventLoopGroup eventLoopGroupWoker;

    private AsyncHttpClient asyncHttpClient;

    public NettyHttpClient(Config config, EventLoopGroup eventLoopGroupWoker) {
        this.config = config;
        this.eventLoopGroupWoker = eventLoopGroupWoker;
        init();
    }

    @Override
    public void init() {
        DefaultAsyncHttpClientConfig.Builder builder = new DefaultAsyncHttpClientConfig.Builder()
                .setEventLoopGroup(eventLoopGroupWoker)
                .setConnectTimeout(config.getHttpConnectTimeout())
                .setRequestTimeout(config.getHttpRequestTimeout())
                .setMaxRedirects(config.getHttpMaxRequestRetry())
                .setAllocator(PooledByteBufAllocator.DEFAULT)
                .setCompressionEnforced(true)
                .setMaxConnections(config.getHttpMaxConnections())
                .setMaxConnectionsPerHost(config.getHttpConnectionsPerHost())
                .setPooledConnectionIdleTimeout(config.getHttpPooledConnectionIdleTimeout());
        this.asyncHttpClient = new DefaultAsyncHttpClient(builder.build());
    }

    @Override
    public void start() {
        AsyncHttpHelper.getInstance().initialized(asyncHttpClient);
    }

    @Override
    public void shutdown() {
        if (asyncHttpClient != null) {
            try {
                this.asyncHttpClient.close();
            } catch (IOException e) {
                log.error("NettyHttpClient shutdown error", e);
            }
        }
    }
}

```

Directly look at the code is actually that I integrated these two tools, the realization of the use of Netty to complete the function of asynchronous communication. And initialized our AsyncHttpClient asynchronous http request sending tool. And we have finished packaging this tool, to realize the response to the request is relatively easy, we only need to write our response content back to the response body of our request can be.

```c
package blossom.project.core.filter.router;

import blossom.project.common.enums.ResponseCode;
import blossom.project.common.exception.ConnectException;
import blossom.project.common.exception.ResponseException;
import blossom.project.core.ConfigLoader;
import blossom.project.core.context.GatewayContext;
import blossom.project.core.filter.Filter;
import blossom.project.core.filter.FilterAspect;
import blossom.project.core.helper.AsyncHttpHelper;
import blossom.project.core.helper.ResponseHelper;
import blossom.project.core.response.GatewayResponse;
import lombok.extern.slf4j.Slf4j;
import org.asynchttpclient.Request;
import org.asynchttpclient.Response;

import java.util.Objects;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeoutException;

import static blossom.project.common.constant.FilterConst.*;

@Slf4j
@FilterAspect(id=ROUTER_FILTER_ID,
        name = ROUTER_FILTER_NAME,
        order = ROUTER_FILTER_ORDER)
public class RouterFilter implements Filter {
    @Override
    public void doFilter(GatewayContext gatewayContext) throws Exception {
        Request request = gatewayContext.getRequest().build();
        CompletableFuture future = AsyncHttpHelper.getInstance().executeRequest(request);

        boolean whenComplete = ConfigLoader.getConfig().isWhenComplete();

        if (whenComplete) {
            future.whenComplete((response, throwable) -> {
                complete(request, response, throwable, gatewayContext);
            });
        } else {
            future.whenCompleteAsync((response, throwable) -> {
                complete(request, response, throwable, gatewayContext);
            });
        }
    }

    private void complete(Request request,
                          Response response,
                          Throwable throwable,
                          GatewayContext gatewayContext) {
        gatewayContext.releaseRequest();

        try {
            if (Objects.nonNull(throwable)) {
                String url = request.getUrl();
                if (throwable instanceof TimeoutException) {
                    log.warn("complete time out {}", url);
                    gatewayContext.setThrowable(new ResponseException(ResponseCode.REQUEST_TIMEOUT));
                    gatewayContext.setResponse(GatewayResponse.buildGatewayResponse(ResponseCode.REQUEST_TIMEOUT));
                } else {
                    gatewayContext.setThrowable(new ConnectException(throwable,
                            gatewayContext.getUniqueId(),
                            url, ResponseCode.HTTP_RESPONSE_ERROR));
                    gatewayContext.setResponse(GatewayResponse.buildGatewayResponse(ResponseCode.HTTP_RESPONSE_ERROR));
                }
            } else {
                gatewayContext.setResponse(GatewayResponse.buildGatewayResponse(response));
            }
        } catch (Throwable t) {
            gatewayContext.setThrowable(new ResponseException(ResponseCode.INTERNAL_ERROR));
            gatewayContext.setResponse(GatewayResponse.buildGatewayResponse(ResponseCode.INTERNAL_ERROR));
            log.error("complete error", t);
        } finally {
            gatewayContext.written();
            ResponseHelper.writeResponse(gatewayContext);
        }
    }
}

```

The above code is still relatively easy to analyze. We first analyze the doFilter method, first of all, this method will build our request, and then use our http tools packaged asynchronous execution of the request, here we use the CompletableFuture tool to asynchronously receive the results of the request, when the request is executed after the implementation of our callback method. By default, we use the single asynchronous mode, which executes the complete method when we receive a response to a request. If the response encounters an exception, then we will catch the exception and report the error, if not, we will normally go finally to write back the response data to the front-end. At this point, we can successfully forward our request to the backend service.

# Effectiveness with Load Balancing Filter

In the previous section I have implemented both random and polling load balancing filters, so let's demonstrate the effect here. First we start two instances of the backend service and make sure they have successfully registered to the registry.![](https://s2.loli.net/2023/11/05/KdTEvySs7z5a6pN.webp)

Also configure the configuration file for our gateway gateway.![](https://s2.loli.net/2023/11/05/Y3n6md1Hf9ZP5Bt.webp) After that, we configure our request header with the name and version of the backend service to be requested![](https://s2.loli.net/2023/11/05/elQwYh5VZIKxH9s.webp) The service is sent multiple times, and here we chose polling load balancing, so you can see that the service is distributed evenly to the two back-end services at the end.

![](https://s2.loli.net/2023/11/05/nJAepu4vFT9S1QW.webp) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4f9f7a9d6234f81af239977c8b9df41~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1740&h=210&s=78635&e=png&b=fbfbfb)
