# selenium控制浏览器滚动条

### 指定下拉距离

需求：部分网站的数据是随着滚动条向下挥动加载的，爬取数据的需要将本页数据全部加载才可以进行翻页爬取。

目的：通过selenium控制浏览器滚动条
原理：通过 `driver.execute_script()`执行 js 代码，达到目的



```javascript
 driver.execute_script("window.scrollBy(0,1000)")
```

语法：`scrollBy(x,y)`

参数

- x		必需。向右滚动的像素值。
- y		必需。向下滚动的像素值。

或者使用

```python
js="var q=document.documentElement.scrollTop=10000"
driver.execute_script(js)
```



示例：

```python
from selenium import webdriver
from lxml import etree
from time import sleep

url = 'https://search.jd.com/Search?keyword=mac&enc=utf-8&wq=mac&pvid=9862d03c24e741c6a58079d004f5aabf'

chrome = webdriver.Chrome()
chrome.get(url)

js = 'document.documentElement.scrollTop=100000'
chrome.execute_script(js)
sleep(3)
html = chrome.page_source
e = etree.HTML(html)
prices = e.xpath('//div[@class="gl-i-wrap"]/div[@class="p-price"]/strong/i/text()')
names = e.xpath('//div[@class="gl-i-wrap"]/div[@class="p-name p-name-type-2"]/a/em')

print(len(names))
for name, price in zip(names, prices):
    print(name.xpath('string(.)'), ":", price)
chrome.quit()
```



###  发送tab键，移动到目标元素

根据页面显示进行变通，发送tab键

可以通过tab键会切换到密码框中，所以根据此思路，在python中也可以发送tab键来切换，使元素显示.

用TAB键（或者向下键）

这里我们测试一下向下键（找到最底下的元素，离提交最近的，再来一次TAB），最后再点击：

```
from selenium.webdriver.common.keys import Keys
driver.find_element_by_id("id_login_method_0").send_keys(Keys.TAB)
```



- 可以发送tab键来切换页面按钮，达到下拉滚动条的目的。但是一定要注意的是，**指定的元素一定要能被TAB键选中，像输入框，超链接，Button等**。

  ```python
  
  from selenium import webdriver
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support.ui import WebDriverWait
  from selenium.webdriver.support import expected_conditions as EC
  from selenium.webdriver.common.action_chains import ActionChains
  from selenium.webdriver.common.keys import Keys
  from selenium.webdriver.support.ui import Select
  
  import time
  import random
  
  # 加载xpath插件
  chrome_options = webdriver.ChromeOptions()
  extension_path = 'D:/extension/XPath-Helper_v2.0.2.crx'
  chrome_options.add_extension(extension_path)
  
  browser = webdriver.Chrome(chrome_options=chrome_options)
  browser.maximize_window()
  wait = WebDriverWait(browser, 25)
  waitPopWindow = WebDriverWait(browser, 25)
  
  browser.get("https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Daps&field-keywords=phone")
  
  time.sleep(random.randrange(5, 10, 1))
  
  # 找到 Next Page 按钮， 属于可见元素
  
  # 指定元素是 超链接 ———— 可以用Tab键切换到
  targetElem = browser.find_element_by_xpath("//a[@id='pagnNextLink']")
  
  # 这个元素不是超链接，所以无法接收Tab键
  # targetElem = browser.find_element_by_xpath("//a[@id='pagnNextLink']/span[@id='pagnNextString']")
  
  targetElem.send_keys(Keys.TAB)
  
  print(f"结束拖动滚动条....")
  time.sleep(random.randrange(5, 10, 1))
  browser.quit()
  
  
  ```



### 拖动到指定元素位置。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select

import time
import random

#加载xpath插件
chrome_options = webdriver.ChromeOptions()
extension_path = 'D:/extension/XPath-Helper_v2.0.2.crx'
chrome_options.add_extension(extension_path)

browser = webdriver.Chrome(chrome_options=chrome_options)
#browser.maximize_window()
wait = WebDriverWait(browser, 25)
waitPopWindow = WebDriverWait(browser, 25)

browser.get("https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Daps&field-keywords=phone")

time.sleep(random.randrange(5, 10, 1))

#找到 Next Page 按钮， 属于可见元素
#js代码有两种写法，但是对元素的要求不同，focus更为严格

#第一种方法：focus
targetElem = browser.find_element_by_xpath("//a[@id='pagnNextLink']")
browser.execute_script("arguments[0].focus();", targetElem)

#第二种方法：scrollIntoView
#targetElem = browser.find_element_by_xpath("//a[@id='pagnNextLink']/span[@id='pagnNextString']")
#browser.execute_script("arguments[0].scrollIntoView();", targetElem)    # 拖动到可见的元素去

print(f"结束拖动滚动条....")
time.sleep(random.randrange(5, 10, 1))
browser.quit()


```



参考

- [selenium_通过selenium控制浏览器滚动条](https://www.jianshu.com/p/e2758e830120) 
- <a href="https://blog.csdn.net/qq_16069927/article/details/81074331" target="">selenium 如何控制滚动条逐步滚动</a> 
- <a href="https://blog.csdn.net/zwq912318834/article/details/79262007" target="">python + selenium + chrome 如何操作滚动条</a> 

