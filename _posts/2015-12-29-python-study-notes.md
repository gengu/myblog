---
layout: post
title: python 只知其然
description: "python学习笔记"
modified: 2015-12-29
tags: [python]
---

python学习笔记，学到什么看到什么就写什么，并没有计划系统性的写什么东西，作为python新手，python的很多概念还是很有新意的，这里只做到只知其然，有时间再知其所以然。

# 1、参数传递

### python可变参数
{% highlight python %}
{% raw %}
def fn1(*c):
    fn2(*c)

def fn2(d,*c):
    print "d => " , d
    print "c => " , c

fn1('x','y','z')
{% endraw %}
{% endhighlight %}

> 返回结果  
> d =>  x  
> c =>  ('y', 'z')

* fn1 调用 fn2 并没有报参数数量不对的错误
* fn2 将fn1 传来的元组第一个元素付给了参数d，剩下的传给c元组


# 2、python修饰符


#### @property
假如一个函数被property修饰符修饰，那么这个函数就变成了一个属性，属性是可以被外界所访问
{% highlight python %}
{% raw %}
class C():
    def __init__(self):
        self.x = 'genxio'
    @property
    def x(self):
        return self.x
    @x.setter
    def x(self,y):
        print y
        self.x = y
if __name__ == "__main__":
    c = C()
    c.x = 'dddd';
    print c.x
{% endraw %}
{% endhighlight %}

> 被@property修饰过的函数，默认会产生一个setter方法，当然这个方法可以不写，那么这里限的x就变成了只读


#### @classmethod
类方法
{% highlight python %}
{% raw %}

class C:
    x = 1
    def test(self):
        C.x = 1
    def add(self):
        C.x = C.x + 1
    @classmethod
    def getX(cls):
        return cls.x

if __name__ == "__main__":
    c = C()
    c.add()
    print C.getX()
    print c.getX()

{% endraw %}
{% endhighlight %}

> 2  
> 2

这里C.getX()方法，是使用类名去调用静态方法，返回的是这个类中的x变量，不论是用对象来调用还是用类来调用，作用是一样的。这里就相当于是把C这个类当做cls参数传给了getX函数

#### @staticmethod
有时候，一个函数与这个类中的变量、其他函数关系不大，例如一些判断、类外的全局变量等。
加@staticmethod修饰符后，调用时不会向函数传递类或者类的实例

{% highlight python %}
{% raw %}

IND = 'c'
class C(object):
    def __init__(self, data):
        self.data = data
    # 如果这里的checkC函数没有self参数，那么 @staticmethod
    # 否则 加self参数
    @staticmethod
    def checkC():
        return (IND == 'c')
    def do_reset(self):
        if self.checkC():
            print('do_reset :', self.data)
    def set_db(self):
        if self.checkC():
            self.db = 'hhh'
        print('set_db : ', self.data)

if __name__ == "__main__":
    ik1 = C(12)
    ik1.do_reset()
    ik1.set_db()
{% endraw %}
{% endhighlight %}

	@classmethod means: when this method is called, we pass the class as the first argument instead of the instance of that class (as we normally do with methods). This means you can use the class and its properties inside that method rather than a particular instance.
	@staticmethod means: when this method is called, we don't pass an instance of the class to it (as we normally do with methods). This means you can put a function inside a class but you can't access the instance of that class (this is useful when your method does not use the instance).
	
# 变量的作用域
> 待完成
	


















