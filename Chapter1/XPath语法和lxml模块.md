# XPath语法和lxml模块

## 什么是XPath？

xpath（XML Path Language）是一门在XML和HTML文档中查找信息的语言，可用来在XML和HTML文档中对元素和属性进行遍历。

## XPath语法

### 选取节点：

XPath 使用路径表达式来选取 XML 文档中的节点或者节点集。这些路径表达式和我们在常规的电脑文件系统中看到的表达式非常相似。

|  表达式  |                             描述                             |       示例       | 结果                            |
| :------: | :----------------------------------------------------------: | :--------------: | ------------------------------- |
| nodename |                    选取此节点的所有子节点                    |    bookstore     | 选取bookstore下所有的子节点     |
|   `/`    | 如果是在最前面，代表从根节点选取。否则选择某节点下的某个节点 |   `/bookstore`   | 选取根元素下所有的bookstore节点 |
|   `//`   |             从全局节点中选择节点，随便在哪个位置             |     `//book`     | 从全局节点中找到所有的book节点  |
|   `@`    |                      选取某个节点的属性                      | `//book[@price]` | 选择所有拥有price属性的book节点 |
|   `.`    |                           当前节点                           |      `./a`       | 选取当前节点下的a标签           |

### 谓语：

谓语用来查找某个特定的节点或者包含某个指定的值的节点，被嵌在方括号中。
在下面的表格中，我们列出了带有谓语的一些路径表达式，以及表达式的结果：

<img src="https://raw.githubusercontent.com/HG1227/image/master/img_tuchuang/20200526105258.png"/>

### 通配符

`*`表示通配符

| 通配符 |         描述         |      示例      |             结果              |
| :----: | :------------------: | :------------: | :---------------------------: |
|  `*`   |     匹配任意节点     | `/bookstore/*` | 选取bookstore下的所有子元素。 |
|  `@*`  | 匹配节点中的任何属性 |  `//book[@*]`  | 选取所有带有属性的book元素。  |

### 选取多个路径：

通过在路径表达式中使用 “|” 运算符，可以选取若干个路径。

```python
//bookstore/book | //book/title
# 选取所有book元素以及book元素下所有的title元素

```

## lxml库

lxml 是 一个HTML/XML的解析器，主要的功能是如何解析和提取 HTML/XML 数据。

lxml和正则一样，也是用 C 实现的，是一款高性能的 Python HTML/XML 解析器，我们可以利用之前学习的XPath语法，来快速的定位特定元素以及节点信息。

### 基本使用：

解析HTML代码，并且在解析HTML代码的时候，如果HTML代码不规范，他会自动的进行补全。示例代码如下：

```
# 使用 lxml 的 etree 库
from lxml import etree 

text = '''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
     </ul>
 </div>
'''

```

```python

#利用etree.HTML，将字符串解析为HTML文档
html = etree.HTML(text) 

# 按字符串序列化HTML文档
result = etree.tostring(html) 

print(result)
输入结果如下：

<html><body>
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
</ul>
 </div>
</body></html>

```

可以看到。lxml会自动修改HTML代码。例子中不仅补全了li标签，还添加了body，html标签。



### 从文件中读取html代码

除了直接使用字符串进行解析，lxml还支持从文件中读取内容。我们新建一个hello.html文件：

```
<!-- hello.html -->
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>

```

然后利用 `etree.parse()`方法来读取文件。示例代码如下：

 ```python
from lxml import etree

# 读取外部文件 hello.html
html = etree.parse('hello.html')
result = etree.tostring(html, pretty_print=True)

print(result)

 ```

## 在lxml中使用XPath语法：

### 读取文本解析节点

```python
from lxml import etree

text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0"><a href="link5.html">a属性</a>
     </ul>
 </div>
'''
html=etree.HTML(text) #初始化生成一个XPath解析对象
result=etree.tostring(html,encoding='utf-8')   #解析对象输出代码
print(type(html))
print(type(result))
print(result.decode('utf-8'))

#etree会修复HTML文本节点
<class 'lxml.etree._Element'>
<class 'bytes'>
<html><body><div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0"><a href="link5.html">a属性</a>
     </li></ul>
 </div>
</body></html>
```

