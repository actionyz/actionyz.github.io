---
title: HackingLab 综合关
tags: [write-up]
date: 2016-12-21 19:58
---
第一关1.首先注册账号  然后让你注册手机号，但是题目提示源码中有内部人员的手机号 于是查看源代码号码为13388453871 通过burpsuit发现该请求有三个字段 于是绑定成功，想想我们将该手机号绑在了admin的账号上。下一步就很明显了，利用查找密码功能找回admin密码。 2.利用Forgetpassword？ 通过用户名：admin；密码：XXXXXXXX 登录进去 即可
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="第一关">第一关</h1>
<p>1.首先注册账号 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161221194136979?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
然后让你注册手机号，但是题目提示源码中有内部人员的手机号 <br>
于是查看源代码号码为13388453871 <br>
通过burpsuit发现该请求有三个字段 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161221195704413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></img></br></p>
<p>于是绑定成功，想想我们将该手机号绑在了admin的账号上。下一步就很明显了，利用查找密码功能找回admin密码。 <br>
2.利用Forgetpassword？ <br>
通过用户名：admin；密码：XXXXXXXX 登录进去 <br>
即可发现flag <br>
key is yesBindphoneErrorGood</br></br></br></br></p>
<h1 id="第二关">第二关</h1>
<p>这个题有点坑，看了题解还原了一下，才知道是什么原因。不说了看题吧 <br>
首先有个登录界面 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161222225435390?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
扫描了一下发现了用户名和密码test:test <br>
登陆之后发现 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161222225601194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
然后就不知道怎么办了，看了题解说是有robots.txt。以后先看有没有这个~ <br>
发现./myadminroot目录进去之后 <br>
发现要登录而且是admin登录，之后怎么注入，怎么登都不行看了题解才知道哦 <br>
1. 先用admin账户登录 <br>
2. 不用管弹框直接去./myadminroot目录即可 <br>
执行步骤如下图 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161222232237108?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
之后就是key了 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161222232346699?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></br></br></br></br></br></img></br></br></br></img></br></br></p>
<h2 id="题目复现">题目复现</h2>
<p>制作了一个登录页面来模仿 <br>
主要代码是</br></p>
<pre class="prettyprint"><code class=" hljs bash">    require(<span class="hljs-string">'conn.php'</span>);
    session_start();
    <span class="hljs-variable">$query_user</span>=<span class="hljs-string">"select * from user where username = '<span class="hljs-variable">$username</span>' and pass = '<span class="hljs-variable">$passwd</span>'"</span>;   
    <span class="hljs-variable">$result</span> = mysqli_query(<span class="hljs-variable">$connect</span>,<span class="hljs-variable">$query_user</span>);
    <span class="hljs-variable">$num_results</span>=<span class="hljs-variable">$result</span>-&gt;num_rows;
    <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isLogin'</span>]=<span class="hljs-number">1</span>;
    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$num_results</span> == <span class="hljs-number">0</span>)   
    {
        <span class="hljs-built_in">echo</span> <span class="hljs-string">'login fail!!'</span>;
        <span class="hljs-built_in">echo</span> <span class="hljs-string">'&lt;script&gt;alert("false");window.location.href="./session.php";&lt;/script&gt;'</span>;   //重点在这里，点击确定之后将session赋值为<span class="hljs-number">0</span>

//      header(<span class="hljs-string">"Location: http://baidu.com"</span>);
    }</code></pre>
<pre class="prettyprint"><code class=" hljs xml">./session.php
<span class="php"><span class="hljs-preprocessor">&lt;?php</span>
    session_start();
    <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isLogin'</span>]=<span class="hljs-number">0</span>;
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>总感觉这题怪怪的</p></div>