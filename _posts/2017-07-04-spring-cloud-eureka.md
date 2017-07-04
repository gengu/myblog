---
layout: post
title: Spring-cloud-eureka 初探
description: "搭建一个单机版eureka"
modified: 2017-07-04
tags: [spring-cloud系列 , eureka , 学习]
---

## 不忘初心
> 最近工作交接，比较清闲
> 在阿里已经工作五年了，五年那天，并没有什么特别的感受，只是有些事想的越来越清楚
> 在业务团队一晃好几年，虽然也一直在想着研究技术的深度，但总是不如我所愿
> 很幸运，还能想清楚自己真的是对技术有热情啊，放弃当下的所有，去技术团队吧，相信自己的选择

## eureka介绍
> Spring Cloud Eureka是Spring Cloud Netflix微服务套件的一部分，Spring Cloud通过为Eureka增加了Spring cloud的自动化配置。    

服务治理，可以说是微服务架构中最核心和基础的模块，它主要用来实现各个微服务示例的自动化注册和发现。   
作用是啥呢？在最开始我们使用微服务架构的时候，每个SpringBoot服务系统都提供模块化的功能，最初可能并不多，我们可以通过静态配置文件放到application.properties中来完成服务间的相互调用，但是随着业务越来越复杂，我们的配置文件会变得越来越难以维护，并且随着业务不断发展，我们的集群规模、服务位置都可能发生变化，如果还是用静态配置文件的来维护，将耗费大量的人力。  

> 为了解决微服务架构中示例治理问题，Eureka框架提供了以下能力     

1. 服务注册  
> 单个服务主动向治理中心注册自己的位置，将自己的IP、端口号、通信协议等通知给注册中心    

2. 服务发现  
> 服务与服务之间的调用不再需要指定对应的端口号和IP，只需要调用在Eureka上被注册的服务名即可  
> client调用其他人的时候，注册中心会将目标服务的地址列表返回给client，client随机抽取一个调用即可   

3. 高可用配置     
> 注册中心服务器也可以是集群，并提供自身健康保障能力

### Spring-Cloud 中文官网
<https://springcloud.cc/>
官网中介绍了比较流行的微服务治理框架，例如集群配置系统、事件驱动器、安全管控等等

### Spring-cloud-eureka配置
要运行eureka，至少需要两个项目，一个是注册中心，一个是客户端

#### 注册中心的配置和编写
核心MVN依赖
{% highlight xml %}  
{% raw %}
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Brixton.SR5</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<!-- 表示这是eureka-server中心 -->
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
</dependencies>

{% endraw %}   
{% endhighlight %}

核心配置文件application.yml
{% highlight java %}
{% raw %}
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false //因为是注册中心，所以不向远端注册自己的服务
    fetchRegistry: false //也不从远端拉取所有配置
  server:
waitTimeInMsWhenSyncEmpty: 0
{% endraw %}   
{% endhighlight %}

核心启动文件
{% highlight java %}
{% raw %}
@SpringBootApplication
//告诉服务自己是注册中心
@EnableEurekaServer  
public class EurekaApplication  
{
    public static void main( String[] args )
    {
    	SpringApplication.run(EurekaApplication .class, args);
    }
}
{% endraw %}   
{% endhighlight %}

然后启动项目，访问<http://localhost:8761>


#### 客户端的配置和编写
其实客户端和服务端的编写大同小异，Mvn的配置一样，仅仅只有StartApplication.java文件与application.yml文件的差异

核心启动文件
{% highlight java %}
{% raw %}
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaApplication  
{
    public static void main( String[] args )
    {
    	SpringApplication.run(EurekaApplication .class, args);
    }
}
{% endraw %}   
{% endhighlight %}

核心配置文件
{% highlight java %}
{% raw %}
server:
  port: 8762
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true //向注册中心注册自己的服务
    fetchRegistry: true //想注册中心拉取所有的配置
    service-url:
      defaultZone : http://localhost:8761/eureka/  //注册中心的地址
  server:
waitTimeInMsWhenSyncEmpty: 0
{% endraw %}   
{% endhighlight %}

#### 分别启动之后
访问<http://localhost:8761>
你就能看到，你的服务已经注册成功了