### 读取HTML文件进行解析

```python
from lxml import etree

html=etree.parse('test.html',etree.HTMLParser()) #指定解析器HTMLParser会根据文件修复HTML文件中缺失的如声明信息
result=etree.tostring(html)   #解析成字节
#result=etree.tostringlist(html) #解析成列表
print(type(html))
print(type(result))
print(result)

#
<class 'lxml.etree._ElementTree'>
<class 'bytes'>
b'<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">\n<html><body><div>
\n    <ul>
\n         <li class="item-0"><a href="link1.html">first item</a></li>
\n         <li class="item-1"><a href="link2.html">second item</a></li>
\n         <li class="item-inactive"><a href="link3.html">third item</a></li>
\n         <li class="item-1"><a href="link4.html">fourth item</a></li>
\n         <li class="item-0"><a href="link5.html">fifth item</a>
\n     </li></ul>
\n </div>
\n</body></html>'
```

### 获取所有节点

返回一个列表每个元素都是Element类型，所有节点都包含在其中

```python
from lxml import etree

html=etree.parse('test',etree.HTMLParser())
result=html.xpath('//*')  #//代表获取子孙节点，*代表获取所有

print(type(html))
print(type(result))
print(result)

#
<class 'lxml.etree._ElementTree'>
<class 'list'>
[<Element html at 0x754b210048>, <Element body at 0x754b210108>, <Element div at 0x754b210148>, <Element ul at 0x754b210188>, <Element li at 0x754b2101c8>, <Element a at 0x754b210248>, <Element li at 0x754b210288>, <Element a at 0x754b2102c8>, <Element li at 0x754b210308>, <Element a at 0x754b210208>, <Element li at 0x754b210348>, <Element a at 0x754b210388>, <Element li at 0x754b2103c8>, <Element a at 0x754b210408>]
```

如要获取li节点，可以使用//后面加上节点名称，然后调用xpath()方法

```python
html.xpath('//li')   #获取所有子孙节点的li节点
```

### 获取子节点

通过/或者//即可查找元素的子节点或者子孙节点，如果想选择li节点的所有直接a节点，可以这样使用

```python
result=html.xpath('//li/a')  
#通过追加/a选择所有li节点的所有直接a节点，
# 因为//li用于选中所有li节点，/a用于选中li节点的所有直接子节点a
```

### 属性匹配

在选取的时候，我们还可以用`@`符号进行属性过滤。比如，这里如果要选取`class`为`item-1`的`li`节点，可以这样实现:

```python
from lxml import etree
from lxml.etree import HTMLParser
text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]')
print(result)
```

### 文本获取

我们用XPath中的text()方法获取节点中的文本

```python
from lxml import etree

text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]/a/text()') #获取a节点下的内容
result1=html.xpath('//li[@class="item-1"]//text()') #获取li下所有子孙节点的内容

print(result)
print(result1)
```

### 属性获取

使用@符号即可获取节点的属性，如下：获取所有li节点下所有a节点的href属性

```python
result=html.xpath('//li/a/@href')  #获取a的href属性
result=html.xpath('//li//@href')   #获取所有li子孙节点的href属性
```

### 属性多值匹配

如果某个属性的值有多个时，我们可以使用contains()函数来获取

```python
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa item-0"><a href="link1.html">第一个</a></li>
         <li class="bbb item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[@class="aaa"]/a/text()')
result1=html.xpath('//li[contains(@class,"aaa")]/a/text()')

print(result)
print(result1)

#通过第一种方法没有取到值，通过contains（）就能精确匹配到节点了
[]
['第一个']
```

### 多属性匹配

另外我们还可能遇到一种情况，那就是根据多个属性确定一个节点，这时就需要同时匹配多个属性，此时可用运用and运算符来连接使用：

```python
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="fore"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[@class="aaa" and @name="fore"]/a/text()')
result1=html.xpath('//li[contains(@class,"aaa") and @name="fore"]/a/text()')


print(result)
print(result1)


#
['second item']
['second item']
```

