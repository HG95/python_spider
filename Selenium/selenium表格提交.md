# selenium表格提交

 1.第一步都是 import

```python
from selenium import webdriver
import time
driver = webdriver.Chrome()
```

2.登录，如果不需要登录则可以跳过这一步：先通过右键查看源码（关键段）

```html
<label class="control-label" for="loginform-employee_no">管理帐号</label>
<input type="text" id="loginform-employee_no" class="form-control" name="LoginForm[employee_no]">
 
<p class="help-block help-block-error"></p>
</div>                            <div class="form-group field-loginform-password required">
<label class="control-label" for="loginform-password">密码</label>
<input type="password" id="loginform-password" class="form-control" name="LoginForm[password]">
```

selenium的工作原理就是首先定位一个元素，然后再对其进行操作。Lucky，这段源码有id值，我们知道在html里id都是唯一的。因此这里我们找到的两个ID：`id="loginform-employee_no" id="loginform-password" `可以用 `elem_user = driver.find_element_by_id `这一方法来定位。

然后是操作部分：用keys类来模拟键盘输入send（也可以是回车或者空格等等操作），先import Keys类：

```python
from selenium.webdriver.common.keys import Keys
```

综合一下：

```python
elem_user = driver.find_element_by_id("loginform-employee_no")  
elem_user.send_keys("用户名")  
elem_pwd = driver.find_element_by_id("loginform-password")  
elem_pwd.send_keys("密码")  
elem_pwd.send_keys(Keys.RETURN)  #return和enter都是回车键，只是表达不同。

time.sleep(2)  #sleep这一步跟网络环境有关，有时候如果输入太快，可能会引起报错。单位为秒。后面不再重复写。
```

3.登录后就可以开始我们的填表了，直接开车（drive）到填表页面（仅针对URL不变的填表地址，如果url是变动的需要另外处理这步driver.get）：

```python
driver.get("http://表单网址")
```

<img src=".\img\20170529154749023.png" alt="20170529154749023" style="zoom:80%;" />

我们来看下这个需要填的内容：

**等级：** 这个内容是个下拉菜单的类型：

```python
<select id="employee-worker_level" class="form-control" name="Employee[worker_level]">
<option value="兼职">兼职</option>
<option value="见习">见习</option>
<option value="初级">初级</option>
<option value="中级">中级</option>
<option value="高级">高级</option>
</select>
```

好消息是这个下拉选择框有id，那么我么就可以像刚才那样的定位，但是坏消息是，这个下拉选择框的选项没有id，这时我们可以另一个定位方法，用`tag name`, `find_elements_by_tag_name` , 这里是 ”option“，并且是以类似 list 的方式存在的，所以引用标签是0-4 代表着五个选项。所以这里我们如果要选选项4的代码如下：

```python
driver.find_element_by_id("employee-worker_level")
			.find_elements_by_tag_name("option")[3].click()
```

这里的click代表的是左键单击。

**工作角色：**这个是个checkbox类的提交内容：

```html
<div class="help-block"></div>
</div>    <div class="form-group field-employee-worker_role">
<label class="control-label" for="employee-worker_role">工作角色</label>
<input type="hidden" name="Employee[worker_role]" value=""><div id="employee-worker_role"><label><input type="checkbox" name="Employee[worker_role][]" value="1"> 宣传人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="2"> 收费人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="3"> 维护人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="4"> 安装人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="5"> 移机人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="9"> 综合人员</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="10"> 投诉专岗</label>
<label><input type="checkbox" name="Employee[worker_role][]" value="11"> 新开销售</label></div>
<div class="hint-block">选择人员的工作角色，如果非实际工作人员，请留空</div>
<div class="help-block"></div>
</div>
```

这里也是类似的，我们看到有label这个tag name所以可以直接用label name，但是，如果我们需要复选的话，很简单，用个循环就行了：

```python
i=0
while (i<8) :
    driver.find_element_by_id("employee-worker_role").find_elements_by_tag_name("label")[i].click()
    i=i+1
```

其他的需要填的内容就和登录里需要的是类似的。

最后我们只需要用keys类回车一下，就可以提交表格了。

**bonus：**

好奇提交表格的时候能不能用click动作来完成，走向了作死之路：

我们先来看下提交的代码：

```html
<div class="form-group">
    <button type="submit" class="btn btn-success">添加</button>    </div>

</form></div>
```

等等，id呢？name呢？让我们看下其他的定位方法：

通过id定位元素：find_element_by_id("id_vaule")
通过name定位元素：find_element_by_name("name_vaule")
通过tag_name定位元素：find_element_by_tag_name("tag_name_vaule")
通过class_name定位元素：find_element_by_class_name("class_name")
通过css定位元素：find_element_by_css_selector();
通过xpath定位元素：find_element_by_xpath("xpath")
通过link定位：find_element_by_link_text("text_vaule")或者find_element_by_partial_link_text()
好的，貌似目前只能用Xpath的方法来定位了

这里有一个教程：http://www.w3school.com.cn/xpath/index.asp 





