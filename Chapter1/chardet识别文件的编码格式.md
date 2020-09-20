# 识别文件的编码格式

chardet库文档:https://chardet.readthedocs.io/en/latest/usage.html

## 模块介绍

`Chardet`：通用字符编码检测器
检测字符集范围：

```python
ASCII，UTF-8，UTF-16（2种变体），UTF-32（4种变体）
Big5，GB2312，EUC-TW，HZ-GB-2312，ISO-2022-CN（繁体中文和简体中文）
EUC-JP，SHIFT_JIS，CP932，ISO-2022-JP（日文）
EUC-KR，ISO-2022-KR（韩文）
KOI8-R，MacCyrillic，IBM855，IBM866，ISO-8859-5，windows-1251（西里尔文）
ISO-8859-5，windows-1251（保加利亚语）
ISO-8859-1，windows-1252（西欧语言）
ISO-8859-7，windows-1253（希腊语）
ISO-8859-8，windows-1255（视觉和逻辑希伯来语）
TIS-620（泰国语）d'y

```

当python程序中某一个数据文件不知道编码时，可使用chardet第三方库来检测，代码如下（path中填对应文件路径即可)

```python
import chardet
 
if __name__ == '__main__':
	path='***'
	f=open(path,'rb')
	data=f.read()
	print(chardet.detect(data))
	# {'language': '', 'confidence': 0.73, 'encoding': 'Windows-1252'}

```

`detect函数`只需要一个 非unicode字符串参数，返回一个字典。该字典包括判断到的编码格式及判断的置信度。
`chardet.detect()` 的返回值，为一个字典：

```python
{'language': '', 'confidence': 0.73, 'encoding': 'Windows-1252'}

```

得到文件的编码方式，可以才采用字典的方式

```python
  codedetect = chardet.detect(data)["encoding"]    #检测得到编码方式

```



将获取的内容进行解码

```python
code = chardet.detect(response.content)["encoding"]  # 获取编码格式
response.encoding = code  # 指定编码格式
return response.text
```

