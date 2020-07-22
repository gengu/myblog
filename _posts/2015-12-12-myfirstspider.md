---
layout: post
title: 用scrapy爬取汽车信息数据
description: ""
modified: 2015-12-12
tags: [学习,爬虫]
---



闲着没事，用爬虫技术爬取网页数据


	选择的语言是python，爬虫框架是scrapy，花了一天的时间学习python，一天时间学习scrapy，一天时间写代码。

[源代码地址](https://github.com/gengu/myFirstSpider)  
[目标网站是买车达人](http://topka.cn)

## 页面分析
1、如果要通过买车达人进行爬取，首先要找到所有汽车品牌信息入口，这个入口罗列了所有汽车品牌和汽车型号

	品牌入口：http://topka.cn/cars/brands
	
2、点进每个汽车型号之后，对每个汽车又有不同的配置信息

	例如奥迪Q7:http://topka.cn/cars/audi/q7

3、然后可以找到不同的车型
	
	例如奥迪Q7:http://topka.cn/cars/audi/q7/2016/26075

## 数据抓取
分析完之后，就可以通过scrapy进行简单的爬取了

{% highlight python %}  
{% raw %}
import scrapy
import re
import json

from myfirstspider.items import BrandItems
from myfirstspider.CarsItem import CarsItem
from myfirstspider.CarItem import CarItem
from scrapy.http import Request
from scrapy.utils.url import urljoin_rfc

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["topka.cn"]
    start_urls = [
        'http://topka.cn/cars/brands'
        # 'http://topka.cn/cars/audi/q7'
        # 'http://topka.cn/cars/audi/q7/2016/26075'
    ]
    def parse(self, response):
        p = re.compile(r'.*cars/brands$')
        p_cars = re.compile(r'http://topka.cn/cars/[\w\d]+/[\w\d]+$');
        p_car = re.compile(r'http://topka.cn/cars/[\w\d]+/[\w\d]+/[\d]+/[\d]+$')
        if p.match(response.url):
            for sel in response.xpath('//ul/li'):
                if sel.xpath('a/@href') != '/':
                    # title = sel.xpath('a/text()').extract()
                    link = sel.xpath('a/@href').extract()
                    item = BrandItems()
                    # item['title'] = title[0]
                    if(len(link) > 0):
                        item['link'] = str("http://topka.cn" + link[0])
                        if p_cars.match(item['link'].__str__()):
                            req = Request(url=item['link'],callback=self.parse)
                            yield req
        elif p_cars.match(response.url):
            t_sel = response.xpath('//*[@id="dLabel"]')
            title = t_sel[1].xpath('text()').extract()[1].replace(' ','')
            brand_sel = response.xpath('//button[@id="dLabel"]')
            brand = brand_sel[0].xpath('@title').extract()
            for sel in response.xpath('//tbody/tr/td/a'):
                if sel.xpath('@href') != '/':
                    link = sel.xpath('@href').extract();
                    item = CarsItem()
                    item['title'] = title[0]
                    item['brand'] = brand[0]
                    if(len(link) > 0):
                        item['link'] = str("http://topka.cn" + link[0])
                        if p_car.match(item['link'].__str__()):
                            req = Request(url = item['link'],callback=self.parse)
                            yield req
        elif p_car.match(response.url):
            t_sel = response.xpath('//div[@id="j_carsmenu"]')
            brand = t_sel.xpath('//div[@id="j_fupinpai"]/*[@id="dLabel"]').xpath('@title').extract()[0].replace(' ','').replace('\n','')
            model = t_sel.xpath('//div[@id="j_fuchexi"]/*[@id="dLabel"]').xpath('text()').extract()[0].replace(' ','').replace('\n','');
            model_v = response.xpath('//h1[@class="car-name"]/text()').extract()[0]
            item = CarItem()
            item['brand'] = brand
            item['model'] = model
            item['model_v'] = model_v
            p_sel = response.xpath('//div[@class="mod_pc param-configure-content"]')
            dict = {}
            for sel1 in p_sel.xpath('ul[@class="mod_pc_ul"]/li'):
                if(len(sel1.xpath('p')[1].xpath('text()').extract()) > 0):
                    dict[sel1.xpath('p')[0].xpath('text()').extract()[0]] = sel1.xpath('p')[1].xpath('text()').extract()[0]
                else :
                    dict[sel1.xpath('p')[0].xpath('text()').extract()[0]] = ""
                item['value'] = dict
            yield item

{% endraw %}   
{% endhighlight %}


## 数据处理
	逻辑实际就是不断的递归回调抓取页面，然后将真实的汽车信息从页面中提取出来存入一个json
	然后通过pipeline写入到aba.txt文件，不过由于python的编码问题，这里要使用sys进行编码指定，如代码
	
{% highlight python %}  
{% raw %}
import json
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )
class MyfirstspiderPipeline(object):
    def __init__(self):
        self.file = open('aba.txt','wb')

    def process_item(self, item, spider):
        if item:
            line = json.dumps(dict(item)) + "\n"
            print line
            self.file.write(line.decode('unicode_escape'))
            # item['brand'] = item['brand'][0].decode('UTF-8')
            # self.file.write(dict(item).__str__() + "\n");
        return item
{% endraw %}   
{% endhighlight %}


## 结果如图
![Mou icon](http://fangzhou.oss-cn-hangzhou.aliyuncs.com/myblog/maichedaren.png?Expires=1485895377=&Signature=EXK5tpboDfAwm%2BEa7T3/m3LnXVU%3D)

	
	
	






