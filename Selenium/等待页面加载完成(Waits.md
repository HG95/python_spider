# 等待页面加载完成(Waits)

selenium 的显示等待与隐式等待

现在很多的网页都采用了 Ajax 技术，那么采用一般的静态爬虫技术会出现抓取不到页面的元素。比如歌曲的主页会有评论数量，一般评论数量是动态加载的。
所以这就涉及到selenium,支持各种浏览器，包括Chrome，Safari，Firefox 等主流界面式浏览器，如果你在这些浏览器里面安装一个 Selenium 的插件，那么便可以方便地实现Web界面的测试。

```python
driver = webdriver.Chrome()
driver.get("http://somedomain/url_that_delays_loading")
driver.page_source #获取网页渲染后的源代码
```

动态加载的页面需要时间等待页面上的所有元素都渲染完成，如果在没有渲染完成之前我们就`switch_to_`或者是`find_elements_by_`，那么就可能出现元素定位困难而且会提高产生 ElementNotVisibleException 的概率。
直接找到我们要抓取的tag或者直接没有等待元素出来就开始交互导致不起作用的问题。

## 隐式等待

如果某些元素不是立即可用的，隐式等待是告诉WebDriver去等待一定的时间后去查找元素。 默认等待时间是0秒，一旦设置该值，隐式等待是设置该WebDriver的实例的生命周期。

```python
from selenium import webdriver

driver = webdriver.Firefox()
driver.implicitly_wait(10) # seconds
driver.get("http://somedomain/url_that_delays_loading")
myDynamicElement = driver.find_element_by_id("myDynamicElement")
```

## 显示等待

显式等待是你在代码中定义等待一定条件发生后再进一步执行你的代码。 最糟糕的案例是使用time.sleep()，它将条件设置为等待一个确切的时间段。 这里有一些方便的方法让你只等待需要的时间。WebDriverWait结合ExpectedCondition 是实现的一种方式。

指定某个条件，然后设置最长等待时间。如果在这个时间还没有找到元素，那么便会抛出异常。只有该条件触发，才执行后续代码，这个使用更灵活。
**主要涉及到selenium.webdriver.support 下的expected_conditions类**。

example

```python
driver = webdriver.Chrome()
driver.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "myDynamicElement"))
    )
finally:
    driver.quit()
```

example2

```python
wait_result = 
WebDriverWait(driver=self.driver, timeout=300, poll_frequency=0.5, ignored_exceptions=None)
.until( EC.text_to_be_present_in_element((By.XPATH, '//*[@id="VolumeTable"]/tbody/tr[1]/td[4]/label'), u'可用'))
```

这里的 `presence_of_element_located(())`、`text_to_be_present_in_element(())`是其中两种方式，其实还有其他相关方式。EC 配合使用的 until() 或者 until_not() 方法说明：

```python
until(method, message='')
调用该方法体提供的回调函数作为一个参数，直到返回值为True
until_not(method, message='')
调用该方法体提供的回调函数作为一个参数，直到返回值为False

```

模块包含一套预定义的条件集合。大大方便了 WebDriverWait 的使用。
Expected Conditions 类提供的预期条件判断方法

在进行浏览器自动化的时候，有一些条件是经常出现的，下面列出的是每个条件的实现。Selenium Python binding provides some convienence 提供了很多实用的方法。

```
title_is：判断当前页面的title是否等于预期

title_contains：判断当前页面的title是否包含预期字符串

presence_of_element_located：判断某个元素是否被加到了dom树里，并不代表该元素一定可见

visibility_of_element_located：判断某个元素是否可见. 可见代表元素非隐藏，并且元素的宽和高都不等于0

visibility_of：跟上面的方法做一样的事情，只是上面的方法要传入locator，这个方法直接传定位到的element就好了

presence_of_all_elements_located：判断是否至少有1个元素存在于dom树中。举个例子，如果页面上有n个元素的class都是'column-md-3'，那么只要有1个元素存在，这个方法就返回True

text_to_be_present_in_element：判断某个元素中的text是否 包含 了预期的字符串

text_to_be_present_in_element_value：判断某个元素中的value属性是否包含了预期的字符串

frame_to_be_available_and_switch_to_it：判断该frame是否可以switch进去，如果可以的话，返回True并且switch进去，否则返回False

invisibility_of_element_located：判断某个元素中是否不存在于dom树或不可见

element_to_be_clickable - it is Displayed and Enabled：判断某个元素中是否可见并且是enable的，这样的话才叫clickable

staleness_of：等某个元素从dom树中移除，注意，这个方法也是返回True或False

element_to_be_selected：判断某个元素是否被选中了,一般用在下拉列表

element_located_to_be_selected

element_selection_state_to_be：判断某个元素的选中状态是否符合预期

element_located_selection_state_to_be：跟上面的方法作用一样，只是上面的方法传入定位到的element，而这个方法传入locator

alert_is_present：判断页面上是否存在alert

```

参数1：By类确定哪种选择方式

```
from selenium.webdriver.common.by import By
```

参数2：值，可能是 `xpath`的值，可能是id,name等，取决于前面是`By.XPATH`, `By.ID`究竟是哪种方式去定位元素。

可以在 `WebDriverWait()`构造时传入下面参数，哪一个浏览器，来控制超时时间，多长时间检测一次这个元素是否加载，是否有异常报出。

- `driver`：浏览器驱动
- `timeout`：最长超时等待时间
- `poll_frequency`：检测的时间间隔，默认为500ms
- `ignore_exception`：超时后抛出的异常信息，默认情况下抛 NoSuchElementException 异常



基本使用：

```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait  # 显示等待
from selenium.webdriver.support import expected_conditions as EC  # 设置等待执行语句
from selenium.webdriver.common.by import By
from selenium.common.exceptions import TimeoutException
from time import sleep
url = 'http://tieba.baidu.com/p/5923312469'
options = webdriver.ChromeOptions()
options.add_extension(r'D:\python_demo\AdBlock_v3.10.0.crx')  # 添加插件
driver = webdriver.Chrome(chrome_options=options)
driver.set_page_load_timeout(10)
try:
    driver.get(url)
    WebDriverWait(driver, 10).until(
        # 必须是元组
        EC.presence_of_element_located((By.CLASS_NAME, 'core_title_txt  ')) 
    )
    print(driver.page_source)
except TimeoutException as e:
    print('错误1')
    print(e)
    print(driver.page_source)
```





## 案例

爬取地理坐标

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import re, pandas as pd
def coordinate(site):
    # 创建浏览器驱动对象
    driver = webdriver.Firefox()
    driver.get('http://api.map.baidu.com/lbsapi/getpoint/index.html')
    # 显式等待，设置timeout
    wait = WebDriverWait(driver, 9)
    # 判断输入框是否加载
    input = wait.until(
        EC.presence_of_element_located(
            (By.CSS_SELECTOR, '#localvalue')))
    # 判断搜索按钮是否加载
    submit = wait.until(
        EC.element_to_be_clickable(
            (By.CSS_SELECTOR, '#localsearch')))
    # 输入搜索词，点击搜索按钮
    input.clear()
    input.send_keys(site)
    submit.click()
    # 等待坐标
    wait.until(
        EC.presence_of_element_located(
            (By.CSS_SELECTOR, '#no_0')))
    # 获取网页文本，提取经纬度
    source = driver.page_source
    xy = re.findall('坐标：([\d.]+),([\d.]+)', source)
    # 转浮点数，取中位数
    df = pd.DataFrame(xy, columns=['longitude', 'latitude'])
    df['longitude'] = pd.to_numeric(df['longitude'])
    df['latitude'] = pd.to_numeric(df['latitude'])
    longitude = df['longitude'].median()
    latitude = df['latitude'].median()
    # 关闭浏览器驱动
    driver.close()
    return [longitude, latitude]
print(coordinate('南海桂城地铁站'))
# [113.1611575, 23.044811000000003]
```



参考

- <a href="https://blog.csdn.net/qq_27717921/article/details/77429512" target="_blank">selenium 的显示等待与隐式等待</a> 