# Scrapy模拟人人网登录

# 模拟人人网登录

## 发送 post 请求

有时候想要请求数据的时候发送 post 请求，那么需要使用 Request 的子类 FormRequest 来实现，

如果想要爬虫一开始的时候就发送 post 请求，那么需要在爬虫类中重写 start_requests(self) 方法，

并且不在调用 start_urls 里的 url 



```python
# -*- coding: utf-8 -*-
# renren.py
import scrapy


class RenrenSpider(scrapy.Spider):
    name = 'renren'
    allowed_domains = ['renren.com']
    start_urls = ['http://renren.com/']

    def start_requests(self):
        # 人人网登录的接口
        url = "http://www.renren.com/PLogin.do"
        
        data = {"email": "1315152****", 'password': "hu******"}

        # 模拟登录
        request = scrapy.FormRequest(url, 
                                     formdata=data,
                                     callback=self.parse_page)
        yield request
		
    def parse_page(self, response):
        # 登录成功后访问个人主页
        url = "http://photo.renren.com/photo/972862448/albumlist/v7?offset=0&limit=40#"
        respons = scrapy.Request(url, 
                                 callback=self.parse_profile
                                )
        yield respons

    def parse_profile(self, response):
        # 将页面存储到本地
        with open('dp.html', 'w', encoding='utf-8') as fp:
            fp.write(response.text)
```

**`start_requests()`** 登录成功后 scrapy 会自动保存cookie 等信息。