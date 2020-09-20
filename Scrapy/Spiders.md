# Spiders

Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。 换句话说，Spider就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。

对spider来说，爬取的循环类似下文:

1. 以初始的URL初始化Request，并设置回调函数。 当该request下载完毕并返回时，将生成response，并作为参数传给该回调函数。

   spider中初始的request是通过调用 [`start_requests()`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider.start_requests) 来获取的。 [`start_requests()`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider.start_requests) 读取 [`start_urls`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider.start_urls) 中的URL， 并以 [`parse`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider.parse) 为回调函数生成 [`Request`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Request) 。

2. 在回调函数内分析返回的(网页)内容，返回 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 对象或者 [`Request`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Request) 或者一个包括二者的可迭代容器。 返回的Request对象之后会经过Scrapy处理，下载相应的内容，并调用设置的callback函数(函数可相同)。

3. 在回调函数内，您可以使用 [选择器(Selectors)](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/selectors.html#topics-selectors) (您也可以使用BeautifulSoup, lxml 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成item。

4. 最后，由spider返回的item将被存到数据库(由某些 [Item Pipeline](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/item-pipeline.html#topics-item-pipeline) 处理)或使用 [Feed exports](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/feed-exports.html#topics-feed-exports) 存入到文件中。



### Spider

```python
classscrapy.spider.Spider
```

Spider是最简单的spider。每个其他的spider必须继承自该类(包括Scrapy自带的其他spider以及您自己编写的spider)。 Spider并没有提供什么特殊的功能。 其仅仅请求给定的 `start_urls`/`start_requests` ，并根据返回的结果(resulting responses)调用spider的 `parse` 方法。

- **name**

定义spider名字的字符串(string)。spider的名字定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。 不过您可以生成多个相同的spider实例(instance)，这没有任何限制。 name是spider最重要的属性，而且是必须的。

- **allowed_domains**

可选。包含了spider允许爬取的域名(domain)列表(list)。 当 [`OffsiteMiddleware`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spider-middleware.html#scrapy.contrib.spidermiddleware.offsite.OffsiteMiddleware) 启用时， 域名不在列表中的URL不会被跟进。

- **start_urls**

URL列表。当没有制定特定的URL时，spider将从该列表中开始进行爬取。 因此，第一个被获取到的页面的URL将是该列表之一。 后续的URL将会从获取到的数据中提取。

- `parse`**(**response**)**
  当response没有指定回调函数时，该方法是 Scrapy 处理下载的response的默认方法。
  `parse` 负责处理response并返回处理的数据以及(/或)跟进的URL。 [`Spider`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider) 对其他的Request的回调函数也有相同的要求。
  该方法及其他的Request回调函数必须返回一个包含 [`Request`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Request) 及(或) [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 的可迭代的对象。

  参数:	**response** ([`Response`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Response)) – 用于分析的response  ，相当于request 库的返回值 可以执行 xpath 语法

#### Spider样例

```python
import scrapy

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        self.log('A response from %s just arrived!' % response.url)
```

另一个在单个回调函数中返回多个Request以及Item的例子:

```python
import scrapy
from myproject.items import MyItem

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        sel = scrapy.Selector(response)
        for h3 in response.xpath('//h3').extract():
            yield MyItem(title=h3)

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
```

### CrawlSpider

```python
classscrapy.contrib.spiders.CrawlSpider
```

爬取一般网站常用的spider。其定义了一些规则(rule)来提供跟进link的方便的机制。 也许该spider并不是完全适合您的特定网站或项目，但其对很多情况都使用。 因此您可以以其为起点，根据需求修改部分方法。当然您也可以实现自己的spider。

除了从Spider继承过来的(您必须提供的)属性外，其提供了一个新的属性:

- **rules**

一个包含一个(或多个) [`Rule`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.contrib.spiders.Rule) 对象的集合(list)。 每个 [`Rule`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.contrib.spiders.Rule) 对爬取网站的动作定义了特定表现。 Rule对象在下边会介绍。 如果多个rule匹配了相同的链接，则根据他们在本属性中被定义的顺序，第一个会被使用。

- `parse_start_url`**(**response**)**

当start_url的请求返回时，该方法被调用。 该方法分析最初的返回值并必须返回一个 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 对象或者 一个 [`Request`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Request) 对象或者 一个可迭代的包含二者对象。

#### 爬取规则(Crawling rules)

```python
classscrapy.contrib.spiders.Rule(link_extractor, 
                                 callback=None, 
                                 cb_kwargs=None, 
                                 follow=None, 
                                 process_links=None, 
                                 process_request=None)
```

- `link_extractor` 是一个 [Link Extractor](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/link-extractors.html#topics-link-extractors) 对象。 其定义了如何从爬取到的页面提取链接。

- `callback` 是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中每获取到链接时将会调用该函数。该回调函数接受一个response作为其第一个参数， 并返回一个包含 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 以及(或) [`Request`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/request-response.html#scrapy.http.Request) 对象(或者这两者的子类)的列表(list)。

- `cb_kwargs` 包含传递给回调函数的参数(keyword argument)的字典。

- `follow` 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。 如果 
- `callback` 为None， `follow` 默认设置为 `True` ，否则默认为 `False` 。

- `process_links` 是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中获取到链接列表时将会调用该函数。该方法主要用来过滤。

- `process_request` 是一个callable或string(该spider中同名的函数将会被调用)。 该规则提取到每个request时都会调用该函数。该函数必须返回一个request或者None。 (用来过滤request)

#### CrawlSpider样例

```python
import scrapy
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # 提取匹配 'category.php' (但不匹配 'subsection.php') 的链接并跟进链接(没有callback意味着follow默认为True)
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # 提取匹配 'item.php' 的链接并使用spider的parse_item方法进行分析
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.log('Hi, this is an item page! %s' % response.url)

        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item
```

该spider将从example.com的首页开始爬取，获取category以及item的链接并对后者使用 `parse_item` 方法。 当item获得返回(response)时，将使用XPath处理HTML并生成一些数据填入 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 中。