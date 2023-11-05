---
title: [Designing a Gateway from 0 to 1] Integrating Nacos - Service Registration and Service Subscription Implementation
date: 2023-11-05 20:12:00
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
You can get the project source code and project information from the video introduction [link to effect demo](https://www.bilibili.com/video/BV1eC4y1n73c/?vd_source=1d4d63e205b3ad352b4771f87295d16d#reply747752344)

# Nacos

Nacos provides a number of powerful features: service discovery, health check. Nacos supports both DNS-based and RPC-based service discovery. Nacos also provides real-time health checking of services, preventing requests from being sent to unhealthy hosts and services. And Nacos also provides a visual console to facilitate the management of instances and other information. Nacos also provides a dynamic configuration service that allows us to manage the application and service configuration of all environments in a centralized, externalized and dynamic way.

![](https://s2.loli.net/2023/11/05/ZDFN5KfolqXx1Ih.webp)

Nacos is one of the registration and configuration centers that I use the most in the process of developing my own projects, and the community of Nacos is more active and the code is easier to read than others. Here is the official Nacos website: [Nacos official website](https://nacos.io/zh-cn/) I won't go into too much detail about the features of Nacos in this article. In this chapter, I will use the interfaces exposed by Nacos to accomplish the service registration function and service discovery function of the project.
Completing this chapter will also give you a deeper understanding of the underlying principles of Nacos, the registry principle. The following are some of the articles I have written in the process of learning Nacos, interested can look at. [Using Nacos to implement dynamic thread pooling techniques and Nacos configuration file update listener events](https://blog.csdn.net/Zhangsama1/article/details/131227567?ops_request_misc=%257B%2522request% 255Fid%2522%253A%2522169812851116800213040303%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id =169812851116800213040303&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1 -131227567-null-null.nonecase&utm_term=Nacos&spm=1018.2226.3001.4450) [[Source Code Analysis] How does Nacos use the AP protocol to accomplish data synchronization between servers?] (https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2FZhangsama1%2Farticle%2Fdetails%2F132143057%3Fops_request_misc%3D% 25257B%252522request%25255Fid%252522%25253A%252522169812851116800213040303%252522%25252C%252522scm%252522%25253A%25253A 25252220140713.130102334.pc%25255Fblog.%252522%25257D%26request_id%3D169812851116800213040303%26biz_id%3D0%26utm_medium%26 3Ddistribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-4-132143057-null-null.nonecase%26utm_term% 3DNacos%26spm%3D1018.2226.3001.4450 "https://blog.csdn.net/Zhangsama1/article/details/132143057?ops_request_misc=%257B%2522request% 255Fid%2522%253A%2522169812851116800213040303%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id =169812851116800213040303&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-4 -132143057-null-null.nonecase&utm_term=Nacos&spm=1018.2226.3001.4450") [[Source Code Analysis] How does the Nacos server side update as well as save registry information?] (https://blog.csdn.net/Zhangsama1/article/details/132141120?ops_request_misc=%257B%2522request%255Fid%2522%253A% 2522169812851116800213040303%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id= 169812851116800213040303&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-5- 132141120-null-null.nonecase&utm_term=Nacos&spm=1018.2226.3001.4450) [Implementation of Nacos auto-registration principle as well as service registration update and how to save it to the registry](https://link.juejin.cn?target =https%3A%2F%2Fblog.csdn.net%2FZhangsama1%2Farticle%2Fdetails%2F132145216%3Fops_request_misc%3D%25257B%252522request%25255Fid% 252522%25253A%2522169812851116800213040303%252522%25252C%252522scm%252522%25253A%252220140713.130102334.pc%25255Fblog.%252522%252522%252522 25257D%26request_id%3D169812851116800213040303%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~blog~first _rank_ecpm_v1~rank_v31_ecpm-6-132145216-null-null.nonecase%26utm_term%3DNacos%26spm%3D1018.2226.3001.4450 "https://blog.csdn.net/ Zhangsama1/article/details/132145216?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169812851116800213040303%2522%252C% 2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=169812851116800213040303&biz_id=0&utm_medium=distribute .pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-6-132145216-null-null.nonecase&utm_term=Nacos&spm= 1018.2226.3001.4450")

Why I chose Nacos was briefly explained in a previous post, so I'll list a few reasons in detail here:

* Nacos provides allows me to manage all services and metadata in the data center from the perspective of building a microservices platform, the specific reasons you can look at the above my analysis of the Nacos source code, Nacos will be the services of fine-grained division into their respective instances, and we can manage the information of these instances
* Nacos supports both DNS-based and RPC-based service discovery, which means that it provides us with strong service discovery options.
* Nacos provides real-time health checking of services, preventing requests from being sent to unhealthy hosts or service instances, i.e., security.
* Dynamic configuration services allow you to manage application configuration and service configuration across all environments in a centralized, externalized, and dynamic way, which I've leveraged before for dynamic configuration of thread pools [check out this article for more details] (https://blog.csdn.net/Zhangsama1/article/details/ 131227567?spm=1001.2014.3001.5502)

# Define the service registration and subscription methods

In this step, we will need to define some interfaces that gateway items use to connect to Nacos, the registry, to implement the interfaces that will link our items to the registry. To register a service to the registry, we need to initialize, register, unregister, and subscribe to the service, i.e., we need to write the following interface to provide such an interface and implement the interface methods in the specific registry instance later.

```java
public interface RegisterCenter {

    void init(String registerAddress, String env);

    void register(ServiceDefinition serviceDefinition, ServiceInstance serviceInstance);

    void deregister(ServiceDefinition serviceDefinition, ServiceInstance serviceInstance);

    void subscribeAllServices(RegisterCenterListener registerCenterListener);
}

```

After implementing the interface, we need to provide a method that listens for configuration changes in the registry. This is a particularly important feature of Nacos as a registry and configuration center, and the interface definition is as follows:

```java
public interface RegisterCenterListener {

    void onChange(ServiceDefinition serviceDefinition,
                  Set serviceInstanceSet);
}

```

# Service Information Loading and Configuration

Based on the above service registration and subscription interfaces, we can roughly write out how to register our gateway to Nacos. Of course, we don't have a specific implementation of how to register with the Nacos registry, but we can write out a general way to call it.

```java

public class Bootstrap
{
    public static void main( String[] args )
    {

        Config config = ConfigLoader.getInstance().load(args);
        System.out.println(config.getPort());

        Container container = new Container(config);
        container.start();

        final RegisterCenter registerCenter = registerAndSubscribe(config);

        Runtime.getRuntime().addShutdownHook(new Thread(){

            public void run(){
                registerCenter.deregister(
                        buildGatewayServiceDefinition(config),
                        buildGatewayServiceInstance(config));
            }
        });
    }

    private static RegisterCenter registerAndSubscribe(Config config) {

        ServiceLoader serviceLoader = ServiceLoader.load(RegisterCenter.class);
        final RegisterCenter registerCenter = serviceLoader.findFirst().orElseThrow(() -> {
            log.error("not found RegisterCenter impl");
            return new RuntimeException("not found RegisterCenter impl");
        });

        registerCenter.init(config.getRegistryAddress(), config.getEnv());

        ServiceDefinition serviceDefinition = buildGatewayServiceDefinition(config);
        ServiceInstance serviceInstance = buildGatewayServiceInstance(config);

        registerCenter.register(serviceDefinition, serviceInstance);

        registerCenter.subscribeAllServices(new RegisterCenterListener() {

            public void onChange(ServiceDefinition serviceDefinition, Set serviceInstanceSet) {
                log.info("refresh service and instance: {} {}", serviceDefinition.getId(),
                        JSON.toJSON(serviceInstanceSet));
                DynamicConfigManager manager = DynamicConfigManager.getInstance();
                manager.addServiceInstance(serviceDefinition.getId(), serviceInstanceSet);
            }
        });
        return registerCenter;
    }

    private static ServiceInstance buildGatewayServiceInstance(Config config) {
        String localIp = NetUtils.getLocalIp();
        int port = config.getPort();
        ServiceInstance serviceInstance = new ServiceInstance();
        serviceInstance.setServiceInstanceId(localIp + COLON_SEPARATOR + port);
        serviceInstance.setIp(localIp);
        serviceInstance.setPort(port);
        serviceInstance.setRegisterTime(TimeUtil.currentTimeMillis());
        return serviceInstance;
    }

    private static ServiceDefinition buildGatewayServiceDefinition(Config config) {
        ServiceDefinition serviceDefinition = new ServiceDefinition();
        serviceDefinition.setInvokerMap(Map.of());
        serviceDefinition.setId(config.getApplicationName());
        serviceDefinition.setServiceId(config.getApplicationName());
        serviceDefinition.setEnvType(config.getEnv());
        return serviceDefinition;
    }

}

```

One of the more important ones is this line of code, which is to load the service provider

```java
ServiceLoader.load(RegisterCenter.class)
```

ServiceLoader is a tool for loading service providers in Java, often used to implement a service provider framework. Its role is to find and load service provider implementation classes for specified interfaces or abstract classes that are dynamically registered into the system at runtime so that other components or applications can use their functionality.

Specifically, here is what it does and how it is used:

* Service Interface Definition: First, you need to define a service interface or abstract class, which is an abstract description of the different implementations you want. In your example, RegisterCenter.class seems to be a service interface.
* Service Provider Implementation: Different modules or libraries can provide different implementations of the service interface, these implementation classes can be developed independently of the application and can be loaded at runtime.
* Service provider registration: Each service provider implementation class requires a file to be created in the META-INF/services directory with the fully qualified name of the service interface and the fully qualified name of the service provider implementation class. This tells the Java runtime which classes implement the service interface.
* Load service providers: Using ServiceLoader.load(RegisterCenter.class), you can load all registered service provider implementation classes. This returns a ServiceLoader object that you can iterate over to get instances of all loaded implementation classes.

This mechanism allows applications to dynamically switch between and use different service provider implementations without modifying the source code, thus increasing the scalability and flexibility of the application. It is commonly used in frameworks and libraries to allow developers to plug in their own implementations such as database drivers, loggers, plug-ins, etc.

# Implement registering gateways to the registry

To register the gateway to the registry, we first need to introduce the Nacos client dependency.

```java

            com.alibaba.nacos
            nacos-client
            2.0.4

            blossom.project
            BlossomGateway-Register-Center-Api
            1.0

```

After that, we can register the service using the service registration method provided in the Nacos client. The way is as follows:

```java

public class NacosRegisterCenter implements RegisterCenter {

    private String registerAddress;

    private String env;

    private NamingService namingService;

    private NamingMaintainService namingMaintainService;

    private List registerCenterListenerList = new CopyOnWriteArrayList<>();

    public void init(String registerAddress, String env) {
        this.registerAddress = registerAddress;
        this.env = env;

        try {
            this.namingMaintainService = NamingMaintainFactory.createMaintainService(registerAddress);
            this.namingService = NamingFactory.createNamingService(registerAddress);
        } catch (NacosException e) {
            throw new RuntimeException(e);
        }

    }

    public void register(ServiceDefinition serviceDefinition, ServiceInstance serviceInstance) {
        try {

            Instance nacosInstance = new Instance();
            nacosInstance.setInstanceId(serviceInstance.getServiceInstanceId());
            nacosInstance.setPort(serviceInstance.getPort());
            nacosInstance.setIp(serviceInstance.getIp());

            nacosInstance.setMetadata(Map.of(GatewayConst.META_DATA_KEY, JSON.toJSONString(serviceInstance)));

            namingService.registerInstance(serviceDefinition.getServiceId(), env, nacosInstance);

            namingMaintainService.updateService(serviceDefinition.getServiceId(), env, 0,
                    Map.of(GatewayConst.META_DATA_KEY, JSON.toJSONString(serviceDefinition)));

            log.info("register {} {}", serviceDefinition, serviceInstance);
        } catch (NacosException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Here you need to have an understanding of the Nacos source code in order for you to understand how to register a service instance to the Nacos registry, which on its side asks for information such as the service's ip, port, and service name. After completing this step, we have probably successfully registered the service with Nacos.

# Implementing Service Subscription

Here we start to realize the service subscription, in order to realize the service subscription, we need to pull all the information of the service above Nacos, and the service information will be constantly updated, so we also need to use a timed task to constantly update our service subscription information. To subscribe to Nacos service information, we need to use the Nacos event listener, NamingEvent. In the Nacos registry, NamingEvent is an event object that represents events related to the service namespace (Naming). NamingEvent is used to listen for and handle changes to the Service Instance in the namespace so that the application can dynamically update the list of Service Instances based on these changes to stay in sync with the registry.

Specifically, NamingEvent is mainly used for the following purposes:

* Listening for changes to Service Instances: The Nacos registry can contain a large number of Service Instances that may change as services come online, go offline, changes to instance metadata, etc. NamingEvent allows applications to register listeners to be notified when changes to Service Instances occur.
* Dynamically update the service instance list: By listening to NamingEvent, applications can get real-time status changes about service instances, so they can update the list of service instances they maintain in a timely manner to ensure that they are using the most up-to-date information about the service instances.
* Implementing Load Balancing: Applications can implement a load balancing policy based on the information provided by NamingEvent, such as selecting appropriate service instances to serve service requests. The load balancing policy can be adjusted based on the availability, health state, and other metadata of the service instances.
* Dynamic routing: Some applications may need to implement dynamic routing, where routing rules are dynamically updated based on changes to service instances to ensure that requests are correctly routed to available service instances.

The rough code implementation is as follows:

```java

    public void subscribeAllServices(RegisterCenterListener registerCenterListener) {

        registerCenterListenerList.add(registerCenterListener);

        doSubscribeAllServices();

        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(1, new NameThreadFactory(
                "doSubscribeAllServices"));

        scheduledThreadPool.scheduleWithFixedDelay(() -> doSubscribeAllServices(), 10, 10, TimeUnit.SECONDS);

    }

    private void doSubscribeAllServices() {
        try {

            Set subscribeService =
                    namingService.getSubscribeServices().stream().map(ServiceInfo::getName).collect(Collectors.toSet());

            int pageNo = 1;
            int pageSize = 100;

            List serviseList = namingService.getServicesOfServer(pageNo, pageSize, env).getData();

            while (CollectionUtils.isNotEmpty(serviseList)) {
                log.info("service list size {}", serviseList.size());

                for (String service : serviseList) {

                    if (subscribeService.contains(service)) {
                        continue;
                    }

                    EventListener eventListener = new NacosRegisterListener();
                    eventListener.onEvent(new NamingEvent(service, null));
                    namingService.subscribe(service, env, eventListener);
                    log.info("subscribe {} {}", service, env);
                }

                serviseList = namingService.getServicesOfServer(++pageNo, pageSize, env).getData();
            }

        } catch (NacosException e) {
            throw new RuntimeException(e);
        }
    }

    public class NacosRegisterListener implements EventListener {

        public void onEvent(Event event) {
            if (event instanceof NamingEvent) {
                log.info("the triggered event info is：{}",JSON.toJSON(event));
                NamingEvent namingEvent = (NamingEvent) event;
                String serviceName = namingEvent.getServiceName();

                try {

                    Service service = namingMaintainService.queryService(serviceName, env);
                    ServiceDefinition serviceDefinition =
                            JSON.parseObject(service.getMetadata().get(GatewayConst.META_DATA_KEY),
                                    ServiceDefinition.class);

                    List allInstances = namingService.getAllInstances(service.getName(), env);
                    Set set = new HashSet<>();

                    for (Instance instance : allInstances) {
                        ServiceInstance serviceInstance =
                                JSON.parseObject(instance.getMetadata().get(GatewayConst.META_DATA_KEY),
                                        ServiceInstance.class);
                        set.add(serviceInstance);
                    }

                    registerCenterListenerList.stream().forEach(l -> l.onChange(serviceDefinition, set));
                } catch (NacosException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
```

At this point, we are done pulling in the latest configuration information at a time when information changes in the Nacos registry. That is, we are done subscribing to the registry.
