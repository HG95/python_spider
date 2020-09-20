# Items

爬取的主要目标就是从非结构性的数据源提取结构性数据，例如网页。 Scrapy提供 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 类来满足这样的需求。

[`Item`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Item) 对象是种简单的容器，保存了爬取到得数据。 其提供了 [类似于词典(dictionary-like)](http://docs.python.org/library/stdtypes.html#dict) 的API以及用于声明可用字段的简单语法。

## 声明Item

Item使用简单的class定义语法以及 [`Field`](https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/items.html#scrapy.item.Field) 对象来声明。例如:

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

### 创建item

```python
>>> product = Product(name='Desktop PC', price=1000)
>>> print product
Product(name='Desktop PC', price=1000)
```

### 获取字段的值

```python
>>> product['name']
Desktop PC
>>> product.get('name')
Desktop PC

>>> product['price']
1000
```



### 设置字段的值

```python
>>> product['last_updated'] = 'today'
>>> product['last_updated']
today
```

### 获取所有获取到的值

```python
>>> product.keys()
['price', 'name']

>>> product.items()
[('price', 1000), ('name', 'Desktop PC')]
```

