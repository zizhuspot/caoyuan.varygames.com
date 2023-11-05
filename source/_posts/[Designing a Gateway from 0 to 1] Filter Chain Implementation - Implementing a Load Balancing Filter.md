---
title: Designing a Gateway from 0 to 1 Filter Chain Implementation - Implementing a Load Balancing Filter
date: 2023-11-05 16:12:00
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

# What is a filter?

In the previous sections we've implemented registering our gateway service to the registry and successfully pulling configurations from the configuration center. Next we'll start implementing the core of a gateway service, which is a filter chain. A filter chain consists of multiple filters. After a filter completes its filtering process, it forwards the request to the next filter for further execution. Thus completing the processing of requests and responses. And if you understand SpringCloudGateway, you will know that the filter is divided into global and local filters. The former for all requests for processing , and local filters SpringCloud has been the default to help us achieve , of course, we can also inherit and implement their own .

Filter in accordance with the chain of request processing, if you understand the gateway project should be aware of, when all the filters request processing is complete, there will be a routing filter will be sent to the corresponding backend service request processing, that is, the request will be forwarded to the backend service when the service processing is complete, it will be returned to the request again. If an exception occurs during the filter chaining process, we can also use the filter chaining method to catch it. If the request is forwarded normally and processed, we can use context.writeAndFlush method to write the data back and return. The general flow is as follows:

