---
layout: post
title: scrapy 源代码学习
description: "scrapy源代码学习"
modified: 2015-12-20
tags: [scrapy,爬虫,源代码]
---



如果想看一个框架的源代码，常见的思路首先对对框架所使用的语言有足够的了解，不过最近练习scrapy的过程中有了新的见解。

**如果有了其它语言的基础，就算对该语言的没有深刻的见解，读源码也不会遇到很大麻烦，相反，通过源代码学习，顺藤摸瓜，你会获得非常多书本上不会指导你的知识**

OK，开始

### 1. scrapy 源代码地址
<https://github.com/scrapy/scrapy/tree/master/scrapy> 

### 2. scrapy的打包
查看源代码，可以看到setup.py 文件，那么该文件应该就是打包配置文件了，有点儿类似于maven的package命令

> python setup.py install 
	
此刻会执行相关的setuptools > setup函数.   
相关资料 <http://blog.csdn.net/ponder008/article/details/6592719/>

从setup.py函数中可以看到非常多的信息。不过我只关心当我执行
> scrapy crawl sp_name 

指令的时候程序如何执行，也就是入口怎么样执行

于是看到了这个参数
{% highlight python %}  
{% raw %}
	entry_points={
        'console_scripts': ['scrapy = scrapy.cmdline:execute']
    },
{% endraw %}   
{% endhighlight %}   
这里应该就是我要的入口了，这一句清楚的告诉我，scrapy函数等同于执行cmdline的execute函数。

### 3. cmdline的execute函数
	1、加载配置，scrapy尝试从不同的地方加载配置文件 scrapy.conf 
{% highlight python %}  
{% raw %}
 # --- backwards compatibility for scrapy.conf.settings singleton ---
    if settings is None and 'scrapy.conf' in sys.modules:
        from scrapy import conf
        if hasattr(conf, 'settings'):
            settings = conf.settings

    if settings is None:
        settings = get_project_settings()
    check_deprecated_settings(settings)
{% endraw %}   
{% endhighlight %}   
	此处大致就是从各种系统模块、系统变量等各个地方收集信息
	
	2、_get_commands_dict()
	该方法将会从scrapy.commands下加载所有文件，那么scrapy.commands下的文件类似于一个个插件，大致看一下该文件夹下有crawl的py文件。可以理解成该文件夹下组成了一个个插件，只有scrapy启动时才会动态加载。
	
	3、
	查看list.py文件
{% highlight python %}  
{% raw %}
from __future__ import print_function
from scrapy.commands import ScrapyCommand

class Command(ScrapyCommand):

    requires_project = True
    default_settings = {'LOG_ENABLED': False}

    def short_desc(self):
        return "List available spiders"

    def run(self, args, opts):
        for s in sorted(self.crawler_process.spider_loader.list()):
            print(s)

{% endraw %}   
{% endhighlight %}  
	分析该代码，run方法是每个插件必须实现的方法，这个方法也必定是每个插件具体执行的入口。
	
	4、Crawl.py 插件，这应该是具体的爬虫执行代码，查看run方法
{% highlight python %}  
{% raw %}
	    def run(self, args, opts):
        if len(args) < 1:
            raise UsageError()
        elif len(args) > 1:
            raise UsageError("running 'scrapy crawl' with more than one spider is no longer supported")
        spname = args[0]

        self.crawler_process.crawl(spname, **opts.spargs)
        self.crawler_process.start()
{% endraw %}   
{% endhighlight %}  

	从代码中看到self.crawler_process.crawl()将开始请求startRequests并开始启动一个Engine。
	
	
本文不对scrapy 爬虫引擎的具体实现细节做过多讲解，留在下一章节中进行。
	




