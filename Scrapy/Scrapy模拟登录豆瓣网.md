# Scrapy模拟登录豆瓣网

```python
# -*- coding: utf-8 -*-
import scrapy
from urllib import request
from PIL import Image
from urllib import request
from base64 import b64encode
import requests


class DoubanSpider(scrapy.Spider):
    name = 'douban'
    allowed_domains = ['douban.com']
    profile_url = 'https://www.douban.com/people/97956064/'
    login_url = 'https://accounts.douban.com/login'
    editsignature_url = 'https://www.douban.com/j/people/97956064/edit_signature'

    # 登录的 url
    start_urls = ["https://accounts.douban.com/login"]

    def parse(self, response):
        # 进入等率界面获取所需要的表单数据
        # 登录需要的表单数据
        formdata = {
            'source': 'None',
            'redir': 'https://www.douban.com/',
            'form_email': '970138074@qq.com',
            'form_password': 'pythonspider',
            'remember': 'on',
            'login': '登录'
        }

        # 获取验证码图片的 url
        captcha_url = response.css('img#captcha_image::attr(src)').get()
        # 判断是否需要验证码
        if captcha_url:
            """
            captcha-solution, captcha-id 都是动态变化的，需要临时获取
            """

            # 识别验证码
            captcha = self.regonize_captcha(captcha_url)
            # 追加表单数据
            formdata['captcha-solution'] = captcha

            captcha_id = response.xpath("//input[@name='captcha-id']/@value").get()
            formdata['captcha-id'] = captcha_id
            yield scrapy.FormRequest(url=self.login_url,
                                     formdata=formdata,
                                     callback=self.parse_after_login)

    def parse_after_login(self, response):
        if response.url == 'https://www.douban.com/':
            yield scrapy.Request(self.profile_url, callback=self.parse_profile)
            print('登录成功！')
        else:
            print('登录失败！')

    def parse_profile(self, response):
        print(response.url)
        if response.url == self.profile_url:
            ck = response.xpath("//input[@name='ck']/@value").get()
            formdata = {
                'ck': ck,
                'signature': '我可以自动识别图形验证码啦~~'
            }
            yield scrapy.FormRequest(self.editsignature_url, formdata=formdata, callback=self.parse_none)
        else:
            print('没有进入到个人中心')

    def parse_none(self,response):
        pass

    def regonize_captcha(self, image_url):
        captcha_url = image_url
        # 将验证码图片保存到本地
        request.urlretrieve(captcha_url, 'captcha.png')

        # 阿里云识别验证码的 网站
        recognize_url = 'http://jisuyzmsb.market.alicloudapi.com/captcha/recognize?type=e'

        formdata = {}
        with open('captcha.png', 'rb') as fp:
            data = fp.read()
            pic = b64encode(data)
            formdata['pic'] = pic

        appcode = '831a890b2cfe4ea0a8e345078434ebfc'
        headers = {
            'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
            'Authorization': 'APPCODE ' + appcode
        }

        # 识别的结果，json格式的文件
        response = requests.post(recognize_url, data=formdata, headers=headers)
        result = response.json()
        # 获取验证码
        code = result['result']['code']
        return code

    # 认为识别验证码
    # def regonize_captcha(self,image_url):
    #     request.urlretrieve(image_url, 'captcha.png')
    #     image = Image.open('captcha.png')
    #     image.show()
    #     image.close()
    #     captcha = input('请输入验证码：')
    #     return captcha
```