### 按序选择

有时候，我们在选择的时候某些属性可能同时匹配多个节点，但我们只想要其中的某个节点，如第二个节点或者最后一个节点，这时可以利用中括号引入索引的方法获取特定次序的节点：

```python
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第二个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第三个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第四个</a></li> 
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())

result=html.xpath('//li[contains(@class,"aaa")]/a/text()') #获取所有li节点下a节点的内容
result1=html.xpath('//li[1][contains(@class,"aaa")]/a/text()') #获取第一个
result2=html.xpath('//li[last()][contains(@class,"aaa")]/a/text()') #获取最后一个
result3=html.xpath('//li[position()>2 and position()<4][contains(@class,"aaa")]/a/text()') #获取第一个
result4=html.xpath('//li[last()-2][contains(@class,"aaa")]/a/text()') #获取倒数第三个


print(result)
print(result1)
print(result2)
print(result3)
print(result4)


#
['第一个', '第二个', '第三个', '第四个']
['第一个']
['第四个']
['第三个']
['第二个']
```

这里使用了last()、position()函数，在XPath中，提供了100多个函数，包括存取、数值、字符串、逻辑、节点、序列等处理功能，它们的具体作用可参考：http://www.w3school.com.cn/xpath/xpath_functions.asp

## lxml结合xpath注意事项：

- 使用`xpath`语法。应该使用`Element.xpath`方法。来执行xpath的选择。示例代码如下

```python
trs = html.xpath("//tr[position()>1]")


```

`xpath函数`返回来的永远是一个列表。

- 获取某个标签的属性：

```python
href = html.xpath("//a/@href")	# 获取a标签的href属性对应的值
```

若再某个标签下  也可以直接用`get()函数`

```python

html = etree.HTML(text)
imgs = html.xpath("//div[@class='page-content text-center']//img[@class!='gif']")
for img in imgs:
	img_url = img.get('data-original')
	alt = img.get('alt')

```

- 获取文本，是通过`xpath`中的`text()`函数。示例代码如下：

```python
address = tr.xpath("./td[4]/text()")[0]

```

- 在某个标签下，再执行xpath函数，获取这个标签下的子孙元素，那么应该在斜杠之前加一个点`.` ，代表是在当前元素下获取。示例代码如下：

```python
 address = tr.xpath("./td[4]/text()")[0]
#  address = tr.xpath("./td[4]//text()")[0] 子孙标签的所有文本
```

> 需要注意的知识点：
>
> - `/` 和`//` 的区别：`/` 代表只获取直接子节点。`//` 获取子孙节点。一般`//` 用得比较多。当然也要视情况而定。
> - `contains`：有时候某个属性中包含了多个值，那么可以使用`contains`函数。示例代码如下：`//div[contains(@class,'job_detail')]`
> - 谓词中的下标是从1开始的，不是从0开始的。
>   





## 练习

```python
#encoding: utf-8

from lxml import etree

# 1. 获取所有tr标签
# 2. 获取第2个tr标签
# 3. 获取所有class等于even的tr标签
# 4. 获取所有a标签的href属性
# 5. 获取所有的职位信息（纯文本）

parser = etree.HTMLParser(encoding='utf-8')
html = etree.parse("tencent.html",parser=parser)

# 1. 获取所有tr标签
# //tr
# xpath函数返回的是一个列表
# trs = html.xpath("//tr")
# for tr in trs:
#     print(etree.tostring(tr,encoding='utf-8').decode("utf-8"))

# 2. 获取第2个tr标签
# tr = html.xpath("//tr[2]")[0]
# print(etree.tostring(tr,encoding='utf-8').decode("utf-8"))

# 3. 获取所有class等于even的tr标签
# trs = html.xpath("//tr[@class='even']")
# for tr in trs:
#     print(etree.tostring(tr,encoding='utf-8').decode("utf-8"))

# 4. 获取所有a标签的href属性
# aList = html.xpath("//a/@href")
# for a in aList:
#     print("http://hr.tencent.com/"+a)

# 5. 获取所有的职位信息（纯文本）
trs = html.xpath("//tr[position()>1]")
positions = []
for tr in trs:
    # 在某个标签下，再执行xpath函数，获取这个标签下的子孙元素
    # 那么应该在//之前加一个点，代表是在当前元素下获取
    href = tr.xpath(".//a/@href")[0]
    fullurl = 'http://hr.tencent.com/' + href
    title = tr.xpath("./td[1]//text()")[0]
    category = tr.xpath("./td[2]/text()")[0]
    nums = tr.xpath("./td[3]/text()")[0]
    address = tr.xpath("./td[4]/text()")[0]
    pubtime = tr.xpath("./td[5]/text()")[0]

    position = {
        'url': fullurl,
        'title': title,
        'category': category,
        'nums': nums,
        'address': address,
        'pubtime': pubtime
    }
    positions.append(position)

print(positions)

```



