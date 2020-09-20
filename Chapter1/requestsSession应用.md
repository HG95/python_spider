# requests.Session应用

## session与cookie存储

 会话对象requests.Session能够跨请求地保持某些参数，比如cookies，即在同一个Session实例发出的所有请求都保持同一个cookies,而requests模块每次会自动处理cookies，这样就很方便地处理登录时的cookies问题。

如果你向同一主机发送多个请求，每个请求对象让你能够跨请求保持session和cookie信息，这时我们要使用到requests的Session()来保持回话请求的cookie和session与服务器的相一致。



## 模拟登录V站

本篇文章的任务是利用request.Session模拟登录V2EX（http://www.v2ex.com/）这个网站，即V站。

工具： Python 3.5，BeautifulSoup模块，requests模块，Chrome

这个网站登录的时候抓到的数据如下：

<img src=".\img\image-20200813230948205.png" alt="image-20200813230948205" style="zoom:80%;" />



其中用户名(u)、密码(p)都是明文传输的，很方便。once的话从分析登录URL： http://www.v2ex.com/signin 的源文件（下图）可以看出，应该是每次登录的特有数据，我们需要提前把它抓出来再放到Form Data里面POST给网站。

<img src=".\img\image-20200813231004951.png" alt="image-20200813231004951" style="zoom:80%;" />

 抓出来还是老方法，用BeautifulSoup神器即可。这里又学到一种抓标签里面元素的方法，比如抓上面的"value",用soup.find('input',{'name':'once'})['value']即可

即抓取含有 name="once"的input标签中的value对应的值。

于是构建postData,然后POST。

怎么显示登录成功呢？这里通过访问 http://www.v2ex.com/settings 即可，因为这个网址没有登录是看不了的：

<img src=".\img\image-20200813231023337.png" alt="image-20200813231023337" style="zoom:80%;" />

经过上面的分析，写出源代码:

```python
import requests
from bs4 import BeautifulSoup
 
url = "http://www.v2ex.com/signin"
UA = "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.13 Safari/537.36"
 
header = { "User-Agent" : UA,
           "Referer": "http://www.v2ex.com/signin"
           }
 
v2ex_session = requests.Session()
f = v2ex_session.get(url,headers=header)
 
soup = BeautifulSoup(f.content,"html.parser")
once = soup.find('input',{'name':'once'})['value']
print(once)
 
postData = { 'u': 'whatbeg',
             'p': '*****',
             'once': once,
             'next': '/'
             }
 
v2ex_session.post(url,
                  data = postData,
                  headers = header)
 
f = v2ex_session.get('http://www.v2ex.com/settings',headers=header)
print(f.content.decode())
```

然后运行发现成功登录：

<img src=".\img\image-20200813231115412.png" alt="image-20200813231115412" style="zoom:80%;" />





## 拉勾网数据爬取

拉钩网的 cookies 是动态变化的，传入的时候不要写固定，用 requests.session() 来保存动态的 cookies。

```python
import requests

headers = {
    'Accept': 'application/json, text/javascript, */*; q=0.01',
    'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'zh-CN,zh;q=0.9',
    'Connection': 'keep-alive',
    'Content-Length': '25',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    'Host': 'www.lagou.com',
    'Origin': 'https://www.lagou.com',
    'Referer': 'https://www.lagou.com/jobs/list_Python?labelWords=&fromSearch=true&suginput=',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-origin',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36',
    'X-Anit-Forge-Code': '0',
    'X-Anit-Forge-Token': 'None',
    'X-Requested-With': 'XMLHttpRequest'
}

base_url2 = 'https://www.lagou.com/jobs/positionAjax.json?needAddtionalResult=false'

base_url = 'https://www.lagou.com/jobs/positionAjax.json?'
page = 1
params = {

    'city': '北京',
    'needAddtionalResult': 'false'
}
form_data = {
    'first': 'false',
    'pn': '3',
    'kd': 'python'
}
session = requests.session()
session.get(base_url2, headers={
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36'
})

response = session.post(url=base_url
                        , headers=headers
                        , params=params
                        , data=form_data
                        , allow_redirects=False

                        )

print(response.json())
```



参考

- <a href="https://blog.csdn.net/xiaozhanger/article/details/78034015" target="_blank">Python模拟登录(一) requests.Session应用</a>

