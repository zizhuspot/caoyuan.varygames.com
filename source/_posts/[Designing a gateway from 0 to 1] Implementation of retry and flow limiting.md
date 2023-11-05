---
title: [Designing a gateway from 0 to 1] Implementation of retry and flow limiting
date: 2023-11-05 18:12:00
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
The above has already covered how to design a highly available and stable gateway, so here are two of the more common ways to implement it.

# Retries

In this case, I'm going to do a request retry for IO exceptions and request timeouts. First, we'll add a retry function to the route filter that will retry the request in case of exceptions like the two above. Of course, we need to add some additional configuration parameters to set the number of retries and other information.

![](https://s2.loli.net/2023/11/05/V72Zx1GXlOErW3v.webp) And the retry code, in fact, calls the doFilter method again to execute the logic in the route filter

```css
 private void doRetry(GatewayContext gatewayContext,int retryTimes){
        System.out.println("当前重试次数为"+retryTimes);
        gatewayContext.setCurrentRetryTimes(retryTimes+1);
        try {
            doFilter(gatewayContext);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

Finally, we have our service request the backend service and set a long blocking sleep where the backend service is.![](https://s2.loli.net/2023/11/05/FPcA8LywVoCEpRJ.webp) The simple implementation of retries is relatively straightforward, provided of course that you understand all the previous code. Or at least understand the request forwarding piece of code

# Flow limiting

Common algorithms for limiting flow are the token bucket algorithm and the leaky bucket algorithm. Here we can consider using both algorithms. At the same time, we need to configure the flow restriction rules in the configuration center. For example, the path or service to limit the flow. Also, depending on whether your service is a distributed service or a monolithic service, you also need to consider using different ways to store information. For example, if it is a distributed service, you need to use Redis, while if it is a monolithic, then consider using a local cache can be used, such as Guava or Caffeine. As usual, we first write an interface, which is used to get the corresponding flow restriction filter.

```css
public interface GatewayFlowControlRule {

    void doFlowControlFilter(Rule.FlowControlConfig flowControlConfig, String serviceId);
}

```

After that, we start writing a flow limiting filter to get the corresponding flow limiting rules based on the request.

```css

@Slf4j
@FilterAspect(id=FLOW_CTL_FILTER_ID,
        name = FLOW_CTL_FILTER_NAME,
        order = FLOW_CTL_FILTER_ORDER)
public class FlowControlFilter implements Filter {
    @Override
    public void doFilter(GatewayContext ctx) throws Exception {
        Rule rule = ctx.getRule();
        if(rule != null){
            //获取流控规则
            Set.FlowControlConfig> flowControlConfigs = rule.getFlowControlConfigs();
            Iterator iterator = flowControlConfigs.iterator();
            Rule.FlowControlConfig flowControlConfig;
            while (iterator.hasNext()){
                GatewayFlowControlRule flowControlRule = null;
                flowControlConfig = (Rule.FlowControlConfig)iterator.next();
                if(flowControlConfig == null){
                    continue;
                }
                String path = ctx.getRequest().getPath();
                if(flowControlConfig.getType().equalsIgnoreCase(FLOW_CTL_TYPE_PATH)
                        && path.equals(flowControlConfig.getValue())){
                    flowControlRule = FlowControlByPathRule.getInstance(rule.getServiceId(),path);
                }else if(flowControlConfig.getType().equalsIgnoreCase(FLOW_CTL_TYPE_SERVICE)){
                    //TODO 可以自己实现基于服务的流控
                }
                if(flowControlRule != null){
                    //执行流量控制
                    flowControlRule.doFlowControlFilter(flowControlConfig,rule.getServiceId());
                }
            }
        }
    }
}

```

Once you've got the specified flow limiting rules, you can start thinking about getting down to writing how to do the specific flow limiting. For example, if we want to restrict the flow based on the path, the first information we need is the service and the request path. The first thing we need to know is the service and the request path, and we need to save the rules so that whenever a request comes in, we get the rules from the cache.

```java

    private static ConcurrentHashMap servicePathMap = new ConcurrentHashMap<>();

    public static FlowControlByPathRule getInstance(String serviceId, String path) {
        StringBuffer buffer = new StringBuffer();
        String key = buffer.append(serviceId).append(".").append(path).toString();
        FlowControlByPathRule flowControlByPathRule = servicePathMap.get(key);

        if (flowControlByPathRule == null) {
            flowControlByPathRule = new FlowControlByPathRule(serviceId, path, new RedisCountLimiter(new JedisUtil()));
            servicePathMap.put(key, flowControlByPathRule);
        }
        return flowControlByPathRule;
    }
```

After obtaining the flow limiting rules, we can start to analyze how to carry out specific flow limiting methods. We get to the specified flow limit configuration, such as whether the service is distributed, whether the flow limit time and limit the number of times and so on, after the information, you can begin to write a specific flow limit code. For example, if the configuration found that the service is distributed, then use Redis, and then save the current request path and limit the number of times and other information.

```java

    public  boolean doFlowControl(String key,int limit,int expire){
        try {

            Object object = jedisUtil.executeScript(key,limit,expire);
            if(object == null){
                return true;
            }
            Long result = Long.valueOf(object.toString());
            if(FAILED_RESULT == result){
                return  false;
            }
        }catch (Exception e){
            throw  new RuntimeException("分布式限流发生错误");
        }
        return true;
    }

 public Object executeScript(String key, int limit, int expire){
        Jedis jedis = jedisPool.getJedis();
        String lua = buildLuaScript();
        String scriptLoad =jedis.scriptLoad(lua);
        try {
            Object result = jedis.evalsha(scriptLoad, Arrays.asList(key), Arrays.asList(String.valueOf(expire), String.valueOf(limit)));
            System.out.println(result);
            return result;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (jedis != null) {
                try {
                    jedis.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }

    private static String buildLuaScript() {
        String lua = "local num = redis.call('incr', KEYS[1])\n" +
                "if tonumber(num) == 1 then\n" +
                "\tredis.call('expire', KEYS[1], ARGV[1])\n" +
                "\treturn 1\n" +
                "elseif tonumber(num) > tonumber(ARGV[2]) then\n" +
                "\treturn 0\n" +
                "else \n" +
                "\treturn 1\n" +
                "end\n";
        return lua;
    }
```

And if it is not a distributed project, you can consider using a local cache like Guava. The implementation is pretty much the same, as follows

```java

public class GuavaCountLimiter {

    private RateLimiter rateLimiter;

    private double maxPermits;

    public GuavaCountLimiter(double maxPermits) {
        this.maxPermits = maxPermits;
        rateLimiter = RateLimiter.create(maxPermits);
    }

    public GuavaCountLimiter(double maxPermits, long warmUpPeriodAsSecond) {
        this.maxPermits = maxPermits;
        rateLimiter = RateLimiter.create(maxPermits, warmUpPeriodAsSecond, TimeUnit.SECONDS);
    }

    public static ConcurrentHashMap resourceRateLimiterMap = new ConcurrentHashMap();

    public static GuavaCountLimiter getInstance(String serviceId, Rule.FlowControlConfig flowControlConfig) {
        if (StringUtils.isEmpty(serviceId) || flowControlConfig == null || StringUtils.isEmpty(flowControlConfig.getValue()) || StringUtils.isEmpty(flowControlConfig.getConfig()) || StringUtils.isEmpty(flowControlConfig.getType())) {
            return null;
        }
        StringBuffer buffer = new StringBuffer();
        String key = buffer.append(serviceId).append(".").append(flowControlConfig.getValue()).toString();
        GuavaCountLimiter countLimiter = resourceRateLimiterMap.get(key);
        if (countLimiter == null) {

            Map configMap = JSON.parseObject(flowControlConfig.getConfig(), Map.class);

            if (!configMap.containsKey(FLOW_CTL_LIMIT_DURATION) || !configMap.containsKey(FLOW_CTL_LIMIT_PERMITS)) {
                return null;
            }

            double permits = configMap.get(FLOW_CTL_LIMIT_PERMITS);
            countLimiter = new GuavaCountLimiter(permits);
            resourceRateLimiterMap.putIfAbsent(key, countLimiter);
        }
        return countLimiter;
    }

    public boolean acquire(int permits) {
        boolean success = rateLimiter.tryAcquire(permits);
        if (success) {
            return true;
        }
        return false;
    }
}

```

So up to this point, the code for how to do flow limiting has been roughly implemented. After that, we can start testing our flow-limiting code once we've configured the configuration center information.

```js
{
    "rules": [
        {
            "id":"1",
            "name":"test-1",
            "protocol":"http",
            "serviceId":"backend-http-server",
            "prefix":"/user",
            "paths":[
                "/http-server/ping","/user/update"
            ],
            "filterConfigs":[{
                    "id":"load_balance_filter",
                    "config":{
                        "load_balance":"Random"
                    }
                },{
                    "id":"flow_ctl_filter"
            }],
            "flowControlConfigs":[{
                "type":"path",
                "model":"distributed",
                "value":"/http-server/ping",
                "config":{
                    "duration":20,
                    "permits":2
                }
            }],
            "retryConfig":{
                "times":5
            },
            "hystixConfigs":[{
                "path":"/http-server/ping",
                "timeoutInMilliseconds":5000,
                "threadCoreSize":2,
                "fallbackResponse":"熔断超时"
            }]
        }
    ]
}
```

When we send an excessive number of requests using apifox, we see that the error is reported as follows![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/870b58d26417474290058db5e27e5f0e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2044&h=414&s=183734&e=png&b=fefdfd) Up to the current position we have implemented retry and flow limiting, then in the next article we need to implement fusing and degrading. Because actually flow limiting and fusing degradation go together.