## 案例应用：抓取TIOBE指数前20名排行开发语言

```python
#!/usr/bin/env python
#coding:utf-8
import requests
from requests.exceptions import RequestException
from lxml import etree
from lxml.etree import ParseError
import json

def one_to_page(html):
    headers={
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36'
    }
    try:
        response=requests.get(html,headers=headers)
        body=response.text  #获取网页内容
    except RequestException as e:
        print('request is error!',e)
    try:
        html=etree.HTML(body,etree.HTMLParser())  #解析HTML文本内容
        result=html.xpath('//table[contains(@class,"table-top20")]/tbody/tr//text()') #获取列表数据
        pos = 0
        for i in range(20):
            if i == 0:
                yield result[i:5]
            else:
                yield result[pos:pos+5]  #返回排名生成器数据
            pos+=5
    except ParseError as e:
         print(e.position)


def write_file(data):   #将数据重新组合成字典写入文件并输出
    for i in data:
        sul={
            '2018年6月排行':i[0],
            '2017年6排行':i[1],
            '开发语言':i[2],
            '评级':i[3],
            '变化率':i[4]
        }
        with open('test.txt','a',encoding='utf-8') as f:
            f.write(json.dumps(sul,ensure_ascii=False) + '\n') #必须格式化数据
            f.close()
        print(sul)
    return None


def main():
    url='https://www.tiobe.com/tiobe-index/'
    data=one_to_page(url)
    revaule=write_file(data)
    if revaule == None:
        print('ok')
        
 
 
        
if __name__ == '__main__':
    main()



#
{'2018年6月排行': '1', '2017年6排行': '1', '开发语言': 'Java', '评级': '15.368%', '变化率': '+0.88%'}
{'2018年6月排行': '2', '2017年6排行': '2', '开发语言': 'C', '评级': '14.936%', '变化率': '+8.09%'}
{'2018年6月排行': '3', '2017年6排行': '3', '开发语言': 'C++', '评级': '8.337%', '变化率': '+2.61%'}
{'2018年6月排行': '4', '2017年6排行': '4', '开发语言': 'Python', '评级': '5.761%', '变化率': '+1.43%'}
{'2018年6月排行': '5', '2017年6排行': '5', '开发语言': 'C#', '评级': '4.314%', '变化率': '+0.78%'}
{'2018年6月排行': '6', '2017年6排行': '6', '开发语言': 'Visual Basic .NET', '评级': '3.762%', '变化率': '+0.65%'}
{'2018年6月排行': '7', '2017年6排行': '8', '开发语言': 'PHP', '评级': '2.881%', '变化率': '+0.11%'}
{'2018年6月排行': '8', '2017年6排行': '7', '开发语言': 'JavaScript', '评级': '2.495%', '变化率': '-0.53%'}
{'2018年6月排行': '9', '2017年6排行': '-', '开发语言': 'SQL', '评级': '2.339%', '变化率': '+2.34%'}
{'2018年6月排行': '10', '2017年6排行': '14', '开发语言': 'R', '评级': '1.452%', '变化率': '-0.70%'}
{'2018年6月排行': '11', '2017年6排行': '11', '开发语言': 'Ruby', '评级': '1.253%', '变化率': '-0.97%'}
{'2018年6月排行': '12', '2017年6排行': '18', '开发语言': 'Objective-C', '评级': '1.181%', '变化率': '-0.78%'}
{'2018年6月排行': '13', '2017年6排行': '16', '开发语言': 'Visual Basic', '评级': '1.154%', '变化率': '-0.86%'}
{'2018年6月排行': '14', '2017年6排行': '9', '开发语言': 'Perl', '评级': '1.147%', '变化率': '-1.16%'}
{'2018年6月排行': '15', '2017年6排行': '12', '开发语言': 'Swift', '评级': '1.145%', '变化率': '-1.06%'}
{'2018年6月排行': '16', '2017年6排行': '10', '开发语言': 'Assembly language', '评级': '0.915%', '变化率': '-1.34%'}
{'2018年6月排行': '17', '2017年6排行': '17', '开发语言': 'MATLAB', '评级': '0.894%', '变化率': '-1.10%'}
{'2018年6月排行': '18', '2017年6排行': '15', '开发语言': 'Go', '评级': '0.879%', '变化率': '-1.17%'}
{'2018年6月排行': '19', '2017年6排行': '13', '开发语言': 'Delphi/Object Pascal', '评级': '0.875%', '变化率': '-1.28%'}
{'2018年6月排行': '20', '2017年6排行': '20', '开发语言': 'PL/SQL', '评级': '0.848%', '变化率': '-0.72%'}
```







