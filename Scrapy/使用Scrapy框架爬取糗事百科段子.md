# 使用Scrapy框架爬取糗事百科段子

#### 使用命令创建一个项目

```python
scrapy startproject spider
```

cd 到 该项目，创建一个爬虫

```python
scrapy genspider qsbk "www.qiushibaike.com"
```

#### 爬虫代码解析

```python
# qsbk.py

class QsbkSpider(scrapy.Spider):
    name = 'qsbk'
    allowed_domains = ['www.qiushibaike.com']
    start_urls = ['https://www.qiushibaike.com/text/page/1/']

    def parse(self, response):
        pass
```

要创建一个Spider，那么必须自定义一个类，继承自`scrapy.Spider`，然后在这个类中定义三个属性和一个方法。

1. name：这个爬虫的名字，名字必须是唯一的。
2. allow_domains：允许的域名。爬虫只会爬取这个域名下的网页，其他不是这个域名下的网页会被自动忽略。
3. start_urls：爬虫从这个变量中的url开始。
4. parse：引擎会把下载器下载回来的数据扔给爬虫解析，爬虫再把数据传给这个`parse`方法。这个是个固定的写法。这个方法的作用有两个，第一个是提取想要的数据。第二个是生成下一个请求的url。

将 start_urls 改为自己想要爬取的页面的url



#### 修改`settings.py`代码

在做一个爬虫之前，一定要记得修改`setttings.py`中的设置。两个地方是强烈建议设置的。

1. `ROBOTSTXT_OBEY`设置为False。默认是True。即遵守机器协议，那么在爬虫的时候，scrapy首先去找robots.txt文件，如果没有找到。则直接停止爬取。
2. `DEFAULT_REQUEST_HEADERS`添加`User-Agent`。这个也是告诉服务器，我这个请求是一个正常的请求，不是一个爬虫。

#### 完成的爬虫代码

1. 爬虫部分代码：

```python
# -*- coding: utf-8 -*-
import scrapy

# 从根目录 导入 item 类
from spider.items import SpiderItem


class QsbkSpider(scrapy.Spider):
    name = 'qsbk'
    allowed_domains = ['www.qiushibaike.com']
    start_urls = ['https://www.qiushibaike.com/text/page/1/']

    def parse(self, response):
        contents = response.xpath('//div[@class="article block untagged mb15 typs_hot"]')
        print("*" * 30)
        for duanzi in contents:
            # get() 获取一个内容
            author = duanzi.xpath(".//h2/text()").get().strip()

            # getall() 获取所有内容，返回一个列表
            content = duanzi.xpath('.//div[@class="content"]//text()').getall()
            content = ' '.join(content).strip()

            # 另外一种传递数据的方式  创建一个 SpiderItem 对象
            item = SpiderItem(author=author, content=content)
            '''
            或者使用这种方式
            item = SpiderItem()
            item['author'] = author
            item['content'] = content
            '''
            yield item  # 将数据传到pipelines
```

- **`response`** 对象可以直执行 **`xpath `**语法来提取数据，如果想要获取里面的字符串，那么应该执行`getall()`或者`get()` 方法,

- **`getall()`** 方法获取所有文本，返回的是一个列表

- **`get()`** 方法获取第一个，返回的是一个 **`str`** 类型
- 如果数据解析回来，要传给 pipeline 处理，那么可以使用 `yield` 来返回，或者收集所有的最后使用一个 return 来返回, 


2. `items.py`部分代码：

```python
import scrapy


class SpiderItem(scrapy.Item):
    author = scrapy.Field()
    content = scrapy.Field()
```

**`item`** 建议在“items.py” 中定义好，

3. `pipeline`部分代码：

```python
import json

class SpiderPipeline:
    def __init__(self):
        # 打开文件 或者在open_spider() 中打开文件
        self.fp = open('duanzi.json','w',encoding='utf8')

    def open_spider(self,spider):
        pass
    def process_item(self, item, spider):
        # 采用 item 的方式  需要将其转化为字典
        item_json = json.dumps(dict(item), ensure_ascii=False)
        self.fp.write(item_json +'\n')
        return item


    def close_spider(self,spider):
        # 关闭文件
        self.fp.close()
```

其中 `def process_item(self, item, spider)` 方法是必须有的，别的虽需要可以自己添加





### 优化 json 数据存储方式 (一)

导入 `from  scrapy.exporters import JsonItemExporter`

```python
from scrapy.exporters import JsonItemExporter


class SpiderPipeline:
    def __init__(self):
        # 以 wb 的方式打开文件
        self.fp = open('duanzi.json', 'wb')
        # 创建对象
        self.exporter = JsonItemExporter(self.fp,
                                         ensure_ascii=False,
                                         encoding='utf8')
        # 开始
        self.exporter.start_exporting()

    def open_spider(self, spider):
        pass

    def process_item(self, item, spider):
        # 采用 item 的方式  需要将其转化为字典
        #item_json = json.dumps(dict(item), ensure_ascii=False)
        # self.fp.write(item_json + '\n')

        # 不再需要将 item 转化为字典
        self.exporter.export_item(item)

        return item

    def close_spider(self, spider):

        self.exporter.finish_exporting()
        # 关闭文件
        self.fp.close()

```

存储的结果 为一个列表，每个字典是列表的一项，当数据过大时，不推荐这种方式，因为其将整个 item z作为一项 导入文件的。

### 优化 json 数据存储方式 (二)

导入 `from scrapy.exporters import JsonLinesItemExporter`

```python
from scrapy.exporters import JsonLinesItemExporter


class SpiderPipeline:
    def __init__(self):
        # 以 wb 的方式打开文件
        self.fp = open('duanzi.json', 'wb')
        # 创建对象
        self.exporter = JsonLinesItemExporter(self.fp,
                                              ensure_ascii=False,
                                              encoding='utf8')

    def open_spider(self, spider):
        pass

    def process_item(self, item, spider):
        self.exporter.export_item(item)
        return item

    def close_spider(self, spider):
        # 关闭文件
        self.fp.close()

```

不需要开启和关闭，导入数据后的文件，仍然时一个字典一行

#### `JsonItemExporter` 和 `JsonLinesItemExporter` 

保存数据的时候使用这两个类，让操作变的更简单

1. `JsonItemExporter` 每次将数据添加到内存中，最后统一写入到磁盘，好处时，存储的数据是一个满足 json 规则的数据，坏处是如果数据量比较大，那么内存消耗严重
2. `JsonLinesItemExporter` 每次调用 `export_item` 的时候就把这个item 存储到硬盘中，坏处是每一个字典是一行，整个文件不是满足json格式的文件，好处是每次处理数据的时候直接存储到硬盘，这样不会耗内存，数据也比较安全。