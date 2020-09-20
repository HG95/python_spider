# Settings

Scrapy设定(settings)提供了定制Scrapy组件的方法。您可以控制包括核心(core)，插件(extension)，pipeline及spider组件。

设定为代码提供了提取以key-value映射的配置值的的全局命名空间(namespace)。 设定可以通过下面介绍的多种机制进行设置。

设定(settings)同时也是选择当前激活的Scrapy项目的方法(如果您有多个的话)。

- `ROBOTSTXT_OBEY`
  设置为`False`。默认是True。即遵守机器协议，那么在爬虫的时候，scrapy首先去找robots.txt文件，如果没有找到。则直接停止爬取。

- `DEFAULT_REQUEST_HEADERS`
  添加`User-Agent`。这个也是告诉服务器，我这个请求是一个正常的请求，不是一个爬虫。

  ```python
  # Override the default request headers:
  DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.84 Safari/537.36'
  }
  
  ```

  

- `ITEM_PIPELINES`
  启用一个Item Pipeline组件

  ```python
  ITEM_PIPELINES = {
      'myproject.pipelines.PricePipeline': 300,
      'myproject.pipelines.JsonWriterPipeline': 800,
  }
  ```

  分配给每个类的整型值，确定了他们运行的顺序，item按数字从低到高的顺序，通过pipeline，通常将这些数字定义在0-1000范围内。

- `DOWNLOAD_DELAY`
  默认: `0`

  下载器在下载同一个网站下一个页面前需要等待的时间。该选项可以用来限制爬取速度， 减轻服务器压力。同时也支持小数:

  ```python
  DOWNLOAD_DELAY = 0.25    # 250 ms of delay
  ```

  



更多参考

<a href="https://scrapy-chs.readthedocs.io/zh_CN/latest/topics/settings.html#topics-settings-ref" target="_blank">Scrapy-Settings</a>

