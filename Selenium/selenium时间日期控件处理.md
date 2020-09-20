# selenium时间日期控件处理

## 背景介绍

我们在使用selenium爬取数据时，有时会需要选择日期，来获取某个时间段的数据。但是网上的日期控件还真是五花八门，有正常一点的：

<img src=".\img\image-20200814085540238.png" alt="image-20200814085540238" style="zoom:80%;" />

- 淘宝联盟上的有这样的：

<img src=".\img\image-20200814085614880.png" alt="image-20200814085614880" style="zoom:80%;" />

当然还有这样的：

<img src=".\img\image-20200814085631719.png" alt="image-20200814085631719" style="zoom:80%;" />

- 简单点的，我们还可以模拟鼠标点击，拖动的方式。但是复杂点的，那就完蛋了，该怎么办呢？

- 其实很简单，不管我们是通过什么方式选的，最终往服务器上发送的都是我们选定的日期数据，那么我们就不用去搞什么时间日期控件了，好好研究一下，我们手动选择的日期数据，是存储在页面的哪个位置，又是如何发送给服务器的，那就简单了。

- 可喜的是，大部分的日期控件，我们都可以把它当成一个普通的input框处理，对value进行赋值操作。

  

<img src=".\img\image-20200814085702752.png" alt="image-20200814085702752" style="zoom:80%;" />

也有一些类型的input框都是禁止手动输入的，那就用 js 代码把禁止输入的readonly属性去掉就好。

1. 可以直接输入值，没有 readonly 属性的，直接输入值就可以了
2. 有 readonly 属性的，先用 js 去掉 readonly 属性，然后直接输入日期文本内容：


```javascript
# 介绍4中操作方法
# js = "document.getElementById('txtBeginDate').removeAttribute('readonly')"  # 1.原生js，移除属性
# js = "$('input[id=txtBeginDate]').removeAttr('readonly')"  # 2.jQuery，移除属性
# js = "$('input[id=txtBeginDate]').attr('readonly',false)"  # 3.jQuery，设置为false
js = "$('input[id=txtBeginDate]').attr('readonly','')"  # 4.jQuery，设置为空（同3）
```

## 使用js方法输入日期

```python
from selenium import webdriver
import time
driver = webdriver.Chrome()
url = "https://www.12306.cn/index/"
driver.get(url)
time.sleep(5)

# 处理时间
# js 去掉 readonly 属性
js = 'document.getElementById("train_date").removeAttribute("readonly");'
driver.execute_script(js)

# js 添加时间
js_value = 'document.getElementById("train_date").value="2017-12-10"'
driver.execute_script(js_value)
```



参考

- <a href="https://blog.csdn.net/zwq912318834/article/details/80975748" target="_blank">python下selenium如何处理日期控件的几种方法</a>
- <a href="https://blog.csdn.net/showgea/article/details/79929092?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param" target="_blank">python selenium 时间日期控件处理</a> 

