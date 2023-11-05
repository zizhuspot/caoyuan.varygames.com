---
title: [Designing a gateway from 0 to 1] Design of Netty, a network communication framework
date: 2023-11-05 15:12:00
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
After completing the current chapter, the code effect is demonstrated as follows:![](https://s2.loli.net/2023/11/05/gdeDE2mivfHUwsO.webp) You can get the project source code and information in the video introduction [link to the effect demo](https://www.bilibili.com/video/BV1eC4y1n73c/?vd_source=1d4d63e205b3ad352b4771f87295d16d#reply747752344)


This request will be redirected to our backend localhost:8080/http-demo/ping address. This is the address of our backend service.


# Netty architecture


The [Netty website](https://netty.io/index.html) philosophy helps us build scalable, high-performance, and maintainable web applications. Here are some of the core concepts of Netty:


Channel: A channel is an abstraction of data communication that can represent underlying network connections such as sockets.Netty provides several types of channels for different transport protocols (e.g., NIO, OIO, local transport).


EventLoop: EventLoop is a loop for handling events and is a core component in Netty. Each Channel has an EventLoop associated with it, which is responsible for handling all the events of the Channel, such as receiving data, processing data, sending data, etc.


ChannelHandler: ChannelHandler is a component that handles channel events and can be used to implement protocol encoding and decoding, business logic processing, etc. A ChannelHandler chain is a sequence of operations performed on a channel.


Bootstrap: Bootstrap is a tool used to start and configure a web application, which is usually used to create and connect to a Channel. it helps to set the configuration of EventLoopGroup, Channel type, ChannelHandler, etc.


ChannelPipeline: ChannelPipeline is a data structure used to maintain and process the ChannelHandler chain. It is used to specify the order in which events are processed and the direction in which they flow in order to perform specific operations on the Channel.


ByteBuffer: The ByteBuffer is the basic data structure for processing data in Netty. It provides read and write operations on the data, and supports the zero-copy feature.


Codecs: Codecs are components used to convert raw data into protocol-specific messages and messages into bytes of data; Netty provides a number of built-in codecs and also supports custom codecs.


Promise: Promise is an abstraction for handling the results of asynchronous operations. It can be used to monitor and get the result or state of an asynchronous operation.


Future: A Future is also an abstraction for handling asynchronous operations and represents an operation that has not yet completed; asynchronous operations in Netty typically return a Future that can be retrieved when the operation completes.


ByteBuf: ByteBuf is a byte data container in Netty that provides efficient read and write operations and supports reference counting for improved performance and reduced memory overhead. We will develop based on Netty Reactor working architecture, below is the architecture diagram (from web):![](https://s2.loli.net/2023/11/05/v9pUoj21SgOkl6N.webp)

# Implementing NettyHttpServer

Above a brief understanding of the Netty architecture, in this point I will implement a client based on Netty. There are roughly the following steps that we need to implement to do: 1: encapsulate properties: 2: implementation of the construction and init methods 3: epoll optimization 4: implementation of the start method 5: implementation of the shutdown method

In this part we need to Write the finished code is as follows:

```java
package blossom.gateway.core.netty;

import blossom.gateway.common.utils.SystemUtil;
import blossom.gateway.core.LifeCycle;
import blossom.gateway.core.config.GatewayConfig;
import blossom.gateway.core.rule.Rule;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.epoll.Epoll;
import io.netty.channel.epoll.EpollEventLoopGroup;
import io.netty.channel.epoll.EpollServerDomainSocketChannel;
import io.netty.channel.epoll.EpollServerSocketChannel;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.util.concurrent.DefaultThreadFactory;
import lombok.extern.slf4j.Slf4j;

public class NettyHttpServer implements LifeCycle {
    private final GatewayConfig config;

    private ServerBootstrap serverBootstrap;

    private EventLoopGroup eventLoopGroupBoss;

    private EventLoopGroup eventLoopGroupWorker;

    public NettyHttpServer(GatewayConfig config) {
        this.config = config;
        init();
    }

    public void init() {

        if (canUserEpoll()) {
            this.serverBootstrap = new ServerBootstrap();
            this.eventLoopGroupBoss = new EpollEventLoopGroup(config.getEventLoopGroupBossNum(),
                    new DefaultThreadFactory("netty-boss-nio"));
            this.eventLoopGroupWorker = new EpollEventLoopGroup(config.getEventLoopGroupWorkerNum(),
                    new DefaultThreadFactory("netty-worker-nio"));
        } else {
            this.serverBootstrap = new ServerBootstrap();
            this.eventLoopGroupBoss = new NioEventLoopGroup(config.getEventLoopGroupBossNum(),
                    new DefaultThreadFactory("netty-boss-nio"));
            this.eventLoopGroupWorker = new NioEventLoopGroup(config.getEventLoopGroupWorkerNum(),
                    new DefaultThreadFactory("netty-worker-nio"));

        }
    }

    public boolean canUserEpoll() {
        return SystemUtil.isLinux() && Epoll.isAvailable();
    }

    public void start() {
        this.serverBootstrap.group(eventLoopGroupBoss,
                eventLoopGroupWorker)
                .channel(canUserEpoll() ?
                EpollServerSocketChannel.class : NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer() {

            protected void initChannel(Channel channel) throws Exception {
                channel.pipeline().addLast(

                        new HttpServerCodec(),
                        new HttpObjectAggregator(config.getMaxContentLength()),
                        new NettyServerConnectionManager(),
                        new NettyHttpServerHandler());
            }
        });
        try {
            this.serverBootstrap.bind().sync();
            log.info("server startup on port {}",config.getPort());
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public void shutdown() {
        if (eventLoopGroupBoss!=null){
            eventLoopGroupBoss.shutdownGracefully();
        }
        if (eventLoopGroupWorker!=null){
            eventLoopGroupWorker.shutdownGracefully();
        }
    }
}

```

# Implementing NettyHttpServerHandler

Our NettyHttpServerHandler inherits from ChannelInboundHandlerAdapter. Inheriting from ChannelInboundHandlerAdapter is to implement custom inbound data handling logic. ChannelInboundHandlerAdapter is an abstract class provided by Netty that implements the ChannelInboundHandler interface, which provides a set of default methods that allow developers to easily handle inbound events. The exact functionality can be searched on the web. In this module, we probably need to achieve the function and implementation steps are as follows: 1: inherit ChannelInboundHandlerAdapter 2: implement channelRead 3: delegate the logic to NettyProcessor

```java
package blossom.gateway.core.netty;

import blossom.gateway.core.wrapper.HttpRequestWrapper;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.handler.codec.http.FullHttpRequest;

public class NettyHttpServerHandler extends ChannelInboundHandlerAdapter {

    private final NettyProcessor processor;

    public NettyHttpServerHandler(NettyProcessor processor) {
        this.processor = processor;
    }

    public void channelRead(ChannelHandlerContext ctx,Object msg) throws Exception{
        FullHttpRequest request = (FullHttpRequest)msg;
        HttpRequestWrapper wrapper = new HttpRequestWrapper();
        wrapper.setFullHttpRequest(request);
        wrapper.setContext(ctx);

        processor.process(wrapper);
    }
}

```

# Implement NettyProcessor

This interface as the core interface, we need to implement the following functions: 1: define the interface 2: the minimum usable version of the implementation 3: routing function implementation 4: get asynchronous configuration, the implementation of complete method 5: exception handling 6: write back the response information and release resources

```java
package blossom.project.core.netty.processor;

import blossom.gateway.common.enums.ResponseCode;
import blossom.gateway.common.exception.BaseException;
import blossom.gateway.common.exception.ConnectException;
import blossom.gateway.common.exception.ResponseException;
import blossom.project.core.ConfigLoader;
import blossom.project.core.context.GatewayContext;
import blossom.project.core.context.HttpRequestWrapper;
import blossom.project.core.helper.AsyncHttpHelper;
import blossom.project.core.helper.RequestHelper;
import blossom.project.core.helper.ResponseHelper;
import blossom.project.core.response.GatewayResponse;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.FullHttpResponse;
import io.netty.util.ReferenceCountUtil;
import lombok.extern.slf4j.Slf4j;
import org.asynchttpclient.Request;
import org.asynchttpclient.Response;

import java.util.Objects;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeoutException;

public class NettyCoreProcessor implements NettyProcessor {

    public void process(HttpRequestWrapper wrapper) {
        FullHttpRequest request = wrapper.getRequest();
        ChannelHandlerContext ctx = wrapper.getCtx();

        try {
            GatewayContext gatewayContext = RequestHelper.doContext(request, ctx);
            route(gatewayContext);
        } catch (BaseException e) {
            log.error("process error {} {}", e.getCode().getCode(), e.getCode().getMessage());
            FullHttpResponse httpResponse = ResponseHelper.getHttpResponse(e.getCode());

            doWriteAndRelease(ctx, request, httpResponse);
        } catch (Throwable t) {
            log.error("process unkown error", t);
            FullHttpResponse httpResponse = ResponseHelper.getHttpResponse(ResponseCode.INTERNAL_ERROR);
            doWriteAndRelease(ctx, request, httpResponse);
        }

    }

    private void doWriteAndRelease(ChannelHandlerContext ctx, FullHttpRequest request, FullHttpResponse httpResponse) {
        ctx.writeAndFlush(httpResponse)
                .addListener(ChannelFutureListener.CLOSE);
        ReferenceCountUtil.release(request);
    }

    private void route(GatewayContext gatewayContext) {
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
                } else {
                    gatewayContext.setThrowable(new ConnectException(throwable,
                            gatewayContext.getUniqueId(),
                            url, ResponseCode.HTTP_RESPONSE_ERROR));
                }
            } else {

                gatewayContext.setResponse(GatewayResponse.buildGatewayResponse(response));
            }
        } catch (Throwable t) {
            gatewayContext.setThrowable(new ResponseException(ResponseCode.INTERNAL_ERROR));
            log.error("complete error", t);
        } finally {
            gatewayContext.written();
            ResponseHelper.writeResponse(gatewayContext);
        }
    }
}

```

# Implementing NettyHttpClient

Above we have implemented the Server server, now start to implement the client. The approximate steps are as follows: 1: implementation of LifeCyclke interface 2: encapsulation of properties 3: implementation of the init method 4: implementation of the start method 5: implementation of the shutdown method can be found in fact, the implementation of the client and the implementation of the server are more or less the same.

```java

public class NettyHttpClient implements LifeCycle {
    private final Config config;

    private final EventLoopGroup eventLoopGroupWoker;

    private AsyncHttpClient asyncHttpClient;

    public NettyHttpClient(Config config, EventLoopGroup eventLoopGroupWoker) {
        this.config = config;
        this.eventLoopGroupWoker = eventLoopGroupWoker;
        init();
    }

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

    public void start() {
        AsyncHttpHelper.getInstance().initialized(asyncHttpClient);
    }

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

# Implementing the core container

In the above we have completed the implementation of Netty-related code, we are going to implement the core container. The approximate steps are as follows: 1: implementation of the LifeCyclke interface 2: encapsulation of properties 3: implementation of the init method 4: implementation of the start method 5: implementation of the shutdown method The implementation of the core container can be used to start our Netty clients and servers, as long as the completion of the step, we have a simple request forwarding and receiving has been completed.

```java
package blossom.project.core;

import blossom.project.core.netty.NettyHttpClient;
import blossom.project.core.netty.NettyHttpServer;
import blossom.project.core.netty.processor.NettyCoreProcessor;
import blossom.project.core.netty.processor.NettyProcessor;
import lombok.extern.slf4j.Slf4j;

public class Container implements LifeCycle {
    private final Config config;

    private NettyHttpServer nettyHttpServer;

    private NettyHttpClient nettyHttpClient;

    private NettyProcessor nettyProcessor;

    public Container(Config config) {
        this.config = config;
        init();
    }

    public void init() {
        this.nettyProcessor = new NettyCoreProcessor();

        this.nettyHttpServer = new NettyHttpServer(config, nettyProcessor);

        this.nettyHttpClient = new NettyHttpClient(config,
                nettyHttpServer.getEventLoopGroupWoker());
    }

    public void start() {
        nettyHttpServer.start();;
        nettyHttpClient.start();
        log.info("api gateway started!");
    }

    public void shutdown() {
        nettyHttpServer.shutdown();
        nettyHttpClient.shutdown();
    }
}

```

# Demonstration

![](https://s2.loli.net/2023/11/05/eogl3C51jU9cwzO.webp)

![](https://s2.loli.net/2023/11/05/gdeDE2mivfHUwsO.webp)
