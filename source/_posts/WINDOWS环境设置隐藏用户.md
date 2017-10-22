---
title: WINDOWS环境设置隐藏用户
tags: [windows]
date: 2017-02-25 00:08
---
WINDOWS环境设置隐藏用户
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h2 id="1实验目的">1.实验目的</h2>
<p>在windows环境下，新建用户，并且实现隐藏，已达到不被用户发现的目的，可以实现随时登录电脑。</p>
<h2 id="2实验方法">2.实验方法</h2>
<ol>
<li>新建用户 <br>
利用命令行创建用户并提升至管理员权限 <br>
net user yz5<span>$</span> 123456 /add &amp; net localgroup administrators yz5$ /add <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224233832948?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></li>
<li>用户隐藏 <br>
新建的用户我们可以再控制面板中找到，但是命令net user却不显示 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224233952982?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224234051624?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
每个用户在注册表中都有注册，我们的目的是要让控制面板中的用户不显示，但也可以让用户登录。打开注册表，发现所有的用户都在HKEY_LCAL_MACHINE/SAM/SAM/Domains/Accout/Users <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224234129593?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
显然知道000001F4是管理员，000003F3同时也是，把3F3的F键值改为管理员的3F3键值（这一步是为了在控制面板隐藏掉用户yz5$，不影响正常的登录），然后将其导出 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224234651695?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
删除注册表中的用户信息 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224234818291?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
再从桌面上双击图标，导入数据库。yz5$键值导入成功</br></img></br></br></img></br></br></img></br></br></img></br></img></br></br></li>
<li>登录隐藏用户 <br>
在此基础上，cmd及控制面板已经看不到新建的用户了。那么问题来了，我们怎么登录呢。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224235346727?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
可以采用一些比较黑科技的方法，这里采用了修改登录页面的方法。如果是固定的则永远登不上去。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224235618244?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
必须把登录页面写成可以直接输入用户名、密码的。利用gpedit.msc 将不显示最后的用户名状态改为 启用 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170224235900900?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170225000958477?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170225001153079?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></img></br></br></img></br></br></img></br></br></li>
</ol>
<h2 id="3实验总结">3.实验总结</h2>
<p>总体来说，基本的功能已经实现。通过实验进一步学习到了许多命令行命令。当然这个实验也不是十分的完美，比如在计算机管理页面，有个新建的用户就会被发现。还要有很长的路要走，实现的功能还很多。</p></div>