## 使用requests和xpath爬取电影天堂



````python
import requests
from lxml import etree

BASE_DOMAIN = 'http://www.dytt8.net'
HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36',
    'Referer': 'http://www.dytt8.net/html/gndy/dyzz/list_23_2.html'
}

def spider():
    url = 'http://www.dytt8.net/html/gndy/dyzz/list_23_1.html'
    resp = requests.get(url,headers=HEADERS)
    # resp.content：经过编码后的字符串
    # resp.text：没有经过编码，也就是unicode字符串
    # text：相当于是网页中的源代码了
    text = resp.content.decode('gbk')
    # tree：经过lxml解析后的一个对象，以后使用这个对象的xpath方法，就可以
    # 提取一些想要的数据了
    tree = etree.HTML(text)
    # xpath/beautifulsou4
    all_a = tree.xpath("//div[@class='co_content8']//a")
    for a in all_a:
        title = a.xpath("text()")[0]
        href = a.xpath("@href")[0]
        if href.startswith('/'):
            detail_url = BASE_DOMAIN + href
            crawl_detail(detail_url)
            break

def crawl_detail(url):
    resp = requests.get(url,headers=HEADERS)
    text = resp.content.decode('gbk')
    tree = etree.HTML(text)
    create_time = tree.xpath("//div[@class='co_content8']/ul/text()")[0].strip()
    imgs = tree.xpath("//div[@id='Zoom']//img/@src")
    # 电影海报
    cover = imgs[0]
    # 电影截图
    screenshoot = imgs[1]
    # 获取span标签下所有的文本
    infos = tree.xpath("//div[@id='Zoom']//text()")
    for index,info in enumerate(infos):
        if info.startswith("◎年　　代"):
            year = info.replace("◎年　　代","").strip()

        if info.startswith("◎豆瓣评分"):
            douban_rating = info.replace("◎豆瓣评分",'').strip()
            print(douban_rating)

        if info.startswith("◎主　　演"):
            # 从当前位置，一直往下面遍历
            actors = [info]
            for x in range(index+1,len(infos)):
                actor = infos[x]
                if actor.startswith("◎"):
                    break
                actors.append(actor.strip())
            print(",".join(actors))


if __name__ == '__main__':
    spider()

````



## 参考

1. <a href="https://blog.csdn.net/zgkli6com/article/details/17491479" target="_blank">XPath语法详解</a> 

2. <a href="https://blog.csdn.net/qq_31518167/article/details/78555076" target="_blank">XPath-语法大全</a>  

3. XPath的更多用法参考：http://www.w3school.com.cn/xpath/index.asp

   python lxml库的更多用法参考：http://lxml.de/

4. [python3解析库lxml](https://www.cnblogs.com/zhangxinqi/p/9210211.html) 

