---
title: Designing a gateway from 0 to 1 Implementing fusion degradation based on Hystrix
date: 2023-11-05 19:12:00
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
Above we have successfully implemented request retry and request flow limitation, next we start to implement meltdown and service degradation. Meltdown and service degradation, designed in SpringCloud is our hystrix, here we will also consider with hystrix to realize meltdown and service degradation. If you don't know about hystix, you can have a look at it first.

# Dependency introduction

Since I'm using hystrix-based meltdown demotion here, I first need to integrate hystrix with the service

```c
	1.5.12
        1.5.12
        1.5.12

            com.netflix.hystrix
            hystrix-core
            ${hystrix.core.version}

            com.netflix.hystrix
            hystrix-metrics-event-stream
            ${hystrix.metrics.version}

            com.netflix.hystrix
            hystrix-javanica
            ${hystrix.javanica.version}

```

After introducing the dependencies as above, we can start writing how to do fuse limiting based on hystrix. I'll start by posting a set of code to give a general overview of how to implement fuse degradation using hystrix.

```java
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixCommandProperties;
import com.netflix.hystrix.HystrixThreadPoolProperties;

public class MyHystrixCommand extends HystrixCommand {

    private final String fallbackValue;

    protected MyHystrixCommand(String fallbackValue) {
        super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey("MyGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("MyCommand"))
            .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                .withExecutionTimeoutInMilliseconds(1000))
            .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
                .withCoreSize(10)
            );
        this.fallbackValue = fallbackValue;
    }

    protected String run() throws Exception {

        return "Result of the actual operation";
    }

    protected String getFallback() {

        return fallbackValue;
    }
}

```

In the above code, I have created a custom MyHystrixCommand class, inherited from HystrixCommand class. In the constructor of this class, you can configure some properties of the Hystrix, such as group, command name, execution timeout, and so on. Then, you need to implement the run method to execute the actual business logic and the getFallback method to execute the demotion logic.

Next, you can use this custom Hystrix command in your application:

```java
String result = new MyHystrixCommand("Fallback Value").execute();
```

By calling the execute method, you can execute Hystrix commands and if a meltdown occurs, it will execute the degradation logic and return the degraded value.

This approach allows you to control the behavior of meltdown and degradation at a finer granularity, but you need to manually configure Hystrix properties, such as timeout, thread pool size, etc. You can customize it according to your specific needs. You can customize it to suit your specific needs. So, based on the above, we've got a rough idea of how to implement a set of logic for meltdown downgrade based on hystrix.

# Service Degradation

So next let's write the specific implementation code. First, we need to add out the configuration of hystrix in the configuration center.

```java
"hystrixConfigs":[{
                "path":"/http-server/ping",
                "timeoutInMilliseconds":5000,
                "threadCoreSize":2,
                "fallbackResponse":"熔断超时"
            }]
```

After that, we know that our specific execution logic goes through the filter, so we need to add additional configuration to hystrix here in our route filter to monitor when we finally forward a request, and if this request fails to be processed or times out, to allow him to perform the meltdown degradation logic.

```java

    public void doFilter(GatewayContext gatewayContext) throws Exception {

        Optional hystrixConfig = getHystrixConfig(gatewayContext);

        if (hystrixConfig.isPresent()) {
            routeWithHystrix(gatewayContext, hystrixConfig);
        } else {
            route(gatewayContext, hystrixConfig);
        }
    }

    private static Optional getHystrixConfig(GatewayContext gatewayContext) {
        Rule rule = gatewayContext.getRule();
        Optional hystrixConfig =
                rule.getHystrixConfigs().stream().filter(c -> StringUtils.equals(c.getPath(),
                        gatewayContext.getRequest().getPath())).findFirst();
        return hystrixConfig;
    }

```

As you can see, my code above gets the configuration of the hystrix from the configuration center, and then determines if there is a fused downgrade configuration and goes to the fused downgrade logic. Instead of changing the original route logic when there is no fuse downgrade, we need to create an additional method that will be used when there is a fuse downgrade logic. Here we follow the code writing method we mentioned in the previous section to write and configure the code.

```java

    private void routeWithHystrix(GatewayContext gatewayContext, Optional hystrixConfig) {

        HystrixCommand.Setter setter =
                HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey(gatewayContext.getUniqueId()))
                        .andCommandKey(HystrixCommandKey.Factory.asKey(gatewayContext.getRequest().getPath()))

                        .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
                                .withCoreSize(hystrixConfig.get().getThreadCoreSize()))
                        .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()

                                .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD)

                                .withExecutionTimeoutInMilliseconds(hystrixConfig.get().getTimeoutInMilliseconds())
                                .withExecutionIsolationThreadInterruptOnTimeout(true)
                                .withExecutionTimeoutEnabled(true));

        new HystrixCommand(setter) {

            protected Object run() throws Exception {

                route(gatewayContext, hystrixConfig).get();
                return null;
            }

            protected Object getFallback() {

                gatewayContext.setResponse(hystrixConfig.get().getFallbackResponse());
                gatewayContext.written();
                return null;
            }
        }.execute();
    }

```

# Demo

Once the above code is written, we have completed the fuse downgrade. Let's take a look at how this works. Start the backend service and let it block for a long time. Then a timeout exception is triggered, and the fusion downgrade occurs.![](https://s2.loli.net/2023/11/05/SJaEQse56bTHwpm.webp)
