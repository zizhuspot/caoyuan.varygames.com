---
title: [Designing a gateway from 0 to 1] Integrating Nacos-Configuration Pull and Configuration Change Information Ding Subscription
date: 2023-11-05 21:12:00
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
In the above article, we have roughly completed the service registration function of the registration center, in this article we will implement the configuration center configuration pull and configuration change listening function. Still first need to define a configuration center interface to initialize the configuration center configuration and configuration center information change listening event.

```java
public interface ConfigCenter {

    void init(String serverAddr, String env);

    void subscribeRulesChange(RulesChangeListener listener);
}

```

```c
public interface RulesChangeListener {

    void onRulesChange(List rules);
}

```

After completing the simple interface definition, start thinking about how to implement the specific configuration pull. We still need to introduce the Nacos client first.

```c

            com.alibaba.nacos
            nacos-client
            2.0.4

```

After that, nacos has provided us with configService, a class that helps us quickly pull Nacos configurations. The usage is as follows:

```c
    private static final String DATA_ID = "api-gateway";

    private String serverAddr;

    private String env;

    private ConfigService configService;

    @Override
    public void init(String serverAddr, String env) {
        this.serverAddr = serverAddr;
        this.env = env;

        try {
            this.configService = NacosFactory.createConfigService(serverAddr);
        } catch (NacosException e) {
            throw new RuntimeException(e);
        }
    }

```

The above has completed the initialization of the configuration center, then we can use the methods provided by the configuration center to pull our configuration

```c

            String configJson = configService.getConfig(DATA_ID, env, 5000);

            log.info("config from nacos: {}", configJson);
            List rules = JSON.parseObject(configJson).getJSONArray("rules").toJavaList(Rule.class);

```

Here you can parse your configuration any way you like.

As usual, we also need to subscribe to the configuration change event. The method is as follows:

```c
 @Override
    public void subscribeRulesChange(RulesChangeListener listener) {
        try {

            String configJson = configService.getConfig(DATA_ID, env, 5000);

            log.info("config from nacos: {}", configJson);
            List rules = JSON.parseObject(configJson).getJSONArray("rules").toJavaList(Rule.class);

            listener.onRulesChange(rules);

            configService.addListener(DATA_ID, env, new Listener() {

                @Override
                public Executor getExecutor() {
                    return null;
                }

                @Override
                public void receiveConfigInfo(String configInfo) {
                    log.info("config from nacos: {}", configInfo);
                    List rules = JSON.parseObject(configInfo).getJSONArray("rules").toJavaList(Rule.class);
                    listener.onRulesChange(rules);
                }
            });

        } catch (NacosException e) {
            throw new RuntimeException(e);
        }
    }
```

More to the point is this line of code

```c

            configService.addListener(DATA_ID, env, new Listener()
```

He will help us add one of our listeners to the list of Nacos listeners, so that when the configuration of Nacos changes, we can listen to the event and execute the processing logic we want.

At this point, the integration of Nacos is simply complete. This module requires a deep understanding of the Nacos source code and interface, so it is recommended that you finish learning Nacos before reading this series of articles.
