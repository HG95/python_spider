# Chrome无头模式与操作窗口

## Chrome无头模式–headless

**在做爬虫时，通常是不需要打开浏览器的，只需要使用浏览器的内核，因此可以使用Chrome的无头模式** 

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument('--headless')
driver = webdriver.Chrome(chrome_options=chrome_options)
driver.get("http://www.baidu.com")
driver.close()
```

## 操作浏览器窗口

```python
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get("http://www.baidu.com")
```

- 全屏

  ```python
  driver.fullscreen_window()
  ```

- 最大化窗口

  ```python
  driver.maximize_window()
  ```

- 最小化窗口

  ```python
  driver.minimize_window()
  ```

- 获取窗口位置

- 获取窗口大小

- 同时获取窗口位置和大小

  ```python
  print(driver.get_window_position())
  print(driver.get_window_size())
  print(driver.get_window_rect())
  ```

- 设置窗口位置

- 设置窗口大小

- 同时设置窗口位置和大小

  ```python
  driver.set_window_position(30,30)
  driver.set_window_size(500,500)
  driver.set_window_rect(10,10,1050,708)
  ```

  

参考：

- <a href="https://blog.csdn.net/bqw18744018044/article/details/81351137" target="_blank">Python爬虫之selenium库(三)：Chrome无头模式与操作浏览器</a>