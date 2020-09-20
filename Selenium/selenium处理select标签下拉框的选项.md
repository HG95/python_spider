# selenium处理select标签下拉框的选项

## 1. 背景

- 在爬取网页是，有时候我们会遇到下图中的下拉框，也就是**< select > < /select >**标签。按照一般的点击方案是无法成功的，而selenium提供了专门的Select类来处理这种下拉框。

<img src=".\img\image-20200814153303033.png" alt="image-20200814153303033" style="zoom:80%;" />

## 2. select下拉框的处理方案

上图中的网页源代码如下：

```html
<select aria-describedby="searchDropdownDescription" class="nav-search-dropdown searchSelect" data-nav-digest="RflwCB5c/WnDs5EtBNnFFArlBJk" data-nav-selected="0" id="searchDropdownBox" name="url" style="display: block; top: 0px;" tabindex="18" title="Search in">
<option selected="selected" value="search-alias=aps">All Departments</option>
<option value="search-alias=alexa-skills">Alexa Skills</option>
<option value="search-alias=amazon-devices">Amazon Devices</option>
<option value="search-alias=instant-video">Amazon Video</option>
<option value="search-alias=warehouse-deals">Amazon Warehouse Deals</option>
<option value="search-alias=appliances">Appliances</option>
<option value="search-alias=mobile-apps">Apps &amp; Games</option>
<option value="search-alias=arts-crafts">Arts, Crafts &amp; Sewing</option>
<option value="search-alias=automotive">Automotive Parts &amp; Accessories</option>
<option value="search-alias=baby-products">Baby</option>
<option value="search-alias=beauty">Beauty &amp; Personal Care</option>
<option value="search-alias=stripbooks">Books</option>
<option value="search-alias=popular">CDs &amp; Vinyl</option>
<option value="search-alias=mobile">Cell Phones &amp; Accessories</option>
<option value="search-alias=fashion">Clothing, Shoes &amp; Jewelry</option>
<option value="search-alias=fashion-womens">&nbsp;&nbsp;&nbsp;Women</option>
<option value="search-alias=fashion-mens">&nbsp;&nbsp;&nbsp;Men</option>
<option value="search-alias=fashion-girls">&nbsp;&nbsp;&nbsp;Girls</option>
<option value="search-alias=fashion-boys">&nbsp;&nbsp;&nbsp;Boys</option>
<option value="search-alias=fashion-baby">&nbsp;&nbsp;&nbsp;Baby</option>
<option value="search-alias=collectibles">Collectibles &amp; Fine Art</option>
<option value="search-alias=computers">Computers</option>
<option value="search-alias=courses">Courses</option>
<option value="search-alias=financial">Credit and Payment Cards</option>
<option value="search-alias=digital-music">Digital Music</option>
<option value="search-alias=electronics">Electronics</option>
<option value="search-alias=lawngarden">Garden &amp; Outdoor</option>
<option value="search-alias=gift-cards">Gift Cards</option>
<option value="search-alias=grocery">Grocery &amp; Gourmet Food</option>
<option value="search-alias=handmade">Handmade</option>
<option value="search-alias=hpc">Health, Household &amp; Baby Care</option>
<option value="search-alias=local-services">Home &amp; Business Services</option>
<option value="search-alias=garden">Home &amp; Kitchen</option>
<option value="search-alias=industrial">Industrial &amp; Scientific</option>
<option value="search-alias=digital-text">Kindle Store</option>
<option value="search-alias=fashion-luggage">Luggage &amp; Travel Gear</option>
<option value="search-alias=luxury-beauty">Luxury Beauty</option>
<option value="search-alias=magazines">Magazine Subscriptions</option>
<option value="search-alias=movies-tv">Movies &amp; TV</option>
<option value="search-alias=mi">Musical Instruments</option>
<option value="search-alias=office-products">Office Products</option>
<option value="search-alias=pets">Pet Supplies</option>
<option value="search-alias=prime-exclusive">Prime Exclusive Savings</option>
<option value="search-alias=pantry">Prime Pantry</option>
<option value="search-alias=software">Software</option>
<option value="search-alias=sporting">Sports &amp; Outdoors</option>
<option value="search-alias=tools">Tools &amp; Home Improvement</option>
<option value="search-alias=toys-and-games">Toys &amp; Games</option>
<option value="search-alias=vehicles">Vehicles</option>
<option value="search-alias=videogames">Video Games</option>
</select>
```

### 第一种方案：select_by_value

- 使用条件：用于选取< option>标签的value值，要求必须要有value属性，当然，这不是废话嘛…。

  ```python
  from selenium import webdriver
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support.ui import WebDriverWait
  from selenium.webdriver.support import expected_conditions as EC
  from selenium.webdriver.common.action_chains import ActionChains
  from selenium.webdriver.common.keys import Keys
  from selenium.webdriver.support.ui import Select
  import random
  import time
  
  
  browser = webdriver.Chrome()
  browser.get("https://www.amazom.com/")
  
  # 选中最后一个选项
  # <option value="search-alias=videogames">Video Games</option>
  classSelectValue = 'search-alias=videogames'
  Select(browser.find_element_by_tag_name("select")).select_by_value(classSelectValue)
  
  # 拿到搜索框
  input = browser.find_element_by_xpath("//form[@name='site-search']/div[@class='nav-fill']//input[@class='nav-input']")
  
  # 输入搜索关键字
  time.sleep(random.randrange(1, 5, 1))
  input.clear()
  input.send_keys('mount')
  
  # 敲enter键
  input.send_keys(Keys.RETURN)
  ```

  

### 第二种方案：select_by_index

使用条件：要求下拉框的选项必须要有**index属性**，例如index=”1”。注意，这不是数组下标值，而是属性值！

```python
# 稍微变一下，这种方法在这不合适，仅仅是mark一下
Select(browser.find_element_by_tag_name("select")).select_by_index(2)
```

### 第三种方案：select_by_visible_text

使用条件：用于选取< option>标签的 text 值！

```python
# <option value="search-alias=toys-and-games">Toys &amp; Games</option>
# 一定要注意是源代码中的text值，而不要从页面去复制，区别就在于，源代码中会有&amp
classSelectText = 'Toys &amp; Games'
Select(browser.find_element_by_tag_name("select")).select_by_visible_text(classSelectText )
```



参考

- <a href="https://blog.csdn.net/zwq912318834/article/details/79197114" target="_blank">selenium + python处理select标签下拉框的选项</a>