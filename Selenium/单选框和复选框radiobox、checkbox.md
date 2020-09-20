# 单选框和复选框(radiobox、checkbox)

## 常规的单(复)选框

一、认识单选框和复选框

  1.先认清楚单选框和复选框长什么样

<img src=".\img\image-20200814103225810.png" alt="image-20200814103225810" style="zoom:80%;" />

上面的单选框是圆的；下图复选框是方的

二、radio和checkbox源码

1.上图的html源码如下，把下面这段复杂下来，写到文本里，后缀改成.html就可以了。

```html
<html>  
    <head>  
    <meta http-equiv="content-type" content="text/html;charset=utf-8" />  
    <title>单选和复选</title>  
    </head>  
    <body>  
    
    </form>  
    <h4>单选：性别</h4>   
    <form>  
    <label value="radio">男</label>   
    <input name="sex" value="male" id="boy" type="radio"><br>  
    <label value="radio1">女</label>  
    <input name="sex" value="female" id="girl" type="radio">  
    </form>  
    
    <h4>微信公众号：从零开始学自动化测试</h4>  
    <form>  
    <!-- <label for="c1">checkbox1</label> -->  
    <input id="c1" type="checkbox">selenium<br>  
    <!-- <label for="c2">checkbox2</label> -->  
    <input id="c2" type="checkbox">python<br>  
    <!-- <label for="c3">checkbox3</label> -->  
    <input id="c3" type="checkbox">appium<br>  
    
    <!-- <form>  
    <input type="radio" name="sex" value="male" /> Male  
    <br />  
    <input type="radio" name="sex" value="female" /> Female  
    </form> -->  
      
    </body>  
    </html> 
```

三、单选：radio

 1.首先是定位选择框的位置

<img src=".\img\image-20200814103353678.png" alt="image-20200814103353678" style="zoom:80%;" />

 2.定位id，点击图标就可以了，代码如下（获取url地址方法：把上面源码粘贴到文本保存为.html后缀后用浏览器打开，在浏览器url地址栏复制出地址就可以了）

 3.先点击boy后，等十秒再点击girl，观察页面变化

<img src=".\img\image-20200814103422124.png" alt="image-20200814103422124" style="zoom:80%;" />

四、复选框：checkbox

 1.勾选单个框，比如勾选selenium这个，可以根据它的id=c1直接定位到点击就可以了

<img src=".\img\image-20200814103448688.png" alt="image-20200814103448688" style="zoom:80%;" />

那么问题来了:如果想全部勾选上呢？

五、全部勾选：

  1.全部勾选，可以用到定位一组元素，从上面源码可以看出，复选框的type=checkbox,这里可以用xpath语法：.//*[@type='checkbox']

 <img src=".\img\image-20200814103534652.png" alt="image-20200814103534652" style="zoom:80%;" />

2.这里注意，敲黑板做笔记了：**`find_elements`**是不能直接点击的，它是复数的，所以只能先获取到所有的checkbox对象，然后通过for循环去一个个点击操作

六、判断是否选中：`is_selected()`

  1.有时候这个选项框，本身就是选中状态，如果我再点击一下，它就反选了，这可不是我期望的结果，那么可不可以当它是没选中的时候，我去点击下；当它已经是选中状态，我就不点击呢？那么问题来了：如何判断选项框是选中状态？

  2.判断元素是否选中这一步才是本文的核心内容，点击选项框对于大家来说没什么难度。获取元素是否为选中状态，打印结果如下图。

  3.返回结果为bool类型，没点击时候返回False,点击后返回True，接下来就很容易判断了，既可以作为操作前的判断，也可以作为测试结果的判断

<img src=".\img\image-20200814103638111.png" alt="image-20200814103638111" style="zoom:80%;" />

代码：

```python
# coding:utf-8
from selenium import webdriver
driver = webdriver.Firefox()
driver.get("file:///C:/Users/Gloria/Desktop/checkbox.html")
# 没点击操作前，判断选项框状态
s = driver.find_element_by_id("boy").is_selected()
print s
driver.find_element_by_id("boy").click()
# 点击后，判断元素是否为选中状态
r = driver.find_element_by_id("boy").is_selected()
print r

# 复选框单选
driver.find_element_by_id("c1").click()
# 复选框全选
checkboxs = driver.find_elements_by_xpath(".//*[@type='checkbox']")
for i in checkboxs:
    i.click()
```

## 特殊的单(复)选框

例如 12306 的复选框

<img src=".\img\image-20200814104240145.png" alt="image-20200814104240145" style="zoom:80%;" />

标签没有 `type="checkbox"` 属性，而是通过改变 `class` 的值，来设置是否勾选：

<img src=".\img\image-20200814104347415.png" alt="image-20200814104347415" style="zoom:80%;" />

解决方案：

用 `driver.execute_script()`这个方法来执行 js 语句, 改变属性的值，达到勾选的目的。

代码：

```python
from selenium import webdriver
import time
from selenium.webdriver.common.action_chains import ActionChains

driver = webdriver.Chrome()
url = "https://www.12306.cn/index/"
driver.get(url)
time.sleep(5)

# 设置出发地
s = driver.find_element_by_id('fromStationText')
ActionChains(driver).move_to_element(s) \
    .click(s) \
    .send_keys_to_element(s, "南京") \
    .move_by_offset(20, 50) \
    .click() \
    .perform()

# 设置目的地
t = driver.find_element_by_id('toStationText')
ActionChains(driver).move_to_element(t) \
    .click(t) \
    .send_keys_to_element(t, "北京") \
    .move_by_offset(20, 50) \
    .click() \
    .perform()


# 处理时间
# js 去掉 readonly 属性
js = 'document.getElementById("train_date").removeAttribute("readonly");'
driver.execute_script(js)

# js 添加时间
js_value = 'document.getElementById("train_date").value="2020-08-20"'
driver.execute_script(js_value)


js = "document.getElementById('isStudentDan').setAttribute('class', 'active')"
driver.execute_script(js)
```

<img src=".\img\12306.gif" alt="12306" style="zoom:80%;" />





参考

- <a href="https://www.it-swarm.dev/zh/testing/selenium：是否可以在selenium中设置webelement的任何属性值？/941081452/" target="_blank">Selenium：是否可以在Selenium中设置WebElement的任何属性值？</a>
- <a href="https://www.cnblogs.com/wanghaihong200/p/8466545.html " target="_blank">Selenium2学习（十五）-- 单选框和复选框（radiobox、checkbox）</a> 