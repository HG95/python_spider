# CrawlSpider爬虫

在上一个糗事百科的爬虫案例中。我们是自己在解析完整个页面后获取下一页的url，然后重新发送一个请求。有时候我们想要这样做，只要满足某个条件的 url，都给我进行爬取。那么这时候我们就可以通过**`CrawlSpider`** 来帮我们完成了。**`CrawlSpider`** 继承自 `Spider`，只不过是在之前的基础之上增加了新的功能，可以定义爬取的url的规则，以后scrapy碰到满足条件的url都进行爬取，而不用手动的yield Request。


### 创建CrawlSpider爬虫：

之前创建爬虫的方式是通过`scrapy genspider [爬虫名字] [域名]`的方式创建的。如果想要创建CrawlSpider爬虫，那么应该通过以下命令创建：

```python
scrapy genspider -t crawl [爬虫名字] [域名]
```

```python
scrapy genspider -t crawl wxapp 'wxapp-union.com'
```

### LinkExtractors链接提取器：

使用`LinkExtractors`可以不用程序员自己提取想要的url，然后发送请求。这些工作都可以交给`LinkExtractors`，他会在所有爬的页面中找到满足规则的`url`，实现自动的爬取。

```python
class scrapy.linkextractors.LinkExtractor(
    allow = (),
    deny = (),
    allow_domains = (),
    deny_domains = (),
    deny_extensions = None,
    restrict_xpaths = (),
    tags = ('a','area'),
    attrs = ('href'),
    canonicalize = True,
    unique = True,
    process_value = None
)
```

主要参数讲解：

- allow：允许的url。所有满足这个正则表达式的url都会被提取。
- deny：禁止的url。所有满足这个正则表达式的url都不会被提取。
- allow_domains：允许的域名。只有在这个里面指定的域名的url才会被提取。
- deny_domains：禁止的域名。所有在这个里面指定的域名的url都不会被提取。
- restrict_xpaths：严格的xpath。和allow共同过滤链接。

### Rule规则类：

定义爬虫的规则类。

```python
class scrapy.spiders.Rule(
    link_extractor, 
    callback = None, 
    cb_kwargs = None, 
    follow = None, 
    process_links = None, 
    process_request = None
)
```

主要参数讲解：

- link_extractor：一个`LinkExtractor`对象，用于定义爬取规则。
- callback：满足这个规则的url，应该要执行哪个回调函数。因为`CrawlSpider`使用了`parse`作为回调函数，因此不要覆盖`parse`作为回调函数自己的回调函数。
- follow：指定根据该规则从response中提取的链接是否需要跟进。
- process_links：从link_extractor中获取到链接后会传递给这个函数，用来过滤不需要爬取的链接。



### 微信小程序社区CrawlSpider案例

初始的 爬虫文件

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class WxappSpiderSpider(CrawlSpider):
    name = 'wxapp_spider'
    allowed_domains = ['wxapp-union.com']
    start_urls = ["http://'wxapp-union.com'/"]

    rules = (
        Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        item = {}
        #item['domain_id'] = response.xpath('//input[@id="sid"]/@value').get()
        #item['name'] = response.xpath('//div[@id="name"]').get()
        #item['description'] = response.xpath('//div[@id="description"]').get()
        return item
```

分析每页(教程栏)的url 

```python
http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1
    
http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=2
```

对于每一页 只有最后面的数字不同

分析每个详情页的 url

```python
http://www.wxapp-union.com/article-5985-1.html

http://www.wxapp-union.com/article-6015-1.html

http://www.wxapp-union.com/article-6002-1.html
```

只有中间的四个数字不同，

修改 rules

```python
    rules = (
        # 匹配每页的url
        Rule(LinkExtractor(allow=r'.+mod=list&catid=2&page=\d'),  # 允许的匹配规则，可以是正则匹配
             follow=True),

        # 匹配详情页的 url
        Rule(LinkExtractor(allow=r'.+/article-.+\.html'),
             callback="parse_detail",  # 解析详情页的回调函数，字符串格式的
             follow=False  # 详情页不跟进
             )
    )
```

CrawlSpider

1. 需要使用 **`LinkExtractor`**  和**`Rule`** 决定爬虫的具体走向
2. **`allow`** 设置规则的方法：能够限制我们想得到的 url  ， 不要根其他的 url 产生相同的正则表达式即可
3. 什么情况下使用 **`follow`** : 如果在爬取页面的时候，需要满足当前的 url 再进行跟进，那么就设置为 True，否在设置为 False
4. 什么情况下指定 **`callback`**  ： 如果这个 url 对应的页面只是为了获取更多的 url, 并不需要页面的数据，那么可以不指定 **`callback`**, 如果想要获取 url 对应页面中的数据，那么就需要指定 **`callback`**







其他相关文件的设置和Spider 一样

