---
title: Python 2.7.8 手动安装 & Python 环境变量添加
tags: [软件及应用配置]
date: 2017-06-06 19:29
---
Python 2.7.8 手动安装 & Python 环境变量添加
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="0x01-python-278-手动安装">0x01 Python 2.7.8 手动安装</h1>
<h2 id="0x1-源码">0x1 源码</h2>
<p><code>https://www.python.org/download/releases/2.7.8</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170606191630640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
倒数第2个</br></img></br></p>
<h2 id="0x2-解压">0x2 解压</h2>
<p><code>xz -d Python-2.7.6.tar.xz</code> <br>
<code>tar xvf Python-2.7.6.tar</code></br></p>
<h2 id="0x3-编译">0x3 编译</h2>
<h3 id="1-执行configure脚本">(1) 执行configure脚本</h3>
<p><code>./configure</code></p>
<h3 id="2-编译源代码">(2) 编译源代码</h3>
<p><code>make</code></p>
<h3 id="3-安装编译好的软件">(3) 安装编译好的软件</h3>
<p><code>sudo make install</code></p>
<p>发现一直编译失败 此时必须删除以前的python文件 <br>
分别存在<code>/usr/lib/python2.x or <br>
/usr/local/lib/python2.x)</br></code> <br>
上述步骤只是完成了python的安装但 没有设置python环境变量 ，导致pip安装的模块import无法找到。下面介绍一下环境变量的添加</br></br></p>
<h1 id="0x02-python-环境变量添加">0x02 Python 环境变量添加</h1>
<p>当import 模块的时候python会便利寻找 sys.path 下的内容 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170606192803669?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
因此如果要想自己的模块直接可以import 需要添加python环境变量</br></img></br></p>
<h3 id="1-永久添加-当前用户">(1) 永久添加 （当前用户）</h3>
<p>在用户主目录下有一个 .bashrc 隐藏文件，可以在此文件中加入 PATH 的设置如下： <br>
<code>vim ~/.bashrc</code> <br>
添加 <br>
<code>export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:$PYTHONPATH</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170606192417211?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<code>source ~/.bashrc</code></br></img></br></br></br></br></p>
<h3 id="2-永久添加-所有用户">(2) 永久添加 （所有用户）</h3>
<p><code>vim  /etc/profile</code> <br>
<code>export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:$PYTHONPATH</code> <br>
需重启才能生效</br></br></p>
<h3 id="2-临时添加">(2) 临时添加</h3>
<p>当前终端下输入 <br>
<code>export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:$PYTHONPATH</code></br></p></div>