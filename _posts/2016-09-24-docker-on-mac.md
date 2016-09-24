---
layout: post
title: 在mac上搭建Docker开发环境
description: "在MAC上进行docker开发"
modified: 2016-09-24
tags: [docker]
---

# 在mac上搭建Docker开发环境

> 闲来无事，折腾下docker,花了一天时间看教程和文档，记录如下

> 我的开发机器是MACBook pro，docker之前对mac的支持不太好，不过现在已经有了非常棒的方案。
> 本文将在本地环境搭建docker开发环境的步骤记录下来，本文不会对Docker的基本概念做详细解释
有兴趣可以参考：https://www.gitbook.com/book/yeasy/docker_practice/details

## 环境
我的机器配置

![本地环境](http://fangzhou.oss-cn-hangzhou.aliyuncs.com/myblog/lALOdRU_u80BYs0CSg_586_354.png_620x10000q90g.jpg)


## 安装环境

> 1、如果你本地安装了brew，这一步请略过

{% highlight bash %}  
{% raw %}  
	curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1
{% endraw %}   
{% endhighlight %}   


> 2、安装docker-machine和docker

{% highlight bash %}  
{% raw %}  
	 brew install docker-machine docker
{% endraw %}   
{% endhighlight %}  

    docker-machine 是用于在MAC上安装容器的虚拟机
    docker是客户端
    安装完成之后可以使用命令
    docker-machine version
    docker version 检查是不是安装正确


> 3、安装VirtualBox

[VirtualBox下载链接](http://download.virtualbox.org/virtualbox/5.1.6/VirtualBox-5.1.6-110634-OSX.dmg)

  下载完成之后直接双击打开安装，解释一下，这个是用于在docker-machine 创建虚拟机时作为驱动使用的


## 创建Docker虚拟机

    1、创建虚拟机  

    docker-machine create --driver virtualbox default  

    2、配置docker client，告诉client接下来操作『default』这个虚拟机

    docker-machine env default

    3、将本地的bash环境与虚拟机进行连接

    eval "$(docker-machine env default)"

> 以为上述环境只在当前进程terminal环境有效，所以我们可以将2、3命令写入到/etc/profile 或者 ~/.zshrc 这样就可以在新的terminal进程启动时自动配置docker-client


## 用Docker启动一个nginx容器

> 我们虽然并没有安装nginx镜像，但是没关系，docker run 会首先检查你本地有没有安装nginx，如果没有就会帮你从镜像仓库安装一个最新稳定版本的nginx镜像

    docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx

> 创建一个Dockerfile文件

    touch Dockerfile


> 创建文件夹static-html-directory

    mkdir test-nginx && cd test-nginx && echo "foo bar" > index.html && cd ../

> 在Dockerfile 中写入以下内容

{% highlight bash %}  
{% raw %}  
  FROM nginx
  COPY test-nginx /usr/share/nginx/html
{% endraw %}   
{% endhighlight %}



> 创建容器

    docker build -t test-nginx .

>  启动容器

    docker run --name my-docker-nginx -p 8088:80 -d test-nginx  

> 验证

  result=`docker-machine ssh default 'curl localhost:8088'`
  echo $result

![结果图片](http://fangzhou.oss-cn-hangzhou.aliyuncs.com/myblog/aaaaaa.jpg)

> 当你看到上面图片的时候就表名，你的MAC 本地容器部署成功了

## 总结
> 本文没有对具体的docker进行深入的解读，有几点需要说明以下

> > 1、在本文中，你的docker命令其实都是作用在你创建的虚拟机上的

> > 2、docker-machine ssh default这个命令可以让你顺利的登陆到你的虚拟机上，和你打开virturlbox进入虚拟机是一样的效果

> > 3、你的容器是运行在虚拟机上，和你本地的环境没有关系

> > 4、-p 8088:80表示是将你虚拟机的8088端口指向test-nginx这个容器的80端口上，所以localhost:8088访问的其实是test-nginx容器上的80端口的index.html

> > 5、在你自己的MAC机器上是没有办法访问到容器的服务，具体原因可能是虚拟机没有对外的网卡


### 参考
  docker-machine 的使用  
  https://docs.docker.com/machine/get-started/

  使用docker-nginx 镜像  
  https://hub.docker.com/_/nginx/

  docker中文文档  https://yeasy.gitbooks.io/docker_practice/content/container/run.html
