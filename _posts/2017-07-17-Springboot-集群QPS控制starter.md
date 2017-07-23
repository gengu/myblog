---
layout: post
title: Spring-boot-集群QPS控制starter实现
description: "rate-limiter-spring-boot-starter"
modified: 2017-07-14
tags: [spring-boot系列 , redis ,starter]
---

## 功能
集群QPS控制，在方法上添加注解即可限制QPS，利用redis的key过期特性，可以很简单的实现集群QPS的控制

### 源码地址
[Github源码地址](https://github.com/gengu/rate-limiter-spring-boot-starter)

在使用上有任何问题，请提issue


## 使用方法

### 注解定义
{% highlight java %}  
{% raw %}
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Limiter {

    /**
     * 限流route
     * @return
     */
    String route();

    /**
     * 限流次数
     * @return
     */
    int limit() default 10;

}
{% endraw %}   
{% endhighlight %}

#### 实现原理
> 实现原理是，使用Redis的key过期规则，固定为每个route创建过期时间为1s的key，每次请求都为这个执行+1，并检测当前value的值是否大于limit，如果大于limit则抛出异常，否则什么也不做。


### 资源
> 在pom.xml中添加依赖

{% highlight xml %}  
{% raw %}
<dependency>
    <groupId>com.genxiaogu</groupId>
    <artifactId>rate-limiter-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
{% endraw %}   
{% endhighlight %}

> 在application.properties中添加redis资源
{% highlight python %}  
{% raw %}
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=8888
spring.redis.password=xxx
spring.redis.pool.max-active=50000
spring.redis.pool.max-wait=-1
spring.redis.pool.max-idle=10
spring.redis.pool.min-idle=0
spring.redis.timeout=500
{% endraw %}   
{% endhighlight %}

> 在需要进行流量控制的方法上添加注解例如
{% highlight java %}  
{% raw %}
在mapping方法上
@Limiter(route = "test", limit = 100)
@RequestMapping("/test")
public String check(){
    return dataPermissionTest.test() ;
}
或者在普通方法上
@Limiter(route = "test", limit = 100)
public String test(){
    return "test" ;
}
{% endraw %}   
{% endhighlight %}

***注意，Limiter的route参数指代控制方法的唯一key，limit表示1s内允许访问多少次***


## DEMO
[spring-boot-rate-limiter-demo](https://github.com/gengu/spring-boot-demos/tree/master/spring-boot-rate-limiter-demo)

## 使用siege测试


> 总结及测试
> > 我尝试把系统的QPS测到1

{% highlight bash %}  
{% raw %}
我们定义每秒限制为1次
@Limiter(route = "test", limit = 1)
然后使用siege进行测试一分钟
siege -c 5 -t 1 'http://localhost:8080/test'
{% endraw %}   
{% endhighlight %}

{% highlight bash %}  
{% raw %}
Transactions:		          59 hits
Availability:		       10.09 %
Elapsed time:		       59.61 secs
Data transferred:	        0.11 MB
Response time:		        0.05 secs
Transaction rate:	        0.99 trans/sec
Throughput:		        0.00 MB/sec
Concurrency:		        0.05
Successful transactions:          59
Failed transactions:	         526
Longest transaction:	        0.17
Shortest transaction:	        0.00
{% endraw %}   
{% endhighlight %}


## 2017-07-19 重要更新
在第一个版本中，为了避免多线程执行过程中出现变量可见性问题，使用了synchronized同步整个function
{% highlight java %}  
{% raw %}
public synchronized boolean execute() {
    BoundValueOperations<String, String> boundValueOps = redisTemplate.boundValueOps(route);
    if(boundValueOps.get() == null){
        boundValueOps.set("1") ;
        boundValueOps.expire(1000, TimeUnit.MILLISECONDS);
        return true ;
    }else{
        ...
    }
}
{% endraw %}   
{% endhighlight %}
这种处理方式能满足单机需求，如果是集群环境下，还是会出现redis的key不完全同步的问题
而且synchronized锁住了整个方法，非常消耗资源  
经过与朋友的商讨发现有更好的方式去解决，问题出现在
{% highlight java %}  
{% raw %}
    if(boundValueOps.get() == null){
        boundValueOps.set("1") ;
        boundValueOps.expire(1000, TimeUnit.MILLISECONDS);
        return true ;
    }
}
{% endraw %}   
{% endhighlight %}
在多核环境下，先拿到值然后判断，然后插入字符串1，无法保证程序执行的原子性
假如A线程执行完boundValueOps.get() == null，B线程已经插入了了"1"，这时候A再插入"1",就出现了bug，这是导致多核下线程不安全的主要原因

### 改造
***知道了问题产生的原因，就知道如何去解决***
{% highlight java %}  
{% raw %}
if(boundValueOps.setIfAbsent("1")){
    boundValueOps.expire(1000, TimeUnit.MILLISECONDS);
    return true ;
}
{% endraw %}   
{% endhighlight %}
这段代码中，将判断和插入合成到一个方法中——setIfAbsent，保证这两个操作的原子性
很好的解决了单机并发、集群并发环境下的线程安全问题

该版本已经更新到1.1.0

## 2017-07-23 重要更新
> 在第一个版本中，我们做了针对方法的限流，但是有人对不同用户也有流量控制的需求，所以我们新增UserLimiter注解，支持参数控制流量，使用方法与Limiter类似，但是注解需要解释在参数上  
> 例如假如user代表用户参数，我们希望单个用户每秒只能访问1次，可以使用如下方法：
{% highlight java %}  
{% raw %}
public String check3(@UserLimiter(route = "test3" , limit = 1) String user){
    return dataPermissionTest.test3() ;
}
{% endraw %}   
{% endhighlight %}


## TOList
[ ] 统计每个route的拦截次数以及通过次数  
[ ] 统计被拦截的IP地址信息等，预防DDOS很有用  
[ ] 现在抛异常的方案有点粗暴，可以定义接口由客户端来实现逻辑  
[√] 新增特性针对不同用户进行进行流量控制