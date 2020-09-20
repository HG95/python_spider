# fake-useragent

Python库(`fake-useragent`),可以随机生成各种UserAgent

安装 `pip install fake-useragent`

**基本使用**

```python
from fake_useragent import UserAgent
ua = UserAgent()
```

```python
from fake_useragent import UserAgent
ua = UserAgent()

ua.ie
# Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US);
ua.msie
# Mozilla/5.0 (compatible; MSIE 10.0; Macintosh; Intel Mac OS X 10_7_3; Trident/6.0)'
ua['Internet Explorer']
# Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB7.4; InfoPath.2; SV1; .NET CLR 3.3.69573; WOW64; en-US)
ua.opera
# Opera/9.80 (X11; Linux i686; U; ru) Presto/2.8.131 Version/11.11
ua.chrome
# Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.2 (KHTML, like Gecko) Chrome/22.0.1216.0 Safari/537.2'
ua.google
# Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4) AppleWebKit/537.13 (KHTML, like Gecko) Chrome/24.0.1290.1 Safari/537.13
ua['google chrome']
# Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11
ua.firefox
# Mozilla/5.0 (Windows NT 6.2; Win64; x64; rv:16.0.1) Gecko/20121011 Firefox/16.0.1
ua.ff
# Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:15.0) Gecko/20100101 Firefox/15.0.1
ua.safari
# Mozilla/5.0 (iPad; CPU OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A5355d Safari/8536.25

# and the best one, random via real world browser usage statistic
ua.random
```

**注意:**

- fake-useragent 将收集到的数据缓存到temp文件夹, 例如 /tmp, 更新数据:

```python
from fake_useragent import UserAgent
ua = UserAgent()
ua.update()
```

- 有时候会因为网络或者其他问题,出现异常(`fake_useragent.errors.FakeUserAgentError: Maximum amount of retries reached`), 可以禁用服务器缓存

```python
from fake_useragent import UserAgent
ua = UserAgent(use_cache_server=False)
```

- 可以自己添加本地数据文件

```python
import fake_useragent

# I am STRONGLY!!! recommend to use version suffix
location = '/home/user/fake_useragent%s.json' % fake_useragent.VERSION

ua = fake_useragent.UserAgent(path=location)
ua.random
```



参考

1. <a href="https://pypi.org/project/fake-useragent/" target="_blank">fake-useragent 0.1.11</a> 