---
title: Python Django项目开发（WIFI-SNIFF）
tags: [python]
date: 2017-02-18 14:53
---
因为数据收集的代码是利用Python写的，因此想到了利用Django进行项目开发，实现的具体功能是在局域网内进行信息嗅探，并将嗅探得到的信息展示在WEB网页上。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>因为数据收集的代码是利用Python写的，因此想到了利用Django进行项目开发，实现的具体功能是在局域网内进行信息嗅探，并将嗅探得到的信息展示在WEB网页上。</p>
</blockquote>
<h2 id="一数据嗅探模块">一.数据嗅探模块</h2>
<h3 id="1利用ettercap进行局域网欺骗">1.利用ettercap进行局域网欺骗</h3>
<p>首先主机利用ARP欺骗，使得所有的数据包都经过攻击机。（工具选择ettercap） <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170217211641365?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h3 id="2python模块利用">2.python模块利用</h3>
<p>python在数据包截获方面有现成的模块，例如pcap。</p>
<pre class="prettyprint"><code class=" hljs vala"><span class="hljs-preprocessor">#listen to sniff packet</span>
pc = pcap.pcap()
b = <span class="hljs-string">'tcp port 80'</span>
pc.setfilter(b)
<span class="hljs-preprocessor">#可以直接对经过网卡的所有数据包进行捕获</span></code></pre>
<p>在数据包分析方面，例如dpkt。</p>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">eth</span> = dpkt.ethernet.<span class="hljs-type">Ethernet</span>(buf)
<span class="hljs-title">ip</span> = eth.<span class="hljs-typedef"><span class="hljs-keyword">data</span></span>
<span class="hljs-title">tcp</span> = ip.<span class="hljs-typedef"><span class="hljs-keyword">data</span></span>
<span class="hljs-preprocessor">#可以把截获的数据包分层处理 如上所示分成 以太网层，IP层，tcp层······</span></code></pre>
<p>对数据中URL的分析就是利用http层的协议内容进行截获分析</p>
<h3 id="3数据包分析">3.数据包分析</h3>
<p>通过python现有的模块将所截获的数据包分层处理。 <br>
目前对用户访问的URL的分析是通过http协议中的host头部获得。 <br>
用户的种类的分析也是根据http协议user-agent头部获得。 <br>
对于图片的的截取，目前做到的是匹配关键字的方法获得（匹配是否有image、jpg、png等，下一阶段直接从数据包中获得图片数据）。</br></br></br></p>
<h2 id="二数据库模块">二.数据库模块</h2>
<h3 id="1数据库设计">1.数据库设计</h3>
<p>Django框架自带sqlite数据库可以直接使用。 <br>
设计一个用于存储用户信息的数据表victim</br></p>
<pre class="prettyprint"><code class=" hljs autohotkey">字段包括 序号、用户种类、mac地址、连接时间、IP地址
CREATE TABLE <span class="hljs-string">"victim"</span> (
    <span class="hljs-escape">`i</span>d<span class="hljs-escape">` </span>   INTEGER,
    <span class="hljs-escape">`u</span>seragent<span class="hljs-escape">` </span>TEXT,
    <span class="hljs-escape">`m</span>ac<span class="hljs-escape">` </span>  TEXT,
    <span class="hljs-escape">`t</span>ime<span class="hljs-escape">` </span> TEXT,
    <span class="hljs-escape">`i</span>p<span class="hljs-escape">` </span>   TEXT
)</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218105859035?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
设计记录url的数据表Record_urls</br></img></p>
<pre class="prettyprint"><code class=" hljs autohotkey">字段包括 序号、访问的URL、访问URL对应的时间
CREATE TABLE <span class="hljs-escape">`R</span>ecord_urls<span class="hljs-escape">` </span>(
    <span class="hljs-escape">`i</span>d<span class="hljs-escape">` </span>   INTEGER,
    <span class="hljs-escape">`u</span>rl<span class="hljs-escape">` </span>  TEXT,
    <span class="hljs-escape">`t</span>ime<span class="hljs-escape">` </span> TEXT
)</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218105913176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h3 id="2嗅探代码与数据库的连接">2.嗅探代码与数据库的连接</h3>
<p>嗅探到的数据信息通过Python语句存储至sqlite数据库，方便web网页访问。</p>
<pre class="prettyprint"><code class=" hljs avrasm">通过该语句连接至数据库
<span class="hljs-preprocessor">#link to database</span>
conn = sqlite3<span class="hljs-preprocessor">.connect</span>(<span class="hljs-string">"db.sqlite3"</span>) 
并通过下面的语句访问数据库
conn<span class="hljs-preprocessor">.execute</span>(sql1)
conn<span class="hljs-preprocessor">.commit</span>()</code></pre>
<h2 id="三web页面模块">三.WEB页面模块</h2>
<h3 id="1django开发简介">1.Django开发简介</h3>
<blockquote>
<p>Django是一个开放源代码的Web应用框架，由Python写成。采用了MVC的框架模式，即模型M，视图V和控制器C。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。框架把控制层给封装了，无非与数据交互这层都是数据库表的读,写,删除,更新的操作.在写程序的时候，只要调用相应的方法就行了，感觉很方便。程序员把控制层东西交给Django自动完成了。 只需要编写非常少的代码完成很多的事情。所以，它比MVC框架考虑的问题要深一步，因为我们程序员大都在写控制层的程序。现在这个工作交给了框架，仅需写很少的调用代码，大大提高了工作效率。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218111452151?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
用户访问的目录存在urls中</br></img></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> url
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> yz <span class="hljs-keyword">import</span> views <span class="hljs-keyword">as</span> yz_views
urlpatterns = [
    url(<span class="hljs-string">r'^admin/'</span>, admin.site.urls),
    url(<span class="hljs-string">r'^$'</span>,yz_views.index),
    url(<span class="hljs-string">r'^login.html$'</span>,yz_views.login),
    url(<span class="hljs-string">r'^index.html'</span>,yz_views.index),
    url(<span class="hljs-string">r'^test.html$'</span>,yz_views.test),
    url(<span class="hljs-string">r'^show_url.html$'</span>,yz_views.show_url),
    url(<span class="hljs-string">r'^image.html$'</span>,yz_views.image),
]   
</code></pre>
<p>只有目录还不行，必须有新建的项目下views的引导。</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">index</span><span class="hljs-params">(request)</span>:</span>
    ······
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test</span><span class="hljs-params">(request)</span>:</span>
    ······
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">show_url</span><span class="hljs-params">(request)</span>:</span>
    ······
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image</span><span class="hljs-params">(request)</span>:</span>
    ······
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">login</span><span class="hljs-params">(request)</span>:</span>
    ······</code></pre>
<p>templates里面存放的views中解析的静态网页 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218125711837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h3 id="2web页面功能介绍">2.WEB页面功能介绍</h3>
<h4 id="1登录页面">（1）登录页面</h4>
<p>设计了登录页面，因为网站在局域网内可以被访问，提高网站的安全性。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218142810144?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h4 id="2用户展示">（2）用户展示</h4>
<p>受害者用户名单展示平台 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218142826347?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h4 id="3url展示">（3）URL展示</h4>
<p>点击其中一个用户的URL，可进入展示该用户访问所有URL的页面 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218142841416?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h4 id="4image展示">（4）image展示</h4>
<p>点击其中一个用户的image，可进入展示该用户访问部分image的页面 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170218143728099?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h2 id="四后期工作">四.后期工作</h2>
<h2 id="1完善图片截获功能">1.完善图片截获功能</h2>
<p>目前只能从访问连接中识别是否是图片链接，下一步实现直接从数据包结构中分析出图片数据。</p></div>