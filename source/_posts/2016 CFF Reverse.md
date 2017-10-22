---
title: 2016 CFF Reverse
tags: [write-up]
date: 2017-03-03 21:12
---
CFF黑客秀的题目 题目做起来也不错分享一下题解
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="1软件密码破解1">1.软件密码破解_1</h1>
<p>首先利用PEID查看有无加壳 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303205552194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
无壳可以正常破解</br></img></br></p>
<h2 id="1利用od初步调试">（1）利用OD初步调试</h2>
<p>首先查找可以利用的字符串进行定位，但是在string中没有找到有用信息 <br>
于是单步调试查找输入字符串的地方，找到了下图 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303210046217?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
下断点分析后续代码，没有找到判断的地方。</br></img></br></br></p>
<h2 id="2ida查找函数">（2）IDA查找函数</h2>
<p>在IDA中找了几个函数，一个文本输入，一个点击按钮 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303210812511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303210820449?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
试了下都不行</br></img></br></img></br></p>
<h2 id="3od输入后查找字符串">（3）OD输入后查找字符串</h2>
<p>寻找突破点，输入了1234 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303210913527?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>下断点 ，这下找到了判断函数，估计代码写的时候运用了多线程，使得在主线程中找不到判断的相关信息 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170303211029621?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h2 id="4分析算法">（4）分析算法</h2>
<p>算法很简单简单的异或，然后内存值的比对</p>
<pre class="prettyprint"><code class=" hljs oxygene"><span class="hljs-keyword">xor</span> = [<span class="hljs-number">0</span>x28, <span class="hljs-number">0</span>x57, <span class="hljs-number">0</span>x64, <span class="hljs-number">0</span>x6B, <span class="hljs-number">0</span>x93, <span class="hljs-number">0</span>x8F, <span class="hljs-number">0</span>x65, <span class="hljs-number">0</span>x51, <span class="hljs-number">0</span>xE3, <span class="hljs-number">0</span>x53, <span class="hljs-number">0</span>xE4, <span class="hljs-number">0</span>x4E, <span class="hljs-number">0</span>x1A, <span class="hljs-number">0</span>xFF]

<span class="hljs-keyword">result</span> = [<span class="hljs-number">0</span>x1B, <span class="hljs-number">0</span>x1C, <span class="hljs-number">0</span>x17, <span class="hljs-number">0</span>x46, <span class="hljs-number">0</span>xF4, <span class="hljs-number">0</span>xFD, <span class="hljs-number">0</span>x20, <span class="hljs-number">0</span>x30, <span class="hljs-number">0</span>xB7, <span class="hljs-number">0</span>x0C, <span class="hljs-number">0</span>x8E, <span class="hljs-number">0</span>x7E, <span class="hljs-number">0</span>x78, <span class="hljs-number">0</span>xDE]

flag = <span class="hljs-string">''</span>

<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(len(<span class="hljs-keyword">result</span>)):
    flag += chr(<span class="hljs-keyword">xor</span>[i] ^ <span class="hljs-keyword">result</span>[i])

print flag</code></pre></div>