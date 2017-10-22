---
title: 2017 X-NUCA 代码审计
tags: [write-up]
date: 2017-08-26 22:06
---
今天的比赛中的一道代码审计的题目。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>今天的比赛中的一道代码审计的题目。</p>
<h1 id="step-1-初步审计">step 1 初步审计</h1>
<p>发现所有的数据库操作都是 PDO操作，这也就意味着不可能是SQL注入了。查找flag出现的位置</p>
<p><code>do_changepass.php</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826214331523?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<code>user.php</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826214358772?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现如果是要获取flag 那么必须要更改userinfo数组的值····</br></img></br></br></img></br></p>
<h1 id="step-2-代码回溯">step 2 代码回溯</h1>
<p>我们发现在上述两个页面userinfo的值就是session的值，所以目标转化为更改session值，找到登录后的操作 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826215547172?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>在这里发现了<code>$_session[userinfo]</code>的赋值操作$userinfo，本来userinfo是数组，我们在这里有个变量覆盖 <br>
<code>common.php</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826215143656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<p>我们如果传入<code>?userinfo=a</code>那么就是一个字符串这时 <br>
<code>$userinfo["id"]=$userinfo[0]=a</code></br></p>
<p>我们看register.php <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826215734073?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<code>role = 1</code></br></img></br></p>
<p>所以<code>$_session[userinfo]='1'</code> session是字符串1</p>
<p>这时再次访问do_changepass.php即可</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170826220556116?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p></div>