![](https://s2.loli.net/2023/11/05/KR7VrLjwict6l8y.webp) [You can take a brief look at Gateway in Gateway through this article](https://blog.csdn.net/Zhangsama1/article/details/133517494?spm=1001.2014.3001.5502) [SpringCLoudGateway Implementing URL encryption and digital signatures](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2FZhangsama1%2Farticle%2Fdetails%2F133522946%3Fspm% 3D1001.2014.3001.5502 "https://blog.csdn.net/Zhangsama1/article/details/133522946?spm=1001.2014.3001.5502")

Having finished understanding the simple concept of filters, let's start analyzing how to implement them. First define a Filter top-level interface following the SpringCloudGateway (hereafter collectively referred to as scg) approach. And we need to implement the Ordered interface to set the processing priority. And we also set up a cutter to enhance the filter , so that we can get some information about the filter , and at the same time to facilitate the development of pluggable , so that we can develop in accordance with the SPI way . After that, we also need to set up a factory production class, FilterFactory, to help us produce the chain table and execute it. And we also need to use the filter chain table, which is GatewayFilterChain in scg. At the same time, we also need to implement the filter chain factory by setting up a class: GatewayFilterChainFactory

So we can get, Filter as the top-level interface of the filter, its subclasses need to implement this interface and implement specific filter methods.

FilterAspect is used to provide filter AOP functionality to facilitate the management of our filters.

FilterFactory filter factory , used to build the filter chain table and provide according to the filter ID to get the filter method .

GatewayFilterChain provides specific methods for adding filters and executing filter chain processing logic.

GatewayFilterChainFactory implements FilterFactory to realize the specific method of constructing filter chain and provide the actual method of getting filters according to their IDs.

Here posted a specific code implementation: First is the filter chain class , used to store the actual filter , and provide filter execution methods .

```c
public class GatewayFilterChain {

    private List filters = new ArrayList<>();

    public GatewayFilterChain addFilter(Filter filter){
        filters.add(filter);
        return this;
    }
    public GatewayFilterChain addFilterList(List filter){
        filters.addAll(filter);
        return this;
    }

    public GatewayContext doFilter(GatewayContext ctx) throws Exception {
        if(filters.isEmpty()){
            return ctx;
        }
        try {
            for(Filter fl: filters){
                fl.doFilter(ctx);
            }
        }catch (Exception e){
            log.error("执行过滤器发生异常,异常信息：{}",e.getMessage());
            throw e;
        }
        return ctx;
    }
}
```

Next, we provide the filter chain factory. The role of the filter chain factory is to store the filter configuration information, create filter chains, and provide methods to get filters. The filter configuration information comes from the configuration center we set up earlier.

```c
@Slf4j
public class GatewayFilterChainFactory implements FilterFactory {

    private static class SingletonInstance {
        private static final GatewayFilterChainFactory INSTANCE = new GatewayFilterChainFactory();
    }

    public static GatewayFilterChainFactory getInstance() {
        return SingletonInstance.INSTANCE;
    }

    private Map processorFilterIdMap = new ConcurrentHashMap<>();

    public GatewayFilterChainFactory() {
        ServiceLoader serviceLoader = ServiceLoader.load(Filter.class);
        serviceLoader.stream().forEach(filterProvider -> {
            Filter filter = filterProvider.get();
            FilterAspect annotation = filter.getClass().getAnnotation(FilterAspect.class);
            log.info("load filter success:{},{},{},{}", filter.getClass(),
                    annotation.id(), annotation.name(), annotation.order());
            if (annotation != null) {

                String filterId = annotation.id();
                if (StringUtils.isEmpty(filterId)) {
                    filterId = filter.getClass().getName();
                }
                processorFilterIdMap.put(filterId, filter);
            }
        });

    }

    public static void main(String[] args) {
        new GatewayFilterChainFactory();
    }

    @Override
    public GatewayFilterChain buildFilterChain(GatewayContext ctx) throws Exception {
        GatewayFilterChain chain = new GatewayFilterChain();
        List filters = new ArrayList<>();

        Rule rule = ctx.getRule();
        if (rule != null) {

            Set filterConfigs = rule.getFilterConfigs();
            Iterator iterator = filterConfigs.iterator();
            Rule.FilterConfig filterConfig;
            while (iterator.hasNext()) {
                filterConfig = (Rule.FilterConfig) iterator.next();
                if (filterConfig == null) {
                    continue;
                }
                String filterId = filterConfig.getId();
                if (StringUtils.isNotEmpty(filterId) && getFilterInfo(filterId) != null) {
                    Filter filter = getFilterInfo(filterId);
                    filters.add(filter);
                }
            }
        }

        filters.add(new RouterFilter());

        filters.sort(Comparator.comparingInt(Filter::getOrder));

        chain.addFilterList(filters);
        return chain;
    }

    @Override
    public Filter getFilterInfo(String filterId) throws Exception {
        return processorFilterIdMap.get(filterId);
    }
}

```

## Writing load balancing filters

## Definition and Implementation of Load Balancing

Before writing a load balancing filter, you need to understand what load balancing is.

> Load Balancing is a technique in computer networking and server architecture designed to distribute network requests, data streams, or loads to multiple servers or computing resources to ensure high availability, improve performance, and avoid overloading any single server or resource. Load balancing plays an important role in distributed systems and network applications by helping to cope with traffic fluctuations and providing redundancy to improve system reliability and performance.

And for the implementation of load balancing, we have the following ways:

* DNS load balancing
* Hardware load balancing
* Software load balancing

DNS Load Balancing (Geographic Level): Principle: DNS Load Balancing uses DNS servers to map domain name resolution requests to multiple different IP addresses, each of which corresponds to a load balancer or server. the DNS servers return the resolved IP addresses to the client, which then sends the request to one of the IP addresses.

Pros: Relatively simple, no additional hardware or software load balancers required, easy to implement and scale.

Cons: DNS load balancing does not have the ability to intelligently distribute traffic, dynamically adjust load, or handle server failure detection and recovery. Client-side caching of DNS records may result in uneven traffic distribution. DNS load balancing cannot detect whether a back-end service is alive or not. there may be requests for down services.

Hardware Load Balancing: Principle: Hardware load balancing is the distribution of traffic to back-end servers by means of specialized hardware devices. These hardware devices usually have performance advantages and can handle a large number of connections and requests. Search for F5 and A10 load balancers if you are interested.

Benefits: High performance, specifically designed for load balancing tasks, usually with high availability and reliability. Supports advanced load balancing algorithms and traffic management.

Cons: Relatively expensive, requiring the purchase of specialized hardware devices. Configuration and management can be complex and require specialized knowledge.

Software Load Balancing: Principle: Software load balancing is the distribution of traffic by running load balancing software on common servers. These software can be open source or commercial, such as Nginx, HAProxy, LVS, etc.

Benefits: Relatively economical, can run on common hardware, easy to deploy and manage. Offers a variety of load balancing algorithms and advanced configuration options.

Cons: Performance may be limited by server hardware and may require increased server count for extremely high traffic loads. Availability and reliability may not be as good as specialized hardware appliances.

It is recommended to learn more about the differences between Nginx and LVS load balancing.

These load balancers are not used individually, but in conjunction with each other in actual production. DNS load balancing is used for geographic load balancing 2. Hardware load balancing is used for cluster load balancing 3. Software load balancing is used for machine load balancing.

## Load balancing algorithms

Static load balancing algorithms: polling, ratio, priority, which is more commonly used is polling, the characteristics are as follows: 1, the order of the cycle of connecting to each server in the queue, once a server anomaly, then it will be removed from the queue. 2, Advantages: the implementation of a simple, efficient, easy to expand horizontally. 3, Disadvantages: uncertainty of the destination node of the request is not suitable for the storage of written scenarios. 4, the number of servers in the load balancing algorithm: the number of servers in the queue is not enough.

Dynamic load balancing algorithms: minimum number of connections, fastest response time, dynamic performance allocation, dynamic server supplementation, quality of service, etc. Dynamic load balancing algorithms are more commonly used dynamic performance allocation, which dynamically adjusts the traffic distribution by means of the performance parameters of the application programs and application servers collected by BIG-IP. Generally, we will work with Prometheus to realize this.

## Design Implementation

The first step is to create our top-level interface, which is used to help us get the back-end service instances selected according to the load balancing policy.

```c
public interface LoadBalanceGatewayRule {

    ServiceInstance choose(GatewayContext ctx);

    ServiceInstance choose(String serviceId);

}

```

After we implement this interface, we first implement a relatively simple random load balancing strategy. The implementation is based on our service id, and then save all the service instances corresponding to the current service id, after which we can randomly return one from the service instances.

```c
@Slf4j
public class RandomLoadBalanceRule implements LoadBalanceGatewayRule {

    private final String serviceId;

    private Set serviceInstanceSet;

    public RandomLoadBalanceRule(String serviceId) {
        this.serviceId = serviceId;
    }

    private static ConcurrentHashMap serviceMap = new ConcurrentHashMap<>();

    public static RandomLoadBalanceRule getInstance(String serviceId) {
        RandomLoadBalanceRule loadBalanceRule = serviceMap.get(serviceId);
        if (loadBalanceRule == null) {
            loadBalanceRule = new RandomLoadBalanceRule(serviceId);
            serviceMap.put(serviceId, loadBalanceRule);
        }
        return loadBalanceRule;
    }

    @Override
    public ServiceInstance choose(GatewayContext ctx) {
        String serviceId = ctx.getUniqueId();
        return choose(serviceId);
    }

    @Override
    public ServiceInstance choose(String serviceId) {
        Set serviceInstanceSet =
                DynamicConfigManager.getInstance().getServiceInstanceByUniqueId(serviceId);
        if (serviceInstanceSet.isEmpty()) {
            log.warn("No instance available for:{}", serviceId);
            throw new NotFoundException(SERVICE_INSTANCE_NOT_FOUND);
        }
        List instances = new ArrayList(serviceInstanceSet);
        int index = ThreadLocalRandom.current().nextInt(instances.size());
        ServiceInstance instance = (ServiceInstance) instances.get(index);
        return instance;
    }
}

```

For the polling load balancing strategy, we would need to maintain a global index number and then keep incrementing it each time we execute, and then take the remainder of the number of service instances to know which backend instance to execute.

```c
@Slf4j
public class RoundRobinLoadBalanceRule implements LoadBalanceGatewayRule {

    private AtomicInteger position = new AtomicInteger(1);

    private final String serviceId;

    public RoundRobinLoadBalanceRule(String serviceId) {
        this.serviceId = serviceId;
    }

    private static ConcurrentHashMap serviceMap = new ConcurrentHashMap<>();

    public static RoundRobinLoadBalanceRule getInstance(String serviceId) {
        RoundRobinLoadBalanceRule loadBalanceRule = serviceMap.get(serviceId);
        if (loadBalanceRule == null) {
            loadBalanceRule = new RoundRobinLoadBalanceRule(serviceId);
            serviceMap.put(serviceId, loadBalanceRule);
        }
        return loadBalanceRule;
    }

    @Override
    public ServiceInstance choose(GatewayContext ctx) {
        return choose(ctx.getUniqueId());
    }

    @Override
    public ServiceInstance choose(String serviceId) {
        Set serviceInstanceSet =
                DynamicConfigManager.getInstance().getServiceInstanceByUniqueId(serviceId);
        if (serviceInstanceSet.isEmpty()) {
            log.warn("No instance available for:{}", serviceId);
            throw new NotFoundException(SERVICE_INSTANCE_NOT_FOUND);
        }
        List instances = new ArrayList(serviceInstanceSet);
        if (instances.isEmpty()) {
            log.warn("No instance available for service:{}", serviceId);
            return null;
        } else {
            int pos = Math.abs(this.position.incrementAndGet());
            return instances.get(pos % instances.size());
        }
    }
}

```

Finally, we will be able to select the load balancing policy for our implementation based on the load balancing policy set in the request header to be used.

```c
@Slf4j
@FilterAspect(id=LOAD_BALANCE_FILTER_ID,
        name = LOAD_BALANCE_FILTER_NAME,
        order = LOAD_BALANCE_FILTER_ORDER)
public class LoadBalanceFilter implements Filter {

    @Override
    public void doFilter(GatewayContext ctx){

        String serviceId = ctx.getUniqueId();

        LoadBalanceGatewayRule gatewayLoadBalanceRule = getLoadBalanceRule(ctx);
        ServiceInstance serviceInstance = gatewayLoadBalanceRule.choose(serviceId);
        System.out.println("IP为"+serviceInstance.getIp()+",端口号："+serviceInstance.getPort());
        GatewayRequest request = ctx.getRequest();
        if(serviceInstance != null && request != null){
            String host  = serviceInstance.getIp()+":"+serviceInstance.getPort();
            request.setModifyHost(host);
        }else{
            log.warn("No instance available for :{}",serviceId);
            throw new NotFoundException(SERVICE_INSTANCE_NOT_FOUND);
        }
    }

    public LoadBalanceGatewayRule getLoadBalanceRule(GatewayContext ctx) {
        LoadBalanceGatewayRule loadBalanceRule = null;
        Rule configRule = ctx.getRule();
        if (configRule != null) {
            Set filterConfigs = configRule.getFilterConfigs();
            Iterator iterator = filterConfigs.iterator();
            Rule.FilterConfig filterConfig;
            while (iterator.hasNext()) {
                filterConfig = (Rule.FilterConfig) iterator.next();
                if (filterConfig == null) {
                    continue;
                }
                String filterId = filterConfig.getId();
                if (filterId.equals(LOAD_BALANCE_FILTER_ID)) {
                    String config = filterConfig.getConfig();
                    String strategy = LOAD_BALANCE_STRATEGY_RANDOM;
                    if (StringUtils.isNotEmpty(config)) {
                        Map mapTypeMap = JSON.parseObject(config, Map.class);
                        strategy = mapTypeMap.getOrDefault(LOAD_BALANCE_KEY, strategy);
                    }
                    switch (strategy) {
                        case LOAD_BALANCE_STRATEGY_RANDOM:
                            loadBalanceRule = RandomLoadBalanceRule.getInstance(configRule.getServiceId());
                            break;
                        case LOAD_BALANCE_STRATEGY_ROUND_ROBIN:
                            loadBalanceRule = RoundRobinLoadBalanceRule.getInstance(configRule.getServiceId());
                            break;
                        default:
                            log.warn("No loadBalance strategy for service:{}", strategy);
                            loadBalanceRule = RandomLoadBalanceRule.getInstance(configRule.getServiceId());
                            break;
                    }
                }
            }
        }
        return loadBalanceRule;
    }
}

```

So up to here we have successfully implemented the load balancing policy filter.
