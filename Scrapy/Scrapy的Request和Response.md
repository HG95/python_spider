# Scrapy的Request和Response

## 请求和响应

Scrapy的`Request` 和`Response`对象用于爬网网站。
通常，Request对象在爬虫程序中生成并传递到系统，直到它们到达下载程序，后者执行请求并返回一个Response对象，该对象返回到发出请求的爬虫程序。

两个类Request和Response类都有一些子类，它们添加基类中不需要的功能。这些在下面的请求子类和 响应子类中描述。

```python
class scrapy.http.Request(url[, 
                              callback, 
                              method='GET', 
                              headers, 
                              body, 
                              cookies, 
                              meta, 
							  encoding='utf-8', 
                              priority=0, 
                              dont_filter=False, 
                              errback]
                         )


```

一个Request对象表示一个HTTP请求，它通常是在爬虫生成，并由下载执行，从而生成Response。
常用参数：

- **`url`（string） - 此请求的网址**
- **`callback`（callable） - 将使用此请求的响应（一旦下载）作为其第一个参数调用的函数。有关更多信息，请参阅下面的将附加数据传递给回调函数。如果请求没有指定回调，parse()将使用spider的 方法。请注意，如果在处理期间引发异常，则会调用errback。**
- **`method`（string） - 此请求的HTTP方法。默认为’GET’。**
- **`meta（dict）` - 属性的初始值Request.meta。如果给定，在此参数中传递的dict将被浅复制。**
- headers（dict） - 这个请求的头。dict值可以是字符串（对于单值标头）或列表（对于多值标头）。如果 None作为值传递，则不会发送HTTP头。
- body（str或unicode） - 请求体。如果unicode传递了a，那么它被编码为 str使用传递的编码（默认为utf-8）。如果 body没有给出，则存储一个空字符串。不管这个参数的类型，存储的最终值将是一个str（不会是unicode或None）。
- cookie（dict或list） - 请求cookie。这些可以以两种形式发送。
- dont_filter（boolean） - 表示此请求不应由调度程序过滤。当您想要多次执行相同的请求时忽略重复过滤器时使用。小心使用它，或者你会进入爬行循环。默认为False。
- priority（int） - 此请求的优先级（默认为0）。调度器使用优先级来定义用于处理请求的顺序。具有较高优先级值的请求将较早执行。允许负值以指示相对低优先级。
- `encoding`（string） - 此请求的编码（默认为’utf-8’）。此编码将用于对URL进行百分比编码，并将正文转换为str（如果给定unicode）。

Request中meta参数的作用是传递信息给下一个函数，使用过程可以理解成：

```
把需要传递的信息赋值给这个叫meta的变量，
但meta只接受字典类型的赋值，因此
要把待传递的信息改成“字典”的形式，即：
meta={'key1':value1,'key2':value2}

如果想在下一个函数中取出value1,
只需得到上一个函数的meta['key1']即可，
因为meta是随着Request产生时传递的，
下一个函数得到的Response对象中就会有meta，
即response.meta，
取value1则是value1=response.meta['key1']

```

```python


class example(scrapy.Spider):
    name='example'
    allowed_domains=['example.com']
    start_urls=['http://www.example.com']
    
    def parse(self,response):
           #从start_urls中分析出的一个网址赋值给url
           url=response.xpath('.......').extract()
           #ExamleClass是在items.py中定义的,下面会写出。
           """记住item本身是一个字典"""
           item=ExampleClass()
           item['name']=response.xpath('.......').extract()
           item['htmlurl']=response.xpath('.......').extract()
           """通过meta参数，把item这个字典，赋值给meta中的'key'键（记住meta本身也是一个字典）。
           Scrapy.Request请求url后生成一个"Request对象"，这个meta字典（含有键值'key'，'key'的值也是一个字典，即item）
           会被“放”在"Request对象"里一起发送给parse2()函数 """
           yield Request(url,meta={'key':item},callback='parse2')
     
     def parse2(self,response):
           item=response.meta['key']
           """这个response已含有上述meta字典，此句将这个字典赋值给item，
           完成信息传递。这个item已经和parse中的item一样了"""
           item['text']=response.xpath('.......').extract()
           #item共三个键值，到这里全部添加完毕了
           yield item

```

meta是浅复制，必要时需要深复制。

```python
import copy
meta={'key':copy.deepcopy('value')}

```

meta是一个dict，主要是用解析函数之间传递值，一种常见的情况：在parse中给item某些字段提取了值，但是另外一些值需要在parse_item中提取，这时候需要将parse中的item传到parse_item方法中处理，显然无法直接给parse_item设置而外参数。 Request对象接受一个meta参数，一个字典对象，同时Response对象有一个meta属性可以取到相应request传过来的meta。所以解决上述问题可以这样做：

```python
def parse(self, response):
    # item = ItemClass()
    yield Request(url, meta={'item': item}, callback=self.parse_item)
def parse(self, response):
    item = response.meta['item']
    item['field'] = value
    yield item

```

#### Request和Response之间如何传参

有些时候需要将两个页面的内容合并到一个item里面，这时候就需要在yield scrapy.Request的同时，传递一些参数到一下页面中。这时候可以这样操作。

```python
        request=scrapy.Request(houseurl,method='GET',callback=self.showhousedetail)
        request.meta['biid']=biid
        yield request
 
 
    def showhousedetail(self,response):
        house=HouseItem()
        house['bulidingid']=response.meta['biid']

```



**`Response`** 对象

Response 对象一般是有 scrapy 自动构建的，因此不用关心如何构建，二十如何使用它，Response 有很多属性可以用来获取数据，主要有以下属性：

1. **`meta`** ： 其他请求传过来的 meta 属性，可以保持多个请求之间的数据连接
2. **`encoding`** : 返回当前字符串编码和解码的格式
3. **`text`** 将返回的数据作为 Unicode 字符返回
4. **`body`** : 将返回的字符串作为 bytes 字符串返回
5. **`xpath`** : xpath 选择器
6. **`css`** : css 选择器



## 发送 post 请求

有时候想要请求数据的时候发送 post 请求，那么需要使用 Request 的子类 FormRequest 来实现，

如果想要爬虫一开始的时候就发送 post 请求，那么需要在爬虫类中重写 start_requests(self) 方法，

并且不在调用 start_urls 里的 url 