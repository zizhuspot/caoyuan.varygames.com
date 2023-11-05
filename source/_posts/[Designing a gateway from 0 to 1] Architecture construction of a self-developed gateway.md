---
title: Designing a gateway from 0 to 1: Architecture construction of a self-developed gateway
date: 2023-11-05 13:12:00
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
[Link to effect demo](https://www.bilibili.com/video/BV1eC4y1n73c/?vd_source=1d4d63e205b3ad352b4771f87295d16d#reply747752344)

# Build the project skeleton

The IDE tool I use here is IDEA. From the above, we know that our project has about five modules, Client, Common, Register Center, Config Center and Core. ![](https://s2.loli.net/2023/11/05/ZqyUlpkLC7uBw6D.webp) The initialization of the subfactory is written as follows. ![](https://s2.loli.net/2023/11/05/R7uQjoOXJETNSdP.webp) The initialization of the subfactory is written as follows.![](https://s2.loli.net/2023/11/05/dCj9tXP7BqIAZH6.webp) After that we can start building our starter class in our Core module. ![](https://s2.loli.net/2023/11/05/XiqkurTF9Nd2Wy8.webp) At this point our project skeleton has been built.

# Domain Model and DDD

Domain Model is a core concept in Domain-Driven Design (DDD), which is an abstract model used to represent and describe a specific domain. A domain model is a structured, object-oriented programming model for capturing and reflecting concepts, rules, and relationships in the actual business domain.

The advantages of a domain model are improved understanding of the business domain, increased modularity, and enhanced business rules, the disadvantages are the potential for increased complexity and development time, and the role is to realize business requirements and ensure that the software system is consistent with the business.

The role of domain modeling in software development includes:

Abstraction of Business Concepts: Domain modeling helps to abstract the various concepts and entities in the business domain into objects, classes and relationships in a programming language. This helps the development team to better understand the business requirements.

Representation of Business Rules: A domain model can contain business rules that can be represented in the model in the form of methods, attributes or constraints. This helps to ensure that the application follows the business rules.

Definition of Domain Objects: A domain model contains domain objects such as entities, value objects, aggregate roots, etc. that have business semantics and interact with other objects in the model.

Modeling of Business Processes: Domain models can be used to represent business processes, state transitions, and workflows. This helps developers to better understand and model business processes.

Separation of problem domain and solution: Domain modeling helps to separate the problem domain (business domain) from the solution domain (software development domain) so that developers can focus more on solving business problems.

Maintainability and Extensibility: The abstraction and clarity of the domain model makes the code easier to maintain and extend because it reflects the structure and rules of the business domain.

In short, the role of the domain model is to design for a **specific business**, and for this business, we will find out a lot of features related to this business, and if you change a business scenario, the current domain model may not be appropriate, you have to continue to design a new one. But for the current scenario, even if there are continuous requirements coming in, the domain model can ensure better usability because it is very well adapted to the business scenario in the previous design process.

And the design idea of domain model is probably:

* Find corresponding domain objects
* Create corresponding entities
* Setting up the corresponding link relationship for the entities

Then according to the above process, we know that, for our gateway project, after the service loading configuration is started, the gateway server receives the front-end request, after the request is parsed, the composition of the internal parameters in the internal flow of the gateway, such as serialization after a set of filters filtered, forwarded to the back-end service, the back-end service to process the request, the results of the gateway to return to the client. After the service loading configuration is started, the gateway server receives the front-end request, parses the request, and then composes the internal parameters to flow inside the gateway, for example, after serialization is filtered by a set of filters, it is forwarded to the back-end service, and after the back-end service handles the request, the result is returned to the client by the gateway. So we can get the following domain objects: Context, Request, Response, Config, Processor, Filter, FilterChain, Rule, HTTP request object.

After getting the domain object, we can start to set the corresponding entity object. For example, our Request should have some request time, parameters, id number, etc., Response should have the return value, status code and other information. And a Context context, is a complete request object processing.

So we probably also came up with the connection between the entities, that is: Context contains Request and Response.

# Core Context Model Encapsulation

The encapsulation of the core Context model involves the following steps:

* Context context core definition
* BaseContext base context implementation
* Context parameters
* GatewayRequest implementation
* GatewayResponse implementation
* Rule, FilterConfig implementation
* Final GatewayContext implementation

We start by defining an abstract class that defines some core functional actions. After the design is complete the code will look roughly as follows: ![](https://s2.loli.net/2023/11/05/cWipOKyBjIFzerG.webp) I won't post the exact code implementation here. But I will briefly introduce why these classes are needed here. Let's start with IContext, this is the top-level interface class for gateway contexts, which defines a number of methods. The role of this class is mainly to qualify some basic operations of the current gateway request. For example, whether the current gateway request is normally executed, or whether there is an exception, whether the request is a long connection, whether the request has a callback function and so on. After the Context is the internal Request and Response implementation. That is, the realization of the gateway request and gateway response information. Gateway request information includes the gateway's global start and end time, request id, request path, request host, path parameters, request body parameters, cookies, and the gateway will be forwarded to the back-end service after processing the information. The gateway response information includes response content, response status code, and asynchronous response object. After that, we need to set a rule to filter. And the rules are organized.

# Static configuration loading

After that we need to start writing some classes and methods to load the configuration of our gateway. Here are the configuration class details, which simply provide some information about the configuration class.

```java

public class GatewayConfig {

    private int port = 8080;

    private String applicationName = "blossom-gateway";

    private String registryAddress = "localhost:8848";

    private String env = "dev";

    private int eventLoopGroupBossNum = 1;

    private int eventLoopGroupWorkerNum = Runtime.getRuntime().availableProcessors();

    private int maxContentLength = 64 * 1024 * 1024;

    private boolean oddEvenAsync = false;
}

```

And the following provides configuration loading methods, respectively, from the configuration file, environment variables, JVM parameters, runtime parameters for loading configuration information.

```java
package blossom.gateway.core.config;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.collection.CollectionUtil;
import com.alibaba.fastjson2.util.BeanUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.ArrayUtils;

import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;
import java.util.Map;
import java.util.Objects;
import java.util.Properties;

public class ConfigLoader {
    public static final String CONFIG_FILE = "gateway.properties";
    public static final String ENV_PREFIX = "GATEWAY_";
    public static final String JVM_PREFIX = "gateway.";

    private static final ConfigLoader INSTANCE = new ConfigLoader();

    private GatewayConfig config;

    private ConfigLoader() {

    }

    public static ConfigLoader getInstance() {
        return INSTANCE;
    }

    public static GatewayConfig getConfig() {
        return INSTANCE.config;
    }

    public GatewayConfig load(String[] args) {

        config = new GatewayConfig();

        loadFromConfigFile();

        loadFromEnv();

        loadFromJvm();

        loadFromRuntimeArgs(args);
        return config;
    }

    private void loadFromRuntimeArgs(String[] args) {
        if (ArrayUtils.isNotEmpty(args)) {
            Properties properties = new Properties();
            for (String arg : args) {
                if (arg.startsWith("--") && arg.contains("=")) {
                    properties.put(arg.substring(2, arg.indexOf("=")), arg.substring(arg.indexOf("=") + 1));
                }
            }
            BeanUtil.copyProperties(properties, config);
        }
    }

    private void loadFromJvm() {
        Properties properties = System.getProperties();
        BeanUtil.copyProperties(properties, config);
    }

    private void loadFromEnv() {
        Map env = System.getenv();
        Properties properties = new Properties();
        properties.putAll(env);
        BeanUtil.copyProperties(properties, config);
    }

    private void loadFromConfigFile() {
        InputStream stream = ConfigLoader.class.getClassLoader().getResourceAsStream(CONFIG_FILE);
        if (Objects.nonNull(stream)) {
            Properties properties = new Properties();
            try {
                properties.load(stream);
                BeanUtil.copyProperties(properties, config);
            } catch (Exception e) {
                e.printStackTrace();

            } finally {
                try {
                    stream.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

# Component Lifecycle

The lifecycle piece is as simple as defining an interface that provides methods for initialization, startup, and shutdown.

```java
public interface LifeCycle {

    void init();

    void start();

    void shutdown();

}
```
