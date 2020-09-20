# Item Pipeline

当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。

每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。

以下是item pipeline的一些典型应用：

- 清理HTML数据
- 验证爬取的数据(检查item包含某些字段)
- 查重(并丢弃)
- 将爬取结果保存到数据库中

## 编写你自己的item pipeline

编写你自己的item pipeline很简单，每个item pipeline组件是一个独立的Python类，同时必须实现以下方法:

- `process_item`**(**self**,** *item***,** *spider***)**
  每个item pipeline组件都需要调用该方法，**这个方法必须返回一个 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item)** (或任何继承类)对象， 或是抛出 [`DropItem`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/exceptions.html#scrapy.exceptions.DropItem) 异常，被丢弃的item将不会被之后的pipeline组件所处理。
  参数:
  - **item** ([`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 对象) – 被爬取的item
  - **spider** ([`Spider`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider) 对象) – 爬取该item的spider

此外,他们也可以实现以下方法:

- `open_spider`(*self*, spider)
  当spider被开启时，这个方法被调用。
  参数:
  - **spider** ([`Spider`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider) 对象) – 被开启的spider

- `close_spider`(*spider*)
  当spider被关闭时，这个方法被调用
  - 参数:   **spider** ([`Spider`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/spiders.html#scrapy.spider.Spider) 对象) – 被关闭的spider

## Item pipeline 样例

验证价格，同时丢弃没有价格的item

让我们来看一下以下这个假设的pipeline，它为那些不含税(`price_excludes_vat` 属性)的item调整了 `price` 属性，同时丢弃了那些没有价格的item:

```python
from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)
```

### 将item写入JSON文件

以下pipeline将所有(从所有spider中)爬取到的item，存储到一个独立地 `items.jl` 文件，每行包含一个序列化为JSON格式的item:

```python
import json

class JsonWriterPipeline(object):

    def __init__(self):
        self.file = open('items.jl', 'wb')

    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item
```

### Write items to MongoDB

```python
import pymongo

class MongoPipeline(object):

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        collection_name = item.__class__.__name__
        self.db[collection_name].insert(dict(item))
        return item
```

## 启用一个Item Pipeline组件

```python
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
```

分配给每个类的整型值，确定了他们运行的顺序，item按数字从低到高的顺序，通过pipeline，通常将这些数字定义在0-1000范围内。



##### pipeline区分传来Items

各个页面都会封装items并将item传递给pipelines来处理，而pipelines接收的入口只有一个就是

```python
def process_item(self, item, spider):
	pass

```


spider对应相应的爬虫，调用`spider.name`也可区分来自不同爬虫的`item`

```python
def process_item(self, item, spider):
	if spider.name == "XXXX":
		pass


```

