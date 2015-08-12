---
layout: post
title: node.js 为请求加唯一标示
description: ""
modified: 2015-08-12
tags: [node.js,编程]
---

对外提供各种API服务，我们总是尽量保证服务的稳定和不出错，可是无论怎么做，都不能保证服务百分百可靠，所以出错后的调试和现场保留显得格外重要了

一个思路是：在用户的一次调用中，从请求到处理到结果返回整一个链路都保持同一个ID，把这个ID写入到请求参数、日志、返回结果、甚至是数据库里。

然后无论是服务器出错，还是产生结果与预期不符合，都能从最初请求参数到运行结果进行全程跟踪

那我们来看看这个ID需要什么元素

{% highlight html %}
1. 为了应对分布式负载均衡处理，这个ID需要能标示服务器地址;
2. 有些服务器上多进程启动服务，所以这个ID也需要能标示进程ID;
3. 需要能保证一个同一个服务下请求的ID不重复，另外还需要一个不会重复的随机数或者自增的数
{% endhighlight %}

每种编程语言应该都有相应的创建独立ID的模块，这里介绍node.js 的rid模块

[rid的地址](https://www.npmjs.com/package/rid)

用法非常简单
{% highlight javascript %}
{% raw %}
  var rid = require('rid') ;
  console.log(rid()) ;
{% endraw %}
{% endhighlight %}

如果你使用了express等前端框架，你可以写一个中间件将rid写进request里
{% highlight javascript %}
{% raw %}
  app.use(function(req,res,next){
    var r_id = rid() ;
    console.log(r_id) ;
    req.rid = r_id ;
    next() ;
  })
{% endraw %}
{% endhighlight %}

这样，在后边对所有的操作都很轻松的能够拿到这个rid，可以把他写到日志里和返回结果数据中，甚至是数据库里 。


