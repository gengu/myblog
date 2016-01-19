[简单的github代码地址，此处要高亮](https://github.com/gengu/jiandan-spider)

基于scrapy的搜索引擎开发 —— 煎蛋爬虫

### 核心功能
  * 可配置数据抓取规则
  * 动态添加抓取源
  * 分布式抓取
  * 自定义爬取方式，js页面加载等

### 参考
  使用scrapy_redis作为基础模块，参考了三个项目，谢谢这些作者的分享

  * https://github.com/gnemoug/distribute_crawler
  * https://github.com/rolando/scrapy-redis
  * https://github.com/brandicted/scrapy-webdriver

### 特点
  * 分布式调度，获取的请求HttpRequest实例化至redis
  * 分布式存储，使用mongodb结构化存储
  * 使用phantomjs，对js渲染页面提供支持

### 组件安装步骤
1、安装redis
  [redis链接](!http://redis.io/download)

2、安装mongodb
  [mongodb链接](!https://www.mongodb.org/downloads)

3、安装phantomjs，由于phantomjs已经成为node.js的一个module，因此需要使用node.js的安装工具npm

> npm -g install phantomjs


### 配置
由于scrapy具有一个我认为设计非常精妙的地方任何组件都具有from_crawler和from_settings函数，所以自己写的插件可以很方便的从settings.py中拿到配置，并可以自己定义配置，
爬虫配置统一在settings.py文件，

{% highlight python %}
{% raw %}
"配置下载中间件，第一个是随机user-agent，第二个是使用phantomjs进行js渲染"
DOWNLOADER_MIDDLEWARES = {
    'jiandan.download.UserAgentMiddler.RotateUserAgentMiddleware': 50,
    'jiandan.download.GhostDownLoadMiddler.DownloadMiddler':600
}
"过滤器"
DUPEFILTER_CLASS = 'jiandan.scrapy-redis.dupefilter.RFPDupeFilter'
"redis服务"
REDIS_HOST = ''
REDIS_PORT =
"mongodb服务"
MONGODB_HOST = ''
MONGODB_PORT =
MONGODB_DB = ''
"phantomjs的安装本地路径，使用npm -g安装可以不用配置"
PHANTOMJS_PATH = '/Users/genxiaogu/GitLab/nvm/v0.11.14/bin/phantomjs'
"phantomjs的配置"
PHANTOMJS_CONF = "--load-images=false,--disk-cache=true"
{% endraw %}
{% endhighlight %}

拿到这些配置可以在插件中这样写，以downloadmiddler为例

{% highlight python %}
{% raw %}
class DownloadMiddler(object):
    def __init__(self,driver):
        self.driver = driver
        pass

    @classmethod
    def from_crawler(cls, crawler):
        instance = cls.from_setting(crawler.settings) ;
        return instance

    @classmethod
    def from_setting(cls,settings):
        path = settings.get('PHANTOMJS_PATH','/Users/genxiaogu/GitLab/nvm/v0.11.14/bin/phantomjs')
        if os.path.exists(path):
            pass
        else:
            raise PhantomjsError("phantomjs path not exists")
        phantomjs_conf = settings.get('PHANTOMJS_CONF')
        if phantomjs_conf != '' and phantomjs_conf is not None:
            conf = phantomjs_conf.split(',') ;
        driver = webdriver.PhantomJS(service_args=conf,executable_path=path)
        return cls(driver);

{% endraw %}
{% endhighlight %}


### 启动
> scrapy crawl ctrip